---
name: Security Findings — Auth & Registration
description: All security vulnerabilities found in AuthContext.jsx and Register.jsx; credential storage, token, role escalation issues
type: project
---

Audit date: 2026-06-26. Files: src/context/AuthContext.jsx, src/pages/Register.jsx.

**Critical issues:**
1. Plaintext passwords stored in `registeredUsers` localStorage key — anyone with browser access can read all credentials.
2. Dummy user passwords hardcoded in DUMMY_USERS constant in AuthContext.jsx (lines 7–111) — visible in source bundle.
3. `authToken` is `'mock-jwt-' + Date.now()` — trivially guessable, no entropy, no server-side validation. Setting any string in `authToken` + valid `jobPortalUser` JSON bypasses auth entirely.
4. Role stored in `jobPortalUser` localStorage — manually settable to `ROLE_ADMIN` or `ROLE_EMPLOYER` to escalate privileges.

**High issues:**
5. No password hashing at any layer.
6. Register only allows `ROLE_JOB_SEEKER` self-registration, but the role field in the stored object is client-controlled — the `userType` form field (which accepts 'employer') is passed to `register()` but the function ignores it and always sets `ROLE_JOB_SEEKER`. The `company` field from the employer form path is also silently dropped.
7. Account enumeration: error message "Email already registered" confirms whether an email exists.
8. No session expiry — `authToken` never expires, no TTL on `jobPortalUser`.

**Medium issues:**
9. No input sanitization against XSS on name/company fields — values stored raw in localStorage and rendered via React (React escapes JSX renders, but raw innerHTML usage elsewhere would be at risk).
10. Password strength: only length (8–20 chars) enforced — no complexity requirements.
11. `Register.jsx` uses default export, not named export (minor violation of CLAUDE.md preference).

**How to apply:** Any future auth work must address plaintext storage and token entropy. Role must not be derived solely from client-side localStorage.
