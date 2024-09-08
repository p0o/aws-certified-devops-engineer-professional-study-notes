### Resource Based Policies
Lambda supports resource-based permissions policies for Lambda functions and layers. Resource-based policies let you grant usage permission to other AWS accounts or organizations on a per-resource basis. You also use a resource-based policy to allow an AWS service to invoke your function on your behalf.

# API GW

1. REST API
	1. Caching
	2. API Keys
	3. usage plans
2. HTTP API
	1. Good for low latency applications
3. WebSocker API

### Mapping template

Could be used to map JSON to XML

### Quota
API Gateway timeout limit = 29 seconds

API Gateway = could be integrated with SQS

Invocation methods: sync, async, poll based

### Lambda Authorizer
Token Based: JWT or OAuth token
Request Parameter-based: A `REQUEST` authorizer receives the caller's identity in a combination of headers, query string parameters, [stageVariables](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html#stagevariables-template-reference), and [$context](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html#context-variable-reference) variables.


> [!NOTE] Need to verify 
>Need to learn this more in depth


###   AWS Serverless Application Repository SAR
The best way to use prebuilt serverless applications under MIT license


### Lambda Networking
- When you connect a function to a VPC in your account, the function can't access the internet unless your VPC provides access.
- Lambda accesses resources in your VPC using a Hyperplane ENI
- A Lambda function always runs inside a VPC owned by the Lambda service
- Lambda supports two types of connections: TCP (Transmission Control Protocol) and UDP (User Datagram Protocol).

- IP v6:
    - Inbound:
        - To invoke your function over IPv6, use Lambda's public [dual-stack endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#dual-stack-endpoints). Dual-stack endpoints support both IPv4 and IPv6.
    - Outbound:
        - Your function can connect to resources in dual-stack VPC subnets over IPv6. This option is turned off by default.

- To establish a private connection between your VPC and Lambda, create an [interface VPC endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-interface.html). Interface endpoints are powered by [AWS PrivateLink](https://aws.amazon.com/privatelink)
- Since ENIs are an exhaustible resource and there is a soft limit of 250 ENIs per Region, you should monitor elastic network interface usage if you are configuring Lambda functions for VPC access. Generally, if you increase concurrency limits in Lambda, you should evaluate if you need an elastic network interface increase. If the limit is reached, this causes invocations of VPC-enabled Lambda functions to be throttled.

# Step Functions

### State Machine Structure

Example:

```json
{
  "Comment": "A Hello World example of the Amazon States Language using a Pass state",
  "StartAt": "HelloWorld",
  "States": {
    "HelloWorld": {
      "Type": "Pass",
      "Result": "Hello World!",
      "End": true
    }
  }
}

```


**Common State Fields in Workflows

These fields are common to all state elements:

- **Type (Required)**: The state's type.
- **Next**: The next state to run when the current one finishes. Not needed for terminal states like Succeed or Fail.
- **End**: Marks the state as terminal if true. Only one of Next or End can be used.
- **Comment (Optional)**: A human-readable description of the state.
- **InputPath (Optional)**: Selects a portion of the state's input for processing. Defaults to $ (entire input).
- **OutputPath (Optional)**: Selects a portion of the state's output for the next state. Defaults to $ (entire output).

