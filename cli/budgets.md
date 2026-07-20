# Task 1 - AWS Budgets and Billing Alerts

## Objective

Create monthly AWS Budgets and configure email notifications using the AWS CLI.

---

## Prerequisites

* AWS CLI installed
* AWS CLI configured
* IAM user with permission to manage AWS Budgets
* IAM access to Billing enabled from the root account

---

## 1. Verify AWS CLI Configuration

```bash
aws sts get-caller-identity
```

Expected Output

```json
{
  "UserId": "AIDA****************",
  "Account": "<AWS_ACCOUNT_ID>",
  "Arn": "arn:aws:iam::<AWS_ACCOUNT_ID>:user/<IAM_USER_NAME>"
}
```

---

## 2. Create the $5 Monthly Budget

```bash
aws budgets create-budget \
    --account-id <AWS_ACCOUNT_ID> \
    --budget file://budgets-config/budget-5.json
```

Example

```bash
aws budgets create-budget \
    --account-id 123456789012 \
    --budget file://budgets-config/budget-5.json
```

---

## 3. Create the $10 Monthly Budget

```bash
aws budgets create-budget \
    --account-id <AWS_ACCOUNT_ID> \
    --budget file://budgets-config/budget-10.json
```

Example

```bash
aws budgets create-budget \
    --account-id 123456789012 \
    --budget file://budgets-config/budget-10.json
```

---

## 4. Configure Billing Alert for the $5 Budget

```bash
aws budgets create-notification \
    --account-id <AWS_ACCOUNT_ID> \
    --budget-name Budget-5USD \
    --notification file://budgets-config/notification.json \
    --subscribers file://budgets-config/subscriber.json
```

Example

```bash
aws budgets create-notification \
    --account-id 123456789012 \
    --budget-name Budget-5USD \
    --notification file://budgets-config/notification.json \
    --subscribers file://budgets-config/subscriber.json
```

---

## 5. Configure Billing Alert for the $10 Budget

```bash
aws budgets create-notification \
    --account-id <AWS_ACCOUNT_ID> \
    --budget-name Budget-10USD \
    --notification file://budgets-config/notification.json \
    --subscribers file://budgets-config/subscriber.json
```

Example

```bash
aws budgets create-notification \
    --account-id 123456789012 \
    --budget-name Budget-10USD \
    --notification file://budgets-config/notification.json \
    --subscribers file://budgets-config/subscriber.json
```

---

## 6. Verify the Budgets

```bash
aws budgets describe-budgets \
    --account-id <AWS_ACCOUNT_ID>
```

Example

```bash
aws budgets describe-budgets \
    --account-id 123456789012
```

---

## 7. Verify Budget Notifications

For the $5 budget:

```bash
aws budgets describe-notifications-for-budget \
    --account-id <AWS_ACCOUNT_ID> \
    --budget-name Budget-5USD
```

For the $10 budget:

```bash
aws budgets describe-notifications-for-budget \
    --account-id <AWS_ACCOUNT_ID> \
    --budget-name Budget-10USD
```

---

## 8. Verify Notification Subscribers

For the $5 budget:

```bash
aws budgets describe-subscribers-for-notification \
    --account-id <AWS_ACCOUNT_ID> \
    --budget-name Budget-5USD \
    --notification file://budgets-config/notification.json
```

For the $10 budget:

```bash
aws budgets describe-subscribers-for-notification \
    --account-id <AWS_ACCOUNT_ID> \
    --budget-name Budget-10USD \
    --notification file://budgets-config/notification.json
```

---

## Files Used

```text
budgets-config/
├── budget-5.json
├── budget-10.json
├── notification.json
└── subscriber.json
```

---

## Expected Result

* AWS CLI successfully authenticated.
* Monthly budgets of $5 and $10 created.
* Email notifications attached to both budgets.
* Budgets and notifications verified using AWS CLI.
