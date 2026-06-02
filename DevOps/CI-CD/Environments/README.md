# CI/CD Environments

CI/CD environments are the places where code is built, tested, reviewed, released, and operated. Each environment has a different purpose and risk level.

## Environment Flow

Typical flow:

1. `Development`: engineer builds and debugs quickly.
2. `Testing`: automated and manual validation.
3. `Preview-Environments`: pull request or branch-specific review.
4. `Staging`: production-like release rehearsal.
5. `Production`: real users and real data.
6. `Ephemeral-Environments`: temporary isolated environments used across testing, preview, demos, and experiments.

## Quick Comparison

| Environment | Main Purpose | Main Risk |
| --- | --- | --- |
| Development | Fast coding and debugging | Different from production |
| Testing | Validate correctness | Flaky tests or dirty shared state |
| Preview | Review running pull requests | Secret exposure and cleanup failure |
| Staging | Final release rehearsal | False confidence if not production-like |
| Production | Serve real users | Customer impact from failures |
| Ephemeral | Temporary isolated validation | Cost and cleanup issues |

## Rule of Thumb

Development optimizes for speed. Testing optimizes for repeatable validation. Preview optimizes for review. Staging optimizes for release confidence. Production optimizes for safety and reliability. Ephemeral environments optimize for isolation and cleanup.
