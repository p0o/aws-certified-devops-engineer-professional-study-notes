# AWS Secrets Manager

AWS Secrets Manager helps you manage, retrieve, and rotate database credentials, API keys, and other secrets throughout their lifecycle.

## Core Concepts

### What is a Secret?
- Encrypted sensitive information (passwords, API keys, tokens, certificates)
- Maximum size: 65,536 bytes
- Stored encrypted using AWS KMS
- Versioned (AWSCURRENT, AWSPENDING, AWSPREVIOUS)
- Can include multiple key-value pairs (JSON structure)

### Secrets Manager vs Parameter Store

| Feature | Secrets Manager | Parameter Store (Advanced) |
|---------|----------------|---------------------------|
| Rotation | Built-in automatic rotation | Manual/custom rotation |
| RDS Integration | Native integration | Manual scripts |
| Cross-account access | Via resource policy | Via resource policy |
| Secret size | 65 KB | 8 KB (standard), 4-8 KB (advanced) |
| Cost | $0.40/secret/month + API calls | Free (standard), $0.05/secret/month (advanced) |
| Versioning | Automatic with labels | Manual version numbers |
| Fine-grained access | Resource-based policies | IAM policies only |

## Secret Storage

### Secret Structure
```json
{
  "username": "admin",
  "password": "MySecurePassword123!",
  "engine": "mysql",
  "host": "mydb.example.com",
  "port": 3306,
  "dbname": "myapp"
}
```

### Encryption
- **At rest**: Encrypted using AWS KMS
- **In transit**: TLS encryption
- **KMS key options**:
  - AWS managed key (`aws/secretsmanager`) - Free
  - Customer managed key - Full control, additional cost
- Can use cross-account KMS keys

### Secret Versioning
- **AWSCURRENT**: Current version in use
- **AWSPENDING**: Version being created during rotation
- **AWSPREVIOUS**: Previous current version (for rollback)
- **Custom labels**: Can add custom version labels
- Version IDs: Unique 32-character string

## Automatic Rotation

### How Rotation Works
1. Secrets Manager calls Lambda rotation function
2. Lambda creates new secret version (AWSPENDING)
3. Lambda tests new credentials
4. Lambda updates AWSCURRENT label
5. AWSPREVIOUS label assigned to old version

### Rotation Strategies

#### Single User Rotation
- Same user credentials rotated
- Brief period of invalid credentials possible
- Simpler, fewer permissions needed
- **Use case**: Application with single DB user

#### Alternating Users Rotation
- Two users alternate (user1 → user2 → user1)
- No credential invalidation period
- Requires two sets of credentials
- **Use case**: High availability requirements

#### Master User Rotation (RDS/Aurora)
- Use master user to rotate app user credentials
- Master credentials remain static
- Most secure option
- **Use case**: Production databases

### Supported Services (Native Rotation)
- Amazon RDS (all engines)
- Amazon DocumentDB
- Amazon Redshift
- Other: Custom Lambda functions

### Custom Rotation
- Write Lambda function with rotation logic
- Must follow Secrets Manager rotation function interface
- Handle four steps:
  - `createSecret`: Create new version
  - `setSecret`: Change the secret
  - `testSecret`: Verify new secret works
  - `finishSecret`: Mark AWSCURRENT

Example Lambda rotation function steps:
```python
def lambda_handler(event, context):
    service_client = boto3.client('secretsmanager')

    # Parse event
    token = event['Token']
    step = event['Step']

    if step == "createSecret":
        create_secret(service_client, event)
    elif step == "setSecret":
        set_secret(service_client, event)
    elif step == "testSecret":
        test_secret(service_client, event)
    elif step == "finishSecret":
        finish_secret(service_client, event)
```

### Rotation Configuration
- **Rotation interval**: 1-365 days
- **Lambda function**: ARN of rotation function
- **Automatic rotation**: On/Off toggle
- **Rotation immediately**: Trigger manual rotation

## Access Control

