# AWS Organizations

AWS Organizations is an account management service that enables you to consolidate multiple AWS accounts into an organization that you create and centrally manage.

## Core Concepts

### Organization Structure
- **Root**: Top of the hierarchy, contains all accounts and OUs
- **Organizational Units (OUs)**: Containers for accounts, can be nested (up to 5 levels)
- **Accounts**: AWS accounts belonging to the organization
  - **Management Account** (formerly Master): Creates organization, manages structure
  - **Member Accounts**: All other accounts in the organization

### Management vs Member Accounts

#### Management Account
- Creates the organization
- Cannot be restricted by SCPs
- Pays all charges for member accounts (consolidated billing)
- Can create, invite, and remove member accounts
- Should have minimal activity (security best practice)

#### Member Account
- Can only belong to one organization
- Can have SCPs applied
- Billed through management account
- Can be moved between OUs
- Can leave organization (if allowed)

## Service Control Policies (SCPs)

### Overview
- JSON policies that define maximum permissions
- Applied to root, OUs, or accounts
- Do NOT grant permissions (only restrict)
- Affect all users and roles in account, including root user
- **Does not affect**: Service-linked roles, management account

### SCP Evaluation Logic
```
Explicit Deny > Allow in SCP > Identity Policy
```

### SCP Strategies

#### Allow List Strategy (Default)
- `FullAWSAccess` SCP attached by default
- Remove `FullAWSAccess`, then explicitly allow services
- More restrictive, better control

#### Deny List Strategy
- Keep `FullAWSAccess` attached
- Add SCPs that explicitly deny specific actions
- Easier to implement initially

### SCP Examples

#### Deny Access to Region
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "eu-west-1"
          ]
        }
      }
    }
  ]
}
```

#### Prevent Users from Disabling CloudTrail
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Require MFA for EC2 Actions
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "ec2:StopInstances",
        "ec2:TerminateInstances"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

#### Prevent Account Leaving Organization
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "organizations:LeaveOrganization",
      "Resource": "*"
    }
  ]
}
```

## AWS Organization Features

### Consolidated Billing
- Single bill for all accounts
- Volume pricing discounts across accounts
- Shared Reserved Instances and Savings Plans
- Free data transfer between accounts in same organization
- **Payer account**: Management account pays all charges

### Account Creation
- Programmatic account creation via API/CLI
- Automatically joins organization
- No invitation required
- IAM role created for management account access: `OrganizationAccountAccessRole`

### Account Invitation
- Invite existing AWS accounts to join
- Invitation must be accepted
- Account leaves previous organization automatically
- Can only be in one organization at a time

### Organization CloudTrail
- Create trail in management account
- Applies to all accounts in organization
- Member accounts cannot modify/remove trail
- Centralized logging to S3 bucket in management account

## Delegated Administrator

- Grant member accounts admin access to specific AWS services
- Management account delegates administration
- Reduces need to use management account
- Supports services like:
  - AWS Config
  - AWS CloudFormation StackSets
  - Amazon GuardDuty
  - AWS Security Hub
  - Amazon Macie
  - AWS Firewall Manager

Example: Delegate Security Hub administration
```bash
aws securityhub enable-organization-admin-account \
  --admin-account-id 123456789012
```

## AWS Service Integration

### Integrated Services
- **IAM Identity Center** (AWS SSO): Centralized access management
- **CloudFormation StackSets**: Deploy stacks across accounts
- **AWS Config**: Aggregated compliance view
- **GuardDuty**: Centralized threat detection
- **Security Hub**: Aggregated security findings
- **Firewall Manager**: Centralized firewall rules
- **Backup**: Centralized backup policies
- **Service Catalog**: Share portfolios across organization
- **License Manager**: Centralized license tracking
- **Resource Access Manager (RAM)**: Share resources across accounts
- **Systems Manager**: Centralized operations

### Tag Policies
- Standardize tags across organization
- Define tag keys and allowed values
- Inheritance from parent OUs
- Compliance reporting in console
- Does not automatically add tags (only enforces)

### Backup Policies
- Centrally manage backup plans
- Apply to resources across accounts
- Define backup frequency, retention
- Inheritance through OU hierarchy

### AI Services Opt-Out Policies
- Control whether AI services can store/use content
- Apply to Amazon Rekognition, Lex, Polly, etc.
- Opt-out at organization, OU, or account level

## Best Practices

1. **Separate Management Account**: Minimal activity, security-focused
2. **OU Structure**: Align with organizational structure
   - By environment (Dev, Test, Prod)
   - By function (IT, HR, Finance)
   - By business unit
3. **SCP Strategy**: Start with deny list, move to allow list
4. **Layered SCPs**: Apply different SCPs at different OU levels
5. **Test SCPs**: Test in lower environment OUs first
6. **Enable All Features**: Use full Organizations capabilities
7. **Tag Strategy**: Implement and enforce with tag policies
8. **Centralized Logging**: Use organization CloudTrail
9. **Delegated Administration**: Reduce management account usage
10. **Regular Review**: Audit account structure and SCPs
11. **Service Control Policies**: Document and version control
12. **Cross-Account Roles**: Use for centralized access
13. **AWS Control Tower**: Consider for automated account setup

