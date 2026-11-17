# Designing for Software Security Engineering — Team 3 (Keycloak)

<!--
TEAM NOTE (doesn't render on GitHub, only visible while editing):
- This file is the deliverable, so only put in what the grader should read. Owners, status and checklists live in the issues/project board.
- Due Fri Nov 20, 11:59 PM. The Canvas submission is a link to this file.
- Scope is the same as before: Keycloak's authorization and credential subsystem, built from our five interactions (Requirements) and five claims (Assurance Cases).
- One Level 1 DFD should cover all five interactions. Add a Level 2 only if one area gets too crowded to read. TMT threat analysis is run on the final diagram only, not on drafts.
- TMT only runs on Windows, and .tm7 files can't be merged, so one person owns the .tm7. Everyone else sends diagram changes and threat decisions through their issue.
- Use TMT's default template (SDL TM Knowledge Base). The Azure template adds cloud threats that don't apply to us.

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

| TMT ID | Threat (STRIDE) | Flow | Mitigation strategy |
|---|---|---|---|
| TBD | TBD | TBD | TBD |

#### Interaction 4 — Identity brokering

| TMT ID | Threat (STRIDE) | Flow | Mitigation strategy |
|---|---|---|---|
| TBD | TBD | TBD | TBD |

#### Interaction 5 — Directory federation

Employee accounts and groups live in Active Directory on the corporate network, which Keycloak doesn't control. Synchronize federated users checks an employee's password by binding to AD as that user (F5.2, F5.3). It also periodically searches AD with a service account (F5.4, F5.5), imports the results, and maps AD groups to Keycloak roles (F5.6). Every workforce identity and role crosses the Corporate directory boundary, so most of this interaction's risk sits on F5.2–F5.5. Authenticate user then uses the result to create the session along the Interaction 1 and 3 flows (F5.9).


| TMT ID | Threat (STRIDE) | Flow | Mitigation strategy |
|---|---|---|---|
| Flow | Data | From → To | Crosses |
| F5.1 | Credential validation request (username, password) | Authenticate user → Synchronize federated users | None (inside Keycloak) |
| F5.2 | LDAP bind request (user DN, password) | Synchronize federated users → Active Directory | Keycloak → Corporate directory |
| F5.3 | Bind result | Active Directory → Synchronize federated users | Corporate directory → Keycloak |
| F5.4 | Directory search request (service account bind, search filter) | Synchronize federated users → Active Directory | Keycloak → Corporate directory |
| F5.5 | User entries and group memberships | Active Directory → Synchronize federated users | Corporate directory → Keycloak |
| F5.6 | Imported users and role mappings | Synchronize federated users → Keycloak database | None (inside Keycloak) |
| F5.7 | Federation configuration and mappers (bind credential, Users DN, Groups DN) | Keycloak database → Synchronize federated users | None (inside Keycloak) |
| F5.8 | Federation events | Synchronize federated users → Event store | None (inside Keycloak) |
| F5.9 | Validated user and federated roles | Synchronize federated users → Authenticate user | None (inside Keycloak) |

The LDAP provider and its mappers are configured over Interaction 2's Realm administrator → Manage realm configuration flow. The mapper threat is listed here because it only matters for federated users.



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

| TMT ID | Expected mitigation | In Keycloak? | Where we looked | Gap |
|---|---|---|---|---|
| TBD | TBD | Yes / Partial / Config-dependent / No | TBD | TBD |

### Interaction 4 — Identity brokering

| TMT ID | Expected mitigation | In Keycloak? | Where we looked | Gap |
|---|---|---|---|---|
| TBD | TBD | Yes / Partial / Config-dependent / No | TBD | TBD |

### Interaction 5 — Directory federation

| TMT ID | Expected mitigation | In Keycloak? | Where we looked | Gap |
|---|---|---|---|---|
| TBD | **Information disclosure.** Employees' passwords and the service account's password are sniffed during LDAP binds, because Keycloak accepts a plaintext `ldap://` connection URL. | F5.2, F5.4 | Use an `ldaps://` connection URL or turn on *Enable StartTLS*. *Quick win.* Add a realm setting that refuses plaintext federation providers. *Structural.* |
| TBD | **Spoofing.** A rogue LDAP server impersonates AD and collects every password sent to it in bind requests. | F5.2, F5.3 | Load AD's CA certificate into Keycloak's truststore and set *Use Truststore SPI* so the server certificate and hostname are checked. *Quick win.* |
| TBD | **Elevation of privilege.** Someone with group-write access in AD creates a group with the same name in another OU, and it gets mapped to a privileged role. | F5.5, F5.6 | Point the group mapper's *LDAP Groups DN* at a dedicated OU, and never map AD groups to `realm-admin` or composite admin roles. *Quick win.* Stay on releases with the DN-based group comparison fix (CVE-2026-19608). *Structural.* |
| TBD | **Spoofing.** A stolen service account password gives read access to the whole directory. | F5.4, F5.7 | Use a read-only, least-privileged bind account and set *Edit Mode* to `READ_ONLY`. *Quick win.* Store the bind credential in a vault (`${vault.<key>}`) instead of the database, and rotate it. *Structural.* |
| TBD | **Spoofing.** Password spraying through Keycloak's login page reaches AD and locks out employee accounts. | F5.1, F5.2 | Turn on *Brute Force Detection* with a lockout threshold below AD's, so Keycloak stops attempts before AD locks the account. *Quick win.* |
| TBD | **Denial of service.** Repeated syncs, or crafted entries with huge multi-valued attributes, exhaust Keycloak's memory and the database. | F5.4, F5.5, F5.6 | Turn on *Pagination*, set a sensible *Batch size*, and narrow the *Users DN* and LDAP filter. *Quick win.* Add a sync throttle and a cap on values per attribute. *Structural.* |
| TBD | **Denial of service.** An AD outage blocks every employee login. | F5.2, F5.3 | List more than one domain controller in the connection URL and set *Connection timeout* and *Read timeout*. *Quick win.* |
| TBD | **Repudiation.** No one can tell why a user received a role, or who triggered a sync. | F5.8 | Turn on admin events with *Include Representation* and forward events to the SIEM. *Quick win.* Record which group DN granted which role at each mapping. *Structural.* |

### Summary of gaps

TBD: the most important gaps across all five interactions, highest priority first.

---

## Project Board

Task assignments and collaboration for this deliverable: [Project Board](https://github.com/users/JBoogieman/projects/1), milestone TBD.

## Team Reflection

TBD: compiled from each member's answers to "What did you learn from this assignment?" and "What did you find most useful?"

