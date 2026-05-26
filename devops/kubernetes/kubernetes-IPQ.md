# Kubernetes Interview Preparation Questions (IPQ)

### Q: What are the key features of Kubernetes?
- **Automated Scheduling**: Kubernetes automatically schedules containers on host servers and handles their startup lifecycle, automating many manual deployment tasks.
- **Cluster Management**: Manages multiple clusters simultaneously.
- **Service Discovery & Load Balancing**: Provides container networking, load balancing, security policies, and storage configurations.
- **Self-Healing**: Continuously monitors the health of nodes and containers, automatically restarting failed containers and replacing nodes when necessary.
- **Horizontal & Vertical Scaling**: Allows quick and easy scaling of resources up or down, both vertically (resizing resources) and horizontally (scaling pod replicas).

### Q: What is a Pod in Kubernetes?
A Pod is the smallest, most fundamental deployable unit in Kubernetes. Rather than running containers directly, Kubernetes wraps one or more containers in a Pod. Containers in the same Pod share the same storage resources, network IP address, and port space (localhost). This allows containers in the same Pod to communicate with each other easily as if they were on the same host, while maintaining container-level isolation.

### Q: What is the role of the `kube-scheduler`?
The `kube-scheduler` is a control plane component that watches for newly created pods that have no assigned node, and selects the most optimal worker node for them to run on. Decisions are made based on resource requirements, constraints, policies, and node affinity/anti-affinity rules.

### Q: What are DaemonSets?
A DaemonSet ensures that all (or a subset of) worker nodes run a single copy of a specific Pod. As new nodes are added to the cluster, Pods are automatically scheduled on them. Typical use cases include running cluster-level storage daemons, network routing agents, or logging and monitoring agents on every host.

### Q: What is a Namespace in Kubernetes?
Namespaces are virtual partitions used to divide cluster resources among multiple users, teams, or environments (e.g., development, staging, production). They provide scopes for resource names and help manage access controls and resource quotas.

### Q: What is the Kubernetes Controller Manager?
The Controller Manager is a control plane daemon that embeds core control loops (controllers) regulating the state of the cluster. It runs multiple logical processes (such as Namespace creation, garbage collection, and node status checking) compiled together to run as a single process on the master node.

### Q: What are the primary types of Controller Managers?
Primary controllers running inside the controller manager include:
- **Endpoints Controller**: Populates Endpoint objects (joining Services to Pods).
- **Service Accounts Controller**: Creates default service accounts for new Namespaces.
- **Namespace Controller**: Handles deletion and cleanup of resources within deleted Namespaces.
- **Node Controller**: Monitors node health and status.
- **Replication Controller**: Ensures the correct number of replicas run for every replication specification.
- **Token Controller**: Creates API access tokens for service accounts.

### Q: What is `etcd`?
`etcd` is a highly available, distributed key-value store used as Kubernetes' backing store for all cluster data (such as configuration, metadata, and state info). It serves as the single source of truth for the entire cluster.

### Q: What is a ClusterIP service?
ClusterIP is the default Kubernetes service type. It exposes the service on an internal IP address within the cluster, making it accessible only by other resources inside the cluster, with no external access.

### Q: What is a NodePort service?
A NodePort service exposes the service on a static port (typically between 30000–32767) on each node's IP. This allows external traffic to reach the service directly by sending requests to `<NodeIP>:<NodePort>`.

### Q: What is a LoadBalancer service in Kubernetes?
A LoadBalancer service exposes the service externally by provisioning a cloud provider's network load balancer (e.g., AWS NLB/ALB, Google Cloud Load Balancer). The external load balancer routes traffic directly to the service's NodePort or Pods.

### Q: What is an Ingress, and how does it work?
An Ingress is an API object that manages external access to services in a cluster, typically handling HTTP and HTTPS traffic. It provides features like SSL termination, name-based virtual hosting, and URL path-based routing.

It works by using an **Ingress Controller** (such as NGINX Ingress) which monitors Ingress resources and translates the defined routing rules into load balancer configuration (like reverse proxy rules). This allows routing traffic to multiple services without creating individual load balancers for each service.
