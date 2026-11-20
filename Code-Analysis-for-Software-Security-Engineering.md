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
| Many of our Design gaps were rated *Config-dependent*. A scanner can't tell an insecure default from a bug, and reading only the docs can't tell us what the code actually enforces. | Manual review follows each of those threats to the line that sets the default or performs the check, and the *Config-dependent* result records when the code is sound but the default isn't. |
| Generic scanner rules don't know Keycloak's own sanitizers, so we expected them to flag every LDAP search as injection. | We read Keycloak's LDAP escaping path once (`Condition` → `EscapeStrategy`) during manual review, then triaged every scanner LDAP-injection hit against it. |
| Some behavior only shows up against a real directory, such as group matching across OUs or StartTLS negotiation, and we have no Active Directory lab. | When the code alone can't settle a question, the row says *Inconclusive*, links the upstream test that exists, and names the test that's missing, instead of claiming a weakness we haven't reproduced. |
| A real weakness found during the review would be public as soon as it was committed to this repository. | We follow Keycloak's SECURITY.md: anything that looks exploitable is reported privately with a proof of concept, and its details stay out of this file until it's resolved. |
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
| TBD | TBD | TBD | TBD | TBD |

#### Interaction 4 — Identity brokering

TBD: one or two sentences on which critical security functions this covers and why.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD |

#### Interaction 5 — Directory federation

This covers the four functions every employee identity and role passes through on the Corporate directory boundary: how Keycloak protects the LDAP connection (F5.2–F5.5), how it builds the search filter from a typed username (F5.4), whether lockout runs before the password reaches Active Directory (F5.1, F5.2), and how AD groups become Keycloak roles and groups and whether that leaves a record (F5.5, F5.6, F5.8). These are the Claim 5 branches our Assurance Case left without code-level evidence.

