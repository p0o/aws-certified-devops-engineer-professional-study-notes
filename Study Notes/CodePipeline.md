# AWS CodePipeline

AWS CodePipeline is a fully managed continuous delivery service that helps you automate your release pipelines for fast and reliable application and infrastructure updates.

## Core Concepts

### Pipeline Structure
- **Source Stage**: Retrieves code from source repositories
  - CodeCommit, GitHub, GitHub Enterprise, Bitbucket, Amazon S3
  - Triggers automatically on code changes via CloudWatch Events/EventBridge
- **Build Stage**: Compiles and tests code
  - CodeBuild, Jenkins, TeamCity, CloudBees
- **Deploy Stage**: Deploys applications
  - CodeDeploy, ECS, EKS, Elastic Beanstalk, CloudFormation, S3, AppConfig
- **Custom Actions**: Invoke Lambda functions, SNS topics, or third-party tools

### Artifacts
- Files produced by one stage and consumed by another
- Stored in S3 buckets (versioned and encrypted)
- Each pipeline has a designated artifact store per region
- **Input Artifacts**: Files used by an action
- **Output Artifacts**: Files produced by an action

## Pipeline Triggers

### Automatic Triggers
- **CloudWatch Events/EventBridge Rules**: Triggered on repository changes
  - CodeCommit repository state changes
  - S3 object-level API calls (via CloudTrail)
- **Webhooks**: For GitHub, GitHub Enterprise, and Bitbucket
- **Polling**: Legacy method (not recommended)

### Manual Triggers
- Execute pipeline manually from console or CLI
- Release changes from stopped execution

## Advanced Features

### Cross-Region Actions
- Deploy to multiple AWS regions within a single pipeline
- Each region requires its own artifact store (S3 bucket)
- Use case: Multi-region deployment for DR or global applications

### Cross-Account Actions
- Deploy resources to different AWS accounts
- Requires:
  - Cross-account IAM roles with trust relationships
  - KMS key policies for cross-account artifact access
  - S3 bucket policies for artifact access

### Manual Approval Actions
- Pause pipeline for human review before proceeding
- Can include:
  - SNS notification to reviewers
  - Custom data URL for review
  - Comments from approvers
- Timeout configuration available

### Parallel Execution
- **Default**: Latest execution supersedes previous executions
- **Parallel Mode**: Multiple executions run simultaneously
- **Queued Mode**: Executions wait for previous to complete

### Pipeline Variables
- **Default Variables**: AWS-provided (e.g., `#{codepipeline.PipelineExecutionId}`)
- **Custom Variables**: Defined at pipeline or action level
- Used to pass data between actions dynamically

## Integration Patterns

### With CodeBuild
```yaml
# buildspec.yml example for pipeline integration
version: 0.2
phases:
  build:
    commands:
      - echo "Building application..."
      - npm run build
artifacts:
  files:
    - '**/*'
  base-directory: 'build'
```

### With CloudFormation
- **CREATE_UPDATE**: Create or update a stack
- **DELETE_ONLY**: Delete a stack
- **REPLACE_ON_FAILURE**: Replace failed stack
- **Configuration Options**:
  - Stack name
  - Template file from input artifact
  - Parameter overrides
  - Capabilities (CAPABILITY_IAM, CAPABILITY_NAMED_IAM)
  - Role ARN

### With ECS
- Update task definitions and services
- Blue/green deployments via CodeDeploy
- Requires `imagedefinitions.json` artifact:
```json
[
  {
    "name": "container-name",
    "imageUri": "123456789012.dkr.ecr.us-east-1.amazonaws.com/app:tag"
  }
]
```

### With Lambda
- Deploy function code or update function configuration
- Supports versions and aliases
- Can use CodeDeploy for traffic shifting

## Security & Permissions

### Service Role
- IAM role that grants CodePipeline permission to:
  - Access source repositories
  - Invoke build/test/deploy providers
  - Access artifact stores
  - Publish to SNS topics
  - Invoke Lambda functions

### Artifact Encryption
- Artifacts encrypted at rest in S3 using AWS KMS
- Default AWS managed key: `aws/s3`
- Customer managed keys for cross-account scenarios
- In-transit encryption via TLS

### VPC Integration
- Not directly supported by CodePipeline
- Use VPC endpoints for S3 access if needed
- CodeBuild can run in VPC for build stage

## Monitoring & Troubleshooting

### CloudWatch Events/EventBridge
- Pipeline state changes (started, succeeded, failed)
- Stage state changes
- Action state changes
- Can trigger Lambda, SNS, SQS, etc.

