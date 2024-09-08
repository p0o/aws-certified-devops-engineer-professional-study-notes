## Secondary Indexes

Replicates the table with new attribute as partition key. It requires attribute projection to move the attributes to the secondary index table. Options for attribute projection:

- Only Keys: only partition and sort key are moved
- Include: you can add custom non-key attributes
- All: all attributes from the base table are replicated

We have 2 types of secondary indexes:

- Global Secondary Index
	- Has independent read and write capacity units
	- Any update to base table is reflected asynchronously to GSI with eventual consistency
- Local Secondary Index (LSI)
	- Maintains the same partition key as base table but with a new sort key
	- Still need to project attributes
	- Could only be created while creation of table
	- RCU and WCU are added to base table
	- Could use "fetching" to get non-projected attributes automatically by DynamoDB
	- 

## DynamoDB Streams

Expires in 24 hours
Does not impact performance of the table
Each record appears only once in stream
Records are ordered
Could be sent to lambda

StreamTypes
1. KEYS_ONLY
2. NEW_IMAGE
	1. new change is written into the stream (entire item)
3. OLD_IMAGE
	1. Item before the change is written into the stream


Using the Amazon Kinesis Adapter is the recommended way to consume streams from Amazon DynamoDB.


## TTL

- Is free
- Does not consume write capacity
- Expired TTL is not deleted immediately, it might take up to 48 hours

## Read Consistency

1. Eventually Consistent  (default)
	1. may not reflect the latest write immediately but offers better performance and lower cost.
2. Strongly Consistent
	1. This ensures that the read always reflects the most recent write

In order to perform strong consistent read:

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('YourTableName')

# Strongly consistent read
response = table.get_item(
    Key={'YourPrimaryKey': 'YourKeyValue'},
    ConsistentRead=True
)

item = response.get('Item')
```


### Calculating RCU and WCU

For provisioned mode tables, you specify throughput capacity in terms of read capacity units (RCUs), as discussed earlier:

- 1 Read Capacity Unit (RCU) =  1 strongly consistent read of up to 4 KB/s = 2 eventually consistent reads of up to 4 KB/s per read.
- 2 RCUs = 1 transactional read request (one read per second) for items up to 4 KB.
- For reads on items greater than 4 KB, total number of reads required = (total item size / 4 KB) rounded up.

For provisioned mode tables, you also specify throughput capacity in terms of write capacity units (WCUs):

- 1 Write Capacity Unit (WCU) = 1 write of up to 1 KB/s.
- 2 WCUs = 1 transactional write request (one write per second) for items up to 1 KB.
- For writes greater than 1 KB, total number of writes required = (total item size / 1 KB) rounded up

## Transactions

They cost double the normal RCU or WCU

1. TransactWriteItems
	* Groups up to 25 actions in a single or nothin operation
	* Can use `PutItem` `UpdateItem` `DeleteItem` and `ConditionCheck` operations
2. TransactGetItems
	* group up to 25 `Get items` and `ConditionCheck` (should not exceed 4MB) 

## Global Tables

Last Writer Wins is used for consensus in the situation of concurrent write on same item
Failover to other tables is not done automatically
