
# Basic Properties

- Could mount to a ssd/hdd:
	- Instance store (temporary)
	- Amazon EBS (persistent)
- Could mount to a shared file server:
	- EFS
	- FSx for Lustre
	- FSx for Windows File Server
- Network
	- Amazon VPC
	- Elastic IP (only v4)
	- ENI:
		- Could add multiple ENI with separate private IP, security group and elastic IP
		- Failover (could be attached to another instance to maintain IP)
		- Network isolation
	- Placement Group (lower network latency, high throughput)
	- Elastic Network Adapter (ENA)
		- High perf capability
	- Elastic Fabric Adapter (EFA)
		- High perf computing (HPC) workload
		- OS bypass to provide ultra low latency
- AMI
	- IDs are unique within regions (could not be used globally)
	- Volume Snapshots
		- EBS
		- Template for root volume (for instance store)
		- Block device mapping
			- EBS Snapshots mapping
		- Launch Permissions
			- Public (any account)
			- Explicit (some accounts)
			- Implicit (default, only owner)
	- Regional
		- Could be copied to another region or account
	- Virtualization Types
		- Is important when choosing instance types
		- Could not choose different virtualization after launch
		- PV (paravirtual)
			- better for storage and network operations
		- HVM (Hardware virtual machine)
			- could use special hardware extensions for enhanced networking or GPU processing
## User Data
- Most be in Base64-encoded format
- Limited to 16 kb in raw form
- accessible from metadata URI:
	http://169.254.169.254/latest/user-data
- Only run once and wouldn't change

## Metadata

Includes
- AMI
- Hostname
- Public IP
- Private IP
- Instance type
- Mac address
- Security group
- Security credentials
- IAM Roles 
- and more ...



# Burstable Instances

- They earn credit if the stay below baseline
- They use credit if they stay above baseline
- There is a limit to credit accrual
- Credits do not expire on a running instance
- If your account is less than 12 months old, you can use a `t2.micro` instance for free

if credit spent are more than credits earned, the behavior depends on credit configuration mode:
- Standard Mode
	- If no more accrued credit is left, instance can't burst above baseline
- Unlimited Mode
	- If there is no more accrued credit left, instance spends surplus credits. If later instance usage falls down below baseline, it uses new credits to pay back the surplus within 24 hours. If there is any surplus after 24 hours period, instance is billed for additional usage at a flat vCPU/hour rate.

### Additional Metrics for Burstable instances

- `CPUCreditUsage` – The number of CPU credits spent during the measurement period.
    
- `CPUCreditBalance` – The number of CPU credits that an instance has accrued. This balance is depleted when the CPU bursts and CPU credits are spent more quickly than they are earned.
    
- `CPUSurplusCreditBalance` – The number of surplus CPU credits spent to sustain CPU utilization when the `CPUCreditBalance` value is zero.
    
