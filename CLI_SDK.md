# CLI | SDK | IAM Roles & Policies

### `--dry-run` to check your permissions

Some stuff is expensive to run, and you may want to check first

- Api calls can return some very long strings as error messages
  - `aws sts decode-authorization-message --encoded-message <message>`
  - maybe alias this whole commandt to something shorter
    - You have to authorize the STS to run this command on the policy

### EC2 Instance Metadata

- instances can find stuff out about themselves
- `curl http://169.254.169.254/latest/meta-data`
  - returns:
    - IAM info
    - instance-id
    - ip
    - metrics
    - etc
- **You don't need permissions to run this**
- **You CANNOT retreive policies this way, just role names**
- IAM roles basically query this endpoint to get short lived credentials

### CLI

#### profiles

`aws configure --profile <name>`

- run configure normally
  - now your `~/.aws/credentials` file has 2 profiles
- call it like this:
  - `aws s3 ls --profile <name>`
  - otherwise goes to the default profile

#### MFA

`aws sts get-session-token --serial-number <arn> --token-code <code from the mfa app>`

- Add key and id to the credentials file
- Add the session token to your credentials file as `aws_session_token`

### SDK

- you need to know when to use the SDK
- Performing actions from inside the code
- Official SDKs:
  - Java
  - .Net
  - Node.js
  - PHP
  - Python (boto3, used by the CLI)
  - Go
  - Ruby
  - C++
- us-east1 is the default region if you don't especify in the SDK

### AWS Limits (quotas)

API Rate Limit

- DescribeInstances 100 cps
- GetObject S3: 5500 cps per prefix
- for intermittent Errors: Implement Exponential Backoff
- for consistent Errors: request a rate limit increase

Service Limites ( Service Quotas )

- On demand standart instances: 1152 vCPU
- Open a ticket to request a limit increase
- Use The Service Quotas API to increase quotas

#### Implementing Exponential Backof

- on THrottlingException
  - the SDK api calls already has a retry mechanism
  - Some specific cases or API as is you must _implement yourself_ - Only retry on 5XX server errors and throttling
    _Keep doubling the time to retry if you still get an error_

#### Credentials Provider Chain

The CLI will look for credentials in the following order

- CLI options ( --region, --output and --profile)
- Environment Variables
  - AWS_ACCESS_KEY_ID
  - AWS_SECRET_ACESS_KEY
  - AWS_SESSION_TOKEN
- CLI credentials file
- CLI configuration file
  - ~/.aws/config
- Container credentials (for ECS tasks)
- Instance profile credentials (for EC2 instance profiles)

#### Best Practices

NEVER STORE AWS CREDENTIALS IN YOUR CODE

- Inside AWS use IAM Roles
  - EC2 Instance Roles
  - ECS Roles
  - Lambda Roles
- Outside AWS use environment variables / named profiles

#### Signing AWS API requests

- signing your http api requests with your credentials (the sdk and cli already does that for you)
- SigV4
  - put the token in: `Authorization` Header || Query string X-Amz-Signature
