---
name: Job Portal Architecture Overview
description: Core architecture facts — context layers, localStorage keys, provider order, service naming conventions
type: project
---

React 19 SPA, Vite 7, Tailwind CSS 4, React Router 7, plain JSX (no TypeScript).

**Why:** Pure mock/demo app — no real backend. All persistence is localStorage.

**Provider order:** AuthProvider → JobsDataProvider → JobProvider → CompaniesProvider → ThemeProvider (in App.jsx).

**Two context layers:**
- `src/context/` — AuthContext, JobContext, ThemeContext (core runtime state)
- `src/contexts/` — JobsDataContext (5-min TTL cache), CompaniesContext (data-fetching)

**localStorage key inventory:**
- `jobPortalUser` — full user object (includes role, userId)
- `authToken` — mock token string (`mock-jwt-{Date.now()}`)
- `registeredUsers` — array of all self-registered users with plaintext passwords
- `globalPostedJobs` — array of all employer-posted jobs (global)
- `postedJobs_{userId}` — per-employer job list
- `jobApplications_{userId}` — per-seeker application array (written by jobApplicationService.js)
- `savedJobIds_{userId}` — per-seeker saved job IDs (written by savedJobService.js; NOTE: key name differs from CLAUDE.md doc which says `savedJobs_{userId}`)
- `allApplications_{userId}` — per-employer applications cache (written by JobContext.jsx, never read by service layer)
- `globalApplications` — used by JobContext.getJobApplications() and updateApplicationStatus(), but NOT written by jobApplicationService.js — always reads as empty

**Service files:** companyService.js, contactService.js, jobApplicationService.js, profileService.js, savedJobService.js

**How to apply:** When suggesting localStorage reads/writes, verify key names against this list. The documented key `savedJobs_{userId}` is actually `savedJobIds_{userId}` in the service layer.
