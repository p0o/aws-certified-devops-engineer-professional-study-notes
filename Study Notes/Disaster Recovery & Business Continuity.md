
- Backup & Restore
	- We have backup only which should be transferred and restored at DR site
- Pilot Light
	- Data is already sync-ed in the DR site
- Warm Standby
	- Data is sync-ed with full copy of instances (but down-scaled)
- Active/Active
	- Exact copy of primary site is running in DR site (200% infra cost)

### Storage
Instance Store & EBS Volumes: Not resistant at all, only works within AZ
	EBS could be replicated to other AZs using a snapshot

EFS: resilient within region, is replicated through different AZs

S3 is regional with most classes and could be replicated cross account and region. 

### Compute
EC2: Auto-scaling group can help to create new instances upon the failure of normal instances.
VPC Enabled Lambda: Is using ENI to use AZ and could easily switch to another AZ in case of failure
Public Lambda: Handled by AWS infrastructure within a region (not resilient cross regional)

### Database
DynamoDB: Resilient to any AZ failure in normal mode, provides global resiliency using global tables. These tables are multi-master replication meaning that you can write into any of them and data will be replicated to other regions.

RDS (normal): Uses Subnet groups, a primary and standby instances exist, there are automatic replication between them cross AZ. Failure of primary leads to failover event to switch over to standby.
	Provides Cross Region Read Replicas which are sync-ed to other regions in an asynchronous way. Unlike Aurora, they are putting extra load on DB.

RDS (Aurora): Could have replicas across all region and is the most resilient relational database offering from AWS. Also Aurora supports Global Tables which is providing read-only clusters in other regions which could be promoted to writer in case of disasters. These replications happens from storage layer and do not put extra load on DB.

### Networking
VPC, Route Tables and IGW are region resilient

Interface Endpoints: Are residing within one AZ, failure of AZ means failure of VPCE
	You can deploy multiple of them within each AZ to add high availability