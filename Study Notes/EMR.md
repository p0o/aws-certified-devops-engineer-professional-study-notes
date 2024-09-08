
### Hadoop and MapReduce 
Hadoop is a framework used for storing and processing large datasets across clusters of computers. It consist of two components:
- HDFS for storage
- MapReduce for processing

MapReduce is a programming model within Hadoop designed to process large data in two phases. Map phase is filtering and processing data (in smaller chunks) and reduce is aggregating and summarizing them. 

### EMR 
A managed implementation of Apache Hadoop. It includes the ecosystem including Spark, HBase, Presto, Flink, Hive and Pig.

Could be operated in 2 modes:
- Long Running
- Transient (ad-hoc)

Runs in one AZ only

Could run in EC2 Spot, Instance Fleet, Reserved, On-demand

Input S3 -> HDFS -> Output S3

Master  Node
- Should be at least 1
- Cluster has a master node which manages cluster and its health. It also distribute workload and act as NAME node within MapReduce. You can SSH into this node.
- Master node could be launched up to 3 to add high availability
- Everything fail if it fails

Core Nodes
- Could be 0 or more
- They are similar to data nodes for HDFS
- Run task trackers and can run map and reduce tasks
- Task tracking tasks should not be disrupted

HDFS
- It's lifetime is tied to the cluster
- Managed by core nodes
- Not zonal resilient
- ephemeral

Task Nodes
- Optional
- No HDFS Involvement
- Don't run task trackers
- Run the tasks (map and reduce)
- Ideal for Spot Instances

EMRFS
- Uses S3 to provide resilience storage
- Persists past the lifetime of the cluster
- Ressillient to core nodes failure
- 