# AWS Cloud Practitioner Notes

My personal notes and study guide for the AWS Certified Cloud Practitioner / Solutions Architect Associate.

---

## Benefits of the Cloud

- **Agility**: Rapidly develop, test, and launch applications.
- **Pay-as-you-go pricing**: Pay only for what you use, avoiding capital expenses.
- **Economy of Scale**: Benefit from massive economies of scale by sharing costs with other customers.
- **Global Reach**: Deploy applications globally in minutes.
- **Security & Reliability**: Built-in compliance and secure infrastructure.
- **High Availability**: Redundant systems that minimize downtime.
- **Scalability**: Ability to handle growing workloads.
- **Elasticity**: Dynamically scale resources up or down based on demand.
- **Fault Tolerance**: System remains operational even when some components fail.
- **Disaster Recovery**: Practices to quickly restore operations after an outage.

### Six Advantages of Cloud Computing
1. Trade static capital expense for variable operational expense (pay-on-demand).
2. Benefit from massive economies of scale.
3. Stop guessing capacity (scale up or down automatically).
4. Increase speed and agility (launch resources in minutes).
5. Stop spending money running and maintaining data centers (focus on customers).
6. Go global in minutes (deploy in multiple regions).

---

## AWS Global Infrastructure

- **Launched Regions**: 32
- **Availability Zones (AZs)**: 102
- **Direct Connect Locations**: 115
- **Points of Presence (Edge Locations)**: 550+
- **Local Zones**: 35
- **Wavelength Zones**: 29

---

## Identity and Access Management (IAM)

> [!IMPORTANT]
> **PoLP (Principle of Least Privilege)**: Only grant permissions required to perform the task.

- **Global Service**: IAM is a global service; users, groups, and policies apply globally.
- **Users & Groups**: Users can belong to multiple groups. Groups cannot contain other groups.
- **Direct Policies**: It is possible to attach policies directly to users, but it is not recommended (use groups instead).

### IAM Policy Structure
Policies are JSON documents containing:
- **Version**: Policy language version (e.g., `2012-10-17`).
- **Id**: An optional identifier for the policy.
- **Statement**: An array of statements (required).
  - **Sid**: Statement identifier (optional).
  - **Effect**: `Allow` or `Deny`.
  - **Principal**: The account, user, or role to which the policy applies.
  - **Action**: List of actions allowed/denied (e.g., `s3:GetObject`).
  - **Resource**: List of resources to which the actions apply.
  - **Condition**: Optional conditions for when the policy is in effect.

### AWS CLI
Use `aws configure` in the terminal to set up access key, secret access key, and region.

### IAM Roles
Similar to user policies, but roles are assumed by AWS services (e.g., EC2 instances needing access to S3).

### Auditing & Analysis
- **Credential Report**: Lists all users in the account and the status of their credentials (MFA, passwords, access keys).
- **IAM Access Advisor**: Shows when services were last accessed by a user or role.
- **IAM Access Analyzer**: Generates least-privilege policies based on access activity.

---

## Elastic Compute Cloud (EC2)

- **User Data**: Bootstrap script that runs once when the instance launches.
- **Keys**: `.pem` for OpenSSH (Linux/Mac/modern Windows); `.ppk` for PuTTY (older Windows).
- **Security Groups (Firewalls)**:
  - By default, denies all incoming traffic.
  - By default, allows all outgoing traffic.
- **IP Addresses**: Public IPs change on instance reboot. Elastic IPs provide a static, persistent public IP.

### Instance Types
Format: `m5.2xlarge` (m: class, 5: generation, 2xlarge: size).
- **General Purpose**: Balanced compute, memory, and networking.
- **Compute Optimized (C class)**: High-performance processors (Batch processing, transcoding, HPC, gaming).
- **Memory Optimized (R class)**: High memory capacity (Relational/non-relational DBs, in-memory caching).
- **Storage Optimized (I/D/H class)**: High-speed local storage (NoSQL DBs, data warehousing).

### Purchasing Options
- **On-Demand**: Pay by the second/hour, no commitment.
- **Reserved Instances (RI)**: Commit to 1 or 3 years for up to 72% discount.
- **Spot Instances**: Bid on spare capacity for up to 90% discount. Can be terminated with a 2-minute warning.
- **Dedicated Hosts**: Physical servers dedicated to you (useful for licensing compliance or BYOL).
- **Dedicated Instances**: Run on hardware dedicated to you, but may share hardware with other instances in your account.
- **Capacity Reservations**: Reserve capacity in a specific AZ; charged at On-Demand rates.

### Spot Fleets
A collection of Spot and (optional) On-Demand instances.
- **Allocation Strategies**:
  - `lowestPrice`: Cost-optimized, short workloads.
  - `diversified`: High availability, longer workloads.
  - `capacityOptimized`: Matches optimal capacity.
  - `priceCapacityOptimized`: Best choice for most workloads.

### Placement Groups
- **Cluster**: Low-latency, high-throughput in a single AZ.
- **Spread**: Placed on distinct physical hardware (max 7 per group per AZ).
- **Partition**: Distributed across multiple logical partitions within an AZ.

