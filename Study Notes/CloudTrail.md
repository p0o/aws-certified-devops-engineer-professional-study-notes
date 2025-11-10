# AWS CloudTrail

AWS CloudTrail is a service that enables governance, compliance, operational auditing, and risk auditing of your AWS account by logging and monitoring API calls and related events.

## Core Concepts

### What CloudTrail Records
- **API Calls**: Actions taken by users, roles, or AWS services
- **Who**: IAM user, role, or AWS service
- **When**: Timestamp of the action
- **Where**: Source IP address
- **What**: Service, action, parameters, response
- **Non-API Events**: Console sign-ins, AWS service events

### Event Types

#### Management Events
- **Control plane operations** on AWS resources
- Examples:
  - Creating EC2 instances (`RunInstances`)
  - Creating S3 buckets (`CreateBucket`)
  - Configuring security (`AttachRolePolicy`)
  - Setting up logging (`PutBucketLogging`)
- **Read Events**: Don't modify resources (e.g., `DescribeInstances`)
- **Write Events**: Modify resources (e.g., `TerminateInstances`)
- Enabled by default in trails

#### Data Events
- **Data plane operations** on or within resources
- High volume operations (disabled by default due to cost)
- Examples:
  - S3 object-level operations (`GetObject`, `PutObject`, `DeleteObject`)
  - Lambda function invocations (`Invoke`)
  - DynamoDB item-level operations (`GetItem`, `PutItem`)
- Can filter by specific resources or all resources

#### Insights Events
- **Anomaly detection** for unusual API activity
- Uses machine learning to establish baseline
- Detects:
  - Unusual call volume
  - Error rate spikes
  - Service usage anomalies
- Examples: Sudden spike in `TerminateInstances` calls
- Additional cost per event analyzed

## Trail Configuration

### Trail Types

#### Single-Region Trail
- Records events in one AWS region
- Legacy configuration
- Lower cost for region-specific monitoring

#### Multi-Region Trail
- Records events in all regions
- Automatically includes new regions
- Recommended for comprehensive auditing
- **One trail can serve multiple regions**

#### Organization Trail
- Records events for all accounts in AWS Organization
- Created in management account
- Automatically applies to new accounts
- Centralized logging

### Trail Settings

#### Log File Delivery
- Stored in Amazon S3 bucket
- Log files delivered every 5 minutes
- Can span multiple accounts/regions
- File format: Gzipped JSON
- File path structure:
  ```
  s3://bucket-name/prefix/AWSLogs/account-id/CloudTrail/region/YYYY/MM/DD/
  ```

#### Log File Validation
- Ensures logs haven't been modified/deleted
- Uses SHA-256 hashing and digital signatures
- Digest files created hourly
- Can validate using AWS CLI:
  ```bash
  aws cloudtrail validate-logs --trail-arn <trail-arn> --start-time <time>
  ```

#### Encryption
- **SSE-S3**: Default encryption
- **SSE-KMS**: Customer-managed key for additional security
- KMS key policy must allow CloudTrail to use the key
- Required for compliance in some industries

### Event Selectors

#### Management Event Selectors
```json
{
  "ReadWriteType": "All",  // All, ReadOnly, WriteOnly
  "IncludeManagementEvents": true
}
```

#### Data Event Selectors
```json
{
  "ReadWriteType": "All",
  "IncludeManagementEvents": false,
  "DataResources": [
    {
      "Type": "AWS::S3::Object",
      "Values": ["arn:aws:s3:::my-bucket/*"]
    },
    {
      "Type": "AWS::Lambda::Function",
      "Values": ["arn:aws:lambda:*:*:function/*"]
    }
  ]
}
```

#### Advanced Event Selectors
- More granular control with field-based filtering
- Filter on attributes like:
  - `eventCategory`
  - `resources.type`
  - `resources.ARN`
  - `readOnly`
  - `eventName`
```json
{
  "Name": "Log S3 write events for specific bucket",
  "FieldSelectors": [
    { "Field": "eventCategory", "Equals": ["Data"] },
    { "Field": "resources.type", "Equals": ["AWS::S3::Object"] },
    { "Field": "resources.ARN", "StartsWith": ["arn:aws:s3:::my-bucket/"] },
    { "Field": "readOnly", "Equals": ["false"] }
  ]
}
```

## CloudTrail Lake

### Overview
- **Managed data lake** for CloudTrail events
- Long-term event retention (up to 7 years)
- SQL-based querying
- Immutable event store
- Aggregation across accounts and regions

