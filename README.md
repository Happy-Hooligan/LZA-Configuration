# For original AWS-Repository, go to https://github.com/aws/lza-universal-configuration

This Repository is an updated version with changes made in the README.

# LZA Minimal Configuration

This repository contains a stripped-down AWS Landing Zone Accelerator (LZA) configuration optimized for:

- **No LZA-managed networking** — Networking managed externally via Terraform/CloudFormation/Console
- **Cost optimization** — Reduced log retention, minimal KMS CMK usage
- **Account migration compatibility** — Supports migrating accounts from other AWS Organizations

This configuration is based on the [AWS LZA Universal Configuration](https://github.com/aws/lza-universal-configuration) with significant modifications for a minimal test environment.

## Prerequisites

- AWS Control Tower 4.0 deployed with pre-created accounts
- Existing AWS Organizations structure with required OUs
- IAM Identity Center configured

## Configuration Overview

### Organization Structure

| OU | LZA Managed | Purpose |
|----|-------------|---------|
| Root | Yes | Management account only |
| Security | Yes | LogArchive (CloudTrail) and Audit (Config) accounts |
| Delegate-Administrator | Yes | Service delegation accounts (Identity Center) |
| Finance | Yes | FinOps and cost management accounts |
| Networking | Yes | Network accounts (resources managed externally) |
| ControlTower-Workloads | Yes | Production workloads |
| ControlTower-Workloads/High | Yes | High-priority workloads |
| ControlTower-Workloads/Low | Yes | Lower-priority workloads |
| Non-ControlTower-Workloads | Yes | Workloads outside CT governance |
| Closed-Accounts | No (ignore: true) | Accounts being decommissioned |
| SCP-Staging | No (ignore: true) | SCP testing area |
| Migrated-Accounts | No (ignore: true) | Accounts migrated from other orgs |

### Account Mapping

| LZA Logical Name | AWS Account Name | OU | Purpose |
|------------------|------------------|-----|---------|
| Management | (your mgmt account) | Root | Org management, Control Tower, LZA pipeline |
| LogArchive | CloudTrail | Security | CT Log Archive, centralized logs |
| Audit | Config | Security | CT Audit, security delegate admin |
| IdentityCenter-DelegateAdministrator | (your IC account) | Delegate-Administrator | IAM Identity Center delegation |
| FinOps-Workspace | FinOps-Workspace | Finance | Cost management, FinOps |

### Delegate Administrator Configuration

| Service | Config File | Property |
|---------|-------------|----------|
| SecurityHub | security-config.yaml | `centralSecurityServices.delegatedAdminAccount` |
| GuardDuty | security-config.yaml | `centralSecurityServices.delegatedAdminAccount` |
| Macie | security-config.yaml | `centralSecurityServices.delegatedAdminAccount` |
| IAM Access Analyzer | security-config.yaml | `centralSecurityServices.delegatedAdminAccount` |
| Detective | security-config.yaml | `centralSecurityServices.delegatedAdminAccount` |
| IAM Identity Center | iam-config.yaml | `identityCenter.delegatedAdminAccount` |
| AWS Config | security-config.yaml | `awsConfig.aggregation.delegatedAdminAccount` |
| CloudFormation StackSets | **Not in LZA** | Manual via Organizations console or API |
| Service Catalog | **Not in LZA** | Manual |
| Systems Manager | **Not in LZA** | Manual |

## Changes from AWS LZA Universal Configuration

### network-config.yaml
- Removed all networking resources (VPCs, Transit Gateways, IPAM, endpoints)
- Kept only: `defaultVpc.delete: true`
- **Rationale:** Networking team manages infrastructure independently via Terraform/CloudFormation

### replacements-config.yaml
- Removed networking variables (TransitGatewayASN, IPAM pools, VPC CIDRs, subnet masks)
- Removed email placeholders for disabled features
- Kept only: `AcceleratorPrefix`, `HomeRegion`, `EnabledRegions`

### global-config.yaml

#### Disabled Features
- SNS topics (security alerts)
- AWS Backup vaults
- Cost and Usage Reports (CUR)
- AWS Budgets
- Central Root User Management (conflicts with account migration)
- EventBridge default event bus policies
- CloudWatch log streaming and dynamic partitioning
- Session Manager logging
- ELB log bucket
- Control Tower controls (using CT defaults only)

#### Cost Optimizations
- CloudWatch log retention: 365 → 30 days
- Control Tower bucket retention: 365 → 30 days
- S3 lifecycle expiration: 1000 → 90 days
- Removed Glacier transitions
- `terminationProtection: false` for easier cleanup

#### Encryption Changes
- S3: `createCMK: false` (uses SSE-S3 instead of customer-managed KMS)
- Lambda: `useCMK: false` (uses AWS-managed keys)
- **Note:** Some buckets always use CMK: Installer, CodePipeline, CentralLogs, Management account assets

### organization-config.yaml

#### Disabled Policies
- **Declarative policies** — VPC Block Public Access conflicts with external networking management
- **Resource Control Policies (RCPs)** — Requires review for environment
- **Tagging policies** — Not needed for test environment
- **Backup policies** — Backups disabled

#### Active SCPs
| SCP | Purpose | Applied To |
|-----|---------|------------|
| lza-quarantine.json | Locks new accounts until LZA processes them | New accounts (dynamic) |
| lza-suspended-guardrails.json | Blocks LZA from closed accounts | Closed-Accounts OU |
| lza-core-guardrails-1.json | Protects LZA Config, Lambda, SNS, CloudWatch, Kinesis, EventBridge resources | All managed OUs |
| lza-core-guardrails-2.json | Protects LZA IAM, CloudFormation, SSM, S3 + security services | All managed OUs |

#### Removed SCPs
| SCP | Reason |
|-----|--------|
| lza-core-security-guardrails-1.json | Blocks VPC/IGW creation, enforces encryption |
| lza-infrastructure-guardrails-1.json | Extensively blocks networking actions |
| lza-core-workloads-guardrails-1.json | Blocks all VPC/networking actions |
| lza-core-sandbox-guardrails-1.json | No Sandbox OU in this configuration |

### accounts-config.yaml
- Added `accountIds` section for existing account mapping
- Mapped Control Tower 4.0 accounts (LogArchive → CloudTrail, Audit → Config)
- Added workload accounts: `IdentityCenter-DelegateAdministrator`, `FinOps-Workspace`
- Removed default network accounts: SharedServices, Network, Perimeter

### Service Control Policy Changes

#### lza-core-guardrails-1.json
Renamed SIDs for readability:
| Original | New |
|----------|-----|
| GRCFGR | LZAProtectConfigRules |
| GRLMB | LZAProtectLambdaFunctions |
| GRSNS | LZAProtectSNSTopics |
| GRCWLG | LZAProtectCloudWatchLogGroups |
| GRKIN | LZAProtectKinesisFirehose |
| GREB | LZAProtectEventBridgeRules |

#### lza-core-guardrails-2.json
Renamed SIDs for readability:
| Original | New |
|----------|-----|
| GRIAMR | LZAProtectIAMRoles |
| GRIAMRT | LZAProtectTaggedIAMRoles |
| GRCFM | LZAProtectCloudFormationStacks |
| GRSSM | LZAProtectSSMParameters |
| GRS3 | LZAProtectS3Buckets |

**Removed statements:**
- `GRRU` (DenyRootUserActions) — Conflicts with account migration

**Modified ProtectSecurityServices:**
- Split into `DenyLeavingOrganization` (unconditional) and `ProtectSecurityServices`
- Removed: EBS encryption controls, `iam:CreateUser`, `iam:CreateAccountAlias`, `s3:PutAccountPublicAccessBlock`, all `ram:*` actions

### Deleted Files
- `event-bus-policies/` — No longer referenced
- `dynamic-partitioning/` — Log streaming disabled
- `rcp-policies/` — RCPs disabled
- `declarative-policies/` — Declarative policies disabled
- `tagging-policies/` — Not needed
- `backup-policies/` — Backups disabled

## Pre-Deployment Checklist

- [ ] Update `HomeRegion` and `EnabledRegions` in replacements-config.yaml
- [ ] Update email addresses in accounts-config.yaml
- [ ] Update account IDs in accounts-config.yaml `accountIds` section
- [ ] Verify Control Tower version matches `landingZone.version: "4.0"` in global-config.yaml
- [ ] Confirm OU names match your AWS Organizations structure
- [ ] Review security-config.yaml delegate admin settings
- [ ] Review iam-config.yaml Identity Center delegate admin

## Cost Considerations

This configuration minimizes costs by:
- Using SSE-S3 instead of KMS CMKs (~$1/key/month savings)
- Reducing log retention (30-90 days vs 365-1000 days)
- Disabling Kinesis/Firehose streaming
- Removing additional Control Tower controls (Config rules have per-evaluation costs)
- Disabling AWS Backup vaults

Estimated monthly cost: **$100-500** (varies by region and activity)

### iam-config.yaml
- Commented out Identity Center block (managed via console, not LZA)
  - EntraID SAML/SCIM federation configured directly in Identity Center console
  - Permission sets easier to manage without running LZA pipeline
- Removed all policySets:
  - End-User-Policy had VPC restrictions that block networking team
  - Permissions boundaries administratively cumbersome (using SCPs instead)
- Removed all roleSets:
  - Backup-Role: AWS Backup disabled
  - EC2-Default-SSM-Role: SSM not currently in use
- Added comprehensive comments for re-enabling SSM role later

### security-config.yaml
- Delegate admin: Audit account (your CT Config account)
- Disabled ebsDefaultVolumeEncryption (not enforcing encryption, uses KMS)
- Disabled s3PublicAccessBlock (developers need public bucket capability)
- Enabled scpRevertChangesConfig (auto-reverts manual SCP changes)
- Disabled Macie (can enable later for sensitive data discovery)
- GuardDuty configuration:
  - s3Protection: enabled (monitors S3 for data exfiltration)
  - eksProtection: enabled (monitors EKS Kubernetes audit logs)
  - exportConfiguration: enabled (exports findings to S3)
- SecurityHub: NIST 800-53 R5 only (removed FSBP and CIS v3.0.0)
- AWS Config:
  - Aggregation enabled with Audit as delegated admin (centralized compliance view)
  - Minimal rule set (7 rules) - rely on SecurityHub NIST controls
- Removed SSM automation documents (referenced deleted resources)

#### Config Rules Kept
| Rule | Purpose |
|------|---------|
| iam-user-group-membership-check | Ensures IAM users belong to groups |
| s3-bucket-policy-grantee-check | Validates S3 bucket policy principals |
| ec2-volume-inuse-check | Identifies unattached EBS volumes (cost optimization) |
| guardduty-non-archived-findings | Alerts on unaddressed GuardDuty findings |
| lambda-dlq-check | Ensures Lambda functions have DLQs |
| api-gw-cache-enabled-and-encrypted | Validates API Gateway cache encryption |
| emr-kerberos-enabled | Ensures EMR uses Kerberos authentication |

#### Config Rules Removed
| Rule | Reason |
|------|--------|
| All backup-* rules | AWS Backup disabled |
| secretsmanager-using-cmk | Not enforcing CMK |
| cloudtrail-enabled | Control Tower manages |
| ec2-instance-profile-attached | Remediation referenced removed SSM role |
| elb-logging-enabled | Remediation referenced removed ELB bucket |

### Deleted Files (iam-config)
- `iam-policies/sample-end-user-policy.json` — Permissions boundary not used
- `iam-policies/ssm-s3-policy.json` — SSM role not deployed
- `iam-policies/` folder — Empty after above deletions

### Deleted Files (security-config)
- `ssm-documents/` folder — Documents referenced removed resources
- `ssm-remediation-roles/` folder — Remediation roles no longer needed

## Optional Security Services (Not Enabled)

The following services can be enabled later based on requirements:

| Service | Purpose | Cost Estimate | Config Location |
|---------|---------|---------------|-----------------|
| Amazon Macie | Sensitive data discovery in S3 | ~$1/account/month + scanning | `centralSecurityServices.macie.enable` |
| Amazon Detective | Security investigation and visualization | ~$2/GB analyzed | `detective.enable` |
| Amazon Inspector | Vulnerability scanning (EC2/ECR/Lambda) | ~$0.01/instance assessment | `inspector.enable` |
| AWS Audit Manager | Automated compliance evidence collection | ~$1.25/resource-month | `auditManager.enable` |
| GuardDuty Malware Protection | EBS malware scanning | ~$0.05/GB scanned | `guardduty.malwareProtection.enable` |
| GuardDuty Runtime Monitoring | Container/EC2 runtime threat detection | Per-vCPU pricing | `guardduty.runtimeMonitoring.enable` |

## Pre-Deployment Checklist

- [ ] Update `HomeRegion` and `EnabledRegions` in replacements-config.yaml
- [ ] Update email addresses in accounts-config.yaml
- [ ] Update account IDs in accounts-config.yaml `accountIds` section
- [ ] Verify Control Tower version matches `landingZone.version: "4.0"` in global-config.yaml
- [ ] Confirm OU names match your AWS Organizations structure
- [ ] Review security-config.yaml delegate admin settings (should be Audit account)
- [ ] Verify iam-config.yaml Identity Center is commented out (managing via console)
- [ ] Delete unused folders: `iam-policies/`, `ssm-documents/`, `ssm-remediation-roles/`

## References

- [AWS LZA Documentation](https://awslabs.github.io/landing-zone-accelerator-on-aws/)
- [LZA TypeDoc Reference](https://awslabs.github.io/landing-zone-accelerator-on-aws/latest/typedocs/)
- [Original LZA Universal Configuration](https://github.com/aws/lza-universal-configuration)
- [Control Tower Controls Reference](https://docs.aws.amazon.com/controltower/latest/controlreference/all-global-identifiers.html)
