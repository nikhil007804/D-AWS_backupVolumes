# AWS EBS Volume Backup Automation with Terraform & GitHub Actions

This project automates daily backups of an AWS EBS volume using Terraform and GitHub Actions.  
It provisions all necessary AWS resources and schedules backups at 3 AM UTC, retaining each backup for 30 days.

---

## Prerequisites

- **An existing EBS volume** in your AWS account.  
  If you do not have one, see [How to Create an EBS Volume](#how-to-create-an-ebs-volume).
- AWS credentials (Access Key ID and Secret Access Key) with permissions to manage IAM, EC2, and AWS Backup.
- [Terraform](https://www.terraform.io/downloads.html) installed locally (for initial setup/testing).
- A GitHub repository for this code.

---

## How to Create an EBS Volume

**Via AWS Console:**
1. Go to the [EC2 Dashboard](https://console.aws.amazon.com/ec2/).
2. In the left menu, click **Volumes** under **Elastic Block Store**.
3. Click **Create Volume**.
4. Choose size, type, and availability zone, then click **Create Volume**.
5. Note the **Volume ID** (e.g., `vol-0efe9d5419d6128e7`).

**Via AWS CLI:**
```sh
aws ec2 create-volume --size 8 --region us-east-1 --availability-zone us-east-1a --volume-type gp2
```
- Note the `VolumeId` in the output.

**Get the ARN for your volume:**
1. Find your AWS Account ID (top right in AWS Console or run:  
   `aws sts get-caller-identity --query Account --output text`)
2. Construct the ARN:  
   ```
   arn:aws:ec2:us-east-1:YOUR_ACCOUNT_ID:volume/VOLUME_ID
   ```
   Example:  
   ```
   arn:aws:ec2:us-east-1:1234571473:volume/vol-0efe9d5419d6128e7
   ```

---

## How This Works

### Terraform Code

- **IAM Role**: Allows AWS Backup to manage backups on your behalf.
- **Backup Vault**: Secure storage for your backups.
- **Backup Plan**: Schedules daily backups at 3 AM UTC, keeps each for 30 days.
- **Backup Selection**: Specifies which EBS volume to back up (using the ARN you provide).

### What Gets Created

- An IAM role with the correct permissions for AWS Backup.
- A backup vault named as per your configuration.
- A backup plan and rule for daily backups.
- A backup selection that targets your specified EBS volume.

---

## How to Use

### 1. Clone this repository

```sh
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

### 2. Set the EBS Volume ARN and (optionally) resource names

Edit `terraform.tfvars`:

```hcl
ebs_volume_arn    = "arn:aws:ec2:us-east-1:515517371473:volume/vol-0efe9d5419d6128e7"
backup_vault_name = "pratyusha-backup-vault"      # Optional, can change
backup_plan_name  = "pratyusha-daily-backup-plan" # Optional, can change
backup_rule_name  = "daily-backup"                # Optional, can change
```

### 3. Initialize and Apply Terraform (optional, for local testing)

```sh
terraform init
terraform plan
terraform apply
```

### 4. Set Up GitHub Actions for Automation

- Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions**.
- Add your AWS credentials as secrets:
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
- The workflow in `.github/workflows/deploy.yml` will automatically run `terraform init`, `plan`, and `apply` on every push to `main`.

---

## Where to Check in AWS Console

- **IAM Role**: [IAM Console → Roles](https://console.aws.amazon.com/iam/) (`pratyusha-backup-role`)
- **Backup Vault**: [AWS Backup → Backup vaults](https://console.aws.amazon.com/backup/)
- **Backup Plan**: [AWS Backup → Backup plans](https://console.aws.amazon.com/backup/)
- **Backup Selection**: In your backup plan details, under "Resource assignments"
- **Backups**: [AWS Backup → Backup vaults → Your vault → Recovery points](https://console.aws.amazon.com/backup/)

---

## Notes

- If you do not have an EBS volume, **create one first** as described above.
- The backup will run automatically at 3 AM UTC every day.
- You can change the backup schedule or retention in `main.tf` if needed.

---

## Troubleshooting

- Make sure your AWS credentials have the necessary permissions.
- If you change the EBS volume, update `terraform.tfvars` and re-apply.
- Check GitHub Actions logs for any automation errors.

---
