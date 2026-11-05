# Policy Review: `s3-report-reader.json` and `s3-security-auditor-readonly.json`

**Reviewed:** 2026-11-05
**Status:** Both policies reviewed and explained. **Neither is deployed** to any AWS account. The policy files in the repository are examples, and a policy file in a repository is not proof that it was deployed.

The first sections (down to "What I learned") review `s3-report-reader.json`, the main least-privilege policy. The section "Second policy: `s3-security-auditor-readonly.json`" follows, with a comparison.

## Purpose
The policy is meant for an identity (a user or role) that only needs to **read monthly reports** stored in one private S3 bucket. It allows that and nothing else.

## Key terms
- **Bucket:** a named container in Amazon S3 that holds files.
- **Object:** a file stored in a bucket.
- **Prefix:** the beginning of an object's name, such as `reports/`, which behaves like a folder.
- **Policy:** a JSON document that allows or denies actions on resources, optionally under conditions.
- **Least privilege:** granting only the actions, resources and conditions a task needs.

## Statement-by-statement review

### Statement 1: `ListOnlyReportsBucket`
| Element | Value | Plain meaning |
|---|---|---|
| Effect | `Allow` | This statement grants access. |
| Action | `s3:ListBucket` | The identity may see the list of object names in the bucket. It cannot open any object. |
| Resource | `arn:aws:s3:::REPLACE-WITH-YOUR-PRIVATE-BUCKET` | One bucket. The ARN has no `/*` because listing applies to the bucket itself. |
| Condition | `StringLike` on `s3:prefix`: `reports`, `reports/*` | A listing request is honored only if it asks for the `reports` prefix or something under it. The rest of the bucket cannot be listed. |

### Statement 2: `ReadOnlyReportObjects`
| Element | Value | Plain meaning |
|---|---|---|
| Effect | `Allow` | This statement grants access. |
| Action | `s3:GetObject` | The identity may open or download an object. |
| Resource | `arn:aws:s3:::REPLACE-WITH-YOUR-PRIVATE-BUCKET/reports/*` | Only objects under `reports/` in that one bucket. The `/*` refers to the objects inside the prefix. |

## What the policy does not allow
- **Uploading or changing objects** (`s3:PutObject` is not granted).
- **Deleting objects** (`s3:DeleteObject` is not granted).
- **Access to other buckets** (only one bucket is named).
- **Reading objects outside `reports/`** (the Resource is limited to that prefix).
- **Listing all buckets in the account** (`s3:ListAllMyBuckets` is not granted).
- **Changing permissions or bucket settings** (no IAM or bucket-configuration actions).
- **Wildcard actions or resources** (there is no `s3:*` and no `Resource: "*"`).

## Why this is least privilege
Each permission is tied to the task "read reports":
1. Listing is needed to find report names, and it is limited to the `reports` prefix.
2. Reading is needed to open a report, and it is limited to objects under `reports/`.
3. Nothing else is granted, so a mistake or a compromised credential could not change, delete or read data elsewhere.

## Excessive permissions and remediation
No excessive permissions were found in this policy. Points to watch if it were ever used:

| Observation | Risk | Remediation |
|---|---|---|
| `s3:GetObject` applies to everything under `reports/*` | Anything stored under that prefix becomes readable by this identity, including a sensitive file saved there by mistake | Keep only report files under `reports/`; keep sensitive data in another prefix or bucket |
| The bucket name is a placeholder | The policy would not work as written, and a careless value could point it at the wrong bucket | Replace it with the exact name of a private bucket you own, and confirm the bucket is private before attaching |
| This is an identity policy only | Other controls (bucket policy, permissions boundary, organization policies) can widen or narrow the real access | Review effective access, not just this file. Stage 6 adds a bucket policy and account-level Block Public Access |
| Not yet attached to a group or role | Attaching it directly to individual users makes access harder to audit | If deployed, attach it to a group or role instead of to users |

## Decision-log entry (D-03)
| ID | Decision | Rationale | Alternative considered | Evidence/status |
|---|---|---|---|---|
| D-03 | Use task-specific IAM policy | Limit permissions | `s3:*` or administrator access | Reviewed `s3-report-reader.json` on 2026-11-05; scoped read access explained in `03-iam-least-privilege/policy-review.md`. Policy not deployed. Evidence: `E03-iam-least-privilege.png`. |

