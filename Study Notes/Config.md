# AWS Config

AWS Config is a service that enables you to assess, audit, and evaluate configurations of your AWS resources continuously.

## Core Concepts

### Configuration Items (CI)
- Point-in-time snapshot of resource configuration
- JSON document containing:
  - Resource metadata
  - Attributes
  - Relationships to other resources
  - Configuration changes
  - CloudTrail events
- Stored in S3 bucket
- Delivered to SNS topic (optional)

### Configuration Recorder
- Records configuration changes
- Must be enabled per region
- Captures:
  - Resource type changes
  - Relationships between resources
  - Configuration data
- Can record all resources or specific resource types

### Configuration History
- Collection of configuration items for a resource over time
- Shows configuration changes
- Stored in S3 bucket
- Can be viewed for up to 7 years
- **Use case**: Audit, compliance, troubleshooting

### Configuration Snapshot
- Complete set of configuration items for all resources
- Point-in-time view of environment
- Can be delivered on schedule
- Stored in S3 bucket
- **Use case**: Backup, disaster recovery

## Config Rules

### Overview
- Desired configuration standards for resources
- Evaluate compliance automatically
- **Evaluation triggers**:
  - Configuration change
  - Periodic (hourly, daily, etc.)
- **Result**: Compliant, non-compliant, not applicable

### Rule Types

#### AWS Managed Rules
- Pre-built by AWS
- Cover common compliance standards
- Examples:
  - `required-tags`: Ensure resources have specific tags
  - `encrypted-volumes`: Ensure EBS volumes encrypted
  - `s3-bucket-public-read-prohibited`: No public read on S3
  - `iam-password-policy`: Enforce IAM password requirements
  - `rds-multi-az-support`: Ensure RDS has Multi-AZ

#### Custom Rules
- Created using AWS Lambda functions
- Write custom evaluation logic
- More flexibility than managed rules
- **Use case**: Organization-specific compliance requirements

### Rule Evaluation
- **Change-triggered**: When resource configuration changes
- **Periodic**: At specified intervals (1 hour, 3 hours, 6 hours, 12 hours, 24 hours)
- **Both**: Combination of change-triggered and periodic

### Evaluation Modes
- **Detective**: Identify non-compliant resources
- **Proactive**: Evaluate resources before creation (CloudFormation)

## Remediation

### Automatic Remediation
- Auto-remediate non-compliant resources
- Uses Systems Manager Automation documents
- Can specify retry attempts
- **Example**: Attach EBS encryption policy automatically

### Manual Remediation
- View non-compliant resources
- Remediate manually or via runbook
- Track remediation actions

### Remediation Actions
```json
{
  "ConfigRuleName": "encrypted-volumes",
  "TargetType": "SSM_DOCUMENT",
  "TargetIdentifier": "AWS-EnableEBSEncryptionByDefault",
  "TargetVersion": "1",
  "Parameters": {
    "AutomationAssumeRole": {
      "StaticValue": {
        "Values": ["arn:aws:iam::123456789012:role/ConfigRemediationRole"]
      }
    }
  },
  "Automatic": true,
  "MaximumAutomaticAttempts": 5,
  "RetryAttemptSeconds": 60
}
```

## Multi-Account & Multi-Region

### Aggregators
- Aggregate Config data across multiple accounts and regions
- Centralized compliance view
- Two types:
  - **Individual account aggregator**: Specify account IDs
  - **Organization aggregator**: All accounts in AWS Organization
- Requires authorization from source accounts

An aggregator is an AWS Config resource type that collects AWS Config configuration and compliance data from:
- Multiple accounts and multiple regions
- Single account and multiple regions
- An organization in AWS Organizations and all the accounts in that organization

### Cross-Account Authorization
1. Create aggregator in aggregator account
2. Source accounts authorize aggregator
3. Aggregator collects data from authorized accounts

### Benefits
- Unified compliance dashboard
- Centralized security posture
- Simplified auditing across organization

