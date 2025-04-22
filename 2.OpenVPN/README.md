# OpenVPN Ansible Automation

This directory contains Ansible playbooks and configuration files for automating the deployment and management of OpenVPN servers.

## Overview

The OpenVPN automation suite provides a complete solution for:

- Installing and configuring OpenVPN servers
- Managing server and client certificates using Easy-RSA
- Configuring firewall rules for secure VPN access
- Generating and distributing client configuration files
- Revoking client access when needed

## Directory Structure

```
2.OpenVPN/
├── clientlist                # List of clients to generate certificates for
├── defaults/                 # Default variable values
│   └── main.yml              # Main configuration variables
├── files/                    # Static files for deployment
├── handlers/                 # Service handlers
├── inventory.yml             # Target server inventory
├── meta/                     # Role metadata
├── playbook.yml              # Main deployment playbook
├── revokelist                # List of clients to revoke access
├── tasks/                    # Implementation tasks
│   ├── client_keys.yaml      # Client certificate generation
│   ├── config.yaml           # OpenVPN server configuration
│   ├── easy-rsa.yaml         # Easy-RSA setup
│   ├── firewall.yaml         # Firewall configuration
│   ├── install.yaml          # OpenVPN installation
│   ├── main.yaml             # Main task orchestration
│   ├── revoke.yaml           # Certificate revocation
│   └── server_keys.yaml      # Server certificate generation
└── templates/                # Configuration templates
    └── server.conf.j2        # OpenVPN server configuration template
```

## Configuration

The deployment is configured through variables in `defaults/main.yml`:

- **server_name**: Name of the OpenVPN server (default: "server")
- **PROTOCOL**: Protocol for OpenVPN (default: "udp")
- **PORT**: Port for OpenVPN (default: "1194")
- **openvpn_server_network**: VPN network range (default: "10.8.0.0")
- **dns_resolver**: DNS server for clients (default: 8.8.4.4)

## Prerequisites

- Ansible 2.9 or higher
- Target server with:
  - SSH access
  - Sudo privileges
  - Supported OS: Ubuntu or CentOS/RHEL

## Usage

### Basic Deployment

1. Update the inventory file with your target server:
   ```yaml
   all:
     hosts:
       server1:
         ansible_host: your-server-ip
         ansible_user: your-ssh-user
   ```

2. Deploy OpenVPN:
   ```bash
   ansible-playbook -i inventory.yml playbook.yml
   ```

### Managing Client Certificates

#### Creating Client Certificates

1. Add client names to the `clientlist` file (one per line)
2. Run the playbook with the client generation tag:
   ```bash
   ansible-playbook -i inventory.yml playbook.yml --tags generate_client_keys
   ```

#### Revoking Client Access

1. Add client names to the `revokelist` file (one per line)
2. Run the playbook with the revocation tag:
   ```bash
   ansible-playbook -i inventory.yml playbook.yml --tags revoke_client_keys
   ```

## Client Configuration

After generating client certificates, the client configuration files (.ovpn) will be available in the `/etc/openvpn/client-configs/files/` directory on the OpenVPN server.

To connect a client:
1. Transfer the appropriate .ovpn file to the client device
2. Import the configuration into the OpenVPN client software
3. Connect to the VPN

## Security Features

- Strong encryption (AES-256-CBC)
- SHA512 authentication
- TLS authentication
- Certificate-based authentication
- Firewall rules to protect the VPN server
- Certificate revocation list (CRL) enforcement

## Firewall Configuration

The playbook configures the firewall to:
- Allow OpenVPN traffic on the configured port and protocol
- Allow established and related connections
- Optionally allow specific ports (80, 443 by default)
- Block all other incoming connections if configured

## Monitoring

The OpenVPN server includes a management interface on 127.0.0.1:5555 that can be used for monitoring and management.

## Troubleshooting

### Common Issues

1. **Connection Issues**:
   - Check firewall settings: Ensure the OpenVPN port (default: 1194/UDP) is open
   - Verify server certificates: Check that all certificates are properly generated
   - Enable verbose logging: Set `verb 6` in the server configuration for detailed logs

2. **Certificate Problems**:
   - Check the Easy-RSA setup: Ensure the PKI is properly initialized
   - Verify certificate paths: Make sure all paths in the server configuration are correct

### Log Locations

- OpenVPN logs: `/var/log/openvpn.log` or via systemd: `journalctl -u openvpn@server`
- Status file: `/etc/openvpn/openvpn-status.log`

## Customization

To customize the OpenVPN configuration:

1. Modify variables in `defaults/main.yml`
2. Edit the server configuration template in `templates/server.conf.j2`
3. Update firewall rules in `tasks/firewall.yaml`
