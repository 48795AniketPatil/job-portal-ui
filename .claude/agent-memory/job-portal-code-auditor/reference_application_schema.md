---
name: Application Object Schema
description: Application object shape in localStorage vs service/context layers — shape mismatches documented
type: reference
---

## jobApplicationService.applyForJob() — what is WRITTEN to localStorage
Key: `jobApplications_{userId}`
```js
{
  id: Date.now(),
  jobId: Number,         // the job's numeric id
  coverLetter: String,
  status: 'APPLIED',
  appliedAt: ISO String,
  // NOTE: no job details embedded — jobId is a foreign key
}
```

## jobApplicationService.getApplicationsByJob() — what employer READS
Scans all `jobApplications_*` keys. For each matching app (a.jobId === parseInt(jobId)), returns:
```js
{
  ...app,                // id, jobId, coverLetter, status, appliedAt
  userId: String,        // extracted from key suffix
  applicantName: `Applicant ${userId}`,   // FABRICATED — not real user data
  applicantEmail: `user${userId}@example.com`,  // FABRICATED
  // NOTE: no userProfile field — JobApplicants.jsx expects app.userProfile but it is always undefined
  // NOTE: no userName, userEmail, userMobileNumber fields — JobApplicants.jsx expects these
}
```

## JobApplicants.jsx — what it EXPECTS from getApplicationsByJob()
```js
{
  id: applicationId,
  userName: String,      // MISSING from service response
  userEmail: String,     // MISSING from service response  
  userMobileNumber: String,  // MISSING from service response
  appliedAt: ISO String,
  status: String,
  userProfile: {         // ALWAYS undefined from mock service
    jobTitle, location, experienceLevel, professionalBio,
    portfolioWebsite, resume, resumeType, profilePicture, profilePictureType
  }
}
```

## JobContext.applyForJob() — what is written to React state (appliedJobs)
```js
{
  ...job,                // full job object spread
  appliedAt: application.appliedAt,
  status: application.status,  // 'APPLIED'
  applicationId: application.id,
  coverLetter: application.coverLetter
}
```

## Status value mismatch
- jobApplicationService writes status: 'APPLIED' (uppercase)
- JobApplicants.jsx getStatusColor() checks for 'Applied' (title case) — will always use default gray style
- statusCounts checks app.applicant.status === 'Applied' — applied count always 0
- Filter uses app.applicant.status.toLowerCase().replace(' ', '') === filter — 'applied' filter tab will NOT match 'APPLIED'
