# VPC - Know it at a high level (1 or 2 questions)

_A regional private network inside the cloud_

Every VPC has a CIDR range: the ip range that can be allocated inside it

### Subnets (a AZ scoped network inside the VPC)

- Public: accessible from the internet
- Private: NOT accessible from the internet

### Gateways

- Internet Gateways: helps VPC instances connect to the net
- NAT gateways: AWS managed -> allows the private subnet access to the net while remaining private ( lives on the public subnet )
- NAT instances: self managed

#### Network ACL & Security Groups

**NACL**: a basic firewall

- **Allow** or **deny** rules
- Attatched to the Subnets
- Only sees IP addresses
- Stateless, you have to map the return traffic

**Security Groups**

- Only **Allow** rules
- attached to the instance level
- Statefull, allows return traffic
- controlls traffic from an ENI / EC2 instance

#### VCP Flow logs

- Information about all the IP traffic in your VPC
  - VPC Flow Logs
  - Subnet Flow Logs
  - Elastic Network Interface Flow Logs
- Monitor every type of co nnection
- Network info of all managed interfaces
  - ELB
  - Elasticache
  - RDS, etc

#### VPC Peerign

_Connecting 2 VPCs as if they are 1_

- CIDR ranges **MUST NOT** overlap
- **NOT TRANSITIVE**

#### VPC Endpoints

_Connect to AWS services using a private network instead of the net_

- More secure
- Lower surface of attack
- VPC Endpoint Gateway:
  - Only for
    - S3
    - DynamoDB
- VPC Endpoint Interface
  - for the rest
- _Only used within your VPC "privately connect to your vpc"_

#### Site to Site VPN & Direct Connect

- S2s:
  - vpn into aws
  - auto encrypted
  - goes over the web
- Direct Conect (DX)
  - takes a **month**
  - **PHYSICAL CABLE**
  - private network

## Classic 3 tier architecture

- public subnet
  - Load Balancer
- private subnet
  - Auto Scaling Group
- data subnet
  - RDS
  - Elasticache
