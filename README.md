# AWS Certified DevOps Engineer - Professional Study Notes

This repository provides a collection of notes, tips, and resources that I found helpful while preparing for the AWS Certified DevOps Engineer - Professional exam. I’m sharing them to support other professionals pursuing this certification. However, please note that this is not a comprehensive guide, and it may not cover everything you need to pass the exam. I might have skipped some lessons due to my experience and you should in no circumstances rely on a single source for your prepration!
At the end of this page, I’ve included some courses that I personally used, and I highly recommend using a structured course to fill any knowledge gaps.

## Good luck to all exam takers and wishing you the best outcome!

## Table of Contents

- [Introduction](#introduction)
- [Disclaimer](#disclaimer)
- [Exam Format](#exam-format)
- [Study Notes](#study-notes)
- [Key Concepts](#key-concepts)
- [Recommended Material](#recommended-material)
- [Contributing](#contributing)

---

## Disclaimer

These notes are provided "as-is" without any guarantees or warranty. While every effort has been made to ensure the accuracy of the content, there is no guarantee that this will cover everything in the actual exam. Use this material as a supplement to your own study plan.

**Note**: AWS certifications are subject to change, so always refer to the [official AWS Certification Guide](https://aws.amazon.com/certification/) for the latest information.

---

## Exam Format

The **AWS Certified DevOps Engineer - Professional (DOP-C02)** exam is designed for individuals who perform a DevOps engineer role and have at least two years of experience provisioning, operating, and managing AWS environments.

**Exam Details:**
- **Level:** Professional
- **Length:** 180 Minutes
- **Cost:** 300 USD
- **Format:** 75 questions, multiple choice or multiple response
- **Passing Score:** 750 (scaled score of 100-1000)
- **Delivery Method:** Testing center or online proctored exam
- **Valid for:** 3 years

The exam validates expertise in:

- Implementing and managing continuous delivery systems and methodologies on AWS
- Implementing and automating security controls, governance processes, and compliance validation
- Defining and deploying monitoring, metrics, and logging systems on AWS
- Implementing systems that are highly available, scalable, and self-healing on the AWS platform
- Designing, managing, and maintaining tools to automate operational processes

For more details, visit the [official AWS Exam Guide](https://aws.amazon.com/certification/certified-devops-engineer-professional/).

---

## Exam Domains & Key Concepts

The exam covers six domains with different weightings. Topics are rated by importance (💀 = moderate focus, 💀💀💀💀💀 = critical focus).

### Domain 1: SDLC Automation (22%)
**Focus:** Implementing CI/CD pipelines, automated testing, and deployment strategies

- [AWS CodePipeline](https://aws.amazon.com/codepipeline/) 💀💀💀💀💀
- [AWS CodeBuild](https://aws.amazon.com/codebuild/) 💀💀💀💀
- [AWS CodeDeploy](https://aws.amazon.com/codedeploy/) 💀💀💀💀
- [AWS CodeCommit](https://aws.amazon.com/codecommit/) 💀💀💀
- [AWS CodeArtifact](https://aws.amazon.com/codeartifact/) 💀💀
- [Amazon CodeGuru](https://aws.amazon.com/codeguru/) 💀
- [AWS CloudFormation](https://aws.amazon.com/cloudformation/) 💀💀💀💀💀
- [Amazon ECR](https://aws.amazon.com/ecr/) 💀💀💀
- [Amazon ECS](https://aws.amazon.com/ecs/) 💀💀💀
- [Amazon EKS](https://aws.amazon.com/eks/) 💀💀
- [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) 💀💀
- [AWS Lambda](https://aws.amazon.com/lambda/) 💀💀💀💀

### Domain 2: Configuration Management and IaC (17%)
**Focus:** Managing infrastructure as code, configuration automation, and compliance

- [AWS Systems Manager](https://aws.amazon.com/systems-manager/) 💀💀💀💀💀
- [AWS Config](https://aws.amazon.com/config/) 💀💀💀💀
- [AWS CloudFormation](https://aws.amazon.com/cloudformation/) 💀💀💀💀💀
- [AWS OpsWorks](https://aws.amazon.com/opsworks/) 💀
- [AWS Service Catalog](https://aws.amazon.com/servicecatalog/) 💀💀
- [AWS AppConfig](https://docs.aws.amazon.com/appconfig/) 💀💀
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) 💀💀💀
- [AWS Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) 💀💀💀

### Domain 3: Resilient Cloud Solutions (15%)
**Focus:** Designing HA/DR solutions, backup strategies, and fault tolerance

- [Amazon Route 53](https://aws.amazon.com/route53/) 💀💀💀💀
- [AWS Backup](https://aws.amazon.com/backup/) 💀💀💀
- [Amazon RDS](https://aws.amazon.com/rds/) 💀💀💀
- [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) 💀💀
- [AWS Auto Scaling](https://aws.amazon.com/autoscaling/) 💀💀💀💀
- [Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/) 💀💀💀💀
- [Amazon S3](https://aws.amazon.com/s3/) 💀💀💀
- [AWS CloudFormation](https://aws.amazon.com/cloudformation/) 💀💀💀💀
- [AWS Resource Access Manager](https://aws.amazon.com/ram/) 💀

### Domain 4: Monitoring and Logging (15%)
**Focus:** Implementing monitoring, logging, and observability solutions

- [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) 💀💀💀💀💀
- [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/) 💀💀💀💀
- [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) 💀💀💀💀
- [AWS X-Ray](https://aws.amazon.com/xray/) 💀💀💀
- [Amazon EventBridge](https://aws.amazon.com/eventbridge/) 💀💀💀
- [AWS SNS](https://aws.amazon.com/sns/) 💀💀
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html) 💀💀
- [Amazon OpenSearch Service](https://aws.amazon.com/opensearch-service/) 💀

### Domain 5: Incident and Event Response (14%)
**Focus:** Automating event response, troubleshooting, and remediation

- [Amazon EventBridge](https://aws.amazon.com/eventbridge/) 💀💀💀💀
- [AWS Systems Manager](https://aws.amazon.com/systems-manager/) 💀💀💀💀
- [AWS Config](https://aws.amazon.com/config/) 💀💀💀
- [AWS Lambda](https://aws.amazon.com/lambda/) 💀💀💀
- [AWS Step Functions](https://aws.amazon.com/step-functions/) 💀💀
- [Amazon SNS](https://aws.amazon.com/sns/) 💀💀
- [Amazon SQS](https://aws.amazon.com/sqs/) 💀💀
- [AWS CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) 💀💀💀

### Domain 6: Security and Compliance (17%)
**Focus:** Implementing security controls, compliance validation, and governance

- [AWS IAM](https://aws.amazon.com/iam/) 💀💀💀💀💀
- [AWS Organizations](https://aws.amazon.com/organizations/) 💀💀💀💀💀
- [AWS IAM Identity Center](https://aws.amazon.com/iam/identity-center/) 💀💀💀💀
- [AWS KMS](https://aws.amazon.com/kms/) 💀💀💀💀
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) 💀💀💀
- [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/) 💀💀
- [Amazon Inspector](https://aws.amazon.com/inspector/) 💀💀
- [Amazon GuardDuty](https://aws.amazon.com/guardduty/) 💀💀
- [AWS Security Hub](https://aws.amazon.com/security-hub/) 💀💀
- [AWS Config](https://aws.amazon.com/config/) 💀💀💀💀
- [AWS WAF](https://aws.amazon.com/waf/) 💀💀
- [AWS Shield](https://aws.amazon.com/shield/) 💀
- [AWS Firewall Manager](https://aws.amazon.com/firewall-manager/) 💀
- [Amazon Cognito](https://aws.amazon.com/cognito/) 💀💀
- [AWS Audit Manager](https://aws.amazon.com/audit-manager/) 💀

---

## Study Notes

All study notes are organized by AWS service or topic in the `Study Notes/` directory:

### Core DevOps Services
- [AWS CodePipeline](Study%20Notes/CodePipeline.md) - CI/CD orchestration
- [AWS CodeBuild](Study%20Notes/CodeBuild.md) - Build and test automation
- [AWS CodeDeploy](Study%20Notes/CodeDeploy.md) - Deployment automation
- [AWS CodeCommit](Study%20Notes/CodeCommit.md) - Source control
- [AWS CloudFormation](Study%20Notes/Cloudformation.md) - Infrastructure as Code

### Configuration & Management
- [AWS Systems Manager (SSM)](Study%20Notes/SSM.md) - Operations management
- [AWS Config](Study%20Notes/Config.md) - Resource configuration tracking
- [AWS OpsWorks](Study%20Notes/OpsWork.md) - Configuration management

### Compute & Containers
- [Amazon EC2](Study%20Notes/EC2.md) - Virtual servers
- [Amazon ECS](Study%20Notes/ECS.md) - Container orchestration
- [Amazon EKS](Study%20Notes/EKS.md) - Kubernetes service
- [AWS Lambda](Study%20Notes/Lambda.md) - Serverless compute
- [AWS Elastic Beanstalk](Study%20Notes/Elastic%20Beanstalk.md) - PaaS

### Monitoring & Logging
- [Amazon CloudWatch](Study%20Notes/CloudWatch.md) - Monitoring and observability
- [AWS CloudTrail](Study%20Notes/CloudTrail.md) - API auditing
- [AWS X-Ray](Study%20Notes/XRay.md) - Distributed tracing
- [Amazon EventBridge](Study%20Notes/EventBridge.md) - Event bus

### Security & Identity
- [AWS IAM](Study%20Notes/IAM.md) - Identity and access management
- [AWS Organizations](Study%20Notes/Organizations.md) - Multi-account management
- [AWS KMS](Study%20Notes/KMS.md) - Key management
- [AWS Secrets Manager](Study%20Notes/SecretsManager.md) - Secrets management

### Networking & Content Delivery
- [Networking (VPC, etc.)](Study%20Notes/Networking.md) - Virtual networking
- [Amazon Route 53](Study%20Notes/Route53.md) - DNS and routing
- [Amazon CloudFront](Study%20Notes/Cloudfront.md) - CDN

### Storage & Databases
- [Storage (S3, EBS, EFS)](Study%20Notes/Storage.md) - Storage solutions
- [Amazon DynamoDB](Study%20Notes/DynamoDB.md) - NoSQL database
- [Amazon RDS](Study%20Notes/RDS.md) - Relational databases

### Additional Topics
- [Disaster Recovery & Business Continuity](Study%20Notes/Disaster%20Recovery%20&%20Business%20Continuity.md)
- [Serverless Architecture](Study%20Notes/Serverless.md)
- [Amazon SQS](Study%20Notes/SQS.md) - Message queuing
- [AWS Backup](Study%20Notes/Backup.md) - Centralized backup

---

## Recommended Material

- [Adrian Cantrill Course](https://learn.cantrill.io/p/aws-certified-devops-engineer-professional)
- [Tutorials Dojo Practice Exam](https://portal.tutorialsdojo.com/courses/aws-certified-devops-engineer-professional-practice-exams/?_gl=1*oei1ua*_gcl_au*MTk0MTYzNDU2MS4xNzIxOTg3MDc4LjEwNzQzNDQ0MzguMTcyMTk4NzA3OS4xNzIxOTg3MDc4*_ga*OTMyMTIzMjcuMTcyMTk4NjYzMw..*_ga_L96TFJ1R9K*MTcyNTc5NTU4NC4xMS4wLjE3MjU3OTU1ODQuMC4wLjA.)
- [AWS FAQ](https://aws.amazon.com/faqs/)
- [AWS Whitepapers](https://aws.amazon.com/whitepapers/)
- [AWS Documentation](https://aws.amazon.com/documentation/)

---

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository.
2. Make your changes.
3. Submit a pull request with details on the changes made.
