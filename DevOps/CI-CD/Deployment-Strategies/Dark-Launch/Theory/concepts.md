# Dark Launch Theory

## What It Is

Dark launch deploys functionality into production without making it visible to users.

The new code may run internally, collect data, warm systems, or process requests, but users do not see the new result yet.

## How It Works

1. New functionality is deployed behind a hidden path or flag.
2. Production traffic or production-like events exercise the new code.
3. The output is logged, compared, or ignored.
4. Metrics are watched for resource usage and correctness.
5. The feature is later exposed through a normal rollout strategy.

## Why It Exists

Some systems need production validation before user exposure. Development and staging cannot always reproduce real data volume, traffic patterns, user behavior, or third-party integration behavior.

Dark launch gives early production learning while keeping the user experience unchanged.

## When To Use

Use dark launch for:

- New recommendation systems.
- Search ranking changes.
- Backend calculations.
- New event processing.
- Cache warming.
- Data pipeline validation.
- New API implementation running internally.

## When To Avoid

Avoid or be careful when:

- The hidden code has side effects.
- The code sends emails, payments, or notifications.
- The code writes production data without control.
- The extra workload could overload systems.
- Users might see inconsistent data.

## Side Effect Control

Dark-launched code should usually be read-only or isolated.

If it must write data:

- Write to separate tables.
- Mark records as test or shadow.
- Block external effects.
- Use idempotency.
- Add a kill switch.

## Observability

Dark launches need monitoring because hidden code can still break production.

Watch:

- CPU and memory.
- Database load.
- Queue depth.
- Error logs.
- External API calls.
- Latency impact on visible features.

## Example

A streaming service builds a new recommendation model. The model runs for real users in the background and logs recommendations, but the UI still shows the old recommendations. Engineers compare model quality and latency before exposing it.

## Pros

- Tests production behavior before visible release.
- Useful for backend validation.
- Reduces surprise at launch time.
- Helps warm caches or build data.
- Can collect real-world metrics.

## Cons

- Hidden code can still cause failures.
- Extra production load.
- Side effects must be controlled.
- Users do not validate experience yet.
- Requires clear kill switch.

## Short Summary

Dark launch runs new functionality invisibly in production. It is useful for backend validation, but hidden code still needs monitoring and side-effect control.

## Deep Technical Notes

### Execution Models

Dark launch can happen in several ways:

- Hidden code path runs in the same request but output is ignored.
- Background job computes new result asynchronously.
- New service consumes copied events.
- New data pipeline runs beside the old pipeline.
- New model inference runs but old model result is shown.

Each model has different risk. Same-request dark launch can affect user latency. Async dark launch reduces user impact but may be harder to compare directly.

### Output Handling

Dark launch output should be handled deliberately.

Options:

- Ignore output but record metrics.
- Store output in a separate table.
- Compare old and new output.
- Send output to analytics only.
- Use output to warm cache but not serve users.

Never let dark-launched output accidentally become user-visible before the release decision.

### Latency Budget

If dark launch code runs during a user request, it consumes latency budget.

Risk:

- Old request path was 80 ms.
- Dark launch adds 100 ms.
- Users now experience 180 ms even though feature is hidden.

Mitigations:

- Run dark code asynchronously.
- Use timeouts.
- Sample only a fraction of traffic.
- Use circuit breakers.
- Record overhead separately.

### Resource Isolation

Hidden production code still uses resources.

Watch:

- Database connections.
- CPU.
- Memory.
- Queue throughput.
- External API rate limits.
- Cache capacity.
- Thread pools.

Dark launches should have independent limits where possible.

### Side Effect Classification

Classify the dark-launched operation:

- Pure read: safest.
- Read plus compute: usually safe.
- Write to isolated store: controlled.
- Write to shared production store: risky.
- External side effect: high risk.

External side effects include emails, payments, SMS, webhooks, partner API calls, and event publication.

### Data Validation

Dark launch is often used to compare old and new calculations.

Useful validation:

- Difference rate.
- Maximum difference.
- Missing output count.
- Invalid output count.
- Latency distribution.
- Failure count by input type.

For ML systems, compare prediction distribution, not only exact output.

### Kill Switch

Dark launches need a kill switch because users may not see the feature, but production can still be harmed.

Kill switch should disable:

- Execution.
- Event consumption.
- External calls.
- Background jobs.
- Cache writes if needed.

### Practical Lab Ideas

- Run a new calculation beside an old one and log differences.
- Add sampling so only 10 percent of requests execute dark code.
- Add timeout and circuit breaker around dark-launched logic.
- Measure latency overhead when dark code runs synchronously.
