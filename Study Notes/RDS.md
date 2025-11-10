# Amazon RDS (Relational Database Service)

Amazon RDS makes it easy to set up, operate, and scale relational databases in the cloud, providing cost-efficient and resizable capacity while automating time-consuming administration tasks.

## Supported Database Engines

- **Amazon Aurora** (MySQL and PostgreSQL compatible)
- **MySQL**
- **PostgreSQL**
- **MariaDB**
- **Oracle**
- **Microsoft SQL Server**

## Core Features

### Automated Backups
- **Point-in-time recovery**: Restore to any second within retention period
- **Retention period**: 0-35 days (0 = disabled)
- **Backup window**: Preferred time for daily snapshot
- **Transaction logs**: Backed up every 5 minutes
- **Stored in S3**: No additional charge for backup storage within retention period
- **Deletion**: Backups deleted when DB instance deleted (unless final snapshot taken)

### Manual Snapshots
- User-initiated backups
- Retained until explicitly deleted
- Can copy across regions and accounts
- Faster than automated backups for large databases
- **Use case**: Before major changes, long-term archives

### Multi-AZ Deployments
- **Synchronous replication** to standby in different AZ
- **Automatic failover**: 1-2 minutes typically
- **Single DNS endpoint**: Application uses same endpoint
- **Standby**: Cannot be used for read traffic (except Aurora)
- **Use case**: High availability, disaster recovery
- **RPO**: Near-zero (synchronous)
- **RTO**: 1-2 minutes

#### Failover Scenarios
- Primary DB failure
- AZ failure
- Instance type change
- Software patching
- Manual failover (reboot with failover)

### Read Replicas
- **Asynchronous replication** from source DB
- **Up to 15 read replicas** (5 for MySQL/MariaDB/PostgreSQL, 15 for Aurora)
- Can be in same AZ, cross-AZ, or cross-region
- Can be promoted to standalone DB
- **Use case**: Read-heavy workloads, analytics, reporting
- Different instance type from source allowed

#### Read Replica Features
- **Replication lag**: Monitor with CloudWatch metric
- **Multiple layers**: Read replica of read replica (limited)
- **Cross-region**: For disaster recovery and local reads
- **Promotion**: Breaks replication, becomes standalone

### Encryption
- **At rest**: AES-256 encryption using KMS
- **In transit**: SSL/TLS connections
- **Encryption of existing DB**: Take snapshot, copy with encryption, restore
- **Encrypted replicas**: If source encrypted, replicas must be encrypted
- **Performance impact**: Minimal

### Parameter Groups
- Database engine configuration
- **Static parameters**: Require reboot
- **Dynamic parameters**: Applied immediately or during maintenance window
- Custom parameter groups for specific configurations
- **Use case**: Buffer sizes, character sets, time zones

### Option Groups
- Enable database engine features
- Examples:
  - Oracle: Transparent Data Encryption (TDE), Advanced Auditing
  - SQL Server: SQL Server Audit, Mirroring
  - MySQL: MariaDB Audit Plugin
- Some options require additional licensing

## High Availability & Disaster Recovery

### Multi-AZ vs Read Replicas

| Feature | Multi-AZ | Read Replicas |
|---------|----------|---------------|
| Purpose | High availability | Scalability, DR |
| Replication | Synchronous | Asynchronous |
| Standby accessible | No (except Aurora) | Yes |
| Failover | Automatic | Manual (promote) |
| Performance | No impact | Can reduce source load |
| Cross-region | Yes (Aurora, PostgreSQL) | Yes |
| Cost | ~2x single-AZ | Additional instance cost |

### Aurora Specific Features
- **Aurora Replicas**: Up to 15, can be failover target
- **Global Database**: Cross-region replication (< 1 second lag)
- **Aurora Serverless**: Auto-scaling compute capacity
- **Aurora Clones**: Fast, copy-on-write clones
- **Backtrack**: Rewind DB to previous state without restore

## Maintenance & Patching

### Maintenance Window
- Weekly time window for updates
- Minor version upgrades (if auto minor version upgrade enabled)
- Required patches applied
- Can modify or postpone (limited)
- **Multi-AZ**: Standby patched first, then failover, then primary

### Upgrade Process
1. **Minor version**: Automatic during maintenance window (if enabled)
2. **Major version**: Manual initiation required
3. **Blue/green**: Create parallel environment, test, swap (Aurora)