### IAM Policies
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:region:account:secret:MySecret-*"
    }
  ]
}
```

### Resource-Based Policies
- Attach directly to secret
- Enable cross-account access
- Useful for Lambda functions, ECS tasks

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/MyAppRole"
      },
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "*"
    }
  ]
}
```

### VPC Endpoint
- Private connectivity from VPC
- No internet gateway required
- Endpoints for `secretsmanager` service
- Use VPC endpoint policies for additional control

## Retrieving Secrets

### AWS CLI
```bash
# Get secret value
aws secretsmanager get-secret-value --secret-id MySecret

# Get specific version
aws secretsmanager get-secret-value \
  --secret-id MySecret \
  --version-stage AWSPREVIOUS
```

### AWS SDKs

#### Python (Boto3)
```python
import boto3
import json

client = boto3.client('secretsmanager')
response = client.get_secret_value(SecretId='MySecret')
secret = json.loads(response['SecretString'])

username = secret['username']
password = secret['password']
```

#### Node.js
```javascript
const AWS = require('aws-sdk');
const client = new AWS.SecretsManager();

const data = await client.getSecretValue({ SecretId: 'MySecret' }).promise();
const secret = JSON.parse(data.SecretString);
```

### Caching
- **AWS Secrets Manager Caching Libraries**
  - Python, Java, Go, .NET
  - Reduces API calls and latency
  - Configurable cache TTL
  - Automatic refresh on rotation

```python
from aws_secretsmanager_caching import SecretCache

cache = SecretCache()
secret = cache.get_secret_string('MySecret')
```

## Monitoring & Auditing

### CloudWatch Metrics
- `RotationAttempt`: Rotation initiated
- `RotationSuccess`: Rotation succeeded
- `RotationFailed`: Rotation failed
- Custom metrics via Lambda

### CloudWatch Logs
- Lambda rotation function logs
- Useful for debugging rotation issues

### CloudTrail Events
- All API calls logged
- Monitor:
  - `GetSecretValue`: Secret retrieval
  - `PutSecretValue`: Secret updates
  - `RotateSecret`: Rotation initiation
  - `DeleteSecret`: Secret deletion
- Integrate with CloudWatch Logs for alerts

### EventBridge (CloudWatch Events)
- React to secret rotation events
- Trigger Lambda, SNS, Step Functions
- Use case: Notify on rotation failure

```json
{
  "source": ["aws.secretsmanager"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventName": ["RotateSecret"]
  }
}
```

## Replication

### Multi-Region Secrets
- Replicate secrets to multiple regions
- Automatic synchronization
- Rotation handled in primary region
- Read replicas in secondary regions
- **Use case**: Multi-region applications, disaster recovery

```bash
aws secretsmanager replicate-secret-to-regions \
  --secret-id MySecret \
  --add-replica-regions Region=us-west-2
```

### Cross-Account Access
- Use resource-based policies
- KMS key policy must allow cross-account access
- IAM role assumption for programmatic access

## Secret Deletion

### Deletion Process
- **Recovery window**: 7-30 days (default 30)
- Secret not immediately deleted
- Can cancel deletion during recovery window
- Prevents accidental deletion
- No charges during recovery window

```bash
# Schedule deletion
aws secretsmanager delete-secret \
  --secret-id MySecret \
  --recovery-window-in-days 7

# Cancel deletion
aws secretsmanager restore-secret --secret-id MySecret

# Force immediate deletion (not recommended)
aws secretsmanager delete-secret \
  --secret-id MySecret \
  --force-delete-without-recovery
```

## Integration Patterns

### With RDS/Aurora
- Store database credentials
- Enable automatic rotation
- Rotation Lambda functions provided by AWS
- Applications retrieve credentials dynamically

### With ECS/Fargate
- Task definition references secret ARN
- Injected as environment variables
- IAM task role needs `GetSecretValue` permission

```json
{
  "containerDefinitions": [{
    "secrets": [{
      "name": "DB_PASSWORD",
      "valueFrom": "arn:aws:secretsmanager:region:account:secret:MySecret"
    }]
  }]
}
```

