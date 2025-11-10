# Amazon CloudWatch

Amazon CloudWatch is a monitoring and observability service that provides data and actionable insights for AWS resources, applications, and services running on AWS and on-premises.

## Core Components

### Metrics
- Time-ordered set of data points
- Represented by timestamp, value, and optional unit
- **Namespaces**: Container for CloudWatch metrics (e.g., `AWS/EC2`, `AWS/RDS`)
- **Dimensions**: Name/value pairs that identify metrics (e.g., InstanceId, DBInstanceIdentifier)

#### Standard vs Detailed Monitoring
- **Basic Monitoring**: 5-minute intervals (free for most services)
- **Detailed Monitoring**: 1-minute intervals (additional cost)
- EC2: Basic = 5 min, Detailed = 1 min
- EBS: Automatically enabled at 1 min (free)
- ELB: 1-minute metrics (free)

#### Metric Resolution
- **Standard Resolution**: 1-minute granularity
- **High Resolution**: Up to 1-second granularity
- Can store for different periods:
  - <60 seconds: 3 hours
  - 60 seconds (1 minute): 15 days
  - 300 seconds (5 minutes): 63 days
  - 3600 seconds (1 hour): 455 days

### Custom Metrics
- Publish your own metrics using PutMetricData API
- Use CloudWatch agent or SDKs
- Can include multiple dimensions
- Supports high-resolution metrics (1-second intervals)

```bash
aws cloudwatch put-metric-data \
  --namespace "MyApp" \
  --metric-name "PageViewCount" \
  --value 10 \
  --dimensions Site=example.com
```

### Statistics
- **Average**: Mean of values
- **Sum**: Total of values
- **Minimum**: Lowest value
- **Maximum**: Highest value
- **SampleCount**: Number of data points
- **Percentile**: p99, p95, p50, etc.

## CloudWatch Logs

### Log Structure
- **Log Groups**: Container for log streams (e.g., `/aws/lambda/my-function`)
- **Log Streams**: Sequence of log events from the same source
- **Log Events**: Record of activity with timestamp and message

### Log Retention
- Default: Never expire
- Configurable: 1 day to 10 years
- Impacts storage costs

### Log Ingestion Methods
1. **CloudWatch Logs Agent** (older)
2. **CloudWatch Unified Agent** (recommended)
   - Supports both logs and custom metrics
   - Additional system metrics (memory, disk, etc.)
3. **AWS SDKs**
4. **VPC Flow Logs**
5. **CloudTrail**
6. **Elastic Beanstalk**
7. **ECS Container Logs**
8. **Lambda Logs**
9. **API Gateway Logs**
10. **RDS Logs**

### Log Insights
- Interactive log analytics service
- Query log data using SQL-like syntax
- Visualize results with charts
- Use for troubleshooting and analysis

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

### Metric Filters
- Extract metric data from logs
- Pattern matching on log events
- Create CloudWatch metrics from log data
- Example: Count occurrences of "ERROR" in logs

```json
{
  "filterPattern": "[ERROR]",
  "metricNamespace": "MyApp",
  "metricName": "ErrorCount",
  "metricValue": "1"
}
```

### Log Subscriptions
- Real-time log event delivery
- Destinations:
  - **Kinesis Data Streams**: Real-time processing
  - **Kinesis Data Firehose**: Delivery to S3, Redshift, OpenSearch
  - **Lambda**: Custom processing
- Cross-account subscriptions supported
- Subscription filters define which log events to send

### Log Exports
- Export log data to S3
- Asynchronous process (up to 12 hours)
- Useful for archival and analysis with Athena
- Can encrypt with SSE-S3 or SSE-KMS

## CloudWatch Alarms

### Alarm States
- **OK**: Metric is within threshold
- **ALARM**: Metric breached threshold
- **INSUFFICIENT_DATA**: Not enough data to determine state

### Alarm Types
1. **Metric Alarms**: Based on single metric or math expression
2. **Composite Alarms**: Combine multiple alarms with AND/OR logic
3. **Anomaly Detection Alarms**: ML-based threshold detection

