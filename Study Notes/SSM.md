# AWS Systems Manager (SSM)

AWS Systems Manager provides unified interface to view operational data and automate operational tasks across AWS resources.

## Core Components

### Systems Manager Agent (SSM Agent)
- Pre-installed on AWS-provided AMIs
- Processes requests from Systems Manager service
- Required for most Systems Manager capabilities
- Sends information to Systems Manager
- Can be installed on on-premises servers (hybrid environment)

### Resource Groups
- Logical collection of AWS resources
- Group by tags, CloudFormation stacks, or AWS Resource Groups query
- Simplify management of related resources
- **Use case**: Organize resources by application, environment, or project

## Run Command

### Overview
- Execute commands on managed instances remotely
- No SSH/RDP access required
- Supports EC2 and on-premises instances
- Command output returned to console or S3
- Integrated with IAM for access control

### Features
- **Rate control**: Concurrency and error thresholds
- **Output**: View in console, send to S3, or CloudWatch Logs
- **Notifications**: SNS integration
- **Audit**: CloudTrail logging
- **Document**: Pre-defined or custom SSM documents

### SSM Documents
- Define actions to perform
- **Types**:
  - **Command documents**: Run Command, State Manager
  - **Automation documents**: Automated workflows
  - **Package documents**: Software packages
  - **Session documents**: Session Manager configuration
- YAML or JSON format
- Share across accounts
- Version control

Example Command Document:
```yaml
schemaVersion: '2.2'
description: Install Apache web server
mainSteps:
- action: aws:runShellScript
  name: InstallApache
  inputs:
    runCommand:
    - sudo yum install -y httpd
    - sudo systemctl start httpd
    - sudo systemctl enable httpd
```

## State Manager

### Overview
- Automate keeping instances in defined state
- **Use case**: Ensure instances always have specific configuration
- Schedule-based or association-based execution
- Similar to Desired State Configuration (DSC)

### Associations
- Link SSM document to managed instances
- Can be scheduled with cron/rate expressions
- Apply to instances with specific tags
- Multiple associations per instance
- **Compliance**: Track and report on compliance status

### Use Cases
- Install software on boot
- Join domain
- Update software on schedule
- Configure OS settings
- Run security scans

## Automation

### Overview
- Automate common maintenance and deployment tasks
- Workflow engine using Automation documents (runbooks)
- Can include manual approval steps
- Integrates with CloudWatch Events/EventBridge

### Features
- **Pre-built runbooks**: AWS-provided automation documents
- **Custom runbooks**: Create your own
- **Multi-step workflows**: Chain multiple actions
- **Cross-account**: Execute across multiple accounts
- **Rate control**: Throttle execution

### Common Automation Scenarios
- **Patch instances**: Automated patching with approval
- **Create AMIs**: Automate AMI creation from instances
- **Restore from backup**: Automated recovery procedures
- **Attach IAM role**: Attach IAM instance profile
- **Copy snapshots**: Cross-region snapshot copy

Example Automation Document:
```yaml
schemaVersion: '0.3'
description: Create AMI and tag
parameters:
  InstanceId:
    type: String
mainSteps:
- name: CreateImage
  action: aws:createImage
  inputs:
    InstanceId: '{{ InstanceId }}'
    ImageName: 'AMI-{{ global:DATE_TIME }}'
- name: TagImage
  action: aws:createTags
  inputs:
    ResourceIds: ['{{ CreateImage.ImageId }}']
    Tags:
    - Key: Environment
      Value: Production
```

## Patch Manager

### Overview
- Automate patching of managed instances
- Scan for missing patches or install patches
- Operating systems: Windows, Linux, macOS
- Applications: Windows applications, Linux packages

### Patch Baselines
- Define approved patches for your instances
- **Predefined baselines**: AWS-provided for common OS
- **Custom baselines**: Define your own approval rules
- **Rules**:
  - Auto-approve patches X days after release
  - Patch classification (security, critical, etc.)
  - Patch severity
- Can include patch exceptions (approved/rejected specific patches)

### Patch Groups
- Tag instances to group for patching
- Different baselines for different groups
- Tag key: `Patch Group`
- **Use case**: Dev instances patch immediately, Prod patch after testing

### Maintenance Windows
- Schedule when patching occurs
- Define:
  - Duration
  - Schedule (cron expression)
  - Targets (instances to patch)
  - Tasks (patch, run command, automation)
  - Max concurrency and errors
- Multiple tasks per window
- **Use case**: Patch during off-hours

### Patch Compliance
- View compliance status in SSM console
- Integrate with AWS Config for reporting
- Track missing patches per instance
- **Types**:
  - Installed
  - Missing
  - Not Applicable
  - Failed

