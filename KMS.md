## AWS KMS (Key Management Service)

- "Encryption" = KMS
- Manages keys for us
- fully integrated with IAM
- Easy to controll acess to the data
- Auditable with CloudTrail
- Integrated into most services (EBS, S3, RDS, SSM
- NEVER STORE SECRETS IN PLAIN TEXT
  - available through api calls
  - encrypted secrets can be stored in code

### KMS Key Types ( customer master key )

- **Symetric (AES-256)**
  - single encryption key to encryp and decrypt
  - integrated services use Symetric CMKs
  - you never get access to the key unencrypted ( KMS API calls encapsulate Use)
- **Asymetric (RSA & ECC key pairs)**

  - public for Encryption
  - Private for decryption
  - public key is downloaded
  - private can't be viewed unencripted
  - use them oputside AWS for clients that cant call the KMS API

- AWS Owned (free): SSE-S3, SSE-SQS, SSE-DDB (default key)
- AWS Managed (free): aws/service-name
- Customer managed created in KMS: 1$ / month
- Customer managed imported ( must be symmetric key): $1 / month
  - pay for api calls (3 cents / 10k calls)

**Automatic Key rotation**

- AWS-managed: every year
- Customer-managed: Must be enabled, every year
- Imported KMS key: Only manual rotation using alias

### KMS Key Policies

- Control access to KMS keys, “similar” to S3 bucket policies
- Difference: you cannot control access without them

- Default KMS Key Policy:
  - Created if you don’t provide a specific KMS Key Policy
  - Complete access to the key to the root user = entire AWS account
- Custom KMS Key Policy:
  - Define users, roles that can access the KMS key
  - Define who can administer the key
  - Useful for cross-account access of your KMS key

#### Copying Snapshots across accounts

1. Create a Snapshot, encr ypted with your own KMS Key (Customer Managed Key)
2. Attach a KMS Key Policy to authorize cross-account access
3. Share the encr ypted snapshot
4. (in target) Create a copy of the Snapshot, encrypt it with a CMK in your account
5. Create a volume from the snapshot

### How does it work?

![clipboard.png](inkdrop://file:-N376xGOS)

- Encrypt has a limit of 4kb, > goes into Envolepe Encryption through `GenerateDataKey`

### Envelope Encryption with GenerateDataKey

![clipboard.png](inkdrop://file:saNTnSvbw)

![clipboard.png](inkdrop://file:HjMAQHHBz)

- The AWS Encryption SDK implemented Envelope Encryption for us - The Encryption SDK also exists as a CLI tool we can install
- Implementations for Java, Python, C, JavaScript
- Feature - Data Key Caching:
  - re-use data keys instead of creating new ones for each encryption
  - Helps with reducing the number of calls to KMS with a security trade-off
  - Use LocalCryptoMaterialsCache (max age, max bytes, max number of messages)

## KMS Symmetric – API Summary

- Encrypt: encrypt up to 4 KB of data through KMS
- **GenerateDataKey**: generates a unique symmetric data key (DEK)
  - returns a plaintext copy of the data key
  - AND a copy that is encrypted under the CMK that you specify
- **GenerateDataKeyWithoutPlaintext**:
  - Generate a DEK to use at some point (not immediately)
  - DEK that is encrypted under the CMK that you specify (must use Decrypt later)
- **Decrypt**: decrypt up to 4 KB of data (including Data Encryption Keys)
- **GenerateRandom**: Returns a random byte string

### KMS Request Quotas

- **ACCOUNT WIDE QUOTAS**
- When you exceed a request quota, you get a **ThrottlingException**:
- To respond, use exponential backoff (backoff and retry)
- For cryptographic operations, they share a quota
- This includes requests made by AWS on your behalf (ex: SSE-KMS)
- For **GenerateDataKey**, consider using DEK caching from the Encryption SDK - You can request a Request Quotas increase through API or AWS support

### Lambda best practices.

- Encrypt your secrets with a KMS key, call decode outside of the handler in the lambda to retreive the value at runtime

### S3 **Bucket Key** for SSE-KMS Encryption

- Bucket key is a new setting to decrease
  - number of api calls made to kms from S3 by 99%
- Leverages data keys
  - A **S3 bucket key** is generated
  - that key is used to encrypt KMS objects with new data keys

### S3 encrypted bucket:

To access you must have the KMS:Decrypt permission on IAM

**aws:SecureTransport**: forces SSL requests to your S3 buckets

### Key Policy examples

- by default: anyone in the account with the IAM can access KMS
- but you can set permissions on all KMS APIs
- Principal Options
  - Account and Root user
  - IAM roles
  - IAM role sessions ( assumed roles through cognito identity )
  - IAM Users
  - Federated User Sessions
  - Aws Services
  - All principals ( _ or "AWS":"_" )

### Cloud HSM

- AWS managed **Hardware** for encryption
- YOU have to manage your own keys
- supports symmetric and asymmetric encryption
- no free tier, must use **CloudHSM** client
- Good for SSE-C encryption
- Iam permissions: CRUD on HSM
- can have High Availability - multiple AZs
- KMS is integrated with CloudHSM through custom key stores

## SSM Parameter Store

- Secure storage for your secrets
- Seamless encryption with KMS
- Serverless, scalable, durable
- version tracking on secrets
- security through IAM
- Notifications with EventBridge
- Integrated with CloudFormation

#### Parameter Store Hierarchy

- works like a folder structure, you specify a path
- security can work on the folder through IAM

| Standart    | Advanced                             |
| ----------- | ------------------------------------ |
| 10k         | 100k                                 |
| 4kb         | 8kb                                  |
| no policies | yes policies                         |
| free        | charges 0.05 per parameter per month |

### Parameter Policies

- Allows for TTL for deleting sensitive data
- multiple policies at a time

## AWS Secrets Manager (competitor)

- Oriented for secrets
- can **force rotation every X days**
- Automate secret generation on rotation ( uses lambda)
- Integrates with RDS ( mostly made for that purpose )
- uses KMS

| SSM parameter Store                             | Secrets Manager                |
| ----------------------------------------------- | ------------------------------ |
| Simple API                                      |                                |
| No rotation ( you can do it yourself )          | Secret Rotation                |
| `$`                                             | `$$$`                          |
| Integrates with CloudFormation                  | Integrates with CloudFormation |
| KMS is mandatory                                | KMS is optional                |
| Can pull a secrets manager secret using the API |                                |

#### CloudWatch logs Encryption

- You can encrypt with KMS
- Happens at the **Log Group** level
- API calls
  - **Associate-kms-key**
  - **create-log-group**

### Codebuild security

- is outside of your VPC by defautl, can run inside via config
- Secrets in codebuild:
  - **Don't store them in environment variables in plain text**
  - referece parameter store parameters through the environment variables

### AWS Nitro Enclaves

- Process higly sensitive data in isolated compute environments
- Fully isolated VMs
  - not a container, no persistent storage, no SSH, no external Networking
- **Cryptographic Atestation**: only signed code can run in the enclaves
- only enclaves can access sensistive data ( integration with KMS )

![clipboard.png](inkdrop://file:jQQaZrv-v)
