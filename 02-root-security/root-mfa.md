# Stage 2: Secure the AWS Root User

## Objective
Protect the account's highest-privilege identity and minimize routine root use.

## What I did
1. Signed in as the root user of my learning account.
2. Opened the MFA registration screen and registered a **virtual MFA device** (authenticator app) named `root-authenticator-phone`.
3. Confirmed the account recovery email address and phone number (including the +234 country code) were correct.
4. **Tested the MFA before relying on it:** signed out, signed back in as root, entered the one-time code from the authenticator app, and reached the console successfully.
5. Opened **Security credentials** and reviewed the root user's access keys.
6. Kept all recovery information out of this repository: no MFA secrets, QR codes or recovery codes were captured or committed.

## Results
| Check | Result |
|---|---|
| Root MFA | Enabled, 1 virtual device assigned |
| MFA tested by signing out and back in | Passed |
| Root access keys | 0 (none exist; none created) |
| Recovery email and phone verified | Yes |

## Decisions
- **MFA type:** I chose an authenticator app over a passkey or hardware security key. Passkeys and security keys are stronger against phishing, but an authenticator app is simpler to set up and recover for a single-person learning account.
- **Access keys:** The target state is no root access keys. I confirmed none exist and did not create any. Long-lived credentials on the root user are high risk because root cannot be restricted by IAM policies.
- **Root usage going forward:** Root will not be used for routine work. Everyday administration moves to a separate protected IAM identity, created in Stage 3.

## Evidence
- `screenshots/after/E01-root-mfa.png`: Security credentials page showing the virtual MFA device assigned to root (account ID redacted)
- `screenshots/after/E02-root-access-keys.png`: Access keys section showing "No access keys" (0)
- Decision recorded in `docs/decision-log.md`

No "before" screenshot is included. This was a brand-new account with MFA not yet enabled, and I did not fabricate one.

## Residual risk
- Only one MFA device is registered. If I lose the phone, root recovery would be slow. Registering a second MFA device (for example a passkey) would reduce this risk.
- Root still exists and can sign in with its password and MFA. The mitigation is to stop using it for everyday tasks.

## What I learned
- Root has unrestricted power over the account, so a password alone is not enough protection.
- MFA should be tested before the session that set it up ends.
- The safest root access key is the one that does not exist.
- Evidence must be redacted: the account ID, QR codes and secret keys never belong in a public repository.