### Patching Process
1. **Scan**: Identify missing patches
2. **Install**: Apply patches according to baseline
3. **Reboot**: If required by patches (configurable)
4. **Report**: Update compliance status

## Session Manager

### Overview
- Interactive shell access to managed instances
- **No open inbound ports**: No SSH/RDP required
- Audit trail in CloudTrail and CloudWatch Logs
- IAM-based access control
- Cross-platform: Windows, Linux, macOS

### Features
- **No bastion host required**: Direct access via console/CLI
- **Port forwarding**: Access internal applications
- **Logging**: Session history and command logging
- **Encryption**: Data encrypted in transit
- **Sudo privilege control**: Restrict privileged access

### Benefits vs SSH/RDP
- No key management
- Centralized access control (IAM)
- Session recording and audit
- No public IP or bastion required
- Works through NAT Gateway or VPC endpoints

### Configuration
```json
{
  "schemaVersion": "1.0",
  "description": "Session Manager preferences",
  "sessionType": "Standard_Stream",
  "inputs": {
    "s3BucketName": "my-session-logs",
    "s3KeyPrefix": "sessions/",
    "s3EncryptionEnabled": true,
    "cloudWatchLogGroupName": "/aws/ssm/sessions",
    "cloudWatchEncryptionEnabled": true,
    "kmsKeyId": "alias/my-key",
    "runAsEnabled": true,
    "runAsDefaultUser": "ssm-user"
  }
}
```

## Parameter Store

### Overview
- Secure storage for configuration data and secrets
- Hierarchical storage with path-based organization
- Versioned parameters
- Integration with CloudFormation, ECS, Lambda, etc.

### Parameter Types
- **String**: Plain text
- **StringList**: Comma-separated values
- **SecureString**: Encrypted with KMS

### Parameter Tiers

| Feature | Standard | Advanced |
|---------|----------|----------|
| Parameters per account | 10,000 | 100,000 |
| Max size | 4 KB | 8 KB |
| Parameter policies | No | Yes |
| Cost | Free | $0.05 per parameter/month |
| API throughput | Standard | Higher |

### Parameter Policies
- TTL (expiration)
- Expiration notifications
- No change notifications
- **Use case**: Rotate secrets, alert on stale parameters

```json
{
  "Type": "Expiration",
  "Version": "1.0",
  "Attributes": {
    "Timestamp": "2024-12-31T23:59:59.000Z"
  }
}
```

### Parameter Hierarchy
```
/MyApp/Dev/DBPassword
/MyApp/Prod/DBPassword
/MyApp/Prod/DBEndpoint
```

### Usage Examples
```bash
# Store parameter
aws ssm put-parameter \
  --name "/MyApp/Prod/DBPassword" \
  --value "MyPassword123!" \
  --type SecureString \
  --key-id "alias/aws/ssm"

# Retrieve parameter
aws ssm get-parameter \
  --name "/MyApp/Prod/DBPassword" \
  --with-decryption

# Get parameters by path
aws ssm get-parameters-by-path \
  --path "/MyApp/Prod/" \
  --recursive
```

## Inventory

### Overview
- Collect metadata from managed instances
- View inventory in Systems Manager console
- Query with AWS Config
- Track software, network config, Windows updates

### Collected Data
- Applications
- AWS components
- Network configuration
- Windows updates
- Instance information
- Services
- Windows roles
- Custom inventory

### Custom Inventory
- Define custom inventory types
- Collect application-specific data
- JSON format
- **Use case**: Track custom software, licenses

## Compliance

### Overview
- Assess patch and configuration compliance
- Aggregate compliance data across accounts/regions
- Integrate with AWS Config for rules
- Dashboard view of compliance status

### Compliance Types
- **Patch compliance**: From Patch Manager
- **Association compliance**: From State Manager
- **Custom compliance**: User-defined compliance items

### Compliance Reporting
- By resource, severity, or compliance type
- Export to CSV
- API access for custom dashboards
- Integration with Security Hub

## Distributor

### Overview
- Package and distribute software packages
- Pre-built packages from AWS and partners
- Custom packages
- Install once or maintain version

### Features
- **Version control**: Manage package versions
- **Cross-platform**: Windows and Linux
- **IAM integration**: Control package access
- **Schedule installation**: Via State Manager

## OpsCenter

### Overview
- Central location for operational issues (OpsItems)
- Aggregate issues from CloudWatch, AWS Config, EventBridge
- Related resources and runbooks
- Collaborate on resolution