### CloudWatch Metrics
- Limited native metrics
- Create custom metrics via CloudWatch Events + Lambda

### CloudTrail Logging
- All API calls logged
- Useful for auditing pipeline changes
- Track who started/stopped pipelines

### Console & CLI
- View execution history
- See detailed logs for each action
- Download input/output artifacts
- View source revisions

## Best Practices

1. **Source Control Everything**: Pipeline definitions, buildspecs, deployment configs
2. **Use Parameter Store/Secrets Manager**: For sensitive configuration
3. **Implement Manual Approvals**: For production deployments
4. **Enable Notifications**: SNS for pipeline state changes
5. **Use Cross-Region Deployment**: For disaster recovery
6. **Version Your Artifacts**: Use semantic versioning
7. **Implement Rollback Mechanisms**: CloudFormation change sets, ECS/Lambda traffic shifting
8. **Use Service Roles**: Principle of least privilege
9. **Enable Pipeline Execution History**: For auditing
10. **Test in Lower Environments**: Before production deployment

## Common Exam Scenarios

### Scenario 1: Failed Stage
**Problem**: Build stage fails intermittently
**Solution**:
- Check CloudWatch Logs for build provider
- Verify IAM permissions
- Check artifact availability
- Review timeout settings

### Scenario 2: Cross-Account Deployment
**Problem**: Pipeline cannot deploy to different account
**Solution**:
- Create IAM role in target account with trust relationship
- Update pipeline service role with sts:AssumeRole
- Configure KMS key policy for cross-account access
- Update S3 bucket policy for artifact access

### Scenario 3: Multi-Region Deployment
**Problem**: Need to deploy to multiple regions
**Solution**:
- Create artifact stores (S3 buckets) in each target region
- Configure cross-region actions in pipeline
- Ensure IAM roles have permissions in all regions
- Use CloudFormation StackSets for infrastructure

### Scenario 4: Blue/Green Deployments
**Problem**: Need zero-downtime deployments
**Solution**:
- Use CodeDeploy with ECS or Lambda
- Configure traffic shifting (linear, canary, all-at-once)
- Implement CloudWatch alarms for automatic rollback
- Use Application Load Balancer for traffic routing

## Pipeline as Code

### CloudFormation
```yaml
MyPipeline:
  Type: AWS::CodePipeline::Pipeline
  Properties:
    RoleArn: !GetAtt PipelineRole.Arn
    ArtifactStore:
      Type: S3
      Location: !Ref ArtifactBucket
    Stages:
      - Name: Source
        Actions:
          - Name: SourceAction
            ActionTypeId:
              Category: Source
              Owner: AWS
              Provider: CodeCommit
              Version: '1'
            Configuration:
              RepositoryName: !Ref RepoName
              BranchName: main
            OutputArtifacts:
              - Name: SourceOutput
```

### AWS CDK
```typescript
const pipeline = new codepipeline.Pipeline(this, 'Pipeline', {
  pipelineName: 'MyPipeline',
  crossAccountKeys: true, // For cross-account deployments
});

// Add stages
const sourceStage = pipeline.addStage({ stageName: 'Source' });
const buildStage = pipeline.addStage({ stageName: 'Build' });
const deployStage = pipeline.addStage({ stageName: 'Deploy' });
```

## Key Exam Points

- CodePipeline orchestrates the entire CI/CD workflow
- Integrates with AWS and third-party tools
- Artifacts are stored in S3 and encrypted with KMS
- Cross-region and cross-account deployments require additional configuration
- Manual approval actions pause pipeline execution
- CloudWatch Events/EventBridge for pipeline state change notifications
- Service role required with appropriate permissions
- Supports both automatic and manual triggers
- Can run actions in parallel within a stage
- Pipeline execution can be superseded, queued, or parallel

## Important Limits

- 50 pipelines per region (soft limit)
- 10 stages per pipeline
- 50 actions per stage
- 50 parallel actions across all stages
- 5 actions per pipeline in sequence
- Artifact size: 5 GB per artifact
- Total artifacts per pipeline: 100

## Related Services

- **CodeCommit**: Source control
- **CodeBuild**: Build and test
- **CodeDeploy**: Deployment
- **CloudFormation**: Infrastructure deployment
- **ECS/EKS**: Container deployment
- **Lambda**: Serverless deployment
- **S3**: Artifact storage and static website deployment
- **SNS**: Notifications
- **CloudWatch Events/EventBridge**: Event-driven automation
