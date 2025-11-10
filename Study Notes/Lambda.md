# AWS Lambda

AWS Lambda is a serverless compute service that runs code in response to events and automatically manages the underlying compute resources.

## Core Concepts

### Function Basics
- **Event-driven execution**: Triggered by events from AWS services or custom applications
- **Stateless**: Each invocation is independent
- **Automatic scaling**: Scales automatically with request volume
- **Pay-per-use**: Billed based on number of requests and compute time
- **No server management**: AWS handles infrastructure

### Execution Environment
- **Runtime**: Language/version for function code
  - Node.js, Python, Ruby, Java, Go, .NET, Custom Runtime (via Lambda Runtime API)
- **Handler**: Entry point method for function
- **Memory**: 128 MB to 10,240 MB (10 GB) in 1 MB increments
- **Ephemeral storage (/tmp)**: 512 MB to 10,240 MB
- **Timeout**: 1 second to 15 minutes (900 seconds)
- **vCPU**: Proportional to memory allocation

## Function Configuration

### Basic Settings
```python
# Example handler (Python)
def lambda_handler(event, context):
    # event: Input data
    # context: Runtime information
    return {
        'statusCode': 200,
        'body': 'Hello from Lambda!'
    }
```

### Environment Variables
- Key-value pairs available to function code
- Can be encrypted with KMS
- Maximum 4 KB total size
- Useful for configuration without code changes

### Execution Role (IAM Role)
- Grants Lambda permission to AWS services
- Must have `lambda.amazonaws.com` as trusted entity
- Attach policies for required permissions
- Automatically includes CloudWatch Logs permissions

### Resource-Based Policy
- Controls which services/accounts can invoke function
- Example: Allow S3 bucket to invoke function
```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "s3.amazonaws.com"
  },
  "Action": "lambda:InvokeFunction",
  "Resource": "arn:aws:lambda:region:account:function:name",
  "Condition": {
    "ArnLike": {
      "AWS:SourceArn": "arn:aws:s3:::my-bucket"
    }
  }
}
```

## Invocation Types

### Synchronous Invocation
- Caller waits for response
- Use cases: API Gateway, ALB, CLI, SDK
- Error handling: Caller receives error
- No built-in retry

### Asynchronous Invocation
- Lambda queues event and returns immediately
- Built-in retry: 2 attempts (0 → 1 min → 2 min delays)
- Use cases: S3, SNS, EventBridge, SES
- Dead Letter Queue (DLQ): SQS or SNS for failed events
- Destinations: On success or failure → Lambda, SQS, SNS, EventBridge

### Event Source Mapping (Poll-Based)
- Lambda polls source for records
- Use cases: Kinesis, DynamoDB Streams, SQS, MQ
- **Batch processing**: Process multiple records per invocation
- **Concurrency control**: Number of concurrent functions
- **Error handling**:
  - Streams (Kinesis/DynamoDB): Block shard until success or records expire
  - Queues (SQS): Return to queue, eventual move to DLQ
- **Partial batch failure**: Report individual failed records (Lambda returns failed item IDs)

## Deployment & Versioning

### Versions
- **$LATEST**: Mutable, latest code
- **Numbered versions**: Immutable snapshots (1, 2, 3, etc.)
- Each version has unique ARN
- Use for production stability

### Aliases
- Pointers to versions
- Mutable (can update to point to different version)
- Have their own ARN
- Use cases:
  - Environment stages (dev, staging, prod)
  - Blue/green deployments
  - Canary deployments
- **Weighted aliases**: Traffic shifting between versions
  ```
  Alias: prod
  - Version 1: 90%
  - Version 2: 10%
  ```

### Deployment Methods

#### All-at-once
- Immediate shift to new version
- Highest risk, fastest deployment

#### Linear
- Traffic shifts gradually in equal increments
- `Linear10PercentEvery1Minute`: 10% every minute
- `Linear10PercentEvery10Minutes`: 10% every 10 minutes

#### Canary
- Small percentage first, then remainder
- `Canary10Percent5Minutes`: 10% for 5 minutes, then 100%
- `Canary10Percent30Minutes`: 10% for 30 minutes, then 100%

### CodeDeploy Integration
- Automated traffic shifting
- Automatic rollback on alarms
- Pre-traffic and post-traffic hooks
- Integrated with CloudWatch alarms

## Concurrency

### Types
- **Account Concurrency Limit**: 1,000 concurrent executions per region (soft limit)
- **Reserved Concurrency**: Dedicated capacity for function (max concurrent executions)
- **Provisioned Concurrency**: Pre-warmed instances to avoid cold starts

### Reserved Concurrency
- Guarantees function won't be throttled below limit
- Reduces available concurrency for other functions
- Use case: Critical functions needing guaranteed capacity
- Can set to 0 to disable function

