Edit test
1. **Project and Operational Environment**
   - GitHub Link: https://github.com/JBoogieman/CYBR8420-SoftwareAssuranceTeam3
   - Team Project Board: https://github.com/users/JBoogieman/projects/1
   - Chosen Software: Keycloak
   - Repository: https://github.com/keycloak/keycloak

2. **Systems Engineering View** — Justin
   - Diagram
   - Explanation of the system in the enterprise environment

3. **Security Needs, Threats, and Features** — Isaiah

   Security needs Why an IAM System Like Keycloak Matters

   Organizations that use multiple applications need a secure way to manage user logins and access. Instead of having each application handle usernames, passwords, and permissions on its own, an Identity and Access         Management (IAM) system like Keycloak can manage these tasks in one central location. Keycloak is designed to add authentication and secure applications without requiring developers to build their own login system       from scratch.

   Because Keycloak becomes a central part of an organization's security, protecting it is very important. A security problem with Keycloak could potentially give an attacker access to multiple applications. Important       security needs include protecting user credentials, securing login sessions and access tokens, preventing brute-force and credential-stuffing attacks, and safely connecting to services such as LDAP or Active             Directory. The Keycloak administration console also needs strong protection because administrators can control access to all connected applications.

   Keycloak's own security policies show how seriously the project takes these risks. Its security policy includes a process for reporting vulnerabilities, a dedicated security response team, and procedures for handling    CVEs. Overall, Keycloak is a good example of how centralizing identity can make managing security easier, but it also means the identity system itself must be strongly protected.

   Threats perceived by users

   Community discussions and GitHub issues show some of the security problems that Keycloak users deal with in real-world situations. These issues are not always major vulnerabilities or CVEs, but they can still create    security risks. For example, issue #16277 discusses making Keycloak's Content Security Policy (CSP) more secure because the default settings may be too permissive. Another issue, #9553, points out concerns with the       use of `'unsafe-inline'` in the CSP. Other issues, such as #31640 and #10340, involve problems with redirect URLs and hostname settings in the admin console. Incorrect configuration of these settings could                potentially create security problems such as open redirects.

   Users also have concerns about keeping Keycloak and its dependencies up to date. Issues such as #50360 discuss vulnerabilities in dependencies, while #12934 shows how older CVEs may require additional attention from    security teams. Overall, these community reports show that many of the security concerns with Keycloak are not necessarily complicated attacks. Instead, they often involve configuration mistakes, exposed admin           settings, weak security policies, and outdated dependencies. This shows why properly configuring, monitoring, and updating an IAM system like Keycloak is an important part of keeping the overall environment secure.

   Keycloak features

   Keycloak includes several built-in security features that help protect user accounts and applications. It has brute-force protection that can temporarily or permanently lock accounts after repeated failed login          attempts. Administrators can also create and enforce password policies. Keycloak supports multi-factor authentication (MFA) through OTP/TOTP, WebAuthn security keys, and passkeys. It also provides recovery codes so       users have another way to access their accounts if they lose their primary authentication method.

   Keycloak also provides tools for managing user sessions. Administrators and users can view and revoke active sessions, set session timeouts, and control how long different types of tokens remain valid. Step-up           authentication can require users to provide stronger authentication when performing sensitive actions. Keycloak also includes built-in protections against common attacks such as CSRF, clickjacking, SQL injection,        open    redirects, and SSRF. It supports HTTPS/SSL enforcement as well as controls for token revocation and access scopes.

   In addition to these technical security features, Keycloak follows several security practices as an open-source project. The project has an OpenSSF Best Practices badge, publishes a security scorecard, and has a         formal process for reporting and handling security vulnerabilities. Security researchers who report vulnerabilities can also receive credit in published security advisories. Overall, Keycloak provides multiple layers    of protection for user accounts, sessions, tokens, and the applications connected to the IAM system.

4. **Team Motivation** — Sean

   Our team chose Keycloak because several members were already interested in Identity and Access Management (IAM) and wanted to work with a real, widely-used tool. Some of us also saw it as a chance to strengthen a weaker area of our technical skills, since we didn't have much hands-on experience working with authentication. Keycloak also worked well since, it is a well-established open-source project with tons of documentation, history, and activity to study over a full semester. To keep things focused, we narrowed our project scope to authentication and credential security in an enterprise envrionment. Overall, we decided on Keycloak because it matched our interests, offered room to learn, and gave us a project we could scope for the semester.

5. **Open-Source Project Description** — Ayden
   - What Keycloak is
   - Contributors and activity
   - Use and popularity
   - Languages and platform
   - Documentation

6. **License and Contributions** – Sewhenu

   Keycloak is licensed under the **Apache License, Version 2.0**. The license permits the software to be used, modified, and distributed under its terms. When modified work is redistributed, required copyright and attribution notices must be retained and modified files must identify that changes were made. Contributions intentionally submitted to Keycloak are also provided under the Apache 2.0 license unless otherwise stated.

   Keycloak is an open-source, community-driven project that accepts contributions through GitHub. Each pull request should have an associated GitHub issue. Minor changes can proceed through an issue and pull request, while larger changes should first be discussed through GitHub Discussions so the proposed change can receive broader review. Pull requests should focus on one feature or change, include relevant tests and documentation, be rebased on the `main` branch, and use a descriptive commit message linked to the issue. Keycloak also requires the commits in a pull request to be squashed into a single commit. Maintainers review proposed changes and are responsible for approving contributions.

   Keycloak uses the **Developer's Certificate of Origin (DCO)** as a contributor requirement. Contributors must submit only work they have the legal right to contribute and that Keycloak can distribute under its license. Contributors are instructed to read the DCO and sign off their commits using the `--signoff` option with `git commit`. This adds a `Signed-off-by` line to the commit message and confirms the contributor's right to submit the contribution.

