# Stage 1: Create or Select a Learning Account

## Objective
Prepare an authorized AWS learning environment and confirm which identity is being used.

## What I did
1. Created a new standalone AWS account to use only for this lab, using a dedicated email address that is not linked to an AWS Builder ID profile.
2. Completed the standard sign-up: email verification, root password, contact details, payment method and phone verification.
3. Selected the **Basic (free)** support plan.
4. Signed in to the console as the **root user** and chose a single working region, **eu-north-1 (Stockholm)**.
5. Stored the root password in a password manager. It is unique to this account.
6. Confirmed my identity from CloudShell (see below).

## Identity check
Command run in AWS CloudShell:

```bash
aws sts get-caller-identity
```

Result: the returned `Arn` ends in `:root`, which confirms I am signed in as the root user of this learning account. The account ID and user ID are redacted in the screenshot.

## Note on a first attempt
My first sign-up went through the AWS Builder ID flow. That created an account accessed through a role (`AccountFullAccessRole`) with no root password. It does not match the root-user model this lab teaches, so I created a second account through the standard sign-up and used it for the lab. The first account was not used for any lab resources.

## Controls and decisions
| Control | Status |
|---|---|
| Account used only for this lab | Yes |
| Unique password in a password manager | Yes |
| Root used only for tasks that require it | Yes. Root will be locked down in Stage 2 and replaced by IAM identities from Stage 3 |
| Root access keys | None created during setup (to be verified in Stage 2) |
| Account ID, email and billing data in this repo | None. All redacted |

## Evidence
- `screenshots/01-account-setup-console-home.png`: console home page, account ID and email blurred
- `screenshots/01-account-setup-caller-identity-root.png`: CloudShell output showing the `Arn` ending in `:root`, IDs blurred

## What I learned
- An AWS Builder ID is a personal profile for training and tools. It is not an AWS account and does not by itself provide a root user.
- Checking `aws sts get-caller-identity` is a reliable way to confirm which identity is acting, so I do not have to guess from the console.
- Because the root user cannot be limited by IAM policies, the account's first priority is to protect it (Stage 2) and then stop using it for daily work (Stage 3).
