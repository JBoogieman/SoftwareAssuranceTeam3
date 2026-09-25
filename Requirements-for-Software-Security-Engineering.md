# Requirements for Software Security Engineering — Team 3 (Keycloak)

**Due:** Tue Sep 29, 2026, 11:59 PM · **Points:** 100 (counts for the whole group)
**Submission:** Link to this markdown file in our GitHub repo, submitted on Canvas.
**Tool:** [draw.io / app.diagrams.net](https://app.diagrams.net/) — instructor provided sample shape files on the Canvas assignment page (Use Case Sample.drawio, Use-Misuse Case Sample.drawio).

**Rubric:** Part 1 misuse case notation & quality (50) · Part 1 reflection (10) · Part 2 doc review (20) · Planning & reflection / project board (20)

---

## How this works

Part 1 requires **five essential interactions** between Keycloak and its environment, ideally spread across *different* external interactors (humans or systems). Each team member claims **one** interaction below and owns the full pipeline for it: use case diagram → misuse case analysis → security requirements → alignment check against Keycloak's actual features.

Part 2 does **not** split into five cases — it's a single team review of Keycloak's security-related documentation (see Part 2 section below).

### Claim board

Put your name next to one interaction. These are suggested candidates (biased toward our authorization/credential scope) — confirm or swap at the Friday meeting. Rule of thumb: five *different* interactor types, not five things one actor does.

| # | External interactor | Candidate interaction / feature | Claimed by | Status |
|---|---|---|---|---|
| 1 | End user (human) | Login / authentication via browser (password + OTP) |[@Sewhenu-Ayeni](https://github.com/Sewhenu-Ayeni) | Completed |
| 2 | Realm administrator (human) | Manage user credentials & password policies via Admin Console | [@AyRidd03](https://github.com/AyRidd03) | Not started |
| 3 | Client application (system) | Obtain tokens via OIDC authorization code flow | [@JBoogieman](https://github.com/JBoogieman) | Completed |
| 4 | External identity provider (system) | Identity brokering / federated login (SAML or OIDC IdP) | [@SeanAnderson0](https://github.com/SeanAnderson0) | Not started |
| 5 | Directory service (system) | User federation with LDAP / Active Directory | [@isaiahjames11](https://github.com/isaiahjames11) | Not started |

Other candidates if we swap: Admin REST API automation, service accounts (client credentials grant), user self-service account console, token introspection by a resource server.

---

## Part 1 — Per-person checklist (do this for YOUR claimed interaction)

Copy this checklist under your section below and work through it in order.

- [ ] **1. Define the interaction.** One sentence: who the actor is, and the *critical feature* of Keycloak they use. Tie it back to the enabling systems in our proposal's systems engineering view.
- [ ] **2. Draw the use case diagram** in draw.io. Use cases = features Keycloak supports (not user goals in the abstract). Keep it simple at first — actor, 2–4 use cases, `<<include>>` dependencies where real (e.g., Login includes Password Hashing).
- [ ] **3. Add misuse cases.** Pick a misuser contextualized to our environment — the *name* should convey motive, resources, attack of choice, and access (e.g., "Credential-stuffing botnet operator with breached password lists," not "Hacker"). Start from the threats in our proposal.
- [ ] **4. Iterate.** Go back and forth: misuse case threatens a use case → add a security use case that mitigates it → ask what threatens *that* → repeat until you hit specific functional security requirements Keycloak could implement. **Prioritize mitigations implemented in Keycloak itself, not the environment.** Every misuse case must be addressed by some use case.
- [ ] **5. Use proper notation** (class materials): white ovals = use cases, black/shaded ovals = misuse cases, `<<threatens>>` and `<<mitigates>>` arrows, misusers on the opposite side. Arrange to reduce clutter.
- [ ] **6. Optionally run your diagram description through an AI prompt** to find missed misuse cases (instructor's sample prompt is on the Canvas page). Save the prompt you used — we need one team example plus a reflection on whether it helped.
- [ ] **7. List your derived security requirements** — numbered, specific, functional ("Keycloak shall lock an account after N failed attempts," not "the system should be secure").
- [ ] **8. Alignment check:** for each requirement, does Keycloak actually advertise/implement it? Cite Keycloak docs or code (links). Note gaps.
- [ ] **9. Export your final diagram** (PNG + keep the .drawio source in the repo) and embed it in your section.
- [ ] **10. Log your tasks** on the GitHub Project Board and write your **individual reflection** (what did you learn, what was most useful).

---

### Interaction 1: End-User Authentication (Browser Login + OTP) — @Sewhenu-Ayeni

**Interaction description:**  
An employee uses Keycloak through a web browser to authenticate with a password and one-time password (OTP) before accessing an enterprise application. This interaction is essential because Keycloak provides the authentication service between the employee and the protected enterprise application.

**Use/misuse case diagram:**  
![Interaction 1 Use/Misuse Case Diagram](diagrams/interaction-1-misuse-iteration-4-final.png)


**Misuser profile:**  
**Misuser:** Credential-stuffing attacker using breached credentials

- **Motive:** Gain unauthorized access to an employee's account and enterprise application.
- **Resources:** Lists of previously breached usernames/passwords and automated login tools.
- **Attack of choice:** Credential stuffing through repeated browser login attempts.
- **Available access:** External access to the Keycloak login page, with no legitimate employee account or administrative access.

**Iteration narrative:**  
**Iteration 1 – Credential Stuffing:** A credential-stuffing attacker may use breached username and password combinations to repeatedly attempt authentication through the Keycloak login page. The misuse case **Submit Reused or Stolen Credentials** threatens **Password Authentication**. **Brute-Force Protection** mitigates repeated failed authentication attempts by tracking failures and applying configured lockout protections.

**Iteration 2 – Correct Stolen Password:** If the attacker possesses a correct stolen password, brute-force protection alone may not prevent unauthorized authentication because the password itself is valid. The misuse case **Use Correct Stolen Password** threatens **Authenticate to Application**. **OTP Authentication** mitigates this misuse by requiring an additional authentication factor when OTP is configured as required.

**Iteration 3 – Repeated OTP Guessing:** After obtaining a correct password, an attacker may repeatedly attempt to guess the employee's OTP. The misuse case **Repeatedly Guess OTP** threatens **OTP Authentication**. **Secondary Authentication Failures Lockout** mitigates repeated failures against the second authentication factor according to its configured threshold.

**Iteration 4 – Account Lockout Denial of Service:** An attacker may intentionally cause repeated authentication failures against known employee accounts so that account lockout protections deny legitimate users access. The misuse case **Abuse Account Lockout to Deny Access** threatens **Authenticate to Application**. **Login Failure Monitoring and IP Blocking** mitigates this misuse by using authentication-failure and client-address information to identify attack sources. Keycloak provides the failure information, while blocking an attacking IP may require an external intrusion-prevention or firewall mechanism.

**Derived security requirements:**

| ID | Requirement | Addresses misuse case | Implemented in Keycloak? (doc/code link) |
|---|---|---|---|
| SR-1.1 | Keycloak shall detect repeated failed password authentication attempts and temporarily or permanently disable further login attempts according to the configured brute-force protection settings. | Submit Reused or Stolen Credentials | **Yes.** Keycloak provides configurable brute-force detection, including temporary and permanent lockout. [Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/#password-guess-brute-force-attacks) |
| SR-1.2 | Keycloak shall require a valid OTP as a second authentication factor when OTP is configured as required in the employee authentication flow. | Use Correct Stolen Password | **Yes.** Keycloak supports OTP as a second-factor authenticator in configurable authentication flows. [Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/) |
| SR-1.3 | Keycloak shall track repeated failed OTP authentication attempts and apply a secondary authentication failure lockout according to the configured threshold. | Repeatedly Guess OTP | **Yes.** Keycloak provides Secondary Authentication Failures Lockout for failures against second-factor authenticators such as OTP. [Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/#password-guess-brute-force-attacks) |
| SR-1.4 | Keycloak shall record authentication failures with information sufficient to support monitoring of repeated login failures and identification of the originating client address. | Abuse Account Lockout to Deny Access | **Partially.** Keycloak provides login-failure and client-IP information, but blocking attacking IP addresses relies on an external intrusion-prevention or firewall mechanism. [Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/#password-guess-brute-force-attacks) |

**Alignment observations:**  
The misuse case analysis generally aligns with security capabilities available in Keycloak. Keycloak provides configurable brute-force detection, OTP-based second-factor authentication, and protection against repeated secondary authentication failures. However, some of these protections require administrator configuration and are not enabled by default. The account-lockout denial-of-service scenario also identifies a limitation in Keycloak's protection boundary: Keycloak can provide authentication-failure and client-IP information, but blocking the source of an attack may require an external intrusion-prevention or firewall mechanism.

---

### Interaction 2: <title> — <name>

**Interaction description:**
*(1–2 sentences: actor, feature, why it's essential)*

**Use/misuse case diagram:**
`![Diagram](images/usecase-1.png)`

**Misuser profile:** *(name, motive, resources, attack of choice, access)*

**Iteration narrative:** *(brief: misuse case → countermeasure → next misuse case → …)*

**Derived security requirements:**

| ID | Requirement | Addresses misuse case | Implemented in Keycloak? (doc/code link) |
|---|---|---|---|
| SR-1.1 | | | |
| SR-1.2 | | | |

**Alignment observations:** *(sufficiency of Keycloak's features vs. what the analysis expects)*

### Interaction 3: Relying Client Application — Token Acquisition via the OIDC Authorization Code Flow — [@JBoogieman](https://github.com/JBoogieman)

#### Description

A registered client application, such as a confidential server-side web app or a public single-page or mobile app, sends the user's browser to Keycloak's authorization endpoint, receives a short-lived authorization code at its registered redirect URI, and exchanges that code at the token endpoint for ID, access, and refresh tokens. It later uses the refresh token to obtain new access tokens without sending the user back through login. In our environment, these clients are the HR and payroll, IT service desk, and finance applications described in our proposal, used by employees on managed workstations, remote employees, and contractors. Every protected resource in a Keycloak deployment is ultimately reached through tokens obtained this way, which makes this the highest-value interaction between Keycloak and the applications it protects. It sits inside the authorization and credential subsystem our team scoped for the design and code-analysis deliverables.

#### Use/misuse case diagram

<img width="2583" height="1406" alt="Interaction 3 use/misuse case diagram" src="https://github.com/user-attachments/assets/af8761a6-5f3f-4058-ae86-bf7604bdf1bc" />

#### Actors and misusers

| *Type* | *Name* | *Motive, resources, attack of choice, access* |
| ----- | ----- | ----- |
| Actor | *Relying Client Application* | Registered confidential or public client, such as our HR and payroll, IT service desk, or finance applications, that needs tokens to act on a user's behalf. |
| Actor | *End User (Resource Owner)* | An employee or contractor who authenticates in the browser and consents to the scopes the client requests. |
| Misuser | *Co-resident Code Interceptor* | A malicious app on the same device as a public client, such as a remote employee's personal phone that the organization does not manage. Needs no network position; registers the same URI scheme and captures the code as the OS routes the redirect. Wants tokens without ever learning the password. |
| Misuser | *Redirect-Manipulating Phisher* | External, no account, no insider access. Wants an employee's or contractor's authorization code, and through it their access to payroll or finance data, without needing the password. Sends victims an authorization link built from a legitimate `client_id` and an attacker-controlled `redirect_uri`. The phish is convincing because the login page really is Keycloak. |
| Misuser | *Refresh Token Scavenger* | Holds a refresh token lifted from browser storage (via XSS), a mobile backup, or a log. Wants durable access that survives a password change, and will race the legitimate client to use it. |

#### Iteration narrative

Each round introduces a countermeasure, then asks what defeats that countermeasure. Requirement IDs refer to the table below.

*Round 1, Code interception.* The base use case is *Authorization Code Grant*, and the misuse case *Intercept Authorization Code* threatens it: the Co-resident Code Interceptor captures the code as the OS delivers the redirect. For a public client with no secret, whoever presents the code receives the tokens, so neither TLS nor the user's correct login helps. → *PKCE Verification*: the client sends a SHA-256 hash of a one-time secret (`code_challenge`) with the authorization request and must present the secret itself (`code_verifier`) at the token endpoint, over a back channel the interceptor cannot observe. Keycloak enforces this whenever a challenge was sent (SR-3.1).

*Round 2, Defeating PKCE by avoiding it.* PKCE only protects flows that use it, and the misuse case *Omit / Downgrade PKCE Challenge* threatens *PKCE Verification* directly. A client that never sends a challenge, whether it is legacy, misconfigured, or not written for PKCE, receives codes bound to nothing, which reopens Round 1. A client that uses the `plain` method sends the verifier itself through the front channel, where the interceptor can read it. Keycloak already rejects a `code_verifier` presented when no challenge was sent, which is the downgrade described in RFC 9700 §4.8.2 (SR-3.3). For a client without a PKCE requirement, though, it has no basis to refuse an authorization request that simply omits the challenge. → *Require PKCE (S256) per Client*: a server-side policy that refuses any authorization request without an S256 challenge (SR-3.2). A control the attacker can opt out of is not a control.

*Round 3, Attacking the redirect instead.* In the misuse case *Manipulate Redirect URI*, which threatens *Authorization Code Grant*, the Redirect-Manipulating Phisher sends a victim an authorization link that pairs a legitimate `client_id` with the attacker's own `redirect_uri`. The victim signs in at the real Keycloak domain, sees a valid certificate, and the code is delivered to the attacker. For a public client, PKCE does not help here because the attacker started the flow and generated the verifier; only a confidential client's secret would stop the attacker from redeeming the code directly. What makes the attacker's URI acceptable is loose matching, such as a trailing wildcard or a localhost registration left in production. → *Exact Redirect URI Validation*, with wildcard registrations prohibited (SR-3.4).

*Round 4, Stealing what the flow produces.* With the code path hardened, the Refresh Token Scavenger targets the output of the flow. A refresh token is a long-lived bearer credential that mints new access tokens and can outlive the user's password change. The base use case is now *Refresh Access Token*, and the misuse case *Replay Stolen Refresh Token* threatens it. → *Refresh Token Rotation & Reuse Detection*: each refresh returns a new refresh token and invalidates the old one, so a spent token is rejected (SR-3.5). Because the server cannot tell which party presented a spent token, the safe response to detected reuse is to revoke the active token as well (SR-3.6). Access tokens stay short-lived regardless of session length (SR-3.7).

*Round 5, Winning the race.* Rotation only helps if the legitimate client refreshes first. If the Scavenger refreshes first, the legitimate client is left holding the spent token; this is the misuse case *Win the Refresh Race*, which threatens *Refresh Token Rotation & Reuse Detection*. Our review of Keycloak's refresh path found that it rejects the spent token (`Stale token`) but found no logic that revokes the active one, so in that race it is the legitimate client that gets locked out while the attacker keeps a valid, rotating chain. Even with revocation, rotation is detective: a thief can use the token until the next legitimate refresh trips detection. → *Sender-Constrained Tokens (DPoP / mTLS)*: bind tokens to a key the thief does not hold, so a copied token is useless on its own (SR-3.8).

*Scope note.* Three further attack classes were considered and left out to keep the diagram focused on the chain above. Replaying a code harvested from logs is already mitigated by default because codes are single-use, and a replay detaches the client session created by the first redemption ([`OAuth2CodeParser`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/utils/OAuth2CodeParser.java), [`AuthorizationCodeGrantType`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/grants/AuthorizationCodeGrantType.java)). A leaked confidential-client secret is addressed by Keycloak's signed-JWT and X.509 client authenticators, which are available but not the default. Token forgery through `alg: none` or algorithm confusion is ultimately decided by how resource servers validate tokens, which occurs outside Keycloak.

#### Derived security requirements

Status key: *Default* (enforced out of the box). *Opt-in* (implemented but off until an administrator enables it). *Partial* (implemented with a material limitation). *Not found* (no implementation located in our code review).

| ID | Misuse case (round) | Requirement | In Keycloak | Evidence |
| ----- | ----- | ----- | ----- | ----- |
| SR-3.1 | Intercept Authorization Code (1) | Keycloak shall verify the PKCE `code_verifier` against the stored `code_challenge` before issuing tokens whenever a challenge was supplied. | *Default* | [`PkceUtils`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/utils/PkceUtils.java); [RFC 7636](https://www.rfc-editor.org/rfc/rfc7636.html) |
| SR-3.2 | Omit / Downgrade PKCE Challenge (2) | Keycloak shall let an administrator require PKCE with the S256 method for a client, and shall then reject authorization requests that omit the challenge or use `plain`. | *Opt-in*, configured via "Require PKCE" in the admin console (26.6+) or the `pkce-enforcer` client-policy executor. Without it the request proceeds, and the code only logs "PKCE non-supporting Client" at debug level, which default logging does not show. | [`AuthorizationEndpointChecker`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/endpoints/AuthorizationEndpointChecker.java); [PR #44365](https://github.com/keycloak/keycloak/pull/44365); [RFC 9700 §2.1.1](https://www.rfc-editor.org/rfc/rfc9700.html#section-2.1.1) |
| SR-3.3 | Omit / Downgrade PKCE Challenge (2) | Keycloak shall reject a token request that presents a `code_verifier` when no `code_challenge` was sent in the authorization request. | *Default*, triggers "PKCE code verifier specified but challenge not present in authorization" | [`PkceUtils`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/utils/PkceUtils.java); [RFC 9700 §4.8.2](https://www.rfc-editor.org/rfc/rfc9700.html#section-4.8.2) |
| SR-3.4 | Manipulate Redirect URI (3) | Keycloak shall match `redirect_uri` against registered values by exact string comparison and shall let an administrator prohibit wildcard registrations. | *Partial*, exact when no wildcard is registered, but a trailing `*` is honored as a prefix match. The `secure-redirect-uris-enforcer` executor (Keycloak 24+) can prohibit wildcards, yet "there are no client policies configured by default." | [`RedirectUtils`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/utils/RedirectUtils.java); [Client policies docs](https://github.com/keycloak/keycloak/blob/main/docs/documentation/server_admin/topics/clients/client-policies.adoc); [Keycloak 24.0.0 release notes](https://github.com/keycloak/keycloak/blob/main/docs/documentation/release_notes/topics/24_0_0.adoc); [RFC 9700 §2.1](https://www.rfc-editor.org/rfc/rfc9700.html#section-2.1) |
| SR-3.5 | Replay Stolen Refresh Token (4) | Keycloak shall issue a new refresh token on every refresh and reject any previously issued one. | *Opt-in*, configured via *Revoke Refresh Token* / *Refresh Token Max Reuse*, realm-wide only (a per-client override is an open proposal, [PR #51798](https://github.com/keycloak/keycloak/pull/51798)). See [CVE-2026-9802](https://www.cve.org/CVERecord?id=CVE-2026-9802). | [`TokenManager`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/TokenManager.java); [RFC 9700 §2.2.2](https://www.rfc-editor.org/rfc/rfc9700.html#section-2.2.2) |
| SR-3.6 | Replay Stolen Refresh Token (4) | On detecting reuse of a spent refresh token, Keycloak shall also revoke the currently active token for that grant. | *Not found*, spent tokens are rejected ("Stale token"), but no revocation of the active token was located in our code review. | [`TokenManager`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/TokenManager.java); [`RefreshTokenGrantType`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/grants/RefreshTokenGrantType.java) |
| SR-3.7 | Replay Stolen Refresh Token (4) | Keycloak shall issue short-lived access tokens with a lifespan independent of session length. | *Default*, configured via realm *Access Token Lifespan*, overridable per client. | [Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/index.html) |
| SR-3.8 | Win the Refresh Race (5) | Keycloak shall support sender-constrained tokens so that a copied token cannot be used without the holder's key. | *Opt-in*, DPoP officially supported since 26.4 (*Require DPoP bound tokens*; can bind only refresh tokens for public clients); mTLS certificate-bound tokens. | [DPoP in Keycloak 26.4](https://www.keycloak.org/2025/10/dpop-support-26-4); [RFC 9449](https://www.rfc-editor.org/rfc/rfc9449.html); [RFC 8705](https://www.rfc-editor.org/rfc/rfc8705.html); [RFC 9700 §2.2.1](https://www.rfc-editor.org/rfc/rfc9700.html#section-2.2.1) |

#### Alignment observations

*Coverage is complete; the default posture is not.* Keycloak implements a mitigation for every misuse case in this analysis, and three of the eight requirements are enforced out of the box. The four that answer the likeliest attacks are opt-in or only partially enforced: requiring PKCE (SR-3.2), exact redirect matching (SR-3.4), refresh token rotation (SR-3.5), and sender-constrained tokens (SR-3.8). Keycloak even ships the enforcement pre-packaged, including the `pkce-enforcer` and `secure-redirect-uris-enforcer` executors, plus global client profiles pre-configured for FAPI and OAuth 2.1. Yet, in the documentation's own words, "there are no client policies configured by default." The code states the consequence plainly: for a client without a PKCE requirement, the authorization endpoint logs "PKCE non-supporting Client" at debug level, invisible under default logging, and carries on. The security of this interaction therefore rests on administrator configuration rather than on the software's defaults. This is the configuration-mistake risk our proposal identified: whether our HR, payroll, and finance applications are protected against these attacks depends on how each of their clients was configured.

*Measured against current best practice.* At default settings, Keycloak meets RFC 9700's authorization-server obligations for PKCE: it supports PKCE, enforces the verifier whenever a challenge was sent, and blocks the §4.8.2 downgrade. It does not enforce two authorization-server MUSTs out of the box: exact string matching of redirect URIs (§2.1), since a registered wildcard is accepted, and rotation or sender-constraining of public-client refresh tokens (§2.2.2), both of which are off by default. RFC 9700 also requires public clients to use PKCE, but Keycloak does not compel them to unless the requirement is configured. The one requirement we could not locate at all, revoking the active token when reuse is detected (SR-3.6), is the difference between rotation that *responds* to theft and rotation that only *notices* it.

*The weak points are where the vulnerabilities have been.* Keycloak's redirect URI validation received three CVEs between December 2023 and September 2024: [CVE-2023-6927](https://www.cve.org/CVERecord?id=CVE-2023-6927) (code or token theft from clients using a wildcard with the JARM `form_post.jwt` response mode), [CVE-2024-1132](https://www.cve.org/CVERecord?id=CVE-2024-1132) (redirect validation bypass, CVSS 8.1), and [CVE-2024-8883](https://www.cve.org/CVERecord?id=CVE-2024-8883) (open redirect when localhost or 127.0.0.1 is registered as a redirect URI). `RedirectUtils` now carries a guard against percent-encoded `../` sequences that is needed only because prefix matching exists; exact matching would make that code unnecessary. The refresh path received [CVE-2026-9802](https://www.cve.org/CVERecord?id=CVE-2026-9802) in May 2026: with rotation enabled and persistent sessions, a server restart allowed previously rotated refresh tokens to be reused. Both findings support the analysis, as the redirect CVEs sit in the wildcard and loopback handling that exact matching would switch off, and the rotation CVE shows that even the opt-in rotation control can fail silently, which is the case for sender-constrained tokens made in Round 5.

*Where Keycloak's responsibility ends.* A sender-constrained access token only helps if each resource server checks the binding, such as the DPoP proof or the client certificate, on every request. Keycloak can issue bound tokens, but that enforcement happens outside it. Bound refresh tokens are different because Keycloak checks those itself at the token endpoint. mTLS also depends on a PKI, a dependency DPoP removes. DPoP has been officially supported since Keycloak 26.4, needs no certificates, and can bind only the refresh tokens of public clients, which is exactly where RFC 9700 §2.2.2 places the obligation. Most of the gap identified here can therefore be closed from inside Keycloak's own configuration, which is why its defaults are the finding. Keycloak 26.6 moved in this direction by adding a "Require PKCE" switch, with a warning shown for public clients, to the admin console ([PR #44365](https://github.com/keycloak/keycloak/pull/44365)). This acts as a nudge rather than a changed default, keeping existing clients working. These configuration dependencies carry forward as explicit assumptions in our assurance case.

#### References

- IETF. [RFC 9700 — Best Current Practice for OAuth 2.0 Security](https://www.rfc-editor.org/rfc/rfc9700.html) (January 2025).
- IETF. [RFC 7636 — Proof Key for Code Exchange](https://www.rfc-editor.org/rfc/rfc7636.html); [RFC 9449 — DPoP](https://www.rfc-editor.org/rfc/rfc9449.html); [RFC 8705 — OAuth 2.0 Mutual-TLS](https://www.rfc-editor.org/rfc/rfc8705.html).
- Keycloak. [Server Administration Guide — Client Policies](https://github.com/keycloak/keycloak/blob/main/docs/documentation/server_admin/topics/clients/client-policies.adoc); [Keycloak 24.0.0 release notes](https://github.com/keycloak/keycloak/blob/main/docs/documentation/release_notes/topics/24_0_0.adoc); [Official Support for DPoP in Keycloak 26.4](https://www.keycloak.org/2025/10/dpop-support-26-4); [PR #44365 — Improve client creation with PKCE](https://github.com/keycloak/keycloak/pull/44365); [PR #51798 — Per-client Revoke Refresh Token](https://github.com/keycloak/keycloak/pull/51798).
- Keycloak source (`main` branch): [`AuthorizationEndpointChecker`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/endpoints/AuthorizationEndpointChecker.java), [`PkceUtils`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/utils/PkceUtils.java), [`RedirectUtils`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/utils/RedirectUtils.java), [`OAuth2CodeParser`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/utils/OAuth2CodeParser.java), [`AuthorizationCodeGrantType`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/grants/AuthorizationCodeGrantType.java), [`TokenManager`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/TokenManager.java), [`RefreshTokenGrantType`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/protocol/oidc/grants/RefreshTokenGrantType.java).
- CVE records: [CVE-2023-6927](https://www.cve.org/CVERecord?id=CVE-2023-6927), [CVE-2024-1132](https://www.cve.org/CVERecord?id=CVE-2024-1132), [CVE-2024-8883](https://www.cve.org/CVERecord?id=CVE-2024-8883), [CVE-2026-9802](https://www.cve.org/CVERecord?id=CVE-2026-9802) ([Keycloak issue #49426](https://github.com/keycloak/keycloak/issues/49426)).

### Interaction 4: <title> — <name>
*(same structure)*

### Interaction 5: <title> — <name>
*(same structure)*

---

## Part 1 — Team-level items (shared, assign at meeting)

| Task | Owner | Status |
|---|---|---|
| Confirm the 5 interactions cover different interactor types (no overlap) | Team — Friday mtg | |
| AI prompt example + reflection on its usefulness for improving diagrams | | |
| Summary of alignment findings across all 5 cases (sufficiency of Keycloak's security features vs. misuse case expectations) | | |
| Compile individual reflections into one team reflection | | |
| GitHub Project Board up to date + link in report: `<link here>` | | |
| Final assembly/formatting of this file + Canvas submission | | |

## Team Reflection (Part 1)

*(Compiled from individual reflections — each member answers: What did you learn? What did you find most useful?)*

---

## Part 2 — OSS documentation review (20 pts, team-level)

This part is **not** split into five cases. It's one deliverable: review Keycloak's **security-related configuration and installation documentation** and summarize what's missing or could be improved. The instructor's angle: docs contributions are an easy on-ramp to the open-source community.

Suggested split if we want everyone touching it — each person reviews one doc area for their claimed interaction's feature (e.g., #1 reviews authentication/OTP config docs, #3 reviews client/OIDC setup docs, #5 reviews LDAP federation docs), then one person merges observations.

**What to produce:**

| Doc area reviewed | Reviewer | Observations (missing / unclear / could improve) |
|---|---|---|
| Server installation & hardening guide | | |
| Authentication / credential configuration |[@Sewhenu-Ayeni](https://github.com/Sewhenu-Ayeni) | The documentation provides detailed guidance for password policies, OTP policies, authentication flows, and brute-force protection. However, the security guidance is spread across multiple sections and could be improved by providing a consolidated secure authentication configuration example for production environments. A step-by-step example combining a strong password policy, required OTP/2FA, and brute-force protection would make it clearer which protections should be configured together rather than requiring administrators to identify them across separate sections.|
| Client & token configuration | | |
| Federation / brokering configuration | | |
| Other: | | |

**Authentication / credential configuration sources reviewed:**
- [Keycloak Password Policies documentation](https://github.com/keycloak/keycloak/blob/main/docs/documentation/server_admin/topics/authentication/password-policies.adoc)
- [Keycloak OTP Policies documentation](https://github.com/keycloak/keycloak/blob/main/docs/documentation/server_admin/topics/authentication/otp-policies.adoc)
- [Keycloak Authentication Flows documentation](https://github.com/keycloak/keycloak/blob/main/docs/documentation/server_admin/topics/authentication/flows.adoc)
- [Keycloak Brute-Force documentation](https://github.com/keycloak/keycloak/blob/main/docs/documentation/server_admin/topics/threat/brute-force.adoc)

**Summary of observations:** *(what could be improved or is missing, overall)*

*(Optional stretch: note whether any finding is worth an actual docs issue/PR to the Keycloak project.)*

---

## Submission checklist (before Sep 29)

- [ ] All 5 interaction sections complete with embedded diagrams
- [ ] Every misuse case is mitigated by a use case in its diagram
- [ ] AI prompt + reflection included
- [ ] Team reflection compiled
- [ ] Part 2 summary complete
- [ ] Project board link works and shows task assignments
- [ ] Canvas submission: link to this file in the repo