7. **Security History** — Sean
   
   Known / Currently Open Vulnerabilities:

   Keycloak has accumulated a substantial CVE history over its lifetime, spanning categories like authentication bypass, injection, and access control failures. Keycloak currently has 9 open CVEs, with one opened 3 days ago, and the longest-standing open CVE was opened on June 30th.

   CVE-2026-89298 - Confidential client secret disclosed to view-clients role via Client Registration GET. A user holding only the "view-clients" role, which should only grant read access to client metadata, can retrieve a client's confidential secret through the Client Registration GET endpoint. This vulnerability was opened 3 days ago.

   CVE-2026-18967 - SAML OneTimeUse Assertion Replay in IdP-Initiated Broker Flow. A SAML assertion flagged "OneTimeUse" (meant to be consumable exactly once) can be replayed within the IdP-initiated broker flow, defeating the protection that flag is supposed to provide. This vulnerability was opened about a month ago.

   CVE-2026-18569 - OIDC broker backchannel logout accepts unsigned forged logout tokens when signature validation is disabled. With signature validation turned off, the OIDC identity broker's backchannel logout endpoint will accept a forged, unsigned logout token, letting an attacker trigger a logout on another user's session. This vulnerability was opened Aug 3rd.

   CVE-2026-12388 - IdP mapper admin role escalation. A flaw in identity provider mapper handling allows escalation to an admin role. It's the most-discussed open CVE in the tracker (11 comments). This vulnerability was opened on June 30th.

   Fixed Vulnerabilities:

   These are examples of vulnerabilities that were identified, assigned a CVE, and patched in a specific release.

   CVE-2026-18963 - Unauthenticated account takeover via reset-credentials flow bypass. A flaw in the reset-credentials flow let an unauthenticated attacker force a password reset for any user without needing to click the required email verification link, allowing full account takeover. Critical severity, and by far the most-discussed fix in the tracker (30 comments).

   CVE-2026-79652 - JWT-bearer authorization grant does not enforce consentRequired. Keycloak's JWT-bearer OAuth2 grant type skipped the consent requirement, allowing a client to obtain tokens on a user's behalf without that user ever consenting. Since consent is a core part of the OAuth2/OIDC trust model, this represented a meaningful gap in the authorization flow.

   CVE-2026-19608 - Name-only group claims let same-name groups satisfy path-specific group policies. Authorization Services policies that were supposed to check a full group path could be satisfied by any group sharing just the same name, letting a user in an unrelated group of the same name pass a policy meant to restrict access to a specific nested group.

   CVE-2026-15571 - Predictable account-linking hash enables account takeover via malicious OIDC client. The hash used to link a user's account during OIDC identity brokering was predictable, so a malicious OIDC client could compute it and take over another user's account through the linking flow.

   Security Policy and Disclosure Process:

   Keycloak's vulnerability disclosure policy follows the CISA vulnerability disclosure policy template, with a dedicated Security Response Team centrally managing research. Access to CVE-related data follows the principle of least privilege among all vendors involved, and coordinated disclosure and embargo dates are agreed on before anything goes public. Severity drives the fix timeline. Depending on how severe a vulnerability is, it may be fixed in the current major or minor release, or deferred to the next one for lower severity issues, and organizations that can't upgrade regularly are pointed towards the Red Hat build of Keycloak for long-term support instead of staying on an unpatched version. The policy also sets explicit rules for AI-assisted vulnerability reports. Reports produced with AI assistance are accepted, but the reporter must validate the finding themselves, disclose that AI was used, and be able to explain the vulnerability and reproduction steps in their own words, while unreviewed AI output with generic descriptions or hallucinated endpoints is rejected outright without further analysis. Similarly, there is no blind trust in scanners. Raw output from automated security scanners is never accepted on its own, and the reporter has to triage the finding and supply a concrete proof of concept for Keycloak specifically, which is a deliberate decision to protect maintainer time from noisy, unvalidated reports. It is also worth noting that despite this formal process, Keycloak currently has no active bug bounty program, so reporters are credited through attribution in security advisories rather than paid.

8. **Team Reflection** — Sewhenu
   - Combined reflection from all five members

## References

- [Keycloak Repository](https://github.com/keycloak/keycloak)
- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [Keycloak Security Advisories](https://github.com/keycloak/keycloak/security/advisories)
- [Keycloak Security Policy](https://github.com/keycloak/keycloak/security)
- [Keycloak License](https://github.com/keycloak/keycloak/blob/main/LICENSE.txt)
- [Keycloak Contributing Guidelines](https://github.com/keycloak/keycloak/blob/main/CONTRIBUTING.md)
- [Keycloak Governance](https://github.com/keycloak/keycloak/blob/main/GOVERNANCE.md)
- [Keycloak Maintainers](https://github.com/keycloak/keycloak/blob/main/MAINTAINERS.md)
