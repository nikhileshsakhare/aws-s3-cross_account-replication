# Overview

This project demonstrates how to configure cross-account replication between two Amazon S3 buckets using the AWS Management Console. It walks through creating versioned source and destination buckets, setting up an S3 replication rule with an IAM service role, and applying a destination bucket policy to securely allow replicated objects from another AWS account.

## Project goals

- Enable cross-account S3 replication from a source bucket in Account A to a destination bucket in Account B.
- Ensure versioning is enabled on both buckets to support replication.
- Configure an S3-managed IAM role that S3 uses to read from the source and write to the destination.
- Add a destination bucket policy that allows the replication role to perform required S3 actions.

## Architecture

- **Source account (Account A)**  
  - Region: `us-east-1` (US East, N. Virginia).
  - Bucket: `source-rep-nik` (general purpose, versioning enabled).
  - S3 replication rule configured on this bucket.

- **Destination account (Account B)**  
  - Region: `us-east-1` (same as source).
  - Bucket: `dist-rep-nik` (general purpose, versioning enabled).
  - Bucket policy granting the replication role permissions to replicate objects.

- **IAM service role**  
  - Automatically created by S3 when the replication rule is saved, with a name like `s3crr_role_for_source-rep-nik`.
  - Trusted by S3 and allowed to read objects from the source bucket and write replicas into the destination bucket.

## Skills and concepts demonstrated

- **Amazon S3:** buckets, versioning, cross-account replication, replication rules, bucket policies.
- **AWS security:** least-privilege access via bucket policies and IAM service roles for S3 replication.
- **Cross-account access:** granting one account’s S3 role permissions to write objects into another account’s bucket.
- AWS Management Console navigation and configuration for S3 and IAM.

## Prerequisites

- Two AWS accounts (Account A as source, Account B as destination).
- Permission to create S3 buckets and configure replication in both accounts.
- Basic understanding of S3 versioning, IAM roles, and bucket policies.

## Step-by-step implementation

### 1. Create the source bucket (Account A)

1. Sign in to the AWS Management Console using Account A and open **Amazon S3**.
2. Click **Create bucket** and choose:  
   - Bucket type: **General purpose**.  
   - Region: **US East (N. Virginia) – us-east-1**.  
   - Bucket name: for example, `source-rep-nik` (must be globally unique).
3. Leave **Block Public Access** enabled.
4. Scroll down, enable **Bucket Versioning**, and create the bucket.

At this point, Account A has a versioned bucket `source-rep-nik` that will act as the replication source.

### 2. Create the destination bucket (Account B)

1. Sign in to the AWS Management Console using Account B and open **Amazon S3**.
2. Click **Create bucket** and configure:  
   - Bucket type: **General purpose**.  
   - Region: **US East (N. Virginia) – us-east-1** (same region as the source).  
   - Bucket name: for example, `dist-rep-nik`.
3. Leave **Block Public Access** enabled and create the bucket.

Now Account B has a general purpose destination bucket `dist-rep-nik` in the same region as the source.

### 3. Configure the replication rule on the source bucket

In Account A:

1. Open **Amazon S3**, go to the **Buckets** list, and select `source-rep-nik`.
2. Navigate to the **Management** tab and locate the **Replication rules** section (initially empty).
3. Click **Create replication rule**.
4. Under **Replication rule configuration**:  
   - Rule name: e.g., `different-account`.
   - Status: **Enabled**.
   - Priority: `0`.
5. Under **Source bucket**, confirm `source-rep-nik` and select **Apply to all objects in the bucket** so the entire bucket is replicated.

### 4. Choose the destination bucket and account

Still in the replication rule wizard in Account A:

1. In the **Destination** section, select **Specify a bucket in another account**.
2. Provide:  
   - **Account ID**: the numeric AWS account ID for Account B.
   - **Bucket name**: `dist-rep-nik`.
   - **Destination Region**: automatically set to `us-east-1`.
3. Enable **Change object ownership to destination bucket owner** so replicated objects are fully owned and managed by Account B.

This ensures that even if the source account does not own the original object, the replica in the destination bucket is owned by the destination account.

### 5. Create the IAM role for S3 replication

1. In the **IAM role** section of the replication rule configuration, choose **Create new role**.
2. S3 will automatically create a service role (for example, `s3crr_role_for_source-rep-nik`) with permissions to:  
   - Read from the source bucket.  
   - Write replicated objects to the destination bucket.
3. Save the replication rule.

Now the replication configuration shows:  
- Rule name: `different-account`.  
- Status: **Enabled**.  
- Destination: `s3://dist-rep-nik`.  
- Region: `us-east-1`.  
- Scope: **Entire bucket**.  
- Replica owner: **Destination bucket owner**.  
- IAM role: `s3crr_role_for_source-rep-nik`.

### 6. Create the destination bucket policy

In Account B, the destination bucket must trust the replication role from Account A to write objects:

1. Open the **AWS Policy Generator** and choose policy type **S3 Bucket Policy**.
2. Add a statement with:  
   - **Effect**: Allow.  
   - **Principal**: the ARN of the replication role from Account A, e.g.  
     `arn:aws:iam::<SOURCE_ACCOUNT_ID>:role/service-role/s3crr_role_for_source-rep-nik`.
   - **Actions**:  
     - `s3:ReplicateDelete`  
     - `s3:ReplicateObject`  
     - `s3:GetObject`  
     - `s3:PutObject`
   - **Resource**: `arn:aws:s3:::dist-rep-nik` (or including object keys if desired).
3. Generate the policy JSON and copy it.
4. In Account B, open the `dist-rep-nik` bucket, go to **Permissions → Bucket policy**, and paste the generated JSON. Save the policy.

This bucket policy allows the S3 replication role from Account A to replicate objects into the destination bucket in Account B.

## Verification and testing

- Upload a test object into `source-rep-nik` in Account A and confirm that a replica appears in `dist-rep-nik` in Account B.
- Check the **Replication rules** section to ensure the rule remains in **Enabled** status.
- Inspect the IAM role `s3crr_role_for_source-rep-nik` to see its trust policy and permissions for replication.
