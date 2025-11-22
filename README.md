# DynamoDB Read-Only Access from EC2 (Console, CLI, and CloudFormation)

**Author:** Will A. Soto — Cloud DevOps Engineer ☁️  

---

## Overview

This project implements the same secure pattern three different ways:

> A DynamoDB table of classic movies, queried from an EC2 instance using an IAM **instance profile** with **read-only** permissions and **no stored access keys** — delivered via **Console**, **CLI**, and **CloudFormation**.

It mirrors a real DevOps workflow:

- Prototype manually in the console  
- Automate using the AWS CLI  
- Standardize with Infrastructure-as-Code (CloudFormation)  

---

## Architecture Summary

- **DynamoDB** table storing movie metadata (`Title`, `Genre`, `Rating`, `ReleaseDate`)  
- **EC2 instance** running Amazon Linux 2023  
- **IAM role + instance profile** granting read-only access to the table  
- **No access keys** on the instance — uses role credentials only  
- **VPC + public subnet + security group** allowing SSH from a trusted IP  

Same architecture, three different deployment paths.

---

## Repository Contents

```text
mediacatalog-dynamodb-iam-ec2/
│
├── ec2-trust-policy.json              # EC2 assume-role trust policy
├── mediacatalog-cli-read-policy.json  # Read-only DynamoDB inline policy (CLI path)
├── mediacatalog-cf.yaml               # CloudFormation template (DynamoDB + IAM + EC2)
├── movies-batch.json                  # Movie dataset for CLI batch write
├── movies-cf-batch.json               # Movie dataset for CloudFormation / follow-up CLI
│
└── validation-screenshots/            # Proof of work across all three implementations
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

Implementation Paths
1. AWS Console — Hands-On Validation

Goal: Prove the design works manually before automating it.

Created DynamoDB table MediaCatalog

Added movie items via the console (Friday, Rush Hour, Bad Boys, The Fifth Element, etc.)

Created IAM role with:

Trust: ec2.amazonaws.com

Permissions: dynamodb:Scan, GetItem, DescribeTable, Query on MediaCatalog

Launched EC2 instance (Amazon Linux 2023, t2.micro) with the instance profile attached

Verified from the instance:

aws dynamodb scan --table-name MediaCatalog succeeds

aws dynamodb put-item ... returns AccessDeniedException

This path validated the security model and least-privilege IAM design.

2. AWS CLI — Fully Scripted Deployment

Goal: Rebuild the same architecture using only the CLI from a local machine.

Key steps (backed by the JSON files in this repo):

Create DynamoDB table
aws dynamodb create-table \
  --region us-east-1 \
  --table-name MediaCatalogCLI \
  --attribute-definitions AttributeName=Title,AttributeType=S \
  --key-schema AttributeName=Title,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
Batch write movie data
aws dynamodb batch-write-item \
  --region us-east-1 \
  --request-items file://movies-batch.json
Create IAM role + policy
aws iam create-role \
  --role-name MediaCatalogCLIReadRole \
  --assume-role-policy-document file://ec2-trust-policy.json

aws iam put-role-policy \
  --role-name MediaCatalogCLIReadRole \
  --policy-name MediaCatalogCLIReadOnly \
  --policy-document file://mediacatalog-cli-read-policy.json
Create instance profile and launch EC2
aws iam create-instance-profile \
  --instance-profile-name MediaCatalogCLIInstanceProfile

aws iam add-role-to-instance-profile \
  --instance-profile-name MediaCatalogCLIInstanceProfile \
  --role-name MediaCatalogCLIReadRole

aws ec2 run-instances \
  --region us-east-1 \
  --image-id $(aws ssm get-parameters \
    --names /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 \
    --query 'Parameters[0].Value' --output text) \
  --instance-type t2.micro \
  --subnet-id <SUBNET_ID> \
  --security-group-ids <SG_ID> \
  --iam-instance-profile Name=MediaCatalogCLIInstanceProfile \
  --key-name <KEY_NAME> \
  --tag-specifications \
    'ResourceType=instance,Tags=[{Key=Name,Value=MediaCatalogCLI-EC2}]'
Validation: from the new EC2 instance

scan on MediaCatalogCLI succeeds

put-item fails as expected

This path demonstrates comfort with IAM, EC2, and DynamoDB from the command line.

3. CloudFormation — MediaCatalogCF Stack

Goal: Capture the full pattern as Infrastructure-as-Code.

The template mediacatalog-cf.yaml provisions:

MediaCatalogCF DynamoDB table (on-demand billing)

MediaCatalogCFReadRole IAM role with read-only DynamoDB permissions

Instance profile and EC2 instance (MediaCatalogCF-EC2)

Security group for SSH from a trusted IP

High-level deployment:
aws cloudformation deploy \
  --stack-name MediaCatalogCFStack \
  --template-file mediacatalog-cf.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    SubnetId=<SUBNET_ID> \
    SecurityGroupId=<SG_ID> \
    KeyName=<KEY_NAME>
Validation mirrors the previous paths:

scan on MediaCatalogCF works

put-item from the instance fails with AccessDenied

This path turns the design into a reusable, version-controlled stack.

Validation Screenshots

All three implementations are backed by screenshots in validation-screenshots/, including:

DynamoDB console view of all 10 movies

IAM role showing read-only DynamoDB permissions

EC2 instance summary with the correct role attached

CLI output for successful scan and denied put-item

CloudFormation stack in CREATE_COMPLETE and the EC2 instance it launched

These images make the project immediately reviewable for teammates, mentors, or hiring managers.

What This Project Demonstrates

IAM & Security

Principle of least privilege with tightly scoped DynamoDB permissions

Correct use of instance profiles instead of long-lived access keys

Automation

Ability to move from console clicks to reproducible CLI workflows

Comfort reading and writing JSON IAM policies

Infrastructure-as-Code

Packaging the full pattern into a single CloudFormation template

Named IAM resources managed through stacks
Reuse Ideas

You can adapt this pattern for:

Internal read-only dashboards over DynamoDB data

Analytics or catalog services that must not mutate production data

Teaching IAM roles vs. user credentials in AWS workshops or labs

Fork the repo, swap the movie dataset, and reuse the CloudFormation/IAM pieces as a secure starting point.

