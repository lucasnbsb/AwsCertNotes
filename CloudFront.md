# Cloudfront - Edge - CDN - Cache

- 216 presence points
- better read performance
- DDoS protection
- Origins:
  - S3 bucket
    - Origin Access Control (OAC) deals with the security
    - for distributing files at the edge
  - Custom Origin (HTTP)
    - ALB
    - EC2 instance
    - S3 website
    - Any Http backend you want
- It's pretty much a writeahead cache.
- Can set a TTL

| CloudFront                           | S3 Cross region replication               |
| ------------------------------------ | ----------------------------------------- |
| global edge                          | you set up each region                    |
| cached for a TTL                     | updates in real time                      |
| good for static content _everywhere_ | Good for dynamic content in a few regions |

### CloudFront Cache Policy

- this is all layer 7
- cache lives in the edge location
- On cache miss
  - reaches into the origin to retrieve the object and cache it
- Each object is identified with a cache key (like a hash)
  - Unique identifier for every object in the cache
  - default: `hostname + resource portion of the URL`
  - You can add other elements using Cloudfront Cache Policies like:
    - Http headers (whitelist)
    - cookies (whitelist , include all except, all)
    - query strings (whitelist, include all except, all)
- You control the TTL _(0 sec to 1 year)_
- By default*EVERYTHING* that's in the cache key is forwarded into the origin request
  - You can change that with an origin request policy

### Origin Request Policy

- a Whitelist specifying what get's forwarded into the origin request
- does not affect cache, only the cache policy does

![clipboard.png](inkdrop://file:F8HU5gjMa)

Basically you need to pass some stuff to the cache key otherwise the request doesn't reach the origin complete enough

### CloudFront - Cache Invalidations

- In case you update the origin, you would have to wait the TTL to refresh the cache
- You can force the update with a CloudFront Invalidation
- you can invalidate All or just a path

### CloudFront - Cache Behaviors

- Different settings for different URL paths
- one route might hit your APL other might hit a bucket
  - all different settings might cache based on different parameters

### CloudFront - ALB or EC2 As an origin

- Instances **must** be public
  - must have a SG to allow public ip of edge locations (dont just open up to everything)
- ALBs must be public
  - same with the SGs

### CloudFront - Geo Restriction

- You can set an allow list or a block list

### CloudFront Signed URL / Signed Cookies

_Distribute paid shared content to premium users over the world_

- Attach a policy with

  - URL expiration
  - Ip ranges that can acess
  - Trusted Signes (what accounts can create urls)
  - Shared content: short lived URLs
  - Private content: you can make it last for years

- Signed
  - URL = acess to individual files
  - Cookies = acess to multiple files

| CF signed URL                          | S3 pre-signed URL                           |
| -------------------------------------- | ------------------------------------------- |
| acess to the path no matter the origin | send the request as the one issuing the url |
| Account wide key pair managed by root  | Uses your IAM key                           |
| Can filter Ip, path, date, expiration  | limited lifetime                            |
| Leverages the cache                    |                                             |

- Two types of keys
  - Either a trusted key group
    - You generate your own public/private keys and your applications use them to sign URLs
  - Or an AWS account with a CloudFront Key Pair
    - not recommended because only the root account can generate one

### CloudFront - Pricing

- The cost per edge location varies
- You can reduce the number of edges to save money
- Price classes:
  - Price Class All: all regions
  - Price Class 200: Most but excludes the most expensive
  - Price class 100: North america and europe

### CloudFront - multiple origin / origin groups

- the "high availability option"
- One primary and one secondary origins for failover
- You get **Region Level** disaster recovery

### Field Level Encryption

- Protect sensitive information
- Extra layer of security
  - Specify in the POST what you want to encrypt
    - <= 10 fields
  - specify the public key
  - The edge location encrypts in-flight, it hits cloudfront already encripted
  - the decrypt private key is in the webservers

### CloudFront - Real Time Logs

yaou can send your logs into a Kinesis Data Streams

- Monitor, Analyze and take actions based on metrics and data
- You choose:
  - Sampling rates
  - Fields or cache behaviors (path patterns) to watch
