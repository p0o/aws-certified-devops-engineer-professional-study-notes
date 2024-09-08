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

## Exam Fromat

The **AWS Certified DevOps Engineer - Professional** exam is designed for individuals who perform a DevOps engineer role and have at least two years of experience provisioning, operating, and managing AWS environments.

> Level: Professional
> Length: 180 Minutes
> Cost: 300 USD
> Format: 75 questions, multiple choice or multiple response

The exam validates expertise in:

- Implementing and managing continuous delivery systems and methodologies on AWS
- Implementing and automating security controls, governance processes, and compliance validation
- Defining and deploying monitoring, metrics, and logging systems on AWS
- Implementing systems that are highly available, scalable, and self-healing on the AWS platform
- Designing, managing, and maintaining tools to automate operational processes

For more details, visit the [official AWS Exam Guide](https://aws.amazon.com/certification/certified-devops-engineer-professional/).

---

## Key Concepts

These topics are the main concepts for your exam prepration rated by 💀.

- Domain 1: SDLC Automation
  - [Amazon CodeGuru](https://aws.amazon.com/codeguru/) 💀
  - [AWS CodePipeline](https://aws.amazon.com/codepipeline/) 💀💀💀💀💀
  - [AWS CodeBuild](https://aws.amazon.com/codebuild/) 💀💀💀💀
  - [AWS CodeCommit](https://aws.amazon.com/codecommit/) 💀💀💀
  - [Amazon ECS](https://aws.amazon.com/ecs/) 💀💀💀
  - [Amazon EKS](https://aws.amazon.com/eks/) 💀💀
  - [AWS Cloudformation](https://aws.amazon.com/cloudformation/) 💀💀💀💀💀
  - [What is SDLC?](https://aws.amazon.com/what-is/sdlc/) 💀💀
- Domain 2: Configuration Management and IaC
  - [AWS System Manager](https://aws.amazon.com/systems-manager/) 💀💀💀💀💀
  - [AWS Config](https://aws.amazon.com/config/) 💀💀💀
  - [AWS OpsWorks](https://aws.amazon.com/opsworks/) 💀
  - [What is Configuration Management?](https://aws.amazon.com/what-is/configuration-management/) 💀💀💀
- Domain 3: Resilient Cloud Solutions
  - [Amazon Route53](https://aws.amazon.com/route53/) 💀💀💀
  - [AWS Resource Access Manager](https://aws.amazon.com/ram/?c=sc&sec=srvm) 💀
  - [Understand resiliency patterns and trade-offs to architect efficiently in the cloud](https://aws.amazon.com/blogs/architecture/understand-resiliency-patterns-and-trade-offs-to-architect-efficiently-in-the-cloud/) 💀💀
  - [Shared Responsibility Model for Resiliency](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/shared-responsibility-model-for-resiliency.html) 💀💀
  -
- Domain 4: Monitoring and Logging
  - [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) 💀💀
  - [Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html) 💀💀💀💀
  - [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) 💀💀💀💀
  - [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html) 💀
  - [AWS X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html) 💀💀
- Domain 5: Incident and Event Response
  - [What is Incident Management?](https://aws.amazon.com/what-is/incident-management/) 💀💀
  - [Remediating Noncompliant Resources with AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/remediation.html) 💀💀💀
- Domain 6: Security and Compliance
  - [AWS Organization](https://aws.amazon.com/organizations/?c=sc&sec=srvm) 💀💀💀💀💀
  - [AWS Identity Center](https://aws.amazon.com/iam/identity-center/?c=sc&sec=srvm) 💀💀💀💀💀
  - [AWS IAM](https://aws.amazon.com/iam/?c=sc&sec=srvm) 💀💀💀💀💀
  - [Amazon GuardDuty](https://aws.amazon.com/guardduty/?c=sc&sec=srvm) 💀
  - [Amazon Cognito](https://aws.amazon.com/cognito/?c=sc&sec=srvm) 💀💀
  - [AWS Directory Service](https://aws.amazon.com/directoryservice/?c=sc&sec=srvm) 💀
  - [AWS Inspector](https://aws.amazon.com/inspector/?c=sc&sec=srvm) 💀💀
  - [AWS Security Hub](https://aws.amazon.com/security-hub/?c=sc&sec=srvm) 💀
  - [AWS Config](https://aws.amazon.com/config/?c=sc&sec=srvm) 💀💀💀💀
  - [AWS Firewall Manager](https://aws.amazon.com/firewall-manager/?c=sc&sec=srvm) 💀
  - [AWS WAF](https://aws.amazon.com/waf/?c=sc&sec=srvm) 💀

---

## Study Notes

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
