# Messaging: SQS, Kinesis, SNS

- SQS: queue
- SNS: pub/sub
- Kinesis: real time streaming

## SQS - message queue

- Standart and FIFO queues
- A producer sends the message into the queue and a consumer **Polls** (pull model) the queue and consumers the messages
- both producers and consumers can be pools
- SQS is fully managed
- Unlimited throuput
- retention: 4 (default) to 14 days (max)
- low latency (10ms)
- message <= 256KB
- can have duplicate messages (at least one delivery, ocasional delivery)
- can have out of order messages (best effort ordering)

#### SQS - Producing messages

- Produce the message with the SQS
- API: `SendMessage`
- message persisted until a consumer deletes it `DeleteMessage`

#### Common use cases

- SQS queues are most commonly used to decouple an applications web tier and worker tiers
  ![clipboard.png](inkdrop://file:WHSW7EigP)

#### SQS - Security

- Encryption

  - in-flight with HTTPS api
  - at rest with KMS
  - client-side if the client does it itself

- Access control: IAM polcies on the SQS api
- SQS Access policies ( similar to S3 policies )
  - enable cross-account
  - enable other services to write to the queue

#### Visibility timeout

- when the message is polled by a consumer it becomes invisible to other consumers and the visibility timeout begins
- by default, it's 30 secconds, but if the message is not processed during the timeout it is returned
- if a consummer is not able to do it in time there is an api: `ChangeMessageVisibility`

#### Dead Letter Queue DLQ

- you set a threshold of how many times a message comes back
- after the MaximumReceives it goes in the DLQ for debugging
- the DLQ has to be the same type as the main queue ( fifo or standard )

#### DLQ redrive to source

- let's you consume messages from the DLQ manually and then redrive them into the queue after changes

#### Delay Queue

- you can delay up to 15 minutes
- default is 0
- there is a default queue level and a message delay that can be set on send

#### Long Polling:

- when the consumer polls and there is nothing there is an option to wait for a while, that's long polling
- it decreases the number of API calls and the latency
- 1 ~ 20 seconds ( more preferable )
- Long Polling is a good practice
- Enabled at the queue level or API call level

#### SQS Extended Client - Larger messages

- is a java library that uses a S3 bucket as a repo and sends the link

#### SQS queue API

- **CreateQueue**
- **DeleteQueue**
- **PurgeQueue**: clears the queue
- **ReceiveMessage & DeleteMessage**
- **MaxNumberOfMessages**
- **ReceiveMessageWaitTimeSeconds**: long polling
- **ChangeMessageVisibility**: prevent going back

#### FIFO queue

- Limited throughput: 300 msg/s without batching 3000 with
- Exactly-once send ( by removing duplicates )
- Messages are processed in order by the consummer
- You need to name it **`*.fifo`**
- Deduplication
  - content-based using a SHA-256 hash on the body
  - Explicitly by providing a unique messageId
- if you specify the same value of MessageGroupId in a fifo queue you can only have one consumer and everything is in order
- Ordering a subset can be done with different values of MessageGroupId
  - order is only guaranteed within groups
  - each group can have a consumer

## SNS: pub/sub queue

- When you need multicast
- the _event producer_ only sends to one **Topic**
- the **event receivers** listen to all messages in the **topic**
- 12,5mi subscriptions per topic 100k topics per account
- Subscribers can be of many types
  - sqs, lambda, http endpoints, email, sms and push notifications, firehose

#### How to publish:

- create a topic -> push to topic

- Direct Publish: (mobile platforms)
  - create a platform app
  - create a platform endpoint
  - publish to the platform endpoint

#### Security:

- in-flight with HTTPS api
- at rest with KMS keys
- Client side if you do it yourself
- IAM policies for the API
- SNS policies similar to S3

#### SNS + SQS for Fan Out

![clipboard.png](inkdrop://file:K7DqS5chD)

- decoupled
- push once
- can add more sqs subs
- cross region dellivery ( messages can be sent into SQS queues in other regions
- Make sure the Queue Access Policy allows for SNS to write
- Other applications: S3 event fan out
- Firehose to S3: SNS has direct integration with fire hose so it's a good bridge

#### FIFO Topics

similar to SQS fifo

- Ordering
- Deduplication
- SQS FIfo queues as subscribers
- limited throughput
- enables Fifo fanout with Fifo SQS queues

#### Message Filtering

with filter policies you can chose who receives messages by body

## Kinesis

- Data streaming in real time
- Streams are broken down in Shards, you can scale the ammount of shards
- retention 1 ~ 365 days
- ability to replay data in retention
- data can't be deleted (immutability)
- Producers send data into kinesis with records:
  - Record:
    - Partition key: gets hashed and sent to the same shard based on hash value
    - Data Block
  - Rate: 1MB/sec/shard
  - API: `PutRecord`
  - the partition key must be well distributed to balance out the shard usage
  - _ProvisionedThroughputExceded_: when you produce too much into a single shard
    - implement exponential backoff
    - increase the number of shards
    - use a better distributed partition key
- Consumers receive the data
  - Record
    - Partition key
    - sequence no.
    - data blob
  - Rate: 2 MB/sec/shard for all consumers OR
    - 2 MB/sec/shard per consumer (enhanced fan out mode)
  - consumers can be
    - Lambdas
      - calls `GetBrach()`
      - supports classic & enhanced
      - can configure batch size and batch window
      - retry on error
      - can process up to 10 batches per shard at the same time
    - Kinesis
      - Data Analytics
      - Data Firehose
    - Custom Consumer thorugh the SDK
      - Classic:
        - calls `getRecords()` every shard outputs up to 2MB/sec for all consumers
        - is a pulled method
        - low number of consumers
        - latency ~ 200ms
        - cheaper
        - returns up to 10MB or 10k records then throttles for 5 seconds
      - Enhanced Fan-out:
        - calls `SubscribeToShard()` each consumer gets 2MB/sec per
        - is a pushed method using HDB2
        - Multiple consuming apps for the same stream
        - latency ~70ms
        - More expensive
        - soft limit of 5 consumer applications per data stream
    - Kinesis Client Library (KCL): simplifies reading from data stream
      - java library to read records from kinesis with distributed applications
      - each shard is read by one KCL instance
      - progress is checkpointed into DynamoDB ( needs IAM )
      - track workers through DynamoDB
      - Runs on EC2, Beanstalk or on-premises
      - Reads in order at the shard level
      - 1.x supports shared consumer
      - 2.x supports shared and enhanced
- Capacity Modes
  - Provisioned:
    - you chose the no. of shards provisioned, scale mannualy or using API
    - 1MB/s/shard in
    - 2MD/s/shard out
    - you pay _per shard per hour_
    - use it if you can plan the capacity events
  - Od Demand Mode
    - is managed, no need to provision
    - Default capacity 4MB/s in or 4000 records per second)
    - scales automatically based on observed _peak throughput_
    - pay _per stream peer hour_ & data in/out per _GB_
- Security
  - access / auth through IAM
  - Encryption
    - in-flight using HTTPS endpoint
    - at rest with KMS
    - client side by you
  - VPC endpoints for access within VPC
  - CloudTrail monitors all api calls
- Operations
  - Shard Splitting
    - increases the stream capacity
    - used to divide a "hot shard"
    - the hot shard is closed when data expires and 2 new ones take it's place
    - no auto-scaling
    - can't split into more than 2
  - Shard Merging
    - decrease the capacity and cost
    - groups 2 cold shards
    - can't merge more than 2
    - old shards closed when data expires

## Kinesis Data Firehose

- can take any type of producer + Kinesis, IOT and CloudWatch logs & events
- Writes **In Batches in the destination**
- Destinations
  - AWS
    - S3
    - RedShift (copy through S3)
    - OpenSearch
  - Partners
    - DataDog, MongoDb, Splunk, New Relic
  - Custom Destinations
    - HTTP endpoint
- After write everything can be sent into a S3 bucket (pass or fail)
- Pay only for the data going through the hose
- Near Real Time
  - 60 seconds minimum latency for non full batches
  - Minimum 1MB data at a time
- Supports many formats, conversions and transformations
- Custom transformations through AWS Lambda'
- No data storage, no replay

| Data Streams                                  | Firehose                                   |
| --------------------------------------------- | ------------------------------------------ |
| Ingest at scale                               | Load streaming into S3/Redshift/OpenSearch |
| write custom code for producers and consumers | fully managed                              |
| real time ~ 200ms                             | near real-time (60 sec buffer time)        |
| You manage scaling                            | Auto scaling                               |
| Storage for 1~365 days                        | no storage                                 |
| supports replay                               | no replay                                  |

### Kinesis Data Analytics

- For SQL Applications
  - reads from streams or firehose
  - apply sql statements to run analytics
  - reference data in S3 to enrich the data
  - send into:
    - Data Stream
    - Firehose
  - fully managed, auto scaled
  - pay for actual consumption rate
- For Apache Flink
  - you write Flink apps in java, scala or SQL
  - Read from:
    - Data Streams
    - Amazon MSK (kafka)
  - Run on a managed cluster or AWS
    - Parallel computation, auto-scaling
    - backups
    - much more powerfull than SQL

#### Ordering Data into Kinesis

- Everything depends on the partition key
  - you can have data from the same published be on the same shard always in order

#### Ordering into SQS

- No ordering in standart
- In SQS FIFO it groups by the Group ID and runs FIFO by ID
  - you can even have one groupID per producer so each consumer gets one

| SQS                              | SNS                            | Kinesis                              |
| -------------------------------- | ------------------------------ | ------------------------------------ |
| Pulled                           | Pushed to many subs            | Standard Pull data, 2MB per shard    |
| Data deleted after consumed      | up to 12.5 mil subs            | Enhanced 2mb per shard per consumer  |
| AS many consumer as we want      | no persistence                 | 1 to 365 days of persistence         |
| No need to provision             | pub/sub                        | Can replay data                      |
| ordering guaranteed only on FIFO | up to 100k topics              | Real-time big data analytics and ETL |
| indivudual message delay         | no need to provision           | Ordering at the shard level          |
| persistence for up to 14 days    | integrate with SQS for fan out | Data expires after X days            |
|                                  | FIFO capable                   | Provisioned mode or on demand mode   |
