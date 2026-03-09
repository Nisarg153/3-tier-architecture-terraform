# Terraform AWS 3-Tier Node.js Application Deployment

## Overview

This repository contains **Infrastructure as Code (IaC)** to provision and automate the deployment of a scalable **multi-tier Node.js web application** on AWS using **Terraform**. The application follows a three-tier architecture consisting of:

- **Web Tier**: Node.js-based application deployed on EC2 instances.
- **Application Tier**: Application logic running within the EC2 instances.
- **Database Tier**: Database hosted on Amazon RDS.

## Node.js Application

The **Node.js application code** used in this project is sourced from the following repository:

- [AWS Three-Tier Web Architecture Workshop](https://github.com/aws-samples/aws-three-tier-web-architecture-workshop)

## Infrastructure Overview
## Things to modify
1. S3 bucket name as you like.
2. First configure the desired, min and max capacity of the auto scaling group to 0. then apply "terraform apply" command.
3. Once infrastructure is provisioned copy the rds instance writer endpoint and paste it into the app-tier/DBconfig.js
4. Modify the internal load balance link into nginx.conf file.
5. Modify the name of the bucket which you set in script files.
6. Now upload the application code to s3 bucket and then set the desired , min and max to 2, 2 and 3 respectively. 
## Reference

For a detailed explanation of the infrastructure being deployed, refer to this [blog post](https://shreyashbhise.hashnode.dev/deploy-a-three-tier-architecture-on-aws-end-to-end-project-demo).

---
