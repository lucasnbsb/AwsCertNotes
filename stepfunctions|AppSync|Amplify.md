# Step Fucntions

- Model your workflows as **state machines** (one per workflow)
- Written in JSON
- Visualization of the workflow and the execution of the workflow, as well as history
- Start workflow with SDK call, API Gateway, Event Bridge (CloudWatch Event)

#### Task States

- _Do some work in your state machine_
- **Invoke one AWS service**
  - Can invoke a Lambda function
  - Run an AWS Batch job
  - Run an ECS task and wait for it to complete
  - Insert an item from DynamoDB
  - Publish message to SNS, SQS
  - Launch another Step Function workflow...
- **Run an one Activity**
  - EC2, Amazon ECS, on-premises
  - Activities poll the Step functions for work
  - Activities send results back to Step Functions

#### States

- **Choice State** - Test for a condition to send to a branch (or default branch)
- **Fail or Succeed State** - Stop execution with failure or success
- **Pass State** - Simply pass its input to its output or inject some fixed data, without performing work.
- **Wait State** - Provide a delay for a certain amount of time or until a specified time/date.
- **Map State** - Dynamically iterate steps.’
- **Parallel State** - Begin parallel branches of execution.

#### Error Handling

- Any state can encounter runtime errors for various reasons:
  - State machine definition issues (for example, no matching rule in a Choice state)
  - Task failures (for example, an exception in a Lambda function)
  - Transient issues (for example, network partition events)
- Use **Retry** (to retry failed state) and **Catch** (transition to failure path) in the State Machine to handle the errors instead of inside the Application Code
- Predefined error codes:
  - **States.ALL** : matches any error name
  - **States.Timeout**:Task ran longer thanTimeoutSeconds or no heartbeat received
  - **States.TaskFailed**: execution failure
  - **States.Permissions**: insufficient privileges to execute code
- The state may report is own errors

#### Retry options (task or parallel state)

- Evaluated from top to bottom
- **ErrorEquals**: match a specific kind of error
- **IntervalSeconds**: initial delay before retrying
- **BackoffRate**: multiple the delay after each retry
- **MaxAttempts**: default to 3, set to 0 for never retried
- When max attempts are reached, the Catch kicks in

### Catch options

- **Evaluated** from top to bottom
- **ErrorEquals**: match a specific kind of error
- **Next**: State to send to
- **ResultPath** - A path that determines what input is sent to the state specified in the Next field.

#### Result Path

this is how you pass the errors to the next:

```
"ResultPath": "$.error"
```

### Step Functions – Wait for Task Token

- Allows you to pause Step Functions during aTask until aTaskToken is returned
- Task might wait for other AWS services, human approval, 3rd party integration, call legacy systems...
- Append **.waitForTaskToken** to the **Resource** field to tell Step Functions to wait for the Task Token to be returned
- Task will pause until it receives thatTaskToken back with a **SendTaskSuccess** or **SendTaskFailure** API call

- Enables you to have theTask work performed by an **Activity Worker**
- **Activity Worker** apps can be running on EC2, Lambda, mobile device...
- Activity Worker poll for a Task using **GetActivityTask** API
- After Activity Worker completes its work, it sends a response of its success/failure using **SendTaskSuccess** or **SendTaskFailure**
- To keep theTask active:
  - Configure how long a task can wait by setting **TimeoutSeconds**
  - Periodically send a heartbeat from your Activity Worker using **SendTaskHeartBeat** within the time you set in **HeartBeatSeconds**
- By configuring a long TimeoutSeconds and actively sending a heartbeat,ActivityTask can wait up to _1 year_ (but why though)

### Step Functions – Standard vs. Express

| Standard                                 | Express                                            |
| ---------------------------------------- | -------------------------------------------------- |
| <= 1 year                                | <= 5 minutes                                       |
| Exactly once execution                   |                                                    |
| Over 2000/second                         | over 100k/second                                   |
| <= 90 days of history or cloudwatch logs | only cloudwatch logs                               |
| priced by # of transitions               | # of executions, duration, memory, other resources |
| Non-idempontent actions                  | Iot ingestion, streaming data, mobile backends     |

#### Express Execution models:

| Asynchronous at-least once                               | Synchronous at-most once                           |
| -------------------------------------------------------- | -------------------------------------------------- |
| doesn't wait for workflow to complete, result in CW logs | waits for workflow to complete                     |
| don't need immediate response                            | need an immediate response                         |
| must manage idempotence                                  | can be invoked from API Gateway or Lambda Function |

# AppSync - GrahpQl, Web Sockets and data dynchronization

- **GraphQL** makes it easy for applications to get exactly the data they need.
- This includes combining data from one or more sources
  - NoSQL data stores, Relational databases, HTTP APIs...
  - Integrates with DynamoDB, Aurora, OpenSearch & others
  - Custom sources with AWS Lambda
- Retrieve data in real-time with WebSocket or MQTT on WebSocket
- Local data acess & data synchronization for mobile apps.
- Starts by uploading a graphql schema

![clipboard.png](inkdrop://file:8s6QzvCCE)

#### Security

- **API_KEY**
- **AWS_IAM**: IAM users / roles / cross-account access
- **OPENID_CONNECT**: OpenID Connect provider / JSON Web Token
- **AMAZON_COGNITO_USER_POOLS**
- for custoom domains and HTTPS use cloudfront in front of AppSync

# AWS Amplify

- Amplify

  - Studio - build frontend and backend
  - CLI - configure the backend with a guided CLI workflow
  - Libraries - like an sdk
  - Hosting

- Set of tools to get started with creating mobile and web applications
- “Elastic Beanstalk for mobile and web applications”
- Must-have features such as **data storage**, **authentication**, **storage**, and **machine-learning**, all powered by AWS services
- Front-end libraries with ready-to-use components for React.js,Vue, Javascript, iOS, Android, Flutter, etc...
- Incorporates AWS best practices to for reliability, security, scalability
- Build and deploy with the Amplify CLI or Amplify Studio

#### Important Features

- Authentication
  - uses Cognito
  - User registration, account recovery, MFA, social login
  - pre-built ui components
  - fine-grained auth
- Datastore
  - AppSync and DynamoDb
  - Local data and auto cloud sync without complex code
  - powered by GraphQl
  - Offline and real-time capabilities
  - Visual data modeling w/ Amplify Studio

#### Hosting

- Build and Host Modern Web Apps
- CICD (build, test, deploy)
- Pull Request Previews
- Custom Domains
- Monitoring
- Redirect and Custom Headers
- Password protection

#### AWS Amplify – End-to-End (E2E) Testing

- Run end-to-end (E2E) tests in the test phase in Amplify
- Catch regressions before pushing code to production
- Use the test step to run any test commands at build time (amplify.yml)
- Integrated with **Cypress** testing framework
  - Allows you to generate UI report for your tests
