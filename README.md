# AWS Security Best Practices Demo

This project provisions a secure EC2 instance and S3 bucket using Terraform, IAM, and Bash hardening techniques. Perfect for portfolio and job readiness.

## Features
- SSH key generated and saved locally
- Hardened EC2 instance (Ubuntu)
- Encrypted and versioned S3 bucket
- IAM least-privilege role
- Free-tier compatible (us-east-1)

## Deploy

```bash
terraform init
terraform apply -auto-approve
