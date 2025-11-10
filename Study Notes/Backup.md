# AWS Backup

AWS Backup is a fully managed service that centralizes and automates data protection across AWS services and hybrid workloads.

## Core Concepts

### Backup Plans
- Schedule and retention policies
- Can have multiple rules within a plan
- Assigned to resources via tags or resource IDs
- Supports lifecycle policies (transition to cold storage)

### Backup Vaults
- Logical containers for backup recovery points
- Encryption with AWS KMS
- Access policies control who can access backups
- Can lock vault to prevent deletion (Vault Lock)
- Organized by compliance or operational requirements

### Recovery Points
- Snapshot or backup of a resource at specific point in time
- Stored in backup vault
- Can restore resources from recovery points
- Retention period defined in backup plan

## Supported Services

### AWS Services
- Amazon EC2 (EBS volumes, instances)
- Amazon EBS
- Amazon RDS (all engines)
- Amazon Aurora
- Amazon DynamoDB
- Amazon EFS
- Amazon FSx (Windows File Server, Lustre, ONTAP, OpenZFS)
- AWS Storage Gateway (Volume Gateway)
- Amazon S3
- Amazon Neptune
- Amazon DocumentDB
- Amazon Timestream
- AWS CloudFormation stacks
- Amazon Redshift
- Amazon ElastiCache (Redis OSS)
- Amazon SAP HANA on EC2

### Hybrid Workloads
- On-premises via AWS Storage Gateway
- VMware workloads via AWS Backup gateway
- SQL Server, Oracle databases on EC2

## Backup Plans

### Plan Components
- **Backup rules**: Define when and how often to backup
- **Backup frequency**: Continuous, hourly, daily, weekly, monthly
- **Backup window**: Time window for backup to start
- **Lifecycle**: Transition to cold storage, deletion
- **Copy to destination**: Cross-region/cross-account copies

### Backup Rule Example
```json
{
  "RuleName": "DailyBackups",
  "ScheduleExpression": "cron(0 5 * * ? *)",
  "StartWindowMinutes": 60,
  "CompletionWindowMinutes": 120,
  "Lifecycle": {
    "MoveToColdStorageAfterDays": 30,
    "DeleteAfterDays": 365
  }
}
```

### Backup Frequency Options
- **Continuous backups**: Point-in-time recovery (PITR) - 5 minutes RPO
- **Scheduled backups**:
  - Every 12 hours (minimum)
  - Daily
  - Weekly
  - Monthly
  - Custom cron expressions

### Resource Assignment
- **By tags**: All resources with specific tag key-value
- **By resource ID**: Specific resource ARN
- **All resources**: Specific resource type

```json
{
  "Selections": [{
    "SelectionName": "ProductionResources",
    "IamRoleArn": "arn:aws:iam::123456789012:role/AWSBackupRole",
    "Resources": ["*"],
    "Conditions": {
      "StringEquals": [{
        "ConditionKey": "aws:ResourceTag/Environment",
        "ConditionValue": "Production"
      }]
    }
  }]
}
```

## Backup Storage

### Backup Vault Features
- **Encryption**: All backups encrypted with KMS
- **Access control**: Vault access policy
- **Vault Lock**: WORM (Write Once Read Many) compliance
- **Cross-account backup**: Copy backups to different account
- **Cross-region backup**: Copy backups to different region
- **Notifications**: SNS on backup job events

### Vault Lock (Compliance Mode)
- Prevents deletion of backups
- Enforces retention period
- Governance vs. Compliance modes
- Once locked, cannot be changed
- Required for certain compliance frameworks

```bash
aws backup put-backup-vault-lock-configuration \
  --backup-vault-name ComplianceVault \
  --min-retention-days 365 \
  --max-retention-days 1825
```

