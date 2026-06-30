---
name: Audit Findings 2026-06-26
description: Complete bug and standards review from initial audit — all bugs, severity, affected files
type: project
---

## Session: 2026-06-26 Initial Audit

### Critical Bugs Found
1. getJobApplications() reads 'globalApplications' key that is never written — employer always sees 0 applicants via this path (JobContext.jsx line 315)
2. updateApplicationStatus() in JobContext reads/writes 'globalApplications' but jobApplicationService writes to 'jobApplications_{userId}' — these are completely different keys (JobContext.jsx lines 325-333)
3. JobApplicants.jsx expects app.userName, app.userEmail, app.userProfile from getApplicationsByJob() but service returns app.applicantName, app.applicantEmail and no userProfile — all applicant details render as undefined
4. Application status written as 'APPLIED' (uppercase) but JobApplicants.jsx filter and statusCounts compare against 'Applied' (title case) — filter tabs always show 0 for Applied, styling always defaults to gray
5. updateJob() in JobContext NEVER updates globalPostedJobs localStorage — only updates React state + postedJobs_{userId} via separate useEffect. After page refresh, globalPostedJobs has stale data.

### Standards Issues Found
6. postJob() in JobContext bypasses the service layer and writes directly to localStorage — violates CLAUDE.md "do not fetch data directly in page components; use services" pattern (same applies to context)
7. PostJob.jsx (page component) also writes directly to localStorage, duplicating job creation logic that should live in JobContext/service
8. JobContext.postJob() and PostJob.jsx both generate IDs with Date.now() — two independent job creation paths, neither calls the other
9. JobContext has no async/delay() in postJob/updateJob/deleteJob — synchronous mutations violate the async service pattern
10. savedJobService uses key 'savedJobIds_{userId}' but CLAUDE.md documents key as 'savedJobs_{userId}' — key name deviation

### Why:** These findings needed to be recorded so future sessions can reference them without re-reading all source files.
**How to apply:** When asked about bugs or fixes, start from this list rather than re-auditing from scratch. Verify the specific lines are still present before citing them.
