# Amazon Cognito

- Give users an identity to interact with our web or mobile application
- Cognito User Pools:
  - Sign in functionality for app users
  - Integrate with API Gateway & Application Load Balancer
- Cognito Identity Pools (Federated Identity):
  - Provide AWS credentials to users so they can access AWS resources directly
  - Integrate with Cognito User Pools as an identity provider
- Cognito vs IAM: “hundreds of users”, ”mobile users”,“authenticate with SAML”

### Cognito user pools (CUP)

- **Create a serverless database of user for your web & mobile apps **
- Simple login: Username (or email) / password combination
- Password reset
- Email & Phone Number Verification
- Multi-factor authentication (MFA)
- _Federated Identities: users from Facebook, Google, SAML..._
- Feature: block users if their credentials are compromised elsewhere
- Login sends back a **JSON Web Token (JWT)**

### Integrations

![clipboard.png](inkdrop://file:YnGHt7XMA)

### Congnito Lambda triggers

![clipboard.png](inkdrop://file:sBR4iawiV)

### Hosted Authentication UI:

- You can reuse the Cognito Ui and just customize with a logo and CSS
- You can use a custom domain, but the certificate must be an **ACM in us-east-1**
- custom domain must be defined in the App Integration section

### CUP - Adaptative Authentication

- Block sign-ins or require MFA if the login appears suspicious
- Cognito examines each sign-in attempt and generates a risk score (low, medium, high) for _how likely the sign-in request is to be from a malicious attacker_
- Users are prompted for a second MFA only when risk is detected
- Risk score is based on different factors such as if the user has used the same device, location, or IP address
- Checks for compromised credentials, account takeover protection, and phone and email verification
- Integration with **CloudWatch** Logs (sign-in attempts, risk score, failed challenges...)

### JWT Json web tokens

- CUP issues JWT tokens (Base64 encoded):
  - Header
  - Payload
  - Signature
- The signature must be verified to ensure the JWT can be trusted
- Libraries can help you verify the validity of JWT tokens issued by Cognito User Pools
- The Payload will contain the user information (sub UUID, given_name, email, phone_number, attributes...)
- From the sub UUID, you can retrieve all users details from Cognito / OIDC

### Auth in the ALB

- Your Application Load Balancer can securely authenticate users
  - Offload the work of authenticating users to your load balancer
  - Your applications can focus on their business logic
- Authenticate users through:
  - Identity Provider (IdP): OpenID Connect (OIDC) compliant
  - Cognito User Pools:
    - SocialIdPs,suchasAmazon,Facebook,orGoogle
    - CorporateidentitiesusingSAML,LDAP,orMicrosoftAD
- Must use an **HTTPS** listener to set _authenticate-oidc_ & _authenticate-cognito_ rules
- **OnUnauthenticatedRequest** – authenticate (default), deny, allow

**How to authenticate**

- Create Cognito User Pool, Client and Domain
- Make sure an ID token is returned
- Add the social or Corporate IdP if needed
- Several URL redirections are necessary (specific to CUP)
- Allow your Cognito User Pool Domain on your IdP app's callback URL. For example:
  - `https://domain- prefix.auth.region.amazoncognito.com/saml2/` idpresponse
  - `https://user-pool-domain/oauth2/idpresponse`

### Application Load Balancer – OIDC Auth.

- Configure a Client ID & Client Secret
- Allow redirect from OIDC to your Application Load Balancer DNS name (AWS provided) and CNAME (DNS Alias of your app)
  - `https://DNS/oauth2/idpresponse`
  - `https://CNAME/oauth2/idpresponse`

![clipboard.png](inkdrop://file:qzeeuIu6M)

### Cognito Identity Pools (Federated Identities)

- Get identities for “users” so they obtain temporary AWS credentials ( attatch IAM policies to users )
- Your identity pool (e.g identity source) can include:
  - Public Providers (Login with Amazon, Facebook, Google, Apple)
  - Users in an Amazon Cognito user pool
  - OpenID Connect Providers & SAML Identity Providers
  - Developer Authenticated Identities (custom login server)
  - Cognito Identity Pools allow for unauthenticated (guest) access
- Users can then access AWS services directly or through API Gateway
  - The IAM policies applied to the credentials are defined in Cognito
  - They can be customized based on the user_id for fine grained control

### Identity Pools how it works and how it integrates User Pools

**Identity Pools**
![clipboard.png](inkdrop://file:1l2I7Mqlv)

**CUP**
![clipboard.png](inkdrop://file:QqHWD5m0M)

### User Pools vs Identity Pools

- **Cognito User Pools (for authentication = identity verification) **
  - Database of users for your web and mobile application
  - Allows to federate logins through Public Social, OIDC, SAML...
  - Can customize the hosted UI for authentication (including the logo)
  - Has triggers with AWS Lambda during the authentication flow
  - Adapt the sign-in experience to different risk levels (MFA, adaptive authentication, etc...)
- **Cognito Identity Pools (for authorization = access control)**
  - Obtain AWS credentials for your users
  - Users can login through Public Social, OIDC, SAML & Cognito User Pools
  - Users can be unauthenticated (guests)
  - Users are mapped to IAM roles & policies, can leverage policy variables
- **CUP + CIP = authentication + authorization**