| Traces to | CWE | Code reviewed | Result | Finding |
|---|---|---|---|---|
| TMT 230, 245; SR-5.1; C5.1 | CWE-1188 | [`LDAPStorageProviderFactory.validateConfiguration`][cr5-url-check]; [`LDAPContextManager`][cr5-admin-bind]; [`LDAPOperationManager.authenticate`][cr5-user-ctx] | Weakness | The only URL check is that the scheme is `ldap` or `ldaps`. With `ldap://` and *Enable StartTLS* off, the service account's bind DN and password go straight into the initial JNDI environment, so the first connection sends them in a simple bind in clear text. User logins open the same kind of plaintext connection before binding with the employee's password. Nothing warns the administrator, and no realm setting can forbid plaintext providers. This confirms our Design observation at the code level. |
| TMT 221, 226, 241; SR-5.1; C5.2 | CWE-1188 | [`LDAPContextManager.startTLS`][cr5-starttls]; [bind deferred until after StartTLS][cr5-bind-after-tls]; [`LDAPOperationManager.authenticate`][cr5-user-starttls]; [`LDAPUtil.shouldUseTruststoreSpi`][cr5-truststore] | No weakness found | When TLS is on, it fails closed. A failed StartTLS negotiation throws `AuthenticationException`, and the admin connection binds only after negotiation succeeds. The user-login path does the same before sending the password. With `ldaps://` or StartTLS, Keycloak's truststore is used unless *Use Truststore SPI* is set to `never`, the only case that falls back to the JVM's default trust. |
| SR-5.1, F5.4 | CWE-90 | [`loadLDAPUserByUsername`][cr5-username-filter]; [`EqualCondition`][cr5-equal]; [`NamedParameterCondition.escapeValue`][cr5-escape-call]; [`EscapeStrategy.DEFAULT`][cr5-escape]; [`LDAPOperationManager.search`][cr5-search]; [`validateCustomLdapFilter`][cr5-custom-filter] | No weakness found | The typed username becomes an `equal` condition, and every condition type (`Equal`, `Substring`, `In`, `LessThan`, `GreaterThan`, `Between`) passes its value through `escapeValue()`. That hex-escapes `\ * ( )`, NUL and non-ASCII bytes as RFC 4515 requires. The search methods send only `condition.toFilter()` output, so no raw string reaches `context.search`. The one unescaped input is the *Custom User LDAP Filter*, which only an administrator can set and which is only checked for surrounding parentheses. |
| TMT 220, 233; C5.1 | CWE-307 | [`AbstractUsernameFormAuthenticator.validatePassword`][cr5-bf-order]; [`LDAPStorageProvider.isValid`][cr5-isvalid] → [`validPassword`][cr5-validpassword]; [`LDAPOperationManager.authenticate`][cr5-empty-pw]; [`RealmManager`][cr5-bf-default] | Config-dependent | In the browser login form, the brute-force check (line 225) runs before `isValid` (line 227), and for a federated user `isValid` is the LDAP bind. So a locked-out employee's guesses never reach AD. Empty passwords are rejected before any bind, which blocks AD's "unauthenticated bind" (a DN with an empty password counts as an anonymous success), and user binds are never pooled. But brute-force detection is off for new realms (`setBruteForceProtected(false)`), so by default nothing stops password spraying before AD's own lockout locks employees out. |
| TMT 220, 233 | CWE-307 | [`ValidateUsername` (direct grant)][cr5-dg-bf] | Inconclusive | The password grant checks lockout before validating the password for enabled users, the same order as the browser form. We found one edge case in this path that may let password checks reach AD without counting toward lockout. Under Keycloak's SECURITY.md we're building a proof of concept and will report it privately, so its details are not in this public file. |
| TMT 248, 263; SR-5.3; R5.3; C5.4 | CWE-266 | [`RoleLDAPStorageMapper.onImportUserFromLDAP`][cr5-role-create] | Weakness | The role name is the AD group's name attribute. If no Keycloak role has that name, the mapper creates one (`roleContainer.addRole`), then grants it. There's no allowlist, and no check for admin or composite roles like the one 26.8.0 added for identity provider mappers. If the mapper targets realm roles, any group under the Roles DN whose name matches an existing privileged role, such as a finance or payroll role, grants it to every member. |
| UM5.2; C5.4, C5.6 | CWE-266 | [`UserRolesRetrieveStrategy.GetRolesFromUserMemberOfAttribute`][cr5-memberof]; [`GroupLDAPStorageMapper.createGroupQuery`][cr5-group-scope]; [upstream test05][cr5-test05] | Inconclusive | The `memberOf` strategy first keeps only DNs under the configured Groups DN (line 106), which is the check SR-5.3 asks for. It then reduces each DN to its first RDN value (line 107, for example `cn=Finance`) and searches for that name under the Groups DN, using the provider's global search scope. With *Search Scope = Subtree*, membership in `cn=Finance,ou=Sandbox,ou=Groups,…` would appear to also match `cn=Finance,ou=Apps,ou=Groups,…`. Upstream test05 covers only a group outside the Groups DN, so this is the duplicate-CN case UM5.2 says the tests omit. It needs an embedded-LDAP test before we call it a weakness, and the attacker must be able to create groups inside the Groups DN, which CT5.1 mostly treats as an authorized AD change. |
| TMT 244; SR-5.4; R5.8; C5.10 | CWE-778 | [`RoleLDAPStorageMapper`][cr5-role-grant]; [`GroupLDAPStorageMapper`][cr5-group-join]; [`UserStorageProviderResource.syncUsers`][cr5-sync-event]; [`UserStorageSyncTask.run`][cr5-sync-task] | Weakness | Mapper role grants and group joins write only a `DEBUG` log line. The LDAP module uses `AdminEventBuilder` only in its admin REST resources, never in mapper code. A sync started from the Admin Console records an admin event with added, updated and removed counts (lines 178–181), but only if *Save admin events* and *Include representation* are on. Scheduled syncs go through `UserStorageSyncTask` and emit no event at all. Nothing in Keycloak can show which AD group DN produced a user's role, so C5.10 can't be supported. |

[cr5-url-check]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/LDAPStorageProviderFactory.java#L301-L317
[cr5-admin-bind]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPContextManager.java#L93-L103
[cr5-user-ctx]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPOperationManager.java#L557-L568
[cr5-starttls]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPContextManager.java#L153-L169
[cr5-bind-after-tls]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPContextManager.java#L105-L122
[cr5-user-starttls]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPOperationManager.java#L570-L584
[cr5-truststore]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPUtil.java#L376-L390
[cr5-username-filter]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/LDAPStorageProvider.java#L1093-L1101
[cr5-equal]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/query/internal/EqualCondition.java#L44
[cr5-escape-call]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/query/internal/NamedParameterCondition.java#L66-L70
[cr5-escape]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/query/EscapeStrategy.java#L30-L63
[cr5-search]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPOperationManager.java#L287-L296
[cr5-custom-filter]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/LDAPUtils.java#L353-L365
[cr5-bf-order]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/authentication/authenticators/browser/AbstractUsernameFormAuthenticator.java#L218-L233
[cr5-isvalid]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/LDAPStorageProvider.java#L952-L959
[cr5-validpassword]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/LDAPStorageProvider.java#L833-L842
[cr5-empty-pw]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPOperationManager.java#L539-L561
[cr5-bf-default]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/managers/RealmManager.java#L307-L309
[cr5-dg-bf]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/authentication/authenticators/directgrant/ValidateUsername.java#L110-L118
[cr5-role-create]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/mappers/membership/role/RoleLDAPStorageMapper.java#L85-L114
[cr5-memberof]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/mappers/membership/UserRolesRetrieveStrategy.java#L95-L118
[cr5-group-scope]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/mappers/membership/group/GroupLDAPStorageMapper.java#L98-L105
[cr5-test05]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/testsuite/integration-arquillian/tests/base/src/test/java/org/keycloak/testsuite/federation/ldap/LDAPGroupMapperTest.java#L624-L640
[cr5-role-grant]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/mappers/membership/role/RoleLDAPStorageMapper.java#L110-L111
[cr5-group-join]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/mappers/membership/group/GroupLDAPStorageMapper.java#L684-L685
[cr5-sync-event]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/model/storage-services/src/main/java/org/keycloak/services/resources/admin/UserStorageProviderResource.java#L178-L181
[cr5-sync-task]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/model/storage-private/src/main/java/org/keycloak/storage/UserStorageSyncTask.java#L39-L60

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

