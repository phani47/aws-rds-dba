# AWS RDS DBA Plan

**Learner:** Phani Kumar  
**Repository:** `aws-rds-dba`  
**AWS Region:** `ap-south-2` (Hyderabad)  
**Database focus:** Amazon RDS for PostgreSQL  
**Additional focus:** Amazon RDS Proxy, security, monitoring, troubleshooting, automation, and migration

## Goal

Build practical, job-relevant AWS RDS DBA skills through a 15-day hands-on lab. All exercises should be documented in this repository with commands, verification output, architecture notes, troubleshooting observations, and cleanup steps.

## Training principles

- Proceed one verified step at a time.
- Use least-privilege security and private networking wherever practical.
- Check pricing and account Free Tier/credit eligibility before creating billable resources.
- Use budget alerts as notifications, not as a hard spending cap.
- Never commit passwords, access keys, tokens, connection strings containing credentials, or other secrets to Git.
- Tag lab resources consistently, for example `Project=aws-rds-dba` and `Environment=lab`.
- Record resource IDs and evidence in the `evidence/` directory, excluding credentials and sensitive values.

## 15-day curriculum

| Day | Module | Practical outcomes |
|---|---|---|
| 1 | AWS account, IAM, MFA, billing and CLI | Verify account access, MFA, AWS CLI profile, region, budget alerts, and safe lab practices. |
| 2 | VPC and private networking | Create and verify a dedicated VPC, private subnets in two AZs, route table, subnet associations, and RDS DB subnet group. |
| 3 | RDS PostgreSQL provisioning | Review current pricing and eligibility; create a small suitable PostgreSQL RDS instance, configure storage, credentials, backups, encryption, and security settings. |
| 4 | PostgreSQL administration on RDS | Connect, inspect parameters and extensions, manage users and privileges, and understand RDS-managed operational restrictions. |
| 5 | Backup and recovery | Configure backup retention, create and restore snapshots, perform point-in-time recovery where available, and document recovery considerations. |
| 6 | High availability and failover | Study Multi-AZ architecture, test failover where feasible and affordable, and distinguish Multi-AZ availability from read scaling. |
| 7 | Monitoring and logs | Use CloudWatch metrics, RDS events, PostgreSQL logs, Performance Insights/Database Insights availability, and alarms where supported. |
| 8 | Performance and connection management | Examine query plans, indexes, waits, connection counts, workload behavior, and PostgreSQL connection limits. Establish direct-connection baselines. |
| 9 | Amazon RDS Proxy — implementation | Review regional pricing and prerequisites; configure authentication with Secrets Manager or supported IAM authentication, create the proxy, register the RDS target, configure security groups, and verify target health. |
| 10 | RDS Proxy — testing and troubleshooting | Connect through the proxy endpoint; compare direct and proxied connections; test connection reuse, connection borrowing, session pinning, metrics, logs, and failover behavior. |
| 11 | RDS security and operations | Review encryption, TLS, IAM database authentication, Secrets Manager rotation options, network access, auditing, and operational controls. |
| 12 | Automation | Use AWS CLI and scripts for resource inventory, status checks, monitoring, and repeatable operational tasks. |
| 13 | Infrastructure as Code | Introduce Terraform for a controlled, reviewable deployment; inspect plans and destroy lab resources safely. |
| 14 | Database migration | Review migration approaches and AWS Database Migration Service concepts; run a scoped migration exercise if the lab environment and budget permit. |
| 15 | Capstone and interview preparation | Document a private PostgreSQL RDS environment with proxy, security, monitoring, backup/recovery, performance tests, cost controls, architecture, and operational runbooks. |

The schedule is a learning sequence. Adjustments may be made when prerequisites, regional service availability, account eligibility, or cost constraints require them.

## Amazon RDS Proxy module

### Learning objectives

- Explain why RDS Proxy is used and how it differs from direct database connections and application-side connection pools.
- Understand connection pooling, connection multiplexing, connection borrowing, and session pinning.
- Configure a proxy for an RDS PostgreSQL database using supported authentication.
- Secure traffic between the client, proxy, and database using appropriate security groups and TLS settings.
- Connect to the proxy endpoint and validate target health.
- Observe proxy metrics and diagnose connection and failover issues.
- Explain operational trade-offs and proxy-specific costs.

### Implementation checklist

