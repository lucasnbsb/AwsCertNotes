# Elastic Beanstalk (1 or 2 questions here)

_Deploying safely and effectively_
Configuring everything is tedious, most stuf is ALB + ASG

- ELB: one interface, everything we saw so far
  - Full control over the config, but one interface
  - Beanstalk is free, you pay for the underlying infra
  - Components:
    - Application
    - App version
    - Environment
      - collection of resources
      - Tiers (web server, workers etc)
      - Envivonments ( test, dev, prod)
  - Supports most back-end languages (backend focused)
  - ELB is basically a wizard for backends

### Beanstalk environments

- you can chose to create web or worker environments
- pick a domain, a platform, and app code
- you can configure all the base infrastructure from beanstalk before launching an environment
  - instances, capacity, load balancer, observability, network, database etc
- all common architectures are also included

### Deployments Options for updates

#### All at once

- downs every v1 and ups the v2
- causes Downtime
- Fastest deployments
- Good for dev environments
- no cost

#### Rolling

- good for apps running belo capacity
- set the bucket size
- downs (bucket size) instances and ups new ones, rolls to the next bucket
- No additional cost, can take some time
- runs 2 versions at the same time

#### Rolling with additional batches

- App running at capacity
- first prod capable mode
- small additional cost
- instead of downing buckets, adds the bucket size to keep same capacity
- then terminates the additional batches
- Zero downtime, new code is deployed into new instances in a temporary ASG
- very quick rollback by terminating the new ASG
- temp asg moves instances into the main ASG

#### Blue/Green

- you deploy a staging environment (green)
- you use rouyte 53 or other routing options to route into the new environment
- it's very manua, you just use ELB to ease the new environment and then wheight traffics everything
- you can also swap the URLs (CNAME change)
- only one that causes DNS change

#### Traffic Splitting - for Canary Testing

- send a small amount of traffict to the temp ASG
- monitor the temp ask
- metrics can trigger an automated rollback
- very automated

![clipboard.png](inkdrop://file:vVVY0dRTJ)

#### Elastic beanstalk CLI

-> more of a Dev-ops exam type of stuff

- Create
- status
- health
- events
- logs
- open
- deploy
- config
- terminate etc

- Describe dependencies
- package code as a zip and describe dependencies (package.json)
- console: upload zip file
- CLI create new app using CLI (sends zip) and deploy
- ELB does the rest

#### Beanstalk Lifecycle Policy

ELB can store at most 1000 app versions, you set up a policy to delete old versions

- based on time
- based on space

#### EB Extentions

- A zip file containing code to be deployed
- all parameters from the UI can be configured using code
- Requirements
  - put everything in `/.ebextentions` file
  - files must end with `.config` (but can be yaml syntax)
  - options_settings can modify the default settings
  - you can add resources from here
  - _everything in the .ebextentions folder gets deleted if the environment goes away_

#### EBS - Under the hood

- ELB uses cloudformation under the hood
- CloudFormation is the infrastructure as code initiative

#### CLoning an EBS environment

- The cloning wizard has a description of what is going to be created
- Options for change are very limited
  - you can change names, urls and platform versions
  - the underlying resources stay mainly the same
- everything is preserved

#### EBSB - Migration how to

- After creating a EBS env you cant change the ELB type, only configs
- To migrate load balancers:
  - create a new environment with the same config except LB
  - deploy app to the new environment
  - Cname swap or Route53 reroute

#### RDS with EBS

- you can use a dev environment on beanstalk with your RDS database
- how to migrate the RDS out
  - Create a snapshot of the DB
  - protect it from deletion on the console
  - Create a new EBS without RDS and point to the existing
  - Cname swap or Route53 update
  - Terminate old instance (RDS stays behind)
  - delete the remaining stuff from the old

#### EBS - Single vs multiple docker

| Single                                                 | Multi                                                      |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| provide a dockerfile or .json with a link to the image | Multiple containers per EC2 instance                       |
| In a single container EBS does not go through ECS      | creates ECS cluster, instances, load balancer (high av)    |
|                                                        | requires a Dockerrun.aws.json to generate task definitions |
|                                                        | Docker images must be pre-built (no dockerfile)            |
|                                                        | The load balancer takes care of routing                    |

### EBS advanced options

#### EBS over HTTPS

- Load the SSL cert in the Load balancer

  - can be done in console or coede
  - Cert can come from ACM or CLI
  - must configure a SG to allow incoming 443 into the LB

- OR -> Redirect HTTP to HTTPS
  - you can configure the instances to redirect http to https or the ALB
  - make sure the health checks are not redirected

#### Web Server vs Worker environment

- Long tasks should go into the worker envirnonment to decouple the app
- Periodic tasks can go into a cron.yaml
- Usualy uses a SQS queue to orchestrate jobs in the worker tier

#### Custom Platform

- very advanced, but more definittions, everything can be configured
- Deefine your own AMI using Platform.yaml
- Build using the Packer software
- Custom platform is custom everything, from image up
