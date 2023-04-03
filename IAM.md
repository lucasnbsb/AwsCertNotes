# IAM

### _Policies_ [are-assigned-to] (_Users_ and _Groups_)

Use the principle of minimal permission per user

### Taxonomy

- **Group** (has policies)
  - **Users** (inherits permissions through policies) ( can have it's own _inline-policies_ )
    - Has aliases
- **Roles**
  - **Services**
  -

#### Policies ( how the json is formed )

- Version Number
- Id
- Statements:
  - Sid
  - Effect ( allow | deny )
  - principal (account | user | role)
  - action ( what is being allowed)
  - resource
  - condition

#### Security Practices

- Password Policy
  - password formation
  - change period
- MFA - Multi Factor Authentication
  - Use at LEAST on the root account, preferably on every user
  - can be:
    - virtual -> authenticator app
    - physical
      - usb drive
      - specific hardware

**Best Practices**

- One physical user = One Aws User
- Use roles for service
- Use Groups for users
- Never share keys
- Audit users

#### Security Tools

- _IAM Credential Reports_ (Account Level)
  - Just a spreadsheet
  - Reports on who can use what
- _IAM Access Advisor_ (User Level)
  - helps reduce the permissions of a user to the least-permission-necessary
  - shows what kind of permissions where actually used and when by each user

### How to access AWS:

- Management Console
- Comand Line Interface (CLI)
- Software Developer Kit (SDK)
  - each language has it's own

For the **CLI && SDK**: you need an _Access Key_

### Roles

Are like groups for services: they provide policies for aws services to access other services

# Advanced IAM concepts

#### Authorization Module and evaluation of policies

![clipboard.png](inkdrop://file:Yn8vfuEWt)

_Explicit deny wins over explicit allow_

#### IAM Policies & S3 bucket policies

- IAM Policies are attached to users, roles, groups
- S3 Bucket Policies are attached to buckets
- When evaluating if an IAM Principal can perform an operation X on a bucket, the **UNION** of its assigned IAM Policies and S3 Bucket Policies will be evaluated.

#### Dynamic Policies

- How do you assign each user a `/home/<user>` folder in an S3 bucket?
- Create one dynamic policy with IAM
- Leverage the special policy variable `${aws:username}`

#### Inline vs Managed Policies

- AWS Managed Policy
  - Maintained by AWS
  - Good for power users and administrators
  - Updated in case of new services / new APIs
- Customer Managed Policy
  - **Best Practice**, re-usable, can be applied to many principals
  - Version Controlled + rollback, central change management
- Inline
  - Strict one-to-one relationship between policy and principal
  - Policy is deleted if you delete the IAM principal

### Granting a User Permissions to Pass a Role to an AWS Service

- To configure many AWS services, you must pass an IAM role to the service (this happens only once during setup)
- The service will later assume the role and perform actions
- Example of passing a role:
  - To an EC2 instance
  - To a Lambda function
  - To an ECS task
  - To CodePipeline to allow it to invoke other services
- For this, you need the IAM permission **iam:PassRole**
- It often comes with **iam:GetRole** to view the role being passed

**Can a role be passed to any service?**

- No: Roles can only be passed to what their trust allows
- A trust policy for the role that allows the service to assume the role

### Directory Services

**What is Microsoft Active Directory (AD)?**

- Found on any Windows Server with AD Domain Services
- Database of objects: User Accounts, Computers, Printers, File Shares, Security Groups
- Centralized security management, create account, assign permissions
- Objects are organized in trees
- A group of trees is a forest

## AWS Directory Services

- AWS Managed Microsoft AD

  - Create your own AD in AWS, manage users
  - Establish “trust” connections with your on- premise AD

- AD Connector

  - Directory Gateway (proxy) to redirect to on-proxy premise AD, supports MFA
  - Users are managed on the on-premise AD

- Simple AD
  - AD-compatible managed directory on AWS
  - Cannot be joined with on-premise AD
