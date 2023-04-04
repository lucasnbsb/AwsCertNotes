# Amazon Simple Mail Sercice (SES)

- Managed, secure and scalable
- Allows inbound/outbound emails
- Reputation dashboard, performance insights, anti-spam feedback
- Provides statistics such as email deliveries, bounces, feedback loop results, email open
- Supports DomainKeys Identified Mail (DKIM) and Sender Policy Framework (SPF)
- Flexible IP deployment: shared, dedicated, and customer-owned IPs
- Send emails using your application using AWS Console, APIs, or SMTP
- Use cases: transactional, marketing and bulk email communications

# Amazon OpenSearch Service

- Successor to Amazon ElasticSearch
- In DynamoDB, queries only exist by primary key or indexes, With OpenSearch, you can search any field, even partially matches
- It’s common to use OpenSearch as a complement to another database
- OpenSearch **requires a cluster** of instances (_not serverless_)
- _Does not support SQL_ (it has its own query language)
- Ingestion from Kinesis Data Firehose, AWS IoT, and CloudWatch Logs
- Security through Cognito & IAM, KMS encryption,TLS
- Comes with OpenSearch Dashboards (visualization)

#### OpenSearch Patterns

![clipboard.png](inkdrop://file:nX89wySUy)

### ![clipboard.png](inkdrop://file:1-Yrkn0lC)

# Amazon Athena

- Serverless query service to analyze data stored in Amazon S3
- Uses standard SQL language to query the files (built on Presto)
- Supports: CSV,JSON,ORC,Avro,andParquet
- Pricing: $5.00 per TB of data scanned
- Commonly used with Amazon Quicksight for reporting/dashboards
- Use cases: Business intelligence / analytics / reporting, analyze & query VPC Flow Logs, ELB Logs, CloudTrail trails, etc...
- _Exam Tip: analyze data in S3 using serverless SQL, use Athena_

- Use columnar data for cost-savings (less scan)
- Apache Parquet or ORC is recommended
- Huge performance improvement
- Use Glue to convert your data your Parquet or ORC
- Compress data for smaller retrievals (bzip2, gzip, lz4, snappy, zlip, zstd...)
- Partition datasets in S3 for easy querying on virtual columns
- s3://yourBucket/pathToTable /<PARTITION_COLUMN_NAME>=<VALUE>
  /<PARTITION_COLUMN_NAME>=<VALUE> /<PARTITION_COLUMN_NAME>=<VALUE>
  /etc...
- Example:s3://athena-examples/flight/parquet/year=1991/month=1/day=1/
- Use larger files (> 128 MB) to minimize overhead © Stephane Maarek
