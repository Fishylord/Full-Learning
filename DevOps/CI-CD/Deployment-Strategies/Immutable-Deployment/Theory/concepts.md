# Immutable Deployment Theory

## What It Is

Immutable deployment replaces runtime units instead of modifying them in place.

Rather than SSH into a server and patch it, the team builds a new image, container, virtual machine, or environment and deploys that new unit.

## How It Works

1. Build a versioned artifact.
2. Create new runtime units from that artifact.
3. Validate the new units.
4. Shift traffic to the new units.
5. Remove old units after confidence or keep them briefly for rollback.

## Why It Exists

Mutable infrastructure drifts over time. Manual changes, patched packages, hidden config edits, and one-off fixes make servers different from each other.

Immutable deployment reduces drift because each release starts from a known artifact.

## Common Immutable Units

Examples:

- Docker images.
- AMIs.
- VM images.
- Kubernetes pods.
- Auto Scaling Group launch templates.
- Serverless function versions.

## When To Use

Use immutable deployment for:

- Containerized apps.
- Cloud infrastructure.
- Auto Scaling Groups.
- Blue-green deployment.
- Reproducible production systems.
- Teams practicing Infrastructure as Code.

## When To Avoid

Avoid or be careful when:

- Provisioning new units is too slow.
- The system has hard-to-move local state.
- Artifact creation is not reliable.
- Infrastructure automation is immature.
- Emergency patching is common and undocumented.

## Mutable vs Immutable

Mutable deployment changes existing servers. Immutable deployment replaces them.

Mutable example: SSH into server, install package, restart process.

Immutable example: build new image, create new instance, route traffic to it, remove old instance.

## Example

An app runs on EC2 Auto Scaling Groups. The team builds a new AMI for each release. Deployment creates a new launch template version and rolls instances so all new EC2 instances come from the new AMI.

## Pros

- Reduces configuration drift.
- Improves reproducibility.
- Works well with rollback.
- Supports clean audit trails.
- Pairs well with IaC and containers.

## Cons

- Requires strong automation.
- New unit provisioning can be slower.
- Local state is difficult.
- Artifact build failures block releases.
- More infrastructure lifecycle management.

## Short Summary

Immutable deployment replaces runtime units instead of modifying them. It improves reproducibility and reduces drift, but requires reliable automation.

## Deep Technical Notes

### Artifact Immutability

Immutable deployment starts with immutable artifacts. A release should point to an exact artifact that will not change.

Examples:

- Container image digest.
- AMI ID.
- Lambda version.
- Build artifact checksum.
- Package version.

Avoid deploying mutable tags such as `latest` to production because the same tag can later point to different content.

### Golden Images vs Application Images

Some teams build golden base images with operating system hardening and standard agents, then build application images on top.

For VMs:

- Golden AMI contains OS patches, monitoring agent, baseline security.
- Application AMI contains app runtime and release code.

For containers:

- Base image contains runtime.
- Application image contains app code and dependencies.

The goal is reproducibility and controlled patching.

### Infrastructure Replacement

Immutable deployment usually pairs with infrastructure automation.

Examples:

- New Kubernetes ReplicaSet from new pod template.
- New EC2 Auto Scaling Group launch template version.
- New ECS task definition revision.
- New Lambda function version.
- New VM scale set image.

The old unit is not manually patched. It is replaced.

### Configuration and Secrets

Immutable artifacts should not contain environment-specific secrets.

Bad:

- Build image with production database password.

Better:

- Build generic image.
- Inject config and secrets at runtime through environment, secret manager, or mounted secret.

This allows the same artifact to move across environments.

### Drift Reduction

Configuration drift happens when running systems differ from declared state.

Mutable operations that cause drift:

- SSH patching.
- Manual package installs.
- Console changes.
- Local config edits.
- Emergency fixes not captured in code.

Immutable deployment reduces drift by making the declared artifact and infrastructure code the source of truth.

### State Problem

Immutable deployment works best for stateless runtime units. State must live outside replaceable compute.

State should be in:

- Database.
- Object storage.
- Managed cache.
- Durable queue.
- Attached volume with clear lifecycle.

If application state is stored on local disk, replacing the instance can lose data.

### Rollback

Rollback means route traffic or desired state back to the previous immutable artifact.

For containers, deploy previous image digest.

For EC2, set launch template back to previous AMI and replace instances.

For Lambda, route alias back to previous version.

Rollback still depends on database and external side-effect compatibility.

### Practical Lab Ideas

- Deploy using image digest instead of tag.
- Build a new AMI per release and roll an Auto Scaling Group.
- Show how manual SSH changes disappear after replacement.
- Store config outside the image and reuse the same artifact in two environments.
