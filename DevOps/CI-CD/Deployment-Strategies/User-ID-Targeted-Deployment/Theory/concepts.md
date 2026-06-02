# User-ID Targeted Deployment Theory

## What It Is

User-ID targeted deployment releases a feature or version to specific users, tenants, groups, or cohorts instead of everyone.

This is usually implemented with feature flags, routing rules, or experiment systems.

## How It Works

1. The application receives a request.
2. The system identifies the user, tenant, account, or cohort.
3. A flag or routing rule decides whether that identity receives the new behavior.
4. The user consistently receives the same experience.
5. The rollout expands to more users if health and feedback are good.

## Targeting Examples

Common targeting rules:

- Specific user IDs.
- Internal employee accounts.
- Beta testers.
- One customer tenant.
- Users in one country.
- Users on one plan type.
- 5 percent of users by stable user ID hash.

## Stable Assignment

Stable assignment is critical. A user should not randomly switch between old and new behavior on each request.

For percentage rollout, use a deterministic hash of user ID or tenant ID. For example, hash the user ID into a number from 1 to 100 and enable the feature for users in the selected range.

## Why It Exists

Some releases should be exposed to controlled audiences first. This lets teams validate behavior with friendly users, internal staff, beta customers, or one tenant before full release.

## When To Use

Use user-ID targeted deployment for:

- Beta features.
- Internal dogfooding.
- SaaS tenant-by-tenant rollout.
- Risky UI or workflow changes.
- Gradual product launch.
- Permission-based features.
- Customer-specific enablement.

## When To Avoid

Avoid or be careful with user targeting when:

- The feature affects shared global state.
- Users collaborate and must see the same behavior.
- Support teams cannot tell which version a user sees.
- Authorization is confused with feature rollout.
- The flag is only enforced on the client for sensitive behavior.

## Security Note

Feature targeting is not authorization. A feature flag can decide whether to show or enable a feature, but permission checks must still be enforced server-side.

Do not rely on hiding a button in the UI as security.

## Operational Risks

User-targeted rollout adds complexity:

- More code paths to test.
- More combinations of flags.
- Support must know user flag state.
- Analytics must segment users correctly.
- Stale flags must be removed.

## Example

A SaaS company releases a new billing dashboard. It enables the feature first for internal employees, then for 3 beta tenants, then for 10 percent of all users by user ID hash, then for everyone.

## Pros

- Precise rollout control.
- Good for beta testing.
- Supports internal dogfooding.
- Allows tenant-by-tenant release.
- Can disable feature without redeploy.

## Cons

- Adds conditional logic.
- Creates feature flag debt.
- Can confuse support and QA.
- Needs stable user identity.
- Must not replace authorization checks.

## Short Summary

User-ID targeted deployment releases behavior to chosen users or cohorts. It is powerful for controlled rollout, but it requires stable assignment, flag discipline, and clear support visibility.

## Deep Technical Notes

### Evaluation Architecture

User-targeted deployment usually depends on a flag or routing evaluation system.

Typical evaluation inputs:

- User ID.
- Tenant ID.
- Organization ID.
- Region.
- Plan type.
- Role.
- Device type.
- App version.
- Request headers.
- Environment.

The evaluator returns a decision such as enabled, disabled, variant A, variant B, or route to version 2.

Evaluation can happen:

- Server-side in application code.
- Client-side in frontend or mobile app.
- At the edge.
- In an API gateway.
- In a service mesh.

For sensitive behavior, server-side enforcement is required.

### Deterministic Bucketing

Percentage rollout should use deterministic bucketing.

Basic idea:

1. Combine flag key and user ID.
2. Hash the result.
3. Convert hash to range 0-99 or 0-9999.
4. Enable if bucket is inside rollout range.

This ensures:

- Same user gets same result.
- Different flags can assign independently.
- Percentage can increase without reshuffling everyone.

If the hash does not include the flag key, unrelated flags may assign the same users too often.

### Identity Problems

Targeting depends on stable identity. Many systems do not have it at every point.

Examples:

- Anonymous users before login.
- Multiple users in one household.
- One user across multiple devices.
- Service accounts.
- Shared admin accounts.
- Users moving between tenants.

Decide whether rollout should be based on user ID, account ID, tenant ID, session ID, or device ID.

For SaaS products, tenant-level targeting is often safer than user-level targeting because users in the same company need a consistent experience.

### Client-Side vs Server-Side Flags

Client-side targeting is useful for UI changes, but it can expose flag names, targeting behavior, or hidden UI code.

Server-side targeting is better for:

- Permissions.
- Billing behavior.
- Data access.
- Security-sensitive features.
- Backend workflows.

Client-side flags should be treated as presentation controls, not security controls.

### Support and Observability

Support teams need to know what a user is seeing.

Useful operational data:

- Active flags for user.
- Assigned variants.
- Rollout cohort.
- App version.
- Tenant ring.
- Last flag evaluation time.

Logs and traces should include flag decisions for important requests, but avoid leaking sensitive targeting data.

### Data Consistency

Targeted rollout can create mixed data behavior.

Example:

- Beta users create records with new fields.
- Non-beta users view the same records.
- Old UI cannot render the new shape.

Mitigations:

- Make data model backward-compatible.
- Hide new fields from old views.
- Use schema versions.
- Roll out readers before writers.
- Target by tenant if shared records exist.

### Rollout Expansion

A practical rollout plan:

1. Specific developer user IDs.
2. Internal staff.
3. One test tenant.
4. Beta tenants.
5. 5 percent stable hash cohort.
6. 25 percent.
7. All users.

Each expansion should have monitoring and rollback criteria.

### Practical Lab Ideas

- Implement stable percentage rollout using a hash of user ID and flag key.
- Add support endpoint showing active flags for a user.
- Compare user-level and tenant-level targeting for shared records.
- Demonstrate why random assignment per request is broken.
