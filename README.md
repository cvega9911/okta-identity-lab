# Okta Identity and Access Lab

An identity and access management lab built in a free Okta org (Integrator Free Plan, Okta Identity Engine). It covers the work a help desk or junior admin does every day: creating users and groups, setting password and MFA policy, controlling who can open which apps, resetting passwords, and offboarding, with every step checked in the audit log.

All users in this lab are test accounts. Email addresses, IP addresses, and the org address are redacted in the screenshots.

## What I built

| Area | Setup |
| --- | --- |
| Groups | `IT` and `Marketing`, two test users each |
| Password policy | 12+ characters, upper and lower case, number, symbol, no parts of the username, common-password check, breached-credential protection, last 4 passwords blocked, lockout after 5 failed attempts |
| MFA enrollment | Okta Verify required with no grace period; email and passkey optional |
| Sign-in policy | MFA required at every sign-in for IT and Marketing |
| App access | "IT Wiki" bookmark app assigned to the IT group only |
| Lifecycle | Onboarding, admin password reset, and offboarding, all traced in the System Log |

## 1. Users and groups

Created the `IT` and `Marketing` groups, then four test users with admin-set temporary passwords that must be changed at first sign-in.

![Groups with two members each](docs/images/01-groups.png)

![Adding a user with a temporary password](docs/images/03-add-person.png)

![People list](docs/images/02-people-list.png)

"Password expired" is expected here. It means each user still has to replace the temporary password the first time they sign in.

## 2. Password policy

A dedicated policy assigned to both groups, instead of editing the default.

![Password complexity rules](docs/images/04a-password-policy.png)

![Breached credentials, password history, and lockout](docs/images/04b-password-policy-lockout.png)

## 3. MFA

An authenticator enrollment policy makes Okta Verify required, and a global session policy rule requires MFA at every sign-in.

![MFA enrollment policy](docs/images/05-mfa-enrollment-policy.png)

![Session rule requiring MFA](docs/images/06a-session-rule-mfa-required.png)

![Global session policy active for both groups](docs/images/06b-global-session-policy-active.png)

## 4. Onboarding: first sign-in

A new user's first sign-in forces a password change that meets every policy rule, then requires Okta Verify enrollment before the dashboard opens.

![Forced password change showing the policy rules](docs/images/07a-first-login-password-change.png)

![Okta Verify enrollment (QR code redacted)](docs/images/07b-okta-verify-enrollment.png)

## 5. App access by group (least privilege)

The "IT Wiki" app is assigned to the IT group only. An IT user sees it on their dashboard. A Marketing user does not.

![IT user sees IT Wiki](docs/images/08a-it-user-sees-app.png)

![Marketing user sees no apps](docs/images/08b-marketing-user-no-app.png)

## 6. Password reset

Reset a user's password from the admin side with a temporary password. A reset email is the other option.

![Admin temporary password reset](docs/images/09-admin-temp-password-reset.png)

## 7. Offboarding

Deactivated a user. In the same moment Okta cleared their sessions and removed their app assignment. Their next sign-in attempts failed, and the System Log recorded each step.

![Deactivate confirmation](docs/images/10a-deactivate-confirm.png)

![User shows Deactivated](docs/images/10b-deactivated-user.png)

![Deactivated user cannot sign in](docs/images/10c-deactivated-login-fails.png)

![System Log: deactivation, session cleared, app removed, failed logins](docs/images/11-system-log-offboarding.png)

## Troubleshooting: "Unable to sign in"

The first test user could not sign in, and the error message was vague.

![Unable to sign in](docs/images/12a-troubleshoot-unable-to-sign-in.png)

What I checked, in order:

1. **System Log.** No sign-in attempts for the user at all, only my admin actions. A failed sign-in on the org would normally be logged, so this was the first real clue.
2. **Browser.** Same result in Chrome private mode and in Safari, so not a browser or extension issue.
3. **Policies.** The Access Testing Tool showed the user was allowed to access the Okta Dashboard, with only Okta Verify left to enroll. The policies were fine.
4. **MFA.** Turned the MFA requirement off temporarily. Still failed, so MFA was not the cause.
5. **Network response.** Browser DevTools showed the sign-in request returning HTTP 401 with Okta error `E0000004` ("Authentication failed").

![E0000004 in the network response](docs/images/12b-troubleshoot-E0000004-response.png)

**Root cause.** A password reset email finally worked, and its link pointed to a different hostname than the one I had been using to sign in. I had built the sign-in address from the org's display name, but the org's real address is different. The wrong address loaded a generic Okta sign-in page for a tenant where the user did not exist. That explains the authentication failure, the missing log entries, and the Access Testing Tool showing the user as allowed.

**Fix.** Used the org's actual sign-in address. The user signed in, changed their password, and enrolled Okta Verify.

**Lesson.** Confirm the exact tenant URL first. "Can't sign in" tickets often come down to the user being on the wrong page or the wrong tenant, and the audit log having no record of the attempt is the quickest way to spot it.

## Skills shown

User and group administration, password and MFA policy, Okta Verify enrollment, group-based app assignment, password resets, offboarding, System Log auditing, Access Testing Tool, and troubleshooting with browser DevTools.
