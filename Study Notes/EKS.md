
- Could run on Outposts, EKS Anywhere, EKS Distro
- Nodes could be self managed, managed node groups (EC2) or Fargate
- Storage could be EBS, EFS, FSx Lustre, FSx for NetApp ONTAP
- EKS Cluster runs on AWS Managed VPC and use ENIs to connect to your VPC
- Worker Nodes run on your VPC and connect to cluster using ENIs or public endpoint

### Difference between EKS Anywhere, Distro and EKS Connector
Distro is the open source implementation of K8s from AWS which includes etcs, networking and storage plugins compatible with AWS. It also provides extened support for K8s versions after community support expiry. However there is no direct path from Distro to installation. EKS Anywhere is using Distro and EKS Connector to provide easy installation for users planning to run EKS Distro on their on-prem infrastructure.

##### OS Support
EKS Anywhere supports by Bottlerocket linux as default node operating system, however it supports Ubuntu and RHEL as well.

##### EKS Connector
EKS Connector is an agent that runs on any K8s compatible cluster including EKS Anywhere. This agent enables you to view your K8s resources from AWS Console.

##### Pricing
EKS Anywhere is free however in order to access EKS curated packages and 24/7 technical support you should purchase a EKS Anywhere Enterprise Support (Curren cost is $24K per year / per cluster)

##### Why use EKS Anywhere?
- Allowing you to create clusters with the latest software updates, security patches, and version release cycle as Amazon EKS in the cloud
- Better compatibility between packages and AWS services
- Run Hybrid environments in AWS and maintain data sovereignty
- Consolidate and reduce support costs from different vendors (in case of enterprise support)