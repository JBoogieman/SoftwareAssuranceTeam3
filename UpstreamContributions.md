# Upstream Contributions

| Issue | PR | Weakness | Status |
|---|---|---|---|
| [#20008](https://github.com/keycloak/keycloak/issues/20008) UMA policy creation missing from the admin audit log | [#53316](https://github.com/keycloak/keycloak/pull/53316) | [CWE-778](https://cwe.mitre.org/data/definitions/778.html) Insufficient Logging | Merged |
| [#53249](https://github.com/keycloak/keycloak/issues/53249) Brute force failure count never reset by identity provider logins | [#53724](https://github.com/keycloak/keycloak/pull/53724) | [CWE-645](https://cwe.mitre.org/data/definitions/645.html) Overly Restrictive Account Lockout Mechanism | PR open, waiting on maintainer review |
| [#53325](https://github.com/keycloak/keycloak/issues/53325) AuthZen caller can impersonate another client towards client policies | [#55186](https://github.com/keycloak/keycloak/pull/55186) | [CWE-807](https://cwe.mitre.org/data/definitions/807.html) Reliance on Untrusted Inputs in a Security Decision | Merged |
| [#16597](https://github.com/keycloak/keycloak/issues/16597) Policy evaluation tool ignores the selected client | [#55494](https://github.com/keycloak/keycloak/pull/55494) | [CWE-684](https://cwe.mitre.org/data/definitions/684.html) Incorrect Provision of Specified Functionality | PR open, waiting on maintainer review |

---

## #20008: UMA policy creation missing from the admin audit log

**Status:** [PR #53316](https://github.com/keycloak/keycloak/pull/53316) was merged into Keycloak `main` on October 12, 2026.

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

**Status:** [PR #53724](https://github.com/keycloak/keycloak/pull/53724) is open and waiting on a maintainer review. On November 14, 2026 it was rebased onto current `main` and squashed to one commit (the only conflict was an admin console message file), so CI now runs on it. Both of Copilot's review points are handled: the first led to the third test below, and its suggestion to move the tests to the new test framework was answered on the PR, since Keycloak's rules for the deprecated testsuite allow adding test cases to an existing test as part of a bug fix. Copilot's review after the rebase recommends approval with no open findings. As of November 24, 2026, no maintainer has reviewed it yet.

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

---

## #53325: AuthZen caller can impersonate another client towards client policies

**Status:** [PR #55186](https://github.com/keycloak/keycloak/pull/55186) was merged into Keycloak `main` on November 23, 2026, and the issue is tagged for the Keycloak 27.0.0 release.

### Working with the maintainers

This one went through the maintainers at every step, not just at review time:

1. **Claim with a plan.** When we asked to take the issue, we posted the planned fix on it (drop request-supplied `kc.*` keys, set `kc.client.id` from the caller's token) and asked whether setting the client id was what they wanted, or whether they'd rather only strip.
2. **Agree on the approach first.** vramik, who had already self-assigned the issue, handed it to us and answered the design question before any code was written: strip *and* set. He also pointed to where it should go (filter the raw request maps in `evaluateSingle()`, before `convertContext()`) and why stripping alone would break client policies. The PR description says it follows that agreed approach.
3. **Review and merge.** vramik approved the PR on November 18, saying it followed the agreed approach and calling out the tests for reserved keys the server never overwrites (the gap Copilot's review found, below). He then asked pedroigor to review; pedroigor approved and merged it on November 23.

### The issue

AuthZen is a standard API that Keycloak supports as a technology preview: an application asks Keycloak whether a subject can perform an action on a resource, and Keycloak answers yes or no using its authorization policies. A request can include a `context` map and `resource.properties`, extra attributes that policies can use. Keycloak copied both maps into the policy evaluation context as-is, including keys in the `kc.` namespace, which Keycloak uses for values it sets itself and policies trust. A client policy decides by checking `kc.client.id`, the client that made the request. So a caller could send `"kc.client.id": "other-client"` and be treated as that client: a decision that should have been `false` flipped to `true`.

**Why it matters for assurance:** this is [CWE-807: Reliance on Untrusted Inputs in a Security Decision](https://cwe.mitre.org/data/definitions/807.html). The policy engine trusted a value the caller controlled, so any caller could impersonate any client towards client-based policies. It's also inside our scoped subsystem (authorization).

### Where it lives in the source

| File | Method | What it does |
|---|---|---|
| [`authzen/services/.../authzen/AuthZenResource.java`](https://github.com/keycloak/keycloak/blob/main/authzen/services/src/main/java/org/keycloak/authorization/authzen/AuthZenResource.java) | `evaluateSingle()` | Converts the request's `resource.properties` and `context` into claims for policy evaluation |
| [`services/.../common/DefaultEvaluationContext.java`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/authorization/common/DefaultEvaluationContext.java) | `getBaseAttributes()` | Starts from those claims, then overwrites the `kc.*` values the server owns (time, network, realm). **Only sets `kc.client.id` for a `KeycloakIdentity`** |
| [`authz/policy/common/.../client/ClientPolicyProvider.java`](https://github.com/keycloak/keycloak/blob/main/authz/policy/common/src/main/java/org/keycloak/authorization/policy/provider/client/ClientPolicyProvider.java) | `evaluate()` | Grants when `kc.client.id` matches one of the policy's clients |

### Thought process

1. **Reproduce it.** The issue came with a reproduction: a client policy that requires another client denies the real caller, but grants once the request adds `kc.client.id` for that other client, through either `context` or `resource.properties`.
2. **Follow the request.** `evaluateSingle()` copies both maps into the claims. `getBaseAttributes()` starts from those claims and overwrites some `kc.*` keys, but it only sets `kc.client.id` when the identity is a `KeycloakIdentity`, which is the normal token flow. AuthZen builds a `UserModelIdentity` or `ClientModelIdentity` instead, so the caller's value was never overwritten.
3. **Strip and set, not strip-only.** Dropping `kc.*` keys from the request alone would leave `kc.client.id` empty, so client policies would never match, even for the real caller. So the fix also sets `kc.client.id` from the token's `azp`, the client that actually authenticated. We proposed this on the issue, and the maintainer agreed before any code was written.
4. **Reserve the whole prefix.** Filtering every `kc.` key, not just `kc.client.id`, also closes `kc.client.user_agent` (the server only sets it when the request has a `User-Agent` header) and covers any `kc.*` attribute added later.
5. **Leave `subject.properties` alone.** Those feed the subject's identity attributes, not the evaluation context, and are meant to carry caller-supplied values.

### The fix

In `evaluateSingle()`, filter both maps before they become claims, then set the client id from the caller's token:

```java
Map<String, List<String>> claims = new HashMap<>();
if (request.resource().properties() != null) {
    convertContext(withoutReservedAttributes(request.resource().properties()), claims);
}
if (request.context() != null) {
    convertContext(withoutReservedAttributes(request.context()), claims);
}
claims.put(EvaluationContext.CLIENT_ID_ATTRIBUTE, List.of(token.getIssuedFor()));
```

`withoutReservedAttributes()` copies the map and drops every key starting with `kc.`. It works on a copy because the batch endpoint shares one context map across all its items. The AuthZen guide now also notes that `kc.*` keys are reserved.

### How we proved it

Seven integration tests in `AuthZenEvaluationTest`:

| Test | What it checks |
|---|---|
| `testClientPolicyDeniesOtherClient` | Baseline: a policy for another client denies the real caller |
| `testContextCannotOverrideClientId` | Spoofing `kc.client.id` through `context` is still denied |
| `testResourcePropertiesCannotOverrideClientId` | Spoofing it through `resource.properties` is still denied |
| `testEvaluationsContextCannotOverrideClientId` | The batch endpoint, with the spoof in the shared default context |
| `testClientPolicyMatchesAuthenticatedClient` | A policy for the real caller now matches (before the fix it never did, because `kc.client.id` was never set) |
| `testContextCannotSupplyReservedAttribute` | A reserved key the server never sets, `kc.test.reserved`, sent through `context`, is ignored |
| `testResourcePropertiesCannotSupplyReservedAttribute` | The same key through `resource.properties` is ignored |

With the whole fix removed, four of the first five fail (all but the baseline). Copilot's automated review then pointed out a gap: because the server sets `kc.client.id` after filtering, those tests would still pass if the filter were deleted. The last two tests close it. They send `kc.test.reserved` and use a small test-only policy, `ReservedAttributePolicyProvider`, that grants only if it sees that key. With only the filter disabled, exactly those two fail. All 123 AuthZen tests pass with the fix.

### Takeaway

A fix with two layers needs a test for each layer. Our first tests went through the server-side overwrite, so they would have passed with the filter deleted. The way to prove each part is to remove it on its own and check that the right tests fail. And values in a reserved namespace should come from the server, never from the caller.

On process: settling the design with the maintainer on the issue, before writing code, is why the review was quick. The PR didn't have to argue for its approach, only show that it followed the agreed one.

---

## #16597: Policy evaluation tool ignores the selected client

**Status:** [PR #55494](https://github.com/keycloak/keycloak/pull/55494) is open and waiting on maintainer review.

### The issue

The admin console's **Authorization > Evaluate** tab lets an admin test policies before relying on them: pick a client and a user, run the evaluation, and see which permissions are granted. Picking a client other than the resource server had no effect. The evaluation always ran as the resource server's own client, which shows up as the `azp` claim in the token under "Show authorization data". The issue was reported against Keycloak 20 and was still reproducible on current `main`. pedroigor confirmed it on the issue and said the `azp` claim should match the client the admin chose.

**Why it matters for assurance:** this is [CWE-684: Incorrect Provision of Specified Functionality](https://cwe.mitre.org/data/definitions/684.html). The evaluation tool is how an admin checks that a policy does what they think it does. If it silently tests a different client than the one selected, a client policy can look like it denies (or grants) access when it wouldn't for the real client, and the admin trusts a wrong result. It's inside our scoped subsystem (authorization).

### Where it lives in the source

| File | Function or method | What it does |
|---|---|---|
| [`admin-ui/.../authorization/AuthorizationEvaluate.tsx`](https://github.com/keycloak/keycloak/blob/main/js/apps/admin-ui/src/clients/authorization/AuthorizationEvaluate.tsx) | `evaluate()` | Builds the evaluation request from the form, including `clientId` |
| [`admin-ui/.../components/client/ClientSelect.tsx`](https://github.com/keycloak/keycloak/blob/main/js/apps/admin-ui/src/components/client/ClientSelect.tsx) | `ClientSelect` | The client picker. Its `clientKey` prop decides what it stores in the form: the `clientId` name (default) or the internal `id` |
| [`services/.../admin/PolicyEvaluationService.java`](https://github.com/keycloak/keycloak/blob/main/services/src/main/java/org/keycloak/authorization/admin/PolicyEvaluationService.java) | `createIdentity()` | Looks the client up with `getClientById()` and issues the evaluation token for it. **Falls back to the resource server's client when `clientId` is missing** |

### Thought process

1. **Reproduce it.** On a resource server, evaluate with another client selected, then check the token: `azp` is still the resource server. The request payload had no `clientId` at all.
2. **Rule out the server.** Calling the evaluation endpoint directly with a `clientId` produced a token for that client, so the server handled it correctly. The bug was in what the admin console sent.
3. **Follow the form value.** The picker stored the selected client's `clientId` string, its default. The request then read `formValues.client.id`, an `id` property on a string, which is `undefined`. So `clientId` was dropped from the request, and the server fell back to the resource server.
4. **Send what the server expects.** The server looks the client up by internal id, so the picker should store the id, not the name. The client policy form already uses the same picker with `clientKey="id"`, so this follows an existing pattern. We also checked the sibling Permissions evaluation tab, which already stores ids, so the bug was only here.

### The fix

Two changes in `AuthorizationEvaluate.tsx`. The picker stores the client's internal id, and the request sends it:

```tsx
<ClientSelect
  name="client"
  label="client"
  helpText={"clientHelp"}
  clientKey="id"
  defaultValue={client.id}
/>
```

```tsx
clientId: formValues.client || undefined,
```

If the admin clears the picker, `clientId` is left out, so the server keeps its old fallback to the resource server. The picker still shows the client's name, because it looks ids up with `clients.findOne()`.

### How we proved it

Two Playwright tests in the admin console's `authorization.spec.ts`. They set up a scope permission whose only policy is a client policy that permits a second client:

| Test | What it checks |
|---|---|
| `Should evaluate as the resource server client by default` | With the default client selected, the permission is denied |
| `Should evaluate as the selected client` | After selecting the second client, the permission is granted |

With the old code, the second test fails: the result is Deny where it expects Permit, which is the bug. With the fix, all 22 tests in the client authorization spec pass, both locally against the Keycloak nightly and in Keycloak's CI.

### Takeaway

When a value is lost between a UI and an API, check the API on its own first. Calling the endpoint directly showed the server already did the right thing, which narrowed a long-standing issue down to one wrong property read in the admin console. A test that checks the *outcome* (Deny vs. Permit) rather than the request payload would also catch any other way the selection could get lost.
