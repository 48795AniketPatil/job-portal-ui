---
name: Coding Standards Findings — JobContext mutations + general
description: CLAUDE.md violations found in JobContext.jsx (postJob/updateJob/deleteJob) and across employer pages
type: project
---

Audit date: 2026-06-26. Primary file: src/context/JobContext.jsx.

**Breaking violations:**
1. `postJob`, `updateJob`, `deleteJob` are all synchronous functions — they do NOT use `delay()`. CLAUDE.md mandates all async service calls use delay(). These are the only mutation functions in the context layer that skip it. All service-layer functions (jobApplicationService, savedJobService, companyService) correctly use delay().

2. `updateJob()` does not update `globalPostedJobs` in localStorage — only updates React state (which syncs to `postedJobs_{userId}`). State and the global listing are out of sync after every edit.

**Major violations:**
3. `postJob` duplicates job-creation logic that also exists inline in `PostJob.jsx` (handleSubmit). The page writes directly to localStorage itself rather than calling JobContext.postJob(). Both paths build a newJob object with `id: Date.now()` — two separate code paths for the same operation.

4. `Register.jsx` and `PostJob.jsx`, `MyJobs.jsx`, `JobApplicants.jsx` all use `export default` — CLAUDE.md prefers named exports for components.

5. `ProtectedRoute.jsx` also uses `export default`.

**Minor violations:**
6. Inline styles present in `JobApplicants.jsx` (lines 482, 545: `style={{ zIndex: 10000 }}`). Also in `JobsSection.jsx`, `CompaniesSection.jsx`, `Hero.jsx`, `Jobs.jsx`, `Companies.jsx`, `Profile.jsx` — all use `style={{}}` for animation delays or dynamic width values not expressible with static Tailwind classes.

7. `postJob` and `deleteJob` have no guard for null/undefined `userId` — if `user.userId` and `user.id` are both undefined, the localStorage key becomes `postedJobs_undefined`, silently corrupting storage.

8. Error handling is inconsistent: `postJob` returns a result object synchronously, `updateJob` returns a result object synchronously, `deleteJob` returns a result object synchronously — but none use try/catch around the localStorage operations, which can throw on storage quota exceeded or private browsing restrictions.

9. `postJob` hardcodes `companyLogo: '🏢'` — an emoji literal in business logic. Should be a named constant (UPPER_SNAKE_CASE per CLAUDE.md) or configurable.

**How to apply:** When refactoring mutations, consolidate PostJob.jsx direct localStorage writes into JobContext.postJob(), add delay() to all three mutation functions, and add null checks on userId.
