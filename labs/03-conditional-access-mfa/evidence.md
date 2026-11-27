# Lab 03 Evidence

All times are UTC on 2026-11-27 (evening of 2026-11-26 local, EST). A `~` means the time is approximate, taken from the portal toast or the local clock rather than a log entry.

## Setup

|Time (UTC)|Action|Target|Result|Detail|
|-|-|-|-|-|
|01:55|Create user|Test User 1 (`testuser1@`)|Success|Regular member, no roles, auto-generated password|
|~01:56|Create user|Break Glass Admin (`breakglass@`)|Success|Global Administrator assigned on the Assignments tab|
|~01:56|Create group|GRP-BreakGlass|Success|Security, Assigned, 1 member (Break Glass Admin)|
|01:57:59|Create CA policy|CA01 Require MFA|Success|Report-only, All users excl. GRP-BreakGlass, All resources, Require MFA|
|~01:59|Create CA policy|CA02 Block legacy auth|Success|Report-only, same scope, Client apps = Exchange ActiveSync + Other clients, Block|

Both policy screens showed *"You must first disable security defaults before enabling a Conditional Access policy."* Report-only creation still succeeded.

## Report-only testing

### Sign-in logs (testuser1, My Apps in a private browser window)

|Time (UTC)|Policy|Result|Detail|
|-|-|-|-|
|02:02:38|CA01 Require MFA|Report-only: **User action required**|Would have prompted for MFA|
|02:02:38|CA02 Block legacy auth|Report-only: Not applied|Browser is a modern-auth client|
|02:04:44|CA01 Require MFA|Report-only: **Success**|Basic info: "MFA requirement satisfied by claim in the token"|
|02:04:44|CA02 Block legacy auth|Report-only: Not applied||

The audit log showed Authentication Methods / UserManagement events at 02:04:40, matching testuser1's password change and MFA registration under security defaults. The sign-in log didn't show these sign-ins until about 20 minutes later.

### What If (Office 365 Exchange Online, Windows)

|User|Client app|Will apply|Will not apply (reason)|
|-|-|-|-|
|Test User 1|Browser|CA01 (Require MFA)|CA02 (Client app)|
|Test User 1|Mobile apps and desktop clients – Other clients|CA01 (Require MFA), CA02 (**Block access**)|none|
|Break Glass Admin|Mobile apps and desktop clients – Other clients|none|CA01, CA02 (**Users and groups**)|

## Enforcement

|Time (UTC)|Action|Result|Detail|
|-|-|-|-|
|02:15:21|testuser1 sign-in|Success|Conditional access tab: **Security Defaults → Success** (last sign-in before the switch)|
|~02:15|Disable security defaults|Success|Reason: "My organization is planning to use Conditional Access"|
|~02:16|Set CA01 Require MFA to On|Success|"Policy will be enabled in a few minutes"|
|~02:16|Set CA02 Block legacy auth to On|Success||
|~02:20|testuser1 sign-in (new private window)|**MFA prompted**|"Approve sign in request", Microsoft Authenticator with number matching|

## Not tested live

- Break-glass sign-in after enforcement. The exclusion was checked with What If only.
- A real legacy-protocol client (IMAP/POP/SMTP AUTH). The block was checked with What If only.
