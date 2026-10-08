# CYBR8420 Software Assurance Team 3, Keycloak

Semester project for [CYBR 8420 Software Assurance](https://mlhale.github.io/swa/pages/project.html), a security
assessment of [Keycloak](https://github.com/keycloak/keycloak), an open source identity
and access management server.

**Scope.** Keycloak is too large to assess in full, so our design and code analysis work
is scoped to its authorization and credential subsystem.

## Deliverables

| # | Deliverable | Status |
|---|---|---|
| 1 | [Project Proposal](Project-Proposal.md) | Complete |
| 2 | [Requirements for Software Security Engineering](Requirements-for-Software-Security-Engineering.md) | Complete |
| 3 | [Assurance Cases Software Security Engineering](https://github.com/JBoogieman/SoftwareAssuranceTeam3/blob/main/Assurance-Cases-for-Software-Security-Engineering.md) | In-Progress |
| 4 | Designing for Software Security Engineering | Not started |
| 5 | Code analysis for Software Security Engineering | Not started |
| 6 | Recorded Presentation | Not started |

Diagram sources and exports are in [`diagrams/`](diagrams).

[Project Board](https://github.com/users/JBoogieman/projects/1)

## Upstream contributions
 
Fixes we've submitted back to Keycloak. For a walkthrough of how each bug was found, traced in the source, and fixed, see [Upstream Contributions](https://github.com/JBoogieman/SoftwareAssuranceTeam3/blob/main/UpstreamContributions.md).
 
| Issue | Fix | Status |
|---|---|---|
| [#20008](https://github.com/keycloak/keycloak/issues/20008) Creating a UMA policy via the Protection API wasn't recorded in the admin audit log | Protection API now records the CREATE admin event, matching update/delete, with an integration test ([PR #53316](https://github.com/keycloak/keycloak/pull/53316)) | Approved, all CI checks passing, waiting on code-owner review |
| [#53249](https://github.com/keycloak/keycloak/issues/53249) Brute-force failure count isn't reset after a successful IdP login | Opt-in per identity provider setting that lets successful provider logins reset the count without letting cookie SSO reset it (avoids reopening [#49960](https://github.com/keycloak/keycloak/issues/49960)), with integration tests for SAML, OIDC, and OAuth2 ([PR #53724](https://github.com/keycloak/keycloak/pull/53724)) | PR open, waiting on maintainer review |

## Team

| Member | Role | GitHub |
|---|---|---|
| Sewhenu Ayeni | Team Lead | [@Sewhenu-Ayeni](https://github.com/Sewhenu-Ayeni) |
| Ayden Riddle | Project Manager | [@AyRidd03](https://github.com/AyRidd03) |
| Justin Brueggemann | Technical Lead | [@JBoogieman](https://github.com/JBoogieman) |
| Isaiah James | Reviewer / QA | [@isaiahjames11](https://github.com/isaiahjames11) |
| Sean Anderson | Documentation | [@SeanAnderson0](https://github.com/SeanAnderson0) |

## Communication and Meetings

Primary channel: Discord

| Meeting | Day | Time |
|---|---|---|
| Primary huddle | Monday | 4:15 PM |
| Overflow | Friday | 2:00 PM |
