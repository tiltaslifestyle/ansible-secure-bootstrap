# Secure Server Bootstrap with Ansible

![Ansible](https://img.shields.io/badge/ansible-%231A1918.svg?style=for-the-badge&logo=ansible&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

Production-ready automation framework for deploying hardened Linux servers with security-first configuration, containerization, and zero-touch provisioning.
The playbook prepares a clean Ubuntu host for production use by applying security best practices, creating a dedicated automation user, hardening SSH access, enabling firewall protection, configuring Fail2Ban, and installing Docker as a container runtime.

## Architecture Overview

```
Operator / CI
   │
   │ SSH (Key-based authentication)
   ▼
[ Target Server (Ubuntu) ]
   │
   ├── Automation User (sudo, key-only access)
   │
   ├── SSH Hardening
   │   ├── Root login disabled
   │   ├── Password authentication disabled
   │   └── Strict user allowlist
   │
   ├── UFW Firewall
   │   ├── Default deny incoming
   │   └── Allow SSH only
   │
   ├── Fail2Ban
   │   └── Protects SSH from brute-force attacks
   │
   ├── System Baseline
   │   ├── Timezone (UTC)
   │   ├── Chrony (time sync)
   │   ├── Python Runtime 
   │   └── Common CLI utilities
   │
   └── Docker Engine
       ├── docker-ce
       ├── docker-compose-plugin
       └── containerd
```

## Key Features
**Security First**
- SSH key-only authentication
- Root login disabled
- Strict SSH user allowlist (`AllowUsers`)
- Hardened SSH configuration validated before reload
- UFW firewall with default deny policy
- Fail2Ban protecting SSH against brute-force attacks

**Automation User Model**
- Dedicated `automation_user` created by Ansible
- Passwordless sudo via `/etc/sudoers.d`
- SSH public key validated locally before deployment (fail-fast)
- User automatically added to docker group

**Modular & Idempotent Design**
- Fully role-based architecture:
    - *common* — base system configuration
    - *security* — hardening (users, SSH, firewall, Fail2Ban)
    - *docker* — container runtime setup
- Safe to re-run multiple times (idempotent)
- Clear separation of concerns

**Docker-Ready**
- Official Docker APT repository
- Secure keyring-based GPG setup (no deprecated apt-key)
- Architecture-aware installation (amd64 / arm64)
- Docker service enabled and started

**CI/CD Friendly**
- SSH keys injected via environment variables
- No secrets stored in repository
- Clear failure messages for missing prerequisites

## Tech Stack
- **Configuration Management:** Ansible
- **Operating System:** Ubuntu (tested on 20.04 / 22.04)
- **Security:** OpenSSH, UFW, Fail2Ban
- **Container Runtime:** Docker CE
- **Time Sync:** Chrony

## Prerequisites
- Ansible 2.14+
- Ubuntu-based target host
- SSH access to the target host
- Local SSH key pair

*Generate an SSH key if you don’t have one:*
``` bash
ssh-keygen -t ed25519
```

## Configuration

The project supports dynamic SSH key configuration via environment variables:
```bash
export ANSIBLE_SSH_KEY=~/.ssh/id_ed25519
export ANSIBLE_PUB_KEY=~/.ssh/id_ed25519.pub
```
*If not set, defaults are used automatically.*

## Installation & Usage
1. **Clone the repository:**
```bash
git clone https://github.com/tiltaslifestyle/ansible-secure-bootstrap.git
cd ansible-secure-bootstrap
```

2. **Configure inventory:**
Edit `inventory/hosts` and select the target group (`staging`, `prod`, or `all`).

3. **Run the bootstrap playbook:**
```bash
ansible-playbook playbooks/bootstrap.yml
```

## Disclaimer
This project intentionally applies **strict SSH hardening**.
Ensure that you:
- Have SSH key access as `automation_user`
- Understand the impact of `AllowUsers` and firewall rules before applying to production systems

## Project Structure
```
.
├── inventory
│   ├── hosts
│   └── group_vars
│       └── all.yml
│
├── playbooks
│   └── bootstrap.yml
│
├── roles
│   ├── common
│   │   └── tasks/main.yml
│   │
│   ├── security
│   │   ├── tasks
│   │   │   ├── main.yml
│   │   │   ├── users.yml
│   │   │   ├── ssh.yml
│   │   │   ├── firewall.yml
│   │   │   └── fail2ban.yml
│   │   ├── handlers/main.yml
│   │   └── templates
│   │       ├── sshd_config.j2
│   │       └── jail.local.j2
│   │
│   └── docker
│       └── tasks/main.yml
│
├── README.md
├── ansible.cfg
└── .gitignore
```