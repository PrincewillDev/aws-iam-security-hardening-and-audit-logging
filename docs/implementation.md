## Task 1 — Configure AWS Budgets

### Goal

Configure monthly AWS Budgets and billing alerts entirely through the AWS CLI.

### Implementation

- Created a $5 monthly budget.
- Created a $10 monthly budget.
- Configured email notifications.
- Verified using the AWS CLI.

### Outcome

Both budgets and notifications were successfully created and validated.

## Task 2 — Implement Least-Privilege IAM

### Goal

Create an IAM user with the minimum permissions required to upload objects to a specific Amazon S3 bucket and verify that unauthorized actions are denied.

### Implementation

* Created an S3 bucket for testing IAM permissions.
* Created a dedicated IAM user (`s3-writer-user`).
* Created a custom IAM policy granting only `s3:PutObject` permissions on the target S3 bucket.
* Attached the policy to the IAM user.
* Generated programmatic access credentials and configured a separate AWS CLI profile.
* Verified that object uploads succeeded.
* Verified that unauthorized actions such as listing buckets and deleting objects were denied.

### Outcome

The least-privilege IAM user successfully uploaded objects to the designated S3 bucket while all unauthorized operations returned `AccessDenied`, confirming that the policy enforced the intended permissions.

---

## Task 3 — Configure CloudTrail Audit Logging

### Goal

Configure AWS CloudTrail to record S3 object-level activity, trigger a security event, and verify that CloudTrail captured the resulting audit logs.

### Implementation

* Created an S3 bucket to store CloudTrail logs.
* Configured the required S3 bucket policy allowing the CloudTrail service to write log files.
* Created and started a CloudTrail trail using the AWS CLI.
* Enabled S3 Data Event logging for the project bucket using Advanced Event Selectors.
* Verified that CloudTrail was actively logging and that the event selectors were successfully applied.
* Performed a successful `PutObject` operation using the restricted IAM user.
* Triggered unauthorized S3 operations that resulted in `AccessDenied`.
* Verified CloudTrail audit records for both management and S3 data events.

### Outcome

CloudTrail was successfully configured to capture both management events and S3 object-level data events. The audit logs recorded successful object uploads as well as denied access attempts, demonstrating effective monitoring of S3 activity and validating the CloudTrail configuration.
