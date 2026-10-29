# Lab 01 Evidence

## Audit log evidence

Exported from Entra ID audit logs (times in UTC, local time is UTC-4). Emails, IP addresses, tenant IDs, and object IDs removed.

### Setup
| Time (UTC) | Activity | Target | Result | Change |
|---|---|---|---|---|
| 09:58:43 | Add group | GRP-Finance | Success | |
| 09:59:01 | Add group | GRP-IT | Success | |
| 09:59:28 | Add group | GRP-Sales | Success | |

### Joiner
| Time (UTC) | Activity | Target | Result | Change |
|---|---|---|---|---|
| 10:12:18 | Add user | Alex | Success | Department Finance, Job title Accountant |
| 10:13:16 | Add user | ulinea | Success | Department IT, Job title Systems Administrator |
| 10:13:57 | Add user | shannan | Success | Department Sales, Job title Sales Representative |
| 10:14:24 | Add user | mike | Success | Department Sales, Job title Sales Representative |
| 10:14:51 | Add user | Ava | Success | Department Finance, Job title Financial Analyst |
| 10:17:17 | Add member to group | Alex | Success | GRP-Finance |
| 10:17:17 | Add member to group | Ava | Success | GRP-Finance |
| 10:19:01 | Add member to group | ulinea | Success | GRP-IT |
| 10:19:29 | Add member to group | shannan | Success | GRP-Sales |
| 10:19:29 | Add member to group | mike | Success | GRP-Sales |

### First sign-in test (Ava)
| Time (UTC) | Activity | Target | Result | Change |
|---|---|---|---|---|
| 10:42:13 | Reset password (by admin) | Ava | Success | |
| 10:42:54 | Change password (self-service) | Ava | **Failure** | |
| 10:43:49 | Change password (self-service) | Ava | Success | |
| 10:44:45 | User registered all required security info | Ava | Success | MFA registration |

### Mover (mike, Sales to Finance)
| Time (UTC) | Activity | Target | Result | Change |
|---|---|---|---|---|
| 10:54:07 | Remove member from group | mike | Success | GRP-Sales removed |
| 10:56:07 | Add member to group | mike | Success | GRP-Finance added |
| 10:57:45 | Update user | mike | Success | Department Sales to Finance, Job title Sales Representative to Financial Analyst |

Old access was removed (10:54) before new access was granted (10:56).

### Leaver (Shannan)
| Time (UTC) | Activity | Target | Result | Change |
|---|---|---|---|---|
| 11:07:08 | Update StsRefreshTokenValidFrom Timestamp | shannan | Success | Sessions revoked |
| 11:07:37 | Disable account | shannan | Success | AccountEnabled true to false |
| 11:09:47 | Remove member from group | shannan | Success | GRP-Sales removed |
| 11:14:22 | Enable account | shannan | Success | Temporary re-enable for a password reset attempt |
| 11:21:19 | Disable account | shannan | Success | AccountEnabled true to false (final state) |

## Sign-in log evidence

Exported from Entra ID interactive sign-in logs (times in UTC). IP addresses, user IDs, and tenant IDs removed.

### First sign-in test (Ava)
| Time (UTC) | User | Application | Result | Error code | Meaning |
|---|---|---|---|---|---|
| 10:42:21 | Ava | My Profile | Interrupted | 50055 | Temporary password expired, so Entra forced a password change |
| 10:44:45 | Ava | My Profile | Interrupted | 50140 | "Keep me signed in" prompt, an expected step |
| 10:44:49 | Ava | My Profile | Success | none | Signed in with the new password |

The forced password change at first sign-in confirms the joiner account was created with a temporary password that must be changed.

### Leaver (Shannan)
No sign-in attempts for Shannan are recorded, because the blocked sign-in test was not completed. The disable event in the audit log is the evidence that sign-in was blocked.
