# Quickstart validation guide

## Purpose

This document describes how to validate the planned password-change feature after implementation. It is intentionally a run guide, not an implementation guide.

## Prerequisites

- Java 21 installed
- Maven available
- Spring Boot application started in a local environment
- User account available for authenticated testing
- Access to a browser or API client that can submit authenticated requests

## Recommended validation scenarios

### 1. Happy path

1. Sign in as an authenticated user.
2. Send a password-change request with:
   - valid current password
   - new password meeting the 15-character policy
   - confirmation matching the new password
3. Submit the request.
4. Confirm the response is a success result and that the user is required to sign in again.
5. Verify all active sessions and access/refresh tokens for that user are invalidated, while other users remain unaffected.

Expected outcome:
- Password is updated.
- Successful security event is recorded.
- User must re-authenticate before continued use.

### 2. Wrong current password

1. Sign in as a valid user.
2. Submit a request with the wrong current password.
3. Confirm the request is rejected.

Expected outcome:
- No password change occurs.
- Error response includes a structured code and message.
- Failed security event is recorded.

### 3. Password policy failure

1. Use a password shorter than 15 characters, or a compromised/common password from the built-in blocklist.
2. Submit the request.

Expected outcome:
- Request is rejected with a policy validation error.
- No password mutation occurs.
- Failure is logged without exposing sensitive password values.

### 4. Repeated failure rate limiting

1. Submit a series of invalid password-change attempts in rapid succession.
2. Observe the response behavior after the threshold is reached.

Expected outcome:
- The system slows or rejects repeated attempts.
- No password is changed.
- Security events record the repeated failure pattern.

### 5. Session invalidation

1. Log in to the system in multiple sessions for the same user.
2. Change the password successfully in one session.
3. Attempt to continue using the other sessions.

Expected outcome:
- The affected user’s sessions are invalidated.
- Only the affected user’s tokens are revoked.
- Other users’ sessions continue to function normally.

## Validation commands

Use the project’s Maven test path after implementation:

```bash
./mvnw test
```

If needed, run the application locally:

```bash
./mvnw spring-boot:run
```

## Exit criteria

The feature is ready when all of the above scenarios pass and the security-sensitive behavior remains consistent with the clarified requirement set.
