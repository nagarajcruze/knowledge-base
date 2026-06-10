# AWS Certified Solutions Architect - Associate Study Guide

### High Availability, Fault Tolerance, and Disaster Recovery
- **High Availability**: The system is designed to remain operational and accessible with minimal down-time, usually implemented via redundancy.
- **Fault Tolerance**: The system can survive the total failure of individual components (like a server or disk) without interrupting service.
- **Disaster Recovery**: Processes and policies to restore operations quickly in the event of a major outage or natural disaster (defined by RTO—Recovery Time Objective, and RPO—Recovery Point Objective).
- **Scalability vs. Elasticity**:
  - *Scalability*: The ability of a system to handle larger workloads by adding resources (scaling up/vertically or scaling out/horizontally).
  - *Elasticity*: The ability to automatically scale resources in or out dynamically based on real-time demand.

### AWS Global Infrastructure
AWS infrastructure is organized globally into:
- **Regions**: Geographical locations containing multiple isolated and redundant Availability Zones.
- **Availability Zones (AZs)**: One or more discrete data centers with redundant power, networking, and connectivity in an AWS Region. All AZs are interconnected with high-bandwidth, low-latency networking.
- **Edge Locations (Points of Presence)**: Worldwide caching locations used by Amazon CloudFront to deliver content to end-users with low latency.

---

## 2. Identity and Access Management (IAM)

> [!IMPORTANT]
> **Principle of Least Privilege (PoLP)**: Users and services should be granted only the minimum permissions required to perform their tasks.

- **Global Scope**: IAM is a global service; users, groups, roles, and policies apply globally across all AWS regions.
- **Users vs. Groups**: Users can belong to multiple groups. **Groups cannot contain other groups** (no nested groups).
- **Policies**: JSON documents defining permissions. Always attach policies to **Groups** or **Roles**, not directly to individual users.

