# Data model: password change feature

## Overview

This feature introduces a small, explicit set of security-oriented data objects around the existing user account and session infrastructure. The goal is to support validation, auditing, and session invalidation without broadening the application model beyond the password change workflow.

## Entities

### UserAccount

Represents the authenticated user whose password is being changed.

| Field | Type | Notes |
| --- | --- | --- |
| id | String/UUID | Stable application user identifier |
| username | String | Existing account identifier |
| passwordHash | String | Strong one-way hash, not plaintext |
| isActive | boolean | Account must be active to allow password changes |
| lastPasswordChangedAt | Instant | Optional audit metadata |

Validation rules:
- The account must be active and authenticated before the request proceeds.
- The stored password hash must be verified against the current password before modifying the account.
- The password update must be atomic and must not leave the account in a partial state.

### PasswordChangeRequest

Represents the submitted request data for the password change action.

| Field | Type | Notes |
| --- | --- | --- |
| currentPassword | String | Must be validated against the user’s stored hash |
| newPassword | String | Must satisfy the password policy |
| confirmPassword | String | Must match newPassword exactly |
| userId | String | Identity of the authenticated user |
| requestTimestamp | Instant | Security audit context |
| clientContext | String | Optional metadata such as device or browser, if available |

Validation rules:
- Current password is required.
- New password is required.
- Confirmation is required and must match exactly.
- New password length must be at least 15 characters.
- New password must not match the current password.
- New password must not appear in the built-in compromised-password blocklist.
- Input must be rejected without side effects when validation fails.

### ActiveSession

Represents a currently active authenticated session belonging to a user.

| Field | Type | Notes |
| --- | --- | --- |
| sessionId | String | Unique session identifier |
| userId | String | Owning user |
| issuedAt | Instant | Session creation time |
| expiresAt | Instant | Expiration / inactivity boundary |
| revoked | boolean | Marked when password change invalidates the session |

Validation rules:
- Only active sessions for the affected user are invalidated after a successful password change.
- Other users’ sessions remain valid.
- The current session must be invalidated as part of the same successful password-change action.

### AuthToken

Represents token-based authentication material that may be used for access or refresh flows.

| Field | Type | Notes |
| --- | --- | --- |
| tokenId | String | Unique token identifier |
| userId | String | Owning user |
| tokenType | enum | access or refresh |
| expiresAt | Instant | Expiration time |
| revoked | boolean | Revoked on successful password change |

Validation rules:
- Access and refresh tokens for the affected user only are invalidated after success.
- Tokens for other users remain valid.
- No secret token values are logged under any circumstance.

### SecurityEvent

Represents a security audit record for a password change attempt.

| Field | Type | Notes |
| --- | --- | --- |
| eventId | String | Unique event identifier |
| userId | String | Owning user |
| eventType | enum | PASSWORD_CHANGE_SUCCESS or PASSWORD_CHANGE_FAILED |
| timestamp | Instant | Signifies when the attempt occurred |
| reasonCode | String | Validation or security code like INVALID_CURRENT_PASSWORD |
| sourceIp | String | Optional operational metadata |
| sessionId | String | Optional session context |

Validation rules:
- Record both successful and failed attempts.
- Repeat failures should be logged with the same structured metadata, including rate-limit triggers.
- Never store plaintext passwords or token values in the event record.

## State transitions

### Password change flow

1. Authenticated user submits password-change request.
2. Server validates required fields and current password.
3. Server evaluates policy constraints and blocklist.
4. If invalid, reject request and persist a failed security event.
5. If valid, update password hash and invalidate affected-user sessions/tokens.
6. Persist success security event and require re-authentication.

### Session/token effect

- Success state: affected user sessions and tokens become invalid/expired.
- Failure state: no password mutation and no token invalidation.
- Rate limited state: repeated failures may be throttled without changing the account credentials.
