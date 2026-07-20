# Lessons Learned

This document captures key technical concepts, implementation challenges, troubleshooting steps, and best practices discovered while completing this project.

---

# Task 1 - AWS Budgets and Billing Alerts

## Key Concepts

* AWS Budgets define spending limits for an AWS account.
* Billing alerts are notifications attached to a budget.
* Budgets and notifications are separate AWS resources.
* AWS Budgets can be managed entirely through the AWS CLI.

## Challenges Encountered

### Challenge 1

**Issue**

The AWS CLI returned an "Unknown options" error when creating a budget.

**Cause**

The budget file path was referenced incorrectly using `file:` instead of `file://`.

**Resolution**

Used the correct syntax:

```text
file://budgets-config/budget-5.json
```

---

### Challenge 2

**Issue**

The notification JSON failed validation.

**Cause**

The JSON object contained an unnecessary top-level `"Notification"` property.

**Resolution**

Passed the notification object directly, matching the AWS CLI parameter requirements.

---

### Challenge 3

**Issue**

The subscriber JSON could not be parsed.

**Cause**

The JSON structure was invalid.

**Resolution**

Corrected the subscriber JSON to the expected array/object structure required by the AWS CLI.

---

## Best Practices Learned

* Configure the AWS CLI before creating resources.
* Use relative paths with `file://` for better portability.
* Store AWS configuration files separately from documentation.
* Verify every resource immediately after creation.
* Keep infrastructure configuration under version control.

---

## Verification Commands

```bash
aws budgets describe-budgets --account-id <AWS_ACCOUNT_ID>
```

```bash
aws budgets describe-notifications-for-budget --account-id <AWS_ACCOUNT_ID> --budget-name Budget-5USD
```

---

## Takeaways

* AWS Budgets and billing alerts work together but are created independently.
* Reading AWS CLI error messages carefully often reveals the exact issue.
* Infrastructure becomes easier to reproduce when configuration is stored as code.