### Elastic Network Interfaces (ENI)
Virtual network cards representing private IPs. Bound to a specific AZ. Can be detached and attached to other instances.

### EC2 Hibernate
Saves RAM contents to EBS boot volume (must be encrypted) and stops the instance. Booting is faster since it restores RAM state instead of doing a full OS boot. Max hibernation duration is 60 days.

---

## Storage: EBS, EFS, and Instance Store

### Elastic Block Store (EBS)
Network-attached block drive.
- Bound to a specific AZ (migrate via snapshots).
- **Snapshots**: Point-in-time backups.
- **Volume Types**:
  - `gp2` / `gp3` (SSD): General-purpose. `gp3` allows setting IOPS and throughput independently.
  - `io1` / `io2` (SSD): Provisioned IOPS for high-performance databases. Supports **EBS Multi-Attach** (up to 16 EC2s in the same AZ).
  - `st1` (HDD): Low-cost, throughput-intensive (MapReduce, log processing).
  - `sc1` (Cold HDD): Lowest cost, infrequent access.
  - Only SSD types (`gp2/3`, `io1/2`) can be boot volumes.

### EC2 Instance Store
Physically attached ephemeral SSD storage. Extremely fast, but data is lost when the instance is stopped or hardware fails.

### Elastic File System (EFS)
Managed network filesystem (NFSv4.1) for Linux instances. Scales automatically and works across multiple AZs.

| Feature | EBS | EFS | Instance Store |
| :--- | :--- | :--- | :--- |
| **Type** | Block Storage | Network File System | Ephemeral Local Storage |
| **Scope** | Single AZ | Multi-AZ | Local Host |
| **Compatibility** | Windows & Linux | Linux only | Windows & Linux |
| **Persistence** | Persistent | Persistent | Ephemeral (Lost on stop) |

---

## Load Balancing & Auto Scaling

### Elastic Load Balancing (ELB)
Distributes incoming traffic across downstream targets.
- **Application Load Balancer (ALB)**: Layer 7 (HTTP/HTTPS/Websockets). Path/Host/Query-routing. Sticky sessions using cookies.
- **Network Load Balancer (NLB)**: Layer 4 (TCP/UDP). Ultra-high performance, static IP/Elastic IP support.
- **Gateway Load Balancer (GWLB)**: Layer 3. Deploys and manages virtual third-party appliances.
- **Cross-Zone Load Balancing**: Distributes traffic evenly across all AZs. (Enabled by default on ALB, disabled on NLB).
- **SNI (Server Name Indication)**: Allows hosting multiple SSL certificates on a single listener (ALB, NLB, CloudFront).
- **Connection Draining (Deregistration Delay)**: Gives existing requests time to finish processing when an instance is being decommissioned.

### Auto Scaling Groups (ASG)
Dynamically scales instances based on demand using:
- **Dynamic Policies**: E.g., target CPU utilization.
- **Scheduled Policies**: Scale at specific times.
- **Predictive Scaling**: Machine learning-based demand forecasting.

---

## Databases: RDS, Aurora, and ElastiCache

### Amazon RDS
Managed relational database service supporting Postgres, MySQL, MariaDB, Oracle, SQL Server, DB2, and Aurora.
- **Read Replicas**: Up to 15 replicas (asynchronous replication). Used to scale reads. Can be promoted to standalone DBs.
- **Multi-AZ**: Synchronous replication to a standby instance in another AZ for disaster recovery. Automatic failover using a single DNS name.
- **RDS Custom**: For Oracle and SQL Server. Grants full OS access to configure settings, install patches, etc.

### Amazon Aurora
AWS-proprietary cloud-optimized relational database.
- Up to 5x faster than MySQL, 3x faster than Postgres.
- Auto-expands storage from 10GB up to 128TB.
- **Aurora Cluster**: 6 copies of data across 3 AZs. Writer endpoint for master, Reader endpoint load balances replicas (up to 15 replicas).
- **Aurora Serverless**: Auto-scales based on actual usage; pay per second.
- **Aurora Global Database**: 1 primary region (read/write), up to 5 secondary read-only regions (replication lag < 1s). Fast disaster recovery promotion (< 1 min).
- **Aurora Machine Learning**: Direct SQL integration with SageMaker and Comprehend.

### RDS & Aurora Security
- Encryption-at-rest via KMS. If master is unencrypted, read replicas cannot be encrypted.
- Encryption-in-transit (TLS/SSL).
- No SSH access (except on RDS Custom).
- **RDS Proxy**: Serverless database connection pooler. Reduces database stress and decreases failover times by up to 66%. Enforces IAM authentication.

### Amazon ElastiCache
In-memory database cache to reduce DB load.
- **Redis**: Multi-AZ with auto-failover, backup/restore, data structures. Supports IAM auth.
- **Memcached**: Multi-node sharding, multi-threaded, non-persistent, no backup/restore. Supports SASL auth.

---

## DNS: Route 53

