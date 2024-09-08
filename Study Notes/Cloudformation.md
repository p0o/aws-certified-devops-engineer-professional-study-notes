### Custom Resources
**Custom resources** enable users to write custom provisioning logic in templates that AWS CloudFormation runs anytime a user creates, updates (if the custom resource has been changed), or deletes stacks. For instance, a user might want to include resources that are not available as AWS CloudFormation resource types. Users can include those resources by using custom resources. That way, users can still manage all related resources in a single stack.

If a **custom resource** is used to invoke a **Lambda function** in AWS CloudFormation, the request will include a **pre-signed URL**. The Lambda function is responsible for returning a response to the pre-signed URL to indicate if the resource creation was successful or not. Otherwise, stack will remain in `CREATE_IN_PROGRESS` state.

##### Use cases
Use Custom Resource to fetch new AMIs. Learn more:

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/walkthrough-custom-resources-lambda-lookup-amiids.html

https://aws.amazon.com/blogs/devops/faster-auto-scaling-in-aws-cloudformation-stacks-with-lambda-backed-custom-resources/


### UpdatePolicy

[`AWS::AutoScaling::AutoScalingGroup`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-autoscaling-autoscalinggroup.html)

With Auto Scaling groups, you can use one or more update policies to control how CloudFormation handles certain updates. These policies include:

- `AutoScalingReplacingUpdate` and `AutoScalingRollingUpdate` policies – CloudFormation can either replace the Auto Scaling group and its instances with an `AutoScalingReplacingUpdate` policy, or replace only the instances with an `AutoScalingRollingUpdate` policy. These replacement operations occur when you make one or more of the following changes:
    
    - Change the Auto Scaling group's `[AWS::AutoScaling::LaunchConfiguration](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-autoscaling-launchconfiguration.html)`.
        
    - Change the Auto Scaling group's `VPCZoneIdentifier` property.
        
    - Change the Auto Scaling group's `LaunchTemplate` property.
        
    - Update an Auto Scaling group that contains instances that don't match the current `LaunchConfiguration`.
        
    If both the `AutoScalingReplacingUpdate` and `AutoScalingRollingUpdate` policies are specified, setting the `WillReplace` property to `true` gives `AutoScalingReplacingUpdate` precedence.