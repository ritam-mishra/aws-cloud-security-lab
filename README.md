# AWS Cloud Security Lab (CSPM)
Simulated AWS security misconfigurations tested with Prowler &amp; ScoutSuite for real-world vulnerability detection

![AWS](https://img.shields.io/badge/cloud-AWS-orange)
![Security](https://img.shields.io/badge/focus-Cloud%20Security-red)
![Tools](https://img.shields.io/badge/tools-Prowler%20%7C%20ScoutSuite-green)

## 📚 Table of Contents
- [Lab Overview](#-lab-overview)
- [Lab Setup](#-lab-setup)
- [Prowler – Setup and Execution](#-prowler--setup-and-execution)
- [ScoutSuite – Setup and Execution](#-scoutsuite--setup-and-execution)
- [Top Vulnerabilities](#-top-vulnerabilities)
- [Final Thoughts](#-final-thoughts)


## Lab Overview

This project simulates a vulnerable AWS environment to evaluate real-world misconfigurations using two leading open-source cloud security assessment tools: **Prowler and ScoutSuite**. The objective is to replicate a realistic cloud setup with intentional security gaps across IAM, EC2, and S3, then assess and analyze the environment using automated auditing tools.

This cloud security lab serves as a practical demonstration of:

   - Evaluating IAM misconfigurations and over-permissive access policies

   - Identifying open ports and insecure network configurations on EC2

   - Auditing public S3 bucket policies and exposure

   - Generating detailed HTML-based security reports using both tools

   - Understanding how misconfigurations propagate even with scoped IAM users

Both tools were executed in a controlled environment using an IAM user with limited, though potent, AWS service privileges. The findings included cross-user visibility of configurations, exposing how scoped permissions can still surface data from broader resource access in a shared AWS account.

## Lab Setup

The following configurations and environment preparation steps were followed to set up the lab:

### 1. IAM User Provisioning
A new IAM user named ```cloud-security-xxxx``` was created to simulate cloud operations and serve as the security assessment identity. The following AWS-managed policies were attached:
- ```AmazonEC2FullAccess```

- ```AmazonEC2FullAccess```

- ```AmazonS3FullAccess```

- ```Billing```

- ```IAMFullAccess```

- ```IAMUserChangePassword```

These permissions enabled full provisioning, identity manipulation, and resource-level access for the testing scope. \
Additionally, an existing IAM user already present in the environment had elevated privileges with the following policies:

- ```AdministratorAccess```

- ```NetworkAdministrator```

**This dual-user setup allowed us to observe how tools like Prowler and ScoutSuite enumerate account-wide configurations, even when executed using scoped credentials.**

## 2. AWS CLI Configuration

Access keys were generated for the cloud-security-labs user and configured in the local terminal using:

``` bash
aws configure
```
The keys granted full programmatic access to AWS resources from the terminal.

## 3.Vulnerable Infrastructure Deployment

Using the cloud-security-labs IAM user, the following insecure infrastructure elements were provisioned to simulate a vulnerable AWS state:

- EC2 Instances: Configured with open ports (e.g., 0.0.0.0/0 access on SSH and HTTP) and unrestricted Security Groups

- S3 Buckets: Misconfigured with public access enabled, lacking bucket policies and encryption

- IAM Role Adjustments: Default password policies and overly permissive IAM user permissions

### 4. Tool Setup & Execution

After deploying misconfigured infrastructure, the environment was audited using two tools:

a. Prowler

Prowler was cloned from its GitHub repository and executed using the pre-configured AWS CLI credentials. It generated an in-depth security assessment aligned with AWS CIS Benchmarks and other compliance standards. Output was exported in HTML for easier interpretation.

b. ScoutSuite

ScoutSuite was installed in a Python virtual environment due to system restrictions on direct package installation. Once set up, it performed a deep analysis of the AWS account and generated a navigable HTML dashboard highlighting misconfigurations across IAM, EC2, S3, and other services.

## Prowler – Setup and Execution