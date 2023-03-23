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