### Provisioned Concurrency
- Keeps functions initialized and ready
- Eliminates cold start latency
- Billed for configured concurrency (not actual usage)
- Can use Application Auto Scaling for dynamic adjustment
- Use case: Latency-sensitive applications

### Throttling
- When concurrent executions exceed limit
- **Synchronous**: Returns 429 TooManyRequestsException
- **Asynchronous**: Event goes to internal queue (6 hours max)
- **Poll-based**: Lambda slows down polling

## Performance Optimization

### Cold Starts
- First invocation or after idle period
- Includes:
  - Download code
  - Start new execution environment
  - Run initialization code
- **Mitigation**:
  - Provisioned concurrency
  - Minimize deployment package size
  - Minimize initialization code
  - Keep functions warm (scheduled CloudWatch Events)

### Memory & CPU
- CPU allocation scales with memory
- More memory = more CPU power
- **Power tuning**: Find optimal memory/cost ratio
- Use AWS Lambda Power Tuning tool

### VPC Configuration
- **VPC**: Access private resources (RDS, ElastiCache)
- **ENI attachment**: Occurs during initialization
- **Hyperplane ENI**: Shared ENI (fast, no cold start impact)
- Requires:
  - Subnets
  - Security groups
  - NAT Gateway for internet access
- Best practice: Only use VPC when necessary

### Layers
- Reusable code packages
- Share libraries, dependencies, custom runtimes
- Reduces deployment package size
- Maximum 5 layers per function
- Maximum 250 MB total (unzipped)
- Can share layers across accounts

### Extensions
- External code runs in Lambda execution environment
- Use cases: Monitoring, security, governance tools
- Types:
  - **Internal extensions**: In-process with function
  - **External extensions**: Separate process
- Examples: AWS AppConfig, AWS Secrets Manager caching

## Event Sources & Integrations

### Direct Invocation
- API Gateway (REST/HTTP API)
- Application Load Balancer (ALB)
- Lambda Function URLs (built-in HTTP endpoint)
- AWS SDKs / CLI
- Step Functions

### Asynchronous Sources
- S3 (object events)
- SNS (notifications)
- EventBridge (scheduled or event patterns)
- SES (email events)
- CloudFormation (custom resources)
- AWS Config (rule evaluation)

### Stream/Poll-Based Sources
- Kinesis Data Streams
- DynamoDB Streams
- SQS (standard and FIFO)
- Amazon MQ
- Managed Streaming for Apache Kafka (MSK)
- Self-managed Apache Kafka

### Integration Patterns

#### API Gateway + Lambda
- RESTful APIs
- Proxy integration or custom integration
- Request/response transformation
- Authentication & authorization
- Throttling & caching

#### ALB + Lambda
- Multi-value headers
- Target group health checks
- Good for existing ALB infrastructure

#### S3 + Lambda
- Trigger on object creation, deletion
- Use case: Image processing, data transformation
- Event notification configuration

#### DynamoDB Streams + Lambda
- React to table changes
- Event types: INSERT, MODIFY, REMOVE
- Ordered stream of item-level modifications

## Monitoring & Debugging

### CloudWatch Metrics
- **Invocations**: Number of times invoked
- **Duration**: Execution time
- **Errors**: Number of failed invocations
- **Throttles**: Number of throttled invocations
- **ConcurrentExecutions**: Concurrent executions at time
- **IteratorAge**: Age of last record (stream sources)
- **DeadLetterErrors**: Failed to send to DLQ

### CloudWatch Logs
- Automatic log group creation: `/aws/lambda/<function-name>`
- Log retention configurable
- Contains:
  - START, END, REPORT (execution details)
  - Console output (print/console.log)
  - Errors and stack traces
- **CloudWatch Logs Insights**: Query logs

### AWS X-Ray
- Distributed tracing
- Trace requests through application
- Visualize service map
- Identify performance bottlenecks
- Enable via function configuration
- Requires X-Ray SDK in code
- Automatic for SDK service calls

### Lambda Insights
- CloudWatch dashboard for Lambda
- System-level metrics (CPU, memory, network)
- Diagnostic information (cold starts, worker shutdowns)
- Enabled via Lambda layer

### Error Handling
```python
# Custom error handling
def lambda_handler(event, context):
    try:
        # Function logic
        result = process_data(event)
        return {
            'statusCode': 200,
            'body': result
        }
    except ValueError as e:
        # Log error
        print(f"ValueError: {str(e)}")
        return {
            'statusCode': 400,
            'body': f"Bad request: {str(e)}"
        }
    except Exception as e:
        # Log unexpected errors
        print(f"Unexpected error: {str(e)}")
        # Re-raise for Lambda to handle
        raise
```

## Security Best Practices

