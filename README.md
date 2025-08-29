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

---
# Deployment Document: AWS CI/CD Pipeline for DevOps Assessment

## 1. Overview

This document provides a comprehensive walkthrough of the setup and deployment of the "devops-assessment" project. The project consists of a fully automated CI/CD pipeline built on AWS services. The pipeline automatically builds, packages, and deploys a simple web application whenever changes are pushed to a GitHub repository. The application includes a static frontend served from an EC2 instance and a serverless backend function powered by AWS Lambda.

This document details a manual, security-conscious approach where IAM service roles were created and configured with specific permissions before being assigned to the pipeline and its components.

---

## 2. Architecture

The architecture is an event-driven CI/CD workflow managed by AWS CodePipeline.

*   **Source Control:** GitHub repository `dhruvish-tatvasoft/devops-assessment`.
*   **CI/CD Orchestration:** AWS CodePipeline (`devops-pipeline`) monitors the GitHub repository and orchestrates the workflow.
*   **Build Environment:** AWS CodeBuild (`assessment-lambda-packager`) packages the Lambda function.
*   **Infrastructure as Code (IaC):** AWS CloudFormation (`DevOps-Assessment-Stack`) provisions all AWS resources.
*   **Compute:** AWS EC2 hosts the frontend; AWS Lambda runs the backend.
*   **Storage:** Amazon S3 (`devops-assessment-artifacts-ddc-20250828`) stores build artifacts.
*   **Region:** All resources were deployed in `eu-north-1` (Stockholm).

---

## 3. Prerequisites

1.  **AWS Account:** An active AWS account.
2.  **GitHub Repository:** A personal copy (fork or clone) of the project code.
3.  **EC2 Key Pair:** An EC2 Key Pair named `devops-assessment-key` created in the `eu-north-1` region.
4.  **IAM Roles:** As detailed in the steps below, specific IAM roles for each service are required.

---

## 4. Detailed Deployment Steps

The entire infrastructure was provisioned and deployed via a custom CI/CD pipeline configured through the AWS Management Console.

### Step 1: Prerequisite Resource Creation (S3 & IAM)

Before building the pipeline, foundational resources for storage and permissions were established.

1.  **S3 Artifact Bucket Creation (S3 Console):**
    *   **Path:** `AWS Console > S3 > Create bucket`.
    *   **Action:** A new S3 bucket was created to serve as the pipeline's artifact store.
    *   **Configuration:**
        *   **Bucket Name:** `devops-assessment-artifacts-ddc-20250828` (a globally unique name).
        *   **Region:** `eu-north-1` (Stockholm), to co-locate with other resources.
        *   **Object Ownership:** `ACLs disabled` was selected for a simpler, bucket-owner-enforced security model.
        *   **Block Public Access:** All settings to block public access were left **enabled** to ensure artifacts remain private.
        *   **Bucket Versioning:** Left `Disabled` for this project.
        *   **Default Encryption:** `Enabled` with `Amazon S3-managed keys (SSE-S3)` for baseline security.

2.  **IAM Role Preparation (IAM Console):**
    *   Specific IAM service roles were configured to ensure each service operated with the necessary permissions.
    *   **CodePipeline Service Role:** An existing role (`AWSCodePipelineServiceRole-eu-north-1-devops-assessment-pipelin`) was customized with managed policies (`AWSCodeBuildDeveloperAccess`,`AWSCloudFormationFullAccess`,`AmazonS3FullAccess`,`AWSCodeDeployFullAccess`,`AWSCodePipeline_FullAccess`, etc.) and an inline policy for `iam:PassRole`.
    *   **CodeBuild Service Role:** An existing role was prepared with managed policies (`AmazonS3FullAccess`, `AmazonEC2FullAccess`, etc.) and a specific inline policy for CloudFormation, Lambda, EC2, and S3 actions.
    *   **CloudFormation Service Role:** A new role named `CloudFormation-Admin-Role-For-Pipeline` was created with a comprehensive policy for all necessary service actions and KMS decryption.

