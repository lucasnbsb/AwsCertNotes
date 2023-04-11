# SAM - Serverless Applicatiom Model

- Framework for developing and deploying serverless applications
- All the configuration is YAML code
- Generate complex CloudFormation from simple SAMYAML file
- Supports anything from CloudFormation: Outputs, Mappings, Parameters, Resources...
- Only two commands to deploy to AWS
- SAM can use CodeDeploy to deploy Lambda functions
- _SAM can help you to run Lambda, API Gateway, DynamoDB locally_

#### Recipes

- Transform Header indicates it’s SAM template:
  - Transform: 'AWS::Serverless-2016-10-31'
- Supported resources
  - AWS::Serverless::Api
  - AWS::Serverless::Application
  - AWS::Serverless::Function
  - AWS::Serverless::HttpApi
  - AWS::Serverless::LayerVersion
  - AWS::Serverless::SimpleTable
  - AWS::Serverless::StateMachine
- the transform and resources headers are the only mandatory ones

- Package & Deploy:
  - aws cloudformation package / sam package
  - aws cloudformation deploy / sam deploy

![clipboard.png](inkdrop://file:zjSUuqNFp)

### SAM – CLI Debugging

- Locally build, test, and debug your serverless applications that are defined using AWS SAM templates
- Provides a lambda-like execution environment locally
- SAM CLI + AWS Toolkits => step-through and debug your code
- Supported IDEs:AWS Cloud9,Visual Studio Code, JetBrains, PyCharm, IntelliJ, ...
- AWS Toolkits: IDE plugins which allows you to build, test, debug, deploy, and invoke Lambda functions built using AWS SAM

### Sam policy templates

- List of templates to apply permissions to your Lambda Functions
- Important examples:
  - **S3ReadPolicy**
  - **SQSPollerPolicy**
  - **DynamoDBCrudPolicy**

### SAM and code deploy

_SAM uses CodeDeploy natively to update Lamdas_

- Traffic shifting with aliases
- Pre and Post traffic hooks to validate deployment
- Automated rollback through cloudwatch Alarms

### SAM Local Capabilities

- Locally start AWS Lambda
  - _sam local start-lambda_
  - Starts a local endpoint that emulates AWS Lambda
  - Can run automated tests against this local endpoint
- Locally Invoke Lambda Function
  - _sam local invoke_
  - Invoke Lambda function with payload once and quit after invocation completes
  - Helpful for generating test cases
  - If the function make API calls to AWS, make sure you are using the correct --profile option
- Locally Start an API Gateway Endpoint
  - sam local start-api
  - Starts a local HTTP server that hosts all your functions
  - Changes to functions are automatically reloaded
- Generate AWS Events for Lambda Functions
  - sam local generate-event
  - Generate sample payloads for event sources
  - S3, API Gateway, SNS, Kinesis, DynamoDB...

### SAM – Exam Summary

- SAM is built on CloudFormation
- SAM requires the Transform and Resources sections
- Commands to know:
  - sam build: fetch dependencies and create local deployment artifacts
  - sam package: package and upload to Amazon S3, generate CF template
  - sam deploy: deploy to CloudFormation
  - SAM Policy templates for easy IAM policy definition
- SAM is integrated with CodeDeploy to do deploy to Lambda aliases

### Serverless Application Repository (SAR)

- Managed repository for serverless applications
- The applications are packaged using SAM
- Build and publish applications that can be re-used by organizations
  - Can share publicly
  - Can share with specific AWS accounts
- This prevents duplicate work, and just go straight to publishing
-
