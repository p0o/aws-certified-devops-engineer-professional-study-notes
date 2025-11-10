# AWS X-Ray

AWS X-Ray helps developers analyze and debug distributed applications by providing end-to-end tracing and service maps for requests flowing through the application.

## Core Concepts

### Distributed Tracing
- Track requests across multiple services
- Identify performance bottlenecks
- Understand service dependencies
- Analyze errors and exceptions
- Measure latency at each hop

### Segments
- Data about work done by application
- Represents single service or resource
- Contains timing data, metadata, subsegments
- Automatically created for AWS services (Lambda, API Gateway, etc.)

### Subsegments
- More granular view of work within segment
- Can represent:
  - Downstream calls to AWS services
  - HTTP requests
  - Database queries
  - Custom subsegments

### Traces
- Collection of all segments for single request
- End-to-end view through application
- Unique Trace ID tracks request flow
- Can span multiple services and regions

### Service Map
- Visual representation of application architecture
- Shows service dependencies
- Color-coded by health (green, orange, red)
- Displays latency and request rate
- Automatically generated from traces

### Annotations
- Key-value pairs indexed for searching
- **Indexed**: Can filter traces by annotations
- Simple data types (string, number, boolean)
- Maximum 50 annotations per segment
- **Use case**: Environment, version, user ID

### Metadata
- Key-value pairs NOT indexed
- Can store complex data structures
- Not searchable/filterable
- Used for detailed debugging information
- **Use case**: Request/response bodies, stack traces

## X-Ray Daemon

### Overview
- Local process that listens for UDP traffic
- Buffers segments and sends to X-Ray API
- Required for applications to send traces
- Available for Linux, Windows, macOS
- Runs on EC2, ECS, Elastic Beanstalk, Lambda (built-in)

### Installation
```bash
# Download and install (Amazon Linux 2)
wget https://s3.us-east-1.amazonaws.com/aws-xray-assets.us-east-1/xray-daemon/aws-xray-daemon-3.x.rpm
sudo yum install -y ./aws-xray-daemon-3.x.rpm

# Start daemon
sudo systemctl start xray
```

### Configuration
- Default port: 2000 (UDP)
- Configuration via command-line flags or config file
- IAM permissions required:
  - `xray:PutTraceSegments`
  - `xray:PutTelemetryRecords`

## Instrumentation

### SDK Support
- **Java**: X-Ray SDK for Java
- **Node.js**: X-Ray SDK for Node.js
- **Python**: X-Ray SDK for Python
- **.NET**: X-Ray SDK for .NET
- **Go**: X-Ray SDK for Go
- **Ruby**: X-Ray SDK for Ruby

### Automatic Instrumentation

#### Lambda Functions
- Built-in X-Ray daemon
- Enable active tracing in console/CLI
- No daemon installation needed
- SDK for custom subsegments

```python
# Python Lambda with X-Ray
from aws_xray_sdk.core import xray_recorder

@xray_recorder.capture('custom_subsegment')
def my_function(event, context):
    # Function logic
    return response
```

#### API Gateway
- Enable X-Ray tracing in stage settings
- Traces include API Gateway latency
- Shows integration with backend services

#### ECS/Fargate
- Run X-Ray daemon as sidecar container
- Task definition includes daemon container
- Application container sends to daemon

```json
{
  "name": "xray-daemon",
  "image": "amazon/aws-xray-daemon",
  "cpu": 32,
  "memoryReservation": 256,
  "portMappings": [
    {
      "containerPort": 2000,
      "protocol": "udp"
    }
  ]
}
```

#### Elastic Beanstalk
- Enable in console or `.ebextensions`
- X-Ray daemon pre-installed
- Automatic IAM role configuration

```yaml
# .ebextensions/xray-daemon.config
option_settings:
  aws:elasticbeanstalk:xray:
    XRayEnabled: true
```

#### EC2
- Install X-Ray daemon manually
- Configure IAM instance profile with X-Ray permissions
- Application sends traces to daemon

### Manual Instrumentation