## Monitoring & Performance

### CloudWatch Metrics
- **CPUUtilization**: Processor usage
- **DatabaseConnections**: Number of connections
- **FreeableMemory**: Available RAM
- **FreeStorageSpace**: Available disk space
- **ReadIOPS / WriteIOPS**: I/O operations
- **ReadLatency / WriteLatency**: Operation latency
- **NetworkReceiveThroughput / NetworkTransmitThroughput**
- **SwapUsage**: Swap file usage

### Enhanced Monitoring
- Granular metrics (down to 1 second)
- OS-level metrics
- Process/thread information
- Requires IAM role
- **Use case**: Detailed performance troubleshooting

### Performance Insights
- Dashboard for database performance
- Identify bottlenecks and top SQL queries
- Wait events analysis
- 7 days free retention, up to 2 years (paid)
- **Use case**: Query optimization, capacity planning

### Database Logs
- Error logs
- Slow query logs
- General query logs
- Audit logs (MariaDB, MySQL, Oracle, SQL Server)
- Published to CloudWatch Logs

## Scaling

### Vertical Scaling (Instance Type)
- Change instance class
- Downtime during modification (Multi-AZ: failover first, reduced downtime)
- Can schedule for maintenance window

### Horizontal Scaling (Read Replicas)
- Add read replicas to distribute read load
- Application must handle read/write split
- Connection pooling recommended

### Storage Scaling
- **Storage Auto Scaling**: Automatically increases storage
- Set maximum storage threshold
- Triggers when:
  - Free space < 10%
  - Low storage lasts 5 minutes
  - 6 hours since last increase
- **Manual scaling**: Initiate storage increase manually
- Cannot decrease storage size

### Storage Types
- **General Purpose SSD (gp2/gp3)**: Balance of price and performance
  - gp2: 3 IOPS per GB, up to 16,000 IOPS
  - gp3: 3,000 baseline IOPS, up to 16,000 IOPS (configurable)
- **Provisioned IOPS SSD (io1/io2)**: High performance, predictable
  - Up to 64,000 IOPS (io1)
  - Up to 256,000 IOPS (io2 Block Express)
  - **Use case**: I/O-intensive workloads
- **Magnetic**: Legacy, not recommended

## Security

### Network Isolation
- **VPC**: Deploy in private subnets
- **Security Groups**: Control inbound/outbound traffic
- **DB Subnet Group**: Minimum 2 subnets in different AZs
- **Publicly accessible**: Optional (not recommended for production)

### Authentication & Authorization
- **Database authentication**: Username/password
- **IAM database authentication**: Use IAM roles (MySQL, PostgreSQL, Aurora)
- **Kerberos authentication**: For SQL Server, Oracle, PostgreSQL
- **Active Directory integration**: For SQL Server

### Encryption
- **At rest**: KMS encryption
- **In transit**: SSL/TLS (enforce with parameter groups)
- **Snapshots**: Encrypted if source encrypted
- **Cannot remove encryption once enabled**

### Secrets Manager Integration
- Store database credentials
- Automatic password rotation
- Retrieve credentials in application code

