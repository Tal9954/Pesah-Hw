# AWS CLI Integration

This directory contains scripts and configuration files for automating AWS service management, particularly focused on Route 53 DNS updates for EC2 instances.

## Overview

The AWS CLI integration provides automated DNS management for EC2 instances by:

1. Retrieving the instance's current public IP address
2. Updating Route 53 DNS records to point to this IP address
3. Running as a systemd service for reliability

This enables dynamic DNS functionality for EC2 instances, ensuring that domain names consistently resolve to the correct IP addresses even when they change (e.g., after instance restarts or when using spot instances).

## Directory Structure

```
3.AWS-CLI/
├── create_service.yml       # Ansible playbook for deploying the service
└── files/
    ├── update_route53.sh    # Script that updates Route 53 DNS records
    └── update_route53.service # Systemd service definition
```

## Components

### update_route53.sh

This script:
- Retrieves the current public IP address of the EC2 instance
- Gets the instance ID using IMDSv2 (Instance Metadata Service)
- Retrieves the instance's Name tag from EC2
- Constructs a DNS record name using the Name tag and a configured domain suffix
- Updates the Route 53 hosted zone with an A record pointing to the instance's public IP

Configuration parameters in the script:
- `HOSTED_ZONE_ID`: The Route 53 hosted zone ID
- `TTL`: Time-to-live for the DNS record (in seconds)
- `DOMAIN_SUFFIX`: The domain suffix to append to the instance name

### update_route53.service

A systemd service definition that:
- Runs the update_route53.sh script
- Executes after the network is available
- Runs as the ubuntu user

### create_service.yml

An Ansible playbook that:
- Copies the update_route53.sh script to the EC2 instance
- Installs the systemd service definition
- Enables and starts the service

## Usage

### Prerequisites

- AWS CLI installed and configured with appropriate permissions
- EC2 instance with:
  - A Name tag assigned
  - IAM role with permissions for Route 53 record updates
  - Access to the EC2 metadata service

### Deployment

Deploy the service to your EC2 instance using Ansible:

```bash
ansible-playbook -i your_inventory create_service.yml
```

### Manual Execution

You can also run the script manually:

```bash
ssh your-instance
sudo bash /home/ubuntu/update_route53.sh
```

## Required IAM Permissions

The EC2 instance requires an IAM role with the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/<HOSTED_ZONE_ID>"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeTags"
      ],
      "Resource": "*"
    }
  ]
}
```

## Troubleshooting

If DNS updates are not working:

1. Check that the script is running successfully:
   ```bash
   sudo systemctl status update_route53.service
   ```

2. Verify the EC2 instance has the correct IAM permissions

3. Ensure the instance has a Name tag assigned

4. Check the Route 53 hosted zone ID is correct

5. Verify network connectivity to AWS API endpoints

## Security Considerations

- The script uses IMDSv2 for enhanced security when accessing instance metadata
- Ensure the IAM role has the minimum necessary permissions
- Consider using VPC endpoints for Route 53 if operating in a restricted network environment
