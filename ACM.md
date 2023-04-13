### Integrated With:

- ELB
- CloudFront
- API Gateway
- CloudFormation - only ACM issued certs
- Elastic Beanstalk
- Nitro Enclaves

_You can also use with_

- EC2 instances
- on-premise servers

### Types of certs

- ACM issued public
- ACM issued private
- Imported:
  - AMC can't renew but helps manage renewal - you can use CloudWatch metrics to monitor expiration dates and import the new ones on expiration

### ACM issued certs

- _Works with WILDCARD domains_
- Valid for 13 months
- RSA-2048 and SHA-256
- you can request to revoke
- Domain Validated (DV) only

### Renewal

- **ACM issued public:**
  - ACM manages renewal and deployment
- **ACM issued private CA:**
  - You can chose to delegate management to ACM
    - ACM renews and deploys to integrated services
    - _Sends CloudWatch notifications when the renewal is completed_ you can write code to download and deploy renewed certs on notification
- **Imported:** AMC can't renew but helps manage renewal - you can use CloudWatch metrics to monitor expiration dates and import the new ones on expiration

- ACM begins the renewal process <= 60 days prior to expiration (13 months, so month 11)
- ACM may renew, rekey and deploy a certificate without notice
- Connections are not dropped on deployment

### Validating Domain Ownership

_when you request a public certificate ACM has to validate that you own all of the domains you want the certificate for._

can't convert one type of validation to the orhter

- **DNS Validation (Recommended)**: by adding a CNAME record to your DNS config
- **EMAIL Validation**: send emails to the registered domain owner and owner then authorizes

### Private Key Protection

- A key pair is created for each certificate provided by ACM
- ACM does not copy certificates across AWS regions

### Billing

- public and private certws are free for use in integrated services
- AWS private CA is pay as you go
