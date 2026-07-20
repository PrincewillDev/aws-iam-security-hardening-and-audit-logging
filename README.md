# AWS IAM Security Hardening and Audit Logging (CLI-Based)

## Overview

This project demonstrates the implementation of core AWS security and governance practices using the AWS CLI. It focuses on identity and access management, cost governance, audit logging, and credential security while following the principle of least privilege.

The project was completed as a hands-on cloud security exercise and is fully documented with configuration files, CLI commands, implementation notes, and supporting evidence.

## Objectives

* Configure and authenticate the AWS CLI
* Create AWS Budgets with billing alerts
* Implement least-privilege IAM permissions
* Configure CloudTrail audit logging
* Simulate and detect a credential leak
* Design an IAM policy for a three-tier web application

## Technologies

* AWS CLI
* AWS IAM
* AWS Budgets
* Amazon S3
* AWS CloudTrail
* Git
* Gitleaks

## Project Structure

```text
.
├── budgets-config/
├── cli/
├── docs/
├── evidence/
├── policies/
├── screenshots/
└── README.md
```

## Progress

* [x] AWS CLI Configuration
* [x] AWS Budgets and Billing Alerts
* [ ] Least-Privilege IAM Role
* [ ] CloudTrail Data Event Logging
* [ ] Credential Leak Detection
* [ ] IAM Policy Design

## Repository Contents

* **budgets-config/** — Budget and notification configuration files.
* **cli/** — AWS CLI commands and implementation steps.
* **docs/** — Project documentation and design decisions.
* **policies/** — IAM and trust policy JSON files.
* **evidence/** — Command outputs and validation results.
* **screenshots/** — Visual evidence of completed tasks.

## Key Learning Outcomes

* Cost governance using AWS Budgets
* Principle of least privilege
* IAM role and policy design
* Cloud audit logging
* Secure credential management
* CLI-driven infrastructure administration

## License

This project is intended for educational and portfolio purposes.
