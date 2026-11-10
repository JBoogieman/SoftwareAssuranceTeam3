# Designing for Software Security Engineering — Team 3 (Keycloak)

<!--
TEAM NOTE (doesn't render on GitHub, only visible while editing):
- This file is the deliverable, so only put in what the grader should read. Owners, status and checklists live in the issues/project board.
- Due Fri Nov 20, 11:59 PM. The Canvas submission is a link to this file.
- Scope is the same as before: Keycloak's authorization and credential subsystem, built from our five interactions (Requirements) and five claims (Assurance Cases).
- One Level 1 DFD should cover all five interactions. Add a Level 2 only if one area gets too crowded to read. TMT threat analysis is run on the final diagram only, not on drafts.
- TMT only runs on Windows, and .tm7 files can't be merged, so one person owns the .tm7. Everyone else sends diagram changes and threat decisions through their issue.
- Use TMT's default template (SDL TM Knowledge Base). The Azure template adds cloud threats that don't apply to us.
- Every TMT threat gets a state and a Justification in TMT itself: the mitigation, or why it doesn't apply. That text is what the HTML report shows, and it's how prior-year teams documented mitigations. Several of them submitted reports with threats still "Not Started", so none of ours should be.
- The markdown tables below only carry the threats we found relevant, starting with cross-boundary flows. Same wording in both places.
- Each interaction lists its DFD flows as F<interaction>.<n>. Use the same noun phrase as the flow's label in TMT so the report's "Interaction:" headings line up with our tables.
- Prior-year reports had about 5-8 threats per flow and 30-120 threats total. Five interactions on one diagram will run higher, so draw only the flows each scenario needs.

DFD wording rules (Model Quality is 30 pts: "proper wording of the model elements, clean, coherent and valid DFD"):
- Process: verb phrase ("Authenticate user", "Issue tokens").
- Data flow: noun phrase for the data that moves ("Authorization code", "OTP code"), never an action.
- Every process has at least one flow in and one flow out.
- No flow goes straight between two external entities, two data stores, or an external entity and a data store. It has to pass through a process.
- Draw a trust boundary wherever the owner or privilege level changes. In TMT, cross-boundary flows have to physically cross the boundary line, or TMT won't generate the boundary threats.

Starting element list (draft, change it as needed):
- External entities: End user (browser), Realm administrator, Client application, Partner identity provider, Active Directory
- Processes: Authenticate user, Manage realm configuration, Issue tokens, Broker external login, Synchronize federated users
- Data stores: Keycloak database (realms, users, credentials, clients, role mappings), Session store, Event store
- Trust boundaries: Internet, Partner network, Corporate directory, Keycloak data tier

Files:
- threat-model/keycloak-threat-model.tm7 (TMT source)
- threat-model/dfd-level-1.png (diagram export, embedded below)
- threat-model/keycloak-threat-model-report.htm (TMT full HTML report)
-->

## Part 1 — Threat Modeling

### Scenarios covered

Each scenario from our [Requirements](Requirements-for-Software-Security-Engineering.md) and [Assurance Cases](Assurance-Cases-for-Software-Security-Engineering.md) deliverables is covered by the DFD below.

| # | Scenario | External entity | Trust boundary crossed | Assurance claim |
|---|---|---|---|---|
| 1 | End-user authentication (browser login + OTP) | End user (browser) | Internet → Keycloak | Claim 1 |
| 2 | Realm admin manages credentials and password policy | Realm administrator | Internet → Keycloak admin endpoints | Claim 2 |
| 3 | Client application token acquisition (OIDC authorization code flow) | Client application | Internet → Keycloak | Claim 3 |
| 4 | Identity brokering from an external identity provider | Partner identity provider | Partner network → Keycloak | Claim 4 |
| 5 | Directory federation (LDAP / Active Directory) | Active Directory | Keycloak → corporate directory | Claim 5 |

### Level 1 DFD

![Level 1 DFD](threat-model/dfd-level-1.png)

*Source: [`threat-model/keycloak-threat-model.tm7`](threat-model/keycloak-threat-model.tm7).*

TBD: two or three sentences walking through the diagram: the main processes and data stores, where the trust boundaries are, and which part of the diagram supports each scenario.

### TMT threat report

Full HTML report: [Keycloak threat model report](https://htmlpreview.github.io/?https://github.com/JBoogieman/SoftwareAssuranceTeam3/blob/main/threat-model/keycloak-threat-model-report.htm)

TMT generated TBD threats for the final diagram. Each one was reviewed in TMT and given a state: TBD mitigated, TBD not applicable, TBD needs investigation.

### Mitigations

<!--
Only the threats we found relevant and logical. Start with threats on flows that cross a trust boundary. Use TMT's threat IDs so each row can be checked against the report.
-->

#### Interaction 1 — End-user authentication

| TMT ID | Threat (STRIDE) | Flow | Mitigation strategy |
|---|---|---|---|
| TBD | TBD | TBD | TBD |

#### Interaction 2 — Realm admin manages credentials and password policy

| TMT ID | Threat (STRIDE) | Flow | Mitigation strategy |
|---|---|---|---|
| TBD | TBD | TBD | TBD |

#### Interaction 3 — Client application token acquisition

<!--
Rows were drafted per flow before the TMT run. Once the final .tm7 is run: put TMT's ID in the first column, match the threat wording to TMT's title, and paste the mitigation into that threat's Justification field. TMT threats on these flows that aren't listed here still get a state and justification in TMT.
-->

The client application sits outside Keycloak in the Internet zone, since public clients such as single-page and mobile apps run on users' devices. The authorization code reaches the client through the user's browser, a hop between two external entities that the DFD leaves out. Keycloak's side of the flow starts again when the client redeems the code at F3.3.

| Flow | Data | From → To | Crosses |
|---|---|---|---|
| F3.1 | Authorization request (`client_id`, `redirect_uri`, `state`, `code_challenge`) | End user (browser) → Authenticate user | Internet → Keycloak |
| F3.2 | Authorization response (code, `state`) | Authenticate user → End user (browser) | Keycloak → Internet |
| F3.3 | Token request (code and `code_verifier`, or refresh token; client credentials) | Client application → Issue tokens | Internet → Keycloak |
| F3.4 | Token response (ID, access, and refresh tokens) | Issue tokens → Client application | Keycloak → Internet |
| F3.5 | Authorization code record (PKCE challenge, redirect URI, user session) | Authenticate user → Session store | Keycloak data tier |
| F3.6 | Code record and client session | Session store → Issue tokens | Keycloak data tier |
| F3.7 | Client registration and signing keys | Keycloak database → Issue tokens | Keycloak data tier |
| F3.8 | Token events | Issue tokens → Event store | Keycloak data tier |
| F3.9 | Client session update (latest refresh token, reuse count) | Issue tokens → Session store | Keycloak data tier |

Redirect URI validation at F3.1 also reads the client's registration from the Keycloak database. That read shares the Keycloak database → Authenticate user flow with Interaction 1.

| TMT ID | Threat (STRIDE) | Flow | Mitigation strategy |
|---|---|---|---|
| TBD | **Spoofing.** An attacker who intercepted a code redeems it as a public client, which has no secret to prove who it is. | F3.3 | Require PKCE with `S256` for every public client through a realm client policy (`pkce-enforcer` executor with the `any-client` condition), so clients added later are covered too. *Quick win.* Move confidential clients from a shared secret to signed-JWT or mTLS client authentication. *Structural.* |
| TBD | **Tampering.** A phishing link pairs a real `client_id` with the attacker's `redirect_uri`, so Keycloak delivers the code to the attacker. | F3.1, F3.2 | Register exact redirect URIs only, and turn on the `secure-redirect-uris-enforcer` executor to reject wildcard and loopback registrations. *Quick win.* |
| TBD | **Information disclosure.** The code leaks from the redirect URL through browser history, logs, the Referer header, or an app on the same device. | F3.2 | Codes are single-use. Shorten the client login timeout from the 60-second default to under 10 seconds, as Keycloak's threat-mitigation docs advise. PKCE makes a leaked code useless without the verifier. *Quick win.* |
| TBD | **Information disclosure.** Tokens or client secrets are sniffed in transit. | F3.3, F3.4 | Run Keycloak in production mode, which won't start without HTTPS, and set the realm's *Require SSL* to all requests. The default (*external requests*) exempts private IP addresses, which covers our corporate LAN. *Quick win.* |
| TBD | **Spoofing.** A stolen refresh token is replayed to mint new access tokens. | F3.3 | Turn on *Revoke Refresh Token* with *Refresh Token Max Reuse* at 0, and keep access tokens short-lived. *Quick win.* Bind public clients' refresh tokens with DPoP so a copied token is useless without the client's key. *Structural.* |
| TBD | **Elevation of privilege.** An access token issued to a lower-trust client carries every role the user holds, so a service desk token is also accepted by payroll. | F3.4 | Turn off *Full Scope Allowed* and enforce it for new clients with the `full-scope-disabled` client-policy executor. *Quick win.* Grant each client only the roles it needs through role scope mappings, and set token audiences so resource servers can reject tokens issued for another client. *Structural.* |
| TBD | **Repudiation.** A client or user denies a code redemption or refresh, and Keycloak has no record to show otherwise. | F3.8 | Turn on *Save events* for `CODE_TO_TOKEN`, `REFRESH_TOKEN`, and their error types, and forward them to the SIEM. By default only error events are logged. *Quick win.* |
| TBD | **Denial of service.** The token endpoint is flooded with token requests. | F3.3 | Set `http-max-queued-requests` so Keycloak sheds load instead of queuing without limit. *Quick win.* Rate-limit by client and source IP at the reverse proxy. *Structural.* |
| TBD | **Information disclosure.** Anyone who can read the Keycloak database gets client secrets and the realm's signing keys, which is enough to forge tokens. | F3.7 | Store client secrets as vault references, and load signing keys through the `java-keystore` key provider instead of the default generated keys that live in the database. Restrict and audit database access. *Structural.* |
| TBD | **Tampering.** The stored code record or refresh-token reuse count is altered to get around PKCE or reuse detection. | F3.5, F3.6, F3.9 | Keep the cache and database on an isolated network segment, reachable only from Keycloak nodes. Cluster traffic between nodes is already TLS-encrypted by default. |

#### Interaction 4 — Identity brokering

<!--
Same process as Interaction 3: TMT IDs and Justification fields get filled in from the final .tm7 run.
-->

With SAML's redirect and POST bindings, F4.2 and F4.3 travel through the user's browser. They are drawn directly between Keycloak and the partner IdP because Keycloak trusts the partner's signature, not the browser. The browser can still read and resend these messages, which is why replay and disclosure threats appear on F4.3. After brokering, Broker external login hands the identity to Authenticate user (F4.6), which creates the session and returns the authorization response along the Interaction 1 and 3 flows.

| Flow | Data | From → To | Crosses |
|---|---|---|---|
| F4.1 | Broker login request | End user (browser) → Broker external login | Internet → Keycloak |
| F4.2 | SAML authentication request | Broker external login → Partner identity provider | Keycloak → Partner network |
| F4.3 | SAML response (signed assertion) | Partner identity provider → Broker external login | Partner network → Keycloak |
| F4.4 | Identity provider configuration and mappers | Keycloak database → Broker external login | Keycloak data tier |
| F4.5 | Brokered user, identity link, and mapped roles | Broker external login → Keycloak database | Keycloak data tier |
| F4.6 | Brokered identity context | Broker external login → Authenticate user | None (inside Keycloak) |
| F4.7 | Broker login events | Broker external login → Event store | Keycloak data tier |

Identity provider mappers are created over Interaction 2's Realm administrator → Manage realm configuration flow. The mapper threat is listed here because it only matters for brokered logins.

| TMT ID | Threat (STRIDE) | Flow | Mitigation strategy |
|---|---|---|---|
| TBD | **Spoofing.** A forged or unsigned SAML response impersonates the partner IdP. | F4.3 | Turn on *Validate Signature* and pin the partner's signing certificate, or load it from the partner's metadata URL over TLS. Set *Identity Provider Entity ID* so the Issuer is checked. *Want Assertions Signed* by itself only checks that a signature is present. *Quick win.* |
| TBD | **Tampering.** A genuinely signed response is wrapped around a second, unsigned assertion naming another user. | F4.3 | Keycloak verifies the signature on the same assertion element it reads. Keep *Validate Signature* on and stay on patched releases. |
| TBD | **Spoofing.** A captured SAML response is replayed. IdP-initiated responses have no `InResponseTo` value to tie them to a request. | F4.3 | Use SP-initiated login. Keycloak only accepts an IdP-initiated brokered login for a client that has an *IDP-Initiated SSO URL name*, so leave that unset on partner-facing clients. *Quick win.* Ask the partner for short assertion lifetimes and a `OneTimeUse` condition, which Keycloak enforces starting in 27.0. *Structural.* |
| TBD | **Information disclosure.** Assertion attributes can be read in the user's browser as the response passes through. | F4.3 | Turn on *Want Assertions Encrypted*, and ask the partner to release only the attributes our mappers use. |
| TBD | **Elevation of privilege.** The partner IdP asserts privileged group values that mappers turn into finance or payroll roles. | F4.3, F4.5 | Map partner claims only to a dedicated contractor role. Set *Sync Mode* to Force so mapped roles are re-checked at every login and removed when the partner drops the group. *Structural.* |
| TBD | **Elevation of privilege.** An administrator who only holds `manage-identity-providers` creates a mapper that grants a privileged role. | Interaction 2 admin flow | Leave *Allow granting admin roles via mappers* off (the default from 26.8.0). Limit who holds `manage-identity-providers`, and review mapper changes through admin events, since Keycloak doesn't restrict mappers that grant non-admin roles. |
| TBD | **Elevation of privilege.** The partner sets a contractor's email to an employee's address to take over that employee's account through linking. | F4.6 | Keep the default first broker login flow, which requires email verification or re-authentication before linking, and leave *Trust Email* off for the partner. |
| TBD | **Elevation of privilege.** IdP-initiated logins skip the *Account Linking Only* and disabled-client checks. | F4.3 | Run a release with the CVE-2026-16442 fix, which applies both checks on every brokered login. |
| TBD | **Repudiation.** No one can tell which assertion produced a login or a role grant. | F4.7 | Turn on *Save events* for `IDENTITY_PROVIDER_LOGIN`, `IDENTITY_PROVIDER_FIRST_LOGIN`, `FEDERATED_IDENTITY_LINK`, and their errors, plus admin events for mapper changes. *Store Tokens* keeps the original assertion, at some privacy cost. |
| TBD | **Denial of service.** The broker endpoint is flooded with large or deeply nested SAML responses that are costly to parse and verify. | F4.3 | Keep the XML parser's DOCTYPE ban in place, cap request size and rate at the reverse proxy, and set `http-max-queued-requests`. |
| TBD | **Elevation of privilege.** A contractor the partner has offboarded keeps using a Keycloak session or offline token. | F4.6 | Keep SSO idle and max timeouts short for partner users, remove `offline_access` from partner-facing clients, accept SAML logout requests from the partner, and agree on an offboarding notice with the partner. |

#### Interaction 5 — Directory federation

| TMT ID | Threat (STRIDE) | Flow | Mitigation strategy |
|---|---|---|---|
| TBD | TBD | TBD | TBD |

---

## Part 2 — Observations

<!--
For each high-priority threat from Part 1, check whether Keycloak actually implements the mitigation we expected. Cite the code, test, documentation, or default setting. Evidence from the Assurance Cases Part 2 tables can be reused where it applies.
-->

### Interaction 1 — End-user authentication

| TMT ID | Expected mitigation | In Keycloak? | Where we looked | Gap |
|---|---|---|---|---|
| TBD | TBD | Yes / Partial / Config-dependent / No | TBD | TBD |

### Interaction 2 — Realm admin manages credentials and password policy

| TMT ID | Expected mitigation | In Keycloak? | Where we looked | Gap |
|---|---|---|---|---|
| TBD | TBD | Yes / Partial / Config-dependent / No | TBD | TBD |

### Interaction 3 — Client application token acquisition

Code links are pinned to Keycloak `main` at commit [`d052236`][d3-commit] (November 10, 2026). The newest release at that time was 26.8.2.

| TMT ID | Expected mitigation | In Keycloak? | Where we looked | Gap |
|---|---|---|---|---|
| TBD | PKCE (`S256`) required for public clients | Config-dependent | [`AuthorizationEndpointChecker`][d3-aec]; [PR #44365][d3-pr44365] | When a client has no PKCE requirement, an authorization request without a `code_challenge` is accepted and only logged at debug level. The *Require PKCE* switch added in 26.6 makes the setting easier to find but doesn't change the default. |
| TBD | Exact redirect URI matching | Partial | [`RedirectUtils`][d3-redirect]; [client policies docs][d3-cp-docs] | A registered trailing `*` is still honored as a prefix match. The executor that blocks wildcards only runs if a policy uses it, and "there are no client policies configured by default." |
| TBD | Short-lived, single-use code bound to its redirect URI | Yes, with a long default lifetime | [`OAuth2CodeParser`][d3-codeparser]; [`AuthorizationCodeGrantType`][d3-codegrant]; [`DefaultExportImportManager`][d3-codelife]; [compromised codes docs][d3-codes-doc] | Codes are removed from the store when redeemed, and the `redirect_uri` must match. New realms get a 60-second lifetime, while Keycloak's own docs say to keep it under 10 seconds. |
| TBD | TLS on every flow | Partial | [Production mode docs][d3-prod]; [`RealmManager`][d3-sslreq]; [SSL docs][d3-ssl-doc] | Production mode won't start without HTTPS. The realm's default *Require SSL* setting exempts private IP addresses, so it adds no protection on our corporate LAN. |
| TBD | Refresh token rotation and a response to reuse | Partial | [`TokenManager.validateTokenReuse`][d3-tokenmgr] | Rotation is opt-in and realm-wide. A reused token is rejected as "Stale token", but the token chain that is still active is left alone, so SR-3.6 is still open on `main`. |
| TBD | Sender-constrained refresh tokens | Config-dependent | [DPoP in Keycloak 26.4][d3-dpop] | DPoP is off by default. Access-token binding only helps if each resource server checks the proof, which happens outside Keycloak. |
| TBD | Tokens limited to the roles a client needs | Config-dependent | [Client policies docs][d3-fullscope]; [scope docs][d3-scope] | Clients are created with *Full Scope Allowed* on, so every access token carries all of the user's roles. Keycloak has deprecated the switch and recommends disabling it, but it is still on by default. |
| TBD | Durable record of code redemptions and refreshes | No (by default) | [Login events docs][d3-events-doc]; [`JBossLoggingEventListenerProviderFactory`][d3-jboss] | *Save events* is off, and the default log listener writes success events at debug level. On a default realm, redeeming a stolen code leaves no record. |
| TBD | Rate limiting at the token endpoint | No | [Production configuration docs][d3-queue]; the `protocol/oidc` package | The only built-in control is `http-max-queued-requests`, which has no limit by default and sheds load for the whole server rather than throttling one client. Per-client limits have to come from the reverse proxy. |
| TBD | Client secrets and signing keys protected at rest | Partial | [`ClientAdapter`][d3-secret]; [vault docs][d3-vault]; [`DefaultKeyProviders`][d3-defkeys]; [`AbstractGeneratedRsaKeyProviderFactory`][d3-genrsa]; [`Constants`][d3-hs512]; [26.8.0 upgrade notes][d3-2680] | Client secrets are stored as entered and compared directly, not hashed, unless they are vault references. Each new realm's signing keys, including the HMAC key that signs refresh tokens, are generated into the database. Read access to the database is enough to forge tokens. 26.8.0 did stop returning client secrets to admins who only hold `view-clients`. |
| TBD | Cache and session data protected in transit | Yes | [Caching docs][d3-cache] | Cluster traffic is TLS-encrypted by default, but the keys for it are also stored in the database. |

[d3-commit]: https://github.com/keycloak/keycloak/commit/d052236442de082d459a404ce453713b36549594
[d3-aec]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/endpoints/AuthorizationEndpointChecker.java#L449-L452
[d3-pr44365]: https://github.com/keycloak/keycloak/pull/44365
[d3-redirect]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/RedirectUtils.java
[d3-cp-docs]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/server_admin/topics/clients/client-policies.adoc?plain=1#L184-L186
[d3-codeparser]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/utils/OAuth2CodeParser.java#L97-L98
[d3-codegrant]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/grants/AuthorizationCodeGrantType.java#L142-L157
[d3-codelife]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/model/storage-private/src/main/java/org/keycloak/storage/datastore/DefaultExportImportManager.java#L299-L300
[d3-codes-doc]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/server_admin/topics/threat/compromised-codes.adoc?plain=1#L6
[d3-prod]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/guides/server/configuration.adoc?plain=1#L242-L249
[d3-sslreq]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/managers/RealmManager.java#L319
[d3-ssl-doc]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/server_admin/topics/threat/ssl.adoc
[d3-tokenmgr]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/protocol/oidc/TokenManager.java#L351-L384
[d3-dpop]: https://www.keycloak.org/2025/10/dpop-support-26-4
[d3-fullscope]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/server_admin/topics/clients/client-policies.adoc?plain=1#L266-L272
[d3-scope]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/server_admin/topics/threat/scope.adoc?plain=1#L6
[d3-events-doc]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/server_admin/topics/events/login.adoc?plain=1#L5
[d3-jboss]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/events/log/JBossLoggingEventListenerProviderFactory.java#L56-L57
[d3-queue]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/guides/server/configuration-production.adoc?plain=1#L44-L53
[d3-secret]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/model/jpa/src/main/java/org/keycloak/models/jpa/ClientAdapter.java#L233-L235
[d3-vault]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/server_admin/topics/vault.adoc?plain=1#L42-L43
[d3-defkeys]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/utils/DefaultKeyProviders.java#L37-L38
[d3-genrsa]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/keys/AbstractGeneratedRsaKeyProviderFactory.java#L116-L120
[d3-hs512]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/Constants.java#L71
[d3-2680]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/upgrading/topics/changes/changes-26_8_0.adoc?plain=1#L7-L16
[d3-cache]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/guides/server/caching.adoc?plain=1#L282-L286

### Interaction 4 — Identity brokering

Code links are pinned to the same `main` commit as Interaction 3.

| TMT ID | Expected mitigation | In Keycloak? | Where we looked | Gap |
|---|---|---|---|---|
| TBD | Signature required on every SAML response from the partner | Config-dependent | [`SAMLIdentityProviderConfig`][d4-cfg]; [`SAMLEndpoint`][d4-sig] | *Validate Signature* is off unless an administrator turns it on, and a [code comment in `SAMLEndpoint`][d4-comment] calls disabled validation "the default today." *Want Assertions Signed* only checks that a signature is present. |
| TBD | Issuer checked against the partner's entity ID | Config-dependent | [`SAMLEndpoint`][d4-issuer]; [SAML identity provider docs][d4-issuer-doc] | Only runs when *Identity Provider Entity ID* is set. The docs say, "If empty, no Issuer validation is performed." |
| TBD | Signature covers the assertion that is processed | Yes | [`SAMLEndpoint`][d4-element]; [`KcSamlSignedBrokerTest`][d4-signed-test] | None found. The upstream tampering tests cover all eight combinations of signed document, signed assertion, and encryption. |
| TBD | Replayed responses rejected | Partial | [`validateInResponseToAttribute`][d4-inresp]; [`samlIdpInitiatedSSO`][d4-idpinit]; [OneTimeUse handling][d4-otu]; [27.0.0 release notes][d4-27] | SP-initiated responses are bound to the request ID. IdP-initiated responses have no request to bind to, so only the assertion's validity window applies. The new `OneTimeUse` cache is on `main` for 27.0 and isn't in 26.8.2, and it only covers assertions that carry that condition. IdP-initiated login is off unless a client has an IdP-initiated SSO URL name. |
| TBD | Hardened XML parsing | Yes | [`DocumentUtil`][d4-xml1]; [`StaxParserUtil`][d4-xml2] | None found. DOCTYPE declarations are rejected and external entities are disabled. |
| TBD | Assertion encrypted for Keycloak | Config-dependent | [`SAMLEndpoint`][d4-enc] | *Want Assertions Encrypted* is off unless set, and the partner has to support encryption. |
| TBD | Mappers can't grant privileged roles | Partial (was No) | [`IdentityProviderResource`][d4-mapper-admin]; [`AbstractIdentityProviderMapper`][d4-mapper-login]; [26.8.0 upgrade notes][d4-2680] | Keycloak 26.8.0 fixed [issue #50444][d4-50444], which our SR-4.3 cited, as CVE-2026-12388. Mappers can no longer grant Keycloak admin roles unless an administrator with `manage-realm` allows it, checked both when a mapper is saved and at login. Business roles, such as our finance and payroll roles, can still be granted by anyone with `manage-identity-providers`. SR-4.3 moves from *No* to *Partial*. |
| TBD | Mapped roles removed when the partner removes the group | Config-dependent | [`IdentityProviderMapperSyncModeDelegate`][d4-sync]; [`AbstractAttributeToRoleMapper`][d4-rolemapper] | Only in Force sync mode. In Import mode, a role granted at first login stays after the partner drops the group. |
| TBD | Proof of ownership before account linking | Yes (default flow) | [`DefaultAuthenticationFlows`][d4-link] | An administrator can replace the default flow with an auto-link flow. |
| TBD | Same checks on IdP-initiated logins | Yes (patched) | [`IdentityBrokerService`][d4-broker-svc]; [CVE-2026-16442][d4-cve16442] | Only on releases that include the fix. |
| TBD | Brokered logins and mapper role grants recorded | No (by default) | [Login events docs][d3-events-doc]; [`AbstractAttributeToRoleMapper`][d4-rolemapper] | *Save events* is off. Mappers call `user.grantRole` directly, which emits no event, and admin events only cover Admin REST API calls. *Store Tokens* is off, so the assertion behind a grant isn't kept. |
| TBD | Session ends when the partner offboards a contractor | Partial | [`SAMLEndpoint` logout handling][d4-logout]; [offline access docs][d4-offline] | Keycloak ends the session if the partner sends a SAML logout request, but it is never told when an account is disabled. Offline tokens outlive the session. |

[d4-cfg]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLIdentityProviderConfig.java#L123-L125
[d4-sig]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L629-L638
[d4-comment]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L882-L884
[d4-issuer]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L605-L614
[d4-issuer-doc]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/server_admin/topics/identity-broker/saml.adoc?plain=1#L22-L23
[d4-element]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L597-L603
[d4-signed-test]: https://github.com/keycloak/keycloak/blob/main/testsuite/integration-arquillian/tests/base/src/test/java/org/keycloak/testsuite/broker/KcSamlSignedBrokerTest.java
[d4-inresp]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L1173-L1198
[d4-idpinit]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L765-L777
[d4-otu]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L876-L937
[d4-27]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/release_notes/topics/27_0_0.adoc?plain=1#L23-L28
[d4-xml1]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/saml-core/src/main/java/org/keycloak/saml/common/util/DocumentUtil.java#L410-L418
[d4-xml2]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/saml-core/src/main/java/org/keycloak/saml/common/util/StaxParserUtil.java#L938-L942
[d4-enc]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L580-L586
[d4-mapper-admin]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/services/resources/admin/IdentityProviderResource.java#L575-L587
[d4-mapper-login]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/broker/provider/AbstractIdentityProviderMapper.java#L108-L123
[d4-2680]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/upgrading/topics/changes/changes-26_8_0.adoc?plain=1#L70-L82
[d4-50444]: https://github.com/keycloak/keycloak/issues/50444
[d4-sync]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/broker/provider/IdentityProviderMapperSyncModeDelegate.java#L16-L30
[d4-rolemapper]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/mappers/AbstractAttributeToRoleMapper.java#L42-L77
[d4-link]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/server-spi-private/src/main/java/org/keycloak/models/utils/DefaultAuthenticationFlows.java#L533-L660
[d4-broker-svc]: https://github.com/keycloak/keycloak/blob/6688a3d63f59e0c4a9131bfdd556c4312799f04e/services/src/main/java/org/keycloak/services/resources/IdentityBrokerService.java#L723-L770
[d4-cve16442]: https://www.cve.org/CVERecord?id=CVE-2026-16442
[d4-logout]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/services/src/main/java/org/keycloak/broker/saml/SAMLEndpoint.java#L348-L352
[d4-offline]: https://github.com/keycloak/keycloak/blob/d052236442de082d459a404ce453713b36549594/docs/documentation/server_admin/topics/sessions/offline.adoc

### Interaction 5 — Directory federation

| TMT ID | Expected mitigation | In Keycloak? | Where we looked | Gap |
|---|---|---|---|---|
| TBD | TBD | Yes / Partial / Config-dependent / No | TBD | TBD |

### Summary of gaps

TBD: the most important gaps across all five interactions, highest priority first.

<!--
Candidate points from Interactions 3 and 4, for whoever writes this summary:
- Most expected mitigations exist but are off by default: PKCE requirement, exact redirect matching, refresh token rotation, DPoP, SAML signature validation, Issuer check, assertion encryption, Force sync mode. Same pattern the Assurance Cases found.
- Repudiation is the weakest STRIDE category on default settings. Successful token redemptions and brokered logins aren't saved, and IdP mapper role grants never emit an event.
- The Keycloak database is the single most sensitive store: plaintext client secrets, the generated signing keys (including the refresh-token HMAC key), and the cluster TLS keys all live there.
- Upstream closed one of our earlier gaps: 26.8.0 (CVE-2026-12388) stops IdP mappers from granting admin roles. Non-admin business roles are still open, and SR-3.6 (revoke on refresh reuse) is still unimplemented.
-->



---

## Project Board

Task assignments and collaboration for this deliverable: [Project Board](https://github.com/users/JBoogieman/projects/1), milestone TBD.

## Team Reflection

TBD: compiled from each member's answers to "What did you learn from this assignment?" and "What did you find most useful?"

