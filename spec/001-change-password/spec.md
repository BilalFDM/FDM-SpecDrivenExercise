# Feature Specification: change-password

**Feature Branch**: `[001-change-password]`

**Created**: 2026-10-08

**Status**: Draft

**Input**: User description: "Create a feature that allows a user to change their password. Capture business rules, assumptions, user flows, inputs and outputs, constraints, and error conditions. Clearly mark unresolved decisions instead of treating assumptions as approved requirements. This exercise produces specification artifacts only."

## Clarifications

### Session 2026-10-08

- Q: What password policy should this feature require for new passwords? → A: Minimum 15 characters, spaces allowed, no required character combinations, and reject common or compromised passwords using a built-in blocklist shipped with the app. Coach review applies to the final PR, not to the decision itself.
- Q: After a successful password change, which session behavior should the system enforce? → A: Invalidate only the affected user’s active sessions, including the current session, and require sign-in with the new password before the user can continue. Other users’ sessions remain valid. Coach review applies to the final PR, not to the decision itself.
- Q: How many previous passwords should the system block from reuse after a password change? → A: The new password must differ from the current password; no history of previous passwords is maintained. Coach review applies to the final PR, not to the decision itself.
- Q: If the application uses JWT or refresh tokens, how should password changes affect them? → A: Invalidate all access and refresh tokens for the affected user only; tokens for other users remain valid. Coach review applies to the final PR, not to the decision itself.
- Q: What should be logged for password-change attempts? → A: Log successful and failed attempts, including repeated failures, and never log passwords, tokens, or token values. Coach review applies to the final PR, not to the decision itself.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Change password from account security settings (Priority: P1)

A signed-in user wants to update their password to a stronger or newer password from their account security settings. The user provides the current password, a new password, and a password confirmation before submitting the update.

**Why this priority**: This is the primary value of the feature and the most common task users will perform when managing account security.

**Independent Test**: A user can sign in, navigate to security settings, submit a valid password change, and continue using the account without disruption.

**Acceptance Scenarios**:

1. **Given** the user is authenticated and has access to account security settings, **When** they enter their current password, a valid new password, and matching confirmation, **Then** the system updates the password and shows a clear success confirmation.
2. **Given** the user enters a new password that does not satisfy the password policy or the confirmation does not match, **When** they submit the form, **Then** the system blocks the change and explains the specific issue to correct.

---

### User Story 2 - Handle failed or invalid password change attempts (Priority: P2)

A user attempts to change their password but enters the wrong current password, forgets a required field, or submits a password that is too weak or already used recently. The system should prevent the change and help the user retry safely.

**Why this priority**: Invalid attempts are common and the feature must protect account security without causing confusion or repeated frustration.

**Independent Test**: A user can intentionally submit incorrect values and receive clear, actionable feedback without being granted access or seeing sensitive account details.

**Acceptance Scenarios**:

1. **Given** the user enters an incorrect current password, **When** they submit the change request, **Then** the system rejects the request and tells them the current password is incorrect.
2. **Given** the user enters a new password that matches the current password or a recent password, **When** they submit the form, **Then** the system rejects the request and asks them to choose a different password.

---

### User Story 3 - Security event handling after a successful password change (Priority: P3)

After a password update, the system should record the security event and apply the organization’s expected session behavior so the user remains secure and the event can be audited.

**Why this priority**: Password changes are a security-sensitive action, and auditability and session controls are required even if they are not the first interaction users notice.

**Independent Test**: A password change can be completed and then verified that the event is logged and the account’s session behavior follows the defined security policy.

**Acceptance Scenarios**:

1. **Given** a password change is completed successfully, **When** the transaction is processed, **Then** the system records the event and creates an auditable security record.
2. **Given** the organization’s session policy requires a reset, **When** the password is changed, **Then** the system enforces the required session behavior as part of the same action.

---

### Edge Cases

- What happens when the user enters the same password they already use?
- How does the system handle a request submitted when the user session expires mid-flow?
- What happens when the user enters a password that matches a recently used password but not the current one?
- How does the system behave if the user performs repeated invalid attempts in a short time period?

## User Flow

1. The authenticated user opens the account security settings experience.
2. The user enters the current password, the new password, and confirmation.
3. The system validates the request and checks the current password before continuing.
4. If validation passes, the system validates the new password against the password policy and blocklist.
5. If the password is acceptable, the system updates the password hash, records the security event, and invalidates only the affected user’s sessions and tokens.
6. The system returns a success response or an actionable error message and requires the user to sign in again with the new password.