### Step 2: Pipeline Creation and Configuration (CodePipeline Console)

A custom pipeline was built using the "Build custom pipeline" option.

1.  **Pipeline Settings:**
    *   **Path:** `CodePipeline > Create pipeline > Step 1: Choose pipeline settings`.
    *   **Configuration:**
        *   **Pipeline Name:** `devops-pipeline`
        *   **Execution Mode:** `QUEUED` was chosen to ensure every triggered execution runs to completion sequentially.
        *   **Service Role:** The pre-configured "Existing service role" for CodePipeline was selected.
2.  **Source Stage:**
    *   **Path:** `Step 2: Add source stage`.
    *   **Configuration:** `GitHub (via GitHub App)` was connected to the `dhruvish-tatvasoft/devops-assessment` repository's `main` branch.
3.  **Build Stage:**
    *   **Path:** `Step 3: Add build stage`.
    *   **Configuration:** An `AWS CodeBuild` project (`assessment-lambda-packager`) was created using a managed `Amazon Linux` container, a single build type, and the pre-configured CodeBuild service role. The input was set to `SourceArtifact`.
4.  **Test Stage:**
    *   This stage was **skipped** as no automated tests were included in the source repository.
5.  **Deploy Stage:**
    *   **Path:** `Step 4: Add deploy stage`.
    *   **Configuration:**
        *   **Deploy Provider:** `AWS CloudFormation`.
        *   **Input Artifact:** `BuildArtifact`.
        *   **Action Mode:** `CREATE_UPDATE`.
        *   **Stack Name:** `DevOps-Assessment-Stack`.
        *   **Template Path:** `BuildArtifact::cloudformation/main.yml`.
        *   **Role ARN:** The pre-created `CloudFormation-Admin-Role-For-Pipeline` was assigned.
        *   **Capabilities:** The mandatory checkbox,checked: `CAPABILITY_IAM, CAPABILITY_NAMED_IAM`
### Review: Code Pipeline Configuration Summary

The following screenshots provide a visual summary of the final pipeline configuration as shown in the "Review" step before creation.


