# AWS CodeCommit

AWS CodeCommit is a fully-managed source control service that hosts secure Git-based repositories in the AWS cloud.

## Core Concepts

### Repository Features
- **Git-based**: Standard Git commands and workflows
- **Unlimited repositories**: No limit on number of repositories
- **Unlimited users**: No limit on users per repository
- **File size**: Up to 2 GB per file
- **Repository size**: Unlimited (no soft limit)
- **Highly available**: Replicated across multiple Availability Zones
- **Encrypted**: At rest (KMS) and in transit (HTTPS/SSH)

### Authentication Methods

#### HTTPS (Git credentials)
- IAM user with Git credentials
- Generated in IAM console
- Username and password for HTTPS Git operations
- Best for: Console users, temporary access

#### HTTPS (AWS CLI credential helper)
- Uses AWS CLI configuration and IAM credentials
- No separate password needed
- Best for: Developers with AWS CLI configured

#### SSH
- Upload SSH public key to IAM user
- Use SSH private key for authentication
- Best for: Advanced users, automation

### Access Control

#### IAM Policies
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codecommit:GitPull",
        "codecommit:GitPush"
      ],
      "Resource": "arn:aws:codecommit:us-east-1:123456789012:MyRepo"
    }
  ]
}
```

#### Common IAM Actions
- `codecommit:GitPull`: Clone, fetch, pull
- `codecommit:GitPush`: Push commits
- `codecommit:CreateBranch`: Create branches
- `codecommit:DeleteBranch`: Delete branches
- `codecommit:GetBranch`: View branch details
- `codecommit:Merge`: Merge branches
- `codecommit:CreatePullRequest`: Create PRs
- `codecommit:GetCommit`: View commit details

## Branch Protection

### Branch Permissions
- Restrict who can push/merge to specific branches
- Implemented via IAM policies with conditions
- Example: Protect main branch from direct pushes

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "codecommit:GitPush",
        "codecommit:DeleteBranch",
        "codecommit:PutFile",
        "codecommit:Merge*"
      ],
      "Resource": "arn:aws:codecommit:*:*:MyRepo",
      "Condition": {
        "StringEqualsIfExists": {
          "codecommit:References": ["refs/heads/main"]
        },
        "Null": {
          "codecommit:References": false
        }
      }
    }
  ]
}
```

### Approval Rule Templates
- Require approvals before merging pull requests
- Can be applied to multiple repositories
- Specify number of approvals needed
- Define approval pool (specific users/roles)
- Override requirements for specific users

```json
{
  "Version": "2018-11-08",
  "Statements": [
    {
      "Type": "Approvers",
      "NumberOfApprovalsNeeded": 2,
      "ApprovalPoolMembers": [
        "arn:aws:sts::123456789012:assumed-role/CodeReviewer/*"
      ]
    }
  ]
}
```

## Notifications & Triggers

### CloudWatch Events / EventBridge
- Automatically generated for repository events
- Event types:
  - Comments on commits/PRs
  - Pull request state changes
  - Branch/tag creation/deletion
  - Repository state changes
- Trigger Lambda, SNS, SQS, etc.

Example event pattern:
```json
{
  "source": ["aws.codecommit"],
  "detail-type": ["CodeCommit Repository State Change"],
  "detail": {
    "event": ["referenceCreated", "referenceUpdated"],
    "referenceType": ["branch"],
    "referenceName": ["main"]
  }
}
```

### Triggers (Legacy)
- Built-in notification mechanism
- Sends to SNS topic or Lambda function
- Events:
  - All repository events
  - Push to existing branch
  - Create branch or tag
  - Delete branch or tag
- **Note**: CloudWatch Events/EventBridge preferred

### Notifications (AWS Chatbot)
- Send notifications to Slack or Amazon Chime
- Configure notification rules for events:
  - Pull request created/updated
  - Comments on commits/PRs
  - Approval status changes
- Uses AWS Chatbot integration

## Pull Requests

### PR Workflow
1. Create feature branch
2. Make changes and commit
3. Create pull request to target branch
4. Code review and comments
5. Approval by reviewers
6. Merge (manual or automatic)

### Merge Strategies
- **Fast-forward merge**: Linear history
- **Squash merge**: Combine commits into one
- **Three-way merge**: Preserve individual commits
- Merge conflicts must be resolved manually

### PR Reviews
- Comment on specific lines
- Comment on overall changes
- Request changes
- Approve PR
- Track approval status

### Approval Rules
- Require minimum number of approvals
- Specify approval pool members
- Override permissions for specific users
- Can prevent merge until approved

## Integration with CI/CD

### With CodePipeline
- Automatic pipeline trigger on commit
- Use CloudWatch Events for specific branches
- Source stage pulls latest commit
- Pass commit ID to subsequent stages

```yaml
# Pipeline source stage
Source:
  Type: AWS::CodePipeline::Pipeline
  Properties:
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
              RepositoryName: MyRepo
              BranchName: main
              PollForSourceChanges: false # Use CloudWatch Events
            OutputArtifacts:
              - Name: SourceOutput
```

### With CodeBuild
- Triggered on commit via CloudWatch Events
- buildspec.yml in repository root
- Can run on PR creation for validation
- Use for automated testing

