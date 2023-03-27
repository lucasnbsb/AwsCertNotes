# ECS, ECR & Fargate (Containers)

_Docker: lightweight containers running over a Daemon_

- ECS = _Elastic Container Cervice_
- EKS = _Elastic Kubernates Service_
- Fargate = Serverless container platform ( ECS and EKS compatible )
- ECR _Elastic Container Register_ - is docker hub for AWS
  - has a public facing register too

## ECS

- Launching a Docker container = Launchiong a ECS Task on ECS clusters
- EC2 Launch Type
  - You provision and maintain the instances
  - Each Ec2 instance runs the ECS agent and registers in the cluster
- Fargate Launch Type
  - You don't manage the EC2 Instances
  - Serverless
  - just create task definitions
  - To scale just increase tasks
  - recommended

#### ECS - IAM roles

- EC2 Launch:
  - use EC2 Instance profile and do everything there
    - EC2 instance calls ECS service
    - Send logs to cloudwatch
    - pull images from ECR
    - Reference secrets in the Secrets Manager
  - The ECS cluster can create his own auto-scaling groups
- ECS Task Role:
  - each task runs it's own stuff
  - lives inside the Task Deffinition

#### ECS - LB integrations

Users -> LB -> ECS Cluster -> EC2 Instances -> tasks

- **ALB** supports most cases
- **NLB** only for high throughput
- **ELB** supported but not recommended ( no fargate )

#### ECS - Data Volumes (EFS)

- mount EFS file system onto the ECS tasks
- Any task running in any AZ shares the data in the EFS
- Fargate + EFS = serverless
  - Persistent Multi-AZ shared storage

_S3 can't be a file system for ECS_
_Application type_

- Service: Group of tasks handling a long-running computation that can be stopped or started ex: webapps
- Tasks: standalone tasks that runs and ends: a job

### ECS Auto-scaling

On:

- average CPU%
- Average Memory%
- ALB request count per target

- Target-Tracking: one of the above targets
- Step Scaling: specified CloudWatch alarm
- Scheduled Scaling: date and time for predictable changes

ECS service Auto Scaling (task level) != EC2 Auto Scaling (instance level)
Fargate Auto scaling is much easier because it's provided (exam pushes)

for the EC2 launch type there are 2 ways to go

- Auto Scaling Group

  - scale based on CPU
  - add EC2 instances over time

- ECS Cluster Capacity Provider
  - Auto watches CPU and Ram to determine if you are missing capacity
  - Paired with an ASG, provisions automatically

### ECS Rolling updates

- you set up:
  - minimum healthy percent (default 100%)
  - maximum percent (default 200%)
    The default means it creates all the V2s and then removes all the V1s

### ECS common Architectures:

**Task invoked by Event Bridge event**

![clipboard.png](inkdrop://file:VA9X4IZg9)

**Task invoked by Event Bridge Schedule**
![clipboard.png](inkdrop://file:or-7viMRO)

### ECS Task Definitions

- Metadata in JSON form to tell ECS how to run a container
- Contains:
  - Image Name
  - Port Binding for Host and container
    - Set the host port to 0 to enable multiple containers of the same type running on the same host
  - Memory And CPU required
  - Environment variables
  - Networking Info
  - IAM Role
  - Logging config (into every kind of service)

Up to 10 containers per class definition
_Basicaly all you need to run a machine, very similar to EC2 profiles + docker stuff_

### ECS Load Balancing ( EC2 Lauch Type )

- If you deffine only the container port the _ALB_ knows how to hit the right ports on your EC2 instances
  - Dynamic Host Port Mapping
  - _ONLY WITH THE ALB_
  - The SG must allow ANY port to the ALB

### ECS Load Balancing ( Fargate )

- Each task has its own Unique Private IP
- Only define the container port
- ECS ENI SG allow port 80 from the ALB
- ALB SG allow 80/443 from the web
- requests from the ALB hit por 80 on the private IP inside the cluster and then get redirected to the right container port

### IAM Role per Task Definition !important!

The role is deffined at the _TASK DEFINITION_ level, ALL tasks created with that definition have that role

### ECS Environment Variables

- Hardcoded (non sensitive stuff like URLs)
- SSM Parameter Store (sensitive)
- Secrets Manager (sensitive)
- Environment Files through SD (bulk)

### ECS - Sharing data in the same task definition (EC2 or Fargate tasks)

Usualy for a metrics & logs sidecar

- Create a shared storage
  - EC2 Tasks
    - Use the EC2 instance storage (tied to the lifecycle of EC2)
  - Fargate Tasks
    - Use ephemeral storage, tied to the containers
    - 20 (default) ~ 200GB

### ECT - Where to place tasks? (only EC2, fargate takes care of it)

- Task Placement Strategy and Task Placement Constraints

* Identify instances that have capacity CPU / memory
* Look at the contraints
* Find the one that best fits the strategy

#### ECT Placement Strategies

- Binpack: least available CPU or memory. Packs as much as possible into one before going into another
  - Most cost Savings
- Random: Just random
- Spread: spread evenly based on a value:
  - instance id
  - availability zones

You can combine: spread on AZ and binpack on memory

#### ECT Placement Contraints

- Distinc instance
- numberOf:
  - uses cluster query language

### ECR

- Private (ECR) and public ( Gallery )
- Integrated with EC3, runs on S3
- Access controlled through IAM
- Can be encrypted with KMS
- Supports:
  - Image tags
  - Vulnerability scanning
  - Image Lifecycle

## AWS Copilot

- CLI tool to build and release production ready apps in containers
- CLI or yaml to describe the architecture -> AWS Copilot -> ECS/Fargate/AppRunner
  - basicaly a cli to provision everything on the container services
  - logs, debugging, Health checks and CD with CodePipeline

## EKS

- Kubernetes is supposed to be cloud agnostic so.
- ASG -> EKS Pods -> EKS Nodes (containers)