- [ ] Confirm RDS Proxy support, prerequisites, and current pricing in `ap-south-2`.
- [ ] Confirm the RDS database is available and its subnet group and security groups are correct.
- [ ] Choose and document the authentication approach (Secrets Manager and/or supported IAM authentication).
- [ ] Create or verify the required secret and permissions without committing secret values.
- [ ] Configure the proxy security group and database security group with least-privilege rules.
- [ ] Create the RDS Proxy and register the RDS database as its target.
- [ ] Verify proxy status and target health.
- [ ] Save the proxy endpoint and non-secret resource identifiers in the evidence file.
- [ ] Connect with `psql` through the proxy using TLS and verify the database identity.
- [ ] Run a controlled connection test and compare direct versus proxy behavior.
- [ ] Review available CloudWatch proxy metrics and logs.
- [ ] Test or analyze failover behavior and document observed limitations.
- [ ] Delete the proxy after the exercise if it is not needed, and verify cleanup.

### Key concepts to document

| Concept | What to record |
|---|---|
| Connection pooling | How the proxy manages backend database connections and when client connections can reuse them. |
| Connection borrowing | Situations in which a client session needs a backend connection. |
| Session pinning | Session features or state that can prevent multiplexing; document observed behavior rather than assuming every session is multiplexed. |
| Failover | Client connection behavior, recovery observations, and application retry requirements. |
| Authentication | Secret/IAM configuration, permissions, rotation considerations, and how credentials are kept out of source control. |
| Monitoring | Proxy and database metrics, alarms, logs, and diagnostic steps. |
| Cost | Proxy pricing basis and the cost of associated database, secrets, monitoring, and networking resources. |

## Networking and lab baseline

The following resources were created or recorded during the lab. Verify current AWS state before relying on these values in a future session.

| Resource | Recorded value |
|---|---|
| Region | `ap-south-2` |
| VPC name | `aws-rds-dba-vpc` |
| VPC ID | `vpc-00684523936b75f79` |
| Private subnet A | `subnet-0d9b263c6e58e8c35` — `ap-south-2a` |
| Private subnet B | `subnet-0c00ecb0aeeb89b4d` — `ap-south-2b` |
| Route table | `rtb-0df711637900581a9` |
| RDS DB subnet group | `aws-rds-dba-subnet-group` |
| Subnet group status at verification | `Complete` |
| VPC CIDR | `10.20.0.0/16` |
| Private subnet A CIDR | `10.20.11.0/24` |
| Private subnet B CIDR | `10.20.12.0/24` |

The route table was observed with the VPC-local route `10.20.0.0/16`. Confirm subnet associations before proceeding with database provisioning. The subnet group was verified to include both active subnets in separate Availability Zones.

## Cost-control and daily cleanup policy

### Resources generally safe to retain for the lab

The VPC, subnets, route tables, security groups, and RDS DB subnet group have no separate hourly charge merely for existing. Keeping this basic network between sessions avoids repeated setup and reduces configuration mistakes.

### Resources that require cost review

- RDS DB instances: instance compute charges apply while running; stopping does not eliminate storage and backup charges.
- RDS Proxy: separately billed; review current regional pricing and delete when not needed.
- NAT Gateways and interface VPC endpoints: can incur ongoing hourly and data-processing charges.
- EC2 instances: compute and attached storage may incur charges.
- Snapshots and retained backups: storage charges may apply.
- Secrets Manager, CloudWatch logs/metrics/alarms, data transfer, and other enabled services: review applicable charges.
- Public IPv4 addresses and Elastic IP allocations: check current AWS pricing and whether any address is billable.

### End-of-session checklist

- [ ] Decide whether the RDS instance is needed before the next session.
- [ ] If deleting the DB, review final snapshot and automated backup retention choices first.
- [ ] Delete the RDS Proxy if it is no longer required for the exercise.
- [ ] Remove temporary EC2 instances, NAT Gateways, endpoints, or other billable resources when no longer needed.
- [ ] Review retained snapshots, backups, secrets, logs, and other storage-bearing resources.
- [ ] Check AWS Billing/Cost Explorer and budget notifications.
- [ ] Record what was retained and what was deleted.

**Important:** AWS Budgets alerts notify you; they do not automatically cap or prevent charges. Verify current account plan, credits, service eligibility, and regional pricing before provisioning.

## GitHub documentation workflow

Suggested repository structure:

```text
aws-rds-dba/
├── README.md
├── docs/
│   ├── 15-day-bootcamp-plan.md
│   ├── architecture.md
│   ├── rds-proxy-lab.md
│   └── cost-control-and-cleanup.md
└── evidence/
    └── lab-resource-ids.txt
```

After saving this plan as `docs/aws-rds-plan.md`, review and commit it:

```bash
git status
git add docs/aws-rds-plan.md
git commit -m "docs: add 15-day RDS DBA plan with RDS Proxy"
git push origin main
```

Before committing, inspect staged changes and ensure no credentials, secret values, tokens, or sensitive account information are included.
