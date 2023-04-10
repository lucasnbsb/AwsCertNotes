# CI/CD on AWS - CodeBuild and CodeDeploy

- push code (repo) -> build and test (build server) -> deploy (deployment server)

### CodeCommit

- **Authentication**
  - _SSH keys / HTTPS_ with git credentials or aws cli credentials for IAM users
- **Authorization**: _IAM policies_
- **Encryption**:
  - repos are encrypted at-rest with KMS
  - in-flight with HTTPS && SSH
- **Cross Account**
  - Use a IAM Role in your account and STS (AssumeRole) for the cross-account

## CodePipeline

- Visual tool to orchestrate CICD, can integrate with any tool at any stage
- Manual approval can be defined at any stage too
- A pipeline is a set of stages, sequential or parallel actions that call the next one.
- _resulting artifacts are stored into S3 buckets and go into the next phase, so the pipeline doesn't go back into CodeCommit_

**Troubleshooting**:

- Cloudwatch events (EventBridge) for _failed_ and _cancelled_ stages
- Console for info on failed stages
- IAM service role must have enough permissions
- Cloudtrail for audits on the API calls

**Triggers**

- Events (defautl && recommended)
- webhooks ( send a request to trigger )
- polling (pulled instead of pushed)

#### Manual Approval Stage

- _Set CodePipeline to trigger a SNS message to an IAM user, configure manual approval_
- the user must have _getPipeline_ and _putApprovalResult_ permissions in IAM

## CodeBuild

