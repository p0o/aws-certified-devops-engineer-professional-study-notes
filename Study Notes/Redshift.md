
Petabyte-scale data warehouse
OLAP (column-based) database
Managed (provisioned)
Could query from S3 using Redshift Spectrum
Supports federated query
Integrates with Quicksight
Not good for ad-hoc queries

##### Architecture
- Has provisioned and serverless
- Single AZ for faster network performance
- Leader Node - query input, planning and aggregation
- Compute Node - performing queries
	- Slices working in parallel - 2, 4, 16, 32 
- VPC Security, IAM Permissions, KMS for encryption at rest, Cloudwatch monitoring
- By Default uses public routes
- With **Enhanced VPC Routing** it could use VPC networking including NACL, Security Groups, Custom DNS, VPC gateways, VPC Endpoints

![[Pasted image 20240808153321.png]]


##### Zero-ETL
Zero-ETL integration is a fully managed solution that makes transactional or operational data available in Amazon Redshift in near real time.
To create a zero-ETL integration, you specify an integration source and an Amazon Redshift data warehouse as the target. The integration replicates data from the source to the target data warehouse. The data becomes available in Amazon Redshift within seconds. The integration monitors the health of the data pipeline and recovers from issues when possible.

The following sources are currently supported for zero-ETL integrations:

- Aurora MySQL-Compatible Edition
    
- Aurora PostgreSQL-Compatible Edition (preview)
    
- RDS for MySQL (preview)


##### Tips
- Enhanced VPC Routing could not be used with Redshift Spectrum
- One benefit of using Enhanced Routing is that all COPY and UNLOAD traffic is logged in VPC flow logs
- For serverless you only pay for capacity and it's multi AZ