## Advanced Features

### Conformance Packs
- Collection of Config rules and remediation actions
- Package compliance as code
- Based on AWS Config rules and remediation actions
- Can deploy across organization
- **Templates**: YAML format, similar to CloudFormation

Example Conformance Pack:
```yaml
Resources:
  EncryptedVolumesRule:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: encrypted-volumes-conformance-pack
      Source:
        Owner: AWS
        SourceIdentifier: ENCRYPTED_VOLUMES
  RequiredTagsRule:
    Type: AWS::Config::ConfigRule
    Properties:
      ConfigRuleName: required-tags-conformance-pack
      Source:
        Owner: AWS
        SourceIdentifier: REQUIRED_TAGS
      InputParameters:
        tag1Key: Environment
```

### Organization Config Rules
- Deploy Config rules across AWS Organization
- Apply to all accounts or specific OUs
- Centrally managed
- Automatic deployment to new accounts

### Organization Conformance Packs
- Deploy conformance packs across organization
- Centralized compliance management
- Applied to OUs or entire organization

## Supported Resources

### Common AWS Resources
- EC2 (instances, security groups, volumes, VPCs)
- S3 (buckets, bucket policies)
- IAM (users, groups, roles, policies)
- RDS (instances, snapshots)
- Lambda functions
- CloudTrail trails
- EBS volumes and snapshots
- Elastic Load Balancers
- Auto Scaling groups
- CloudFormation stacks
- And 100+ more resource types

### Third-Party Resources
- Custom resource types via Lambda
- Track on-premises resources
- **Use case**: Hybrid compliance monitoring

## Compliance Reporting

### Compliance Dashboard
- Visual overview of compliance status
- Filter by rule, resource type, account
- Drill down into non-compliant resources
- Export compliance data

### Compliance Timeline
- Historical view of compliance
- Track compliance over time
- Identify compliance trends
- **Use case**: Demonstrate continuous compliance

### Compliance by Config Rule
- View compliance per rule
- List non-compliant resources
- See rule parameters and scope

## Integration Patterns

### With CloudWatch Events / EventBridge
- Config events trigger automated actions
- Event types:
  - Configuration change
  - Compliance change
  - Configuration snapshot delivered
  - Configuration history delivered
- Trigger Lambda, SNS, SQS, etc.

Example event pattern:
```json
{
  "source": ["aws.config"],
  "detail-type": ["Config Rules Compliance Change"],
  "detail": {
    "configRuleName": ["required-tags"],
    "newEvaluationResult": {
      "complianceType": ["NON_COMPLIANT"]
    }
  }
}
```

### With Systems Manager
- Remediation via SSM Automation
- Run Command for configuration management
- State Manager for compliance enforcement

### With Security Hub
- Config findings sent to Security Hub
- Centralized security posture
- Compliance standards (CIS, PCI-DSS)

### With CloudTrail
- Config records WHO made changes
- CloudTrail records WHEN and HOW
- Combined view in Config timeline

### With Lambda
- Custom Config rules
- Advanced evaluation logic
- Custom remediation actions

## Monitoring & Notifications

### SNS Integration
- Notifications on:
  - Configuration changes
  - Compliance changes
  - Snapshot delivery
- Configure SNS topic per region

### CloudWatch Metrics
- Limited native metrics
- Create custom metrics via Lambda
- Monitor rule compliance rates

### CloudTrail Logging
- All Config API calls logged
- Track configuration changes
- Audit who modified Config rules

## Security & Access Control

### IAM Permissions
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "config:PutConfigRule",
        "config:DescribeConfigRules",
        "config:GetComplianceDetailsByConfigRule"
      ],
      "Resource": "*"
    }
  ]
}
```

### Service-Linked Role
- AWS Config service role
- Grants permissions to:
  - Describe resources
  - Deliver configuration snapshots to S3
  - Publish to SNS
- Automatically created when enabling Config

### S3 Bucket Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSConfigBucketPermissionsCheck",
      "Effect": "Allow",
      "Principal": {
        "Service": "config.amazonaws.com"
      },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::my-config-bucket"
    },
    {
      "Sid": "AWSConfigBucketPutObject",
      "Effect": "Allow",
      "Principal": {
        "Service": "config.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-config-bucket/AWSLogs/*/Config/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    }
  ]
}
```