#### Express (Node.js)
```javascript
const AWSXRay = require('aws-xray-sdk');
const express = require('express');
const app = express();

// Automatically trace all incoming requests
app.use(AWSXRay.express.openSegment('MyApp'));

app.get('/api/endpoint', (req, res) => {
  // Business logic
  res.json({ message: 'Hello' });
});

app.use(AWSXRay.express.closeSegment());
```

#### Flask (Python)
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

app = Flask(__name__)
xray_recorder.configure(service='MyFlaskApp')
XRayMiddleware(app, xray_recorder)
```

### Capturing AWS SDK Calls
```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# Patch all supported libraries
patch_all()

# Now all AWS SDK calls are automatically traced
import boto3
s3 = boto3.client('s3')
s3.list_buckets()  # This call will appear in X-Ray
```

## Sampling

### Why Sampling?
- Reduce cost and overhead
- Capture representative sample of requests
- Maintain meaningful trace data
- Default: 1 request/second + 5% of additional requests

### Sampling Rules
- Define which requests to trace
- Match based on:
  - Service name
  - HTTP method
  - URL path
  - Host
- **Reservoir**: Fixed number of requests per second
- **Rate**: Percentage of additional requests

```json
{
  "version": 2,
  "rules": [
    {
      "description": "High priority endpoints",
      "service_name": "*",
      "http_method": "*",
      "url_path": "/api/critical/*",
      "fixed_target": 10,
      "rate": 1.0
    },
    {
      "description": "Default sampling",
      "service_name": "*",
      "http_method": "*",
      "url_path": "*",
      "fixed_target": 1,
      "rate": 0.05
    }
  ],
  "default": {
    "fixed_target": 1,
    "rate": 0.05
  }
}
```

### Centralized Sampling Rules
- Managed in X-Ray console
- Applied to all instrumented applications
- No code changes required
- Dynamic updates without redeployment

## Service Map

### Features
- Visual representation of architecture
- Node for each service
- Edges show request flow
- Color-coded health indicators:
  - **Green**: Healthy (< 5% errors)
  - **Orange**: Elevated errors (5-20%)
  - **Red**: High errors (> 20%)
- Response time histograms
- Request rate per service
- Identifies upstream/downstream dependencies

### Service Map Analysis
- Click nodes for detailed traces
- Filter by time range
- Group by custom annotations
- Export as image

## Trace Analysis

### Trace Timeline
- Visualize request flow through services
- See timing for each segment/subsegment
- Identify slowest operations
- Drill down into errors

### Filter Expressions
```
service("my-service") AND http.status = 500
duration > 5
annotation.user_id = "12345"
response_time > 2
fault = true
error = true
```

### Groups
- Create trace groups with filter expressions
- Monitor specific subsets of traffic
- Custom CloudWatch metrics per group
- **Use case**: Track specific customer, feature, or environment

## Analytics

### Insights
- Machine learning-powered anomaly detection
- Identifies unusual patterns:
  - Latency spikes
  - Error rate increases
  - Fault patterns
- Root cause analysis
- Impact timeline

### Queries
- SQL-like language for trace analysis
- Query against trace data
- Aggregate metrics
- Export results

```sql
SELECT service.name, AVG(duration)
FROM traces
WHERE trace.id = "1-5e9a9e9e-1234567890abcdef"
GROUP BY service.name
```

## Integration with AWS Services

### Application Load Balancer
- Propagates X-Ray trace header
- Enables end-to-end tracing through ALB
- Add custom trace header: `X-Amzn-Trace-Id`

### SNS & SQS
- Automatic trace context propagation
- Trace messages through queues/topics
- See latency at each hop

### Step Functions
- Integrated X-Ray tracing
- Trace across state machine executions
- See timing for each state transition

### AWS AppSync
- Enable tracing in AppSync console
- Trace GraphQL resolvers
- See backend data source latency

## Monitoring & Alerting

### CloudWatch Integration
- X-Ray publishes metrics to CloudWatch
- Metrics per service:
  - Call count
  - Error count
  - Fault count
  - Duration (p50, p90, p99)
- Create CloudWatch alarms

### EventBridge
- Limited native integration
- Use Lambda to process traces and generate events
- **Use case**: Alert on specific trace patterns

## Security

### IAM Permissions
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "xray:PutTraceSegments",
        "xray:PutTelemetryRecords"
      ],
      "Resource": "*"
    }
  ]
}
```

