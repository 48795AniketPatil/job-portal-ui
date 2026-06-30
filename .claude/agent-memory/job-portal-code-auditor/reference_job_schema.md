---
name: Job Object Schema
description: Complete job object shapes — mockData jobs vs employer-posted jobs, salary field inconsistency
type: reference
---

## mockData.js generated jobs (src/data/mockData.js)
```js
{
  id: Number,            // sequential integer 1..1000
  title: String,
  company: String,
  companyLogo: String,   // path like "/logos/techcorp.png"
  location: String,
  workType: String,      // "On-site" | "Remote" | "Hybrid"
  jobType: String,       // "Full-time" | "Part-time" | "Contract" | "Freelance"
  category: String,
  experienceLevel: String,
  salary: {              // NESTED object
    min: Number,
    max: Number,
    currency: "USD",
    period: "year"
  },
  description: String,
  requirements: String[],
  benefits: String[],
  postedDate: ISO String,
  applicationDeadline: ISO String,
  applicationsCount: Number,
  featured: Boolean,
  urgent: Boolean,
  remote: Boolean,
  companyId: Number,
  // NOTE: no salaryMin/salaryMax/salaryCurrency/salaryPeriod flat fields
}
```

## PostJob.jsx posted jobs (src/pages/PostJob.jsx)
```js
{
  id: Date.now(),        // timestamp integer — collision risk if two employers post within same millisecond
  title, description, company, companyLogo: '🏢', location,
  jobType, workType, category, experienceLevel,
  salary: { min, max, currency, period },   // nested (for JobDetail compatibility)
  salaryMin, salaryMax, salaryCurrency, salaryPeriod,  // ALSO flat (for MyJobs.jsx compatibility)
  requirements, benefits, applicationDeadline,
  remote, featured, urgent,
  status: 'ACTIVE',      // PostJob sets status; JobContext.postJob does NOT set status
  postedBy: userId,
  postedDate: ISO String,
  applicationsCount: 0
}
```

## JobContext.postJob posted jobs (src/context/JobContext.jsx line 263)
```js
{
  id: Date.now(),
  ...jobData,            // spreads caller-supplied fields
  company: user.company,
  companyLogo: '🏢',
  postedBy: user.userId || user.id,
  postedDate: ISO String,
  applicationsCount: 0,
  featured: false,
  urgent: false
  // NOTE: no status field set here — relies on caller to include it in jobData
  // NOTE: no salaryMin/salaryMax flat fields — relies on caller including them in jobData
}
```

## CRITICAL SCHEMA INCONSISTENCY
- mockData jobs have salary as NESTED object only
- PostJob.jsx jobs have BOTH nested and flat fields
- JobContext.postJob jobs have neither flat fields unless caller includes them
- MyJobs.jsx formatSalary() reads `job.salaryMin` / `job.salaryMax` (flat) — will show $NaN for mockData jobs
- JobsDataContext.getJobById looks up by parseInt(id) — safe for both numeric IDs