### Event Data Stores
- Container for CloudTrail Lake events
- Configurable retention (7 days to 7 years)
- Pricing based on:
  - Ingestion: Per GB
  - Storage: Per GB/month
  - Queries: Per GB scanned

### Querying
- SQL syntax similar to Athena
- Faster than querying S3 directly
- Example query:
```sql
SELECT
  userIdentity.principalId,
  eventName,
  eventTime,
  sourceIPAddress
FROM <event-data-store-id>
WHERE eventTime > '2024-01-01'
  AND eventName = 'RunInstances'
ORDER BY eventTime DESC
LIMIT 100
```

### Use Cases
- Compliance auditing
- Security analysis
- Operational troubleshooting
- Cost anomaly investigation

## Integration Patterns

### With CloudWatch Logs
- Stream trail events to CloudWatch Logs
- Enables:
  - Real-time monitoring
  - Metric filters
  - CloudWatch alarms on specific API calls
- Use case: Alert on root account usage

```json
// Metric filter pattern
{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }
```

### With Amazon EventBridge
- CloudTrail events automatically sent to EventBridge
- Create rules for specific API calls
- Trigger automation:
  - Lambda functions
  - SNS notifications
  - Systems Manager Automation
  - Step Functions
- Use case: Auto-remediate security group changes

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventName": ["AuthorizeSecurityGroupIngress"]
  }
}
```

### With Amazon Athena
- Query CloudTrail logs in S3 using SQL
- Create table using CloudTrail console or manually
- Partition by date for performance
- Use case: Ad-hoc analysis and investigation

```sql
CREATE EXTERNAL TABLE cloudtrail_logs (
  eventVersion STRING,
  userIdentity STRUCT<
    type: STRING,
    principalId: STRING,
    arn: STRING>,
  eventTime STRING,
  eventName STRING,
  ...
)
PARTITIONED BY (region STRING, year STRING, month STRING, day STRING)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
LOCATION 's3://my-bucket/AWSLogs/123456789012/CloudTrail/'
```

### With AWS Organizations
- Organization trail logs all member accounts
- Centralized security and compliance
- Member accounts cannot modify/delete organization trails
- Requires trust between member accounts and management account

## Security & Compliance

### IAM Permissions
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudtrail:LookupEvents",
        "cloudtrail:GetTrailStatus"
      ],
      "Resource": "*"
    }
  ]
}
```