In the LDAP provider (`federation/ldap`), `LDAPConstants` and the user-storage sync code (`model/storage-private`, `model/storage-services`), Semgrep reported 27 findings and CodeQL 12. All 27 Semgrep findings were false positives. Of the 12 CodeQL findings, one was a false positive, and the other 11 point at real code that isn't exploitable in our environment. Neither tool flagged the plaintext `ldap://` default, the role mapper creating roles from AD group names, or the missing mapper audit events, so every Interaction 5 weakness above came from manual review.

| Tool | Rule | CWE | Location | Triage | Justification |
|---|---|---|---|---|---|
| Semgrep | `gitlab.find_sec_bugs.LDAP_INJECTION-1` (3 hits) | CWE-90 | [`LDAPOperationManager.java` L296][sa5-ldapi-1], [L351][sa5-ldapi-2], [L460][sa5-ldapi-3] | False positive | The filter passed to `context.search` is always `condition.toFilter()`, and every condition escapes its values through `EscapeStrategy.DEFAULT` (see the CWE-90 row in manual review). Semgrep checks one file at a time, so it can't see the escaping in the condition classes. |
| Semgrep | `gitlab.find_sec_bugs.HARD_CODE_KEY-2`, `-3`, `-4` (24 hits) | CWE-321 | `LDAPConstants.java` (18), [`MSADUserAccountControlStorageMapper.java` L54][sa5-key-msad] (2), [`PasswordModifyRequest.java` L37][sa5-key-oid], `UserStorageEventListener.java` L25, [`UserStorageProviderResource.java` L258, L265][sa5-key-direction] | False positive | The rule matches constant names containing words like "password" or "key". These strings are LDAP attribute names, configuration keys, the RFC 3062 password-modify OID and the sync direction names `fedToKeycloak`/`keycloakToFed`. None is a secret. |
| CodeQL | `java/sensitive-log` | CWE-532 | [`LDAPContextManager.java` L224][sa5-masked] | False positive | The bind password is replaced with asterisks on lines 221–223 before the connection environment is logged, and only at `DEBUG`. |
| CodeQL | `java/sensitive-log`, `java/log-injection` (8 hits) | CWE-532, CWE-117 | [`UserAttributeLDAPStorageMapper.java` L325][sa5-attr-trace]; [`LDAPObject.java` L188][sa5-multivalue]; [`LDAPIdentityStore.java` L543][sa5-store-543], [L608][sa5-store-608] | Not exploitable here | These lines log directory attribute values or DNs, not passwords: L325 only at `TRACE`, the rest at `WARN`. The values come from Active Directory, which CT5.1 treats as authoritative, or from administrators. L188 does show a real side effect: it logs *every* value of a multi-valued attribute at `WARN`, so the unbounded attribute values in E5.12 would also flood the logs. |
| CodeQL | `java/tainted-arithmetic` (3 hits) | CWE-190 | [`UserStorageManager.java` L348, L399, L423][sa5-paging] | Not exploitable here | `firstResult + 1` and `offset + maxResults` come from the admin user-search paging parameters. Only an administrator with `view-users` can set them, and an overflow at most returns the wrong page. |

