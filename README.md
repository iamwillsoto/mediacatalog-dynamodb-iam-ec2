DynamoDB Read-Only Access from EC2 (Console, CLI, and CloudFormation)

Author: Will A. Soto — Cloud DevOps Engineer ☁️

Overview

This project demonstrates three different deployment methods (AWS Console, AWS CLI, and CloudFormation) to implement a secure, read-only DynamoDB architecture accessed from an EC2 instance using an IAM instance profile — with no stored credentials on the server.

This mirrors real DevOps workflows:

Prototype manually

Automate via CLI

Standardize with IaC

Architecture Summary

DynamoDB table storing classic movie metadata

EC2 instance running Amazon Linux 2023

IAM role with read-only DynamoDB permissions

No access keys stored on the instance

Public subnet + SG allowing SSH from trusted IP

Repository Contents
mediacatalog-dynamodb-iam-ec2/
│
├── ec2-trust-policy.json
├── mediacatalog-cli-read-policy.json
├── mediacatalog-cf.yaml
├── movies-batch.json
├── movies-cf-batch.json
│
└── validation-screenshots/
    ├── 01-dynamodb-console-scan-mediacatalog.png
    ├── 02-iam-policy-readonly-dynamodb.png
    ├── 03-ec2-console-mediacatalog-reader.png
    ├── 04-ec2-cli-dynamodb-scan.png
    ├── 05-ec2-cli-putitem-access-denied.png
    ├── 06-local-cli-dynamodb-scan-mediacatalogcli.png
    ├── 07-cli-ec2-runinstances-mediacatalogcli.png
    ├── 08-ec2-mediacatalogcli-scan-and-putitem-error.png
    ├── 09-cloudformation-stack-create-complete.png
    └── 10-ec2-cloudformation-mediacatalogcf-ec2.png

Deployment Methods
1. AWS Console
Manual validation of:

Table creation

IAM role

Instance profile

EC2 instance

Read-only tests

2. AWS CLI
Automated deployment of:

DynamoDB table

Batch write

IAM role + instance profile

EC2 instance launch

3. CloudFormation (IaC)
Reusable template creating:

DynamoDB table

IAM role + instance profile

EC2 instance

Security group

Notes
All screenshots are included for validation

JSON and YAML files are clean and production-ready

No access keys stored anywhere — only role-based access
