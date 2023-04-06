# Lambda

Virtual functions, limited by time (15 min executions) run on-demand with Auto-scaling
Pros:

- Pay per request and compute time
- Generous free tier
- integrates with the whole Suite of services
- Easy monitoring on CloudWatch
- More ram improves CPU and network.

Language Support:

- NodeJs, Python, Java, C#, Golang, C#/powershell, Ruby, Custom Runtime API
- Lambda Container Image
  - ( fargaete is preferable for random images )

**Main integrations:**

- Api gateway, kinesis, DynamoDB, S3, CloudFront, Cloudwatch events / EventBridge, Cloudwatch Logs, SNS, SQS, Cognito

**Usage:**

- reacting to events: create a thumbnail or resize an image
- CRON Jobs: cloudwatch events + lambda

#### Lambda - Synchronous Invocation

_Through - CLI, SDK, API Gateway, ALB_: results are returned right away, error handling happens client side (retrie, exponential backoff, etc)

- Everytime the invocation is _User-Invoked_ it's Synchronous:
  - ELB, Gateway, CloudFront, Batch
- Some _Service Invoked_:
  - Cognito
  - Step Fucntions
- Other Services:
  - Amazon Lex, Alexa, Kinesis Data Firehose

#### Lambda integration with ALB

_To expose a lambda as a HTTP(S) endpoint_ - Lambda must be registered in a Target Group of the ALB

- HTTP request gets serialized into JSON containing:

  - Elb info
  - HTTP method & Path
  - Query String as KV pairs (json object)
  - Headers as KV pairs
  - Body: json
  - isBase64Encoded: binary

- The Lambda must return a JSON and the ALB converts Back:
  - statusCode and statusDescription
  - headers as KV
  - body
  - isBase64Encoded

**Multi-Header Values:** is an ALB setting, that converts _headers and query string parameters_ with multiple values into arrays in the event and response objects

#### Lambda Asynchronous Invocations

- When invoked by:
  - S3, SNS, CloudWatch Events / EventBridge, CodeCommit, CodePipeline, CloudwatchLogs, SES, CloudFromation, Config, IoT, IoT events
- The events go into an internal **Event Queue** for entry into the function
- On error, 3 auto-retries (1 min and 2 min wait)
- Processing must be idempotent: same results on retries
  - Uppon retry there will be duplicate log entries in CloudWatch
- The results go into an SQS or SNS queue
- Can define a DLQ - SNS or SQS for failed processing ( needs correct IAM )
- respond with a 202 for ok but pending result\

#### Lambda with EventBridge for CRON jobs

- Eventbridge configures the permissions automatically

#### Lambda with S3 events notifications

- On: created, removed, restore, replication
- filterable by name
- S3 delivers events in seconds but can take a minute or longer
- two writes to the same non-versioned object may trigger just one event notification
- To ensure event notifications for every successful write enable file versioning
- One common use case is to update metadata on files and save it to RDS or DynamoDb

#### Lambda - Event Source Mapping - trigger for streams and queues

- Kinesis Streams, SQS & SQS FIFO, DynamoDB Strems - All **Polled** (pulled) services
- The lambda function is invoked **Synchronously** to process the data

**Streams & Lambda** (kinesis or dynamoDB)

- An event source mapping creates an iterator for each shard, processes items in order
- starts with new items, from the behining or from timestamp
- processed items aren't removed from the stream
- for low traffic use bactch window to accumulate records before processing
- You can process multiple batches in parallel
- <= 10 batches per shard
- in order processing guaranteed for each partition key

**Error Handling**

- By default it the function errors, the entire batch is reprocessed until success, or items in the batch expire.
- to ensure order, processing for the shard affected by the error is paused until the error is resolved
- You can configure the event source mapping to:
  - discard old events
  - restrict the number of retries
  - split the bach on error (to work around the lambda timeout)

**Queues** sqs & sqs fifo

- Event source mapping will poll SQS ( long polling )
- Batch size: 1-10 messages
- Set the queue visibility timeout to 6x the timeout of the lambda
- To Use a DLQ -> set the DLQ on the SQS queue not Lambda ( for lambda is just on async )

**Queues & lambda**

- Lamda supports in order processing for FIFO **Scales up to the nuumber of active message groups** to maintain order
- for std queues items aren't processed in order, lambda scales up to process standart queue as **Quickly as possible**

- **Kinesis Data Streams & DynamoDB Streams:**
  - One Lambda invocation per stream shard
  - If you use parallelization, up to 10 batches processed per shard simultaneously
  - SQS Standard:
- **Lambda adds 60 more instances per minute to scale up**
  - Up to 1000 batches of messages processed simultaneously
- **SQS FIFO:**
  - Messages with the same GroupID will be processed in order
  - The Lambda function scales to the number of active message groups

