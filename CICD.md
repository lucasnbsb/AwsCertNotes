# CI/CD on AWS - CodeBuild and CodeDeploy

- Automating everything for security and speed
- CodeCommit -> CodePipeline -> CodeBuild -> CodeDeploy -> CodeStart -> CodeArtifact -> CodeGuru
- Deploy often, keep results visible
- push code (repo) -> build and test (build server) -> deploy (deployment server)

### CodeCommit

- github by aws basically
- better for security, very private
- SSH keys in the IAM or HTTP for password
- Authorization by IAM
- Encryption by KMS
- Encryption in-flight through https or ssh
- for cross account aceess use STS
- minimal UI

### CodePipeline

- Visual workflow tool to orchestrate CICD, use any toll for each stage
  - Source
  - Build
  - Test
  - Deploy

A pipeline is a set of stages, sequential or parallel actions that call the next one.
_resulting artifacts are stored into S3 buckets and go into the next phase, so the pipeline doesn't go back into CodeCommit_

**Troubleshooting**: through CloudWatch and EventBridge, events for failed pipelines or cancelled stages

- If a stage fails your pipeline stops, info in the console
- If the pipeline can't perform an ection check the IAM Service Role is attached in the policy
- Cloudtrail can be used to Audit API calls

- Cloudwatch events is the recommended way to trigger the pipeline
- you can visualy add action groups sequentially or in parallel

#### Manual Approval Stage

- Set codepipeline to trigger a SNS message to an IAM user configured for manual action. Usualy done before the deploy stage
- the user must have getPipeline and putApprovalResult permissions in IAM

### CodeBuild

- buildspec.yml or manually insert build instructions in the console
- Output logs into S3 or CloudWatch logs
- Cloudwatch
  - metrics, events, alarms

![clipboard.png](inkdrop://file:luZ0rLHjz)

**The buildspec.yml file:**
Must be put in the root directory of the project being built, you can use scripts but it's strongly advised to go with the buildspec.yml

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

- Install Docer
- Install Codebuild Agent

#### CodeBuild Inside VPC

- By default launched outside the VPC
  - VPC config:
    - id
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

but everything must be running the **CodeDeploy** agent. Agents continuously pool the trigger for
deploy targets. The app + appsec.yaml file is pulled from S3. EC2 instances run the deployment instructions
in the appsec.yml

Components:

- _Application_ – a unique name functions as a container (revision, deployment configuration, ...)
- _ComputePlatform_ –EC2/On-Premises,AWSLambda,orAmazonECS
- _Deployment Configuration_ – a set of deployment rules for success/failure
- _EC2/On-premises_ – specify the minimum number of healthy instances for the deployment - AWS Lambda or Amazon ECS – specify how traffic is routed to your updated versions
- _Deployment Group_ - group of tagged EC2 instances (allows to deploy gradually, or dev, test, prod...)
- _Deployment Type_ – method used to deploy the application to a Deployment Group
- _In-place Deployment_ – supports EC2/On-Premises
- _Blue/Green Deployment_ – suppor ts EC2 instances only, AWS Lambda, and Amazon ECS
- _IAM Instance Profile_ – give EC2 instances the permissions to access both S3 / GitHub
- _Application Revision_ – application code + appspec.yml file
- _Service Role_ – an IAM Role for CodeDeploy to perform operations on EC2 instances, ASGs, ELBs... • Target Revision – the most recent revision that you want to deploy to a Deployment Group

#### CodeDeploy - Appspec.yml

- Files: just source/destination to copy from the source to the deploy servers
  - your CodeDeploy instances must have S3 access.
- Hooks: Instructions to deploy new versions that execute in order:
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

- One at a time
- Half at a time
- All at once
- Blue Green
  - must have an ASG
  - it launches another ASG
  - it swaps the ALB to the new ASG
- Custom: set the min. healthy host%

#### Failures:

- EC2 instances stay in _Failed_ state

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

#### Redeploys and Rollbacks options

- Automatically: when a CloudWatch Alarm threshold is met
- Manually
- Disable rollbacks

If a rollback happens CodeDeploy redeploys the last good revision as a _new deployment_ (not a restore)

#### CodeDeploy Troubleshooting

- "InvalidSignatureException - signature expired"
- CodeDeploy needs acurate time references
- if the date in the EC2 instance is not correct the signature date of the deployment requests may get rejected
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
