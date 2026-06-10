# Ansible: Infrastructure Automation & Configuration Management

## 1. Introduction & Push Architecture

Ansible is an open-source automation tool used for IT configuration management, application deployment, and task orchestration. 

### Key Characteristics
- **Agentless**: Unlike other configuration systems (like Chef or Puppet) that require a client daemon (agent) to run on every target machine, Ansible is agentless. It pushes commands directly to the targets.
- **SSH-Based**: Communication with managed nodes occurs over standard SSH (for Linux/Unix) or WinRM (for Windows), using existing network security protocols.
- **Push Architecture**: The Control Node connects to the Managed Nodes, pushes executable code (Modules), runs it on the target, and deletes the modules when finished.
- **Idempotency**: A core design feature of Ansible modules. If a playbook is run multiple times, Ansible checks the current state of the node and only applies changes if the actual state differs from the desired state defined in the task. If no change is needed, it skips the execution, preventing unintended side-effects.

### Core Terminology
- **Control Node**: The machine where Ansible is installed. It manages and runs commands/playbooks targeting managed nodes. (Note: Windows cannot be a Control Node; it must run Linux/Unix/MacOS).
- **Managed Nodes**: The target computers managed by Ansible.
- **Inventory**: A file containing a list of IP addresses, hostnames, and groupings of the Managed Nodes.
- **Modules**: Standalone scripts that do the actual work (e.g. `apt`, `yum`, `service`, `copy`).
- **Tasks**: An individual execution statement representing one action using a module.
- **Playbooks**: Ordered lists of tasks.

---

## 2. Setup & SSH Configuration

To run Ansible, the Control Node must have passwordless SSH access to the Managed Nodes.

### Step 1: Install Ansible on Control Node
```bash
sudo apt update
sudo apt install -y python3-pip
pip install ansible
```

### Step 2: Configure Passwordless SSH Connection
1. **Generate SSH Key-pair** on the Control Node:
   ```bash
   ssh-keygen -t ed25519 -C "ansible-key"
   ```
2. **Copy Public Key** to the Managed Node:
   ```bash
   ssh-copy-id -i ~/.ssh/id_ed25519.pub sysadmin@managed-node-ip
   ```
3. **Configure SSH Client Aliases** (optional, inside `~/.ssh/config`):
   ```text
   Host 192.168.1.*
       User sysadmin
       IdentityFile ~/.ssh/id_ed25519
   ```

---

## 3. Configuration & Inventory Management

Create a workspace structure:
```text
my-ansible-project/
├── ansible.cfg
├── inventory.ini
└── playbook.yml
```

### 1. The Configuration File (`ansible.cfg`)
Define default behaviors for Ansible in your project root folder:
```ini
[defaults]
# Path to default inventory file
inventory = ./inventory.ini

# Default remote SSH user to connect as
remote_user = sysadmin

# Skip SSH host key verification prompts (useful for large scale fresh VMs)
host_key_checking = False

[ssh_connection]
# Configure custom private key path
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -i ~/.ssh/id_ed25519
```

### 2. Inventory Formats
An inventory lists target hosts. You can write it in INI or YAML format.

#### INI Format (`inventory.ini`)
```ini
[webservers]
web1 ansible_host=192.168.1.101
web2 ansible_host=192.168.1.102

[dbservers]
db1 ansible_host=192.168.1.201

# Group of groups
[production:children]
webservers
dbservers
```

#### YAML Format (`inventory.yml`)
```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.101
        web2:
          ansible_host: 192.168.1.102
    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.201
```

---

## 4. Ad-Hoc Commands

Ad-hoc commands are quick, single-task operations executed from the command line without writing playbooks.

- **Ping all hosts in inventory**:
  ```bash
  ansible all -m ping
  ```
- **Execute shell command on webservers**:
  ```bash
  ansible webservers -a "free -m"
  ```
- **Install package (using apt module)**:
  ```bash
  ansible webservers -m apt -a "name=nginx state=present" --become
  ```
- **Restart a system service**:
  ```bash
  ansible webservers -m service -a "name=nginx state=restarted" --become
  ```

---

## 5. Privilege Escalation (`become`)

Managed nodes often require root/administrator privileges to execute tasks (like installing packages or creating users). Ansible handles this using the `become` mechanism.

- **`become: yes`**: Tells Ansible to escalate privileges (defaults to using `sudo`).
- **`become_user: root`**: The user identity to escalate to.

### Escalation Options

#### Option A: Passwordless Sudo (Best Practice)
Configure passwordless sudo access for the Ansible SSH user on the managed node using `sudo visudo`:
```text
sysadmin ALL=(ALL) NOPASSWD: ALL
```

#### Option B: Prompt for Password at Runtime (Recommended without passwordless sudo)
If passwordless sudo is disabled, force Ansible to prompt you for the sudo password when executing commands or playbooks:
- Run playbook: `ansible-playbook site.yml --ask-become-pass` (or `-K`)
- Run ad-hoc command: `ansible all -m ping -K`

#### Option C: Storing Password in Inventory (Security Risk - Not Recommended)
Avoid storing plain text passwords inside configurations.
```yaml
ansible_become_pass: "my-cleartext-sudo-password"
```

---

## 6. Playbook Examples

Playbooks are written in YAML and group one or more plays.

### 1. Docker Installation Playbook (Targeting Ubuntu/Debian)
```yaml
---
- name: Configure and Install Docker Engine
  hosts: webservers
  become: yes
  tasks:
    - name: Update apt software repository cache
      apt:
        update_cache: yes

    - name: Install required system packages
      apt:
        name:
          - ca-certificates
          - curl
          - gnupg
        state: present

    - name: Create apt keyrings directory
      file:
        path: /etc/apt/keyrings
        state: directory
        mode: '0755'

    - name: Download Docker official GPG key
      get_url:
        url: https://download.docker.com/linux/ubuntu/gpg
        dest: /etc/apt/keyrings/docker.asc
        mode: '0644'

    - name: Set up the Docker repository
      apt_repository:
        repo: "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu focal stable"
        state: present
        filename: docker

    - name: Update apt index and install Docker engine
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-buildx-plugin
          - docker-compose-plugin
        state: present
        update_cache: yes

    - name: Ensure Docker service is running and enabled
      service:
        name: docker
        state: started
        enabled: yes
```

### 2. File Download, Distribution, and Extraction Playbook
This playbook demonstrates downloading an archive to the control node, copying it to target nodes, and extracting it.

```yaml
---
- name: Download, Distribute, and Extract CNI Plugins
  hosts: all
  become: yes
  tasks:
    - name: Create local directory on control node for caching downloads
      delegate_to: localhost
      become: no
      file:
        path: /tmp/ansible-downloads
        state: directory
        mode: '0755'

    - name: Download CNI archive locally on the control node
      delegate_to: localhost
      become: no
      get_url:
        url: https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz
        dest: /tmp/ansible-downloads/cni-plugins.tgz

    - name: Ensure target directory exists on managed nodes
      file:
        path: /opt/cni/bin
        state: directory
        mode: '0755'

    - name: Copy and extract archive to the managed nodes
      unarchive:
        src: /tmp/ansible-downloads/cni-plugins.tgz
        dest: /opt/cni/bin/
        remote_src: no  # Tells Ansible the archive file is on the control node
```
- **`delegate_to: localhost`**: Directs Ansible to run this task locally on the Control Node itself, rather than attempting to connect via SSH to managed hosts.
- **`remote_src: no`**: (Under the `unarchive` module) Tells Ansible that the source archive file (`src`) resides on the Control Node filesystem, and needs to be uploaded to the target hosts before extraction.
