# Ansible Nginx Setup

This directory contains Ansible playbooks and roles for automating the installation and configuration of Nginx web servers with customizable settings.

## Overview

The Ansible Nginx Setup provides a complete solution for deploying and configuring Nginx web servers across your infrastructure. It includes:

- A modular Ansible role structure for Nginx configuration
- Customizable port configuration (default: 6789)
- Custom HTML content deployment
- Template-based configuration management
- CI/CD integration with Jenkins pipeline

## Directory Structure

```
4.Ansible_Nginx_Setup/
├── inventory.yml                # Inventory file defining target servers
├── JenkinsFile                  # CI/CD pipeline for automated deployment
├── playbook-Nginx.yml           # Ansible playbook with inline tasks
└── nginx/                       # Nginx role directory
    ├── defaults/                # Default variables
    ├── files/                   # Static files (HTML content)
    ├── handlers/                # Handlers for service restarts
    ├── meta/                    # Role metadata
    ├── tasks/                   # Task definitions
    ├── templates/               # Jinja2 templates
    ├── tests/                   # Role tests
    └── vars/                    # Role variables
```

## Components

### Playbooks

**playbook-Nginx.yml**: Implements Nginx setup using inline tasks:
- Installs Nginx package
- Configures custom port (6789)
- Deploys custom HTML content
- Personalizes content with variable substitution

### Jenkins Pipeline

The `JenkinsFile` provides CI/CD automation for Nginx deployment:

- **Parameters**: Customizable name for HTML personalization
- **Stages**:
  1. **Checkout**: Retrieves the latest code from GitHub
  2. **Prepare SSH**: Sets up SSH authentication for Ansible
  3. **Run Ansible Playbook**: Executes the Ansible playbook with parameters

### Role Structure

The `nginx` role follows Ansible best practices with a complete directory structure:

- **tasks/main.yml**: Core tasks for installing and configuring Nginx
- **templates/nginx.conf.j2**: Jinja2 template for Nginx configuration
- **handlers/main.yml**: Service restart handlers
- **vars/main.yml**: Variable definitions (e.g., `nginx_port: 6789`)

## Usage

### Prerequisites

- Ansible 2.9 or higher
- Jenkins with Ansible plugin
- SSH access to target servers
- Sudo privileges on target servers

### Manual Deployment

1. Update the inventory file with your target servers
2. Run the playbook:
   ```bash
   ansible-playbook -i inventory.yml --extra-vars "name=YourName" playbook-Nginx.yml
   ```

### Automated Deployment with Jenkins

1. Configure Jenkins with the necessary credentials
2. Create a new pipeline job using the provided JenkinsFile
3. Run the pipeline with optional parameters:
   - **NAME**: The name to display in the personalized greeting

### Customization

1. **Changing the Nginx Port**:
   Edit `nginx/vars/main.yml` to change the default port (6789)

2. **Custom HTML Content**:
   Replace the content in the playbook's HTML template

## Security Considerations

- The default configuration only opens the custom port (6789)
- Standard ports 80 and 443 are explicitly removed from the configuration
- SSH keys are properly secured with appropriate permissions in the pipeline

## CI/CD Integration

The Jenkins pipeline automates the entire deployment process:

1. Retrieves the latest code from version control
2. Securely sets up SSH authentication
3. Executes the Ansible playbook with parameterized values
4. Can be extended with additional stages for testing and validation