[sa5-ldapi-1]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPOperationManager.java#L287-L296
[sa5-ldapi-2]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPOperationManager.java#L351
[sa5-ldapi-3]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPOperationManager.java#L450-L460
[sa5-key-msad]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/mappers/msad/MSADUserAccountControlStorageMapper.java#L54
[sa5-key-oid]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/extended/PasswordModifyRequest.java#L37
[sa5-key-direction]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/model/storage-services/src/main/java/org/keycloak/services/resources/admin/UserStorageProviderResource.java#L258-L265
[sa5-masked]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPContextManager.java#L218-L226
[sa5-attr-trace]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/mappers/UserAttributeLDAPStorageMapper.java#L324-L325
[sa5-multivalue]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/model/LDAPObject.java#L187-L188
[sa5-store-543]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPIdentityStore.java#L541-L544
[sa5-store-608]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/federation/ldap/src/main/java/org/keycloak/storage/ldap/idm/store/ldap/LDAPIdentityStore.java#L606-L608
[sa5-paging]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/model/storage-private/src/main/java/org/keycloak/storage/UserStorageManager.java#L345-L424

---

## Part 2 — Key Findings and Contributions

### Summary of key findings

<!--
The most significant findings from Part 1, manual and automated, highest risk first. Risk is for our operational environment (see the team note).
-->

| # | Finding | CWE | Found by | Interaction | Risk in our environment |
|---|---|---|---|---|---|
| 1 | TBD | TBD | Manual / Semgrep / CodeQL | TBD | High / Medium / Low: TBD reason |
| TBD | Keycloak accepts a plaintext `ldap://` federation provider without warning, sending the service-account and employee passwords in clear text | CWE-1188 | Manual | 5 | High: one configuration choice exposes every federated employee's AD password, which also unlocks Windows, email and VPN, to anyone on the network path to a domain controller |
| TBD | The LDAP role mapper grants, and creates if missing, any Keycloak role whose name matches an AD group under the Roles DN | CWE-266 | Manual | 5 | Medium: anyone who can create or rename a group under the Roles DN, such as a delegated OU admin or a helpdesk tool, can grant finance or payroll roles in Keycloak |
| TBD | Role grants and group joins made by LDAP mappers leave no audit event, and scheduled syncs leave none either | CWE-778 | Manual | 5 | Medium: after an unauthorized role shows up, nothing in Keycloak shows which AD group or sync caused it, so C5.10 can't be supported |
| TBD | Brute-force detection, which is what stops guesses before the LDAP bind, is off for new realms | CWE-1188, CWE-307 | Manual | 5 | Medium: by default, password spraying through Keycloak reaches AD and can lock out employees' AD accounts |

TBD: a short paragraph tying the findings together.

### Upstream contributions

Contributions to Keycloak across this project. For how each bug was found, traced in the source, and fixed, see [Upstream Contributions](UpstreamContributions.md).

| Issue | Contribution | Status |
|---|---|---|
| [#20008](https://github.com/keycloak/keycloak/issues/20008) Creating a UMA policy via the Protection API wasn't recorded in the admin audit log | Protection API now records the CREATE admin event, matching update/delete, with an integration test ([PR #53316](https://github.com/keycloak/keycloak/pull/53316)) | Merged |
| [#53249](https://github.com/keycloak/keycloak/issues/53249) Brute-force failure count isn't reset after a successful IdP login | Opt-in per identity provider setting that lets successful provider logins reset the count without letting cookie SSO reset it (avoids reopening [#49960](https://github.com/keycloak/keycloak/issues/49960)), with integration tests for SAML, OIDC, and OAuth2 ([PR #53724](https://github.com/keycloak/keycloak/pull/53724)) | PR open, waiting on maintainer review |
| [#53325](https://github.com/keycloak/keycloak/issues/53325) AuthZen evaluation let a caller impersonate another client towards client policies | Request-supplied `kc.*` attributes are dropped and `kc.client.id` is set from the caller's token, with integration tests for both injection paths, the batch endpoint, and the reserved-attribute filter on its own ([PR #55186](https://github.com/keycloak/keycloak/pull/55186)) | PR open, waiting on maintainer review |
| TBD | TBD: contribution planned from this code review | Planned |
| To be opened | Documentation PR adding a "Security considerations" subsection to the Server Admin Guide's LDAP chapter: require LDAPS or StartTLS in production, link to truststore setup, scope the Roles DN so group names can't collide with privileged roles, and note that brute-force detection is what protects AD's lockout. This builds on the documentation gap from our Design review and the CWE-1188 and CWE-266 rows above | Planned |
| To be opened | Enhancement issue: emit an event when an LDAP role or group mapper grants or removes a role or group (mapper, user, role, source group DN), so mapper-driven access can be audited (CWE-778, SR-5.4) | Planned |
| Private (SECURITY.md) | Report the Interaction 5 lockout edge case once we have a proof of concept. Details stay out of this file until Keycloak resolves it | Planned |

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
