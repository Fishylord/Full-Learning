# AWS Study Guide

This AWS section sits under `DevOps/AWS` because AWS is a broad cloud platform with its own compute, networking, storage, security, deployment, monitoring, and architecture topics.

Each topic follows the same structure:

- `Theory`: concepts, diagrams, tradeoffs, notes, and interview questions.
- `Practical/Code`: CLI commands, Terraform, CloudFormation, CDK, scripts, app examples, and lab work.

## Compute

### EC2

EC2 is AWS virtual machine compute. Learn instance types, AMIs, key pairs, user data, security groups, EBS attachment, instance profiles, placement groups, spot instances, reserved instances, and lifecycle states.

Practice by launching an EC2 instance, installing a web server through user data, attaching an IAM role, opening only HTTP/HTTPS/SSH as needed, and replacing manual setup with an infrastructure template.

### AMI

An AMI is a machine image used to launch EC2 instances. It captures the operating system, packages, configuration, and sometimes application dependencies. AMIs are useful for repeatable server builds, faster scaling, and controlled releases.

Practice by creating a custom AMI from a configured instance, launching a new instance from it, and comparing this workflow with Docker images.

### Auto Scaling

Auto Scaling adjusts compute capacity based on desired count, metrics, schedules, or health. It is commonly paired with load balancers and launch templates.

Practice by creating an Auto Scaling Group with a launch template, health checks, target tracking policy, and rolling instance replacement.

### Lambda

Lambda is serverless compute for event-driven workloads. Learn cold starts, concurrency, timeouts, memory sizing, IAM execution roles, retries, event source mappings, and logs.

Practice by triggering Lambda from SQS, S3, and EventBridge. Add idempotency and structured logging.

### Elastic Beanstalk

Elastic Beanstalk is a managed application deployment service. It abstracts EC2, load balancing, scaling, and deployment configuration.

Practice by deploying a small web app, changing environment variables, rolling back a version, and inspecting the AWS resources it creates.

## Containers

### EKS

EKS is managed Kubernetes on AWS. Learn clusters, node groups, Fargate profiles, IAM Roles for Service Accounts, ingress controllers, load balancers, cluster autoscaler, EBS CSI, and observability.

Practice by deploying a containerized API to EKS, exposing it through an AWS load balancer, and giving the pod access to one AWS service using IRSA.

### ECS

ECS is AWS container orchestration. Learn clusters, task definitions, services, capacity providers, Fargate, EC2 launch type, service discovery, task roles, and deployment strategies.

Practice by running a web service on ECS Fargate behind an Application Load Balancer.

### ECR

ECR stores Docker and OCI images. Learn repositories, image tags, lifecycle policies, vulnerability scanning, authentication, and cross-account access.

Practice by pushing an image to ECR and deploying that image to ECS or EKS.

## Networking

### Route 53

Route 53 is AWS DNS and domain routing. Learn hosted zones, record types, alias records, routing policies, health checks, private hosted zones, and domain registration.

Practice by creating DNS records for an application load balancer, adding a health check, and testing failover routing.

### VPC Management

A VPC is an isolated cloud network. Learn CIDR blocks, subnets, route tables, gateways, security groups, NACLs, endpoints, peering, flow logs, and network segmentation.

Practice by designing a VPC with public and private subnets across multiple availability zones.

### Subnets

Subnets divide a VPC into smaller networks. Public subnets route to an internet gateway; private subnets do not directly accept inbound internet traffic.

Practice by placing load balancers in public subnets and application/database resources in private subnets.

### Security Groups

Security groups are stateful virtual firewalls attached to resources. They allow inbound and outbound traffic rules.

Practice by allowing HTTP from the internet to a load balancer, allowing app traffic only from the load balancer, and allowing database traffic only from app instances.

### NACLs

Network ACLs are stateless subnet-level firewall rules. They can allow or deny traffic and require both inbound and outbound rules.

Practice by creating a restrictive NACL and observing how stateless rules differ from security groups.

### Internet Gateway

An internet gateway allows public subnet resources to reach and be reached from the internet when routing and public IPs are configured correctly.

Practice by routing a public subnet to an internet gateway and confirming an EC2 instance can serve traffic.

### NAT Gateway

A NAT Gateway allows private subnet resources to initiate outbound internet traffic without exposing inbound access from the internet.

Practice by placing an EC2 instance in a private subnet and using a NAT Gateway for package updates.

### Elastic Load Balancing

ELB distributes traffic across targets. Learn Application Load Balancer, Network Load Balancer, listeners, target groups, health checks, TLS termination, path routing, and connection draining.

Practice by putting two app instances behind an ALB and testing failover when one instance becomes unhealthy.

### API Gateway

API Gateway exposes, secures, throttles, and routes APIs. Learn REST APIs, HTTP APIs, WebSocket APIs, stages, authorizers, throttling, usage plans, and Lambda integration.

Practice by building an API Gateway endpoint that invokes Lambda and enforces rate limits.

### CloudFront

