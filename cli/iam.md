# Task 2: Least-Privilege IAM User

## Objective

Create an IAM user with permission to upload objects to one specific S3 bucket and no permission to read IAM policies.

## Placeholders

Replace these values before running the commands:

```text
<AWS_ACCOUNT_ID>     AWS account ID, for example 123456789012
<IAM_USER_NAME>      IAM user name, for example s3-writer-user
<POLICY_NAME>        Policy name, for example S3WriteOnlyPolicy
<BUCKET_NAME>        Target S3 bucket name
<AWS_REGION>         AWS Region, for example us-east-1
<PROFILE_NAME>       CLI profile name, for example s3-writer
```

## 1. Confirm the Administrator Identity

```bash
aws sts get-caller-identity
```

This confirms which AWS identity is currently being used before creating IAM resources.

## 2. Confirm the S3 Bucket Exists

```bash
aws s3api head-bucket \
    --bucket <BUCKET_NAME>
```

No output usually means the bucket exists and is accessible.

Alternatively:

```bash
aws s3 ls
```

## 3. Create the IAM User

```bash
aws iam create-user \
    --user-name <IAM_USER_NAME>
```

Example:

```bash
aws iam create-user \
    --user-name s3-writer-user
```

## 4. Verify the IAM User

```bash
aws iam get-user \
    --user-name <IAM_USER_NAME>
```

Example:

```bash
aws iam get-user \
    --user-name s3-writer-user
```

## 5. Create the Policy Document

Create the following file:

```text
policies/s3-write-policy.json
```

