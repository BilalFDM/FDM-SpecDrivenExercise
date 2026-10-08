# Research: password change feature

## Decision: Password policy

- Decision: Require a minimum of 15 characters, allow spaces, reject required character classes, and reject common or compromised passwords using a built-in blocklist shipped with the app.
- Rationale: This matches the clarified requirement and keeps the implementation self-contained within the existing Spring Boot application without introducing external dependencies.
- Alternatives considered: Third-party password service, regex-only validation, and requiring digit/uppercase/symbol classes. These were rejected because the clarified requirement explicitly allows spaces, does not require character-class combinations, and prefers a built-in blocklist.

## Decision: Session and token invalidation

- Decision: After a successful password change, invalidate all access and refresh tokens for the affected user only, including the current session. Other users' sessions remain valid.
- Rationale: This preserves the least-privilege security model and satisfies the clarified requirement without broad disruption to unrelated users.
- Alternatives considered: invalidating all user sessions globally and rotating tokens for everyone. Rejected because the clarified rule limits invalidation to the affected user.

## Decision: Password history

- Decision: Do not maintain a password history; the only reuse rule is that the new password must differ from the current password.
- Rationale: This matches the clarified design decision and keeps the feature simple and low-risk for this exercise.
- Alternatives considered: maintaining a password history store and blocking the last N passwords. Rejected because the requirement explicitly does not maintain a history.

## Decision: Repeated failures

- Decision: Use rate limiting only for repeated failed password-change attempts.
- Rationale: The clarified requirement prefers throttling over account lockout for this feature, while preserving protection against brute-force attempts.
- Alternatives considered: account lockout and combined lockout+rate limiting. Rejected because the chosen rule is rate limit only.

## Decision: Error contract

- Decision: Return standard HTTP status codes with structured JSON payloads containing a stable machine-readable code and a user-safe message.
- Rationale: This yields clear client handling and supports differentiated validation, authentication, throttling, and policy failures without leaking sensitive details.
- Alternatives considered: plain-text messages and ad hoc custom status codes. Rejected because the clarified requirement prefers structured JSON and standard HTTP semantics.

## Decision: Audit logging

- Decision: Log successful and failed password-change attempts, including repeated failures, while never logging passwords or tokens.
- Rationale: Security auditability is required, but credential data must remain protected in logs.
- Alternatives considered: logging only success, or logging raw credential values. Both are rejected because of compliance and confidentiality requirements.

## Dependency notes

- The repository is already a Spring Boot web MVC application running on Java 21 and Spring Boot 4.1.1.
- The feature does not require a new framework or a new persistence system; it should use the existing application layer for credential verification and session management.
- The implementation should integrate with the app’s authentication and session system, but the scope remains limited to the password-change flow.