### With Lambda
- Environment variables reference secret
- Or retrieve in function code
- Use caching to reduce latency

### With EC2/On-Premises
- Retrieve via AWS SDK or CLI
- Use instance profile for EC2
- For on-premises: IAM user or assumed role

### With CodeBuild/CodePipeline
- Reference secrets in buildspec.yml
- Environment variables injected during build

```yaml
env:
  secrets-manager:
    API_KEY: MySecret:api_key
```

## Best Practices

1. **Enable Rotation**: Automatically rotate credentials regularly
2. **Use Resource Policies**: For cross-account/cross-service access
3. **VPC Endpoints**: For private connectivity
4. **Audit Access**: Monitor with CloudTrail
5. **Least Privilege**: Grant minimal required permissions
6. **Cache Secrets**: Use caching libraries to reduce API calls
7. **Tag Secrets**: For organization and cost allocation
8. **Replicate Critical Secrets**: For multi-region resilience
9. **Use JSON Structure**: For multiple values in single secret
10. **Test Rotation**: Verify rotation works before production
11. **Monitor Rotation**: Set up alarms for rotation failures
12. **Separate Secrets**: By environment, application
13. **Use Parameter Store**: For non-sensitive configuration
14. **Document Secrets**: Describe secret purpose in metadata

## Cost Optimization

### Pricing
- **Secret storage**: $0.40 per secret per month
- **API calls**: $0.05 per 10,000 API calls
- **Replication**: Additional $0.40 per replica region
- **No charge during deletion recovery window**

### Cost Reduction
1. Use Parameter Store for non-sensitive configuration (free tier)
2. Implement caching to reduce API calls
3. Delete unused secrets
4. Use single secret with JSON for related credentials
5. Consolidate secrets where appropriate

## Common Exam Scenarios

### Scenario 1: RDS Password Rotation
**Solution**: Store RDS credentials in Secrets Manager, enable automatic rotation with AWS-provided Lambda function

### Scenario 2: ECS Task Needs DB Password
**Solution**: Reference secret ARN in task definition `secrets` section, grant task role `GetSecretValue` permission

### Scenario 3: Multi-Region Application
**Solution**: Replicate secret to all application regions, applications read from local region

### Scenario 4: Cross-Account Secret Access
**Solution**: Add resource-based policy to secret, configure KMS key policy for cross-account access

### Scenario 5: Audit Secret Access
**Solution**: Enable CloudTrail, create CloudWatch Logs metric filter for `GetSecretValue`, create alarm

### Scenario 6: Zero-Downtime Rotation
**Solution**: Use alternating users rotation strategy with two database users

## Key Exam Points

- Secrets Manager manages, retrieves, and rotates secrets automatically
- Integrates natively with RDS, Aurora, DocumentDB, Redshift
- Rotation performed by Lambda functions
- Secrets encrypted with KMS (at rest) and TLS (in transit)
- Version labels: AWSCURRENT, AWSPENDING, AWSPREVIOUS
- Maximum secret size: 65 KB
- Deletion has recovery window (7-30 days)
- Can replicate secrets across regions
- Resource-based policies enable cross-account access
- VPC endpoints for private connectivity
- All API calls logged to CloudTrail
- Caching libraries available to reduce API calls and latency
- ECS can inject secrets as environment variables
- Cost: $0.40/secret/month + API call charges
- Rotation strategies: single user, alternating users, master user
- CloudWatch Events/EventBridge for rotation monitoring
- Supports JSON structure for multiple key-value pairs

## Related Services

- **AWS Systems Manager Parameter Store**: Alternative for simpler secrets/configuration
- **AWS KMS**: Encryption key management
- **Amazon RDS**: Database credential rotation
- **AWS Lambda**: Custom rotation functions
- **Amazon ECS**: Secret injection into containers
- **CloudTrail**: API audit logging
- **CloudWatch**: Monitoring and alarms
- **VPC**: Private endpoints
- **IAM**: Access control