### Backup Vault Access Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Principal": "*",
    "Action": "backup:DeleteRecoveryPoint",
    "Resource": "*"
  }]
}
```

## Lifecycle Management

### Storage Classes
- **Warm storage**: Fast retrieval (default)
- **Cold storage**: Lower cost, slower retrieval
  - Minimum 90 days retention
  - Additional retrieval fees
  - Not all services supported

### Lifecycle Policies
- Transition to cold storage after X days
- Delete after Y days
- Automatically applied based on backup plan

```
Day 0-29: Warm storage
Day 30-364: Cold storage
Day 365: Deleted
```

## Cross-Region & Cross-Account

### Cross-Region Copy
- Automatically copy backups to another region
- Define in backup plan rules
- Independent retention policy
- Encrypted with KMS in destination region
- **Use case**: Disaster recovery

### Cross-Account Copy
- Copy backups to different AWS account
- Requires:
  - Destination account backup vault with policy allowing source account
  - Source account backup plan with copy action
  - KMS key policy for cross-account access
- **Use case**: Centralized backup management

### Copy Configuration
```json
{
  "CopyActions": [{
    "DestinationBackupVaultArn": "arn:aws:backup:us-west-2:123456789012:backup-vault:MyVault",
    "Lifecycle": {
      "DeleteAfterDays": 90
    }
  }]
}
```

## Restore Operations

### Restore Types
- **Full restore**: Complete resource restoration
- **Item-level restore**: S3 objects, DynamoDB items, EFS files
- **Point-in-time restore**: For continuous backups (DynamoDB, RDS)
- **Cross-region restore**: Restore in different region
- **Cross-account restore**: Restore to different account

### Restore Process
1. Select recovery point
2. Configure restore parameters
3. Execute restore
4. Monitor restore job
5. Verify restored resource

### Supported Restore Paths
- Restore to same account/region
- Restore to different region
- Restore to different account (via copy)
- In-place restore or new resource

## Monitoring & Compliance

### AWS Backup Audit Manager
- Automated compliance reporting
- Built-in frameworks:
  - GDPR
  - HIPAA
  - PCI
  - ISO
  - FedRAMP
- Custom frameworks supported
- Daily compliance reports
- Integration with SNS for alerts

### CloudWatch Metrics
- `NumberOfBackupJobsCreated`
- `NumberOfBackupJobsCompleted`
- `NumberOfBackupJobsFailed`
- `NumberOfRestoreJobsCompleted`
- Per backup vault metrics

### CloudWatch Events / EventBridge
- Backup job state changes
- Restore job state changes
- Copy job state changes
- Recovery point deleted
- Trigger Lambda, SNS, etc.

### CloudTrail Integration
- All AWS Backup API calls logged
- Track configuration changes
- Audit backup and restore operations

## AWS Organizations Integration

### Backup Policies
- Centrally manage backup plans across organization
- Apply to OUs or accounts
- Inheritance from parent OUs
- Member accounts create backups automatically
- **Use case**: Enforce backup compliance org-wide

```json
{
  "plans": {
    "OrgBackupPlan": {
      "regions": ["us-east-1", "us-west-2"],
      "rules": {
        "DailyBackup": {
          "schedule_expression": "cron(0 5 ? * * *)",
          "lifecycle": {
            "delete_after_days": 30
          },
          "target_backup_vault_name": "Default"
        }
      },
      "selections": {
        "tags": {
          "BackupPolicy": {
            "iam_role_arn": "arn:aws:iam::$account:role/AWSBackupDefaultServiceRole",
            "tag_key": "backup",
            "tag_value": ["true"]
          }
        }
      }
    }
  }
}
```

## IAM Permissions

### Service Role
- AWS Backup assumes role to perform backups
- Managed policies:
  - `AWSBackupServiceRolePolicyForBackup`
  - `AWSBackupServiceRolePolicyForRestores`
- Custom policies for specific resources

### User Permissions
- Control who can create/modify backup plans
- Separate permissions for backup vs. restore
- Tag-based access control

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "backup:CreateBackupPlan",
      "backup:StartBackupJob"
    ],
    "Resource": "*",
    "Condition": {
      "StringEquals": {
        "aws:RequestedRegion": ["us-east-1"]
      }
    }
  }]
}
```

## Advanced Features