## Business Rules and Constraints

### BA Review

- Only an authenticated user may initiate a password change from the account management experience.
- The request must include the user’s current password, the replacement password, and confirmation of the new value.
- The system must not reveal whether a username or account exists when a password change fails beyond the standard account security messaging already approved for the product.
- A password change must be treated as a security-sensitive action and logged as such.
- The chosen replacement password must be different from the current password and must satisfy the approved password policy: minimum 15 characters, spaces allowed, no required character combinations, and common or compromised passwords rejected using a built-in blocklist shipped with the app.
- After a successful password change, the system must invalidate only the affected user’s active sessions, including the current session, and require sign-in with the new password before further access. Other users’ sessions remain active.
- The system must not maintain a history of previous passwords for reuse enforcement; the new password must differ from the current password only.
- Password changes must be processed only through the approved account security flow, not through ad hoc administrative edits or unsupported channels.

### Preconditions

- The user is already authenticated and has access to the account security settings experience.
- The account is active and not in a restricted state that prevents credential changes.
- The identity and session context for the authenticated user are available to the request processor.

### Postconditions

- On success, the user’s password is updated and a security event is recorded.
- The affected user’s active sessions are invalidated, including the current session.
- The user must authenticate again with the new password to resume access.
- On failure, the account remains unchanged and the user receives a clear validation or security message.

### Input Rules

- Current password, new password, and confirmation are required fields.
- New password must be at least 15 characters.
- Spaces are allowed in the new password.
- The new password must not match the current password.
- The new password must not be in the configured common or compromised blocklist.
- The new password and confirmation values must match exactly.

### Non-Functional Requirements

- The feature must be safe against enumeration and avoid exposing account status in validation messages.
- Validation and audit messaging must be clear and actionable to the end user.
- Password updates must be processed atomically so the account is not left in a partially updated state.
- Security events must be recorded consistently for both successful and failed attempts.
- The feature must operate on the Java 21 / Spring Boot 4.1.1 baseline defined for this project.

## Inputs and Outputs

### Inputs

- Current password entered by the authenticated user
- New password entered by the authenticated user
- Confirmation of the new password
- Current user identity and account session context
- Optional security or compliance metadata such as device, timestamp, and browser context

### Outputs

- Success confirmation message after a completed password change
- Validation errors for invalid current password, mismatched confirmation, weak password, or reused password
- Security audit event entry for successful or failed actions
- Required session or access state changes after the password update, based on the organization’s policy

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow an authenticated user to initiate a password change from the account security or profile settings experience.
- **FR-002**: The system MUST require the user to provide the current password before accepting a replacement password.
- **FR-003**: The system MUST require the user to enter a new password and confirm that new password before submission.
- **FR-004**: The system MUST reject the password change when the current password is incorrect, the confirmation does not match, or the submission is incomplete.
- **FR-005**: The system MUST validate the new password against the approved policy: minimum 15 characters, spaces allowed, no required character combinations, and common or compromised passwords rejected using a built-in blocklist shipped with the app.
- **FR-006**: The system MUST require the new password to differ from the current password and MUST NOT maintain a history of previous passwords for reuse enforcement.
- **FR-007**: The system MUST display a clear success message after a password change is completed.
- **FR-008**: The system MUST display clear, actionable error feedback when a password change attempt fails.
- **FR-009**: The system MUST record successful and failed password change attempts as security events for audit and support investigation.
- **FR-010**: The system MUST invalidate only the affected user’s active sessions and all access/refresh tokens after a successful password change and require the user to re-authenticate with the new password.
- **FR-011**: The system MUST ensure the password change process is available only to the authenticated user or an authorized account owner and must not allow unintended bypasses through alternative flows.

### QA Review

- Negative scenarios must include wrong current password, mismatched confirmation, missing required fields, weak passwords, same-as-current password, previously blocked compromised passwords, and repeated invalid attempts in a short period.
- Validation must fail closed without revealing account existence or whether a credential is valid beyond the standard user-facing message.
- Error codes should clearly distinguish between client input issues, authentication failures, policy violations, and security or session invalidation actions. Recommended specification-level codes include: `INVALID_CURRENT_PASSWORD`, `PASSWORD_CONFIRMATION_MISMATCH`, `PASSWORD_POLICY_VIOLATION`, `PASSWORD_SAME_AS_CURRENT`, `SESSION_EXPIRED`, `RATE_LIMIT_EXCEEDED`, and `AUTH_REQUIRED`.
- The system must use standard HTTP status codes with structured JSON error payloads containing a machine-readable code and a user-safe message.
- The system must reject attempts that are incomplete or maliciously duplicated, and it must not create a side effect when validation fails.
- Testable acceptance criteria: when a user submits an invalid current password, the system returns a validation or security error and does not update the password; when a password fails the policy, the system blocks the change and explains the policy issue; when the operation succeeds, the user is forced to authenticate again and only their sessions are invalidated.

