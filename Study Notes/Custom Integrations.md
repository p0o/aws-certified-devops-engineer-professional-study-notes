
### ALB Origin Protection with WAF
Cloudfront → Custom HTTP Header (X-Origin-Verify: xxx) → WAF Filtering Rule → ALB

Secrete Manager → Auto Rotate → Lambda → Updates Cloudfront and WAF

### WAF Logs
WAF → Cloudwatch || Data Firehose || S3 Bucket

### Control Tower with Config
Control Tower → Event Bridge → SQS (FIFO) → Lambda → Cloudformation StackSets → Deploy AWS Config Conformance Packs into new Account