## What I learned
- A policy answers four questions: is it allowed (Effect), what can be done (Action), to what (Resource), and under what extra rules (Condition).
- Listing a bucket and reading an object are separate permissions that apply to different resources, so the two statements have different ARNs.
- Least privilege is about what is *missing* from a policy as much as what it contains.
- Reviewing a policy and deploying it are different things; this stage required the review.

## Second policy: `s3-security-auditor-readonly.json`

### Purpose
Illustrative **read-only review permissions** for someone checking IAM and S3 configuration, such as a security auditor. The repository README states it is not a complete account-wide audit policy. It lets the identity *look at settings*; it does not let it change anything or open stored files.

### How to read the action names
Each action is written `service:Verb`. `iam` is the access-control service and `s3` is storage. Every action here starts with **List** (show names) or **Get** (show details of one thing). None creates, changes, attaches or deletes anything.

### Statement 1: `ReadSelectedIAMConfiguration`
| Action | Plain meaning |
|---|---|
| `iam:ListUsers`, `iam:ListGroups`, `iam:ListRoles`, `iam:ListPolicies` | List the users, groups, roles and policies in the account |
| `iam:ListAttachedUserPolicies`, `iam:ListAttachedGroupPolicies`, `iam:ListAttachedRolePolicies` | Show which managed policies are attached to a user, group or role |
| `iam:ListUserPolicies`, `iam:ListGroupPolicies`, `iam:ListRolePolicies` | Show inline policies written directly into a user, group or role |
| `iam:GetPolicy`, `iam:GetPolicyVersion` | Read the details and the JSON wording of a policy |
| `iam:GetRole` | Read the details of a role |

These match the checks done in the access review (Part A): who exists, and which permissions they have.

### Statement 2: `ReadSelectedS3Configuration`
| Action | Plain meaning |
|---|---|
| `s3:ListAllMyBuckets` | List all buckets in the account |
| `s3:GetAccountPublicAccessBlock` | Check the account-level Block Public Access settings |
| `s3:GetBucketPolicyStatus` | Check whether a bucket is considered public |
| `s3:GetBucketPublicAccessBlock` | Check one bucket's Block Public Access settings |
| `s3:GetBucketPolicy` | Read a bucket's policy |
| `s3:GetBucketAcl` | Read a bucket's access control list |
| `s3:GetBucketLocation` | See which Region a bucket is in |

These match the S3 protection checks in Stage 6.

### What the policy does not allow
- Creating, changing, attaching or deleting users, roles or policies
- Changing any bucket setting
- **Reading or listing the objects (files) inside a bucket.** There is no `s3:GetObject` and no `s3:ListBucket`.
- Access to services other than IAM and S3

### About `"Resource": "*"`
Both statements use a wildcard resource. That is normal here: account-wide List actions (for example listing all users or all buckets) cannot be pinned to one item, and every action is read-only. It is still noted because a wildcard resource should always be checked and justified.

### Risks and remediation
| Observation | Risk | Remediation |
|---|---|---|
| Wildcard `Resource: "*"` on all actions | Read access to configuration across the whole account | Acceptable because it is read-only; grant it only to identities that perform audits |
| Read-only is not the same as harmless | Knowing how permissions and bucket settings are configured helps an attacker plan | Attach to a dedicated audit group or role, not to individual users; review who has it periodically |
| The policy is incomplete as an audit policy | It does not cover CloudTrail, budgets or other services, so it could give a false sense of coverage | Treat it as an example; extend it deliberately if a fuller audit role is needed |

## Comparison
| | `s3-report-reader` | `s3-security-auditor-readonly` |
|---|---|---|
| Who it is for | Someone who needs to read reports | Someone who inspects configuration |
| Looks at | **Contents**: files under `reports/` | **Settings**: IAM setup and S3 configuration |
| Can open a file in a bucket | Yes, but only under `reports/` | No |
| Can list file names in a bucket | Only under the `reports` prefix | No |
| Can see how permissions or bucket settings are set up | No | Yes |
| Resource scope | One bucket and one prefix | `*` (whole account), read-only |
| Can change anything | No | No |
