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

- _A_: hostname = IPv4
- _AAAA_: hostname = IPv6
- _CNAME_: hostname = hostname (NO CNAME for the top node of a DNS namespace)
  - no CNAME for `amazon.com` but can for `www.amazon.com`
  - NS: Name Servers for the hosted zone
- _Alias_: for AWS resources
- _CAA_ (certification authority authorization).
- _MX_ (mail exchange record).
- _NAPTR_ (name authority pointer record).
- _NS_ (name server record).
- _PTR_ (pointer record).
- _SOA_ (start of authority record).
- _SPF_ (sender policy framework).
- _SRV_ (service locator).
- _TXT_ (text record).

#### Aliases

- points at the dns name of the service aliased
- has TTL
- Alias can be used for resolving apex/naked domains (example.com)
- CNAME cannot
- Use an alias where possible (CNAME queries are paid and alias is free)

**Alias Targets**

- ELB
- Cloud front distros
- API gateway
- Elastic Beanstailk
- S3
- VPC endpoints
- Global Acelerator
- Route 53 records in the same zone

### How it works

Recursively calls through the domains from most general to most specific
TLD -> SLD -> Sub-domain

the name servers might not know where the final destination is, but they know where there is a server
that knows where the next subsection is.

- When you register through R53 it becomes the authoritative DNS record for your domain
- To bring a domain into R53 create a public hosted zone and change the DNS Name Servers from the provider to the R53 name servers
  - But only if the Top-level Domain is supported
-

### Hosted Zones

_A container for reccords and rules on how to route to subdomains_

- Public Hosted Zones: Records on how to route on the public internet
- Private Hosted Zones: Records on how to route traffic in one or more VPCs
  - lives inside the VPC
  - can be associated with the VPC on another account
    - accA owns hosted zone, accB owns VPC
    - From accA create the association authorization
    - from accB create the association
    - delete the association authorization

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

## Health Checks

- not possible on simple routing policies
- usually hits the LB
- **ALL HEALTHCHECKERS LIVE ON THE PUBLIC INTERNET**

**Can be pointed at:**

- Endpoints (ip or domain name)
- Status of other health checks
- Status of a CloudWatch Alarm

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

**Explicit types of Health checks**

- _HTTP_ waits for a code > 200 < 400
- _HTTPS_ waits for a code > 200 < 400
- _HTTPSTRMATCH_: searches the **first 5120 bytes** of the response body for the string that you specify in **SearchString**.
- _HTTPSSTRMATCH_: searches the **first 5120 bytes** of the response body for the string that you specify in **SearchString**.
- _TCP_: Route 53 tries to establish a TCP connection.
- _CLOUDWATCHMETRIC_: The health check is associated with a CloudWatch alarm.
  - If the state of the alarm is OK, the health check is considered healthy.
  - If the state is ALARM, the health check is considered unhealthy.
  - If CloudWatch doesn’t have sufficient data to determine whether the state is OK or ALARM, the health check status depends on the setting for _InsufficientDataHealthStatus: Healthy, Unhealthy, or LastKnownStatus_.
- _CALCULATED_: For health checks that monitor the status of other health checks, Route 53 adds up the number of health checks that Route 53 health checkers consider to be healthy and compares that number with the value of _HealthThreshold_.

## Routing Policies

_Routing from a DNS point_

#### Simple

- Route to a single resource
  - 1 name = N values, client picks at random
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
-     If primary is down (based on health checks), routes to secondary destination

### Geolocation

- By Continent, Country or US state
- Should create a default record (for no location match)
- Localization, restricting distribution for compliance, Etc

### Geoproximity

- routes to the closest region within a geographic area

### Multi-Value

- Route to multiple resources
- Can Health check
- Up to **8 healthy records** for each Multi-value Query
- Client choses which one of the up to 8
- the Health checked alternative to simple routing policies

### Traffic Flow

- Visual editor for your rules
- you can combine different routing policies
- one policy can send you to another
