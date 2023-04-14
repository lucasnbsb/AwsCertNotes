# API Gateway

_the ALB killer?_

- Proxies the requests into our lambda functions
- AWS Lambda + API Gateway: No infrastructure to manage
- Support for the WebSocket Protocol
- Handle API **versioning** (v1, v2...)
- Handle **different environments** (dev, test, prod...)
- Handle security (Authentication and Authorization)
- Create API keys, handle request throttling
- Swagger / Open API import to quickly define APIs
- Transform and validate requests and responses
- Generate SDK and API specifications
- Cache API responses

### Integrations

- **Lambda Function**
  - Invoke Lambda function
  - Easy way to expose REST API backed by AWS Lambda
- **HTTP**
  - Expose HTTP endpoints in the backend
  - Example: internal HTTP API on premise, Application Load Balancer...
  - Why? Add rate limiting, caching, user authentications, API keys, etc...
- **AWS Service**
  - Expose any AWS API through the API Gateway
  - Example: start an AWS Step Function workflow, post a message to SQS
  - Why? Add authentication, deploy publicly, rate control...

![clipboard.png](inkdrop://file:iGupE99Jc)

### Endpoint types

- **Edge-Optimized** (default): For global clients
  - Requests are routed through the CloudFront Edge locations (improves latency)
  - The API Gateway still lives in only one region
- **Regional**:
  - For clients within the same region
  - Could manually combine with CloudFront (more control over the caching strategies and the distribution)
- **Private**:
  - Can only be accessed from your VPC using an **interface VPC endpoint** (ENI)
  - Use a **resource policy** to define access

### API Gateway – Security

- **User Authentication through**
  - _IAM Roles_ (useful for internal applications)
  - _Cognito_ (identity for external users – example mobile users)
  - _Custom Authorizer_ (your own logic) - Now Called _Lambda Authorizer_
- **Custom Domain Name**
  - HTTPS security through integration with AWS Certificate Manager (ACM)
  - If using Edge-Optimized endpoint, then the certificate _must be in us-east-1_
  - If using Regional endpoint, the certificate must be in the _API Gateway region_
  - Must setup _CNAME or A-alias_ record in Route 53

### API Gateway – Deployment Stages

- making changes don't mean they're effective
- you need to make _Deployments_ for them to apply
- Changes are deployed to “Stages” (as many as you want)
  - Use the naming you like for stages (dev, test, prod)
- Each stage has its own configuration parameters
- Stages can be rolled back as a history of deployments is kept

#### API Gateway – Stage Variables

- Stage variables are like environment variables for API Gateway
- Use them to change _often changing_ configuration values
- They can be used in:
  - Lambda function ARN
  - HTTP Endpoint
  - Parameter mapping templates
- Use cases:
  - Configure HTTP endpoints your stages talk to (dev, test, prod...)
  - Pass configuration parameters to AWS Lambda through mapping templates
- Stage variables are passed to the ”context” object in AWS Lambda - Format: ${stageVariables.variableName}

### API Gateway – Canary Deployment

- Choose the % of traffic the canary channel receives
- Metrics & Logs are separate (for better monitoring)
- Possibility to override stage variables for canary
- This is blue / green deployment with AWS Lambda & API Gateway

### API Gateway - IntegrationTypes

- **MOCK**
  - returns a fixed response
- **HTTP / AWS (Lambda & AWS Services)**
  - you must configure both the _integration request and integration response_
  - Setup data mapping using _mapping templates_ for the request & response
- **AWS_PROXY (Lambda proxy)**
  - incoming request is piped to Lambda
  - The function is responsible for the logic of request / response
  - No mapping template, headers and query string parameters are passed as arguments
- **HTTP_PROXY**
  - No mapping template
  - The HTTP request is passed to the backend
  - The HTTP response from the backend is forwarded by API Gateway
  - Possibility to add HTTP Headers if need be (ex:API key)

### Mapping Templates (AWS & HTTP Integration)

- Mapping templates can be used to modify request / responses
- Rename / Modify query string parameters
- Modify body content
- Add headers
- Uses _Velocity Template Language_ (VTL): for loop, if etc...
- Filter output results (remove unnecessary data)
- Content-Type can be set to **application/json or application/xml**

### API Gateway - Open API spec

- Common way of defining REST APIs, using API definition as code
- Import existing OpenAPI 3.0 spec to API Gateway
  - Method
  - Method Request
  - Integration Request
  - Method Response
  - - AWS extensions for API gateway and setup every single option
- Can export current API as OpenAPI spec
- OpenAPI specs can be written inYAML or JSON
- Using OpenAPI we can generate SDK for our applications

### REST API – Request Validation

- You can configure API Gateway to perform basic validation of an API request before proceeding with the integration request
- When the validation fails, API Gateway immediately fails the request
  - Returns a 400-error response to the caller
- This reduces unnecessary calls to the backend
- Checks:
  - The required request parameters in the URI, query string, and headers of an incoming request are included and non-blank
  - The applicable request payload adheres to the configured JSON Schema request model of the method
- Setup request validation by importing OpenAPI definitions file and defining the validators in the x-amazon-apigateway-request-validators attribute

### Caching API responses

- Caching reduces the number of calls made to the backend
- Default TTL (time to live) is 300 seconds (min: 0s, max: 3600s)
- Caches are defined _per stage_
- Possible to override cache settings _per method_
- Cache encryption option
- Cache capacity between 0.5GB to 237GB
- _Cache is expensive, makes sense in production, may not make sense in dev / test_

### API Gateway Cache Invalidation

- Able to flush the entire cache (invalidate it) immediately
- Clients can invalidate the cache with `header: Cache- Control: max-age=0` (with proper IAM authorization)
- If you don't impose an InvalidateCache policy (or choose the Require authorization check box in the console), any client can invalidate the API cache

### API Gateway – Usage Plans & API Keys (Saas)

- If you want to make an API available as an offering ($) to your customers
- **Usage Plan:**
  - who can access one or more deployed API stages and methods
  - how much and how fast they can access them
  - uses API keys to identify API clients and meter access
  - configure throttling limits and quota limits that are enforced on individual client
- **API Keys:**
  - alphanumeric string values to distribute to your customers
  - Ex:WBjHxNtoAb4WPKBC7cGm64CBibIb24b4jt8jJHo9
  - Can use with usage plans to control access
  - Throttling limits are applied to the API keys
  - Quotas limits is the overall number of maximum requests

### Logging & Tracing

- CloudWatch Logs
  - Log contains information about request/response body ( sensitive data might be here )
  - Enable CloudWatch logging at the Stage level (with Log Level - ERROR, DEBUG, INFO)
  - Can override settings on a per API basis
- X-Ray
  - Enable tracing to get extra information about requests in API Gateway
  - X-Ray API Gateway + AWS Lambda gives you the full picture

### CloudWatch Metrics

- Metrics are _by stage_, Possibility to enable detailed metrics
- **CacheHitCount** & **CacheMissCount**: efficiency of the cache
- **Count**:The total number API requests in a given period.
- **IntegrationLatency**:The time between when API Gateway relays a request to the backend and when it receives a response from the backend.
- **Latency**:The time between when API Gateway receives a request from a client and when it returns a response to the client.The latency includes the integration latency and other API Gateway overhead.
- 4XXError (client-side) & 5XXError (server-side)

### API Gateway Throttling

- **Account Limit**
  - API Gateway throttles requests at10000 rps across all API
  - Soft limit that can be increased upon request
- In case of throttling => _429_ Too Many Requests (retriable error)
- Can set Stage limit & Method limits to improve performance
- Or you can define Usage Plans to throttle per customer
- _Just like Lambda Concurrency, one API that is overloaded, if not limited, can cause the other APIs to be throttled_

### API Gateway - Errors

- **4xx means Client errors **
  - 400: Bad Request
  - 403:Access Denied,WAF filtered
  - 429: Quota exceeded,Throttle
- **5xx means Server errors**
  - 502: Bad Gateway Exception, usually for an incompatible output returned from a Lambda proxy integration backend and occasionally for out-of-order invocations due to heavy loads.
  - 503: Service Unavailable Exception
  - 504: Integration Failure – ex Endpoint Request Timed-out Exception API Gateway requests time out after 29 second maximum

### CORS

- CORS must be enabled when you receive API calls from another domain.
- The OPTIONS pre-flight request must contain the following headers:
  - Access-Control-Allow-Methods
  - Access-Control-Allow-Headers
  - Access-Control-Allow-Origin
- CORS can be enabled through the console
- Make sure the integration is not a proxy, otherwise you have to put the header manually in the function (because the function is the proxy)

### API Gateway - Authorization

- **Resource Based Policies**:
  - Can authenticate
    - Users from specific accounts
    - Source Ip ranges or CIDR blocks
    - VPCs or VPC endpoints in any account
  - Available in all endpoint types
- **Identity Based policies**
  - leverages sigv4
  - passes iam credentials into headers
  - authentication and authorization
  - for users/roles within your account
- **Lambda Authorizer**
  - Lambda validates the token in the header
  - option to cache
  - must return an IAM policy
  - authentication and authorization
  - good for 3rd party stuff, OAuth, SAML
    ![clipboard.png](inkdrop://file:Lxy8trns-)
- **Cognito User Pools**
  - Users can sign in to a web or mobile app through cognito
  - can integrate with social identity providers
  - all members have a directory profile accessible through the SDK
  - provides sign-up and sign in
  - customizable Web ui
  - supoprts MFA
    ![clipboard.png](inkdrop://file:iO-SnZH-8)

### API Gateway – Security - IAM Permissions

- **IAM:**
  - Great for users / roles already within your AWS account, + resource policy for cross account
  - Handle authentication + authorization
  - Leverages Signature v4
- **Custom Authorizer / Lambda Authorizer: **
  - Great for 3rd party tokens
  - Very flexible in terms of what IAM policy is returned
  - Handle Authentication verification + Authorization in the Lambda function
  - Pay per Lambda invocation, results are cached
- **Cognito User Pool:**
  - You manage your own user pool (can be backed by Facebook, Google login etc...)
  - No need to write any custom code
  - Must implement authorization in the backend

### HTTP API vs REST API

- HTTP APIs
  - low-latency, cost-effective AWS Lambda proxy, HTTP proxy APIs and private integration (no data mapping)
  - support OIDC and OAuth 2.0 authorization, and built-in support for CORS
  - No usage plans and API keys
- REST APIs
  - All features (except Native OpenID Connect / OAuth 2.0)

![clipboard.png](inkdrop://file:Xi-vZ_zGb)

### WebSocket API

- What’s WebSocket?
  - Two-way interactive communication between a user’s browser and a server
  - Server can push information to the client
  - This enables stateful application use cases
- Used in real time Apps, financial trading and games
- Works With AWS Services or HTTP hendpoints

![clipboard.png](inkdrop://file:VCiY6J0jc)

### API Gateway – WebSocket API – Routing

- Incoming JSON messages are routed to different backend
- If no routes => sent to $default
- You request a _route selection expression_ to
- The result is evaluated against the route keys available in your API Gateway
- The route is then connected to the backend you’ve setup through API Gateway
- _Basically a router based on an attribute from the incoming JSON message that routes based on a table on API Gateway_

### API Gateway - Architecture

- Create a single interface for all the microservices in your company
- Use API endpoints with various resources
- Apply a simple domain name and SSL certificates
- Can apply forwarding and transformation rules at the API Gateway level

![clipboard.png](inkdrop://file:yRDulPkrM)

### Logging and Monitoring

- Logs through CloudWatch
- API Gateway dashboard for visualization (there is a rest api for it too)
- CloudTrail auditable for changes to the API
- **Useful metrics to know**
  - _IntegrationLatency_: time from gateway to endpoint and back, measures the responsiveness of the backend
  - _Latency_: full roundtrip, measures the responsiveness of the API
  - _CacheHitCount_ & _CacheMissCount_ optimize cache capacities