1. **Least Privilege IAM**: Grant only required permissions
2. **Environment Variable Encryption**: Use KMS for sensitive data
3. **Secrets Manager/Parameter Store**: For passwords, API keys
4. **VPC when needed**: Access private resources securely
5. **Resource-based policies**: Control who can invoke
6. **Code signing**: Verify code integrity
7. **Function policies**: Use conditions (IP, VPC, source ARN)
8. **Enable X-Ray**: Trace and audit requests
9. **CloudTrail logging**: Track Lambda API calls
10. **Rotate credentials**: For any embedded secrets

## Cost Optimization

### Pricing Components
- **Requests**: $0.20 per 1M requests
- **Duration**: GB-second of compute time
  - First 400,000 GB-seconds per month free
  - $0.0000166667 per GB-second thereafter
- **Provisioned concurrency**: Hourly charge
- **Data transfer**: Standard AWS data transfer rates

### Optimization Strategies
1. **Right-size memory**: Use Power Tuning tool
2. **Reduce timeout**: Set realistic timeout values
3. **Optimize cold starts**: Reduce package size, use layers
4. **Use ARM (Graviton2)**: Up to 34% better price-performance
5. **Avoid provisioned concurrency**: Unless absolutely needed
6. **Efficient code**: Minimize execution time
7. **Reuse connections**: Database, HTTP clients
8. **Use appropriate triggers**: Poll-based vs. event-based

## Common Exam Scenarios

### Scenario 1: High Latency (Cold Starts)
**Solution**: Provisioned concurrency, optimize initialization, reduce package size, consider keeping warm with CloudWatch Events

### Scenario 2: Function Throttling
**Solution**: Increase account limit, use reserved concurrency, implement retry logic with exponential backoff, use SQS as buffer

### Scenario 3: VPC Timeout Issues
**Solution**: Check security groups, route tables, NAT Gateway for internet access, verify ENI limits not exceeded

### Scenario 4: Failed Asynchronous Invocations
**Solution**: Configure DLQ (SQS/SNS) or destinations, increase retries if applicable, check CloudWatch Logs for errors

### Scenario 5: Long-Running Tasks
**Solution**: Use Step Functions for orchestration, break into smaller functions, consider ECS/Fargate for >15 min tasks

### Scenario 6: Access to RDS
**Solution**: Place Lambda in same VPC as RDS, configure security groups, use RDS Proxy for connection pooling

## Best Practices

1. **Single Responsibility**: One function, one purpose
2. **Idempotency**: Function should produce same result for same input
3. **Stateless Design**: Use external storage (S3, DynamoDB)
4. **Environment Variables**: For configuration
5. **Connection Reusing**: Initialize outside handler
6. **Error Handling**: Implement proper try/catch and logging
7. **Monitoring**: Use CloudWatch and X-Ray
8. **Testing**: Unit tests, integration tests
9. **CI/CD**: Automate deployment (CodePipeline, SAM, CDK)
10. **Versions & Aliases**: For safe production deployments

## Infrastructure as Code

### AWS SAM (Serverless Application Model)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: python3.9
      CodeUri: ./src
      MemorySize: 512
      Timeout: 30
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

### AWS CDK
```typescript
const fn = new lambda.Function(this, 'MyFunction', {
  runtime: lambda.Runtime.NODEJS_18_X,
  handler: 'index.handler',
  code: lambda.Code.fromAsset('lambda'),
  memorySize: 512,
  timeout: Duration.seconds(30),
  environment: {
    TABLE_NAME: table.tableName
  }
});
```

## Key Exam Points

- Lambda is serverless, event-driven, and automatically scales
- Maximum timeout: 15 minutes
- Memory: 128 MB to 10 GB (vCPU scales with memory)
- Three invocation types: synchronous, asynchronous, poll-based
- Asynchronous invocations retry twice automatically
- Versions are immutable, aliases are mutable pointers
- Reserved concurrency guarantees capacity; provisioned concurrency pre-warms instances
- Account limit: 1,000 concurrent executions per region (soft limit)
- Cold starts occur on first invocation or after idle period
- VPC configuration allows access to private resources
- Layers enable code reuse and reduce deployment package size
- X-Ray provides distributed tracing
- CloudWatch Logs automatic for all executions
- Can use DLQ (SQS/SNS) or Destinations for failure handling
- CodeDeploy supports traffic shifting and automated rollback
- Function URLs provide built-in HTTP endpoints
- RDS Proxy recommended for database connections
- ARM (Graviton2) offers better price-performance
- Lambda@Edge runs at CloudFront edge locations

## Related Services

- **API Gateway**: RESTful API frontend
- **Step Functions**: Orchestration of Lambda functions
- **EventBridge**: Event bus for triggering Lambda
- **SNS/SQS**: Asynchronous messaging
- **DynamoDB**: NoSQL database (common with Lambda)
- **S3**: Object storage and event source
- **CodePipeline/CodeDeploy**: CI/CD for Lambda
- **SAM/CDK**: Infrastructure as code
- **CloudWatch**: Monitoring and logging
- **X-Ray**: Distributed tracing
