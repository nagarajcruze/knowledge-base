# Ansible Interview Preparation Questions (IPQ)

### Q: What are Ansible Modules?
Ansible modules are units of code (like small programs) that Ansible pushes from a control machine to all the managed nodes or remote hosts. These modules are executed via playbooks to control system aspects like services, packages, and files. Once the tasks (such as installing updates or applying configurations) are completed, Ansible automatically removes the modules from the remote hosts. Ansible provides more than 450 built-in modules for everyday tasks.

### Q: What are Plugins in Ansible?
Plugins are extra pieces of code that augment or extend Ansible's core functionality. Ansible comes with several built-in plugins, but you can also write custom plugins. Some common types of plugins include action, cache, and callback plugins.

### Q: What are Inventories in Ansible?
An inventory is a simple text file listing all the hosts and nodes managed by the Ansible control machine, along with their details (such as IP addresses, databases, servers, groups, etc.). Once the inventory is registered, you can assign variables to any of the hosts. Inventories can be static (local files) or dynamic, which pull hosts from external sources like AWS EC2.

### Q: What are Playbooks in Ansible?
Ansible playbooks are YAML configuration files that act as instruction manuals orchestrating tasks. They are written in YAML (human-readable serialization language), which makes them easy to read and write without needing to know a complex programming language. 

A playbook is composed of one or more "plays." The goal of a play is to map a group of hosts to well-defined roles represented by tasks. Playbooks can orchestrate manually ordered steps, and execute tasks across different nodes synchronously or asynchronously.