### Legal Hold
- Prevents deletion of recovery point
- For litigation or investigations
- Can be removed when no longer needed
- Different from Vault Lock (more flexible)

### Backup Gateway
- Hybrid cloud backup solution
- Backup on-premises VMware VMs to AWS
- Uses AWS Backup for management
- Encrypted data transfer

### AWS Backup for S3
- Application-consistent S3 backups
- Continuous backups with 5-minute RPO
- Supports versioned and unversioned buckets
- Point-in-time restore

## Best Practices

1. **Tag Resources**: Consistent tagging for automated backup selection
2. **Test Restores**: Regularly test backup restoration
3. **Cross-Region Copies**: For disaster recovery
4. **Vault Lock**: For compliance and immutability
5. **Lifecycle Policies**: Optimize storage costs
6. **Audit Reports**: Regular compliance checking
7. **Separate Vaults**: By environment or compliance requirement
8. **Monitor Jobs**: Set up CloudWatch alarms for failures
9. **IAM Least Privilege**: Separate backup and restore permissions
10. **Backup All Critical Data**: Comprehensive backup coverage
11. **Document RPO/RTO**: Define and test recovery objectives
12. **Use Organization Policies**: Enforce backup compliance
13. **Encrypt Backups**: Use customer-managed KMS keys for sensitive data

## Cost Optimization

### Pricing Components
- **Backup storage**: Per GB per month
  - Warm storage: ~$0.05/GB/month
  - Cold storage: ~$0.01/GB/month
- **Restore**: Per GB restored
- **Cross-region data transfer**: Standard AWS rates
- **Early deletion**: Charged for minimum retention period

### Cost Reduction Strategies
1. Use lifecycle policies to move to cold storage
2. Set appropriate retention periods
3. Delete unnecessary backups
4. Use incremental backups (automatic for most services)
5. Consider using native service backups for lower volume
6. Monitor backup vault usage with Cost Explorer

## Common Exam Scenarios

### Scenario 1: Centralized Backup Across Accounts
**Solution**: AWS Organizations backup policies applied to OUs, cross-account vault access

### Scenario 2: Compliance Requirement for 7-Year Retention
**Solution**: Backup plan with 7-year retention, Vault Lock in compliance mode

### Scenario 3: Disaster Recovery to Another Region
**Solution**: Backup plan with cross-region copy action, test restore in DR region

### Scenario 4: Automated Backup Based on Tags
**Solution**: Backup plan with resource selection by tag (e.g., backup=true)

### Scenario 5: Alert on Backup Failures
**Solution**: EventBridge rule for backup job failures → SNS → Email/PagerDuty

### Scenario 6: Point-in-Time Recovery
**Solution**: Enable continuous backups in backup plan (supported for DynamoDB, RDS, S3)

## Key Exam Points

- AWS Backup centralizes and automates backups across AWS services
- Backup plans define schedule, retention, and lifecycle
- Backup vaults are logical containers with encryption and access control
- Vault Lock provides WORM compliance for immutable backups
- Supports cross-region and cross-account backup copies
- Lifecycle policies transition backups to cold storage (cost savings)
- Continuous backups provide 5-minute RPO (PITR)
- AWS Organizations integration for centralized backup policies
- Audit Manager provides automated compliance reporting
- All backups encrypted with AWS KMS
- CloudWatch Events/EventBridge for job state changes
- Resource selection by tags or resource IDs
- Legal hold prevents recovery point deletion
- Supports hybrid workloads via Backup Gateway
- Cold storage requires minimum 90-day retention
- Cross-region copies use independent retention policies

## Related Services

- **Amazon EBS**: Volume snapshots
- **Amazon RDS**: Database backups
- **Amazon S3**: Object versioning and replication
- **AWS Storage Gateway**: Hybrid cloud backup
- **AWS Organizations**: Centralized backup policies
- **AWS KMS**: Backup encryption
- **CloudWatch**: Monitoring and alarms
- **CloudTrail**: Audit logging
- **EventBridge**: Event-driven automation
- **SNS**: Notifications
- **DynamoDB**: Point-in-time recovery