Translates human-friendly hostnames to IP addresses.
- **CNAME**: Maps subdomain to subdomain (e.g., `app.example.com` to `something.elasticbeanstalk.com`). Cannot be used for root domains.
- **Alias**: AWS-specific record mapping root domains to AWS resources (e.g., load balancers, S3 buckets) for free.
- **Routing Policies**:
  - **Simple**: Returns one or multiple IP addresses randomly.
  - **Weighted**: Distributes traffic based on pre-defined percentage weights.
  - **Failover**: Routes to primary; falls back to secondary if primary health check fails.
  - **Latency-based**: Routes to the region with the lowest latency for the user.
  - **Geolocation**: Routes based on the user's geographic location.
  - **Geoproximity**: Routes based on geographic proximity with optional "bias" values.
  - **Multi-value Answer**: Returns up to 8 healthy records with health checks.

---

## Amazon S3

- **Durability**: 11 9's of durability (`99.999999999%`).
- **S3 Replication**: Requires versioning to be enabled on both source and destination buckets.
  - **CRR (Cross-Region)**: Lower latency access, compliance.
  - **SRR (Same-Region)**: Log aggregation, dev/prod sync.
- **Encryption**: Server-side encryption (SSE-S3 is default, or SSE-KMS), or client-side encryption.

---

## Migration, Hybrid Storage, and Data Sync

### AWS Snow Family
Physical data transfer devices.
- **Data Migration**: Move large datasets offline (Snowcone, Snowball Edge, Snowmobile).
- **Edge Computing**: Run EC2/Lambda on-site with no internet (Snowcone, Snowball Edge). Managed via **AWS OpsHub**.
- *Note*: Cannot import directly to Glacier. Import to S3 first, then transition via lifecycle policies.

### Amazon FSx
High-performance file systems.
- **FSx for Windows**: Native SMB and NTFS, Active Directory integration. Supports Multi-AZ.
- **FSx for Lustre**: High Performance Computing (HPC), machine learning, video processing. Sub-ms latencies. Reads/writes directly from/to S3.
- **FSx for NetApp ONTAP**: Multi-protocol (NFS, SMB, iSCSI), storage autoscaling, instantaneous cloning.
- **FSx for OpenZFS**: NFS only, migrated ZFS workloads to AWS.

### Storage Gateway
Hybrid cloud storage.
- **S3 File Gateway**: NFS/SMB access to S3 with local caching.
- **FSx File Gateway**: Low-latency local cache for FSx Windows File Server.
- **Volume Gateway**: Block storage via iSCSI. Stored volumes (all data local, backed up to S3) or Cached volumes (frequently accessed local, rest in S3).
- **Tape Gateway**: Virtual tape library (VTL) to back up tape workloads to S3/Glacier.

### AWS Transfer Family
Fully managed FTP, FTPS, and SFTP endpoints backed by S3 or EFS.

### AWS DataSync
Agent-based online data transfer to sync on-premises storage (NFS, SMB, HDFS) or other clouds to AWS S3, EFS, or FSx.

---

## Application Integration & Containers

### Amazon SQS (Simple Queue Service)
Decouples application tiers (producer/consumer model).
- Message retention: Default 4 days, max 14 days.
- **Visibility Timeout**: Default 30s. Time the message becomes invisible to other consumers. Can be changed via `ChangeMessageVisibility` API.
- **Long Polling**: Wait up to 20 seconds for messages to arrive, reducing API requests and cost.

### Amazon SNS (Simple Notification Service)
Pub/Sub messaging model. Subscribers receive notifications immediately (HTTP, Email, SQS, Lambda).

### Amazon Kinesis
- **Kinesis Data Streams**: Low-latency ingestion of real-time streaming data at scale.
- **Kinesis Firehose**: Loads streaming data into S3, Redshift, Elasticsearch, or Splunk (supports minor transformations).
- **Kinesis Data Analytics**: Run SQL queries on streaming data.

### Containers
- **ECS (Elastic Container Service)**:
  - **EC2 Launch Type**: You manage the underlying EC2 instances.
  - **Fargate Launch Type**: Serverless container orchestration. No EC2 instances to manage.
- **EKS (Elastic Kubernetes Service)**: Managed Kubernetes.

---

## AWS Lambda

Serverless function executions.
- **Limits**:
  - Memory: 128MB to 10GB.
  - vCPU scales proportionally with memory.
  - Max Execution Time: 900 seconds (15 minutes).
  - Environment variables: Max 4KB.
  - Ephemeral `/tmp` storage: Up to 10GB.
  - Concurrency: Default 1000 concurrent executions.
  - Deployment size: 50MB (zipped), 250MB (unzipped).
- **SnapStart**: Free performance booster for Java 11+ that reduces cold starts by taking a VM state snapshot.
- **Lambda@Edge / CloudFront Functions**:
  - **CloudFront Functions**: Lightweight JS scripts, runs at edge, handles millions of requests/sec (viewer request/response only).
  - **Lambda@Edge**: Node.js/Python, handles thousands of requests/sec, runs at edge (can modify origin request/response).
- *VPC Access*: Lambda runs in a secure VPC by default. To access internal resources (like RDS), configure the Lambda to connect to your private VPC and route DB traffic through **RDS Proxy**.