### Lamba - Event and Context Objects

- When a lambda is called by an event it receives an **event Object**
- the lambda receives metadata about it's invocation and runtime in a **context object**

- Event Object

  - JSON-formatted document contains data for the function to process
  - Contains information from the invoking service (e.g., EventBridge, custom, ...) - Lambda runtime converts the event to an object (e.g., dict type in Python)
  - Example: input arguments, invoking service arguments, ...

- Context Object
  - Provides methods and properties that provide information about the invocation,
    function, and runtime environment
  - Passed to your function by Lambda at runtime
  - Example: aws_request_id, function_name, memory_limit_in_mb, ...

```python
def lambda_handler(event, context):
   # here goes the code of the lambda
```

### Lambda - Destinations (new feature)

- You can send the result of an ASYNC invocation into a **Destination**
- Destinations for failed or successful:
  - Amazon SQS
  - Amazon SNS
  - AWS Lambda
  - Amazon EventBridge bus

The new recommendation is that you use the **Destination** instead of **DLQ**

For Event source mapping, discarted events batches can go into:

- SQS (allows for a DLQ also)
- SNS

### Lambda Execution Role (IAM)

Grant permissions to access AWS services. When you use an event source mapping labda uses the execution role to read event data, so no special role there.

_The basic execution role_ allows lambda to send stuff into cloudwatch logs

**Best practice**: Create one lambda execution role per function

- To give other accounts permissions for your lambdas use Lambda Resource Policies
- A principle can access a lambda if:
  - There is an IAM policy in the principle authorizing it ( user to service )
  - OR there is a resource based policy authorizing it ( service to service )

### Lambda Environment Variables

- KV pairs in string form
- lambda service adds it's own system env vars too
- you can encrypt with KMS to store secretsv

### Lambda Logging:

CloudWatch Logs:

- AWS Lambda execution logs are stored in AWS CloudWatch Logs
- **Make sure your AWS Lambda function has an execution role with an IAM policy that authorizes writes to CloudWatch Logs**
- CloudWatch Metrics:
  - AWS Lambda metrics are displayed in AWS CloudWatch Metrics
  - Invocations, Durations, Concurrent Executions
  - Error count, Success Rates,Throttles
  - Async Delivery Failures
  - Iterator Age (Kinesis & DynamoDB Streams)

### Lambda tracing with X-Ray

- Enable **Active Tracing in the console **
- Use the SDK in code, the daemon runs automatically
- ensure propper IAM role (AWSXRayDaemonWriteAccess)
- Environment variables to communicate with X-Ray
  - \_X_AMZN_TRACE_ID: contains the tracing header
  - AWS_XRAY_CONTEXT_MISSING: by default, LOG_ERROR
  - AWS_XRAY_DAEMON_ADDRESS: the X-Ray Daemon IP_ADDRESS:PORT

### Lambda @ Edge

- Pay only for what you use
- Fully serverless
- two types: **CloudFront Functions** & **Lambda@Edge**

#### Use cases

- Website Security and Privacy
- Dynamic Web Application at the Edge
- Search Engine Optimization (SEO)
- Intelligently Route Across Origins and Data Centers
- Bot Mitigation at the Edge
- Real-time Image Transformation
- A/BTesting
- User Authentication and Authorization
- User Prioritization
- User Tracking and Analytics

**CloudFront Functions**

- Lightweight functions written in **JavaScript**
- For high-scale, latency-sensitive CDN customizations ONLY for **CloudFront viewer requests/response**
- Sub-ms startup times, **millions** of requests/second
- Used to change Viewer requests and responses:
- Viewer Request: after CloudFront receives a request from a
  viewer
- Viewer Response: before CloudFront forwards the response to the viewer
- Native feature of CloudFront **(manage code entirely within CloudFront)**

**Lambda@Edge**

- Lambda functions written in NodeJS or Python
- Scales to **1000s** of requests/second
- Used to change CloudFront requests and responses (viewer and origin):
- Author your functions in one AWS Region (us-east-1), then CloudFront replicates to its

