# Upstream Contributions

| Issue | PR | Weakness | Status |
|---|---|---|---|
| [#20008](https://github.com/keycloak/keycloak/issues/20008) UMA policy creation missing from the admin audit log | [#53316](https://github.com/keycloak/keycloak/pull/53316) | [CWE-778](https://cwe.mitre.org/data/definitions/778.html) Insufficient Logging | Approved, all CI checks passing, waiting on code-owner review |
| [#53249](https://github.com/keycloak/keycloak/issues/53249) Brute force failure count never reset by identity provider logins | [#53724](https://github.com/keycloak/keycloak/pull/53724) | [CWE-645](https://cwe.mitre.org/data/definitions/645.html) Overly Restrictive Account Lockout Mechanism | PR open, waiting on maintainer review |

---

## #20008: UMA policy creation missing from the admin audit log

**Status:** [PR #53316](https://github.com/keycloak/keycloak/pull/53316) is approved and all CI checks pass. It's waiting on code-owner review before it can merge.

### The issue

Keycloak's Protection API lets a resource server manage UMA policies on behalf of resource owners. With admin events turned on, updating or deleting a UMA policy through `/realms/{realm}/authz/protection/uma-policy/...` recorded an `AUTHORIZATION_POLICY` admin event, but **creating** one did not. Anyone reading the audit log would see a policy being updated or deleted that, as far as the log showed, never existed.

**Why it matters for assurance:** admin events are Keycloak's audit trail. A gap in it is [CWE-778: Insufficient Logging](https://cwe.mitre.org/data/definitions/778.html): an authorization change happened, and the people investigating can't see it.

### Where it lives in the source

All under `services/src/main/java/org/keycloak/authorization/`:

| File | Method | What it does |
|---|---|---|
| [`protection/policy/UserManagedPermissionService.java`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/authorization/protection/policy/UserManagedPermissionService.java) | `create()` | Protection API endpoint that creates a UMA policy |
| [`admin/PolicyService.java`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/authorization/admin/PolicyService.java) | `create(String payload)` | Admin REST endpoint. Calls the overload below, **then records the audit event** |
| [`admin/PolicyService.java`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/authorization/admin/PolicyService.java) | `create(AbstractPolicyRepresentation)` | Only saves the policy. **No audit event** |
| [`admin/PolicyResourceService.java`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/authorization/admin/PolicyResourceService.java) | `update()`, `delete()` | Save the change, **then record the audit event** |

### Thought process

1. **Reproduce it.** With admin events on, create, update and delete a UMA policy through the Protection API, then count the `AUTHORIZATION_POLICY` events. There were two (UPDATE and DELETE) instead of three.
2. **Compare the working path with the broken one.** Update and delete go through `PolicyResourceService`, which records its own audit event. Create was the odd one out.
3. **Follow the create call.** `UserManagedPermissionService.create()` calls `delegate.create(representation)`, which resolves to `PolicyService.create(AbstractPolicyRepresentation)`, the overload that only saves. The audit event is recorded one level up, in the `create(String payload)` overload that the admin console uses. The Protection API went around the method that does the logging.
4. **Choose where to fix it.** Moving the audit call down into `create(AbstractPolicyRepresentation)` would change behavior for every caller of that method, and the admin console path would log twice unless it was changed too. Recording the event in `UserManagedPermissionService.create()` fixes only the broken path.

### The fix

In `UserManagedPermissionService.create()`, after the policy is saved, record the CREATE event the same way update and delete do:

```java
Policy policy = delegate.create(representation);

representation.setId(policy.getId());
adminEvent.resource(ResourceType.AUTHORIZATION_POLICY)
        .operation(OperationType.CREATE)
        .resourcePath("authz", "protection", "uma-policy", policy.getId())
        .representation(representation)
        .success();

return findById(policy.getId());
```

### How we proved it

A new integration test, `testAdminEventsOnUmaPolicyLifecycle` in `UserManagedPermissionServiceTest`, creates, updates and deletes a UMA policy and expects exactly three `AUTHORIZATION_POLICY` admin events: CREATE, UPDATE and DELETE. Without the fix it fails (only two events); with the fix it passes.

### Takeaway

When two operations on the same object behave differently, compare their code paths side by side. This bug wasn't wrong code, it was a missing call: one entry point skipped the method that does the logging.

---

## #53249: Brute force failure count never reset by identity provider logins

**Status:** [PR #53724](https://github.com/keycloak/keycloak/pull/53724) is open, and Copilot's review has been addressed. It's waiting on a maintainer to approve the CI runs and review it.

### The issue

With brute force detection on, each failed password login adds to a user's failure count, and a successful login resets it. Some users normally sign in through an identity provider (SAML, OIDC or social) and only occasionally mistype their local Keycloak password. Their successful provider logins never reset the count, so the failures piled up until they were locked out, permanently if the realm uses permanent lockout.

**Why it matters for assurance:** this is [CWE-645: Overly Restrictive Account Lockout Mechanism](https://cwe.mitre.org/data/definitions/645.html), a lockout control that denies service to legitimate users. The catch is that the fix must not go too far the other way, into [CWE-307: Improper Restriction of Excessive Authentication Attempts](https://cwe.mitre.org/data/definitions/307.html), where an attacker gets unlimited guesses. Balancing those two drove the whole design.

### Where it lives in the source

All under `services/src/main/java/org/keycloak/services/`:

| File | Method or field | What it does |
|---|---|---|
| [`managers/AuthenticationManager.java`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/services/managers/AuthenticationManager.java) | `logSuccess()` | After a successful login, tells the brute force protector which credential types were used |
| [`managers/DefaultBruteForceProtector.java`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/services/managers/DefaultBruteForceProtector.java) | `ALLOWED_AUTHENTICATION_CATEGORIES`, `successfulLogin()` | Only resets the count if the login used a password, an OTP or a recovery code |
| [`resources/IdentityBrokerService.java`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/services/resources/IdentityBrokerService.java) | `finishBrokerAuthentication()` | Marks a login as coming through an identity provider (the `IDENTITY_PROVIDER` session note) |

### Thought process

1. **Reproduce it.** Write a failing test: fail one password login, log in successfully through the identity provider, then check the count. It stayed at 1, for both SAML and OIDC brokering.
2. **Find when it started.** Git history led to [PR #49996](https://github.com/keycloak/keycloak/pull/49996). Before that PR, any successful login reset the count, which caused its own bug, [#49960](https://github.com/keycloak/keycloak/issues/49960): logging in with an existing SSO cookie also reset the count, wiping out an attacker's failed guesses. #49996 fixed that by resetting only after a login that used a password, OTP or recovery code. A provider login uses none of those, so it stopped resetting too. That side effect is this issue.
3. **Reject the obvious fix.** Simply counting identity provider logins as a reset would reopen #49960. A provider can log a user in without them typing anything (from an existing session at the provider), which is the same silent-login problem cookie SSO had.
4. **Make it opt-in.** How far to trust a provider login depends on the provider: a corporate identity provider that enforces MFA is not the same as a social login. So the fix adds a per-provider setting that's off by default. Existing deployments behave exactly as before, and no database schema change is needed because the setting is stored in the provider's existing config.
5. **Make sure cookie SSO still can't reset the count.** The check reads the `IDENTITY_PROVIDER` note from the *current login's* authentication session. That note is only there on a login that actually went through the provider, not on a later cookie SSO login.

### The fix

- A new identity provider setting, `resetLoginFailures` (admin console: **Advanced settings > Reset brute force failures on login**).
- When the setting is on, `AuthenticationManager.logSuccess()` adds an `identity-provider` category for logins completed through that provider, and `DefaultBruteForceProtector` accepts that category as a reason to reset the count:

```java
Set<String> authenticationCategories = new HashSet<>(AuthenticatorUtil.getAuthnCredentials(authSession));
if (isBrokeredLoginResettingLoginFailures(session, authSession)) {
    authenticationCategories.add(DefaultBruteForceProtector.IDENTITY_PROVIDER_CATEGORY);
}
```

- Updated docs: the identity provider configuration table and the brute force detection page.

### How we proved it

Three integration tests in `AbstractAdvancedBrokerTest`, which run for SAML, OIDC and OAuth2 brokering:

| Test | What it checks |
|---|---|
| `loginWithBrokerDoesNotResetBruteForceFailureCountByDefault` | With the setting off, the count stays at 1 after a provider login |
| `loginWithBrokerResetsBruteForceFailureCountWhenEnabled` | With the setting on, the count resets to 0 |
| `loginWithBrokerThenCookieSsoDoesNotResetBruteForceFailureCount` | With the setting on: provider login, then a failed password attempt, then a cookie SSO login. The count stays at 1, so #49960 can't come back |

The third test came from Copilot's automated review, which pointed out that the PR described the cookie SSO case but didn't test it. The test makes its failed attempt with a direct grant (a password POST to the token endpoint), because the browser's SSO cookie skips the login form.

### Takeaway

Security fixes have side effects. #49996 closed one hole and created this bug, and the obvious fix for this bug would have reopened the first hole. Before changing security-relevant code, look up *why* it's written the way it is (git history and linked issues), and add a test for the case you're trying not to break.
