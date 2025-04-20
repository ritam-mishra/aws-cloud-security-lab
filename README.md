# AWS Cloud Security Lab (CSPM)
Simulated AWS security misconfigurations tested with Prowler &amp; ScoutSuite for real-world vulnerability detection

![AWS](https://img.shields.io/badge/cloud-AWS-orange)
![Security](https://img.shields.io/badge/focus-Cloud%20Security-red)
![Tools](https://img.shields.io/badge/tools-Prowler%20%7C%20ScoutSuite-green)

## Table of Contents
  - [Lab Overview](#lab-overview)
  - [Lab Setup](#lab-setup)
    * [1. IAM User Provisioning](#1-iam-user-provisioning)
    * [2. AWS CLI Configuration](#2-aws-cli-configuration)
    * [3.Vulnerable Infrastructure Deployment](#3vulnerable-infrastructure-deployment)
    * [4. Tool Setup & Execution](#4-tool-setup-&-execution)
  - [Prowler – Setup and Execution](#prowler-–-setup-and-execution)
    * [1. Prowler Setup](#1-prowler-setup)
      + [Step 1: Install Build Dependencies](#step-1:-install-build-dependencies)
      + [Step 2: Download and Compile Python 3.12](#step-2:-download-and-compile-python-312)
      + [Step 3: Use Python 3.12 with Poetry (Fix the Version Issue)](#step-3:-use-python-312-with-poetry-(fix-the-version-issue))
      + [Step 4: Install Prowler Dependencies](#step-4:-install-prowler-dependencies)
    * [2. Prowler Execution](#2-prowler-execution)
    * [3. Top Misconfigurations reported by Prowler](#3-top-misconfigurations-reported-by-prowler)
  - [ScoutSuite – Setup and Execution](#scoutsuite-–-setup-and-execution)
    * [1. ScoutSuite Setup](#1-scoutsuite-setup)
      + [Step 1: Clone ScoutSuite Repository](#step-1:-clone-scoutsuite-repository)
      + [Step 2: Create Python Virtual Environment](#step-2:-create-python-virtual-environment)
      + [Step 3: Install Dependencies](#step-3:-install-dependencies)
    * [2. ScoutSuite Execution](#2-scoutsuite-execution)
    * [3. Top Misconfigurations reported by ScoutSuite](#3-top-misconfigurations-reported-by-scoutsuite)
  - [Verdict](#verdict)
  - [Interesting Finding](#interesting-finding)
  - [📌 Final Thoughts](#📌-final-thoughts)


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

**This dual-user setup allowed me to observe how tools like Prowler and ScoutSuite enumerate account-wide configurations, even when executed using scoped credentials.**

### 2. AWS CLI Configuration

Access keys were generated for the cloud-security-labs user and configured in the local terminal using:

``` bash
aws configure
```
The keys granted full programmatic access to AWS resources from the terminal.

### 3.Vulnerable Infrastructure Deployment

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

To audit AWS resources against security and compliance standards, I used **Prowler**, a robust open-source tool that supports checks across multiple compliance frameworks.

### 1. Prowler Setup

The setup wasn’t straightforward due to version constraints in the project’s ```pyproject.toml```, which rejects Python 3.13 (the version pre-installed on my system). 

I took the help of GenAI to solve my problem here. 
Here's how I successfully set up Prowler using Python 3.12 compiled from source:

#### Step 1: Install Build Dependencies
```bash
sudo apt update
```
```bash
sudo apt install -y wget build-essential zlib1g-dev libncurses5-dev \
  libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev curl libbz2-dev
  ```

#### Step 2: Download and Compile Python 3.12

```bash 
cd /usr/src
sudo wget https://www.python.org/ftp/python/3.12.2/Python-3.12.2.tgz
sudo tar xzf Python-3.12.2.tgz
```
```bash
cd Python-3.12.2
sudo ./configure --enable-optimizations
sudo make -j$(nproc)
sudo make altinstall
```

```make altinstall``` ensures it won’t override your system’s default Python version. It took me a lot of time (~20-30 mins) for this code to finish up. You have to wait patiently until it gets completed. 

#### Step 3: Use Python 3.12 with Poetry (Fix the Version Issue)

```bash
poetry env use /usr/local/bin/python3.12
```

#### Step 4: Install Prowler Dependencies

```bash
poetry install
```

### 2. Prowler Execution

After setting up the virtual environment correctly, I ran the tool using:
```bash
poetry run prowler aws -p default -M html,json
```

- ```-p default``` uses the default AWS CLI profile (already configured with valid credentials).

- ```-M html,json``` outputs both human-readable HTML reports and structured JSON for automation.

To open the HTML report I went to the output file that was generated on the prowler directory. I clicked on the ```prowler-output-xxx-xxxx.html``` file and it successfully opened my report. 

### 3. Top Misconfigurations reported by Prowler

| Status | Severity | Service | Check Title | Resource | Extended Status | Risk | Recommendation | Compliance |
|--------|----------|---------|-------------|----------|------------------|------|----------------|------------|
| ❌ FAIL | Critical | EC2 | Ensure no EC2 instances allow ingress from the internet to TCP port 22 (SSH) | `instance-name:xxyyzz-instance` | `xxyyzz-instance` has SSH exposed to `0.0.0.0/0` on a public IP address in a public subnet | SSH is a common target for brute force attacks. If an EC2 instance allows ingress from the internet to TCP port 22, it is at risk of being compromised. | Restrict SSH access to known IPs, use bastion hosts or AWS Session Manager. | KISA-ISMS-P-2023-korean: 2.6.1, 2.6.2, 2.6.6, 2.10.1, 2.10.3 <br> ISO27001-2022: A.8.20, A.8.21, A.8.22 <br> SOC2: cc_6_6 |
| ❌ FAIL | Critical | IAM | Ensure only hardware MFA is enabled for the root account | `arn:aws:iam:us-east-1:xxyyxxzz:mfa` | Root account has a virtual MFA instead of a hardware MFA device enabled | Root accounts need the highest level of protection. Virtual MFA is not sufficient for Level 2. | Enable **hardware MFA** for the root account via IAM Console → Dashboard → Activate MFA. | KISA-ISMS-P-2023-korean: 2.6.1, 2.6.2, 2.6.6, 2.10.1, 2.10.3 <br> ISO27001-2022: A.8.20, A.8.21, A.8.22 <br> SOC2: cc_6_6 |
| ❌ FAIL | High | EC2 | Ensure Instance Metadata Service Version 2 (IMDSv2) is enforced for EC2 instances at the account level to protect against SSRF vulnerabilities | `arn:aws:ec2:us-east-1:xxxyyyzzz:account` | IMDSv2 is not enabled by default for EC2 instances | EC2 instances that use IMDSv1 are vulnerable to SSRF attacks | Enable IMDSv2 on EC2 instances and enforce this setting at the account level per region | KISA-ISMS-P-2023-korean: 2.10.8 <br> ISO27001-2022: A.8.20, A.8.21, A.8.22 <br> CIS-2.0: 5.6 |
| ❌ FAIL | High | IAM | Ensure IAM AWS-Managed policies that allow full "*:*" administrative privileges are not attached | `arn:aws:iam::aws:policy/AdministratorAccess` | AWS policy AdministratorAccess is attached and allows '*:*' administrative privileges | Full privileges increase attack surface and contradict the principle of least privilege | Replace with least-privilege custom policies based on actual user or role requirements | AWS-Foundational-Security-Best-Practices: IAM.1 <br> KISA-ISMS-P-2023-korean: 2.5.1 <br> HIPAA: 164_308_a_1_ii_b, 164_308_a_3_i, etc. <br> ISO27001-2022: A.5.18, A.8.2 <br> NIST-CSF-1.1: ac_1, ac_4, pt_3 |
| ❌ FAIL | High | S3 | Check S3 Account Level Public Access Block | `arn:aws:s3:us-east-1:xxxyyzzz:account` | Block Public Access is not configured for the account | Public access policies may be applied to sensitive data buckets | Enable Block Public Access at the account level to avoid accidental public exposure | PCI-4.0: 1.2.8.31, 1.2.8.32, etc. <br> AWS-Foundational-Security-Best-Practices: S3.1 |
| ❌ FAIL | High | EC2 | Ensure no security groups allow ingress from 0.0.0.0/0 or ::/0 to any port | `arn:aws:ec2:eu-north-1:xxyyxxzz:security-group/sg-xxzzzyyyzzxx` | Security group `launch-wizard-1` has open ports to the Internet; interface type not allowed | Unrestricted access may allow unauthorized users to reach EC2 instances | Implement Zero Trust. Restrict ingress to necessary IPs and ports. Monitor east-west and north-south traffic. | PCI-4.0: 1.2.5.17, 1.2.8.41, etc. <br> KISA-ISMS-P-2023-korean: 2.6.1, 2.6.2, 2.6.6, 2.10.1, 2.10.3 |

**Assessment Overview**
| Total Passed | Total Failed |
|--------------|----------|
| 130 | 86 |

## ScoutSuite – Setup and Execution

I was eager to check the results provided by another very famous and robust open-source tool **ScoutSuite**.

### 1. ScoutSuite Setup

Setting up the environment for running ScoutSuite was quite simple. I made sure that I am currently running in a virtual environment `venv`. 

#### Step 1: Clone ScoutSuite Repository
```bash
git clone https://github.com/nccgroup/ScoutSuite.git
```
```
cd ScoutSuite
```

#### Step 2: Create Python Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate
```

#### Step 3: Install Dependencies 
```bash
pip install -r requirements.txt
```

### 2. ScoutSuite Execution

To verify that my AWS credentials are setup correctly, I ran

``` bash
 aws sts get-caller-identity
```
To run ScoutSuite Against AWS, I ran:
```bash
python scout.py aws
```
It took some time to analyse my AWS Account and the report was generated under the /reports file in the ScoutSuite Directory. 
I clicked on the report to view it.

### 3. Top Misconfigurations reported by ScoutSuite

- **❌ Credentials Unused for 90 Days or Greater Are Not Disabled**  
  Inactive IAM user credentials pose a security risk if compromised. Best practice recommends disabling or removing access keys that haven’t been used in over 90 days.

- **❌ Lack of Key Rotation for 90 Days (Key Status: Active)**  
  Active KMS keys that haven’t been rotated in over 90 days weaken cryptographic hygiene. Enabling automatic key rotation helps maintain compliance and reduce risk.

- **❌ Root Account without Hardware MFA**  
  The AWS root user has unrestricted access. Without MFA—preferably hardware-based—the account is at significant risk of unauthorized access.

- **❌ IAM User Without MFA**  
  IAM users without Multi-Factor Authentication are vulnerable to credential theft. Enabling MFA adds an additional security layer for AWS console access.

- **❌ EBS Volume Not Encrypted**  
  Unencrypted Elastic Block Store (EBS) volumes expose stored data at rest. Encryption should be enforced to secure sensitive workloads.
a
- **❌ Security Group Opens SSH Port (22) to 0.0.0.0/0**  
  Allowing global access to SSH is highly insecure. This configuration exposes EC2 instances to brute-force and other network attacks.

- **❌ S3 Bucket Access Logging Disabled**  
  Without access logging, it’s difficult to trace data access or unauthorized operations. Enabling logging helps with audits and incident response.

- **❌ S3 Bucket Allows Clear Text (HTTP) Communication**  
  HTTP access to buckets transmits data in plain text. Enforce HTTPS to ensure encrypted communication and prevent data interception.


## Verdict

I personally preferred **ScoutSuite** over other tools like Prowler because of its **clean and intuitive UI**, along with **dashboard-style visualizations** that make it easy to interpret the findings. The ability to **filter, sort, and navigate misconfigurations** through a centralized web-based report makes it far more comprehensive and user-friendly for auditing and compliance checks.

## Interesting Finding

While testing, I provided **CLI credentials for only one IAM user**, but ScoutSuite was able to generate findings for **both IAM accounts under the root account**. Even more impressively, it scanned and flagged issues related to the **root account itself**, such as lack of MFA and usage of inactive keys. This deeper insight made the analysis more thorough and unexpected.


## 📌 Final Thoughts

This project was aimed at exploring and comparing open-source AWS security auditing tools. ScoutSuite stood out due to its **depth of analysis, ease of navigation, and insightful reporting** capabilities. From setup to report generation, the tool proved to be **smooth, reliable, and beginner-friendly**.

> Thanks for checking out my project! Hope it helps others in securing their AWS environments with better visibility and actionable insights.

---

