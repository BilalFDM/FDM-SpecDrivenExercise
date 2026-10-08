# Password change API contract

## Endpoint

`POST /api/users/me/password`

## Purpose

Allow an authenticated user to change their password after verifying the current password and validating the new password policy.

## Request body

```json
{
  "currentPassword": "OldPassword123!",
  "newPassword": "NewPasswordWith15Chars!",
  "confirmPassword": "NewPasswordWith15Chars!"
}
```

## Success response

**HTTP status:** 200 OK

```json
{
  "success": true,
  "message": "Password updated successfully. Please sign in again.",
  "errorCode": null
}
```

## Validation error response

**HTTP status:** 400 Bad Request

```json
{
  "success": false,
  "message": "The new password does not meet the policy requirements.",
  "errorCode": "PASSWORD_POLICY_VIOLATION"
}
```

## Authentication failure response

**HTTP status:** 401 Unauthorized

```json
{
  "success": false,
  "message": "The current password is incorrect.",
  "errorCode": "INVALID_CURRENT_PASSWORD"
}
```

## Rate-limit response

**HTTP status:** 429 Too Many Requests

```json
{
  "success": false,
  "message": "Too many password change attempts. Please try again later.",
  "errorCode": "RATE_LIMIT_EXCEEDED"
}
```

## Notes

- The response body should always be structured JSON.
- Error messages must be user-safe and must not include password or token values.
- The API must reject invalid attempts without changing the password or invalidating unrelated sessions.
