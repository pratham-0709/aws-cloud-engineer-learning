# Day 2: AWS Backup for RDS

## Date Completed
January 17, 2026

## Objective
Implement automated, centralized backup solution for RDS MySQL database with lifecycle management and disaster recovery capabilities.

## What is AWS Backup?
AWS Backup is a fully-managed service that makes it easy to centralize and automate data protection across AWS services, in the cloud, and on premises. Using this service, you can configure backup policies and monitor activity for your AWS resources in one place. It allows you to automate and consolidate backup tasks that were previously performed service-by-service, and removes the need to create custom scripts and manual processes. With a few clicks in the AWS Backup console, you can automate your data protection policies and schedules.

#### Note:- AWS backup is similar to Velero Both do automated backup/restore with schedules and retention. Different: AWS Backup = infrastructure level, Velero = application/cluster level.

## Architecture
```
RDS MySQL Instance (Public - Dev/Testing)
    ↓
AWS Backup (automated daily snapshots)
    ↓
Backup Vault: dev-backup-vault
    ├─ Retention: 35 days
    ├─ Storage: Warm (RDS doesn't support cold in ap-south-1)
    └─ Auto-delete after retention period
```

## Implementation Steps

### 1. Create Backup Vault
```bash
Vault name: dev-backup-vault
Encryption: AWS managed key (default)
Tags:
  Environment: Development
  Cluster: Te-Cluster
```

**Why separate vault?**
- Isolated from production resources
- Can't accidentally delete backups with resources
- Different access policies for security team

### 2. Create Backup Plan
```json
{
    "templateRules": [
        {
            "ruleName": "dev-backup-rule",
            "scheduleExpression": "cron(0 2 ? * * *)",
            "scheduleExpressionTimezone": "Asia/Kolkata",
            "startWindowMinutes": 60,
            "completionWindowMinutes": 120,
            "lifecycle": {
                "toDeletedAfterDays": 35
            }
        }
    ],
}
```
**Configuration explained:**
- **Schedule:** Daily at 2:00 AM IST (low-traffic time)
- **Start window:** 1 hour flexibility (2:00-3:00 AM)
- **Completion window:** Must finish within 2 hours
- **Retention:** 35 days, then auto-delete
- **Point-in-time recovery:** Disabled (cost optimization)

### 3. Assign Resources
```yaml
Assignment name: dev-rds-db
IAM role: AWSBackupDefaultServiceRole (auto-created)
Selection method: Specific resource
Resource type: RDS
Database: dev-db
```
**Why specific resource vs. tags?**
- Single RDS instance → specific selection is simpler
- Tags better for dynamic environments (multiple DBs)


### 4. Test On-Demand Backup
```yaml
Backup type: On-demand
Retention: 5 days (test backup)
Vault: dev-backup-vault
Status: Running → Completed ✅
```

## Verification

### Check Backup Jobs
```bash
# Via Console
AWS Backup → Jobs → Backup jobs → Status: Completed

# Via CLI
aws backup list-backup-jobs --by-backup-vault-name dev-backup-vault
```

## Key Learnings

### Cold Storage Not Available
**Region limitation:** ap-south-1 doesn't support cold storage for RDS backups

**Impact:**
- All backups stay in warm storage ($0.095/GB/month)
- Can't reduce costs with cold storage transition
- Alternative: Use us-east-1 or eu-west-1 for cold storage support

## Cost Analysis

### Monthly Cost (20GB MySQL DB)
```
Warm storage: 20GB × $0.095/GB/month = $1.90/month
Retention: 35 days (rolling window)
Total: ~$1.90/month
Annual: ~$22.80/year
```

**Cost optimization:**
- Reduce retention to 7 days → $0.54/month
- Use cold storage (if available) → 70% savings after 30 days

## Production Enhancements

### Multi-Region Copy (DR)
```
Primary Backup (ap-south-1)
    ↓
Copy to us-east-1 (disaster recovery)
    ↓
Cost: 2x storage + cross-region transfer
```

### Automated Testing
```
Weekly Lambda function:
1. Restore backup to test instance
2. Run validation queries
3. Verify data integrity
4. Delete test instance
5. Report results
```

### Compliance Tags
```yaml
Tags:
  Compliance: PCI-DSS
  DataClassification: Sensitive
  RetentionReason: Regulatory
  Owner: Platform-Team
```
## Disaster Recovery Procedures

### RTO (Recovery Time Objective)
**Target:** 1 hour
- Restore from backup: ~30-45 minutes
- Update application config: ~10 minutes
- Validation: ~5 minutes

### RPO (Recovery Point Objective)
**Current:** 24 hours (daily backups)
- Acceptable data loss: Up to 24 hours
- For critical production: Enable PITR (5-minute RPO)

### Recovery Steps
1. AWS Backup → Backup vaults → dev-backup-vault
2. Select latest recovery point
3. Click "Restore"
4. Configure:
   - Instance identifier: `<original>-restored`
   - Instance class: Same as original
   - VPC/Subnet: Same or different (for testing)
5. Wait 30-45 minutes for restore
6. Update application connection string
7. Verify data integrity

## Troubleshooting

### Backup Job Failed
**Common causes:**
- DB not in "Available" state
- Insufficient IAM permissions
- Database locked/in use

**Solution:**
```bash
# Check DB status
aws rds describe-db-instances --db-instance-identifier my-mysql-db

# Check backup job details
aws backup describe-backup-job --backup-job-id 
```

### Restore Failed
**Check:**
- Sufficient VPC IP addresses available?
- Security groups configured correctly?
- Parameter group compatible?

## References
- [AWS Backup Documentation](https://docs.aws.amazon.com/aws-backup/)
- [RDS Backup Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_CommonTasks.BackupRestore.html)