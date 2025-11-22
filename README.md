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

Deployed three different ways (Console, CLI, CloudFormation)

Repository Contents
mediacatalog-dynamodb-iam-ec2/
│
├── ec2-trust-policy.json
├── mediacatalog-cli-read-policy.json
├── mediacatalog-cf.yaml
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
1. AWS Console (Manual Validation)

Created DynamoDB table MediaCatalog

Attached read-only IAM policy

Launched EC2 instance with MediaCatalogReadRole

Verified:

Scan works

PutItem denied

No stored credentials

2. AWS CLI (Fully Scripted)

Includes:

aws dynamodb create-table
aws dynamodb batch-write-item
aws iam create-role
aws iam put-role-policy
aws ec2 run-instances


Validates automation and repeatability.

3. CloudFormation (IaC)

All resources deployed using:

mediacatalog-cf.yaml


CloudFormation creates:

DynamoDB table

IAM role + instance profile

EC2 instance

Security group

Perfect for production standardization and CI/CD pipelines.

Validation

All three deployments produced identical outcomes:

✅ Scan allowed
❌ Write operations blocked
✅ Instance uses IAM role (not access keys)
✅ Template fully reproducible

See the validation-screenshots/ folder for complete output.

Concluding Insights

Implementing the same architecture three different ways strengthens cloud fluency and demonstrates adaptability across operational, automated, and IaC-driven environments.

This repository serves as a reference blueprint for secure, role-based DynamoDB access from EC2, applicable to internal dashboards, analytics tools, and read-heavy microservices.