CloudFront is AWS CDN. Learn distributions, origins, cache policies, invalidations, signed URLs, origin access control, TLS, and edge locations.

Practice by serving a static site from S3 through CloudFront with HTTPS and blocked public S3 access.

### Direct Connect

Direct Connect provides dedicated private network connectivity from on-premises networks to AWS.

Practice by drawing a hybrid network design with Direct Connect, VPC routing, redundancy, and failover.

### VPN

AWS VPN connects networks securely over encrypted tunnels. Learn site-to-site VPN, client VPN, customer gateways, virtual private gateways, transit gateway, and route propagation.

Practice by designing a site-to-site VPN architecture and documenting routing and security requirements.

## Storage

### S3

S3 is object storage for files, backups, logs, artifacts, static sites, and data lakes. Learn buckets, objects, keys, policies, versioning, lifecycle rules, encryption, storage classes, and pre-signed URLs.

Practice by uploading files, generating pre-signed URLs, enabling versioning, adding lifecycle rules, and locking down public access.

### EBS

EBS provides block storage for EC2. Learn volume types, IOPS, throughput, snapshots, encryption, resizing, and attachment limits.

Practice by attaching a volume to EC2, formatting it, mounting it, snapshotting it, and restoring from the snapshot.

### EFS

EFS is managed network file storage that can be mounted by multiple compute resources. Learn mount targets, performance modes, throughput modes, access points, and security groups.

Practice by mounting EFS from two EC2 instances and verifying shared file access.

### S3 Glacier

S3 Glacier storage classes are for archival data with lower cost and slower retrieval.

Practice by adding lifecycle rules that transition old objects to Glacier and documenting restore time expectations.

## Databases

### RDS

RDS manages relational databases such as PostgreSQL, MySQL, MariaDB, Oracle, and SQL Server. Learn instances, storage, backups, Multi-AZ, read replicas, parameter groups, subnet groups, and maintenance windows.

Practice by provisioning PostgreSQL in private subnets, connecting from an app instance, enabling backups, and testing a restore.

### DynamoDB

DynamoDB is managed NoSQL key-value and document storage. Learn partition keys, sort keys, access-pattern-first design, GSIs, LSIs, capacity modes, TTL, streams, and conditional writes.

Practice by modeling users and orders using single-table design and testing hot partition behavior.

### Aurora

Aurora is AWS's cloud-native relational database engine compatible with PostgreSQL and MySQL. Learn clusters, writer and reader endpoints, replicas, storage scaling, serverless, backups, and failover.

Practice by creating an Aurora cluster and testing reader endpoint behavior.

### ElastiCache

ElastiCache provides managed Redis or Memcached. Learn caching strategies, TTLs, eviction, replication, clustering, snapshots, and failover.

Practice by caching API responses in Redis and measuring latency before and after.

### Redshift

Redshift is a data warehouse for analytics. Learn columnar storage, distribution keys, sort keys, Spectrum, snapshots, workload management, and query performance.

Practice by loading sample analytics data and optimizing a query with sort and distribution choices.

## Messaging Events

### SQS

SQS is managed queueing for decoupling producers and consumers. Learn visibility timeout, long polling, dead-letter queues, FIFO queues, batching, and idempotent workers.

Practice by building a worker that consumes SQS messages and sends failed messages to a DLQ.

### SNS

SNS is pub/sub messaging for fan-out notifications. Learn topics, subscriptions, filter policies, delivery retries, and protocols such as email, HTTP, Lambda, and SQS.

Practice by publishing an event to SNS and delivering it to multiple SQS queues using filters.

### EventBridge

EventBridge routes events between AWS services, custom apps, and SaaS providers. Learn event buses, rules, schedules, targets, schemas, archives, and replay.

Practice by triggering a Lambda function on a schedule and from a custom application event.

### Kinesis

Kinesis handles real-time streaming data. Learn streams, shards, partition keys, consumers, enhanced fan-out, retention, and replay.

Practice by producing events into a stream and consuming them with a small worker.

### MSK

MSK is managed Apache Kafka. Learn brokers, topics, partitions, consumer groups, networking, authentication, scaling, and monitoring.

Practice by deploying a topic and running producer and consumer clients against MSK.

## Security IAM

### IAM

IAM controls identities and permissions. Learn users, groups, roles, policies, permission boundaries, trust policies, STS, MFA, and least privilege.

Practice by creating roles for EC2, Lambda, and CI/CD instead of using long-lived access keys.

### KMS

KMS manages encryption keys. Learn customer managed keys, AWS managed keys, key policies, grants, rotation, envelope encryption, and service integration.

Practice by encrypting S3 objects and Secrets Manager values with a customer managed key.

### Secrets Manager

Secrets Manager stores, rotates, and retrieves secrets. Learn secret versions, rotation Lambda, IAM access, encryption, and application retrieval patterns.

Practice by storing a database password and reading it from an application role at runtime.

### ACM

ACM manages TLS certificates. Learn public certificates, DNS validation, renewal, regional behavior, and integration with ALB, CloudFront, and API Gateway.