![clipboard.png](inkdrop://file:luZ0rLHjz)

- **Build instructions**: _buildspec.yml_ or manually insert build instructions in the console
- Output logs into S3 or CloudWatch logs
- Cloudwatch
  - metrics for statistics
  - events for failed builds and notifications
  - alarms for thresholds for failures ( number of errors, warnings, etc )
- S3 for cache of reusable pieces in the build
- Fully managed, scales on it's own
- leverages docker for reproductible builds, use your own images or get prepackaged ones
- charged per minute on the builds

**Security**

- KMS ecrypts build artifacts
- IAM for permissions
- VPC for network security
- Cloudtrail for audits on the API calls

### buildspec.yml - IN THE ROOT DIRECTORY OF THE PROJECT

you can use scripts but it's strongly advised to go with the **buildspec.yml**

- Env:
  - variables: plaintext
  - parameter-store: from SSM
  - secrets-mananger: from AWS Secrets manager
- Phases:
  - install (install dependencies)
  - pre_build (finish setting up)
  - build
  - post_build: (zip the output)
- Artifacts:
  - what must be uploaded to S3 (encyrpted with KMS)
- Cache:
  - what to keep (in s3 for future build steps)
  - usually the dependencies

The main backend languages have pre-configured build environments for building
_Caching in S3 is optional but recommended_

#### CodeBuild Locally

- Install Docker
- Install Codebuild Agent

#### CodeBuild Inside VPC

- By default launched outside the VPC
  - you can specify a VPC config:
    - VPC id
    - subnet id
    - SGs Id
  - Your build can then acess resources inside the VPC
  - Good for integration testing

#### CodeBuild and CloudFormation integration

- CloudFormation is used to deploy complex infrastructure using an API
  - Create_Update mode: can create a whole test stack as part of the pipeline
  - Run the test suite on the test stack
  - Delete_Only mode: to clean up the test environment via CF
  - Deploy to prod via CF

### CodeDeploy

- _Going from V1 to V2_
- You can create a deploy group with:

  - EC2 instances
  - On-premise

- but everything must be running the **CodeDeploy** agent.
- Agents continuously pool the trigger for deploy targets.
- The app + appsec.yaml file is pulled from S3/github.
- EC2 instances run the deployment instructions in the appsec.yml

Components:

- _Application_: a unique name functions as a container (revision, deployment configuration, ...)
- _ComputePlatform_ EC2/On-Premises, AWSLambda, orAmazonECS
- _Deployment Configuration_: a set of deployment rules for success/failure
  - _EC2/On-premises_: specify the minimum number of healthy instances for the deployment
  - _AWS Lambda or Amazon ECS_: specify how traffic is routed to your updated versions
- **Deployment Group**: group of tagged EC2 instances (allows to deploy gradually, or dev, test, prod...)
- _Deployment Type_: method used to deploy the application to a Deployment Group
  - _In-place Deployment_: supports EC2/On-Premises
  - _Blue/Green Deployment_: suppor ts EC2 instances only, AWS Lambda, and Amazon ECS
- _IAM Instance Profile_: give EC2 instances the permissions to access both _S3 / GitHub_
- _Application Revision_: application code + appspec.yml file
- _Service Role_: an IAM Role for CodeDeploy to perform operations on EC2 instances, ASGs, ELBs...
- _Target Revision_: the most recent revision that you want to deploy to a Deployment Group

#### Appspec.yml structure - PUT IT IN THE ROOT DIRECTORY OF THE APP

- _Files_: just source/destination to copy from the source to the deploy servers
  - your CodeDeploy instances must have S3 access.
- _Hooks_: Instructions to deploy new versions that execute in order:
  - ApplicationStop
  - DownloadBundle
  - BeforeInstall
  - Install
  - AfterInstall
  - ApplicationStart
  - ValidateService (important!)
    - BeforeAllowTraffic
    - AllowTraffic
    - AfterAllowTraffic

#### Deploy Configurations

- One at a time: if one instance fails, deployment stops
- Half at a time
- All at once: causes downtime
- Blue Green
  - must have an ASG
  - it launches another ASG
  - it swaps the ALB to the new ASG
- Custom: set the min. healthy host%

### Deploying to an ASG

- **In-Place**
  - Updates existing EC2 instances
  - newly created instances by the ASG also get updated
- **Blue-green**
  - New ASG is created (same settings)
  - choose how long to keep the OLD onde
  - Must be using an ALB

#### Deployment groups

- set of _tagged EC2 instances_
- mix of ASG && tags to build deployment segments
- _DEPLOYMENT_GROUP_NAME_ environment variables for customization scripts

### CodeDeploy Hands On

- You must create a DeployGroup
- Upload the data into a S3 bucket
- review permissions in the DeployGroup

**Deployment to EC2** - Define how to deploy: appsec.yml + deploy strategy
**Deploy to an ASG**

- In-place
  - Updates existing EC2 instances
  - The new EC2s created by the ASG will also get the same automated deployment
- Blue/Green
  - new ASG is created (settings are copied) and switch
  - choose how long to keep the old ASG

#### Failures:

- **EC2 instances stay in Failed state**
- New deployments first to the _Failed_ instances
- _To rollback, redeploy the old one or enable auto rollback for failures_

#### Redeploys and Rollbacks options

- **Rollback:** redeploy a previous version
  - **Automatically**: when a CloudWatch Alarm threshold is met || deployment fails
  - Manually
- _Disable rollbacks_ can be set for a deployment

If a rollback happens CodeDeploy redeploys the last good revision as a _new deployment_ (not a restored version)

#### CodeDeploy Troubleshooting

- "InvalidSignatureException - signature expired"
  - CodeDeploy needs acurate time references, if the date in the EC2 instance is not correct the signature date of the deployment requests may get rejected
- Check the logs to undestand the deployment issues ( `/opt/codedeploy-agent/deployment-root/deployment`)

### AWS CodeStar

Integrated solution grouping:

- Git Repository, CodeBuild, CodeDeploym CloudFormation, CodePipeline, Cloudwatch, Issue tracking with JIRA, and IDE
- creates projects that are CICD-ready for different deployment targets
- Supports the most common backend runtimes and html5
- Free, pay for the underlying usage
- Limited Customizatiohn
- each bundle tells you how the underlying infra is organized

## AWS CodeArtifact

_Is it just Jfrog?_
An artifact management system. Integrates with maven, gradle, npm, yarn, twine pip etc... Packages can live inside your vpc and be accessible from inside after approval. Whenever you have security or compliance concers

An event is emmited whenever a package version is created, modified or deleted. So updates can trigger rebuilds automatically

#### CodeArtifact - Resource policies

- Authorize accounts to access the artifacts
- A principal can read all the packages or none of them

#### CodeArtifact - Upstream Repos

- can be the public repositories in which the packages are published, so your local package managements can fetch there on "cache miss"
- Up to 10 Upstream Repositories in one single connection, one external connection per repo
- Usually you configure one repo inside with an external connection to the public and then let the child repos use it as cache
- Artifact Retention:
  - A requested package is always retained and available downstream
  - the retained package is isolated from the upstream
  - Intermediate repos don't keep the package, just the one with the external and the **MOST-DOWNSTREAM**

#### CodeArtifact - Domains

- across multiple repos and multiple accounts, it's a group for deduplication ( a mono repo for monorepos? )
- Easy to share and encrypt with the same KMS key
- Domain Resource Policy
  - Who has the right to create public connections, etc..

### CodeGuru - ML-powered code reviews and performance recommendations

_It's Static Analysis!_

- CodeGuru Reviewer: automated code reviews
- CodeGuru Profiler: recommendations for performance in runtime
  - heap summary
  - anomaly detection
  - decrease compute costs
- Supports Java and Python for now
- integrates with github etc

#### CodeGuru - Agent Configuration

- MaxStackDepth: how deep the Agent is going in the stack
- MemoryUsageLimit%
- MinimumTimeForReportingMilliseconds
- ReportingIntervallMilisseconds
- SamplingIntervalMilisseconds: smaller is faster smapling

### Cloud9 Cloud IDE

- all of that telemetry
- Code Editor, debugger, terminal in browser
- Work from anywhere with all the tools in the cloud
- Share development envirnments.
- Do pair programming
- Integrates with SAM & Lambda to build serverless
- Runs inside an EC2 Instance