### Alarm Actions
- **SNS Notifications**: Send alerts
- **Auto Scaling Actions**: Scale EC2/ECS resources
- **EC2 Actions**: Stop, terminate, reboot, recover instances
- **Systems Manager Actions**: Trigger automation

### Alarm Configuration
```yaml
AlarmName: HighCPUAlarm
MetricName: CPUUtilization
Namespace: AWS/EC2
Statistic: Average
Period: 300  # 5 minutes
EvaluationPeriods: 2
Threshold: 80
ComparisonOperator: GreaterThanThreshold
TreatMissingData: notBreaching
```

### Evaluation Settings
- **Period**: Time interval for metric evaluation (60 seconds to 1 day)
- **Evaluation Periods**: Number of periods to evaluate
- **Datapoints to Alarm**: How many datapoints must breach (M out of N)
- **Treat Missing Data**:
  - notBreaching (default)
  - breaching
  - ignore
  - missing

### Composite Alarms
- Reduce alarm noise
- Create complex alarm logic
- Example: Alarm only if CPU high AND disk full

```yaml
CompositeAlarm:
  AlarmRule: "ALARM(HighCPU) AND ALARM(HighDisk)"
  ActionsEnabled: true
  AlarmActions:
    - !Ref MySNSTopic
```

### Anomaly Detection
- Machine learning models learn metric behavior
- Automatically adjusts thresholds
- Useful for metrics with patterns or trends
- Band thickness configurable (1-10, default 2)

## CloudWatch Agent

### Installation
- Available for EC2, on-premises servers
- Download from S3 or Systems Manager
- Configure via JSON or wizard

### Configuration
- Stored in Parameter Store or local file
- Defines metrics and logs to collect
- Can collect:
  - System metrics (memory, disk, network)
  - Custom application metrics
  - Log files
  - Windows event logs

```json
{
  "metrics": {
    "namespace": "CWAgent",
    "metrics_collected": {
      "mem": {
        "measurement": [
          {"name": "mem_used_percent"}
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [
          {"name": "used_percent"}
        ],
        "metrics_collection_interval": 60
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/app.log",
            "log_group_name": "/aws/ec2/app",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```

### Unified Agent Benefits
- Single agent for both metrics and logs
- More system-level metrics than default EC2 monitoring
- procstat plugin for process-level metrics
- StatsD and collectd support

## CloudWatch Dashboards

- Visual representation of metrics and alarms
- Customizable widgets:
  - Line graphs
  - Stacked area charts
  - Number displays
  - Text widgets (Markdown)
  - Query results from Logs Insights
- Can be shared across accounts
- Automatic dashboards for AWS services
- Maximum 500 metrics per dashboard

## CloudWatch Events / EventBridge

**Note**: CloudWatch Events is now part of Amazon EventBridge

### Event Sources
- AWS services (EC2 state changes, CodePipeline events, etc.)
- Scheduled events (cron expressions)
- Custom applications (PutEvents API)

### Event Targets
- Lambda functions
- SNS topics
- SQS queues
- Kinesis streams
- Step Functions
- Systems Manager Automation
- EC2 actions
- ECS tasks
- CodePipeline/CodeBuild