### IAM Permissions
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rds:CreateDBInstance",
        "rds:ModifyDBInstance",
        "rds:DeleteDBInstance"
      ],
      "Resource": "arn:aws:rds:region:account:db:*"
    }
  ]
}
```

## RDS Proxy

### Overview
- Connection pooling for RDS and Aurora
- Reduces database connection overhead
- Improves failover time (up to 66% faster)
- **Use case**: Lambda functions, serverless applications

### Features
- **Connection pooling**: Reuse database connections
- **IAM authentication**: Enforce IAM credentials
- **Secrets Manager**: Automatic credential management
- **Failover**: Preserve connections during failover
- **TLS/SSL**: Encrypted connections

### Configuration
- Deploy in VPC
- Requires subnets in multiple AZs
- Target: DB instance or Aurora cluster
- **Endpoint**: Applications connect to proxy endpoint, not DB directly

## Backup & Restore

### Backup Strategies
- **Automated backups + Multi-AZ**: Daily backups with PITR
- **Manual snapshots**: Before major changes
- **Cross-region copies**: For disaster recovery
- **Export to S3**: For long-term archival or data lakes

### Restore Process
1. Choose snapshot or point-in-time
2. Restore creates new DB instance
3. Update application endpoint
4. Validate data
5. Delete old instance if successful

### Cross-Region Backup Copy
- Manual snapshots: Copy to any region
- Automated backups: Cross-region automated backups feature
- **Use case**: Disaster recovery, compliance

## Cost Optimization

### Pricing Components
- **Instance hours**: Charged per instance-hour
- **Storage**: GB per month
- **IOPS**: For Provisioned IOPS storage
- **Backup storage**: Free within retention period, charged for excess
- **Data transfer**: Standard AWS rates

### Cost Reduction Strategies
1. **Reserved Instances**: 1-year or 3-year commitment (up to 69% savings)
2. **Right-sizing**: Choose appropriate instance type
3. **Delete unused snapshots**: Especially old manual snapshots
4. **Aurora Serverless**: For variable workloads
5. **Single-AZ for non-production**: Dev/test environments
6. **Stop/Start**: For non-production, temporary workloads
7. **Graviton instances**: Better price-performance (ARM-based)

## Best Practices

1. **Enable Multi-AZ**: For production databases
2. **Enable automated backups**: Set appropriate retention period
3. **Use read replicas**: For read-heavy workloads
4. **Regular snapshot testing**: Verify restore procedures
5. **Monitor metrics**: Set up CloudWatch alarms
6. **Use parameter groups**: Optimize database configuration
7. **Enable Enhanced Monitoring**: For detailed insights
8. **Use Performance Insights**: Identify query performance issues
9. **Encrypt databases**: For sensitive data
10. **VPC deployment**: Never use publicly accessible for production
11. **Connection pooling**: Use RDS Proxy for serverless
12. **Tag resources**: For cost allocation and organization
13. **Upgrade during low traffic**: Plan maintenance windows
14. **Test failover**: Regularly test Multi-AZ failover

## Common Exam Scenarios

### Scenario 1: Zero Downtime Patching
**Solution**: Multi-AZ deployment - standby patched first, failover, patch primary

### Scenario 2: Read-Heavy Workload
**Solution**: Create read replicas, distribute read traffic across replicas

### Scenario 3: Disaster Recovery
**Solution**: Multi-AZ for HA + cross-region read replica + automated backup copying

### Scenario 4: Point-in-Time Recovery
**Solution**: Enable automated backups with sufficient retention period, restore to specific timestamp

### Scenario 5: Improve Lambda Performance with RDS
**Solution**: Use RDS Proxy for connection pooling, reduces connection overhead

### Scenario 6: Secure Database Credentials
**Solution**: Store in Secrets Manager, enable automatic rotation, IAM authentication where possible

### Scenario 7: Monitoring Database Performance
**Solution**: Enable Performance Insights + Enhanced Monitoring + CloudWatch alarms

## Key Exam Points

- RDS supports MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Aurora
- Multi-AZ provides high availability with synchronous replication
- Read replicas provide scalability with asynchronous replication
- Automated backups enable point-in-time recovery (0-35 day retention)
- Manual snapshots retained until explicitly deleted
- Multi-AZ failover is automatic (1-2 minutes typically)
- Read replicas can be promoted to standalone databases
- Encryption must be enabled at creation time
- Storage auto-scaling available for gp2, gp3, io1, io2
- Cannot decrease allocated storage
- RDS Proxy improves Lambda and serverless connectivity
- Enhanced Monitoring provides OS-level metrics
- Performance Insights helps identify slow queries
- IAM database authentication available for MySQL, PostgreSQL, Aurora
- Multi-AZ standby cannot be used for read traffic (except Aurora)
- Maintenance window used for patches and minor upgrades
- Major version upgrades must be manually initiated
- Parameter groups control database engine configuration
- Option groups enable additional database features
- Cross-region read replicas support disaster recovery

## Related Services

- **Amazon Aurora**: AWS proprietary relational database
- **AWS DMS**: Database migration service
- **AWS Backup**: Centralized backup management
- **AWS Secrets Manager**: Credential management and rotation
- **CloudWatch**: Monitoring and alarms
- **CloudTrail**: API audit logging
- **AWS Lambda**: Serverless compute (use with RDS Proxy)
- **VPC**: Network isolation
- **KMS**: Encryption key management
- **IAM**: Access control and authentication
- **S3**: Backup storage and export destination
