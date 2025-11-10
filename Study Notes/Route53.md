# Amazon Route 53

Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service, domain registration service, and health checking service.

## Core DNS Concepts

### DNS Record Types
- **A Record**: Maps hostname to IPv4 address
- **AAAA Record**: Maps hostname to IPv6 address
- **CNAME**: Maps hostname to another hostname (cannot be used for zone apex)
- **ALIAS**: AWS-specific, maps hostname to AWS resource (can be used for zone apex)
- **MX**: Mail exchange servers
- **TXT**: Text information (SPF, DKIM, verification)
- **NS**: Name servers for hosted zone
- **SOA**: Start of Authority (zone information)
- **PTR**: Reverse DNS lookup
- **SRV**: Service locator
- **CAA**: Certificate Authority Authorization

### Hosted Zones
- **Public Hosted Zone**: Responds to DNS queries from the internet
- **Private Hosted Zone**: Responds to DNS queries from within VPC(s)
- **Cost**: $0.50 per hosted zone per month

### TTL (Time to Live)
- How long DNS resolver caches the record (seconds)
- High TTL (24-48 hours): Less traffic to Route 53, outdated records longer
- Low TTL (60 seconds): More traffic to Route 53, easier to change records
- Consider during migrations and updates

## Routing Policies

### Simple Routing
- Single resource or multiple values for one record
- Returns all values in random order
- **Use case**: Single web server
- No health checks

### Weighted Routing
- Distribute traffic across resources by percentage
- Assign weight (0-255) to each record
- Traffic % = Weight / Sum of all weights
- **Use case**: A/B testing, gradual migration
- Supports health checks

```
www.example.com (70%) → 1.2.3.4
www.example.com (30%) → 5.6.7.8
```

### Latency-Based Routing
- Route to resource with lowest latency
- Based on latency between user and AWS regions
- **Use case**: Global applications optimizing performance
- Supports health checks

### Failover Routing
- Active-passive setup
- Primary and secondary resources
- Routes to secondary when primary fails health check
- **Use case**: Disaster recovery
- Requires health checks

### Geolocation Routing
- Route based on user geographic location
- Specify by continent, country, or US state
- Most specific location wins
- **Default location**: For users not matching any rule
- **Use case**: Content localization, compliance restrictions
- Supports health checks

### Geoproximity Routing
- Route based on geographic location of resources and users
- **Bias**: Expand or shrink geographic region (1 to 99 or -1 to -99)
- Requires Route 53 Traffic Flow
- **Use case**: Shift traffic gradually from one region to another
- Supports health checks

### Multi-Value Answer Routing
- Return multiple values (up to 8 healthy records)
- Each record can have health check
- Client chooses which IP to use
- **Use case**: Simple load distribution with health checks
- Not a substitute for ELB

### IP-Based Routing
- Route based on client IP address (CIDR blocks)
- Define CIDR blocks and corresponding endpoints
- **Use case**: Routing by ISP, compliance requirements

## Health Checks

### Types of Health Checks

#### Endpoint Health Checks
- Monitor endpoint (IP or domain name)
- Protocols: HTTP, HTTPS, TCP
- Interval: 30 seconds (standard) or 10 seconds (fast)
- Failure threshold: Number of consecutive failures (default: 3)
- **String matching**: Check for specific text in response body (first 5120 bytes)
- **Status code**: HTTP 2xx or 3xx for success

#### Calculated Health Checks
- Combine results of multiple health checks
- Logic: AND, OR, NOT
- Can monitor up to 256 child health checks
- Specify number of healthy children required

#### CloudWatch Alarm Health Checks
- Monitor CloudWatch alarm state
- **Use case**: Monitor metrics not directly accessible (private resources, custom metrics)

### Health Check Features
- **SNS notifications**: Alert on status changes
- **Health checker locations**: 15+ locations worldwide
- **Fast interval**: 10-second intervals (additional cost)
- **Latency measurements**: Track response times
- **Status page**: View health check status in console

