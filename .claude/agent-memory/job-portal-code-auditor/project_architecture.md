---
name: Architecture Overview
description: Provider nesting, two-context pattern, localStorage key registry, service layer wiring
type: project
---

## Provider Nesting (App.jsx)
AuthProvider → JobsDataProvider → JobProvider → CompaniesProvider → ThemeProvider

## Context Roles
- JobsDataContext (src/contexts/): Cached job list, 5-min TTL, `fetchAllJobs` from companyService. Owns `updateJobApplicationsCount`.
- JobContext (src/context/): Per-user state — appliedJobs, savedJobs, postedJobs, allApplications. Owns employer CRUD.
- AuthContext: Session, dummy users, registeredUsers in localStorage.

## localStorage Key Registry (as observed in code)
- `jobPortalUser` — current user session object
- `authToken` — mock JWT
- `registeredUsers` — array of registered users
- `globalPostedJobs` — all employer-posted jobs (used by companyService.fetchAllJobs)
- `postedJobs_{userId}` — employer's own job list (read by MyJobs.jsx directly)
- `jobApplications_{userId}` — per-user application records (written by jobApplicationService)
- `savedJobIds_{userId}` — saved job entries (savedJobService uses THIS key, NOT `savedJobs_{userId}`)
- `allApplications_{userId}` — employer's copy of applications (written by JobContext useEffect)
- `globalApplications` — ORPHANED key referenced by getJobApplications/updateApplicationStatus in JobContext but NEVER written by any service

## Service Layer
- jobApplicationService.js: reads/writes `jobApplications_{userId}`. getApplicationsByJob scans all `jobApplications_*` keys.
- savedJobService.js: reads/writes `savedJobIds_{userId}` (note: NOT `savedJobs_{userId}`)
- companyService.js: fetchAllJobs merges mockData jobs + globalPostedJobs

**Why:** Needed for cross-referencing bugs across the codebase.
**How to apply:** When tracing a data flow, use this registry to quickly identify which key is canonical for a given entity.
