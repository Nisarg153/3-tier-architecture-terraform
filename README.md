# AWS 3-Tier Architecture — Infrastructure as Code with Terraform

## About This Project

This project provisions a complete three-tier web application infrastructure on AWS using Terraform. The architecture includes a web tier, application tier, and database tier — deployed across multiple availability zones for high availability.

This was my first Infrastructure as Code project. I built it to learn Terraform fundamentals — modules, state management, variables, and how to translate a manual AWS architecture into reproducible, automated infrastructure. I successfully deployed the full stack and verified the Node.js application running end-to-end.

> **Based on:** [AWS Three-Tier Web Architecture Workshop](https://github.com/aws-samples/aws-three-tier-web-architecture-workshop) and [Shreyash Bhise's walkthrough](https://shreyashbhise.hashnode.dev/deploy-a-three-tier-architecture-on-aws-end-to-end-project-demo) — I used these as references to understand the architecture and then implemented the entire infrastructure in Terraform with a modular structure.

---

## Architecture

```
                         Internet
                            │
                            ▼
                    ┌───────────────┐
                    │  Internet     │
                    │  Gateway      │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  Application  │
                    │  Load Balancer│
                    │  (Public)     │
                    └───────┬───────┘
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
     ┌─────────────────┐        ┌─────────────────┐
     │  Web Tier (EC2) │        │  Web Tier (EC2) │
     │  AZ-1 (Public)  │        │  AZ-2 (Public)  │
     │  Nginx + Node.js│        │  Nginx + Node.js│
     └────────┬────────┘        └────────┬────────┘
              │                          │
              └─────────┬────────────────┘
                        ▼
                ┌───────────────┐
                │  Internal     │
                │  Load Balancer│
                │  (Private)    │
                └───────┬───────┘
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
 ┌─────────────────┐        ┌─────────────────┐
 │  App Tier (EC2) │        │  App Tier (EC2) │
 │  AZ-1 (Private) │        │  AZ-2 (Private) │
 │  Node.js API    │        │  Node.js API    │
 └────────┬────────┘        └────────┬────────┘
          │                          │
          └──────────┬───────────────┘
                     ▼
             ┌───────────────┐
             │  Amazon RDS   │
             │  (MySQL)      │
             │  Multi-AZ     │
             └───────────────┘
```

---

## AWS Resources Provisioned

| Resource | Purpose |
|----------|---------|
| **VPC** | Isolated network with custom CIDR block |
| **Public Subnets (2)** | Web tier — across 2 AZs |
| **Private Subnets (4)** | App tier (2) + Database tier (2) — across 2 AZs |
| **Internet Gateway** | Public internet access for web tier |
| **NAT Gateway** | Outbound internet for private subnets |
| **Application Load Balancer** | Distributes traffic to web tier EC2 instances |
| **Internal Load Balancer** | Routes requests from web tier to app tier |
| **Auto Scaling Groups (2)** | Web tier and app tier — scales EC2 instances based on demand |
| **EC2 Instances** | Web servers (Nginx + Node.js) and app servers (Node.js API) |
| **Amazon RDS (MySQL)** | Database tier with multi-AZ support |
| **S3 Bucket** | Stores application code for EC2 bootstrapping |
| **Security Groups** | Controls traffic between tiers (web → app → db) |
| **IAM Roles** | EC2 instance profiles for S3 access |

---

## Terraform Structure

```
├── main.tf                   # Root module — calls child modules
├── variables.tf              # Input variables
├── outputs.tf                # Output values (ALB DNS, RDS endpoint, etc.)
├── provider.tf               # AWS provider configuration
├── .terraform.lock.hcl       # Provider version lock
├── modules/
│   ├── vpc/                  # VPC, subnets, IGW, NAT, route tables
│   ├── alb/                  # Application & internal load balancers
│   ├── asg/                  # Auto Scaling Groups + launch templates
│   ├── rds/                  # RDS MySQL instance
│   ├── s3/                   # S3 bucket for app code
│   └── security/             # Security groups for each tier
└── README.md
```

The infrastructure is broken into **reusable Terraform modules**, each responsible for a specific layer. This modular approach keeps the code organized and makes individual components easy to understand and modify.

---

## Tech Stack

- **IaC Tool:** Terraform (HCL)
- **Cloud Provider:** AWS
- **Application:** Node.js (from AWS 3-Tier Workshop)
- **Web Server:** Nginx (reverse proxy)
- **Database:** Amazon RDS (MySQL)
- **Scripting:** Shell scripts for EC2 bootstrapping

---

## What I Learned

This was my first Terraform project, and it taught me several foundational concepts:

- **Terraform Modules:** How to break infrastructure into reusable, isolated components instead of writing one massive configuration file.
- **State Management:** Understanding `terraform.tfstate`, why it matters, and why `.terraform.lock.hcl` should be committed.
- **Multi-Tier Networking:** How VPCs, public/private subnets, NAT gateways, and security groups work together to isolate traffic between tiers.
- **Dependency Ordering:** Terraform handles most dependencies automatically, but some resources (like setting ASG capacity to 0 first, then uploading app code to S3, then scaling up) require a deliberate deployment sequence.
- **Load Balancer Architecture:** The difference between an internet-facing ALB (web tier) and an internal ALB (app tier), and how they route traffic through the architecture.

---

## Deployment Notes

The deployment requires a specific sequence due to dependencies between the application code and infrastructure:

1. Set Auto Scaling Group desired/min/max capacity to **0** in `variables.tf`
2. Run `terraform apply` to provision the base infrastructure
3. Copy the **RDS writer endpoint** and update `app-tier/DBconfig.js`
4. Copy the **internal load balancer DNS** and update `nginx.conf`
5. Upload the application code to the **S3 bucket**
6. Update ASG desired/min/max to **2/2/3** and run `terraform apply` again

> For detailed deployment steps, see [SETUP_GUIDE.md](./SETUP_GUIDE.md)

---
