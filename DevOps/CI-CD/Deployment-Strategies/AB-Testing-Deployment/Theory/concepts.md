# A/B Testing Deployment Theory

## What It Is

A/B testing deployment exposes different user groups to different variants so the team can compare outcomes.

It is not mainly a safety strategy. It is an experiment strategy. Canary asks, "Is this release healthy?" A/B testing asks, "Which version performs better?"

## How It Works

1. Define a hypothesis.
2. Choose variants, such as A and B.
3. Assign users to stable cohorts.
4. Expose each cohort to its variant.
5. Measure success and guardrail metrics.
6. Decide whether to keep, change, or remove the variant.

## Common Examples

Examples:

- Checkout button text.
- Onboarding flow.
- Search ranking.
- Recommendation layout.
- Pricing page.
- Notification timing.
- Signup form design.

## Stable Cohorts

Users should stay in the same experiment group. If a user randomly changes from A to B between visits, results become noisy and the user experience becomes confusing.

Cohorts are often assigned using a stable hash of user ID.

## Metrics

A/B testing requires predefined metrics.

Success metrics:

- Conversion rate.
- Signup completion.
- Purchase rate.
- Retention.
- Click-through rate.
- Revenue per user.

Guardrail metrics:

- Error rate.
- Latency.
- Support tickets.
- Cancellation rate.
- Payment failures.

## When To Use

Use A/B testing for:

- Product experiments.
- UI or UX decisions.
- Growth experiments.
- Recommendation changes.
- Pricing or funnel changes.

## When To Avoid

Avoid or be careful when:

- Safety is the main question.
- Sample size is too small.
- Users collaborate and need consistent experience.
- Ethical or legal constraints apply.
- The metric can be gamed or misread.

## Difference From Canary

Canary deployment is operational. It checks health and limits risk.

A/B testing is experimental. It compares outcomes and supports product decisions.

A release can use both: canary first for safety, then A/B test for product effectiveness.

## Example

An ecommerce site tests two checkout page designs. Half of users see version A, half see version B. The team measures purchase completion and payment error rate. If version B increases conversion without hurting guardrail metrics, it becomes the default.

## Pros

- Measures real user behavior.
- Helps product decisions.
- Reduces opinion-based debates.
- Works well with feature flags.
- Can reveal unexpected user preferences.

## Cons

- Requires statistical care.
- Needs stable assignment.
- Can create inconsistent user experiences.
- Results can be misleading with bad metrics.
- Usually slower than simple rollout.

## Short Summary

A/B testing deployment compares variants across stable user groups. It is for product learning, not just operational safety.

## Deep Technical Notes

### Experiment Design

A/B testing is valid only if the experiment is designed well.

You need:

- Hypothesis.
- Primary metric.
- Guardrail metrics.
- Assignment unit.
- Sample size expectation.
- Experiment duration.
- Exclusion criteria.
- Decision rule.

Bad example:

- "Try a new checkout page and see what happens."

Better example:

- "Variant B reduces checkout abandonment for logged-in users. Primary metric is purchase completion. Guardrails are payment error rate, p95 latency, and support tickets."

### Assignment Unit

Choose the unit that gets assigned to a variant:

- User.
- Account.
- Tenant.
- Device.
- Session.
- Region.

The assignment unit must match the product behavior.

For SaaS collaboration tools, tenant-level assignment may be better than user-level assignment because users in one company interact with shared resources.

### Randomization and Bucketing

Users should be randomly but stably assigned.

Stable assignment prevents users switching variants during the experiment. Randomization reduces bias.

Implementation usually uses a hash of:

- Experiment key.
- User or tenant ID.

Then the hash bucket maps to variant A or B.

### Guardrail Metrics

Primary metrics tell whether the experiment is successful. Guardrails tell whether it is causing harm.

Examples:

- Primary: conversion rate.
- Guardrail: payment failures.
- Primary: click-through rate.
- Guardrail: page latency.
- Primary: signup completion.
- Guardrail: account deletion rate.

Never optimize one metric blindly while damaging important safety or business metrics.

### Statistical Pitfalls

Common mistakes:

- Ending the test as soon as the result looks good.
- Running too many experiments that interact.
- Using too small a sample size.
- Ignoring novelty effects.
- Assigning by request instead of user.
- Changing the metric during the test.
- Forgetting seasonality or day-of-week effects.

Repeatedly checking results and stopping early can inflate false positives unless the experiment design accounts for it.

### Interaction With Deployment

A/B testing should usually happen after basic operational safety is proven.

A practical sequence:

1. Deploy behind flag.
2. Enable internally.
3. Canary for safety.
4. Start A/B experiment.
5. Analyze result.
6. Roll out winning variant.
7. Remove losing code.

Do not use A/B testing as the first safety check for a risky backend release.

### Data Collection

Experiment data should include:

- User or tenant assignment.
- Variant.
- Exposure event.
- Conversion event.
- Timestamp.
- App version.
- Relevant segmentation fields.

An exposure event matters. A user assigned to variant B but never seeing the feature should not always count the same as a user who actually experienced it.

### Practical Lab Ideas

- Implement stable A/B assignment by hashing user ID and experiment key.
- Track exposure and conversion events.
- Add guardrail metrics.
- Simulate biased assignment and show how results become misleading.