### Event Patterns
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}
```

## Container Insights

- Monitoring for containerized applications
- Supports ECS, EKS, Kubernetes on EC2
- Collects metrics and logs
- **Metrics collected**:
  - CPU and memory utilization
  - Task/pod counts
  - Network metrics
- Requires CloudWatch agent as DaemonSet (EKS) or sidecar (ECS)

### Performance Data
- **Performance Log Events**: JSON format logs with embedded metrics
- Automatic dashboards
- Logs Insights queries for troubleshooting

## Lambda Insights

- Monitoring for AWS Lambda functions
- System and application-level metrics
- Automatically collects:
  - Cold starts
  - CPU time
  - Memory usage
  - Network activity
- Enabled via Lambda layer

## Application Insights

- Automated dashboard for applications
- Supports common application stacks:
  - .NET on IIS
  - Java applications
  - Databases (SQL Server, MySQL, PostgreSQL)
- Detects and monitors application components
- Creates alarms automatically

## CloudWatch Contributor Insights

- Analyze high-cardinality data
- Identify top contributors to network traffic, errors, etc.
- Uses log data from VPC Flow Logs, CloudTrail, etc.
- Built-in rules for common use cases
- Custom rules with JSON

## CloudWatch Synthetics

- Canary monitoring for endpoints
- Proactive monitoring of endpoints
- **Canary Types**:
  - Heartbeat: Endpoint availability
  - API Canary: REST API monitoring
  - Broken Link Checker: Find broken links
  - Visual Monitoring: Screenshot comparison
  - GUI Workflow: Multi-step user flows
- Uses Node.js or Python scripts
- Runs on schedule
- Integrates with CloudWatch alarms

## Cross-Account Observability

- New feature for unified monitoring
- Central monitoring account
- Source accounts share data
- Unified dashboards across accounts
- Useful for organizations with multiple AWS accounts

## Security & Encryption

### Encryption
- **Logs**: Encrypted with AES-256 at rest
- **KMS Integration**: Use customer managed keys for log groups
- **In-transit**: TLS encryption

### IAM Permissions
- Control access to CloudWatch resources
- Resource-based policies for log groups
- Service roles for CloudWatch to access other services

## Pricing Considerations

### Metrics
- First 10 custom metrics: Free
- Custom metrics: $0.30 per metric/month
- High-resolution metrics: Additional cost
- API requests: $0.01 per 1,000 requests

### Logs
- Ingestion: $0.50 per GB
- Storage: $0.03 per GB/month
- Insights queries: $0.005 per GB scanned

### Alarms
- Standard alarms: $0.10 per alarm/month
- High-resolution alarms: $0.30 per alarm/month
- Composite alarms: $0.50 per alarm/month

### Dashboards
- First 3 dashboards: Free (up to 50 metrics each)
- Additional dashboards: $3.00 per dashboard/month

## Best Practices

1. **Use Metric Math**: Combine metrics for advanced analysis
2. **Implement Composite Alarms**: Reduce false positives
3. **Enable Detailed Monitoring**: For critical resources
4. **Use Log Retention Policies**: Control costs
5. **Leverage Anomaly Detection**: For dynamic workloads
6. **Create Custom Dashboards**: For operational visibility
7. **Use Logs Insights**: For log analysis vs. exporting to S3
8. **Implement Log Subscription Filters**: For real-time processing
9. **Tag Resources**: For cost allocation and organization
10. **Use CloudWatch Agent**: For enhanced EC2 monitoring

## Common Exam Scenarios

### Scenario 1: High Memory Usage Monitoring
**Problem**: Need to monitor EC2 memory usage
**Solution**: CloudWatch Agent with custom metrics (memory not available by default)

### Scenario 2: Log Aggregation
**Problem**: Centralize logs from multiple sources
**Solution**: CloudWatch Logs with Unified Agent, use log groups for organization

### Scenario 3: Automatic Remediation
**Problem**: Auto-restart failed application
**Solution**: CloudWatch Alarm → SNS → Lambda or Systems Manager Automation

### Scenario 4: Cost Optimization
**Problem**: CloudWatch costs too high
**Solution**:
- Adjust log retention policies
- Use metric filters instead of storing all logs
- Archive old logs to S3
- Use Logs Insights instead of exporting for analysis

### Scenario 5: Multi-Region Monitoring
**Problem**: Monitor resources across regions
**Solution**: Cross-region dashboards, EventBridge event aggregation to central region

## Key Exam Points

- Default EC2 monitoring is 5-minute intervals; detailed is 1-minute
- Memory, disk usage require CloudWatch Agent
- Logs Insights uses query language for analysis
- Metric filters create metrics from log patterns
- Alarms can trigger Auto Scaling, SNS, EC2 actions, Systems Manager
- Composite alarms reduce alarm noise
- Anomaly detection uses ML for dynamic thresholds
- High-resolution metrics support 1-second granularity
- Log retention can be 1 day to 10 years or never expire
- CloudWatch Events is now EventBridge
- Container Insights for ECS/EKS monitoring
- Cross-account log subscriptions supported
- Logs can be exported to S3 or streamed to Kinesis
