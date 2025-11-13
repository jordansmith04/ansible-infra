# Ansible Infrastructure Automation for EC2

This repository contains an Ansible project designed to automate the deployment of a simple containerized web application onto an Ubuntu EC2 instance, along with essential infrastructure monitoring.

The deployment process has been modularized into distinct Ansible roles for better reusability and clarity.

## Project Workflow

The main playbook, install_app.yml, executes the roles sequentially to transform a clean EC2 instance into a fully operational and monitored deployment target.

- users: Creates user and keys for access on instance.

- docker_setup: Prepares the host machine.

- cloudwatch_monitor: Configures host-level monitoring.

- app_deploy: Builds and runs the application container.

## Ansible Roles Overview

### 1. user setup

Purpose: Provision accesses

1. Create user accounts and add to groups

2. Add authorization (ssh) keys for the user

### 2. docker_setup

Purpose: Basic setup

1. Installs the latest Docker Engine (CE) and required packages on the Ubuntu host.

2. Adds the deployment user to the docker group to enable running container commands without sudo.

3. Ensures the Docker service is running and enabled at boot.

### 3. cloudwatch_monitor

Purpose: Observability

Prerequisite: The target EC2 instance must have an IAM role attached with the CloudWatchAgentServerPolicy.

1. Downloads and installs the Amazon CloudWatch Agent.

2. Copies a baseline configuration file (cloudwatch_agent_config.json) to the host.

3. Starts the agent, enabling system metrics collection (CPU, Memory, Disk) under the custom namespace EC2/Ansible-App.

### 4. app_deploy

Purpose: Application Delivery

1. Copies the Dockerfile and static content (index.html) to the remote host.

2. Uses the community.docker collection to build the application image (static_web_app:latest).

3. Stops and removes any previous container instances.

4. Runs the new container, mapping the host's port 80 to the container's port 80, and configures it to restart automatically.

### Prerequisites

Before running the playbook, ensure you have:

- Ansible Installed (version 2.9+) on your control machine.

- AWS EC2 Instance: A running Ubuntu EC2 instance (e.g., Ubuntu 20.04 or 22.04).

- IAM Role: The EC2 instance must have an associated IAM Role with the CloudWatchAgentServerPolicy attached.

- Inventory File (hosts): Updated with the IP or hostname of your target EC2 instance and correct SSH configuration.

- Ensure the Docker collection is installed:
```
ansible-galaxy collection install community.docker
```

### Usage

Encrypt vault secrets:
```
ansible-vault encrypt vars/user_secrets.yml
```

Generate a vault password_hash using:
```
ansible-vault create_hash --vault-password-file ~/.ansible-vault-pass.
```

Execute the main playbook from the root directory of the project:
```
ansible-playbook -i hosts -vault-password-file ~/.ansible-vault-pass ansible/playbooks/install_app.yml
```

Upon successful completion, your application will be running on the host's port 80, and the host's performance metrics will be streaming to the CloudWatch console.