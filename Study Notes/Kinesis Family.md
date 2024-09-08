# Kinesis Data Stream

Kinesis data stream is:

Stream
- Data is restored as Kinesis Data Record (1 MB)
- Data retention period is the length of time that data is available in stream. the default is 24 hours. Maximum is 365 days. (additional cost)

A set of shards
Each shard has a sequence of records
Each record has seq number, parttion key and data blob 
Data blob could be up to 1 MB

Capacity is on-demand or provisioned
- on-demand automatically manages the shards
- provisioned you should specify the number of shards

2 types of consumers:
- shared fan-out
- enhanced fan-out

each shard can:
- support up to 5 tx per second for read
- up to maximum 2 MB/s data read rate
- up to 1000 records per second for write
- up to 1 MB/s data write rate

Partition key is:
- used to group data by shard in a stream

Sequence number:
- unique per partition key within its shard
- assigns automatically after client.putRecords
- it increases for the same partition over time

Kinesis Client Library:
- Used in apps to enable fault tolerant consumption of data from stream
- It ensures that for every shard, there is a record processor running.
- It uses DynamoDB to store control data
- Pushes custom metrics to cloudwatch

ASG for auto-scaling metric should use `MillisBehindLatest`, which represents the time that the current iterator is behind from the latest record (tip) in the shard.


![[Pasted image 20240810181158.png]]


### KDS vs SQS
- Kinesis is about ingestion, SQS is about decoupling, worker pools and async communication
- SQS messages have no persistence or time window
- Kinesis is for huge scale ingestion to multiple consumers within rolling window
- SQS is usually 1 group of producers and 1 groups of consumers

## Errors

`ProvisionedThroughputExceededException`: request rate for the stream is too high, or the requested data is too large for the available throughput. Reduce the frequency or size of your requests.

Each PutRecords request can support up to 500 records. Each record in the request can be as large as 1 MiB, up to a limit of 5 MiB for the entire request.

To address this error:

- Reshard your stream to increase the number of shards in the stream.
- Reduce the frequency or size of your requests.
- Distribute read and write operations as evenly as possible across all of the shards in Data Streams.
- Use an error retry and exponential backoff mechanism.


https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html

[https://docs.aws.amazon.com/streams/latest/dev/key-concepts.html](https://docs.aws.amazon.com/streams/latest/dev/key-concepts.html)

[https://docs.aws.amazon.com/streams/latest/dev/monitoring-with-kcl.html](https://docs.aws.amazon.com/streams/latest/dev/monitoring-with-kcl.html)

# Firehose

### Sources
- AWS SDK
- AWS Lambda
- AWS CloudWatch Logs
- AWS CloudWatch Events
- AWS Cloud Metric Streams
- AWS IOT
- AWS Eventbridge
- Amazon Simple Email Service
- Amazon SNS
- AWS WAF web ACL logs
- Amazon API Gateway - Access logs
- Amazon Pinpoint
- Amazon MSK Broker Logs
- Amazon Route 53 Resolver query logs
- AWS Network Firewall Alerts Logs
- AWS Network Firewall Flow Logs
- Amazon Elasticache Redis SLOWLOG
- Kinesis Agent (linux)
- Kinesis Tap (windows)
- Fluentbit
- Fluentd
- Apache Nifi
- Snowflake

### Destinations
- Amazon OpenSearch Service
- Amazon OpenSearch Serverless
- Amazon Redshift
- Amazon S3
- Coralogix
- Datadog
- Dynatrace
- Elastic
- HTTP Endpoint
- Honeycomb
- Logic Monitor
- Logz.io
- MongoDB Cloud
- New Relic
- Splunk
- Splunk Observability Cloud
- Sumo Logic
- Snowflake
- Apache Iceberg Tables

### Backup in S3
Amazon Data Firehose uses Amazon S3 to backup all or failed only data that it attempts to deliver to your chosen destination.


> [!NOTE] Important
> Backup settings are only supported if the source for your Firehose stream is Direct PUT or Kinesis Data Streams.

You can specify the S3 backup settings for your Firehose stream if you made one of the following choices.

- If you set Amazon S3 as the destination for your Firehose stream and you choose to specify an AWS Lambda function to transform data records or if you choose to convert data record formats for your Firehose stream.
- If you set Amazon Redshift as the destination for your Firehose stream and you choose to specify an AWS Lambda function to transform data records.
- If you set any of the following services as the destination for your Firehose stream – Amazon OpenSearch Service, Datadog, Dynatrace, HTTP Endpoint, LogicMonitor, MongoDB Cloud, New Relic, Splunk, or Sumo Logic, Snowflake, Apache Iceberg Tables.

### Firehose with VPC

You can use an interface VPC endpoint (AWS PrivateLink) to keep traffic between your Amazon VPC and Amazon Data Firehose from leaving the Amazon network. Interface VPC endpoints don't require an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection