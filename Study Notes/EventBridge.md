
### Targets available in the EventBridge console

You can configure the following targets for events in the EventBridge console:

- [API destination](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-api-destinations.html)
- [API Gateway](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-api-gateway-target.html)
- [AWS AppSync;](https://docs.aws.amazon.com/eventbridge/latest/userguide/target-appsync.html)
- [Batch job queue](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html#targets-specifics-batch)
- [CloudWatch log group](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html#targets-specifics-cwl)
- [CodeBuild project](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html#targets-specifics-codebuild)
- CodePipeline
- Amazon EBS `CreateSnapshot` API call
- EC2 Image Builder
- EC2 `RebootInstances` API call
- EC2 `StopInstances` API call
- EC2 `TerminateInstances` API call
- [ECS task](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html#targets-specifics-ecs-task)
- [Event bus in a different account or Region](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-cross-account.html)
- [Event bus in the same account and Region](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-bus-to-bus.html)
- Firehose delivery stream
- Glue workflow
- [Incident Manager response plan](https://docs.aws.amazon.com//incident-manager/latest/userguide/incident-creation.html#incident-tracking-auto-eventbridge)
- Inspector assessment template
- Kinesis stream
- Lambda function (ASYNC)
- [Amazon Redshift cluster data API queries](https://docs.aws.amazon.com/redshift/latest/mgmt/data-api-calling-event-bridge.html)
- [Amazon Redshift Serverless workgroup data API queries](https://docs.aws.amazon.com/redshift/latest/mgmt/data-api-calling-event-bridge.html)
- SageMaker Pipeline
- Amazon SNS topic
    EventBridge does not support [Amazon SNS FIFO (first in, first out) topics](https://docs.aws.amazon.com/sns/latest/dg/sns-fifo-topics.html).
- Amazon SQS queue
- Step Functions state machine (ASYNC)
- Systems Manager Automation
- Systems Manager OpsItem
- Systems Manager Run Command

https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html