### S3 Bucket Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": { "Service": "cloudtrail.amazonaws.com" },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::my-trail-bucket"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": { "Service": "cloudtrail.amazonaws.com" },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-trail-bucket/*",
      "Condition": {
        "StringEquals": { "s3:x-amz-acl": "bucket-owner-full-control" }
      }
    }
  ]
}
```

### SNS Notifications
- Notify when log files delivered
- Can trigger Lambda for processing
- Topic policy must allow CloudTrail to publish

### Compliance Standards
- **PCI DSS**: API activity logging
- **HIPAA**: Audit trail requirement
- **SOC**: Access monitoring
- **GDPR**: Data access tracking
- **FedRAMP**: Government compliance

## Monitoring & Alerting

### Common Security Scenarios

#### 1. Root Account Usage
```json
// CloudWatch Log metric filter
{ $.userIdentity.type = "Root" }
```

#### 2. Unauthorized API Calls
```json
{ $.errorCode = "AccessDenied" || $.errorCode = "UnauthorizedOperation" }
```

#### 3. Console Sign-in Without MFA
```json
{ $.eventName = "ConsoleLogin" && $.additionalEventData.MFAUsed = "No" }
```

#### 4. IAM Policy Changes
```json
{ $.eventName = PutUserPolicy || $.eventName = PutRolePolicy || $.eventName = PutGroupPolicy }
```

#### 5. S3 Bucket Policy Changes
```json
{ $.eventSource = s3.amazonaws.com && $.eventName = PutBucketPolicy }
```

#### 6. Security Group Changes
```json
{ $.eventName = AuthorizeSecurityGroupIngress || $.eventName = RevokeSecurityGroupIngress }
```

#### 7. Network ACL Changes
```json
{ $.eventName = CreateNetworkAclEntry || $.eventName = DeleteNetworkAclEntry }
```

#### 8. KMS Key Deletion
```json
{ $.eventSource = kms.amazonaws.com && $.eventName = DisableKey || $.eventName = ScheduleKeyDeletion }
```

## Troubleshooting

### Common Issues

#### 1. Log Files Not Appearing
- Check S3 bucket policy allows CloudTrail to write
- Verify trail is started
- Check IAM permissions for CloudTrail service role
- Verify no bucket-level access restrictions

#### 2. Incomplete Logs
- Management events only by default (enable data events if needed)
- Data events require specific resource ARNs
- Some AWS services don't log to CloudTrail (check service documentation)

#### 3. High Costs
- Data events generate high volume
- Consider filtering to specific resources
- Use CloudTrail Lake for long-term retention instead of S3
- Implement S3 lifecycle policies for older logs

#### 4. Delayed Log Delivery
- Typical delivery: 5-15 minutes
- Delays can occur during high API call volumes
- Not suitable for real-time monitoring (use EventBridge instead)

## Best Practices

1. **Enable in All Regions**: Use multi-region trails
2. **Enable Log File Validation**: Ensure integrity
3. **Encrypt Logs**: Use SSE-KMS for sensitive environments
4. **Enable CloudTrail Insights**: Detect anomalies
5. **Integrate with CloudWatch Logs**: Real-time monitoring
6. **Use Organization Trails**: Centralized logging for multi-account
7. **Implement S3 Lifecycle Policies**: Archive old logs to Glacier
8. **Set Up Alerts**: Use EventBridge for critical events
9. **Regular Log Review**: Automated analysis with Athena
10. **Restrict Access**: Principle of least privilege for trail management
11. **Monitor Trail Configuration**: Alert on trail modifications/deletions
12. **Use Separate S3 Bucket**: Dedicated bucket for CloudTrail logs
13. **Enable MFA Delete**: On CloudTrail S3 bucket
14. **Document Trail Purpose**: Tag trails appropriately

## Cost Optimization

### Free Tier
- First trail in each region: Free (management events only)
- Event history in console: 90 days (free)

### Cost Factors
- **Additional Trails**: $2.00 per 100,000 management events
- **Data Events**: $0.10 per 100,000 events
- **Insights Events**: $0.35 per 100,000 events analyzed
- **CloudTrail Lake Ingestion**: ~$2.50 per GB
- **CloudTrail Lake Storage**: ~$0.023 per GB/month
- **S3 Storage**: Standard S3 pricing
- **S3 API Requests**: GET/PUT request charges

### Cost Reduction Strategies
1. **Single Multi-Region Trail**: Instead of multiple trails
2. **Selective Data Events**: Only critical resources
3. **S3 Lifecycle**: Move to Glacier after 90 days
4. **CloudTrail Lake**: For long-term retention vs. S3
5. **Filter Unnecessary Events**: Use advanced event selectors

## Key Exam Points

- CloudTrail logs API calls and is essential for auditing
- First trail per region is free (management events only)
- Management events record control plane operations
- Data events record data plane operations (S3, Lambda, DynamoDB)
- Insights events detect unusual activity patterns
- Multi-region trails recommended for comprehensive coverage
- Organization trails centralize logging across accounts
- Log files delivered to S3 every 5 minutes
- Log file validation ensures integrity with hashing
- Integration with CloudWatch Logs enables real-time monitoring
- EventBridge receives all CloudTrail events for automation
- CloudTrail Lake provides queryable event data store
- Athena can query logs stored in S3
- 90-day event history available in console (free)
- KMS encryption required for sensitive compliance requirements
- Not real-time (5-15 minute delay typical)
- Some AWS services/actions not logged by CloudTrail

## Common Exam Scenarios

### Scenario 1: Audit Root Account Activity
**Solution**: Enable CloudTrail → Stream to CloudWatch Logs → Create metric filter for root account usage → Create CloudWatch alarm → Send to SNS

### Scenario 2: Centralized Logging for Organization
**Solution**: Create organization trail in management account → Configure S3 bucket in log archive account → Apply bucket policy for cross-account access

### Scenario 3: Automated Remediation
**Solution**: CloudTrail → EventBridge rule (on specific API call) → Lambda function (remediate action)

### Scenario 4: Long-term Compliance Retention
**Solution**: CloudTrail → S3 → Lifecycle policy (transition to Glacier after 90 days) or use CloudTrail Lake with 7-year retention

### Scenario 5: Real-time Security Monitoring
**Solution**: CloudTrail → CloudWatch Logs + EventBridge → Lambda/SNS for alerts and automated response

## Related Services

- **AWS Config**: Resource configuration tracking (complements CloudTrail)
- **Amazon GuardDuty**: Uses CloudTrail logs for threat detection
- **Amazon Detective**: Investigates security findings using CloudTrail data
- **AWS Security Hub**: Aggregates findings including CloudTrail insights
- **Amazon Macie**: Uses CloudTrail for data access monitoring
