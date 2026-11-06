# Security Decision Log

| ID | Decision | Rationale | Alternative considered | Evidence/status |
|---|---|---|---|---|
| D-01 | Enable root MFA | Reduce password-only compromise risk | Password only | Verified 2026-11-04: root MFA active (virtual authenticator app); tested by signing out and back in. Evidence: `E01-root-mfa.png`. |
| D-02 | Avoid root access keys | Minimize exposure of privileged credentials | Root key for CLI | Verified 2026-11-04: root has 0 access keys; none created. Evidence: `E02-root-access-keys.png`. |
| D-03 | Use task-specific IAM policy | Limit permissions | `s3:*` or administrator access | Reviewed `s3-report-reader.json` on 2026-11-05; scoped read access explained in `03-iam-least-privilege/policy-review.md`. Policy not deployed. Evidence: `E03-iam-least-privilege.png`. |
| D-04 | Enable multi-Region CloudTrail management events | Improve account activity visibility | No trail / one Region | Pending |
| D-05 | Configure budget notification | Improve cost visibility | Manual bill checks | Pending |
| D-06 | Enable all four account-level S3 Block Public Access settings | Reduce accidental public exposure | Bucket-level only | Pending |

## For each change, record

### D-01: Root MFA
- **Date/time:** 2026-11-04
- **Change:** Registered a virtual MFA device (authenticator app) named `root-authenticator-phone` on the root user.
- **Risk addressed:** Root sign-in with a stolen or guessed password alone.
- **Expected result:** Root sign-in requires a one-time code from the authenticator app.
- **Validation and result:** Signed out and back in as root; AWS prompted for a code and sign-in succeeded. The Security credentials page shows 1 virtual MFA device assigned.
- **Rollback/recovery:** If the phone is lost, use AWS account recovery with the registered email and phone, then register a new MFA device. A second MFA device would make recovery easier.

### D-02: Avoid root access keys
- **Date/time:** 2026-11-04
- **Change:** Reviewed the root user's access keys and created none.
- **Risk addressed:** Long-lived root credentials being leaked or misused. Root cannot be limited by IAM policies.
- **Expected result:** The root user has no access keys.
- **Validation and result:** The Security credentials page shows 0 access keys.
- **Rollback/recovery:** Nothing to roll back. If a root key were ever created and exposed, deactivate and delete it immediately.
