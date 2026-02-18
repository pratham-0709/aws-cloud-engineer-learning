# Day 3: EBS Snapshots & AMI - Complete Guide

## Overview
**Date Completed:** January 11, 2026  
**Time Invested:** 1 hours  
**Day 3 Status: ✅ COMPLETE**

---

## What Are EBS Snapshots & AMIs?

### EBS Snapshots
Point-in-time backups of EBS volumes stored in S3. They are **incremental** - only changed blocks are saved after the first snapshot, making them cost-effective. You can restore volumes from snapshots or create new volumes in different availability zones.

**Use cases:** Volume-level backups, data migration, disaster recovery

### AMI (Amazon Machine Image)
Complete backup of an EC2 instance including OS, configurations, and all attached EBS volumes. AMIs are **bootable** - you can launch identical instances from them. Behind the scenes, AMIs create snapshots of all volumes.

**Use cases:** Instance cloning, scaling, golden images, full disaster recovery

### Why Both?
- **Snapshots:** Granular volume-level backups, faster to restore individual volumes
- **AMIs:** Complete instance backups, faster to launch full working servers

## Task 1: Manual EBS Snapshot

### What This Task Does
Learn the manual snapshot process to understand how backups work at the volume level. This gives you hands-on experience with the fundamental backup mechanism before automating it.

### Steps Completed

1. **Identified EBS Volumes**
   - Navigated to EC2 → Volumes
   - Located volumes attached to instances (State = "in-use")
   - Noted Volume IDs

2. **Created Manual Snapshot**
   - Selected volume → Actions → Create snapshot
   - Configuration:
     - Description: `manual-backup-<instance-name>-2026-01-11`
     - Tags added:
       - `Name`: `backup-<instance-name>-2026-01-11`
       - `Environment`: `production` or `dev`
       - `Backup`: `manual`

3. **Verified Snapshot**
   - Checked EC2 → Snapshots
   - Confirmed Status = "completed"
   - Noted Snapshot ID

### Key Learnings

- Snapshots are incremental (only changed blocks after first snapshot)
- Snapshots don't require stopping the instance
- Snapshots are stored separately from volumes
- First snapshot takes longer (full copy), subsequent ones are faster

---

## Task 2: Automated Snapshots with DLM

### What This Task Does
Set up Data Lifecycle Manager to automatically create and manage snapshots on a schedule. This eliminates manual work and ensures consistent backups. DLM also handles retention - automatically deleting old snapshots to control costs.

### Configuration

**DLM Lifecycle Policy Created:**
- **Policy Name:** `daily-ebs-backup-policy`
- **Target:** Volumes tagged with `Backup=true`
- **Schedule:** Daily at 07:30 UTC
- **Retention:** 2 most recent snapshots
- **Tags:** Copy tags from source volume enabled

### Steps Completed

1. **Tagged Volumes for Automation**
   - EC2 → Volumes → Selected volumes
   - Added tag: `Backup=true`

2. **Created DLM Policy**
   - EC2 → Lifecycle Manager → Create lifecycle policy
   - Policy type: EBS snapshot policy
   - Resource type: Volume
   - Configured schedule and retention

3. **Verified Policy**
   - Policy State = "Enabled" ✅
   - First snapshot scheduled for next 07:30 UTC run

### Key Benefits

- **Automation:** No manual intervention needed
- **Cost optimization:** Auto-deletes snapshots older than retention period
- **Consistency:** Runs at same time daily
- **Tag inheritance:** Snapshots automatically inherit volume tags

---

## Task 3: Create AMI from EC2 Instance

### What This Task Does
Create a complete, bootable backup of your EC2 instance. AMIs capture everything - OS, installed software, configurations, and data across all volumes. This enables you to launch identical instances for scaling, testing, or disaster recovery.

### Configuration

**AMI Created:**
- **Name:** `<instance-name>-backup-2026-01-11`
- **Description:** Backup with configurations and application code
- **Reboot:** Enabled (ensures filesystem consistency)
- **Delete on Termination:** Left as default

### Steps Completed

1. **Selected EC2 Instance**
   - EC2 → Instances → Selected target instance

2. **Created Image**
   - Actions → Image and templates → Create image
   - Configured name, description, and tags
   - Enabled reboot for consistency

3. **Verified AMI**
   - EC2 → AMIs
   - Confirmed Status = "available"
   - Noted that snapshots were created for all volumes

### Key Learnings

- AMI includes OS, configurations, and all attached EBS volumes
- AMI is bootable - can launch identical instances
- Snapshots are created automatically for each volume
- AMI reboot ensures filesystem consistency (2-minute downtime)
- AMI snapshots persist even after instance deletion

---

## Task 4: Cleanup & Cost Management

### What This Task Does
Understand the cost implications of snapshots and AMIs. Learn to identify and delete unnecessary backups to avoid accumulating storage costs. Implement retention strategies that balance data protection with cost efficiency.

### Actions Taken

- Reviewed all snapshots and AMIs
- Deleted unnecessary test snapshots
- Kept only required backups for learning
- Understood cost implications of snapshot retention

### Cost Optimization Notes

**Snapshot Costs:**
- $0.05/GB/month for incremental storage
- First snapshot: Full volume size
- Subsequent snapshots: Only changed blocks

**Best Practices Implemented:**
- DLM auto-cleanup (keeps only 2 snapshots)
- Manual snapshot deletion after testing
- Regular AMI cleanup (deregister old AMIs + delete associated snapshots)

---

## Tomorrow's Verification

**To Complete Day 3:**
- [ ] Verify DLM created automatic snapshot at 07:30 UTC
- [ ] Check snapshot appears in EC2 → Snapshots
- [ ] Confirm snapshot has DLM tags

---

## Production Best Practices Learned

### Tagging Strategy
```
Required Tags:
- Name: Descriptive identifier
- Environment: production/staging/dev
- Backup: true/false (for DLM targeting)
- CreatedBy: manual/automated/dlm
```

### Retention Strategy
```
Personal/Learning Account:
- Daily snapshots: 7 days
- AMIs: 1-2 recent versions

Production Account:
- Daily snapshots: 7 days
- Weekly snapshots: 4 weeks
- Monthly snapshots: 12 months
- AMIs: Golden images only
```

### Recovery Capabilities
- **EBS Snapshots:** Restore individual volumes
- **AMI:** Launch complete, identical instances
- **Cross-region copies:** For disaster recovery (not configured yet)

---

## Key Commands Reference

### AWS CLI - Manual Snapshot
```bash
# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-xxxxx \
  --description "manual-backup-2026-01-11" \
  --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=backup-2026-01-11}]'

# List snapshots
aws ec2 describe-snapshots --owner-ids self
```

### AWS CLI - Create AMI
```bash
# Create AMI
aws ec2 create-image \
  --instance-id i-xxxxx \
  --name "instance-backup-2026-01-11" \
  --description "Backup with app configuration" \
  --no-reboot  # or remove for reboot

# List AMIs
aws ec2 describe-images --owners self
```

## Resources & References

- [AWS EBS Snapshots Documentation](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html)
- [AWS Data Lifecycle Manager](https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-lifecycle.html)
- [AWS AMI Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)

---

## Notes & Observations

- Snapshots completed successfully without instance downtime
- DLM setup was straightforward with tag-based targeting
- AMI creation with reboot ensured data consistency
- Understanding incremental snapshots helps with cost planning
- Proper tagging is crucial for automation and organization

---