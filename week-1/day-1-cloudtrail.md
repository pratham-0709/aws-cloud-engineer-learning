# Day 1: CloudTrail Setup

## Date Completed
January 10, 2026

## Objective
Set up AWS CloudTrail for comprehensive audit logging across all regions with automated lifecycle management.

## What is CloudTrail?
AWS CloudTrail records all API calls made in your AWS account, providing audit trails for security analysis, compliance, and troubleshooting.

## Architecture
```
AWS API Calls (all services, all regions)
    ↓
CloudTrail (multi-region trail)
    ↓
S3 Bucket (cloudtrail-logs-xxxxx)
    ├─ 0-30 days: Standard-IA storage
    └─ 365 days: Auto-delete
```

## Implementation Steps

### Step 1:- Created S3 Bucket to hold logs 

```bash 
Bucket name: cloudtrail-logs-teckexplorers
Region: ap-south-1 (Mumbai)
Versioning: Suspended
Public Access: Blocked (all)
```

### 2. Configure Lifecycle Policy
**Transition Rule:**
- After 30 days → Standard-IA storage
- After 365 days → Permanently delete

**Steps to be avoided**
- Standard-IA (not Glacier) to avoid small file penalty for CloudTrail logs
- 365-day retention meets most compliance requirements
- Auto-cleanup prevents storage bloat

### 3. Enable CloudTrail


```bash

Go to CloudTrail → Trails → Create trail

Trail name: Te-Account-Trail

Storage location: Use existing S3 bucket → select your bucket from Step 1

Log file SSE-KMS encryption: Disable (or enable if you want extra security, but costs more)

Log file validation: ✅ Enable (detects tampering)

SNS notification: ❌ Leave disabled (this was causing your spam)

CloudWatch Logs: ❌ Disable for now (optional, costs extra)

```
```bash
Event Settings:

Event type: Management events

API activity: Both Read and Write

Exclude AWS KMS events: ✅ Check (reduces noise)

Exclude RDS Data API events: ✅ Check (reduces noise)

Multi-region trail: ✅ Enable (captures all regions)

```
Once everything is verified **Create Trail** wait for few minutes logs will start getting stored in respective S3 bucket. 

### 4. Verify 
**Check Event History:**
```bash
Go to CloudTrail → Event history
You should see recent events (takes 15 minutes for first logs)
Go to your S3 bucket → you'll see folders like AWSLogs/<account-id>/CloudTrail/...
Download one log file → it's JSON format with all API calls
```
Verified by testing cli command :- **aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=RunInstances**

# PROD SETUP 
## Setups and points to be taken care off. 

### What Production Actually Looks Like:

Multi-Account Organization Setup

```bash
AWS Organization (Master Account)
├── CloudTrail Organization Trail (logs ALL accounts)
├── Centralized S3 Bucket (in Security account)
├── Cross-account access for Security/Audit teams
└── Member Accounts (Dev, Staging, Prod) - all logs flow to one place
```
Security teams need visibility across ALL AWS accounts from one place. You don't want 50 separate trails in 50 accounts.

Real-Time Alerting & Monitoring
Production shouldn't just store logs - should be actively monitored. For which the flow should be as below:- 

```
CloudTrail → CloudWatch Logs → Metric Filters → CloudWatch Alarms → SNS → PagerDuty/Slack
```

Real-world alerts:

⚠️ Root account login → Immediate alert to security team

⚠️ Security group opened to 0.0.0.0/0 → Alert + auto-remediation Lambda

⚠️ IAM policy changes → Notify security team

⚠️ S3 bucket made public → Instant alert + rollback

⚠️ Unauthorized API calls → Flag for investigation

⚠️ Failed authentication attempts (>10 in 5 min) → Potential breach alert

======================================================================================

**Integration with SIEM Tools**

- Production sends CloudTrail to Security Information and Event Management systems
- Who accessed what resources, when, from where
- Unusual API patterns
- Compliance violations (PCI-DSS, SOC2, HIPAA)
- Cost anomalies (someone spun up 100 GPU instances?)
- Log Integrity & Tamper Detection
- You enabled: Log file validation ✅

**Production adds:**

- SNS notifications for digest file delivery (proves logs weren't tampered)

- S3 Object Lock (WORM - Write Once Read Many) - logs can't be deleted even by admins

- MFA Delete on S3 bucket - can't delete logs without MFA token

- Separate AWS account for log storage - even if prod account is compromised, logs are safe

**Note:** **Attackers often delete CloudTrail logs to cover their tracks. Production makes this impossible.**

=======================================================================================

**Advanced Filtering & Cost Optimization**

**Production optimizes:**

Exclude noisy events: Read-only S3 API calls (GetObject logged millions of times)

Exclude KMS events: Crypto operations generate massive logs

Data events on critical S3 buckets only: Not all buckets, just sensitive ones

Advanced event selectors: Log only DeleteBucket, PutBucketPolicy - not ListBucket


**Cost impact:**

Your way: Might log 100GB/month → $0.50/month (fine for personal)

Production (unoptimized): 10TB/month → $50/month in S3 + retrieval costs

Production (optimized): 500GB/month → $2.50/month

## References
- [AWS CloudTrail Documentation](https://docs.aws.amazon.com/cloudtrail/)
- [CloudTrail Best Practices](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/best-practices-security.html)