# Stage 3: IAM, Least Privilege, and Access Auditing

## Objective
Review who and what has access to the account, review example IAM policies, and explain how least privilege applies.

## Concepts I used
- **Identity:** a user or role making AWS requests.
- **Policy:** JSON permissions that allow or deny actions on resources, optionally under conditions.
- **Role:** an identity assumed by a trusted principal; it supplies temporary credentials.
- **Trust policy:** controls who or what can assume a role.
- **Least privilege:** only the actions, resources and conditions required for the task.

## Part A: Access review
I opened the IAM dashboard and the Roles page in my learning account while signed in as the root user.

| Item | Finding |
|---|---|
| Users | 0 |
| User groups | 0 |
| Roles | 2, both AWS service-linked roles created automatically at account creation: `AWSServiceRoleForSupport` and `AWSServiceRoleForTrustedAdvisor` |
| Customer-managed policies | 0 |
| Identity providers | 0 |
| Security recommendations | Root user has MFA; root user has no active access keys |

Each role has one AWS-managed permissions policy and shows no activity yet. Neither carries administrator-level access. The access review is recorded in `access-review-template.csv` (root user plus the two roles).

**Main gap found:** root is currently the only sign-in identity. It cannot be restricted by IAM policies, so it should not be used for routine work. The remediation is to create a protected IAM identity for everyday administration, which is planned after this review.

## Role trust policy review
I reviewed the trust policy of `AWSServiceRoleForTrustedAdvisor`:
- **Principal:** only the service `trustedadvisor.amazonaws.com`
- **Action:** `sts:AssumeRole`
- **Effect:** Allow

No users, other AWS accounts or wildcards are trusted. This is a service-linked role whose trust is managed by AWS, so no change is needed.

I have not yet reviewed the trust policy of `AWSServiceRoleForSupport`.

## Part B: Example policy review
Full explanations are in `policy-review.md`.

1. **`s3-report-reader.json`** (least-privilege example):
   - Allows `s3:ListBucket` on one named bucket, limited by condition to the `reports` prefix.
   - Allows `s3:GetObject` only on objects under `reports/`.
   - Does not allow writing, deleting, other buckets, wildcard actions, or permission changes.
2. **`s3-security-auditor-readonly.json`** (illustrative read-only audit example):
   - Allows only List and Get actions for IAM and S3 configuration.
   - Cannot read or list the contents of any bucket, and cannot change anything.
   - Uses `Resource: "*"`, which is acceptable only because every action is read-only.

Neither policy was deployed. The README states that a policy file in a repository is not proof it was deployed, and the example bucket name is a placeholder that I did not replace.

## Excessive permissions and remediation
| Finding | Risk | Remediation |
|---|---|---|
| Root is the only sign-in identity | Root cannot be limited by IAM policies | Create a protected IAM identity for daily work and stop routine root use |
| `s3:GetObject` on `reports/*` in the reader policy | Anything saved under that prefix becomes readable | Keep only report files under that prefix |
| `Resource: "*"` in the auditor policy | Read access to configuration across the account | Acceptable for read-only actions; assign only to dedicated audit identities |
| Policies attached to individual users | Harder to audit | Attach to groups or roles instead |

## Decisions
- **D-03 (task-specific IAM policy):** Reviewed `s3-report-reader.json` and explained its scoped read access instead of using `s3:*` or administrator access. Not deployed.

## Evidence
- `screenshots/after/E03-iam-least-privilege.png`: policy review
- Supporting screenshots: `E04a-iam-dashboard.png`, `E04b-iam-roles.png`, `E03b-role-trust-policy.png` (account IDs redacted)

## What I learned
- IAM answers two questions: who is making a request, and what are they allowed to do.
- A policy has an effect, actions, resources and optional conditions. Listing a bucket and reading an object are separate permissions that apply to different resources.
- A role has two halves: a trust policy (who may assume it) and a permissions policy (what it can do).
- Least privilege depends on what is left out of a policy as much as what is in it.
- Read-only access is still sensitive, because configuration details help an attacker plan.
- Reviewing a policy and deploying it are different things.