### Encryption
- Data encrypted in transit (TLS)
- Data encrypted at rest
- Uses AWS KMS
- 30-day retention period (cannot be changed)

### IAM Resource Policies
- Control access to X-Ray data
- Cross-account access for service maps
- Restrict by service, HTTP method, etc.

## Best Practices

1. **Use Annotations**: For searchable metadata (user ID, tenant ID)
2. **Use Metadata**: For detailed, non-searchable data
3. **Configure Sampling**: Balance cost and visibility
4. **Create Groups**: Monitor specific traffic patterns
5. **Enable on All Services**: Comprehensive view
6. **Instrument Custom Code**: Use subsegments
7. **Monitor Service Map**: Identify dependencies
8. **Set Up Alarms**: For error rates and latency
9. **Use Filter Expressions**: Quickly find problematic traces
10. **Regularly Review Insights**: Catch anomalies early
11. **Propagate Trace Context**: For async operations
12. **Tag Segments**: With environment, version information

## Cost Considerations

### Pricing
- **Traces recorded**: $5.00 per 1 million traces
- **Traces retrieved/scanned**: $0.50 per 1 million traces
- **Insights**: Additional $0.005 per trace analyzed
- **Free tier**: 100,000 traces recorded and 1,000,000 traces retrieved per month

### Cost Optimization
1. Configure appropriate sampling rates
2. Use trace groups to focus on critical paths
3. Set retention to minimum required
4. Avoid over-instrumenting (unnecessary subsegments)
5. Use filter expressions to reduce retrieved traces

## Common Exam Scenarios

### Scenario 1: Identify Performance Bottleneck
**Solution**: Enable X-Ray on all services, analyze trace timeline, identify slowest subsegments

### Scenario 2: Trace Request Across Microservices
**Solution**: Instrument all services with X-Ray SDK, propagate trace context, view service map

### Scenario 3: Monitor Specific Customer
**Solution**: Add customer ID as annotation, create trace group with filter, set up CloudWatch alarm

### Scenario 4: Debug Lambda Function
**Solution**: Enable active tracing in Lambda, use X-Ray SDK for custom subsegments, analyze traces in console

### Scenario 5: Reduce X-Ray Costs
**Solution**: Adjust sampling rules, reduce trace retrieval, disable on non-critical services

### Scenario 6: Root Cause Analysis
**Solution**: Use X-Ray Insights to identify anomalies, analyze affected traces, review service map

## Key Exam Points

- X-Ray provides distributed tracing and service maps
- Traces consist of segments (services) and subsegments (operations)
- X-Ray daemon collects and sends trace data
- Annotations are indexed and searchable, metadata is not
- Sampling rules control which requests are traced
- Lambda has built-in X-Ray daemon
- ECS/Fargate uses sidecar container for daemon
- Service map shows dependencies and health
- Automatic instrumentation for many AWS services
- SDK available for Java, Node.js, Python, .NET, Go, Ruby
- CloudWatch metrics generated for traces
- 30-day retention period (not configurable)
- Encryption in transit and at rest
- IAM permissions required: PutTraceSegments, PutTelemetryRecords
- Trace groups allow monitoring specific subsets
- X-Ray Insights use ML for anomaly detection
- Cost based on traces recorded and retrieved
- Can filter traces by annotations, HTTP status, duration, etc.

## Related Services

- **CloudWatch**: Metrics and alarms
- **Lambda**: Serverless compute with built-in X-Ray
- **API Gateway**: API management with X-Ray tracing
- **ECS/Fargate**: Container orchestration
- **Elastic Beanstalk**: PaaS with X-Ray support
- **Application Load Balancer**: Trace header propagation
- **SNS/SQS**: Message tracing
- **Step Functions**: Workflow tracing
- **AppSync**: GraphQL tracing
