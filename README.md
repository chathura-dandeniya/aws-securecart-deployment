# 🔐 SecureCart — Secure Application Deployment on AWS
### AWS Well-Architected Framework | Security Pillar

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws) ![VPC](https://img.shields.io/badge/Amazon-VPC-blue) ![EC2](https://img.shields.io/badge/Amazon-EC2-yellow) ![RDS](https://img.shields.io/badge/Amazon-RDS-blue) ![S3](https://img.shields.io/badge/Amazon-S3-green) ![WAF](https://img.shields.io/badge/AWS-WAF-red)

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture Diagram](#-architecture-diagram)
- [Services Used](#-services-used)
- [What I Achieved](#-what-i-achieved)
- [Phase 1 — Insecure Baseline](#-phase-1--insecure-baseline)
- [Phase 2 — Secure Architecture](#-phase-2--secure-architecture)
  - [1.4 Secure Networking (VPC + Subnets + Gateways)](#14-secure-networking-vpc--subnets--gateways)
  - [1.5 EC2 in Private Subnet + ALB + Bastion Host](#15-ec2-in-private-subnet--alb--bastion-host)
  - [1.6 Private RDS and S3](#16-private-rds-and-s3)
  - [1.7 AWS WAF Integration](#17-aws-waf-integration)
- [Security Group Architecture](#-security-group-architecture)
- [Before vs After Comparison](#-before-vs-after-comparison)
- [Key Security Learnings](#-key-security-learnings)
- [Cleanup](#-cleanup)

---

## 📖 Project Overview

**SecureCart** is a simulated e-commerce platform used as the basis for a hands-on cloud security project. The platform initially launched with infrastructure built for speed rather than security — leaving critical resources exposed to the public internet.

> **Scenario:** SecureCart serves 10,000 customers. Its EC2 instances, RDS database, and S3 bucket are all publicly exposed, creating critical risks including unauthorised access, data breaches, and compliance violations. A secure cloud redesign is urgently needed.

**My Role:** Cloud Security-focused Solutions Architect  
**Mission:** Transform the vulnerable environment into a production-ready, secure architecture using AWS-native tools and security best practices, aligned with the **AWS Well-Architected Framework (Security Pillar)**.

### What I Built
- ✅ Intentionally insecure baseline to demonstrate real-world vulnerabilities
- ✅ Production-grade secure architecture replacing every insecure component
- ✅ Full documentation of the before/after security posture

---

## 🏗 Architecture Diagram

```
                          ┌─────────────────────────────────────────────────────────┐
                          │                    securecart-vpc (10.0.0.0/16)          │
                          │                                                           │
  Internet  ──────────►  │  ┌──────────────────────┐   ┌────────────────────────┐  │
  Users                  │  │   PUBLIC SUBNETS       │   │   PRIVATE SUBNETS      │  │
                          │  │                        │   │                        │  │
                          │  │  ┌────────────────┐   │   │  ┌──────────────────┐ │  │
                          │  │  │   AWS WAF      │   │   │  │   EC2 Instance   │ │  │
                          │  │  │ (securecart-   │   │   │  │  securecart-app  │ │  │
                          │  │  │  waf-acl)      │   │   │  │  10.0.2.102      │ │  │
                          │  │  └───────┬────────┘   │   │  │  (No Public IP)  │ │  │
                          │  │          │             │   │  └────────┬─────────┘ │  │
                          │  │  ┌───────▼────────┐   │   │           │           │  │
                          │  │  │      ALB        │───┼───┼───────────┘           │  │
                          │  │  │ securecart-alb  │   │   │  ┌──────────────────┐ │  │
                          │  │  │ (Internet-      │   │   │  │   RDS MySQL      │ │  │
                          │  │  │  facing)        │   │   │  │securecart-db-    │ │  │
                          │  │  └────────────────┘   │   │  │  secure          │ │  │
                          │  │                        │   │  │  (No Public IP)  │ │  │
                          │  │  ┌────────────────┐   │   │  └──────────────────┘ │  │
                          │  │  │  NAT Gateway   │   │   │                        │  │
                          │  │  │ securecart-nat │   │   │  ┌──────────────────┐ │  │
                          │  │  └────────────────┘   │   │  │  Bastion Host    │ │  │
                          │  │                        │   │  │  (SSH Only from  │ │  │
                          │  │  ┌────────────────┐   │   │  │  Admin IP)       │ │  │
                          │  │  │Internet Gateway│   │   │  └──────────────────┘ │  │
                          │  │  └────────────────┘   │   │                        │  │
                          │  └──────────────────────┘   └────────────────────────┘  │
                          └─────────────────────────────────────────────────────────┘
```

**Traffic Flow:**
1. User request arrives over HTTP/HTTPS
2. **AWS WAF** filters the request for known threats (SQLi, XSS, bad inputs)
3. **ALB** forwards clean traffic to the private EC2 instance
4. **EC2** runs the backend app and fetches data from **RDS**
5. **NAT Gateway** allows EC2 to access the internet for updates (outbound only)
6. **Bastion Host** provides secure, IP-restricted SSH access to private EC2

---

## 🛠 Services Used

| Service | Purpose |
|---|---|
| **Amazon VPC** | Network isolation using public/private subnets |
| **Amazon EC2** | Application backend hosted in private subnet |
| **Amazon RDS (MySQL)** | Relational database with private connectivity only |
| **Amazon S3** | Object storage configured for private access only |
| **Application Load Balancer (ALB)** | Public-facing entry point forwarding to private EC2 |
| **AWS WAF** | Layer 7 firewall blocking SQLi, XSS, and bad inputs |
| **AWS IAM** | Role-based access control with least privilege |
| **NAT Gateway** | Controlled outbound internet access for private subnet |
| **Internet Gateway** | Inbound/outbound access for public subnet resources |
| **Bastion Host (EC2)** | Secure SSH jump server in public subnet |

---

## 🎯 What I Achieved

| Security Outcome | Implementation |
|---|---|
| **Isolated backend resources** | EC2 and RDS placed in private subnets with no direct internet access |
| **Minimal attack surface** | Only ALB is public-facing — no other resource has a public IP |
| **Least privilege enforced** | Security Groups and IAM restrict every layer to only required traffic |
| **Encryption in transit** | SSL/TLS enforced on RDS connections via `--ssl-mode=VERIFY_IDENTITY` |
| **WAF protection** | SQL injection, XSS, and known bad inputs blocked at the ALB level |

---

## ❌ Phase 1 — Insecure Baseline

> **Note:** This phase was intentionally deployed to demonstrate real-world vulnerabilities. These configurations represent actual misconfigurations that have led to major data breaches.

### What Was Deployed

| Resource | Insecure Configuration | Real-World Risk |
|---|---|---|
| **EC2** | Public subnet + SSH open to `0.0.0.0/0` + HTTP port 80 open | Anyone on the internet can attempt brute-force SSH or exploit the app |
| **RDS MySQL** | `PubliclyAccessible: true` + port 3306 open to `0.0.0.0/0` | Database credentials and data directly accessible without VPN or bastion |
| **S3 Bucket** | Block Public Access disabled + `s3:GetObject` for `Principal: *` | Any file URL publicly readable — customer PII, orders, invoices exposed |

### Vulnerabilities Summary

**Network Security Issues**
- No network segmentation between application tiers
- All resources deployed in public subnets
- Overly permissive security groups allowing global access

**Data Protection Failures**
- Database exposed directly to the public internet
- No encryption for data at rest or in transit
- Publicly accessible file storage with no access controls

**Access Control Problems**
- SSH accessible from any IP address (`0.0.0.0/0`)
- Database port accessible globally
- No IAM role-based access controls

**Monitoring & Compliance Gaps**
- No logging or monitoring configured
- No threat detection enabled
- No compliance tracking

> **Real-world impact:** Similar misconfigurations have exposed over 200 billion records globally, resulting in significant financial penalties and regulatory consequences.

---

## ✅ Phase 2 — Secure Architecture

### 1.4 Secure Networking (VPC + Subnets + Gateways)

**Objective:** Build a properly segmented, multi-tier network architecture separating public-facing resources from sensitive backend systems.

#### VPC Configuration

| Setting | Value |
|---|---|
| **VPC Name** | `securecart-vpc` |
| **IPv4 CIDR** | `10.0.0.0/16` |
| **DNS Resolution** | Enabled |
| **DNS Hostnames** | Enabled |

#### Subnet Design

| Subnet | CIDR | AZ | Role |
|---|---|---|---|
| `public-subnet-1` | `10.0.1.0/24` | `us-east-1a` | ALB, NAT Gateway, Bastion Host |
| `public-subnet-2` | `10.0.3.0/24` | `us-east-1b` | ALB (multi-AZ redundancy) |
| `private-subnet-1` | `10.0.2.0/24` | `us-east-1a` | EC2 application server |
| `private-subnet-2` | `10.0.4.0/24` | `us-east-1b` | RDS database |

#### Route Table Configuration

**Public Route Table (`public-rt`)**
| Destination | Target |
|---|---|
| `0.0.0.0/0` | Internet Gateway (`igw-01ab7a6fe41a90fa0`) |
| `10.0.0.0/16` | local |

*Associated with:* `public-subnet-1`, `public-subnet-2`

**Private Route Table (`private-rt`)**
| Destination | Target |
|---|---|
| `0.0.0.0/0` | NAT Gateway (`securecart-nat`) |
| `10.0.0.0/16` | local |

*Associated with:* `private-subnet-1`, `private-subnet-2`

**Key Security Principle:** Private subnets route outbound traffic through NAT Gateway — EC2 can pull updates but is completely unreachable from the internet inbound.

---

### 1.5 EC2 in Private Subnet + ALB + Bastion Host

**Objective:** Host the application in a private subnet, accessible only through the ALB, with a Bastion Host for controlled admin SSH access.

#### EC2 Application Server

| Setting | Value |
|---|---|
| **Name** | `securecart-app` |
| **Instance Type** | `t3.micro` |
| **AMI** | Amazon Linux 2023 |
| **Subnet** | `private-subnet-1` |
| **Public IP** | ❌ None |
| **Private IP** | `10.0.2.102` |

#### Application Load Balancer

| Setting | Value |
|---|---|
| **Name** | `securecart-alb` |
| **Scheme** | Internet-facing |
| **Subnets** | `public-subnet-1` (us-east-1a) + `public-subnet-2` (us-east-1b) |
| **Listener** | HTTP:80 → `securecart-tg` (100%) |

#### Bastion Host

| Setting | Value |
|---|---|
| **Name** | `bastion-host` |
| **Subnet** | `public-subnet-1` |
| **Public IP** | `18.208.173.123` |
| **SSH Access** | Restricted to admin IP only (`49.184.232.105/32`) |

**SSH Access Pattern:**
```bash
# Step 1 — SSH into Bastion Host
ssh -i DevOps-Project.pem ec2-user@18.208.173.123

# Step 2 — From Bastion, SSH into private EC2
chmod 400 DevOps-Project.pem
ssh -i DevOps-Project.pem ec2-user@10.0.2.102
```

---

### 1.6 Private RDS and S3

**Objective:** Secure the data layer — RDS accessible only from EC2, S3 assets not publicly accessible.

#### RDS Configuration

| Setting | Value |
|---|---|
| **Identifier** | `securecart-db-secure` |
| **Engine** | MySQL 8.4.8 Community |
| **Instance Class** | `db.t4g.micro` |
| **Subnet Group** | `securecart-db-subnet-group` (private-subnet-1 + private-subnet-2) |
| **Public Accessibility** | ❌ Disabled |
| **Internet Access Gateway** | ❌ Disabled |
| **Security Group** | `rds-ec2-access` — port 3306 from `app-sg` only |

**Connecting to RDS (via EC2 with SSL):**
```bash
mysql -h securecart-db-secure.ccdg28skylq9.us-east-1.rds.amazonaws.com \
  -P 3306 -u admin -p \
  --ssl-mode=VERIFY_IDENTITY \
  --ssl-ca=./global-bundle.pem
```

#### S3 Configuration

| Setting | Value |
|---|---|
| **Bucket Name** | `securecart-assets-084375585267-us-east-1-an` |
| **Block All Public Access** | ✅ Enabled |
| **Public Bucket Policy** | ❌ None |
| **Direct URL Access** | Returns `AccessDenied` |

---

### 1.7 AWS WAF Integration

**Objective:** Add application-layer protection by integrating AWS WAF with the ALB to block common web exploits.

#### Web ACL Configuration

| Setting | Value |
|---|---|
| **Name** | `securecart-waf-acl` |
| **Resource Type** | Regional (ALB) |
| **Region** | `us-east-1` |
| **Associated Resource** | `securecart-alb` |
| **Default Action** | Allow (matched threats are blocked) |

#### Managed Rule Groups Applied

| Rule Group | Capacity | Protection |
|---|---|---|
| `AWSManagedRulesCommonRuleSet` | 700 WCUs | OWASP Top 10 — broad web application protection |
| `AWSManagedRulesKnownBadInputsRuleSet` | 200 WCUs | Malformed/malicious HTTP request patterns |
| `AWSManagedRulesSQLiRuleSet` | 200 WCUs | SQL injection attack prevention |
| **Total** | **1100 / 5000 WCUs** | Within free tier limits |

---

## 🔒 Security Group Architecture

Security groups are chained so each layer only accepts traffic from the layer directly above it — a textbook **defense-in-depth** model.

```
Internet
    │
    ▼
[alb-sg]          HTTP/HTTPS from 0.0.0.0/0 (WAF filters first)
    │
    ▼
[app-sg]          HTTP:80 from alb-sg only
                  SSH:22 from bastion-sg only
    │
    ▼
[rds-ec2-access]  MySQL:3306 from app-sg only
```

| Security Group | Inbound Rule | Source |
|---|---|---|
| `bastion-sg` | SSH:22 | Admin IP (`49.184.232.105/32`) only |
| `app-sg` | HTTP:80 | `alb-sg` only |
| `app-sg` | SSH:22 | `bastion-sg` only |
| `rds-ec2-access` | MySQL:3306 | `app-sg` only |

---

## 📊 Before vs After Comparison

| Security Control | Phase 1 (Insecure) | Phase 2 (Secure) |
|---|---|---|
| **EC2 location** | Public subnet with public IP | Private subnet, no public IP |
| **EC2 SSH access** | `0.0.0.0/0` — anyone | Admin IP only via Bastion |
| **App access URL** | Raw public IP | ALB DNS name only |
| **RDS accessibility** | Public internet, port 3306 open globally | Private subnet, EC2 access only |
| **RDS port exposure** | `0.0.0.0/0` on port 3306 | `app-sg` source only |
| **S3 access** | Public — any file URL works | Access Denied for all public requests |
| **Network segmentation** | None — all resources in public subnet | Public/private tier separation |
| **WAF** | None | 3 managed rule groups active |
| **Encryption in transit** | None | SSL enforced on RDS |

---

## 🧠 Key Security Learnings

**1. Network Isolation is the Foundation**
Placing EC2 and RDS in private subnets eliminates direct internet exposure. Even if application-layer security fails, the network layer prevents direct access.

**2. Minimal Attack Surface**
Exposing only the ALB means there is a single, hardened, monitored entry point. Every other resource is invisible to the public internet.

**3. Security Group Chaining Enforces Least Privilege**
Instead of IP-based rules, using security group references ensures only specifically authorised AWS resources can communicate with each other — regardless of IP changes.

**4. Defense-in-Depth**
Each security layer operates independently. If WAF is bypassed, ALB security groups still protect EC2. If EC2 is compromised, RDS is still inaccessible from the internet.

**5. Bastion Host Pattern**
Placing admin SSH access behind a dedicated, IP-restricted bastion host eliminates the need for any production instance to have a public IP or open SSH port.

---

## 🧹 Cleanup

To avoid ongoing charges, terminate or delete the following resources after the project:

```
1. Bastion Host EC2 instance (once admin tasks are complete)
2. securecart-app EC2 instance
3. securecart-db-secure RDS instance
4. securecart-alb (Application Load Balancer)
5. securecart-tg (Target Group)
6. securecart-nat (NAT Gateway — charges per hour)
7. Elastic IP associated with NAT Gateway
8. S3 buckets (empty before deleting)
9. securecart-waf-acl (WAF Web ACL)
10. securecart-vpc and all subnets/route tables/IGW
```

> **Note:** NAT Gateway and ALB incur hourly charges. Delete these first if keeping other resources temporarily.

---

## 📎 Related

- [AWS Well-Architected Framework — Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [AWS Managed WAF Rule Groups](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-list.html)
- [VPC Security Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)

---

*Built as part of a hands-on AWS Solutions Architect portfolio project.*
