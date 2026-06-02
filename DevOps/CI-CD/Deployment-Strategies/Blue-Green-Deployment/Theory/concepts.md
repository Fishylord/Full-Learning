# Blue-Green Deployment Theory

## What It Is

Blue-green deployment uses two production-capable environments:

- Blue: the current live environment.
- Green: the new release environment.

The new version is deployed to green while blue continues serving users. After green passes checks, traffic switches from blue to green.

## How It Works

1. Blue serves production traffic.
2. Green is created or updated with the new version.
3. Smoke tests and validation run against green.
4. Traffic is switched to green using a load balancer, DNS, service selector, or routing layer.
5. Blue is kept temporarily for rollback.
6. After confidence is high, blue is removed or becomes the idle environment for the next release.

## Why It Exists

Blue-green deployment reduces release risk by testing the new version before it receives production traffic. It also makes rollback fast: switch traffic back to the previous environment.

## When To Use

Use blue-green deployment for:

- Important production systems.
- Releases requiring fast rollback.
- Apps where duplicate infrastructure is acceptable.
- Systems where pre-traffic validation is valuable.
- Releases with moderate risk but clear health checks.

## When To Avoid

Avoid or be careful with blue-green when:

- Infrastructure cost must stay very low.
- Database migrations are destructive.
- Traffic switching is slow or unreliable.
- The app has state tied to one environment.
- Sessions cannot survive environment switch.

## Traffic Switching Methods

Common ways to switch traffic:

- Load balancer target groups.
- Kubernetes service selectors.
- Ingress routing.
- DNS records.
- API gateway routes.
- Service mesh routing.

Load balancer switching is usually faster and more controlled than DNS because DNS caching can delay cutover.

## Database Challenge

Blue-green is easy for stateless compute and harder for shared databases.

If blue and green use the same database, both versions must be compatible with the same schema. If green changes the database in a way that breaks blue, rollback is not safe.

Use expand-and-contract migrations for safer blue-green deployments.

## Example

An API runs behind an Application Load Balancer. Blue target group has version 1. Green target group has version 2.

The pipeline deploys version 2 to green, runs smoke tests, then changes the load balancer listener to forward traffic to green. If errors increase, the listener is switched back to blue.

## Pros

- Fast traffic rollback.
- New version can be tested before users see it.
- Clear separation between old and new environments.
- Good fit for immutable infrastructure.
- Reduces downtime when traffic switch is controlled.

## Cons

- Requires duplicate infrastructure.
- More expensive than rolling deployment.
- Shared database compatibility remains difficult.
- DNS switching can be slow.
- Operational setup is more complex.

## Short Summary

Blue-green deployment keeps old and new environments side by side, then switches traffic. It is strong for rollback and validation, but costs more and still needs database compatibility.

## Deep Technical Notes

### Environment Model

Blue-green deployment is not just "two folders" or "two servers." A real blue-green setup defines which resources are duplicated and which are shared.

Usually duplicated:

- Application compute.
- Load balancer target groups.
- Container services or Kubernetes workloads.
- Runtime config.
- App-level caches if needed.

Often shared:

- Database.
- Object storage.
- Message queues.
- DNS zone.
- Authentication provider.
- External third-party services.

The shared resources are where most complexity lives.

### Cutover Mechanisms

Traffic can move from blue to green using different layers.

Load balancer cutover:

- Fast and controlled.
- Common with AWS ALB target groups.
- Can support weighted traffic during transition.

Kubernetes service selector cutover:

- Service changes selector from blue pods to green pods.
- Fast inside cluster.
- Requires careful label design.

Ingress or gateway routing:

- Useful for path, header, or weighted routing.
- Common with Nginx ingress, Istio, Envoy, API Gateway.

DNS cutover:

- Simple in concept.
- Risky because DNS caching delays convergence.
- Better for coarse failover than precise release switching.

### Pre-Production Validation On Green

The value of blue-green is that green can be tested before users hit it.

Useful green checks:

- App starts with production-like config.
- Health endpoint is healthy.
- Version endpoint shows expected commit.
- Database connectivity works.
- Main read path works.
- Main write path works with test data.
- Queue consumers can start safely.
- Logs and metrics appear.
- Security headers and TLS are correct.

Do not run destructive tests against shared production data unless the test is isolated.

### Database Compatibility

The most important technical issue is the database.

If blue and green share one database, then:

- Blue must survive any migration done for green.
- Green must tolerate data created by blue.
- Rollback to blue must still work after green receives traffic.

Unsafe blue-green example:

1. Green migration drops a column.
2. Green starts successfully.
3. Traffic switches to green.
4. Green has an issue.
5. Traffic switches back to blue.
6. Blue crashes because the column is gone.

Safe approach:

- Use expand-and-contract migrations.
- Avoid destructive changes before cutover.
- Keep old and new code compatible during rollback window.
- Delay cleanup until after rollback window expires.

### Session and State Handling

Blue-green cutover can break users if session state is stored inside the old environment.

Safer session designs:

- Store sessions in shared Redis or database.
- Use stateless signed tokens.
- Keep sticky sessions only if rollback behavior is understood.
- Ensure green can validate tokens issued by blue.

For WebSockets, clients may remain connected to blue after cutover unless connections are drained or closed deliberately.

### Queue and Worker Concerns

If both blue and green workers run at the same time, duplicate processing or mixed behavior can occur.

Decide explicitly:

- Should green workers start before cutover?
- Should blue workers stop before green workers start?
- Can both consume the same queue?
- Are messages versioned?
- Are workers idempotent?

For some systems, application traffic can switch blue-green while background workers use a separate controlled rollout.

### Rollback Semantics

Blue-green rollback is fast only for traffic. It does not reverse:

- Database writes.
- Sent emails.
- Published events.
- Payments.
- External API calls.
- Cache mutations.

Therefore rollback should be understood as "route users back to previous compute," not "undo every effect of the release."

### Cost and Capacity Planning

Blue-green may require double compute capacity during release. In cloud environments, this affects:

- EC2 or container compute cost.
- Database connection limits.
- Load balancer target limits.
- IP address usage.
- Cluster capacity.
- Autoscaling limits.

Green should be production-capable before traffic moves. A half-sized green environment gives less confidence and can fail at cutover.

### Example: AWS ALB Target Groups

A common AWS design:

- Blue ECS service registered in blue target group.
- Green ECS service registered in green target group.
- ALB listener forwards 100 percent to blue.
- Pipeline deploys green.
- Health checks and smoke tests run.
- Listener rule changes to 100 percent green.
- Blue remains alive for rollback window.

This can also be made weighted, for example 90/10, before full cutover.

### Practical Lab Ideas

- Implement blue-green with two local containers and an Nginx upstream switch.
- Implement blue-green with Kubernetes labels and Service selectors.
- Simulate a breaking database migration and explain why rollback fails.
- Add shared Redis sessions and test whether users remain logged in after cutover.
- Add a rollback window and define when old environment can be destroyed.
