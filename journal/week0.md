# Week 0 — Billing and Architecture

## Required Homework

### Install AWS-CLI in Gitpod Workspace
Installed AWS CLI in the workspace directory via the command:
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Configured CLI to access my AWS account by passing my credentials as Environmental Variable:
```
export AWS_ACCESS_KEY_ID="access-key"
export AWS_SECRET_ACCESS_KEY="secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

