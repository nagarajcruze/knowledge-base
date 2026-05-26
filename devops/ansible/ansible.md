# Ansible Notes

Ansible is agentless and uses YAML to define configuration and automation tasks.

## Key Concepts

- **Modules**: Units of code that perform specific tasks (e.g., managing services, packages, files).
- **Tasks**: Modules with specific arguments and parameters.
- **Playbooks**: Ordered lists of tasks.

## Requirements

- **Control Node**: Must run Linux (Windows is not supported for control nodes).
- **Managed Nodes**: Can be Linux or Windows.
- **Communication Protocol**: Speaks over SSH, SFTP, SCP, or WinRM (for Windows).
- **Control Node Python**: Python 3.8 or above.
- **Managed Node Python/PowerShell**:
  - Python 2.6 or Python 3.5 and later for Linux/Unix nodes.
  - PowerShell 3.0 or later and .NET 4.0 for Windows nodes.

> [!IMPORTANT]
> Run Ansible from a non-root account; otherwise, it cannot SSH to the root account on managed nodes.

---

## Setup & SSH Configuration

1. Install Python 3 and Pip:
   ```bash
   sudo apt update
   sudo apt install -y python3-pip
   pip install ansible
   ```

2. Generate an SSH key-pair on the control node:
   ```bash
   mkdir -p ~/.ssh
   ssh-keygen -t rsa -b 4096
   ```

3. Copy the SSH public key to the managed nodes:
   ```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub ansibleuser@managed-node-ip
   ```

4. Configure SSH (`~/.ssh/config`):
   ```ini
   Host 192.168.1.*
       User sysadmin
       IdentityFile ~/.ssh/ansible
   ```

---

## Configuration & Inventory

### Ansible Configuration (`ansible.cfg`)
Create an `ansible.cfg` file in your project directory to define default values:
```ini
[defaults]
inventory = ./inventory
remote_user = sysadmin

[ssh_connection]
ssh_args = -i ~/.ssh/ansible
```

### Inventory File (`inventory`)
An inventory file lists the nodes. Example entry:
```ini
node1 ansible_host=192.168.1.99 ansible_user=sysadmin ansible_ssh_private_key_file=~/.ssh/ansible
```
*(If `ansible.cfg` is set up with the default username and private key, only the hostname/IP is required).*

### Ad-Hoc Command Verification
Verify the configuration by pinging all managed nodes:
```bash
ansible all -m ping -i inventory
```

**Expected Output:**
```json
node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

List all connected nodes in YAML format:
```bash
ansible-inventory --list -y
```

Run a single command on a specific host:
```bash
ansible all -i hosts --limit host2 -a "/bin/echo hello"
```

---

## Privilege Escalation (Sudo)

### Option 1: Hardcoded become password (Not Recommended)
```yaml
- name: Example playbook with sudo password provided
  hosts: all
  become: yes
  become_user: root
  become_method: sudo
  become_pass: "your_sudo_password_here"
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
```

### Option 2: Passwordless sudo on managed nodes (using visudo)
Add the following line using `sudo visudo`:
```text
ansibleuser ALL=(ALL) NOPASSWD: ALL
```

### Option 3: Prompt for the password at runtime (Recommended)
Run playbooks with the `--ask-become-pass` flag:
```bash
ansible-playbook your_playbook.yml --ask-become-pass
```

---

## Playbook Examples

### Docker Installation Playbook
```yaml
---
- name: Install Docker on Ubuntu
  hosts: all
  become: yes
  become_user: root
  tasks:
    - name: Update package list
      apt:
        update_cache: yes

    - name: Install required packages
      apt:
        name:
          - ca-certificates
          - curl
          - gnupg
        state: present

    - name: Create directory for keyrings
      file:
        path: /etc/apt/keyrings
        state: directory
        mode: '0755'

    - name: Download and install Docker GPG key
      shell: |
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker-archive-keyring.gpg
        sudo chmod a+r /etc/apt/keyrings/docker-archive-keyring.gpg
      args:
        executable: /bin/bash

    - name: Add Docker repository to sources.list.d
      shell: |
        echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      args:
        executable: /bin/bash

    - name: Update package list again
      apt:
        update_cache: yes

    - name: Install Docker packages
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
        state: present

    - name: Install Docker plugins
      apt:
        name:
          - docker-buildx-plugin
          - docker-compose
        state: present

    - name: Print a message indicating installation success
      debug:
        msg: "Docker installed successfully."
```

### File Distribution Playbook
```yaml
- name: Download and extract CNI plugins
  get_url:
    url: https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz
    dest: /opt/cni/bin
  register: download_result
  remote_src: yes

- name: Copy downloaded file to other nodes
  copy:
    src: "/opt/cni/bin/cni-plugins-linux-amd64-v1.2.0.tgz"
    dest: "/opt/cni/bin/cni-plugins-linux-amd64-v1.2.0.tgz"
  loop: "{{ groups['other_nodes'] }}"
  when: download_result is succeeded
```
