# EBS - Elastic Block Store

### EC2 Storage volumes

- Volume attatched over network (latency)
- great for failover (usually using snapshots)
- Multiattach is possible in some cases
- **Scoped to AZ**
- Free tier: 30gb of ssd
- has to be provisioned in advance (size and IOPS)

#### Delete on termination (disabled by default)

When the instance terminates delete the root of the EBS
_Keep disabled for failover_

#### Creating and attaching is easy

Formatting and using not so much, but there are tutorials

### Volume types

- gp2 / gp3 (SSD) general purpose, cost effective
  - gp3 can set the storage and iops independently
  - gp2 iops linked to storage ( 3 per GB )
    - max IOPS is reached at 5tb
  - good for vms
- io1 / io2 (SSD) high performance low latence
  - where IOPS is critical
  - Max IOPS
    - io1: 64k
    - io2: 256k
  - Great for databases
  - iops and storage independant
  - io2 has sub-milliseccond latency
  - **supports multi-attach in the same AZ**
  - file system has to be cluster-aware
- st1 / sc1 (HDD)
  - can't be a boot volume
  - Optmized for throughput
  - st1
    - Big data, DW
  - sc1
    - cold storage

## EBS - Snapshots ( goes in a bucket )

_It's recommended to unatatch before creating the snapshot_

- Thats how you transfer EBS from AZ to AZ
- Moving the Snap to an archive tier bucket is 75% cheaper
- takes 1 to 3 days to restore
- You set up the rules for your snap recycle bin ( Retention Rules )
- FSR - Fast Snapshot Restore -> you can pay more to have no latency on first use

## AMI - Amazon Machine Image

- A customization of a EC2 instance
- Faster launches pre-configured by you
- AWS Marketplace has custom AMIs to buy
- Creating an AMI from a machine creates a EBS snapshot behind the scenes
- AMI's are locked to regions, but can be copied

## EC2 Instance Store

- Some kinds of instances have the capacity for Instance store
- Much better performance - **HARDWARE ATTATCHED**
- In-memory
- Risk of data loss
- Deleted on deactivation

## EFS - Elastic File System

- Managed NFS (network file system)
- Can be mounted in multi AZs
- Needs a security group to access
- More expensive than EBS
- Scales automaticaly, pay per use
- Scales to Petabytes auto
- Automatic backups
- Can run it's own user data to attatch to instances
- can create SGs automatically to attatch into different instances
- Performance
  - General purpose ( default )
  - Max i/o
- Throughput
  - Bursting ( 1tb = 50MiB/s )
  - Provisioned ( throughput nad storage decoupled )
  - Elastic
    - Scales throughput up and down

**Storage Tiers**

- Standart: frequent access
- Infrequent access (EFS-IA): lowest cost to store, cost to retrieve
  - a lifecycle policy determines when an item can go into infrequent acess

**AZ**

- Regional: Multi-Az (standart)
- One Zone (good for dev)

| EBS                                  | EFS                                    | Instance Store                |
| ------------------------------------ | -------------------------------------- | ----------------------------- |
| One Instance (some can multiattatch) | Mounts to 100s of instances across AZs | Attatched to the instance     |
| Locked to AZ                         | EFS-IA for cost savings                | goes away with the instance   |
| gp2: io bound to disk size           | more expensive                         | High performance, low latency |
| io1: io independant from disk size   |                                        |                               |
| Migrates via snapshot                |                                        |                               |
| Backups use the IO                   |                                        |                               |
| Root EBS terminates by default       |                                        |                               |

# ELB - Elastic Load Balancer

_High-Availability_: At least 2 AZs are involved, the goal is to survive a data center loss

- Spreads the load
- Single point of DNS access
- SSL termination (HTTPS)
- Spearate public and private traffic
- Handle Failure
- Perform Health Checks
  - Done on a port and route, checks a http 200 return

_ELB is a managed load balancer_

- Automatically integrates with most services

AWS provides 4 kinds of Load Balancer

- CLB: classic load balancer (deprecated)
- ALB: application load balancer (layer 7)
- NLB: network load balancer 2017
- GLB: Gateway load balancer 2020: operates at layer 3 (ip)

_Security_: point the SG from the instances to only allow from the LB
users <-> LB <-> EC2

### ALB

Layer 7: HTTP

- balances within the same machine
- route based on
  - path
  - hostname
  - Query string
  - headers
- Fixed hostname
- aplication serverzs don't see the ip
- performs connection termination
  - real ip goes in the X-Forwarded-For Header, also port and proto
- Great for micro-services and container based apps
- you can choose different AZ for the same load balancer

#### Target Groups (where the traffic is going to be sent)

- EC2 Instances
- ECS tasks
- Lambda Functions
- Private Ip Addresses
  _Health checks is done on the taget group level_

### NLB - Network Load Balancer

- Layer 4 -> TCP | UDP
- Extremely high performance
- No free tier
- One static IP per AZ, supports elastic IP

Same as ALB - create Security Groups and redirect traffic to the LB

- Can point to EC2 instances or IP addresses
- It's possible to have a NLB in front of a ALB to get the best of both worlds

**Health check**

- TCP
- HTTP
- HTTPS
