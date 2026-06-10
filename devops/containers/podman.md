# Podman & Quadlets: Modern Container Operations

## 1. Introduction to Podman

Podman (Pod Manager) is an open-source, Linux-native container engine designed to develop, manage, and run OCI (Open Container Initiative) containers. While it shares a near-identical CLI command set with Docker, its internal architecture is fundamentally different.

### Key Differences between Docker and Podman

| Feature | Docker | Podman |
|---|---|---|
| **Architecture** | Daemon-based (requires a central `dockerd` background process running as root). | Daemonless (runs containers directly as child processes of the user's shell). |
| **Privileges** | Standard deployments require root access (which can present security vulnerabilities). | Native support for **Rootless** containers (runs entirely in user-space without root). |
| **Pod Support** | No native pod concept. Requires Kubernetes or Docker Compose to group containers. | Native support for **Pods** (groups of containers sharing network/namespaces, matching Kubernetes concepts). |
| **Kubernetes Integration** | Requires external tools to export configuration to Kubernetes. | Can directly generate and execute Kubernetes YAML configuration files (`podman play kube`). |

---

## 2. Podman Command-Line Basics

Because Podman's CLI is OCI-compliant, it is designed as a drop-in replacement for Docker. You can even set up an alias in your shell configuration:
```bash
alias docker=podman
```

### Running Basic Containers
- **Run a container rootless**:
  ```bash
  podman run -d --name my-web -p 8080:80 alpine:latest
  ```
- **List containers**:
  ```bash
  podman ps -a
  ```

### Managing Native Pods
Pods are groups of one or more containers that share the same network, IP address, and port space.
1. **Create an empty Pod**:
   ```bash
   podman pod create --name my-pod -p 8080:80
   ```
2. **Add a container to the Pod**:
   ```bash
   podman run -d --pod my-pod --name web-app nginx:alpine
   ```
   *Since the container is in `my-pod`, it shares the port mappings (port 8080 on the host maps to port 80 inside the pod).*
3. **List running pods**:
   ```bash
   podman pod ps
   ```

---

## 3. Podman Compose & Kubernetes YAMLs

### 1. Podman Compose
For multi-container applications defined in `docker-compose.yml`, Podman can run them using the `podman-compose` utility:
- **Start compose services**:
  ```bash
  podman-compose up -d
  ```
- **Stop compose services**:
  ```bash
  podman-compose down
  ```

### 2. Native Kubernetes Configurations (`play kube`)
Instead of using compose files, Podman allows you to run containers using standard Kubernetes YAML declarations.
- **Generate Kubernetes YAML from a running pod**:
  ```bash
  podman generate kube my-pod > my-pod.yaml
  ```
- **Play/Deploy a Kubernetes YAML file directly**:
  ```bash
  podman play kube my-pod.yaml
  ```

---

## 4. Quadlets (Systemd Integration)

In production deployments, you need containers to start automatically when the host boots up. Traditionally, this was handled by systemd service files calling `podman run` commands. Podman introduced **Quadlets** to make this seamless.

Quadlets are systemd generators. You write a simple declarative configuration file (similar to a desktop entry or simplified systemd file), and Quadlet automatically generates a fully functional systemd service unit behind the scenes.

### Quadlet File Locations
- **System-wide (Root)**: `/etc/containers/systemd/`
- **User-space (Rootless)**: `~/.config/containers/systemd/`

### Example: `.container` File Definition
Create a file named `nginx-app.container` inside `~/.config/containers/systemd/`:

```ini
[Unit]
Description=My Nginx Quadlet Service
After=network-online.target

[Container]
Image=docker.io/library/nginx:alpine
PublishPort=8080:80
Volume=nginx-data:/usr/share/nginx/html:z

[Install]
# Pull in the service when the user logs in or system boots
WantedBy=default.target
```
*Note: The `:z` flag on the volume is critical on SELinux systems (like RHEL/Fedora) to configure the correct file security context.*

### Commands to Manage Quadlet Services
Once the configuration file is in place, manage it using standard `systemctl` commands:

1. **Reload systemd daemon** (triggers Quadlet to generate the systemd unit files):
   ```bash
   systemctl --user daemon-reload
   ```
2. **Start the container service**:
   ```bash
   systemctl --user start nginx-app
   ```
3. **Enable automatic start on boot**:
   ```bash
   systemctl --user enable nginx-app
   ```
4. **Inspect service logs via journald**:
   ```bash
   journalctl --user -u nginx-app -f
   ```

---

## 5. Advanced Quadlet Stack Management

When managing a multi-container application stack (such as the **Simnovator container stack** using Podman 5.4.1 and systemd Quadlets on Ubuntu 24.04/26.04), you must structure files, manage dependencies, handle health probes, and debug startup loops.

### A. Lifecycle & Architecture

Quadlets are not standalone background runtimes; they are a declarative configuration compiler integrated directly into the `podman` binary. They compile simple configurations into systemd services during system initialization.

- **The Blueprint**: Define your infrastructure using `.container`, `.network`, and `.pod` files inside `~/.config/containers/systemd/` (for user-space/rootless) or `/etc/containers/systemd/` (for system-wide/root).
- **The Master Anchor**: Only the `.pod` file requires an `[Install]` section. This designates the Pod as the "anchor" service.
- **The Linkage**:
  - Link containers to a Pod by specifying the directive `Pod=simnovator-pod` in the container configuration.
  - Systemd automatically generates `BindsTo=` and `After=` directives under the hood, creating a strict parent-child relationship where the Pod acts as the infrastructure root.

---

### B. Dependency Management: Wants, Requires, and After

To enforce container startup order and stability, use these systemd directives in the `[Unit]` section of your files:

- **`After=` (Ordering)**: Guarantees that Service B waits for Service A to be initialized before starting. During shutdown, this ordering is reversed automatically (Service B stops before Service A).
- **`Requires=` (Strict Dependency)**: Creates a hard dependency relationship. If Service A fails to start, Service B aborts immediately. If Service A crashes or is stopped during runtime, Service B is also stopped. Use this for mandatory database/infrastructure backends.
- **`Wants=` (Soft Dependency)**: A weaker dependency option. If the dependency fails to start, Service B continues running. Use this for optional secondary services (e.g. logging sidecars or metrics collectors).

---

### C. Health & Readiness (The Notify Workflow)

By default, systemd only checks if a container process ID (PID) exists. To make systemd "application-aware"—waiting for databases to become ready before launching API applications—integrate Podman's internal health checks with systemd's state machine.

- **`HealthCmd=`**: Defines the command to run inside the container to probe health (e.g., `pg_isready -U postgres`).
- **`Notify=healthy`**: Instructs Quadlet to inject `Type=notify` into the generated service file.

#### The Workflow:
1. Systemd launches the database container and marks the service status as `activating`.
2. Dependent containers (like your backend API) remain paused or waiting.
3. The `HealthCmd` executes inside the container. Once it succeeds, Podman sends an `sd_notify` signal to systemd.
4. Systemd changes the database service status from `activating` to `active`, releasing the block and triggering dependent containers to launch.

---

### D. Loop Prevention & Troubleshooting

When services fail continuously and enter a restart loop ("flapping"), the server experiences severe CPU and disk I/O exhaustion due to rapid namespace creation and log writing.

#### Preventing Loops
Tune your services with these parameters in the `[Service]` or `[Unit]` sections:
- **`RestartSec=10`**: Adds a 10-second delay between crashes, stopping rapid-fire restart loops.
- **`StartLimitBurst=5`**: If a service crashes 5 times within a 60-second window, systemd suspends the service in a failed state, stopping the loop.
- **`HealthOnFailure=stop`**: Converts a frozen or unhealthy application state into a hard exit, allowing systemd to trigger its restart and rate-limiting limits.

#### Inspecting Logs
- Standard `podman logs` might return empty outputs because Quadlet configures container output to pass directly through to `journald`.
- Use the journal logger as your source of truth:
  ```bash
  journalctl --user -u <service_name> -f
  ```
- Filter out noise to show only error logs using the priority flag `-p err`:
  ```bash
  journalctl --user -u <service_name> -p err --no-pager
  ```

---

### E. Deployment & Troubleshooting Cheat Sheet

#### Summary Checklist

| Action | Command |
|---|---|
| **Apply Configuration Changes** | `systemctl --user daemon-reload` |
| **Refresh/Restart Stack** | `systemctl --user restart simnovator-pod.service` |
| **Debug Specific Unit** | `systemctl --user status <unit_name>` |
| **View Errors Only** | `journalctl --user -u <unit_name> -p err --no-pager` |
| **Reset Failed State** | `systemctl --user reset-failed <unit_name>` |

#### Diagnostic Pro-Tip: "Patient Zero" Loops
If your entire container stack is stuck in a restart loop:
1. Find which services are in a failed state:
   ```bash
   systemctl --user list-units --type=service
   ```
2. Query the exact exit code and crash reason:
   ```bash
   systemctl --user status <failed_service_name>
   ```
3. Stop the crashing loop to stop journal noise and free up CPU:
   ```bash
   systemctl --user stop <affected_services>
   ```
4. Focus debugging solely on the root dependency (the database or worker queue) that failed first.