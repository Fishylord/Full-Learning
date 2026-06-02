# Deployment Strategies

This folder contains theory notes for the main CI/CD deployment strategies. Each strategy has its own `Theory/concepts.md` file and a `Practical/Code` folder for labs, pipeline examples, manifests, scripts, or demos.

## Suggested Learning Order

1. `Recreate-Deployment`: simplest model; understand downtime and cutover risk.
2. `Rolling-Deployment`: common default for replicated services.
3. `Blue-Green-Deployment`: learn environment switching and rollback.
4. `Canary-Deployment`: learn traffic splitting and metric-based confidence.
5. `Feature-Flag-Deployment`: learn how deployment and release can be separated.
6. `User-ID-Targeted-Deployment`: learn cohort-based rollout.
7. `Dark-Launch`: learn hidden production validation.
8. `Shadow-Deployment`: learn copied production traffic testing.
9. `AB-Testing-Deployment`: learn product experimentation.
10. `Ring-Deployment`: learn ordered exposure groups.
11. `Immutable-Deployment`: learn replacement over in-place mutation.
12. `Progressive-Delivery`: learn how the above combine into controlled, evidence-based rollout.

## Quick Comparison

| Strategy | Main Purpose | Main Risk |
| --- | --- | --- |
| Recreate | Simple replacement | Downtime |
| Rolling | Gradual instance replacement | Old and new versions overlap |
| Blue-green | Fast traffic switch and rollback | Duplicate infrastructure and database compatibility |
| Canary | Limited production exposure | Requires strong metrics |
| Feature flag | Separate deploy from release | Flag debt and test combinations |
| User-ID targeted | Controlled user or tenant rollout | Support and cohort complexity |
| Dark launch | Hidden production validation | Hidden side effects |
| Shadow | Test new version with copied traffic | Duplicate side effects if not isolated |
| A/B testing | Product outcome comparison | Misleading results if experiment design is weak |
| Ring | Ordered rollout by risk group | Slow rollout and version fragmentation |
| Immutable | Replace units instead of patching | Requires automation |
| Progressive delivery | Evidence-based gradual release | Operational complexity |