- `CPUSurplusCreditsCharged` – The number of surplus CPU credits exceeding the [maximum number of CPU credits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-credits-baseline-concepts.html#burstable-performance-instances-credit-table) that can be earned in a 24-hour period, and thus attracting an additional charge.

The last two metrics apply only to instances configured as `unlimited`.

### VM /Import/Export
The **VM Import/Export** enables you to easily import virtual machine images from your existing environment to Amazon EC2 instances and export them back using S3 to your on-premises environment.


# ASG

### Launch Templates and Configurations
Are used to defined what is launched which could be used with ASG for automating the scaling process.
Launch templates are new and recommended compared to launch configurations.

### Scaling 

##### Dynamic Scaling
- Simple Scaling
	- One condition for scale out and one for scale in using alarm with an additional wait
- Step Scaling
	- Can add remove or set to specific capacity unit based on alarm with additional warmup
- Target Tracking
	- Maintains the metric value to a specific amount. higher values lead to adding more instanced and lower values remove instances. Could use predefined metrics or any custom metric.
	- These metrics are predefined for target tracking scaling policy:
		- `ASGAverageCPUUtilization` - Average CPU utilization of the Auto Scaling group.
		- `ASGAverageNetworkIn` - Average number of bytes received on all network interfaces by the Auto Scaling group.
		- `ASGAverageNetworkOut` - Average number of bytes sent out on all network interfaces by the Auto Scaling group.
		- `ALBRequestCountPerTarget` - Average Application Load Balancer request count per target for your Auto Scaling group.

##### Predictive Scaling
Scales using a forecast model from historical data. Uses 2 metrics:
- Load metric: use the history of this metric to determine load
- Scaling metric: Is best predictor for ideal utilization like CPU

##### Scheduled Actions
Could happen once or on recurrence for specific events. Supports Timezone and could modify the desired, min and max values.

### Predefined Metric Type
These metrics are predefined for target tracking scaling policy:

- `ASGAverageCPUUtilization` - Average CPU utilization of the Auto Scaling group.
    
- `ASGAverageNetworkIn` - Average number of bytes received on all network interfaces by the Auto Scaling group.
    
- `ASGAverageNetworkOut` - Average number of bytes sent out on all network interfaces by the Auto Scaling group.
    
- `ALBRequestCountPerTarget` - Average Application Load Balancer request count per target for your Auto Scaling group.

### Hooks

**Scale out:**
hook: `autoscaling:EC2_INSTANCE_LAUNCHING 
Could be used to load or index data
InService -> Pending -> Pending:Wait -> Pending:Proceed

**Scale in:**
hook: `autoscaling:EC2_INSTANCE_TERMINATING`
Could be used for backing up data or collecting logs
Terminating -> Terminating:Wait (1 hour timeout) -> Expires or CompleteLifecycleAction -> Terminating:Proceed -> Terminated

##### Integrations

Default: EventBridge could initiate other processes based on Hooks
Notifications on launch and terminate could be sent to SNS, SQS

##### Completing the lifecycle
When an Auto Scaling group responds to a lifecycle event, it puts the instance in a wait state and sends an event notification. You can perform a custom action while the instance is in a wait state.
Completing the lifecycle action with a result of `CONTINUE` is helpful if you finish before the timeout period has expired.

There are 2 ways:

**Manual:** Use `complete-lifecycle-action` API from CLI or SDK
**Automatic:** User user data script to automatically continue the lifecycle after bootstrap is completed. Use instance metadata to retrieve the instance ID needed for the API command with CLI.

### Standby State

- Users can transition instances from an "InService" state to a "Standby" state to facilitate updates or troubleshooting without affecting application performance.
- Instances in standby are still considered part of the Auto Scaling group but do not process load balancer traffic.
- When setting an instance to standby, the desired capacity can either remain the same or be decremented (by your choice), influencing the instance replacement strategy.
- Health checks are not performed on instances that are in a standby state, preserving their last known health status until they are resumed.
- Load balancers deregister standby instances to ensure they are not processing traffic, with optional connection draining for active requests.

https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-enter-exit-standby.html

### Behavior with unhealthy instances

- **Health Checks**: ASGs perform periodic health checks on instances using Amazon EC2 status checks or Elastic Load Balancer (ELB) health checks, depending on your configuration.
    
- **Unhealthy Instances**: If an instance fails these health checks, it is marked as unhealthy.
    
- **Termination and Replacement**: The ASG then terminates the unhealthy instance and launches a new one to replace it, ensuring that the desired capacity of the ASG is maintained.


### ASG with Load Balancers

ASG could be integrated with target groups to automatically add and remove instanced from TG based on health checks.

### Rolling Update

You can use Instance Refresh feature to perform a rolling update including changing the version of template configuration with automatic rollback.

### Warm pools
A warm pool is a pool of pre-initialized EC2 instances that sits alongside an Auto Scaling group. Whenever your application needs to scale out, the Auto Scaling group can draw on the warm pool to meet its new desired capacity. This helps you to ensure that instances are ready to quickly start serving application traffic, accelerating the response to a scale-out event.

By default, the size of the warm pool is calculated as the difference between the Auto Scaling group's maximum capacity and its desired capacity.


> [!NOTE] Limitation
> You can't add a warm pool to an Auto Scaling group that has a [mixed instances policy](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-mixed-instances-groups.html). You also can't add a warm pool to an Auto Scaling group that has launch template or launch configuration that requests Spot Instances.

### Health Checks

- EC2 Status check (always enabled)
- VPC Lattice Health check
- ALB Health check (application aware)
- Custom Health checks
	- Using `set-instance-health` API to check the health status manually
	- You can use it to set an instance healthy after it has passed some checks on boot or after a specific tag added (e.g from compliance). More info:
		- https://aws.amazon.com/blogs/compute/how-to-create-custom-health-checks-for-your-amazon-ec2-auto-scaling-fleet/

# Dedicated Host vs Instance
Dedicated host allow you to deploy your instances to the same physical host and maintain server bound licenses and compliance. Also billing is per host instead of per instance. In a dedicated host you have more visibility to sockets, cores, host id, instance placement etc

## Tips

- Termination protection prevents users from terminating an instance but doesn't prevent the ASG from terminating instances.
- Auto Scaling Hooks have default 1 hour timeout