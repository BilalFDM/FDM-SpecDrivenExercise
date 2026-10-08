# Tasks: change-password

**Input**: Design documents from `/spec/001-change-password/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Organization**: Tasks are grouped by user story to enable independent implementation and testing with no new scope beyond the password-change feature.

## Phase 1: Setup

**Purpose**: Confirm the existing project baseline and the available security/session structure before implementation.

- [ ] T001 Review the existing Spring Boot baseline in `pom.xml` and `src/main/java/com/example/exercise/ExerciseApplication.java` to confirm the Java 21 / Spring Boot 4.1.1 benchmark and current app conventions.
- [ ] T002 [P] Review the existing security/session setup in the app configuration and current authentication flow to confirm where password-change logic and session invalidation should integrate.

---

## Phase 2: Foundational (blocking prerequisites)

**Purpose**: Create the minimum shared validation, audit, and error-handling contracts required before story work can proceed.

- [ ] T003 Define the password policy contract and validation rules in the implementation design: minimum 15 characters, spaces allowed, no required character classes, built-in blocklist check, and current-password inequality enforcement.
- [ ] T004 [P] Define the shared structured error contract for 400, 401, and 429 responses, including: HTTP status, error code, and user-safe message fields.
- [ ] T005 [P] Define the security-event contract for successful and failed password-change attempts, including the requirement to log repeated failures without recording passwords or tokens.

---

## Phase 3: User Story 1 - Change password from account security settings (Priority: P1) 🎯 MVP

**Goal**: Allow an authenticated user to change a password successfully from the account security flow.

**Independent Test**: A signed-in user can submit a valid current password, new password, and confirmation, and the system accepts the change and forces re-authentication.

### Implementation for User Story 1

- [ ] T006 [P] [US1] Define the request and response models for the password-change flow, including required fields and the success/error payload behavior.
- [ ] T007 [US1] Implement the password-change validation flow, including current-password verification, confirmation matching, password-length check, same-as-current rejection, and blocklist validation.
- [ ] T008 [US1] Implement the successful password update path, ensuring the password hash is updated atomically and the user is prompted to sign in again.
- [ ] T009 [US1] Wire the controller and exception mapping so the API returns the structured JSON error responses defined in the spec.
- [ ] T010 [US1] Record the successful security event without logging passwords, confirmation values, or tokens.

**Checkpoint**: User Story 1 is independently functional and ready for validation.

---

## Phase 4: User Story 2 - Handle failed or invalid password change attempts (Priority: P2)

**Goal**: Reject invalid attempts safely and protect the account from repeated brute-force attempts without changing the password.

**Independent Test**: A user can submit an incorrect current password, mismatched confirmation, same password, weak password, or repeated invalid attempts and the request is rejected with the correct response and no password change.

### Implementation for User Story 2

- [ ] T011 [P] [US2] Add validation tests for wrong current password, mismatched confirmation, missing required fields, weak password, same-as-current password, and blocklist rejection.
- [ ] T012 [US2] Implement the failure path so invalid requests return the correct error code and message without mutating the account.
- [ ] T013 [US2] Implement the rate-limit guard for repeated invalid password-change attempts: 5 failed password-change attempts per user in a rolling 10-minute window, blocking only further password-change attempts for that user until the oldest failure leaves the window; do not lock the account.
- [ ] T014 [US2] Record failed password-change attempts and repeated failures in the security event stream without recording credential data.

**Checkpoint**: User Story 2 is independently functional and the account remains protected during invalid attempts.

---

## Phase 5: User Story 3 - Session and token invalidation (Priority: P3)

**Goal**: Confirm that a successful password change invalidates only the affected user’s sessions and tokens.

**Independent Test**: After a successful password change, the affected user’s active session and all access/refresh tokens are invalidated, while other users remain unaffected.

### Implementation for User Story 3

- [ ] T015 [P] [US3] Add integration-style tests for user-scoped invalidation after successful password change.
- [ ] T016 [US3] Implement user-scoped session invalidation so the current session and all active sessions for the affected user are invalidated.
- [ ] T017 [US3] Implement token invalidation for the affected user only, including access and refresh tokens, while leaving other users’ tokens valid.
- [ ] T018 [US3] Confirm that the success flow requires fresh sign-in and that unrelated users remain unaffected.

**Checkpoint**: User Story 3 is independently functional and scoped to the affected user only.

---

## Phase 6: Final validation

**Purpose**: Confirm the feature is ready for review and aligns with the spec, plan, and security controls.

- [ ] T019 [P] Validate the final password-change behavior against the success, validation, rate-limit, and re-authentication scenarios in the feature quickstart.
- [ ] T020 [P] Validate the API contract against the structured JSON and standard HTTP behavior required by the specification.
- [ ] T022 Review the feature artifacts for scope matching: no unrelated account-management features, no password history requirement, no broad invalidation beyond the affected user, no password logging, and no account lockout.

---

## Dependencies & Execution Order

### Phase dependencies

- **Setup**: no dependencies.
- **Foundational**: must be complete before user stories begin.
- **User Story 1**: depends on foundational tasks.
- **User Story 2**: depends on foundational tasks and story 1 validation patterns.
- **User Story 3**: depends on foundational tasks and story 1 success behavior.
- **Final validation**: depends on all stories.

### Parallel opportunities

- T002 can run in parallel with T001.
- T004 and T005 can run in parallel after T003.
- T006 and T007 can run in parallel within User Story 1 after core validation definitions are complete.
- T011 and T013 can run in parallel within User Story 2.
- T015 and T017 can run in parallel within User Story 3.

### MVP first

1. Complete setup and foundational tasks.
2. Complete User Story 1 to establish the happy path.
3. Validate User Story 2 and User Story 3 as independent security hardening increments.

## Notes

- [P] indicates tasks that can run in parallel when the dependencies are satisfied.
- The tasks intentionally stay within the feature scope and do not introduce new product capabilities beyond the specified password-change flow.
- This task list is planning-only; it does not implement production code.
