# Jenkins and HashiCorp Vault Integration

This directory contains configuration for integrating Jenkins with HashiCorp Vault for secure secrets management in CI/CD pipelines.

## Overview

This setup enables Jenkins to securely retrieve and use secrets (such as AWS credentials) from HashiCorp Vault during pipeline execution. The integration is implemented using Docker containers for both Jenkins and Vault, with a sample Jenkins pipeline that demonstrates retrieving AWS credentials from Vault and using them to interact with AWS services.

## Prerequisites

### Docker Setup
- Docker and Docker Compose installed
- Network connectivity between containers

### Jenkins Requirements
- Jenkins LTS (running in Docker)
- Required Jenkins plugins:
  - Pipeline
  - Pipeline Utility Steps
  - Pipeline: Groovy
  - Recommended basic plugins from Jenkins setup

### Vault Requirements
- HashiCorp Vault (running in Docker)
- Initialized and unsealed Vault instance

## Setup Instructions

### 1. Start the Services

Use the docker-compose.yaml file in the parent directory to start both Jenkins and Vault:

```bash
docker-compose up -d
```

This will start:
- Jenkins on port 8080
- Vault on port 8200

### 2. Initialize HashiCorp Vault

After the Vault container is running, initialize it:

```bash
docker exec -it vault-new vault operator init
```

This command will generate:
- 5 unseal keys
- 1 initial root token

**IMPORTANT**: Save these keys and token securely. You will need them to unseal the vault and authenticate.

### 3. Unseal the Vault

Vault starts in a sealed state and needs to be unsealed using 3 of the 5 keys generated during initialization:

- Access the Vault UI at http://localhost:8200
- Enter 3 of the 5 unseal keys when prompted

Alternatively, you can unseal via CLI:

```bash
docker exec -it vault-new vault operator unseal <unseal-key-1>
docker exec -it vault-new vault operator unseal <unseal-key-2>
docker exec -it vault-new vault operator unseal <unseal-key-3>
```

### 4. Configure Jenkins Credentials

In Jenkins:
1. Navigate to Manage Jenkins > Manage Credentials
2. Add a new Secret Text credential with ID `vault-token`
3. Use the root token from Vault initialization as the value

### 5. Store AWS Credentials in Vault

Store your AWS credentials in Vault:

```bash
docker exec -it vault-new /bin/sh
export VAULT_TOKEN=<your-root-token>
vault secrets enable -path=secret kv-v2
vault kv put secret/aws/jenkins access_key_id=<your-aws-access-key> secret_access_key=<your-aws-secret-key>
```

## Using the Jenkins Pipeline

The included `JenkinsFile` demonstrates:
1. Retrieving AWS credentials from Vault using the vault-token credential
2. Using those credentials to make AWS API calls (listing EC2 instances)

Key components:
- Environment variables for Vault address and secret path
- Authentication using the Jenkins credential store
- Parsing JSON responses from Vault
- Securely passing AWS credentials to the AWS CLI

## Important Notes

- **DISCLAIMER**: The Jenkins pipeline will only work while the Vault is unsealed
- Vault must be re-unsealed after container restarts
- The secret path in the JenkinsFile (`v1/secret/aws/data/aws/jenkins`) should match your Vault configuration
- AWS region in the pipeline is set to `il-central-1` - adjust as needed

## Troubleshooting

- If Jenkins cannot connect to Vault, check network connectivity between containers
- Verify the Vault token has appropriate permissions to access the secrets
- Ensure Vault is unsealed before running the pipeline
- Check Jenkins logs for authentication or permission issues