## Security Considerations

### Multi-Account Strategy Benefits
- **Blast radius containment**: Limit impact of security incidents
- **Isolation**: Separate workloads, environments, teams
- **Compliance**: Meet regulatory requirements
- **Billing**: Track costs by account/workload
- **Service limits**: Per-account limits apply independently

### Security Hub Integration
- Aggregate findings across accounts
- Centralized security posture view
- Compliance standards (CIS, PCI-DSS)
- Automated remediation with EventBridge

### GuardDuty Integration
- Enable in management account for all accounts
- Centralized threat detection
- Findings aggregated to management account

## CloudFormation StackSets

- Deploy CloudFormation stacks across accounts/regions
- Service-managed permissions (via Organizations) or self-managed
- Automatic deployment to new accounts
- Use case: Baseline resources (IAM roles, Config rules, etc.)

**Organization deployment**:
```yaml
StackSetName: BaselineResources
PermissionModel: SERVICE_MANAGED
AutoDeployment:
  Enabled: true
  RetainStacksOnAccountRemoval: false
OrganizationalUnitIds:
  - ou-xxxx-yyyyyyyy
```

## Common Architectures

### Hub and Spoke (Centralized)
- Central security/logging account
- Spoke accounts for workloads
- Use SCPs to enforce security baseline
- GuardDuty, Security Hub in central account

### Sandbox Accounts
- Developers experiment without prod impact
- Restrictive SCPs (no expensive services)
- Automated cleanup scripts
- Separate billing visibility

### Multi-Environment
```
Root
├── Prod OU
│   ├── Prod Account 1
│   └── Prod Account 2
├── Dev OU
│   ├── Dev Account 1
│   └── Dev Account 2
└── Security OU
    ├── Log Archive
    └── Security Tooling
```

## Monitoring & Compliance

### AWS Config Aggregator
- Multi-account, multi-region compliance view
- Aggregate configuration and compliance data
- Query resources across all accounts
- Deploy config rules via StackSets

### Organization Activity
- CloudTrail logs all Organizations API calls
- Monitor:
  - Account creation/deletion
  - OU structure changes
  - SCP modifications
  - Policy attachments

## Cost Management

### Consolidated Billing Benefits
- Volume discounts applied across all accounts
- Single payment method
- Shared RI and Savings Plans
- Free tier shared across organization

### Cost Allocation Tags
- Tag resources in member accounts
- Aggregate costs in Cost Explorer
- Cost and Usage Reports by tag
- Implement tag policies for consistency

### Reserved Instances Sharing
- RIs purchased in any account available to all
- Automatically applied to matching usage
- Zonal RIs shared within AZ
- Regional RIs shared across AZ

## Limits

- **Accounts per organization**: 10,000 (default)
- **OUs per organization**: 1,000
- **OU nesting depth**: 5 levels
- **Policies per entity**: 5 SCPs per account/OU
- **Policy size**: 5,120 characters
- **Accounts per invitation**: 1,000 per 24 hours
- **Member account invitation**: 1 outstanding invitation at a time

## Common Exam Scenarios

### Scenario 1: Prevent Data Exfiltration
**Solution**: SCP to deny actions outside approved regions, deny S3 bucket policy changes

### Scenario 2: Enforce MFA
**Solution**: SCP with Deny effect on critical actions without MFA present

### Scenario 3: Centralized Security Monitoring
**Solution**: Delegate GuardDuty/Security Hub to security account, enable organization-wide

### Scenario 4: Automated Account Provisioning
**Solution**: Create accounts programmatically, apply baseline via StackSets, enforce with SCPs

### Scenario 5: Prevent CloudTrail Deletion
**Solution**: Organization trail + SCP denying CloudTrail stop/delete actions

### Scenario 6: Cost Allocation
**Solution**: Tag policies to enforce tags, Cost Explorer with tag-based filtering

## Key Exam Points

- Organizations centrally manage multiple AWS accounts
- SCPs define maximum permissions, do not grant access
- Management account not affected by SCPs
- SCPs affect all principals including root user (except service-linked roles)
- Consolidated billing aggregates charges to management account
- Organization CloudTrail applies to all member accounts
- Member accounts cannot be in multiple organizations
- `OrganizationAccountAccessRole` created for management account access
- Tag policies enforce tagging standards
- Backup policies centralize backup management
- StackSets deploy resources across accounts automatically
- Delegated administrators reduce management account usage
- Volume discounts and RIs shared across organization
- Free data transfer between accounts in organization
- Service integration with GuardDuty, Security Hub, Config, etc.
- Can have up to 5 levels of nested OUs
- Root user of member accounts still affected by SCPs

## Related Services

- **IAM Identity Center**: Centralized SSO and access management
- **CloudFormation StackSets**: Multi-account/region deployments
- **AWS Control Tower**: Automated multi-account environment setup
- **Service Catalog**: Share products across organization
- **AWS RAM**: Share resources across accounts
- **GuardDuty**: Threat detection across organization
- **Security Hub**: Security posture management
- **AWS Config**: Configuration compliance across accounts
- **CloudTrail**: Audit trail across organization
- **Cost Explorer**: Analyze costs across accounts
