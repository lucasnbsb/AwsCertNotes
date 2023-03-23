# Taxonomy

- Region:

  - Availability Zone

- IAM

  - Groups
    - Users
  - Roles
    - Services

- EC2
  - EBS volumes (elastic block store)
    - Snapshots
    - AMI (amazon machine image)
    - Types
      - gp
      - io
      - st/sc
      - instance store
  - EFS file systems

## Region

#### How to choose:

- Compliance: some governments have rules about where the data is
- Latency
- Available services: vary between regions
- Pricing: differs from region to region

**Most services are reagion scoped, some are global**

### Availability Zones:

Each have different:

- Energy ( and redundancy )
- Networking

Mainly there for disaster prevention

_Global Infrastructure -> AWS regional services to see all that is available in your region_
