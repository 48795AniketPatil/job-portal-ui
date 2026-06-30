---
name: Bug Findings — JobContext & Application Service
description: Logic bugs, data inconsistencies, and missing guards in JobContext.jsx and jobApplicationService.js
type: project
---

Audit date: 2026-06-26. Files: src/context/JobContext.jsx, src/services/jobApplicationService.js.

**Critical bugs:**
1. `getJobApplications()` in JobContext reads from `globalApplications` key — but jobApplicationService.js NEVER writes to `globalApplications`. It writes to `jobApplications_{userId}`. This means getJobApplications() always returns []. JobApplicants.jsx correctly calls `jobApplicationService.getApplicationsByJob()` directly, bypassing this broken function — but the broken function is still exposed via context value.

2. `updateApplicationStatus()` in JobContext also reads/writes `globalApplications` — same broken key. The working implementation is in jobApplicationService.updateApplicationStatus() which iterates all `jobApplications_*` keys.

**High bugs:**
3. `updateJob()` in JobContext updates React state and `postedJobs_{userId}` via the save effect, but does NOT update `globalPostedJobs`. Jobs edited by employers will show stale data on the public job listing and JobDetail pages. PostJob.jsx and MyJobs.jsx both correctly update both keys when creating/status-changing jobs, but the JobContext.updateJob() function is the canonical mutation path and it is broken.

4. Duplicate application guard in `applyForJob()` (JobContext line 141) checks React in-memory `appliedJobs` state only — does not re-check localStorage. If state is cleared (e.g., page refresh between applying and checking), the guard is ineffective. The service layer has no duplicate guard at all in `applyForJob()`.

5. Employer data init in JobContext uses `user.userId || user.id` (line 87) but the save effect on line 120 also uses `user.userId || user.id`. If a session has only `userId`, the key is consistent; however dummy users use `id` (not `userId`), so employer dummy users (id: 1, 2) get keyed as `postedJobs_1` on load but may get keyed differently depending on which property is set at login time — potential key mismatch.

**Medium bugs:**
6. `getApplicationsByJob()` in jobApplicationService iterates `localStorage.length` and calls `localStorage.key(i)` — safe but O(n) over all localStorage keys, including unrelated keys. Benign for a mock app but architecturally fragile.

7. `savedJobService.js` uses key `savedJobIds_{userId}` but CLAUDE.md and JobContext fallback code references `savedJobs_{userId}`. The JobContext fallback on localStorage failure (lines 69-76) reads `savedJobs_{userId}` which will always be empty — the fallback is non-functional.

8. `getCompanyApplications()` in jobApplicationService is a stub returning `[]` — called nowhere currently but exported and misleading.

9. `allApplications_{userId}` is written by JobContext's save effect (line 127) from `allApplications` state, but `allApplications` state is never populated anywhere — it initializes to [] and has no setters called. This localStorage key is always written as `[]`.

**How to apply:** When fixing employer-side application visibility, the root fix is ensuring jobApplicationService.getApplicationsByJob() is the single source of truth. The globalApplications key and the allApplications context state should be removed or replaced.
