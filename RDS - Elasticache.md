# RDS - Relational Data Services

_Managed_ DB service
Supports:

- Postgres
- MySQl
- MariaDB
- Oracle
- MSSql
- Aurora

Managed:

- Auto provisioning
- OS patching
- Read replicas
- Dashboards
- Scaling
  - Vert
  - Horizontal
- Storage backed by EBS
  - Auto-Scaling Storage: has to be enabled
- Access can be managed by IAM
- Up to 5 read replicas (remember the 6db diagram)
- _CAN'T SSH into the instance_
- To enable SSL connections, run Require SSL statement for all users

### Read Replicas:

- Replication happens Asynchronously
- Read replicans don't pay extra within the same **REGION**
  - Those exceptions are usually for managed services
- Create new dns names for the replicas

### Multi AZ (for disaster recovery)

- One DNS name, auto failover of the master database
- SYNC replication
- no manual intervention
- Not for scaling, just a backup instance for the master
- Read replicas can be the multi az backup
- Can cross region lines
- Going from Single AZ to Multi AZ is **ZERO DOWNTIME**, just modify the settings
  - Internally:
    - creates a snapshot
    - restores it on the other DB

### Amazon Aurora

- Proprietary DB
- Compatible with Postgres or MySql
- Cloud Optimized, up to 5x performance on AWS
- Shared storage volume
- Grows auto in 10GB increments up to 128 TB
- up to 15 read replicas (instead of 5)
- Failover ins instant
- Costs 20% more but it's so efficient that it's cheaper
- _High Availability mode_

  - 6 copies across 3 AZs
  - 4 needed for writes
  - 3 for reads
  - Self Healing with p2p replication
  - Storage is striped across 100s of volumes
  - One Write Master
  - Failover in less than 30 sec
  - Supports Cross Region Replication
  - Auto-scaling read replicas
  - Automated Patching
  - Backtrack: ( restores without backups )
  - Reader Endpoint
    - It's a read replica connection load balancer

- At rest Encryption
  - Master and replicas can be encrypted
  - If master is not, RR can't be
  - to encrypt it snapshots and restores encrypted
- In-flight encryption: Uses the AWS TLS root
- Security Groups: controll access to the DB
- _NO SSH_ to the underlying instance (except RDS Custom)
- Audit logs can be enabled and watched with Cloud-Watch

### RDS Proxy

- Allows for connection pooling
- If you have lots of connections
- Serverless
- Autoscaling
- High Availability (multi-AZ)
- Reduces failover time by 2/3
- Supports
  - Aurora
  - MySql
  - Postgres
  - MariaDb
- No code changes
- Enforce IAM auth for DB and store in the screts manager
- Never publicaly accessible ( only inside the VPC )
- Fundamental for serverless with lambda functions

## ElastiCache

- _Managed_ service
- Redis or Memcached
- in memory DBs for _High Performance_ and _Low Latency_
- Change the **APP** to query the cache first
- Relieves load in the DB
- Max 5 RR with cluster-mode disabled
- You can store session data there and make the app stateless
- You need a cache eviction policy.

| Redis                          | Memcached             |
| ------------------------------ | --------------------- |
| Multi Az                       | Multi-node (sharding) |
| Read Replicas                  | No HA                 |
| AOF persistence                | Non persistent        |
| Backup and restore             | no backup             |
| Sets and sorted Sets           | multithreaded         |
| Redis Auth to require password |                       |

Do i cache?

- is it safe to cache?
- is it effective to cache
- is the structure cacheable?

**Lazy loading | Cache-Asside | Lazy Population** first option

- All cache misses are written to the cache
- On startup it needs to warm up
- A cache miss causes 3 round trips
- Updates to the database are not reflected automatically

**Write through** not the first priority

- Whenever you write you write on the cache too
- No stale data in the cache
- Write pennalty instead of read penalty
- cache misses can go through lazy loading
- Causes some cache churn

### Cache Eviction and TTL

Eviction policies:

- explicit deletion
- LRU
- TTL (seconds to hours)
  - leaderboards
  - Comments
  - Activity streams

if you have too many evictions: scale your cache

### Amazon MemoryDb for redis

- Redis compatible in memory DB
- over 160 mi requests/second
- durable and scalable
