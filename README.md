🚀 Ansible Deployment on Azure VM using GitHub Actions
📌 Overview

This project demonstrates a secure and automated DevOps deployment workflow using Ansible, Azure Virtual Machine, GitHub Actions, and Azure Key Vault.

The application/server configuration is managed using Ansible, while GitHub Actions automates the deployment process. Sensitive credentials and configuration values are securely managed through Azure Key Vault, avoiding hardcoded secrets in the source code.

🏗️ Architecture
                  ┌──────────────────────┐
                  │   Developer / Git    │
                  │      Repository      │
                  └──────────┬───────────┘
                             │
                             │ Push / Pull Request
                             ▼
                  ┌──────────────────────┐
                  │    GitHub Actions    │
                  │      CI/CD Pipeline  │
                  └──────────┬───────────┘
                             │
                             │ Authenticate
                             ▼
                  ┌──────────────────────┐
                  │    Azure Key Vault   │
                  │                      │
                  │  Secrets / Credentials│
                  └──────────┬───────────┘
                             │
                             │ Secure Secret Retrieval
                             ▼
                  ┌──────────────────────┐
                  │     Azure VM         │
                  │                      │
                  │  Ansible Controller  │
                  │       / Target       │
                  └──────────┬───────────┘
                             │
                             │ Ansible Playbook
                             ▼
                  ┌──────────────────────┐
                  │      Deployment      │
                  │   Server Config /    │
                  │   Application Setup  │
                  └──────────────────────┘

🛠️ Technologies Used
Technology	Purpose
Azure VM	Hosting the target server
Ansible	Configuration management and automation
GitHub Actions	CI/CD automation
Azure Key Vault	Secure secrets management
GitHub	Source code and pipeline management
YAML	Ansible playbooks and GitHub Actions workflow
🔄 Deployment Workflow

The deployment follows this workflow:

Developer pushes changes to the GitHub repository.

GitHub Actions workflow is triggered.

The workflow authenticates with Azure.

Required secrets are securely retrieved from Azure Key Vault.

GitHub Actions prepares the Ansible environment.

Ansible connects to the Azure VM.

The Ansible playbook executes the required configuration/deployment tasks.

The application/server configuration is updated automatically.

Deployment status is reported by GitHub Actions.

Workflow
Code Push
    ↓
GitHub Repository
    ↓
GitHub Actions
    ↓
Azure Authentication
    ↓
Azure Key Vault
    ↓
Retrieve Secrets
    ↓
Ansible
    ↓
Azure VM
    ↓
Application / Server Deployment

📁 Project Structure
.
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── ansible/
│   ├── inventory
│   ├── playbook.yml
│   ├── roles/
│   │   └── application/
│   │       ├── tasks/
│   │       │   └── main.yml
│   │       ├── templates/
│   │       └── handlers/
│   │           └── main.yml
│   │
│   └── ansible.cfg
│
├── scripts/
│   └── deployment.sh
│
├── requirements.txt
└── README.md


Update the structure according to your actual project files.

🔐 Secrets Management

Security is an important part of this deployment architecture.

Sensitive information such as:

VM credentials

SSH private keys

Application secrets

API credentials

Database credentials

Azure authentication credentials

should not be hardcoded in the repository.

These secrets are securely stored in Azure Key Vault and accessed by the CI/CD workflow when required.

Secret Management Flow
Azure Key Vault
       │
       │ Secure Authentication
       ▼
GitHub Actions
       │
       │ Runtime Secret
       ▼
Ansible
       │
       ▼
Azure VM


This approach helps keep sensitive information separate from application and infrastructure code.

⚙️ GitHub Actions CI/CD

The GitHub Actions workflow is responsible for automating the deployment.

A typical workflow performs the following steps:

name: Ansible Deployment

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Azure Login
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Retrieve Secrets
        # Retrieve required secrets from Azure Key Vault

      - name: Install Ansible
        run: |
          sudo apt-get update
          sudo apt-get install -y ansible

      - name: Run Ansible Playbook
        run: |
          ansible-playbook \
            -i ansible/inventory \
            ansible/playbook.yml


Replace the example configuration with the actual authentication and Key Vault implementation used in the project.

