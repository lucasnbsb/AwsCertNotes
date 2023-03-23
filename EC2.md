# EC2 - Elastic Compute Cloud

#### Configuration

- OS
  - Linux
  - Windows
  - Mac
- RAM
- STORAGE
  - network-attached (EBS & EFS)
  - Hardware (EC2 instance store)
- Network Card
- Firewall rules
- Bootstrap script (EC2 User Data)

#### User Data:

A script that runs _Once_ when the instance is first started
_Always runs in the Root user of the machine_

_Obs:_ if you stop an instance it changes the public ipv4

#### Instance Types:

- General Purpose: **T**
- Compute Optimized: **C**
  - Use it for gaming servers or machine learning
- Memory Optimized: **R** (for ram)
  - use it for BI type stuff
- Accelerated Computing
- Storage Optimized: **I**
  - DW, databases, etc

#### Instance name convention:

```
formation:
<Instance Class><Generation>.<size>

Examples:
m5.2xlarge
t2.micro
```

#### In EC2: Security Groups === Firewall

Controll traffic in and out of the EC2 instances

- Only contain _Allow_ rules

- SGs can be attatched to multiple instances
- Scoped to region | VPC
- Lives Outside of EC2
- Good practice to block SSH
- A SG can authorize another SG to access inbound
- _Any_ timeout in a EC2 Instance is because of a security group rule.

**Defaults**

- ALL INBOUND - _BLOCKED_
- ALL OUTBOUND - _AUTHORIZED_

- SSH instance connect: browser based SSH

#### Pricing

- On-demand instances (pay by the second)
  -
- Reserved (1 && 3 years)
  - locked attributes
  - cheaper than on demand
  - scoped into region or zone
- Convertible Reserved Instance
  - can change some attributes
- Savings Plans (long workload, commitment to usage)
  - Set how much you want to spend/month
  - extra is billed on demand
- Spot Instances (short workloads)
  - You can lose the instances
  - define the amount you want to pay
  - NOT SUITABLE FOR DATABASES
  - Use it fow failure resistant workloads
- Dedicated instances (no hardware sharing)

  - Use it for compliance reasons
  - you can use your own software licenses
  - On demand | Reserved
  - Most expensive
  - For software with complicated licensing models.
  - instances placed automatically
  - May share hardware with instances on the same account

- Capacity Reservations -> oon demand price, pay even if you dont use it

_Dedicated Host_ you have visibility into everything including hardware