![Pipeline Settings Review](https://i.postimg.cc/Yqxhw4sT/img1.png)

---



![Source Stage Configuration](https://i.postimg.cc/Dy5yM9WT/img2.png)

---


![Build Stage Configuration](https://i.postimg.cc/G20cDrfk/img3.png)

---


![Deploy Stage Configuration](https://i.postimg.cc/GtddMKdt/img4.png)

---

### Step 3: Pipeline Execution, Verification, and Final Status

1.  **Creation and Initial Trigger:**
    *   After clicking **"Create pipeline"**, the configuration was saved and the pipeline's first execution was automatically triggered. The `Source` stage successfully pulled the code from GitHub.

2.  **Successful Build Stage:**
    *   The `Build` stage executed successfully. It followed the instructions in the `buildspec.yml` to package the Lambda function, creating a `lambda.zip` file.
    *   As part of its process, the build stage successfully uploaded this `lambda.zip` artifact to the `devops-assessment-artifacts-ddc-20250828` S3 bucket.

3.  **Automatic Triggers on Commit:**
    *   The CI/CD trigger mechanism was verified. Any new `git push` to the `main` branch of the connected repository correctly and automatically started a new pipeline execution.

4.  **Monitoring & Troubleshooting the Deploy Stage:**
    *   The pipeline consistently failed during the `Deploy` stage. Multiple issues were identified and resolved, including missing IAM capabilities, incorrect S3 template URLs, and region-specific AMI ID errors (as detailed in the accompanying Issue Document).
    *   Despite resolving several preliminary issues, a final error persisted, preventing a fully successful deployment.

5.  **Final Status and Verification:**
    *   **Successful Components:** The Source and Build stages are fully functional and automated. The `lambda.zip` artifact is correctly being generated and stored in S3.
    *   **Pending Issue:** The `Deploy` stage is in a failed state due to a "NoSuchKey" error when creating the Lambda function. I worked diligently to resolve this final issue, but a complete solution was not reached due to time constraints. As a result, the EC2 and Lambda resources could not be verified as live.

---
![Final Status and Verification](https://i.postimg.cc/630pxgHn/image5.png)

---
---
# Issue Document :


This document outlines the key issues encountered during the setup and deployment of the CI/CD pipeline, along with their root causes, specific solutions, and current status.

| ID  | Status    | Stage & Issue Description                                | Summarized Error Message                                    | Root Cause Analysis                                                                                                 | Specific Solution Implemented                                                                                                                                                                                                                                      |
|:---:|:----------|:----------------------------------------------------------|:------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|  1  | Resolved  | **Build Stage:** IAM Role missing general permissions.    | (Initial setup) Permissions errors during execution.        | The IAM roles for the pipeline services were created with insufficient permissions to manage other AWS services like S3, CloudFormation, and Lambda. | **File Changed: None (IAM Console)** <br> The IAM role for the CodeBuild project was edited. The following AWS-managed policies were attached: `AWSCodeBuildDeveloperAccess`, `AWSCloudFormationFullAccess`, `AWSLambda_FullAccess`, and `AmazonS3FullAccess`.          |
|  2  | Resolved  | **Build Stage:** Fails with S3 "Access Denied" error.     | `AccessDenied: User ... is not authorized to perform: s3:GetObject` | The CodeBuild role lacked the specific permission to download the source artifact object from the designated S3 bucket. | **File Changed: None (IAM Console)** <br> An inline policy was added to the CodeBuild service role, granting the `s3:GetObject` action on the specific resource ARN: `arn:aws:s3:::devops-assessment-artifacts-ddc-20250828/*`.                          |
|  3  | Resolved  | **Deploy Stage:** Fails with an invalid `TemplateURL`.    | `TemplateURL must be a supported URL.`                        | The CloudFormation templates for the nested stacks were pointing to an incorrect S3 URL, likely due to a region mismatch. | **File Changed: `cloudformation/main.yml`** <br> The `TemplateURL` properties for both the `IAMRolesStack` and `LambdaStack` resources were updated to point to the correct S3 object URL in the `eu-north-1` region.                                          |
|  4  | Resolved  | **Deploy Stage:** Fails because the EC2 AMI does not exist. | `The image id '[ami-0c02fb55956c7d316]' does not exist`        | The EC2 AMI ID was hardcoded. AMI IDs are region-specific, so the hardcoded ID was invalid for the `eu-north-1` deployment region. | **Files Changed: `cloudformation/main.yml` & `cloudformation/ec2.yml`** <br> 1. A new `Parameter` named `LatestAmiId` was added to `main.yml` to dynamically fetch the latest AMI. 2. The `ec2.yml` template was updated to use `ImageId: !Ref LatestAmiId` instead of the hardcoded value. |
|  5  | Resolved  | **Deploy Stage:** Fails with Lambda "NoSuchKey" error due to bucket name. | `S3 Error Code: NoSuchKey. The specified key does not exist.` | The bucket name was hardcoded incorrectly in the Lambda template, preventing CloudFormation from finding the `lambda.zip` file. | **File Changed: `cloudformation/lambda-deploy.yml`** <br> The `Code` block for the Lambda function was updated to specify the correct `S3Bucket` name: `devops-assessment-artifacts-ddc-20250828`.                                                    |
|  6  | **Pending** | **Deploy Stage:** Lambda creation fails with "NoSuchKey" error due to missing artifact. | `S3 Error Code: NoSuchKey. The specified key does not exist.` | The `lambda.zip` file, which the Lambda function's code depends on, is not present in the `BuildArtifact` that is being passed to the Deploy stage. The `buildspec.yml` is likely discarding it instead of passing it through. | **Next Action:** The next step is to modify the `buildspec.yml` file. The `artifacts` section needs to be configured with a wildcard (`files: - '**/*'`) to ensure `lambda.zip` is created and included in the output artifact. I worked diligently to solve this final issue within the time constraints, but it remains the last pending item to achieve a fully successful deployment. |


    *   
