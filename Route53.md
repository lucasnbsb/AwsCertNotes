# Route 53 - DNS (53 is the original DNS port)

_dig can perform DNS queries in the terminal_

### DNS Terminology

- Domain Registrar: godaddy, r53
- DNS Records:
- Zone File: Contains DNS records
- Name Servers: the app servers for DNS
- Top Level Domain: .com, .net
- Second level domain: Amazon.com, google.com
- Sub:Domain -> comes before the SDL
- Fully Qualified name: includes the protocol: `http://google.com.br`

### Route 53

- Domain Registrar
- performs Health Checks
- 100% uptime SLA
- Each record Contains
  - Domain / sub name
  - Record Type
  - Value (ip)
  - Routing Policy
  - TTL

Record Types:

- A: hostname = IPv4
- AAAA: hostname = IPv6
- CNAME: hostname = hostname (NO CNAME for the top node of a DNS namespace)
  - no CNAME for `amazon.com` but can for `www.amazon.com`
- NS: Name Servers for the hosted zone
- Alias: for AWS resources

### How it works

Recursively calls through the domains from most general to most specific
TLD -> SLD -> Sub-domain

### Hosted Zones

_A container for reccords and rules on how to route to subdomains_

- Public Hosted Zones: Records on how to route on the public internet
- Private Hosted Zones: Records on how to route traffic in one or more VPCs
  - lives inside the VPC

### Record TTL

- tells the client to cache for TTL seconds
- prevents overquerying the DNS
- Too high and you can get outdated records
- Too low and you get too much traffic in your DNS

| CNAME                    | ALIAS                     |
| ------------------------ | ------------------------- |
| Host <-> Host            | Hostname <-> AWS resource |
| Only for non root domain | Root or non root          |
|                          | Free                      |
|                          | Ip independant            |
|                          | Always A / AAAA           |
| Can health check         | can health check          |

Alias Targets

- ELB
- Cloud front distros
- API gateway
- Elastic Beanstailk
- S3
- VPC endpoints
- Global Acelerator
- Route 53 records in the same zone

## Health Checks

- not possible on simple routing policies
- usually hits the LB
- **ALL HEALTHCHECKERS LIVE ON THE PUBLIC INTERNET**

Types of Health Check:

- **Monitor an Endpoint**
  - layer 7
  - about 15 global health checkers test the endpoint
  - interval 30/10 sec
  - if 18% report healthy it's healthy
  - you can chose which locations the HC come from
  - Pass only on 2XX or 3XX http code
  - can be set up to pass/fail based on the first 5120 bytes of the response
  - You need to configure the firewall to allow incoming from Route 53 HCers
- **Calculated HC**
  - combine multiple HC into a single HC
  - use OR, AND or NOT
  - Up to 256 child HC
  - you specify how many mean pass/fail
- **Private Hosted Zones**
  - since the HC are outside the VPC they can't access private endpoints
  - You can create a CloudWatch Metric, associate with a CloudWatch Alarm and create a health check on the alarm.

## Routing Policies

_Routing from a DNS point_

#### Simple

- Route to a single resource
  - 1 name <-> N values, client picks at random
  - **NO HEALTH CHECKS**

### Weighted

- X% of requests go for each resource by weight
- Use it to load balance and test new versions (a/b testing)
- weight can be 0-255
- weight 0 blocks the resource

### Latency

- Redirects to the lowest latency (closer)
- **Can** combine with Healthchecks for failover

### Failover

- 2 instances, Primary and secundary
- primary needs to have a health check
  - When it fails Route 53 maps the traffic to the seconday ( can be anywhere else )

### Geolocation

- By Continent, Country or US state
- Should create a default record (for no location match)
- Localization, restricting distribution for compliance, Etc
  -

### Traffic Flow

- Visual editor for your rules
- you can combine different routing policies
- one polocy can send you to another

### Multi-Value

- Route to multiple resources
- Can Health check
- Up to **8 healthy records** for each Multi-value Query
- Client choses which one of the up to 8
- the Health checked alternative to simple routing policies

### 3rd party domains in R53

- Create a hosted zone in the R53
- update the NS records on the 3rd party to point to R53
