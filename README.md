# AWS Cloud Engineer Learning Journey

## Overview
This repository documents my hands-on learning path to becoming a Cloud Engineer, focusing on AWS services, infrastructure automation, and production-grade practices.

## Learning Timeline
**Start Date:** January 10, 2026  
**Target:** 12-week structured learning plan  
**Goal:** Production-ready Cloud Engineer skills

## Current Stack
- **IaC:** Terraform, CloudFormation
- **Containers:** ECS Fargate, EKS (learning)
- **CI/CD:** Jenkins, SonarQube, AWS CodePipeline
- **Monitoring:** CloudWatch, X-Ray (planned)
- **Networking:** VPC, NAT instances, API Gateway
- **Security:** IAM, MFA, Secrets Manager, Parameter Store

## Week 1: Disaster Recovery & Backup Strategy
**Goal:** Implement production-grade backup and audit logging

- [x] [Day 1: CloudTrail Setup](week-1/day-1-cloudtrail.md)
- [x] [Day 2: AWS Backup for RDS](week-1/day-2-aws-backup.md)
- [x] [Day 3: EBS Snapshots & AMI](week-1/day-3-ebs-ami.md)
- [ ] [Day 4: Disaster Recovery Testing](week-1/day-4-disaster-recovery-test.md)
- [ ] [Day 5: DR Documentation](week-1/day-5-dr-documentation.md)

## Week 2-12: Planned Topics
- Multi-account AWS Organizations setup
- Kubernetes/EKS deep dive
- Event-driven architecture (EventBridge, SQS, SNS)
- Advanced observability (X-Ray, distributed tracing)
- AWS Certifications prep
- Production incident response

## Project Portfolio
### 1. Three-Tier Web Application
**Architecture:**
- Frontend: S3 static hosting → CloudFlare (CDN, SSL)
- Backend: API Gateway → VPC Link → Private ALB → ECS Fargate
- Database: Private RDS MySQL
- CI/CD: Jenkins → CodePipeline → ECR

**Key Features:**
- Private networking with NAT instance (cost-optimized)
- Automated deployments
- Infrastructure as Code (Terraform)
- Secrets management (AWS Secrets Manager)

[View detailed architecture →](projects/three-tier-app.md)

## Contact
- **LinkedIn:** [http://www.linkedin.com/in/prathamesh-shukla-615007159]
- **GitHub:** [https://github.com/pratham-0709]

---
*Last Updated: February 18, 2026*