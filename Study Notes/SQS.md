
- Maximum inflight messages are 120k for standard queues and 20k for FIFO
- Maximum messages in a queue is unlimited
- Use DLQ if you want to isolate messages with error in order to process later
- You can configure the Amazon SQS message retention period to a value from **1 minute to 14 days**. The default is 4 days.