### Private Resource Health Checks
- Cannot directly health check private resources
- **Solution**: Create CloudWatch metric → CloudWatch alarm → Route 53 health check monitoring alarm

## Traffic Flow

- Visual editor for complex routing configurations
- Create traffic policies
- Version control for policies
- Apply policies to multiple hosted zones
- **Use case**: Complex geoproximity routing, weighted failover
- **Cost**: $50 per policy record per month

## Domain Registration

- Register new domains
- Transfer existing domains
- Auto-renewal configuration
- Transfer lock protection
- Domain transfer authorization codes
- WHOIS privacy protection (free)
- **Common TLDs**: .com, .net, .org, and many more
- **Registration period**: 1-10 years

## DNS Security (DNSSEC)

- Protects against DNS spoofing and cache poisoning
- Cryptographically signs DNS records
- Chain of trust from root to domain
- **DNSSEC signing**: Enable on hosted zone
- **DS records**: Add to parent zone
- **KSK (Key Signing Key)** and **ZSK (Zone Signing Key)**
- Not supported for private hosted zones

## Resolver (DNS Resolution)

### Route 53 Resolver
- Default DNS for VPC (VPC CIDR + 2)
- Resolves DNS queries for VPC resources
- Recursively queries public DNS

### Resolver Endpoints
- **Inbound endpoints**: Forward DNS queries from on-premises to VPC
- **Outbound endpoints**: Forward DNS queries from VPC to on-premises
- Elastic Network Interfaces (ENIs) in VPC
- **Use case**: Hybrid cloud DNS resolution

### Resolver Rules
- Forward DNS queries for specific domains
- **System rules**: Automatically created for private hosted zones
- **Forwarding rules**: Custom rules to forward queries
- **Conditional forwarding**: Forward based on domain name
- Can share rules across accounts via AWS RAM

## Private Hosted Zones

### Configuration
- Associate with one or more VPCs
- Must enable **DNS hostnames** and **DNS resolution** in VPC settings
- Can associate with VPCs in different accounts (via CLI/API)
- Split-view DNS: Same domain for public and private (different records)

### Cross-Account VPC Association
1. Create association authorization in hosted zone account
2. Associate VPC from other account
3. Must be done via CLI or API (not console)

## Integration Patterns

### With CloudFront
- Use Alias record to point to CloudFront distribution
- No charge for Alias queries to CloudFront
- Automatic IPv6 support

### With ELB
- Use Alias record to point to load balancer
- ELB DNS name changes; Alias automatically updates
- Works with ALB, NLB, CLB

### With S3 Website
- Use Alias record to point to S3 website endpoint
- Bucket name must match domain name
- Website endpoint region-specific

### With API Gateway
- Use custom domain name
- Point Route 53 to API Gateway endpoint
- Can use Alias record for API Gateway edge-optimized endpoints

### With CloudWatch
- Health check results published as metrics
- Create alarms on health check status
- Integrate with SNS for notifications

## Alias vs CNAME

| Feature | Alias | CNAME |
|---------|-------|-------|
| Zone apex (example.com) | ✅ Supported | ❌ Not allowed |
| AWS resources | ✅ Optimized | ✅ Supported |
| Queries | Free for AWS resources | Charged |
| TTL | Automatic (cannot set manually) | Manual |
| Health checks | ✅ Supported | ✅ Supported |
| Non-AWS resources | ❌ Not supported | ✅ Supported |

## Monitoring & Logging

### CloudWatch Metrics
- **Query Logs**: Number of queries
- **Health Check Status**: Endpoint health
- **Health Check Latency**: Response time
- **Child Health Checks**: For calculated health checks

### Query Logging
- Log DNS queries to CloudWatch Logs
- Includes:
  - Query timestamp
  - Hosted zone ID
  - Query name and type
  - Response code
  - Layer 4 protocol
  - Route 53 edge location
- **Use case**: Security analysis, troubleshooting
- **Cost**: CloudWatch Logs pricing applies

### CloudTrail Integration
- All Route 53 API calls logged
- Who made the change, when, from where
- **Use case**: Audit compliance, security analysis