## Cost Optimization

### Pricing Components
- **Configuration items**: $0.003 per CI recorded
- **Config rule evaluations**: $0.001 per evaluation (first 100K free per account/region/month)
- **Conformance pack evaluations**: $0.0012 per evaluation
- **S3 storage**: Standard S3 pricing
- **SNS notifications**: Standard SNS pricing

### Cost Reduction Strategies
1. **Selective resource recording**: Record only needed resource types
2. **Reduce rule frequency**: Use periodic rules sparingly
3. **Aggregators**: Centralize instead of per-account rules
4. **S3 lifecycle**: Archive old configuration items to Glacier
5. **Disable unnecessary rules**: Remove rules not actively monitored

## Best Practices

1. **Enable in All Regions**: Comprehensive resource tracking
2. **Use Aggregators**: Centralized multi-account compliance
3. **Automate Remediation**: Use Systems Manager for auto-fix
4. **Tag Resources**: Easier rule scoping and organization
5. **Start with Managed Rules**: Leverage AWS-provided rules
6. **Use Conformance Packs**: Package compliance requirements
7. **Enable for Organization**: Use organization config rules
8. **Monitor Compliance**: Set up alerts for non-compliance
9. **Regular Reviews**: Audit rules and compliance status
10. **Integrate with Security Hub**: Unified security posture
11. **Use EventBridge**: Automate response to config changes
12. **Document Rules**: Clear purpose and remediation steps

## Common Exam Scenarios

### Scenario 1: Ensure All S3 Buckets Encrypted
**Solution**: Enable `s3-bucket-server-side-encryption-enabled` managed rule

### Scenario 2: Auto-Remediate Non-Compliant Resources
**Solution**: Config rule + automatic remediation with SSM Automation document

### Scenario 3: Track Resource Changes Over Time
**Solution**: Enable Config recorder, view configuration history in S3

### Scenario 4: Centralized Compliance Across Accounts
**Solution**: Organization aggregator to collect compliance data from all accounts

### Scenario 5: Alert on Security Group Changes
**Solution**: Config rule for security groups + EventBridge rule + SNS

### Scenario 6: Multi-Region Compliance View
**Solution**: Aggregator collecting from multiple regions in management account

## Key Exam Points

- Config tracks resource configuration changes over time
- Configuration items (CI) are point-in-time snapshots
- Config rules evaluate resource compliance
- Managed rules pre-built by AWS, custom rules use Lambda
- Remediation can be automatic (via SSM Automation) or manual
- Aggregators centralize compliance across accounts/regions
- Conformance packs bundle rules and remediation
- Organization config rules deploy across AWS Organization
- Configuration recorder must be enabled per region
- SNS notifications for configuration and compliance changes
- All data stored in S3 bucket (configuration history)
- Integration with Security Hub for unified security view
- EventBridge for automated response to config changes
- Pricing based on configuration items and rule evaluations
- Config does not prevent changes (detective, not preventive)
- Proactive evaluation available for CloudFormation templates
- 100+ AWS resource types supported
- CloudTrail integration shows WHO made changes

## Related Services

- **AWS CloudTrail**: API call logging (complements Config)
- **AWS Organizations**: Multi-account management
- **AWS Security Hub**: Aggregated security findings
- **Systems Manager**: Automation and remediation
- **EventBridge**: Event-driven automation
- **Lambda**: Custom rules and remediation
- **CloudFormation**: Proactive evaluation
- **S3**: Configuration data storage
- **SNS**: Notifications
- **IAM**: Access control
- **AWS Audit Manager**: Compliance evidence collection