### Dev Review

- Target technology baseline: Java 21 and Spring Boot 4.1.1 as defined in the project Maven configuration.
- Preferred implementation pattern is the framework’s standard web application stack without broadening the project scope beyond the password change feature.
- Allowed: Spring Boot web MVC, the built-in validation framework, secure session management already configured by the application, and standard logging and audit hooks.
- Forbidden: bypassing the account security flow, writing direct database updates outside the approved service path, storing plaintext passwords in logs, or introducing unapproved auth libraries or custom credential storage implementations.
- Security rules: passwords must never be logged or exposed in error messages, password hashing must use a strong one-way algorithm supported by the platform, and the feature must validate and enforce the policy before storing any new password.
- Rate limits and throttling: repeated failed attempts must be limited with rate limiting only to protect the account from brute-force behavior. The system blocks further password-change attempts for that user only after 5 failed attempts within a rolling 10-minute window, and the block remains until the oldest failure leaves the window. The account itself is not locked.
- Logging: successful and failed password change events must be logged in a security-safe way without recording the cleartext password, cleartext confirmation, tokens, or token values. Repeated failures must also be recorded.
- Session and token behavior: when a password succeeds, only the affected user’s active sessions are invalidated, including the current session, and all access and refresh tokens for that user must be invalidated or treated as expired. Other users’ sessions and tokens remain valid unless separately invalidated for a different reason.

### Error Handling Matrix

| Condition | HTTP status | Error code | User-safe message |
| --- | --- | --- | --- |
| Missing or malformed input | 400 | `VALIDATION_ERROR` | “Please complete all required fields.” |
| Wrong current password | 401 | `INVALID_CURRENT_PASSWORD` | “The current password is incorrect.” |
| Confirmation mismatch | 400 | `PASSWORD_CONFIRMATION_MISMATCH` | “The new password and confirmation do not match.” |
| Password violates policy | 400 | `PASSWORD_POLICY_VIOLATION` | “The new password does not meet the password requirements.” |
| Same as current password | 400 | `PASSWORD_SAME_AS_CURRENT` | “Choose a different password.” |
| Rate limit exceeded | 429 | `RATE_LIMIT_EXCEEDED` | “Too many password-change attempts. Please try again later.” |
| Session expired or invalid | 401 | `SESSION_EXPIRED` or `AUTH_REQUIRED` | “Your session expired. Please sign in again.” |

### Key Entities *(include if feature involves data)*

- **User Account**: A user’s identity record, including the credential state and account security settings.
- **Password Change Request**: The submitted request containing the current password, the new password, confirmation data, and the result of validation.
- **Security Event**: An auditable record created when a password change is attempted or completed.
- **Active Session**: A currently authenticated access session associated with the user account that may need to be revalidated or invalidated after a password change.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 95% of authenticated users can complete a password change in under 2 minutes without assistance.
- **SC-002**: 99% of password change attempts produce a clear success or actionable error message on the first submission.
- **SC-003**: Unauthorized password change attempts are prevented and logged for review, with no successful changes occurring outside the validated account flow.
- **SC-004**: Users understand the outcome of their update immediately and can continue common account activities without security confusion.

## Assumptions

- The user is already authenticated before starting the password change flow.
- The feature operates within an existing account-management system with established identity and session controls.
- The password policy described in this exercise is the accepted requirement unless a later final PR changes it.
- After a successful password change, only the affected user’s active sessions are invalidated, including the current session, and the user must sign in again with the new password.
- The exercise does not maintain a history of previous passwords for reuse enforcement; only a difference from the current password is required.
- Password recovery, account lockout, and multi-factor policy enforcement are separate concerns and are not assumed to be part of this feature unless specified elsewhere.
- Standard secure storage and secure transport practices already exist for credentials and user data in the broader product.

## Known Gaps


- The exact API route path for the endpoint remains dependent on the existing application’s controller conventions.
- Error payload details such as field-level validation metadata may need final product contract approval beyond this exercise.

These are product- or integration-level details and do not change the core password-change requirements.

