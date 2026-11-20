# Code Analysis for Software Security Engineering — Team 3 (Keycloak)

<!--
TEAM NOTE (doesn't render on GitHub, only visible while editing):
- This file is the deliverable, so only put in what the grader should read. Owners, status and checklists live in the issues/project board (milestone "Assignment - Code Analysis for Software Security Engineering").
- Due Mon Dec 7, 11:59 PM. The Canvas submission is a link to this file.
- Scope is the same as before: Keycloak's authorization and credential subsystem, through our five interactions. Each of us reviews the code behind our own interaction.
- Nothing in the strategy section is set in stone. Anyone can change their topic or focus in #90, and the scope table, CWE checklist and tools change with it.
- Pin every Keycloak link to commit d052236, the same commit as our Design observations, so line numbers match what we cite. Open the file at that commit on GitHub, click the line number, and use "Copy permalink".
- The assignment asks for 5 to 10 CWEs, so the checklist stays at 10 or fewer. Swap one out rather than adding an 11th.
- There's no minimum number of rows, findings or CWEs per person. The checklist is a menu, not a quota. Manual review is 25 pts, and a "No weakness found" row counts as long as it says what we checked and links the lines.
- The assignment allows AI chatbots for manual review. Use them to walk through unfamiliar code, but every row needs a line link you read yourself.
- Automated scanning is 25 pts and needs links to tool outputs. One person runs both scans and commits the raw outputs to code-analysis/. Each of us then lists the findings in our own interaction's code that matter (real issues, checklist CWEs, high severity), each with a triage state and a one-line justification. The totals sentence and the raw output links cover everything else, so nobody triages every hit.
- Part 2 risk levels are for our operational environment from the Project Proposal: about 2,500 employees, Keycloak as the single sign-on for HR and payroll, the service desk and finance, employees federated from Active Directory, and contractors coming through a partner identity provider.
- Anything that looks exploitable goes through Keycloak's private process in SECURITY.md (keycloak-security@googlegroups.com), not a public issue or PR. Keycloak rejects reports based only on scanner or AI output that come without a proof of concept. Hardening and documentation ideas can go public.
- Put the milestone on your PR as well as your issue so the milestone page shows our reviews. Team collaboration is 20 pts.
- Owner names and @ tags come out of the final file.

Files:
- code-analysis/semgrep.sarif, code-analysis/semgrep.txt (Semgrep raw output)
- code-analysis/codeql.sarif, code-analysis/codeql.csv (CodeQL raw output)
-->

## Part 1 — Code Review

### Code review strategy

Our review is scenario-based and checklist-based. It is scenario-based because we review only the code behind our five interactions, starting from the classes our Design observations cited and the threats we left as **Needs Investigation** or **Partial**. It is checklist-based because each of us checks that code against the same CWE checklist below. Most gaps in our Design deliverable were insecure defaults and missing audit records, which scanners don't look for, so manual review carries most of the weight. Two scanners cover the injection and signature weaknesses they are good at finding. Every Keycloak link is pinned to `main` at commit [`d052236`](https://github.com/keycloak/keycloak/commit/d052236442de082d459a404ce453713b36549594), the same commit as our Design observations.

#### Code in scope

| # | Interaction | Claim | Where we start | What we follow up from Design |
|---|---|---|---|---|
| 1 | End-user authentication (browser login + OTP) | Claim 1 | `LoginActionsService`, `SessionCodeChecks`, `AuthenticationProcessor`, `UsernamePasswordForm`, `OTPFormAuthenticator`, `OTPCredentialProvider`, `DefaultBruteForceProtector` | Forged, replayed or out-of-order login steps, OTP binding and reuse, lockout |
| 2 | Realm admin manages credentials and password policy | Claim 2 | `UserResource`, `UserPermissionEvaluator`, `UserPermissions`, `AdminEventBuilder`, `RealmManager`, action-token handlers, `PasswordCredentialProvider` | Who can reset whose password, admin event records, new-realm defaults, reset links |
| 3 | Client application token acquisition (OIDC authorization code flow) | Claim 3 | `AuthorizationEndpointChecker`, `PkceUtils`, `RedirectUtils`, `OAuth2CodeParser`, `AuthorizationCodeGrantType`, `TokenManager`, client authenticators | PKCE, redirect URI matching, code and refresh-token reuse, roles in tokens |
| 4 | Identity brokering from an external identity provider | Claim 4 | `SAMLEndpoint`, `SAMLIdentityProviderConfig`, `IdentityBrokerService`, identity provider mappers, first broker login authenticators | SAML signature, Issuer and replay checks, mapper role grants, account linking |
| 5 | Directory federation (LDAP / Active Directory) | Claim 5 | `LDAPStorageProviderFactory`, `LDAPConfig`, `LDAPContextManager`, `LDAPUtil`, LDAP role and group mappers, LDAP query escaping | LDAP over TLS, roles granted by AD group name, filter escaping, lockout before the LDAP bind |

#### CWE checklist

| CWE | Why it matters for our scope | Interactions | Checked by |
|---|---|---|---|
| [CWE-1188](https://cwe.mitre.org/data/definitions/1188.html): Initialization of a Resource with an Insecure Default | Our Design found that most protections exist but are off by default: brute-force detection, PKCE, SAML signature checks, LDAP over TLS and saved events. | 1–5 | Manual |
| [CWE-307](https://cwe.mitre.org/data/definitions/307.html): Improper Restriction of Excessive Authentication Attempts | Password and OTP guessing against employees and admins. Lockout also has to stop password spraying before it locks employees' AD accounts. | 1, 2, 5 | Manual |
| [CWE-841](https://cwe.mitre.org/data/definitions/841.html): Improper Enforcement of Behavioral Workflow | Login, OTP and first broker login are multi-step flows. A skipped or out-of-order step bypasses MFA or the account-linking check. | 1, 4 | Manual |
| [CWE-294](https://cwe.mitre.org/data/definitions/294.html): Authentication Bypass by Capture-replay | OTP codes, authorization codes, refresh tokens, reset links and SAML assertions should each work once, for the session they were issued to. | 1–4 | Manual |
| [CWE-347](https://cwe.mitre.org/data/definitions/347.html): Improper Verification of Cryptographic Signature | Keycloak should trust partner SAML assertions and signed client JWTs only after verifying their signatures. | 3, 4 | Manual + scanners |
| [CWE-601](https://cwe.mitre.org/data/definitions/601.html): URL Redirection to Untrusted Site ('Open Redirect') | A loose redirect URI match delivers authorization codes to an attacker's site. | 3 | Manual + scanners |
| [CWE-863](https://cwe.mitre.org/data/definitions/863.html): Incorrect Authorization | Admin permission checks decide who can reset whose password, including a realm admin's. | 2 | Manual |
| [CWE-266](https://cwe.mitre.org/data/definitions/266.html): Incorrect Privilege Assignment | Mappers and client scopes decide which roles a federated user or an access token carries, such as finance or payroll roles. | 3, 4, 5 | Manual |
| [CWE-778](https://cwe.mitre.org/data/definitions/778.html): Insufficient Logging | Repudiation was our weakest STRIDE category. Successful logins, token redemptions and mapper role grants leave no saved record by default. | 1, 2, 4, 5 | Manual |
| [CWE-90](https://cwe.mitre.org/data/definitions/90.html): Improper Neutralization of Special Elements used in an LDAP Query ('LDAP Injection') | Usernames typed at login become part of the LDAP search filters Keycloak sends to Active Directory. | 5 | Manual + scanners |

#### Automated scanning tools

Keycloak's server is Java, built with Maven. Its TypeScript admin and account consoles are outside our scope. Tools that need a compiled build first, such as SpotBugs and SonarQube's Java analysis, would mean building all of Keycloak, so we chose two tools that analyze Java source directly.

| Tool | Why we chose it |
|---|---|
| [Semgrep Community Edition](https://semgrep.dev/docs/languages/java), `p/java` and `p/findsecbugs` rulesets | Fast pattern rules for common Java weaknesses, with no build needed. It only looks at one file at a time. |
| [CodeQL](https://codeql.github.com/docs/), `java-security-extended` suite, `none` build mode | Tracks untrusted data across methods and files, which Semgrep Community Edition can't. Keycloak's CI already runs CodeQL on pull requests, so we expect few new findings from it. |

#### Challenges we expected

<!--
Built from the challenges each of us posted in the strategy issue before we started reviewing.
This one table answers both assignment questions: what we expected, and how the strategy addressed it.
-->

| Challenge we expected | How our strategy addressed it |
|---|---|
| TBD | TBD |

### Manual code review findings

<!--
One table per interaction. Every row traces back to a security requirement (SR-x.y), an assurance case node (C1.2, E3.4) or a TMT threat ID, and links the exact lines at d052236.
Result: Weakness / No weakness found / Config-dependent / Inconclusive.
-->

#### Interaction 1 — End-user authentication

TBD: one or two sentences on which critical security functions this covers and why.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD |

#### Interaction 2 — Realm admin manages credentials and password policy

TBD: one or two sentences on which critical security functions this covers and why.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD |

#### Interaction 3 — Client application token acquisition

Interaction 3 covers the code that decides where an authorization code goes, who can redeem it, how refresh tokens are protected from replay, which roles end up in a token, and how client secrets, signing keys and token traffic are protected. It follows the five misuse cases from our Requirements (intercept the code, omit or downgrade PKCE, manipulate the redirect URI, replay a stolen refresh token, win the refresh race), Claim 3, and the Interaction 3 threats our Design deliverable left Partial or Config-dependent.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| SR-3.4; C3.4, C3.5; TMT 89, 172 | CWE-601 | [`RedirectUtils`][m3-redirect] | Config-dependent | With no wildcard registered, the redirect URI has to match a registered one exactly (L287–L288). A registered `*` accepts any http or https redirect and skips the wildcard safety checks (L251–L253). A trailing `*` turns a registration into a path prefix match that ignores the query and fragment (L255–L273). The scheme, host and port still have to match (L338–L370), a `*` right after the host is treated as `/*` (L420–L423), wildcards are ignored when the path contains `..` (L226–L231), and redirect URIs carrying response parameters such as `code` or `state` are refused (L53–L62, L174–L187). Nothing blocks wildcard registrations unless the `secure-redirect-uris-enforcer` client policy is on. |
| SR-3.2; C3.2, C3.3; TMT 3 | CWE-1188 | [`AuthorizationEndpointChecker`][m3-aec] | Config-dependent | PKCE is only enforced when the client has a PKCE method configured (L343–L350). Otherwise an authorization request with no `code_challenge` goes through with only a debug log line (L449–L453), and `plain` is accepted as readily as `S256` (L455–L465), so an intercepted code from a public client that skipped PKCE can be redeemed by anyone. With a method configured, a missing challenge, a missing method or a different method is rejected (L405–L438). |
| SR-3.1, SR-3.3; C3.1; TMT 3 | CWE-294 | [`PkceUtils`][m3-pkce] | No weakness found | Once the authorization request carried a challenge, the token request must carry a verifier (L82–L87), a verifier with no stored challenge is rejected (L89–L94), the verifier's length and characters are checked (L142–L153), and for `S256` it is hashed and compared with the stored challenge (L117–L131). A stolen code without the matching verifier can't be redeemed. Minor: the verifier is written to the debug log (L112), which matters little because the code it unlocks works only once. |
| Misuse case "Intercept Authorization Code"; C3.1; TMT 48, 96 | CWE-294 | [`OAuth2CodeParser`][m3-codeparser], [`AuthorizationCodeGrantType`][m3-codegrant] | No weakness found | The parser removes the code from Keycloak's single-use store as soon as it reads it (parser L97–L104), so a second redemption fails, and when the same client tries again the grant type also detaches the client session the code created (grant type L92–L100). The code must match the user session and client it was issued to (parser L108–L121) and not be expired (parser L123–L127), and the token request has to repeat the same `redirect_uri` (grant type L152–L158) from the same client (grant type L160–L165). One leftover: a code with no stored client ID is still accepted so upgrades between micro versions don't break (parser L116–L118). |
| SR-3.5, SR-3.6; C3.7, C3.10; TMT 5 | CWE-294 | [`TokenManager`][m3-tokenmanager], [`AbstractRefreshTokenProvider`][m3-refresh] | Config-dependent | Reuse detection only runs when the realm's *Revoke Refresh Token* setting is on (refresh provider L300), which is opt-in. When it's on, a refresh token older than the latest one is rejected as stale, including a replay within the same second (token manager L358–L369), and a token used more than *Refresh Token Max Reuse* times is rejected (token manager L380–L384). On either failure Keycloak detaches the client session from the user session (refresh provider L307–L316), and the next refresh with any token from that chain fails because the session no longer has the client ([token manager L245–L248][m3-tmsession]), so whoever holds the newest refresh token loses it too. This corrects our Design observation for TMT 5, the *Not found* we gave SR-3.6 and the gap we showed at C3.10: the revocation exists at this commit, scoped to the one client session. |
| SR-3.8; C3.9; TMT 5 | CWE-294 | [`TokenManager`][m3-binding], [`DPoPUtil`][m3-dpop], [`OIDCAdvancedConfigWrapper`][m3-dpopflag] | Config-dependent | If a public client requires DPoP, a refresh token without a DPoP key binding is refused (token manager L431–L436). A bound refresh token only works with a DPoP proof made with the same key (token manager L478–L487, DPoPUtil L488–L500), and each proof works only once (DPoPUtil L405–L411). A copied refresh token is then useless without the client's private key, which is what SR-3.8 asks for. *Require DPoP bound tokens* is off unless it's set on the client (wrapper L184–L186). |
| TMT 2, 97 | CWE-266 | [`TokenManager`][m3-getaccess], [`RepresentationToModel`][m3-r2m] | Config-dependent | With *Full Scope Allowed* on, every role the user holds goes into the client's tokens (token manager L604–L608). With it off, a token carries only the user's roles that also appear in the client's own roles or its client scopes' role mappings (token manager L610–L633). New clients get *Full Scope Allowed* unless they require consent (RepresentationToModel L571–L585, L617–L622), so by default a service desk client's tokens also carry the user's payroll and finance roles. |
| C3.1; TMT 3 | CWE-347 | [`JWTClientAuthenticator`][m3-jwtclient], [`AbstractJWTClientValidator`][m3-jwtchain], [`AbstractBaseJWTValidator`][m3-jwtreplay], [`OIDCClientSecretConfigWrapper`][m3-secret] | No weakness found | A signed client assertion has to pass the client lookup, the configured signing algorithm, the signature, the audience and the expiry checks, in that order (validator chain L66–L73). The signature is checked against the client's registered public key, and the algorithm must be a supported asymmetric one (authenticator L99–L131). Each assertion's `jti` goes into the single-use store, so a replayed assertion is refused (base validator L95–L120). Clients that authenticate with a shared secret have it compared in constant time with `MessageDigest.isEqual` (wrapper L233–L234, L254). |
| TMT 56 | CWE-312 | [`DefaultKeyProviders`][m3-defkeys], [`ClientAdapter`][m3-clientsecret] | Config-dependent | Each new realm gets generated RSA, HMAC and AES keys (DefaultKeyProviders L36–L47). The [RSA][m3-genrsa] and [HMAC][m3-gensecret] key generators write the private key and the secret into the key provider's configuration (L116–L120 and L57–L60), which is stored in the Keycloak database. Refresh tokens are signed with that HMAC key (HS512 is the default for internal tokens, [`Constants`][m3-hs512] L71). Client secrets are saved as entered and compared with the stored value (ClientAdapter L218–L219, L233–L235), unless the secret is a vault reference, which the [client secret wrapper][m3-secret] resolves first (L228). Anyone who can read the database can sign tokens and authenticate as a confidential client. Keys from the `java-keystore` provider and vault references keep that material out of the database. |
| Design threat *TLS on every flow*; F3.3, F3.4 | CWE-1188 | [`RealmManager`][m3-sslreq], [`TokenEndpoint`][m3-tokenssl], [`SslRequired`][m3-sslenum] | Config-dependent | New realms get *Require SSL* set to external requests (RealmManager L319). The token endpoint refuses plain HTTP only when that setting requires HTTPS for the caller's address (TokenEndpoint L203–L205), and with that setting, callers from loopback, private, link-local or unique-local addresses don't need HTTPS (SslRequired L44–L60). For callers on our corporate LAN the check doesn't apply, so keeping token traffic encrypted there depends on how the server and proxy are set up. |

[m3-redirect]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/RedirectUtils.java#L226-L430
[m3-aec]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/endpoints/AuthorizationEndpointChecker.java#L334-L474
[m3-pkce]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/PkceUtils.java#L81-L153
[m3-codeparser]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/OAuth2CodeParser.java#L68-L132
[m3-codegrant]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/grants/AuthorizationCodeGrantType.java#L88-L165
[m3-tokenmanager]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/TokenManager.java#L352-L385
[m3-refresh]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/refresh/AbstractRefreshTokenProvider.java#L298-L320
[m3-getaccess]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/TokenManager.java#L601-L635
[m3-r2m]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/utils/RepresentationToModel.java#L571-L622
[m3-jwtclient]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/authentication/authenticators/client/JWTClientAuthenticator.java#L87-L131
[m3-jwtchain]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/authentication/authenticators/client/AbstractJWTClientValidator.java#L66-L73
[m3-jwtreplay]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/authentication/authenticators/client/AbstractBaseJWTValidator.java#L95-L120
[m3-secret]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/OIDCClientSecretConfigWrapper.java#L219-L255
[m3-tmsession]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/TokenManager.java#L245-L248
[m3-binding]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/TokenManager.java#L421-L492
[m3-dpop]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/util/DPoPUtil.java#L399-L500
[m3-dpopflag]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/OIDCAdvancedConfigWrapper.java#L184-L187
[m3-defkeys]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/utils/DefaultKeyProviders.java#L36-L94
[m3-genrsa]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/keys/AbstractGeneratedRsaKeyProviderFactory.java#L116-L120
[m3-gensecret]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/keys/AbstractGeneratedSecretKeyProviderFactory.java#L57-L63
[m3-hs512]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/Constants.java#L71
[m3-clientsecret]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/model/jpa/src/main/java/org/keycloak/models/jpa/ClientAdapter.java#L218-L235
[m3-sslreq]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/managers/RealmManager.java#L319
[m3-tokenssl]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/endpoints/TokenEndpoint.java#L203-L205
[m3-sslenum]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/common/src/main/java/org/keycloak/common/enums/SslRequired.java#L34-L60

#### Interaction 4 — Identity brokering

Interaction 4 covers the code that decides whether Keycloak trusts a SAML response from the partner identity provider, what it does with a login the partner starts, how a partner identity gets linked to an existing account, and which roles a partner's assertion can grant. It follows the five misuse cases from our Requirements (smuggle an unsigned assertion, hijack account linking with a victim's email, assert privileged group claims, bypass broker checks through IdP-initiated login, keep a session after offboarding), Claim 4, and the Interaction 4 threats our Design deliverable left Partial, Config-dependent or No.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| SR-4.1; C4.1, C4.2, C4.3; TMT 25 | CWE-347 | [`SAMLIdentityProviderConfig`][m4-cfg], [`SAMLEndpoint`][m4-sig] | Config-dependent | *Validate Signature* and *Want Assertions Signed* are both off unless an administrator turns them on (config L123–L125, L192–L194). With *Validate Signature* off, nothing in a response is verified (endpoint L629–L639), so anyone who can post to the broker endpoint can log in as any partner user. *Want Assertions Signed* on its own only checks that the assertion has a signature element pointing at it, not that the signature is valid (L630), which is the mix-up behind UM4.1. The response and assertion `Issuer` are only compared with the partner's entity ID when *Identity Provider Entity ID* is set (L605–L614, L653–L658). |
| SR-4.1; C4.1; TMT 112 | CWE-347 | [`SAMLEndpoint`][m4-element], [`AssertionUtil`][m4-assertel], [`XMLSignatureUtil`][m4-getsig] | No weakness found | Keycloak checks and then reads the same assertion: the first `Assertion` child of the response (endpoint L600–L602, L651; AssertionUtil L583–L594). A signature only counts for an element when its reference resolves to that element (XMLSignatureUtil L186–L224), so a genuine signature carried over onto a second, forged assertion doesn't validate it. When the whole response is signed instead, every signature in it has to verify and the root element has to be one of the signed ones ([XMLSignatureUtil L468–L496][m4-docsig]). With *Validate Signature* on, the smuggled-assertion misuse case fails. |
| TMT 24, 113 | CWE-294 | [`SAMLEndpoint`][m4-replay] | Config-dependent | For a login Keycloak started, the response's `InResponseTo` has to match the stored request ID and its subject confirmation has to name the client the login is for (L616–L627, L1173–L1199), so a captured response can't be replayed into another login. An IdP-initiated response has no request ID, so that check is skipped (L1174–L1176). Keycloak only accepts one for an enabled client that has an *IDP-Initiated SSO URL name* ([L765–L777][m4-idpinit]), and then the only limits are the assertion's validity window plus clock skew (L699–L723) and, if the partner adds a `OneTimeUse` condition, a single-use record ([L889–L937][m4-otu], new on `main` for 27.0). |
| SR-4.4; C4.4, C4.5; TMT 23, 113 | CWE-841 | [`IdentityBrokerService`][m4-broker] | No weakness found | Every brokered login, IdP-initiated ones included, ends in the same `authenticated` callback (SAML endpoint L747). It refuses a login for a disabled client (L736–L738) and, unless the user is linking an account, any login through a provider set to *Account Linking Only* (L776–L784). CVE-2026-16442 was this check missing for IdP-initiated logins. At this commit it covers them, so our deployment only needs a release with the fix (C4.5). |
| SR-4.2; TMT 37 | CWE-841 | [`DefaultAuthenticationFlows`][m4-flow] | No weakness found | In the default first broker login flow, a partner identity whose email or username matches an existing account is only linked after the user confirms the link and then either opens a link sent to that account's email or signs in as that account, with its 2FA if it has one (flow L562–L698). The email step steps aside when no mail server is set up or the user changed the email or username on the review page, which leaves re-authentication as the only way through ([email verification][m4-emailverify] L81–L92). Nothing in this flow links on a matching email alone. An administrator could swap in a flow with the [auto-link authenticator][m4-autolink], which links without either check (L40–L52). |
| Misuse case "Assert Privileged Role and Group Claims"; TMT 27 | CWE-266 | [`AbstractAttributeToRoleMapper`][m4-rolemap] | Config-dependent | A SAML role mapper grants its role whenever the partner's assertion carries the configured attribute value (L42–L52), so the partner decides who gets it. After the first login the mapper re-checks only in Force sync mode, or in Legacy, which runs the same update for this mapper ([base mapper][m4-basemapper] L100–L101), and then it also removes the role once the partner stops sending the value (L70–L75). A provider with no sync mode set counts as Legacy ([sync mode handling][m4-sync] L18–L20). In Import mode a granted role stays after the partner drops the group. Mapping partner claims only to a contractor role keeps a partner's mistake or compromise away from finance and payroll. |
| SR-4.3; TMT 27 | CWE-266 | [`IdentityProviderResource`][m4-mapperadmin] | Weakness | Creating or changing a mapper only needs `manage-identity-providers` (L359–L374, L426–L452). The one role check, applied when the mapper is saved ([L575–L587][m4-mappercheck]) and again at login ([base mapper][m4-basemapper] L109–L125), blocks admin roles unless the provider allows them. Any other role, payroll and finance included, can be granted to partner users through a mapper, even by an administrator with no permission to assign that role directly. SR-4.3 stays *Partial*. |
| TMT 109, 100 | CWE-778 | [`AbstractAttributeToRoleMapper`][m4-rolemap], [`UserAdapter`][m4-grant] | Weakness | A role mapper grants its role by calling `user.grantRole` (mapper L50, L68). That fires an internal `RoleGrantedEvent` (UserAdapter L538–L542), which only the workflows feature listens to, to start workflows an administrator has defined. It isn't a login or admin event, so even with *Save events* on, the record shows an identity provider login ([broker L760–L764][m4-loginevent]) but not that the login made a contractor a payroll user. This corrects our Design note that the grant emits no event. |
| Design threat *assertion attributes readable in the browser*; F4.3 | CWE-311 | [`SAMLEndpoint`][m4-enc] | Config-dependent | *Want Assertions Encrypted* is off unless set (config L200–L202). When it's on, an unencrypted assertion is refused (endpoint L580–L587) and an encrypted one is decrypted with the realm's encryption keys (L591–L598). Off, the partner's attribute values, such as group names, can be read in the user's browser as the response passes through. |
| SR-4.5; C4.6, C4.7 | CWE-613 | [`SAMLEndpoint`][m4-logout], [`DefaultClientScopes`][m4-offline] | Config-dependent | Keycloak only ends a contractor's session early if the partner sends a SAML logout request (endpoint L348–L352). Otherwise it runs until the realm's idle or maximum timeout, 30 minutes and 10 hours by default ([`Constants`][m4-constants] L73–L74). Every new realm also gives each user the `offline_access` role ([`KeycloakModelUtils`][m4-offlinerole] L736) and adds an `offline_access` scope that clients can request (DefaultClientScopes L48–L57), and an offline token keeps working as long as it's used within the offline idle timeout, 30 days by default (`Constants` L66). A contractor the partner has offboarded keeps any offline token until an administrator signs them out or revokes it, so C4.7 needs `offline_access` taken off partner-facing clients. |

[m4-cfg]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLIdentityProviderConfig.java#L91-L202
[m4-sig]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L605-L658
[m4-element]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L589-L651
[m4-assertel]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/saml-core/src/main/java/org/keycloak/saml/processing/core/saml/v2/util/AssertionUtil.java#L575-L597
[m4-getsig]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/saml-core/src/main/java/org/keycloak/saml/processing/core/util/XMLSignatureUtil.java#L186-L224
[m4-docsig]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/saml-core/src/main/java/org/keycloak/saml/processing/core/util/XMLSignatureUtil.java#L468-L496
[m4-replay]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L1173-L1199
[m4-idpinit]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L765-L777
[m4-otu]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L876-L937
[m4-broker]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/resources/IdentityBrokerService.java#L722-L784
[m4-flow]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/utils/DefaultAuthenticationFlows.java#L533-L698
[m4-emailverify]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/authentication/authenticators/broker/IdpEmailVerificationAuthenticator.java#L81-L116
[m4-autolink]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/authentication/authenticators/broker/IdpAutoLinkAuthenticator.java#L40-L52
[m4-rolemap]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/mappers/AbstractAttributeToRoleMapper.java#L42-L112
[m4-basemapper]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/broker/provider/AbstractIdentityProviderMapper.java#L100-L125
[m4-sync]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/broker/provider/IdentityProviderMapperSyncModeDelegate.java#L16-L32
[m4-mapperadmin]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/resources/admin/IdentityProviderResource.java#L359-L452
[m4-mappercheck]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/resources/admin/IdentityProviderResource.java#L575-L587
[m4-grant]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/model/jpa/src/main/java/org/keycloak/models/jpa/UserAdapter.java#L538-L542
[m4-loginevent]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/resources/IdentityBrokerService.java#L760-L764
[m4-enc]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L580-L598
[m4-logout]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L348-L352
[m4-constants]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/Constants.java#L66-L74
[m4-offlinerole]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/utils/KeycloakModelUtils.java#L730-L737
[m4-offline]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/utils/DefaultClientScopes.java#L48-L57

#### Interaction 5 — Directory federation

TBD: one or two sentences on which critical security functions this covers and why.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD |

### Automated code scanning findings

| Tool | Version | Rules | Raw output |
|---|---|---|---|
| Semgrep Community Edition | 1.186.0 | `p/java`, `p/findsecbugs` | [SARIF](code-analysis/semgrep.sarif) · [text](code-analysis/semgrep.txt) |
| CodeQL | 2.27.4 | `codeql/java-queries`, `java-security-extended` suite | [SARIF](code-analysis/codeql.sarif) · [CSV](code-analysis/codeql.csv) |

Semgrep scanned the main (non-test) Java source of the nine Keycloak modules in our scope at `d052236`. CodeQL built its database from the whole repository at the same commit so it could follow data between modules, and we triaged only the findings in our modules. Semgrep reported 1,148 findings and CodeQL 586, 434 of them in our modules. 183 Semgrep and 148 CodeQL findings fall in the code behind our five interactions, and the tables below list the ones that matter. TBD: triage counts once every interaction is done.

<!--
List the findings that matter in our in-scope code, under the interaction whose code they're in, not every hit. If an interaction has none worth listing, replace its table with one line saying so.
Triage: True positive / False positive / Not exploitable here (real, but our environment or another control blocks it).
-->

#### Interaction 1 — End-user authentication

| Tool | Rule | CWE | Location | Triage | Justification |
|---|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD | TBD |

#### Interaction 2 — Realm admin manages credentials and password policy

| Tool | Rule | CWE | Location | Triage | Justification |
|---|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD | TBD |

#### Interaction 3 — Client application token acquisition

| Tool | Rule | CWE | Location | Triage | Justification |
|---|---|---|---|---|---|
| CodeQL | `java/log-injection` | CWE-117 | [`RedirectUtils` L197][a3-redirect-log] | True positive | When a redirect URI is rejected, its decoded path goes into a WARN line unescaped, so an encoded newline in the path could forge a log entry that reaches our SIEM. Low risk: nothing else is affected. |
| CodeQL | `java/log-injection` | CWE-117 | 15 more across redirect URI, key manager and scope code ([CSV](code-analysis/codeql.csv)) | Not exploitable here | 13 are debug or trace lines, which are off in production. The other two, an INFO and an ERROR line in the key manager, log an algorithm name that can come from a key set Keycloak fetched from a registered client or identity provider, so only a party we already trust with keys could forge an entry. |
| CodeQL | `java/sensitive-log` | CWE-532 | 43 across introspection, PKCE, ACR and redirect URI code ([CSV](code-analysis/codeql.csv)) | False positive | All 43 log identifiers or configuration names, not secrets: client, user, session, token and key IDs, usernames, ACR values, algorithm and claim names, redirect URIs and PKCE challenges. The closest thing to a secret in a log line, the raw PKCE verifier at `PkceUtils` L112 (debug level), wasn't flagged by CodeQL and is covered in our manual review. |
| Semgrep | `HARD_CODE_KEY-2`, `-3`, `-4` | CWE-321 | 58 across key providers, client authenticators, request parsers and `OIDCLoginProtocol` ([text](code-analysis/semgrep.txt)) | False positive | No key material is in the source. 52 hits are string constants that name a setting, protocol, provider or cache key, or hold help text, such as `SECRET_KEY = "secret"`. The other 6 (`-4`) are `equals` checks on algorithm names, client authentication methods or an error message. |

[a3-redirect-log]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/RedirectUtils.java#L197-L198

#### Interaction 4 — Identity brokering

| Tool | Rule | CWE | Location | Triage | Justification |
|---|---|---|---|---|---|
| CodeQL | `java/user-controlled-bypass` | CWE-290 | 6 in the [identity broker service, L262–L360][a4-linking] | False positive | These are the checks on client-initiated account linking, not a way around them. The user has to be signed in already (L262), the request has to carry a SHA-256 hash of a nonce, the user's session ID, the client and the provider, compared in constant time (L287–L289), and the user needs `manage-account` or `manage-account-links` (L302–L318). Only the client application holding the user's tokens knows the session ID, so another site can't forge a link request. |
| Semgrep | `XPATH_INJECTION-1` | CWE-643 | [XPath attribute mapper L216][a4-xpath] | False positive | The XPath expression comes from the mapper's configuration (L147), which only an administrator can set. The partner identity provider controls the document the expression runs against, not the expression. |
| Semgrep | `HARD_CODE_KEY-4` | CWE-208 | [OID4VP endpoint L292][a4-oid4vp] | Not exploitable here | Not a hard-coded key, but a real minor issue: the `response_code` is checked with `String.equals`, which isn't constant-time. OID4VP is an experimental feature that's off by default, and each code works only once (L295–L296), so it doesn't affect us. Switching to a constant-time comparison would be a small upstream hardening fix. |
| CodeQL | `java/potentially-weak-cryptographic-algorithm` | CWE-327 | 3 in SAML signature and broker client authentication code ([CSV](code-analysis/codeql.csv)) | Not exploitable here | `SHA1withRSA` and `SHA1withDSA` stay available so older identity providers can still sign (the SAML signature algorithm list L29, L34, and `SignatureUtil` L211–L212). Keycloak's own SAML requests default to RSA-SHA256 (SAML identity provider L524–L531), and the third hit is an HMAC key whose algorithm defaults to HS256 (OAuth2 identity provider L972–L975). We'd ask the partner to sign with SHA-256. |
| CodeQL | `java/log-injection` | CWE-117 | 15 across the OAuth2, OIDC and broker code ([CSV](code-analysis/codeql.csv)) | Not exploitable here | 8 are debug or trace lines, which are off in production. Of the 7 WARN or ERROR lines, 5 log text from a provider's own responses (userinfo and token endpoint errors, introspection results, its `sid` and `session_state`), so only an identity provider we registered could forge an entry. The other 2 log a provider alias from the request path, but only a caller holding the broker `read-token` role gets that far, and no user has it by default (`RealmManager` L590–L609). |
| CodeQL | `java/sensitive-log` | CWE-532 | 7 across the OIDC provider, mapper and SAML library code ([CSV](code-analysis/codeql.csv)) | False positive | The logged values are identifiers and configuration names, not secrets: a provider's `sid` and `session_state`, usernames, client IDs and token audiences, a claim's name and value, and a token registry class name. |
| Semgrep | `IMPROPER_UNICODE-1` | CWE-176 | 9 in the SAML XML writers ([text](code-analysis/semgrep.txt)) | False positive | Each hit converts a URI to ASCII text while writing a destination, consumer URL, format or attribute value into an outgoing SAML message. No security decision compares the result, which is what the rule looks for. |
| Semgrep | `HARD_CODE_KEY-1` to `-4` | CWE-321 | 75 more across SAML, OAuth2, OID4VP and mapper code ([text](code-analysis/semgrep.txt)) | False positive | No key material is in the source. 63 hits are string constants such as error codes and configuration attribute names (`-2`, `-3`), 10 are `equals` checks on algorithm, attribute or claim names (`-4`), and the two `-1` hits build keys from a random secret or the provider's configured client secret (SAML binding builder L319–L320, OAuth2 identity provider L974–L975). |

[a4-linking]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/resources/IdentityBrokerService.java#L236-L360
[a4-xpath]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/mappers/XPathAttributeMapper.java#L147-L216
[a4-oid4vp]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/oid4vp/OID4VPIdentityProviderEndpoint.java#L284-L296

#### Interaction 5 — Directory federation

| Tool | Rule | CWE | Location | Triage | Justification |
|---|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD | TBD |

---

## Part 2 — Key Findings and Contributions

### Summary of key findings

<!--
The most significant findings from Part 1, manual and automated, highest risk first. Risk is for our operational environment (see the team note).
-->

| # | Finding | CWE | Found by | Interaction | Risk in our environment |
|---|---|---|---|---|---|
| 1 | TBD | TBD | Manual / Semgrep / CodeQL | TBD | High / Medium / Low: TBD reason |

TBD: a short paragraph tying the findings together.

### Upstream contributions

Contributions to Keycloak across this project. For how each bug was found, traced in the source, and fixed, see [Upstream Contributions](UpstreamContributions.md).

| Issue | Contribution | Status |
|---|---|---|
| [#20008](https://github.com/keycloak/keycloak/issues/20008) Creating a UMA policy via the Protection API wasn't recorded in the admin audit log | Protection API now records the CREATE admin event, matching update/delete, with an integration test ([PR #53316](https://github.com/keycloak/keycloak/pull/53316)) | Merged |
| [#53249](https://github.com/keycloak/keycloak/issues/53249) Brute-force failure count isn't reset after a successful IdP login | Opt-in per identity provider setting that lets successful provider logins reset the count without letting cookie SSO reset it (avoids reopening [#49960](https://github.com/keycloak/keycloak/issues/49960)), with integration tests for SAML, OIDC, and OAuth2 ([PR #53724](https://github.com/keycloak/keycloak/pull/53724)) | PR open, waiting on maintainer review |
| [#53325](https://github.com/keycloak/keycloak/issues/53325) AuthZen evaluation let a caller impersonate another client towards client policies | Request-supplied `kc.*` attributes are dropped and `kc.client.id` is set from the caller's token, with integration tests for both injection paths, the batch endpoint, and the reserved-attribute filter on its own ([PR #55186](https://github.com/keycloak/keycloak/pull/55186)) | PR open, waiting on maintainer review |
| TBD | TBD: contribution planned from this code review | Planned |

<!-- Optional. The assignment accepts contributions based on any earlier assignment, so the three PRs above already cover it. Delete the Planned row and the line below if nothing comes out of this review. -->

TBD: what we plan to contribute from this review (documentation, design change, code change, issue or discussion) and why.

### Project repository and task assignments

- [Team repository](https://github.com/JBoogieman/SoftwareAssuranceTeam3)
- [Code Analysis milestone](https://github.com/JBoogieman/SoftwareAssuranceTeam3/milestone/5): every task for this deliverable, who it was assigned to, and the pull requests that closed it
- [Project Board](https://github.com/users/JBoogieman/projects/1)

### Team reflection

#### What did we learn from this assignment?

TBD

#### What did we find most useful?

TBD

<!-- Compiled from individual reflections in the issue: What did you learn? What did you find most useful? -->