Add:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowWriteToProjectBucket",
            "Effect": "Allow",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<BUCKET_NAME>/*"
        }
    ]
}
```

Replace `<BUCKET_NAME>` with the actual bucket name.

Example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowWriteToProjectBucket",
            "Effect": "Allow",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::princewill-storage/*"
        }
    ]
}
```

The `/*` means the permission applies to objects stored inside the bucket.

## 6. Create the Customer-Managed Policy

```bash
aws iam create-policy \
    --policy-name <POLICY_NAME> \
    --description "Allows uploads to one specific S3 bucket only" \
    --policy-document file://policies/s3-write-policy.json
```

Example:

```bash
aws iam create-policy \
    --policy-name S3WriteOnlyPolicy \
    --description "Allows uploads to one specific S3 bucket only" \
    --policy-document file://policies/s3-write-policy.json
```

Save the policy ARN returned by AWS.

Its format will be:

```text
arn:aws:iam::<AWS_ACCOUNT_ID>:policy/<POLICY_NAME>
```

## 7. Attach the Policy to the IAM User

```bash
aws iam attach-user-policy \
    --user-name <IAM_USER_NAME> \
    --policy-arn arn:aws:iam::<AWS_ACCOUNT_ID>:policy/<POLICY_NAME>
```

Example:

```bash
aws iam attach-user-policy \
    --user-name s3-writer-user \
    --policy-arn arn:aws:iam::123456789012:policy/S3WriteOnlyPolicy
```

## 8. Verify the Attached Policy

```bash
aws iam list-attached-user-policies \
    --user-name <IAM_USER_NAME>
```

Example:

```bash
aws iam list-attached-user-policies \
    --user-name s3-writer-user
```

Expected result:

```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "S3WriteOnlyPolicy",
            "PolicyArn": "arn:aws:iam::<AWS_ACCOUNT_ID>:policy/S3WriteOnlyPolicy"
        }
    ]
}
```

## 9. Create an Access Key

```bash
aws iam create-access-key \
    --user-name <IAM_USER_NAME>
```

Example:

```bash
aws iam create-access-key \
    --user-name s3-writer-user
```

The command returns:

```text
AccessKeyId
SecretAccessKey
```

Save the secret access key securely because AWS displays it only when the key is created.

Do not save credentials in this repository.

## 10. Configure a Separate CLI Profile

```bash
aws configure --profile <PROFILE_NAME>
```

Example:

```bash
aws configure --profile s3-writer
```

Enter the IAM user's:

```text
AWS Access Key ID: <ACCESS_KEY_ID>
AWS Secret Access Key: <SECRET_ACCESS_KEY>
Default region name: <AWS_REGION>
Default output format: json
```

## 11. Confirm the IAM User Profile

```bash
aws sts get-caller-identity \
    --profile <PROFILE_NAME>
```

Example:

```bash
aws sts get-caller-identity \
    --profile s3-writer
```

The returned ARN should identify the restricted IAM user:

```text
arn:aws:iam::<AWS_ACCOUNT_ID>:user/<IAM_USER_NAME>
```

## 12. Create a Test File

For Git Bash, Linux or macOS:

```bash
echo "Least-privilege S3 upload test" > test-upload.txt
```

For Windows Command Prompt:

```cmd
echo Least-privilege S3 upload test > test-upload.txt
```

For PowerShell:

```powershell
"Least-privilege S3 upload test" | Out-File test-upload.txt
```

## 13. Test the Allowed S3 Upload

```bash
aws s3 cp test-upload.txt \
    s3://<BUCKET_NAME>/test-upload.txt \
    --profile <PROFILE_NAME> \
    --region <AWS_REGION>
```

Example:

```bash
aws s3 cp test-upload.txt \
    s3://princewill-storage/test-upload.txt \
    --profile s3-writer \
    --region us-east-1
```

Expected result:

```text
upload: ./test-upload.txt to s3://<BUCKET_NAME>/test-upload.txt
```

This confirms that the IAM user can write an object to the selected bucket.

## 14. Test Access to Another Bucket

Attempt to upload to a different bucket:

```bash
aws s3 cp test-upload.txt \
    s3://<DIFFERENT_BUCKET_NAME>/test-upload.txt \
    --profile <PROFILE_NAME>
```

Expected result:

```text
AccessDenied
```

This confirms that the permission is limited to the specified bucket.

## 15. Verify IAM Access Is Denied

Attempt to list IAM policies:

```bash
aws iam list-policies \
    --profile <PROFILE_NAME>
```

Expected result:

```text
AccessDenied
```

The error should indicate that the IAM user is not authorized to perform:

```text
iam:ListPolicies
```

You can perform an additional denied test:

```bash
aws iam list-users \
    --profile <PROFILE_NAME>
```

Expected result:

```text
AccessDenied
```

## 16. Optional Negative S3 Tests

The policy permits uploads but does not permit listing, downloading or deleting objects.

### Attempt to list the bucket

```bash
aws s3 ls s3://<BUCKET_NAME> \
    --profile <PROFILE_NAME>
```

Expected result:

```text
AccessDenied
```

### Attempt to download the object

```bash
aws s3 cp \
    s3://<BUCKET_NAME>/test-upload.txt \
    downloaded-test.txt \
    --profile <PROFILE_NAME>
```

Expected result:

```text
AccessDenied
```

### Attempt to delete the object

```bash
aws s3 rm \
    s3://<BUCKET_NAME>/test-upload.txt \
    --profile <PROFILE_NAME>
```

Expected result:

```text
AccessDenied
```

## Verification Summary

| Test                                 | Expected result |
| ------------------------------------ | --------------- |
| Upload object to the approved bucket | Successful      |
| Upload object to another bucket      | Access denied   |
| List objects in the approved bucket  | Access denied   |
| Download an object                   | Access denied   |
| Delete an object                     | Access denied   |
| List IAM policies                    | Access denied   |
| List IAM users                       | Access denied   |

## Security Notes

* The IAM user receives only `s3:PutObject`.
* Access is restricted to objects inside one named S3 bucket.
* No IAM permissions are granted.
* No read, list or delete permissions are granted.
* The administrator profile should be used for resource creation.
* The restricted profile should be used only for permission testing.
* Access keys and secret keys must never be committed to Git.

## Evidence to Capture

Save evidence such as:

```text
screenshots/
├── 06-iam-user-created.png
├── 07-policy-created.png
├── 08-policy-attached.png
├── 09-restricted-profile-identity.png
├── 10-s3-upload-success.png
├── 11-iam-list-policies-denied.png
└── 12-s3-read-access-denied.png
```

Do not expose access keys, secret keys, email addresses or other sensitive information in screenshots.

## Expected Outcome

* The IAM user is created successfully.
* A customer-managed policy allows only `s3:PutObject`.
* The policy is restricted to one S3 bucket.
* The IAM user can upload an object to the approved bucket.
* The IAM user cannot read IAM policies.
* Unnecessary S3 actions are denied.