![clipboard.png](inkdrop://file:26BlstCa5)

#### Use cases differences

| CloudFront Functions                                                                               | Lambda@Edge                                                                       |
| -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Cache key normalization                                                                            | Longer execution time (several ms)                                                |
| Transform request attributes (headers, cookies, query strings, URL) to create an optimal Cache Key | Adjustable CPU or memory                                                          |
| Header manipulation                                                                                | Your code depends on a 3rd libraries (e.g., AWS SDK to access other AWS services) |
| Insert/modify/delete HTTP headers in the request or response                                       | Network access to use external ser vices for processing                           |
| URL rewrites or redirects                                                                          | File system access or access to the body of HTTP requests                         |
| Request authentication & authorization                                                             |                                                                                   |

### Lambda in VPC

- By default, your Lambda function is launched outside your own VPC (in an AWS-owned VPC)
- Therefore it cannot access resources in your VPC (RDS, ElastiCache, internal ELB...)
- Define the VPC ID, subnets and SGs
- You can configure lambdas to access private VPCs, fore each pair Lambda will create an _ENI (Elastic Network Interface)_ in the subnet's security group. _That's how you give lambda RDS access_
- IAM: AWSLambdaVPCAccessExecutionRole

**Internet Access**

- _A Lambda function in your VPC does not have internet access_
- Deploying a Lambda function in a public subnet does not give it _internet access or a public IP_
- Deploying a Lambda function in a private subnet gives it internet
  access _if you have a NAT Gateway / Instance_
- You can use VPC endpoints to privately access AWS services without a NAT

![clipboard.png](inkdrop://file:PidUzGaZb)

### Lambda Function Configuration

- RAM:
  - From 128MB to 10GB in 1MB increments
  - The more RAM you add, the more vCPU credits you get
  - At 1,792 MB, a function has the equivalent of one full vCPU
  - _After 1,792 MB, you get more than one CPU, and need to use multi-threading in your code to benefit from it (up to 6 vCPU)_
- If your application is CPU-bound (computation heavy), increase RAM - Timeout: default 3 seconds, maximum is 900 seconds (15 minutes)
- Timeout: default 3 seconds, maximum is 900 seconds (15 minutes)

### Lambda Execution Context

- The execution context is a temporary runtime environment that initializes any external dependencies of your lambda code
- Great for database connections, HTTP clients, SDK clients...
- The execution context is maintained for some time in anticipation of
  another Lambda function invocation
- The next function invocation can “re-use” the context to execution time and save time in initializing connections objects
- The execution context includes the /tmp directory
- **Good practices**
  - run initialization code outside of the handler method, to run only once in case of context re-use
  - the /tmp directory ca be used for up to 10GB of transient storage, persists when the context is frozen but goes away when it times out, for permanent persist use S3
  - Encrypting on /tmp generate a KMS key and encrypt it yourself

### Lambda Layers

- A way to add pre-compiled stuff to the runtime of lambdas without repackaging everything
- Enables Custom Runtimes ( C++ and Rust )
- Externalize dependencies for Re-use

### File system mounting into lambdas

- If the λ runs inside a vpc it can access EFS
- Use the EFS AccessPoints for access
- λ mount efs to local directory in initialization
- EFS has connection limits that might break as λ scale

  ![clipboard.png](inkdrop://file:E7948MR2M)

#### λ Concurency

- Concurrency limit: 1000 concurrent executions
  - that can be limited: set a "reserved concurrency"
    - if you need higher than 1000 open a ticket
  - Each invocation over the limit triggers a throttle:
    - if synchronous => Throttle error 429
    - if async => retries and goes into DLQ

**If you dont set the limit**
you get up to 1000 λs per ACCOUNT. So if a burst of clients come into the ALB
that might create 1000 λs and leave the rest of the account starved

#### Concurrent in Async

If you get throttled λ retries again for up to 6 hours in exponential backoff

### Cold Starts:

- A new instance needs to load the code and run initialization

**Provisioned Concurrency**

- concurrency allocated before the λ runs
- no cold starts, all invocations have low latency
- Application auto scaling can mana

#### λ function dependencies

-**you need to install the packages alongside the code and zip it together**

- <= 50Mb straight into λ, > 50Mb goes into S3
- natrive libraries work when compiled on amazon linux
- AWS SDK come by default in every λ

### Making λs through CloudFormation - Inline

- Inline functions are very simple
- Use the Code.ZipFile proper ty
- You cannot include function dependencies with inline functions

### Making λs through CloudFormation - Through S3

- You must store the Lambda zip in S3
- You must refer the S3 zip location in the CloudFormation `code` attribute
  - S3Bucket
  - S3Key: full path to zip
  - S3ObjectVersion: if versioned bucket
- _If you update the code in S3, but don’t update S3Bucket, S3Key or S3ObjectVersion, CloudFormation won’t update your function_
- For cross account S3 access, use execution roles in the λ and a bucket policy in the S3 account

### λ container images

- complex dependencies and images up to 10GB from ECR
- the base image must implement the lambda runtime API
- Base images are available for Python, Node.js, Java, .NET, Go, Ruby
- Can create your own image as long as it implements the Lambda Runtime API
- Test the containers locally using the Lambda Runtime Interface Emulator
- Unified workflow to build apps

**Best Practices**

- Strategies for optimizing container images:
  - Use AWS-provided Base Images
    - Stable,BuiltonAmazonLinux2,cachedbyLambdaservice
  - Use Multi-Stage Builds
    - Build your code in larger preliminary images, copy only the artifacts you need in your final container image, discard the preliminary steps
  - Build from Stable to Frequently Changing
    - Make your most frequently occurring changes as late in your Dockerfile as possible
  - Use a Single Repository for Functions with Large Layers
    - ECR compares each layer of a container image when it is pushed to avoid uploading and storing duplicates
- Use them to upload large Lambda Functions (up to 10 GB)

#### λ Versions

- When you work on a Lambda function, we work on $LATEST
- When we’re ready to publish a Lambda function, we create a version
- Versions are immutable and increasing
- Version = code + configuration (immutable) + ARN
- Each version of the lambda function can be accessed

#### λ Aliases

- Aliases are ”pointers” to Lambda function versions
- We can define a “dev”, ”test”, “prod” aliases and have them point at different lambda versions
- Aliases are mutable, _can't reference Aliases_ and have their own ARNs
- Aliases enable Canary deployment by assigning weights to lambda functions
  - 10% to blue, 90% to green type stuff
- Aliases enable stable configuration of our event triggers / destinations
- you can keep piling on versions and referencing the aliases

#### Lambda & CodeDeploy

- CodeDeploy can help you automate traffic shift for Lambda aliases
- Feature is integrated within the SAM framework
- Linear: grow traffic every N minutes until 100%
  - Linear10PercentEvery3Minutes
  - Linear10PercentEvery10Minutes
- Canary: try X percent then 100%
  - Canary10Percent5Minutes
  - Canary10Percent30Minutes
- AllAtOnce: immediate
- Can create Pre & Post Traffic hooks to check the health of the Lambda function

**AppSpec.yml**

- Name (required) – the name of the Lambda function to deploy
- Alias (required) – the name of the alias to the Lambda function
- CurrentVersion (required) – the version of the Lambda function traffic currently points to
- TargetVersion (required) – the version of the Lambda function traffic is shifted to

### λ function Url

- Dedicated, unique and immutable HTTP(S) endpoint for your Lambda function
- `https://<url-id>.lambda-url.<region>.on.aws` (dual-stack IPv4 & IPv6)
- Invoke via a web browser, curl, Postman, or any HTTP client
- Access your function URL through the public Internet only
- Doesn’t support PrivateLink (Lambda functions do support)
- Supports Resource-based Policies & CORS configurations
- Can be applied to any function alias or to $LATEST (can’t be applied to other function versions)
- Create and configure using AWS Console or AWS API
- Throttle your function by using Reserved Concurrency

**Function URL Security**

- Resource-based Policy
  - Authorize other accounts / specific CIDR / IAM principals
- Cross-Origin Resource Sharing (CORS)
  - If you call your Lambda function URL from a different domain
- AuthType NONE – allow public and unauthenticated access
  - Resource-based Policy is always in effect (must grant public access)
- AuthType AWS_IAM – IAM is used to authenticate and authorize requests
  - Both Principal’s Identity-based Policy & Resource-based Policy are evaluated
  - Principal must have lambda:InvokeFunctionUrl permissions
  - _Same account – Identity-based Policy OR Resource-based Policy as ALLOW_
  - _Cross account – Identity-based Policy AND Resource Based Policy as ALLOW_

### CodeGuru Profiling

- Gain insights into runtime performance of your Lambda functions using CodeGuru Profiler
- CodeGuru creates a Profiler Group for your Lambda function
- Supported for Java and Python runtimes
- Activate from AWS Lambda Console
- When activated, Lambda adds:
  - CodeGuru Profiler layer to your function
  - Environment variables to your function
  - AmazonCodeGuruProfilerAgentAccess policy to your function

### Lambda Limits - per region

- Execution:
  - Memory allocation: 128 MB – 10GB (1 MB increments)
  - Maximum execution time: 900 seconds (15 minutes)
  - Environment variables (4 KB)
  - Disk capacity in the “function container” (in /tmp): 512 MB to 10GB
  - Concurrency executions: 1000 (can be increased)
- Deployment:
  - Lambda function deployment size (compressed .zip): 50 MB
  - Size of uncompressed deployment (code + dependencies): 250 MB
  - Can use the /tmp directory to load other files at startup
  - Size of environment variables: 4 KB

### Best Practices

- Perform heavy-duty work outside of your function handler
  - Connect to databases outside of your function handler
  - Initialize the AWS SDK outside of your function handler
  - Pull in dependencies or datasets outside of your function handler
- Use environment variables for:
  - Database Connection Strings, S3 bucket, etc... don’t put these values in your code
  - Passwords, sensitive values... they can be encrypted using KMS
- Minimize your deployment package size to its runtime necessities.
  - Break down the function if need be
  - Remember the AWS Lambda limits
  - Use Layers where necessary
- Avoid using recursive code, never have a Lambda function call itself
