# DevOps Infrastructure Automation

A concise collection of automation scripts for essential DevOps infrastructure components.

## Repository Structure
```
.
├── 1.Vault-Jenkins/              # Vault integration with Jenkins
│   ├── JenkinsFile               # Jenkins pipeline for Vault integration
│   └── README.md                 # Documentation for Vault-Jenkins setup
│
├── 2.OpenVPN/                    # OpenVPN server automation
│   ├── defaults/                 # Default variables
│   ├── files/                    # Scripts and configuration files
│   ├── handlers/                 # Event handlers
│   ├── tasks/                    # Ansible tasks
│   ├── templates/                # Configuration templates
│   ├── playbook.yml              # Main OpenVPN playbook
│   └── README.md                 # OpenVPN documentation
│
├── 3.AWS-CLI/                    # AWS CLI integration
│   ├── files/                    # AWS scripts and service files
│   ├── create_service.yml        # Playbook to create systemd service
│   └── README.md                 # AWS CLI documentation
│
├── 4.Ansible_Nginx_Setup/        # Nginx deployment automation
│   ├── nginx/                    # Nginx role
│   ├── JenkinsFile               # CI/CD pipeline for automated deployment
│   ├── playbook-Nginx.yml        # Nginx deployment playbook
│   └── README.md                 # Nginx setup documentation
│
├── 5.Ansible_Artifactory/        # Artifactory deployment
│   ├── defaults/                 # Default variables
│   ├── handlers/                 # Event handlers
│   ├── tasks/                    # Ansible tasks
│   ├── templates/                # Configuration templates
│   ├── tests/                    # Test configurations
│   ├── playbook.yml              # Artifactory deployment playbook
│   └── README.md                 # Artifactory documentation
│
├── docs/                         # Documentation files
│   ├── infra.dot                 # Infrastructure diagram (DOT)
│   └── infra.svg                 # Infrastructure diagram (SVG)
│
├── docker-compose.yaml           # Core services configuration
├── inventory.yml                 # Main Ansible inventory
├── playbook.yml                  # Main playbook
└── README.md                     # This file
```

## Completed Tasks

### 1. Vault-Jenkins Integration
- HashiCorp Vault for secrets management
- Jenkins pipeline with AWS credential retrieval from Vault
- Secure CI/CD credential handling

### 2. OpenVPN Server Automation
- Ansible playbooks for OpenVPN deployment
- Client certificate management
- Secure network configuration

### 3. AWS Route53 Dynamic DNS
- Automatic DNS updates with AWS Route53
- EC2 instance metadata integration
- Dynamic hostname registration

### 4. Nginx Deployment with Ansible
- Automated Nginx installation and configuration
- Custom port configuration (6789)
- Dynamic content management

### 5. Artifactory Setup
- JFrog Artifactory OSS deployment
- Repository structure configuration
- CI/CD pipeline integration

## Setup Requirements
- Ansible 2.9+
- Docker and Docker Compose
- AWS CLI with proper permissions
- Python 3.x

## Docker Services
- Jenkins: CI/CD pipeline (port 8080)
- Nexus: Artifact repository (ports 8081-8083)
- Vault: Secrets management (ports 8200-8201)