Practice by issuing a certificate and attaching it to an ALB or CloudFront distribution.

### WAF

WAF filters HTTP traffic for web applications. Learn web ACLs, managed rules, rate-based rules, IP sets, regex matching, and logging.

Practice by attaching WAF to CloudFront or an ALB and blocking a test path or IP range.

### Shield

Shield helps protect against DDoS attacks. Learn Standard versus Advanced, protected resources, DDoS response, and integration with WAF and CloudFront.

Practice by documenting DDoS protections for a public web app architecture.

### CloudTrail

CloudTrail records AWS API activity. Learn management events, data events, trails, organization trails, log integrity, and detective security use cases.

Practice by finding who changed a security group or IAM policy using CloudTrail logs.

## Monitoring Operations

### CloudWatch

CloudWatch collects metrics, logs, alarms, dashboards, and events. Learn namespaces, dimensions, log groups, metric filters, alarms, dashboards, and agent setup.

Practice by creating alarms for CPU, error rate, queue depth, and application log patterns.

### X-Ray

X-Ray provides distributed tracing. Learn traces, segments, subsegments, sampling, service maps, and latency debugging.

Practice by instrumenting an API and tracing calls to a database or downstream service.

### AWS Config

AWS Config tracks resource configuration and compliance. Learn recorders, rules, aggregators, conformance packs, and remediation.

Practice by creating a rule that detects public S3 buckets or unrestricted security groups.

### Systems Manager

Systems Manager manages instances and operations. Learn Session Manager, Parameter Store, Run Command, Patch Manager, Automation, and Inventory.

Practice by connecting to a private EC2 instance through Session Manager without opening SSH.

### Trusted Advisor

Trusted Advisor recommends improvements for cost, security, fault tolerance, performance, and service limits.

Practice by reviewing findings and turning them into an action checklist.

## IaC Templates

### CloudFormation Templates

CloudFormation defines AWS infrastructure as templates. Learn resources, parameters, outputs, mappings, conditions, nested stacks, change sets, drift detection, and rollback behavior.

Practice by writing a template for a VPC, EC2 instance, security group, and S3 bucket.

### CDK

CDK defines AWS infrastructure using programming languages. Learn constructs, stacks, apps, synthesis, context, environments, and generated CloudFormation.

Practice by creating the same VPC and EC2 resources using CDK and comparing the generated template.

### Terraform on AWS

Terraform is a widely used infrastructure-as-code tool. Learn providers, resources, modules, state, backends, workspaces, plans, applies, imports, and drift.

Practice by building a reusable AWS VPC module and storing state remotely in S3 with DynamoDB locking.

## DevOps CI/CD

### CodePipeline

CodePipeline orchestrates release workflows. Learn sources, stages, actions, approvals, artifacts, and deployment integration.

Practice by creating a pipeline that builds and deploys an app to ECS or Elastic Beanstalk.

### CodeBuild

CodeBuild runs builds and tests. Learn buildspec files, environments, artifacts, reports, caching, IAM roles, and Docker builds.

Practice by running tests and building a Docker image that is pushed to ECR.

### CodeDeploy

CodeDeploy automates deployments to EC2, Lambda, and ECS. Learn appspec files, deployment groups, hooks, blue-green deployment, and rollback.

Practice by deploying an app to EC2 with lifecycle hooks and automatic rollback.

### CodeArtifact

CodeArtifact stores software packages. Learn repositories, domains, upstreams, authentication tokens, and package manager integration.

Practice by publishing and consuming a private package.

## Architecture

### Well-Architected Framework

The Well-Architected Framework evaluates systems across operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability.

Practice by reviewing an app architecture and writing risks, tradeoffs, and improvements for each pillar.

### Multi-Account Strategy

Multi-account AWS separates environments, workloads, teams, and security boundaries. Learn AWS Organizations, organizational units, service control policies, centralized logging, shared networking, and account vending.

Practice by designing accounts for dev, staging, production, security, logging, and shared services.

### Cost Management

Cost management keeps AWS usage financially controlled. Learn budgets, cost explorer, tags, savings plans, reserved instances, rightsizing, storage lifecycle, and data transfer costs.

Practice by tagging resources, creating a budget alert, and estimating monthly cost for a small architecture.

## Suggested AWS Learning Order

1. `IAM`, `VPC-Management`, `EC2`, `S3`, and `CloudWatch`.
2. `Security-Groups`, `Subnets`, `Internet-Gateway`, `NAT-Gateway`, and `Elastic-Load-Balancing`.
3. `Route-53`, `ACM`, `CloudFront`, and `WAF`.
4. `RDS`, `DynamoDB`, `ElastiCache`, and `SQS`.
5. `ECR`, `ECS`, `EKS`, and `Lambda`.
6. `CloudFormation-Templates`, `CDK`, and `Terraform-on-AWS`.
7. `CodePipeline`, `CodeBuild`, `CodeDeploy`, and production architecture topics.
