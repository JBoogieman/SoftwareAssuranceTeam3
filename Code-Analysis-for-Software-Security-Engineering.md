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

TBD: one or two sentences on which critical security functions this covers and why.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| SR-3.4; TMT 89, 172 | CWE-601 | [`RedirectUtils.matchesRedirects`](https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/RedirectUtils.java#L248-L293), [`redirectUriStartsWith`](https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/RedirectUtils.java#L338-L382), [`checkValidRedirectWildcard`](https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/RedirectUtils.java#L408-L430) | Config-dependent | With no wildcard registered, the redirect URI has to match a registered one exactly (L287–L288). A registered `*` accepts any http or https redirect and skips the wildcard safety checks (L251–L253). A trailing `*` turns a registration into a path prefix match that ignores the query and fragment (L255–L273). The scheme, host and port still have to match (L338–L370), a `*` right after the host is treated as `/*` (L420–L423), wildcards are ignored when the path contains `..` (L226–L231), and redirect URIs carrying response parameters such as `code` or `state` are refused (L53–L62, L174–L187). Nothing blocks wildcard registrations unless the `secure-redirect-uris-enforcer` client policy is on. |
| SR-3.2; TMT 3 | CWE-1188 | [`AuthorizationEndpointChecker.checkPKCEParams`](https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/endpoints/AuthorizationEndpointChecker.java#L334-L351), [`checkParamsForPkceNotEnforcedClient`](https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/endpoints/AuthorizationEndpointChecker.java#L440-L474) | Config-dependent | PKCE is only enforced when the client has a PKCE method configured (L343–L350). Otherwise an authorization request with no `code_challenge` goes through with only a debug log line (L449–L453), and `plain` is accepted as readily as `S256` (L455–L465), so an intercepted code from a public client that skipped PKCE can be redeemed by anyone. With a method configured, a missing challenge, a missing method or a different method is rejected (L405–L438). |
| SR-3.1, SR-3.3; TMT 3 | CWE-294 | [`PkceUtils.checkParamsForPkceNotEnforcedClient`](https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/PkceUtils.java#L81-L99), [`verifyCodeVerifier`](https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/PkceUtils.java#L101-L140) | No weakness found | Once the authorization request carried a challenge, the token request must carry a verifier (L82–L87), a verifier with no stored challenge is rejected (L89–L94), the verifier's length and characters are checked (L142–L153), and for `S256` it is hashed and compared with the stored challenge (L117–L131). A stolen code without the matching verifier can't be redeemed. Minor: the verifier is written to the debug log (L112), which matters little because the code it unlocks works only once. |
| Misuse case "Intercept Authorization Code"; TMT 48, 96 | CWE-294 | [`OAuth2CodeParser.parseCode`](https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/OAuth2CodeParser.java#L68-L132), [`AuthorizationCodeGrantType`](https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/grants/AuthorizationCodeGrantType.java#L88-L165) | No weakness found | The code is removed from Keycloak's single-use store as soon as it's parsed (L97–L104), so a second redemption fails, and when the same client tries again Keycloak also detaches the client session the code created (L92–L100). The code must match the user session and client it was issued to (L108–L121) and not be expired (L123–L127), and the token request has to repeat the same `redirect_uri` (L152–L158) from the same client (L160–L165). One leftover: a code with no stored client ID is still accepted so upgrades between micro versions don't break (L116–L118). |
#### Interaction 4 — Identity brokering

TBD: one or two sentences on which critical security functions this covers and why.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD |

#### Interaction 5 — Directory federation

TBD: one or two sentences on which critical security functions this covers and why.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD |

### Automated code scanning findings

| Tool | Version | Rules | Raw output |
|---|---|---|---|
| Semgrep Community Edition | TBD | `p/java`, `p/findsecbugs` | [SARIF](code-analysis/semgrep.sarif) · [text](code-analysis/semgrep.txt) |
| CodeQL | TBD | `codeql/java-queries`, `java-security-extended` suite | [SARIF](code-analysis/codeql.sarif) · [CSV](code-analysis/codeql.csv) |

Semgrep scanned the main (non-test) Java source of the nine Keycloak modules in our scope at `d052236`. CodeQL built its database from the whole repository at the same commit so it could follow data between modules, and we triaged only the findings in our modules. TBD: total findings per tool, how many fell in our scope, and how they were triaged.

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
| TBD | TBD | TBD | TBD | TBD | TBD |

#### Interaction 4 — Identity brokering

| Tool | Rule | CWE | Location | Triage | Justification |
|---|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD | TBD |

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
