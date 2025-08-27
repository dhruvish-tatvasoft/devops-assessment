# AWS DevOps CI/CD Starter Project

## Overview
This project demonstrates a basic CI/CD pipeline using AWS CodePipeline, CloudFormation, Lambda, and EC2.

- Lambda function written in Python (`lambda/`)
- Simple static frontend served from EC2 (`frontend/`)
- CloudFormation templates in `cloudformation/`
- Buildspec for CodeBuild to package Lambda

## Setup

1. Replace `your-key-name` in `cloudformation/main.yml` with your EC2 key pair.
2. Push this repo to GitHub.
3. Create CodePipeline with:
   - Source: GitHub repo (main branch)
   - Build: AWS CodeBuild using `buildspec.yml`
   - Deploy: CloudFormation to deploy stack

## Notes

- Lambda code is packaged and uploaded to S3 by CodeBuild.
- EC2 instance serves simple HTML on port 80.
- IAM roles are created via CloudFormation (`roles.yml`).
- You need to create or provide:
  - EC2 key pair
  - Permissions to create IAM roles/policies

## Bonus

- Add API Gateway in `lambda-deploy.yml` to expose Lambda via HTTP.
- Add CloudWatch alarms for monitoring.

---
