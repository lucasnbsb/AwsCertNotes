# CDK

- Define your cloud infrastructure using a familiar language:
  - JavaScript/TypeScript,Python,Java,and.NET
- Contains high level components called _constructs_
- The code is “compiled” into a CloudFormation template (JSON/YAML) ( _TYPESAFETY!_ )
- You can therefore deploy infrastructure and application runtime code together
  - Great for Lambda functions
  - Great for Docker containers in ECS / EKS

![clipboard.png](inkdrop://file:Vv2cCrgzj)

### CDK vs SAM

- SAM:
  - Serverless focused
  - Write your template declaratively in JSON or YAML
  - Great for quickly getting started with Lambda
  - Leverages CloudFormation
- CDK:
  - _All AWS services_
  - Write infra in a programming language JavaScript/TypeScript, Python, Java, and .NET
  - Leverages CloudFormation

### CDK + SAM

- SAM CLI can be used to test CDK apps locally
- Run `CDK synth` to generate the cloud formation template
- run `sam local invoke -t formation.template.json myFunction` with the SAM CLI

#### CDK Constructs

- CDK Construct is a component that encapsulates everything CDK needs to create the final CloudFormation stack
- Can represent a single AWS resource (e.g., S3 bucket) or multiple related resources (e.g., worker queue with compute)
- AWS Construct Library
- A collection of Constructs included in AWS CDK which contains Constructs for
  every AWS resource
- Contains 3 different levels of Constructs available (L1, L2, L3)
- Construct Hub – contains additional Constructs from AWS, 3rd parties, and open-source CDK community

#### L1 constructs (1-1 no abstraction into cloudformation):

- Can be called CFN Resources which represents all resources directly
  available in CloudFormation
- Constructs are periodically generated from CloudFormation Resource Specification
- Construct names start with Cfn (e.g., CfnBucket)
- You must explicitly configure all resource properties

#### CDK Constructs – Layer 2 Constructs (L2)

- Represents AWS resources but with a higher level (intent-based API)
- _Similar functionality as L1 but with convenient defaults and boilerplate _
  - You don’t need to know all the details about the resource properties
- Provide methods that make it simpler to work with the resource (e.g., bucket.addLifeCycleRule())

### CDK Constructs – Layer 3 Constructs (L3)

- Can be called Patterns, which represents multiple related resources
- Examples:
  - _aws-apigateway.LambdaRestApi_ represents an API Gateway backed by a Lambda function
  - _aws-ecs-patterns.ApplicationLoadBalancerFargateService_ which represents an architecture that includes a Fargate cluster with Application Load Balancer

### Common commands

![clipboard.png](inkdrop://file:4rzAxzBs2)

### CDK – Bootstrapping

- The process of provisioning resources for CDK before you can deploy CDK apps into an AWS environment
- AWS Environment = account & region
- CloudFormation Stack called CDKToolkit is created and contains:
  - S3 Bucket – to store files
  - IAM Roles – to grant permissions to perform deployments
- You must run the following command for each new environment:
- cdk bootstrap aws://<aws_account>/<aws_region>
- Otherwise, you will get an error “Policy contains a statement with one or more invalid principal”

### CDK – Testing

- To test CDK apps, use CDK Assertions Module combined with popular test frameworks such as Jest (JavaScript) or Pytest (Python)
- Verify we have specific resources, rules, conditions, parameters...
- Two types of tests:
  - Fine-grained Assertions (common) – test specific aspects of the CloudFormation template (e.g., check if a resource has this property with this value)
  - SnapshotTests–testthesynthesizedCloudFormation template against a previously stored baseline template
- To import a template
  - Template.fromStack(MyStack) : stack built in CDK
  - Template.fromString(mystring) : stack build outside CDK