### With Lambda
- Trigger Lambda function on commit
- Use cases:
  - Automated code scanning
  - Slack notifications
  - Jira ticket updates
  - Custom validations

## Migration & Replication

### Migrating to CodeCommit
```bash
# From existing Git repository
git clone --bare https://github.com/user/repo.git
cd repo.git
git push https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyRepo --all
git push https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyRepo --tags
```

### Cross-Region Replication
- Not built-in feature
- Implement using:
  - CloudWatch Events + Lambda
  - Mirror repository in different region
  - Use for disaster recovery

## Security Best Practices

### Encryption
- **At rest**: Encrypted using AWS KMS
- **In transit**: HTTPS (TLS) or SSH
- Can use customer-managed KMS keys
- Default uses AWS-managed keys

### Access Control
1. **IAM policies**: Control repository access
2. **Branch restrictions**: Protect important branches
3. **Approval rules**: Enforce code review
4. **MFA**: Require for sensitive operations
5. **IP restrictions**: Limit access by IP range
6. **VPC endpoints**: Private connectivity from VPC

### Auditing
- **CloudTrail**: All API calls logged
- Track:
  - Repository creation/deletion
  - Branch operations
  - PR activities
  - Access attempts
- **CloudWatch Events**: Real-time monitoring

### Credential Management
- Rotate Git credentials regularly
- Use temporary credentials via STS
- Avoid embedding credentials in code
- Use AWS Secrets Manager for automation

## Monitoring & Troubleshooting

### CloudWatch Metrics
- Limited built-in metrics
- Custom metrics via CloudWatch Events + Lambda
- Track:
  - Commit frequency
  - PR turnaround time
  - Repository size growth

### Common Issues

#### 1. Authentication Failures
- Verify Git credentials are correct
- Check IAM permissions
- Ensure credentials not expired
- For SSH, verify public key uploaded

#### 2. Push Rejected (Protected Branch)
- Check branch protection policies
- Verify user has required permissions
- Use pull request workflow instead

#### 3. Large File Issues
- Files >2 GB not supported
- Use Git LFS for large files
- Consider S3 for very large assets

#### 4. Slow Clone/Pull
- Repository size issue
- Consider shallow clone: `git clone --depth 1`
- Use sparse checkout for partial repository

## Best Practices

1. **Branch Strategy**: Use GitFlow or GitHub Flow
2. **Protect Main Branch**: Require PRs for changes
3. **Code Review**: Implement approval rules
4. **Automated Testing**: Trigger builds on commits
5. **Tag Releases**: Use semantic versioning
6. **Small Commits**: Easier to review and revert
7. **Descriptive Messages**: Clear commit messages
8. **PR Templates**: Standardize PR descriptions
9. **CI/CD Integration**: Automate deployments
10. **Regular Backups**: Mirror to another repository
11. **Notification Rules**: Stay informed of changes
12. **Clean History**: Squash commits when appropriate

## Pricing

### Free Tier
- 5 active users per month (free forever)
- 50 GB storage per month
- 10,000 Git requests per month

### Standard Pricing
- **Additional active users**: $1 per active user per month
- **Additional storage**: $0.06 per GB per month
- **Additional requests**: $0.001 per request

**Active user**: Makes a Git request during the month

## Common Exam Scenarios

### Scenario 1: Automated Pipeline Trigger
**Problem**: Trigger pipeline only on main branch commits
**Solution**: Use CloudWatch Events with event pattern filtering on `referenceName: main`

### Scenario 2: Protect Production Branch
**Problem**: Prevent direct pushes to main branch
**Solution**: IAM policy with Deny effect on GitPush action with condition on `codecommit:References`

### Scenario 3: Cross-Account Access
**Problem**: Allow another account to access repository
**Solution**: Create IAM role in repository account, allow assume role from other account, grant CodeCommit permissions to role

### Scenario 4: Automated Code Review
**Problem**: Run linting/tests before allowing merge
**Solution**: PR creation trigger → CodeBuild → Post results as PR comments → Approval rule requires passing

### Scenario 5: Notification on PR Creation
**Problem**: Notify team when PR created
**Solution**: CloudWatch Event rule for PR creation → SNS topic → Email/Slack notification

## Key Exam Points

- CodeCommit is a fully-managed Git repository service
- Supports standard Git operations and workflows
- Authentication via HTTPS (Git credentials), SSH, or AWS CLI credential helper
- Access control via IAM policies and resource policies
- Branch protection via IAM policies with conditions
- Approval rules enforce code review requirements
- Integrates with CodePipeline, CodeBuild, Lambda
- CloudWatch Events/EventBridge for repository events
- Encrypted at rest (KMS) and in transit (TLS/SSH)
- CloudTrail logs all API activity
- Pull requests support comments, reviews, and approvals
- No repository size limit (file size limit 2 GB)
- Triggers can invoke SNS or Lambda (legacy feature)
- Cross-account access via IAM roles
- Can use VPC endpoints for private connectivity

## Related Services

- **CodePipeline**: CI/CD orchestration
- **CodeBuild**: Build and test automation
- **CodeDeploy**: Deployment automation
- **Lambda**: Event-driven automation
- **EventBridge**: Event routing
- **SNS**: Notifications
- **CloudTrail**: Audit logging
- **IAM**: Access control
- **KMS**: Encryption key management
- **VPC**: Private connectivity via endpoints