☁️ Azure VM

The Azure Virtual Machine acts as the deployment target.

Ansible is used to automate tasks such as:

Installing required packages

Managing services

Copying configuration files

Deploying application files

Updating application versions

Restarting services

Applying server configuration

Example:

ansible-playbook \
  -i inventory \
  playbook.yml

🤖 Ansible

Ansible provides configuration management and deployment automation.

A basic playbook structure:

---
- name: Deploy Application
  hosts: azure_vm
  become: true

  tasks:

    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install required packages
      apt:
        name:
          - nginx
        state: present

    - name: Deploy application
      # Application deployment task

    - name: Restart application service
      service:
        name: nginx
        state: restarted


The actual tasks can be customized according to the application and server requirements.

🔑 Azure Key Vault Integration

Azure Key Vault is used as the centralized secret-management layer.

Instead of storing credentials directly inside:

ansible/playbook.yml


or:

.github/workflows/deploy.yml


the sensitive values are stored securely in Azure Key Vault and retrieved during the deployment process.

Benefits

🔐 Centralized secret management

🚫 No hardcoded credentials

🔄 Easier secret rotation

🛡️ Reduced risk of credential exposure

📋 Better separation between code and secrets

🔒 Security Practices

This project follows several security-focused DevOps practices:

Never commit passwords or private keys to Git.

Store sensitive values in Azure Key Vault.

Use GitHub Secrets for required CI/CD authentication values.

Apply least-privilege access to Azure resources.

Avoid printing secrets in CI/CD logs.

Keep Ansible variables separate from sensitive credentials.

Use SSH keys instead of passwords where possible.

Rotate credentials periodically.

Restrict Azure VM network access using appropriate firewall/security rules.

🚀 How to Run
1. Clone the Repository
git clone <repository-url>
cd <repository-name>

2. Configure Azure Resources

Create/configure:

Azure Resource Group

Azure Virtual Machine

Azure Key Vault

Required identities/permissions

3. Configure Secrets

Store required sensitive values in Azure Key Vault.

Example:

VM_USERNAME
VM_SSH_PRIVATE_KEY
APPLICATION_SECRET


Use the actual secret names configured in your project.

4. Configure GitHub Actions

Add the required Azure authentication/configuration values under:

GitHub Repository
    → Settings
    → Secrets and variables
    → Actions

5. Configure Ansible Inventory

Example:

[azure_vm]
azure-server ansible_host=<VM_IP>

[azure_vm:vars]
ansible_user=<VM_USER>


Do not commit private credentials to the repository.

6. Run Deployment

Push changes to the configured branch:

git add .
git commit -m "Deploy application"
git push origin main


GitHub Actions will automatically trigger the deployment workflow.

📊 DevOps Benefits

This architecture provides:

Automation — Deployment is executed automatically through CI/CD.

Consistency — Ansible ensures repeatable server configuration.

Security — Secrets are managed through Azure Key Vault.

Scalability — The same Ansible approach can be extended to multiple VMs.

Version Control — Infrastructure and deployment configuration are maintained in Git.

Reduced Manual Work — Server configuration and deployment tasks are automated.

🧪 Deployment Validation

After deployment, the pipeline can perform validation checks such as:

ansible -i inventory azure_vm -m ping


Application/service health checks can also be added to the GitHub Actions workflow.

Example:

curl -f http://<SERVER-IP>/health


If the validation fails, the GitHub Actions job can mark the deployment as failed.

🔮 Future Improvements

Possible enhancements include:

Infrastructure provisioning using Terraform

Azure Managed Identity instead of long-lived credentials

Deployment to multiple Azure VMs

Ansible Vault for additional secret protection

Automated rollback strategy

Application health checks

Blue-Green deployment

Monitoring with Azure Monitor

Containerization using Docker

Kubernetes deployment using AKS

🎯 Project Objective

The main objective of this project is to demonstrate how Ansible and GitHub Actions can be integrated with Azure infrastructure to create a secure, automated, and repeatable deployment pipeline, while using Azure Key Vault for centralized secrets management.

👨‍💻 Author - Ugrasen Gangwar

DevOps Engineer | Azure | Ansible | GitHub Actions | CI/CD