### OpsItems
- Operational work items
- Severity, status, title, description
- Related resources
- Runbooks for remediation
- **Deduplication**: Prevent duplicate items

### Integration
- CloudWatch alarms → OpsItems
- AWS Config compliance → OpsItems
- EventBridge events → OpsItems
- Manual OpsItem creation

## Explorer & Insights

### Explorer
- Customizable dashboard for operational data
- Widgets for OpsItems, compliance, inventory
- Filter by account, region, resource group
- **Use case**: Unified operational view

### Insights
- Automated analysis of operational data
- Identify common issues
- Group similar OpsItems
- **Use case**: Quickly identify widespread problems

## Hybrid Environments

### On-Premises Activation
- Register on-premises servers as managed instances
- Activation code and ID
- SSM Agent on on-premises servers
- **Use case**: Unified management of cloud and on-premises

### Activation Process
1. Create activation in Systems Manager
2. Install SSM Agent on on-premises server
3. Register with activation code and ID
4. Server appears as managed instance

```bash
# Register on-premises server
sudo amazon-ssm-agent -register \
  -code "activation-code" \
  -id "activation-id" \
  -region "us-east-1"
```

## Monitoring & Logging

### CloudWatch Integration
- Run Command output to CloudWatch Logs
- Session Manager logs to CloudWatch Logs
- Metrics for automation executions
- Alarms on command/automation failures

### CloudTrail Integration
- All API calls logged
- Track parameter changes
- Audit Run Command execution
- Session Manager session starts/ends

## Security Best Practices

1. **Least Privilege IAM**: Grant minimal SSM permissions
2. **VPC Endpoints**: Private connectivity (no internet gateway)
3. **Session Logging**: Enable CloudWatch Logs and S3
4. **Parameter Encryption**: Use SecureString for secrets
5. **Instance Profile**: Attach SSM-required IAM role
6. **Tag-Based Access**: Limit command execution by tags
7. **Approval Gates**: Use Automation approvals for sensitive tasks
8. **Regular Patching**: Automate with Patch Manager
9. **Compliance Monitoring**: Track with Compliance dashboard
10. **Audit Regularly**: Review CloudTrail logs

## Cost Optimization

### Pricing
- **No charge**: Run Command, Session Manager, Patch Manager, State Manager (standard parameters)
- **Advanced parameters**: $0.05 per parameter/month
- **API calls**: Standard rate (high volume)
- **Automation**: $0.002 per step
- **OpsCenter**: Free

### Free Features
- Up to 10,000 standard parameters
- Unlimited Run Command executions
- Session Manager sessions
- Patch Manager operations
- State Manager associations

## Common Exam Scenarios

### Scenario 1: Remote Command Execution Without SSH
**Solution**: Use Run Command with IAM-based access control

### Scenario 2: Automated Patching Schedule
**Solution**: Patch Manager with maintenance windows and patch baselines

### Scenario 3: Configuration Drift Prevention
**Solution**: State Manager associations with desired state documents

### Scenario 4: Centralized Parameter Management
**Solution**: Parameter Store with hierarchical naming and SecureString encryption

### Scenario 5: Cross-Account Automation
**Solution**: Automation documents with cross-account execution capabilities

### Scenario 6: Audit Interactive Sessions
**Solution**: Session Manager with CloudWatch Logs and S3 logging enabled

## Key Exam Points

- Systems Manager provides unified operational management
- SSM Agent required on instances (pre-installed on AWS AMIs)
- Run Command executes commands without SSH/RDP
- Session Manager provides interactive shell access
- Parameter Store stores configuration and secrets (up to 8 KB)
- Patch Manager automates OS and application patching
- State Manager maintains desired configuration state
- Automation provides workflow engine for operational tasks
- Maintenance windows schedule operational tasks
- Patch baselines define approved patches
- Compliance dashboard tracks patch and configuration compliance
- Hybrid activation allows on-premises server management
- VPC endpoints enable private connectivity
- All API calls logged to CloudTrail
- Session Manager logs sessions to CloudWatch and S3
- Parameter hierarchy enables organized storage (/App/Env/Name)
- Distributor manages software package distribution
- OpsCenter aggregates operational issues
- Free for most features (except advanced parameters and automation steps)

## Related Services

- **EC2**: Managed instances
- **CloudWatch**: Logging and monitoring
- **CloudTrail**: Audit logging
- **IAM**: Access control
- **VPC**: Private endpoints
- **KMS**: Parameter encryption
- **EventBridge**: Automation triggers
- **AWS Config**: Compliance rules
- **Lambda**: Automation integration
- **Secrets Manager**: Alternative for secrets (with rotation)
