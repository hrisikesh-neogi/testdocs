# CI/CD Pipeline Documentation: Python Function Deployment to Azure

## Overview
This document provides a comprehensive walkthrough of our GitHub Actions workflow for deploying a Python Function App to Azure, specifically targeting the `itip-ecc-rag-dev-np` environment.

## Workflow Architecture
Our CI/CD pipeline is designed to automate the deployment process for our Python Function App, ensuring consistent and reliable deployments with minimal manual intervention.

### Deployment Trigger
The workflow is currently configured with `workflow_dispatch`, which means it can be manually triggered. We've commented out the automatic push trigger (`push` on `main` branch), giving us more controlled deployment flexibility.

### Environment Configuration
We've set up several key environment variables to standardize our deployment process:
- Python Version: 3.11
- Function App Name: `itip-ecc-rag-dev-np`
- Resource Group: `itip-dev-rg-np`

## Authentication Mechanism
### Microsoft Entra ID App Registration
We leverage Microsoft Entra ID (formerly Azure AD) for secure authentication using an App Registration. This provides a robust and standardized approach to accessing Azure resources.

#### Secret Management
Sensitive credentials are securely stored as GitHub repository secrets:
- `AZURE_CLIENT_ID`: Application (client) ID from Entra ID app registration
- `AZURE_TENANT_ID`: Directory (tenant) ID
- `AZURE_SUBSCRIPTION_ID`: Azure subscription identifier
- `AZURE_FUNCTIONAPP_PUBLISH_PROFILE`: Deployment credentials for the Function App

### Authentication Flow
1. The workflow uses the `azure/login@v2` action for authentication
2. Credentials are passed securely via GitHub secrets
3. We use Workload Identity Federation, allowing secure, passwordless authentication

## Deployment Steps
The workflow progresses through several critical stages:

### 1. Repository Checkout
- Uses `actions/checkout@v4`
- Retrieves the latest code from the specified branch

### 2. Python Environment Setup
- Configures Python 3.11 using `actions/setup-python@v5`
- Ensures consistent Python runtime across deployments

### 3. Dependency Resolution
- Upgrades pip
- Installs project dependencies from `requirements.txt`
- Targets `.python_packages/lib/site-packages` for dependency management

### 4. Azure Authentication
- Authenticates with Azure using Entra ID credentials
- Supports no-subscription scenarios with `allow-no-subscriptions: true`

### 5. Environment Configuration
A crucial step where we set multiple application settings dynamically:
- Key Vault URL configuration
- User-Assigned Managed Identity (UAMI) setup
- Storage account details
- AI and search service endpoints
- Deployment-specific parameters like model names and API versions
- Logging and monitoring configurations

### 6. Deployment Execution
- Uses `Azure/functions-action@v1`
- Enables Oryx build system for optimized deployment
- Performs SCM build during deployment

## Runner Configuration
- Uses a custom `uhg-runner`, likely an internally managed GitHub Actions runner

## Key Security Considerations
- Principle of least privilege with specific permissions
- Secrets management through GitHub encrypted secrets
- Workload Identity Federation for enhanced security
- No hard-coded credentials in the workflow file

## Recommended Next Steps
- Implement branch protection rules
- Set up required reviewers for critical branches
- Configure periodic secret rotation
- Add comprehensive logging and monitoring

## Troubleshooting
If deployment fails, check:
- Dependency compatibility
- Azure resource configurations
- Secret and permission settings
- Network and firewall rules

## Version Control
- Workflow File: `.github/workflows/deploy-function.yml`
- Last Updated: [Current Date]
- Maintainer: [Your Team/Name]