### IAM Policy JSON Structure
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket-name",
        "arn:aws:s3:::my-bucket-name/*"
      ]
    }
  ]
}
```
- **Version**: Version of the policy language.
- **Effect**: `Allow` or `Deny`.
- **Action**: List of actions (e.g., `s3:GetObject`).
- **Resource**: The AWS resource the policy applies to (using Amazon Resource Names - ARNs).

### IAM Roles
Roles are assumed by trusted identities (such as AWS services like EC2, Lambda, or external users) to acquire temporary security credentials. Unlike users, roles do not have credentials (passwords or access keys) attached to them.

### Auditing & Security Analysis Tools
- **Credential Report**: Generates a CSV file listing all users, password age, access key usage, and MFA status.
- **IAM Access Advisor**: Shows which services a user or role has access to and when they last accessed them. Useful for trimming unused permissions.
- **IAM Access Analyzer**: Audits resource policies to find resources shared with external accounts.

---

## 3. Compute Services: EC2 & Lambda

### Amazon EC2 (Elastic Compute Cloud)
EC2 provides resizable virtual machine compute capacity.

#### Purchasing Options
- **On-Demand**: Pay by the second/hour. High flexibility, no long-term commitments. Ideal for short-term, unpredictable workloads.
- **Reserved Instances (RI)**: Commit to 1 or 3 years. Offers up to a 72% discount.
- **Spot Instances**: Bid on spare AWS capacity. Up to 90% discount. Can be terminated by AWS with a 2-minute warning. Ideal for fault-tolerant workloads (e.g., batch processing).
- **Dedicated Hosts**: Physical servers dedicated entirely to you. Required for compliance, virtualization licensing, or BYOL (Bring Your Own License).
- **Dedicated Instances**: Run on dedicated hardware, but can share hardware with other instances within your own AWS account.

#### Placement Groups
- **Cluster**: Packs instances close together inside a single AZ. Provides low-latency, high-throughput network performance.
- **Spread**: Places instances on separate physical hardware racks (maximum 7 per AZ) to reduce risk of simultaneous failures.
- **Partition**: Spreads instances across logical partitions. Instances in one partition do not share hardware with instances in other partitions.

#### EC2 Hibernate
Saves the contents of the instance RAM to the EBS boot volume (which must be encrypted). Booting up is significantly faster because the OS does not perform a cold boot; instead, it resumes from the saved RAM state.

#### Web Server Bootstrapping Example (User Data Script)
User Data scripts execute with root privileges once during the first boot of the EC2 instance.
```bash
#!/bin/bash
sudo apt-get update
sudo apt-get install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
echo "<h1>Hello! Your Nginx Virtual Machine is running.</h1>" > /var/www/html/index.html
```

---

### AWS Lambda
Lambda is a serverless, event-driven compute service that executes code without provisioning or managing servers. You pay only for the compute time you consume.

- **Limits**:
  - **Memory**: 128MB to 10GB.
  - **Maximum Execution Duration**: 900 seconds (15 minutes).
  - **Deployment Package Limits**: 50MB (zipped), 250MB (unzipped).
  - **Ephemeral `/tmp` space**: 128MB to 10GB.
- **Lambda SnapStart**: Reduces start-up latency (cold starts) for Java-based functions by taking a snapshot of the initialized execution environment.
- **VPC Access**: By default, Lambda runs in a secure service VPC. To connect to resources inside your private VPC (e.g., RDS DBs), configure the Lambda to attach to your private subnet. Use RDS Proxy to manage database connection pools.

---

## 4. Storage Solutions

AWS offers diverse storage services optimized for performance, access patterns, and durability.

### Block, File, and Object Storage Comparison

| Storage Service | Type | Scope | Access Protocol | Persistence |
|---|---|---|---|---|
| **EBS (Elastic Block Store)** | Block | Single AZ | Proprietary Network Block | Persistent (Network-attached) |
| **Instance Store** | Block | Local Host | Physical PCIe/NVMe Bus | Ephemeral (Lost on stop/termination) |
| **EFS (Elastic File System)** | File | Multi-AZ | NFSv4 (Linux only) | Persistent (Shared filesystem) |
| **Amazon S3** | Object | Global/Multi-AZ | REST API (HTTP/HTTPS) | Persistent (Object Store) |

### EBS Volume Types
- **General Purpose SSD (`gp2` / `gp3`)**: General compute workloads. `gp3` allows provisioning IOPS and throughput independently of volume size.
- **Provisioned IOPS SSD (`io1` / `io2`)**: For high-performance, I/O-intensive databases. Supports **Multi-Attach** (mounting a volume to up to 16 instances in the same AZ).
- **Throughput Optimized HDD (`st1`)**: Low-cost, sequential read/write throughput (e.g., MapReduce, log pipelines). Cannot be a boot volume.
- **Cold HDD (`sc1`)**: Lowest cost storage for infrequently accessed data. Cannot be a boot volume.

### Amazon S3 (Simple Storage Service)
S3 is a highly durable object storage service offering $99.999999999\%$ (11 nines) durability.
- **S3 Versioning**: Keeps multiple copies of an object. Required to set up Replication.
- **S3 Replication**:
  - **Cross-Region Replication (CRR)**: Replicates objects across different AWS Regions.
  - **Same-Region Replication (SRR)**: Replicates objects within the same Region.
- **S3 Encryption**: Server-side encryption using S3-managed keys (SSE-S3), KMS keys (SSE-KMS), or client-side encryption.

### Hybrid Storage & Migration Services
- **AWS Snow Family**: Physical storage devices sent to your location to migrate large datasets offline (Snowcone, Snowball Edge, Snowmobile) or run edge computing.
- **Amazon FSx**: Managed high-performance filesystems (FSx for Windows, FSx for Lustre for HPC, FSx for NetApp ONTAP).
- **AWS Storage Gateway**: Hybrid cloud storage connecting on-premises applications to S3, Glacier, or FSx. Includes File Gateways, Volume Gateways (Cached/Stored), and Tape Gateways.
- **AWS DataSync**: Online agent-based service to automate transferring data between local storage and AWS.

---

## 5. Networking & VPC

### Virtual Private Cloud (VPC)
A VPC is a logically isolated virtual network allocated to your AWS account.

```text
 ┌────────────────────────────────────────────────────────┐
 │ VPC (e.g., 10.0.0.0/16)                                │
 │  ┌────────────────────────┐  ┌──────────────────────┐  │
 │  │ Public Subnet (10.0.1.0/24) │  │ Private Subnet (10.0.2.0/24) │  │
 │  │  ┌──────────────┐      │  │  ┌────────────────┐  │  │
 │  │  │ EC2 (Web)    │ ───┐  │  │  │ EC2 (DB)       │  │  │
 │  │  └──────────────┘    │  │  │  └────────────────┘  │  │
 │  └──────────────────────┼─┘  └───────────▲──────────┘  │
 │                         ▼                │             │
 │                 [Internet Gateway]   [NAT Gateway]     │
 └─────────────────────────┬────────────────┼─────────────┘
                           ▼                │
                    [Public Internet] ──────┘