## High Availability & Disaster Recovery

### Multi-Region Failover
1. Create health checks for each region
2. Use failover routing with primary and secondary
3. Automatic failover on health check failure
4. Can chain with latency or geolocation

### Active-Active Setup
- Use weighted or latency-based routing
- Distribute traffic across regions
- All regions serve traffic simultaneously

### Active-Passive Setup
- Use failover routing
- Secondary region only receives traffic on primary failure
- Lower cost than active-active

## Best Practices

1. **Use Alias Records**: For AWS resources (free queries)
2. **Implement Health Checks**: For all routing policies except simple
3. **Set Appropriate TTLs**: Lower TTL for frequently changing records
4. **Use Traffic Flow**: For complex routing configurations
5. **Enable Query Logging**: For troubleshooting and security
6. **Use Calculated Health Checks**: Combine multiple health checks
7. **Tag Resources**: For cost allocation and organization
8. **Use Private Hosted Zones**: For internal resources
9. **Implement DNSSEC**: For security-sensitive domains
10. **Monitor Health Checks**: Set up CloudWatch alarms
11. **Use Geolocation**: For compliance and localization
12. **Plan for Failover**: Test DR scenarios regularly

## Cost Optimization

### Pricing Components
- **Hosted zones**: $0.50 per zone per month
- **Standard queries**: $0.40 per million queries
- **Latency-based routing**: $0.60 per million queries
- **Geo queries**: $0.70 per million queries
- **Health checks**: $0.50 per health check per month
- **Alias queries to AWS resources**: Free
- **Domain registration**: Varies by TLD

### Cost Reduction
1. Use Alias records instead of CNAME for AWS resources
2. Consolidate hosted zones where possible
3. Increase TTL to reduce query volume
4. Use calculated health checks to reduce number of endpoints monitored
5. Choose appropriate health check interval (30s vs 10s)

## Common Exam Scenarios

### Scenario 1: Blue/Green Deployment
**Solution**: Use weighted routing policy, gradually shift weight from blue to green

### Scenario 2: Global Low-Latency Application
**Solution**: Deploy to multiple regions, use latency-based routing with health checks

### Scenario 3: Active-Passive DR
**Solution**: Failover routing policy with health check on primary, route to secondary on failure

### Scenario 4: Hybrid Cloud DNS
**Solution**: Route 53 Resolver with inbound and outbound endpoints for bidirectional DNS resolution

### Scenario 5: Geo-Restricted Content
**Solution**: Geolocation routing policy with records for each allowed country, no default

### Scenario 6: Private VPC DNS
**Solution**: Private hosted zone associated with VPC, enable DNS hostnames and resolution in VPC

## Key Exam Points

- Route 53 provides DNS, domain registration, and health checking
- Alias records can be used for zone apex (unlike CNAME)
- Alias queries to AWS resources are free
- Seven routing policies: Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, Multi-value, IP-based
- Health checks monitor endpoints (HTTP/HTTPS/TCP) or CloudWatch alarms
- Private hosted zones for VPC-internal DNS resolution
- Route 53 Resolver endpoints enable hybrid cloud DNS
- DNSSEC protects against DNS attacks
- Query logging to CloudWatch Logs for analysis
- Traffic Flow for complex routing visualizations
- Calculated health checks combine multiple health checks
- Low TTL enables faster changes but increases query costs
- Failover routing requires health checks
- Multi-value answer routing returns up to 8 healthy records
- Can create records for zone apex with Alias
- Health checks can trigger SNS notifications
- Private hosted zones require DNS resolution and hostnames enabled in VPC

## Related Services

- **CloudFront**: Global content delivery with Route 53 integration
- **ELB**: Load balancing with Alias record support
- **S3**: Static website hosting with custom domains
- **API Gateway**: Custom domains for APIs
- **CloudWatch**: Monitoring and alarming
- **CloudTrail**: API audit logging
- **VPC**: Private hosted zone integration
- **AWS Certificate Manager**: SSL/TLS certificates for custom domains
- **AWS RAM**: Share Resolver rules across accounts