```

### CIDR Notation & Subnet Math
CIDR (Classless Inter-Domain Routing) defines network block allocations:
- Format: `192.168.1.0/24`. The number after the slash `/X` represents the bits allocated to the network prefix.
- The remaining bits ($32 - X$) define the number of host IP addresses available: $2^{(32 - X)}$.
- **AWS reserves 5 IP addresses** in every subnet (the first 4 and the last 1):
  - `.0`: Network address.
  - `.1`: VPC router.
  - `.2`: Amazon Provided DNS.
  - `.3`: Reserved for future use.
  - `.255`: Network broadcast address.
- **Example Subnets**:
  - `/24` Subnet: $2^8 = 256$ IPs. Usable: $256 - 5 = 251$ addresses.
  - `/26` Subnet: $2^6 = 64$ IPs. Usable: $64 - 5 = 59$ addresses.
  - `/32` Subnet: Represents a single unique IP address.

### Network Components
- **Subnets**: Public subnets have a direct route to an Internet Gateway (IGW). Private subnets route outbound internet traffic through a **NAT Gateway** placed in a public subnet.
- **Security Groups vs. Network Access Control Lists (NACLs)**:
  - **Security Group**: Operating system/instance-level firewall. **Stateful** (return traffic is automatically allowed).
  - **NACL**: Subnet-level firewall. **Stateless** (both inbound and outbound traffic must be explicitly allowed).
- **Ephemeral Ports**: Outbound responses are sent back to clients via random, high-numbered ports (ranges `32768–61000` on Linux, `1024-65535` on Windows). NACLs must permit traffic on these ports for successful communication.
- **Bastion Host**: A jump box EC2 instance residing in a public subnet used to securely SSH/RDP into private instances.
- **VPC Peering**: Connects two VPCs using private AWS routing. Does not support transitive peering (if VPC A peers with B, and B with C, A does not peer with C). CIDR ranges cannot overlap.
- **VPC Endpoints**: Securely connect your VPC to AWS services privately without traversing the public internet.
  - *Interface Endpoints*: Private IPs using ENIs (costs apply).
  - *Gateway Endpoints*: Free, routes traffic directly. Supports **S3** and **DynamoDB** only.
- **VPC Flow Logs**: Captures IP traffic metadata going in/out of network interfaces, subnets, or VPCs, and publishes them to CloudWatch Logs or S3.
- **Transit Gateway**: A hub-and-spoke transit hub to simplify complex peering networks across multiple VPCs and on-premises networks.
- **Egress-Only Internet Gateway**: Provides egress-only (outbound-only) internet access for IPv6 addresses from a private subnet, preventing inbound connections.

---

## 6. Load Balancing & Auto Scaling

### Elastic Load Balancing (ELB)
ELBs distribute incoming application traffic across target groups (e.g., EC2 instances, containers, or Lambda functions).

- **Application Load Balancer (ALB)**: Operates at Layer 7 (HTTP/HTTPS). Supports advanced path, host, query-string routing, and sticky sessions.
- **Network Load Balancer (NLB)**: Operates at Layer 4 (TCP/UDP/TLS). Ultra-high performance, capable of handling millions of requests per second. Supports assigning static/Elastic IPs.
- **Gateway Load Balancer (GWLB)**: Operates at Layer 3 (IP). Used to scale and manage virtual security/firewall appliances.
- **Server Name Indication (SNI)**: Allows multiple SSL certificates to be hosted on a single listener, routing users based on the hostname requested.

### Auto Scaling Groups (ASG)
ASG manages EC2 instances dynamically to maintain application availability.
- **Dynamic Scaling**: Scales based on metrics (e.g., Average CPU > 70%).
- **Scheduled Scaling**: Scales based on predictable calendar events.
- **Predictive Scaling**: Uses machine learning to forecast traffic patterns and launch instances ahead of demand spikes.

---

## 7. Databases & Caching

- **Amazon RDS**: Managed SQL databases (PostgreSQL, MySQL, MariaDB, Oracle, SQL Server).
  - *Read Replicas*: Asynchronous replication used to scale read workloads.
  - *Multi-AZ*: Synchronous replication to a standby instance in another AZ for automatic disaster recovery failover.
- **Amazon Aurora**: Cloud-optimized relational database. Replicates 6 copies of data across 3 AZs.
  - *Aurora Serverless*: Auto-scaling, on-demand relational database configuration.
  - *Aurora Global Database*: Replicates data with latency under 1 second to up to 5 secondary read regions worldwide.
- **RDS Proxy**: Pool database connections to prevent compute resource exhaustion. Reduces database failover times and enforces IAM auth.
- **Amazon ElastiCache**: In-memory caching services.
  - *Redis*: Supports complex data structures, multi-AZ, clustering, and persistence.
  - *Memcached*: Simple, high-speed multi-threaded cache with no persistence or replication.

---

## 8. Integration & Containers

- **Amazon SQS (Simple Queue Service)**: Fully managed message queues.
  - Decouples microservices using a pull-based model.
  - *Visibility Timeout*: The duration a message is hidden from other consumers while being processed.
  - *Long Polling*: Waits up to 20 seconds for messages to arrive, reducing API calls and empty responses.
- **Amazon SNS (Simple Notification Service)**: Pub/Sub messaging system pushing notifications instantly to subscribers (SQS, email, mobile).
- **Amazon Kinesis**: Real-time streaming data ingestion and analytics.
- **AWS ECS (Elastic Container Service)**:
  - *EC2 Launch Type*: You manage the cluster instances.
  - *Fargate Launch Type*: Fully serverless container execution.
- **AWS EKS**: Managed Kubernetes service.

---

## 9. AWS CLI Cheat Sheet

Configure credentials:
```bash
aws configure
```

### S3 Operations
- **List Buckets**:
  ```bash
  aws s3 ls
  ```
- **Copy File to S3**:
  ```bash
  aws s3 cp localfile.txt s3://my-bucket/path/
  ```
- **Download File from S3**:
  ```bash
  aws s3 cp s3://my-bucket/path/localfile.txt .
  ```
- **Sync Directories**:
  ```bash
  aws s3 sync ./my-local-dir s3://my-bucket/
  ```

### EC2 Operations
- **Describe Instances**:
  ```bash
  aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
  ```
- **Start Instance**:
  ```bash
  aws ec2 start-instances --instance-ids i-0123456789abcdef0
  ```

### IAM Operations
- **List Users**:
  ```bash
  aws iam list-users
  ```
- **Create IAM User**:
  ```bash
  aws iam create-user --user-name john-developer
  ```
- **Attach Policy to User**:
  ```bash
  aws iam attach-user-policy --user-name john-developer --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
  ```

---

## 10. Security Best Practices

### Identity & Access Management (IAM)
- Enforce Multi-Factor Authentication (MFA) on the root account and all user accounts.
- Transition from static user Access Keys to temporary security credentials using IAM Roles.
- Implement least-privilege resource access policies.

### Network Security
- Restrict Security Group access to specific IP CIDR blocks instead of using `0.0.0.0/0` (especially for SSH/RDP).
- Place database servers and backend engines in private subnets.
- Inspect and filter web traffic with AWS WAF.

### Data Protection
- Enable AWS KMS encryption for all S3 buckets, RDS databases, and EBS volumes.
- Use HTTPS listener configurations to enforce encryption-in-transit.
- Configure S3 Object Lock or S3 Versioning to prevent ransomware deletions.
- Never hardcode credentials; fetch them dynamically from AWS Secrets Manager.