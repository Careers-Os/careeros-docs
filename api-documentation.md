# CareerOS API Documentation

This document describes the REST API contract for the CareerOS backend (`careeros-api`), exposed through the Spring Cloud Gateway.

**Base URL:** `https://api.careeros.dev` (production) · `http://localhost:8080` (local dev)

All request/response bodies are JSON. All timestamps are ISO 8601 UTC (`2026-07-04T09:30:00Z`).

## 1. Authentication Headers

Except where noted "Auth Required: No", every endpoint requires a JWT access token obtained from login/register.

```
Authorization: Bearer <access_token>
```

- **Access token** — short-lived (15 min), used on every authenticated request.
- **Refresh token** — long-lived, sent only to `/api/auth/refresh` to mint a new access token. Stored as an HTTP-only cookie or passed in the request body, never in the `Authorization` header.
- Requests missing or with an invalid/expired `Authorization` header receive `401 Unauthorized`.
- Requests with a valid token but insufficient permissions receive `403 Forbidden`.

## 2. Error Response Format

All errors follow a consistent shape:

```json
{
  "timestamp": "2026-07-04T09:30:00Z",
  "status": 400,
  "error": "BAD_REQUEST",
  "message": "email must be a valid email address",
  "path": "/api/auth/register",
  "details": [
    { "field": "email", "issue": "must be a valid email address" }
  ]
}
```

| Status | Error code | Meaning |
|---|---|---|
| 400 | `BAD_REQUEST` | Validation failed on the request body/params |
| 401 | `UNAUTHORIZED` | Missing, invalid, or expired token |
| 403 | `FORBIDDEN` | Authenticated but not allowed to access this resource |
| 404 | `NOT_FOUND` | Resource does not exist or does not belong to the caller |
| 409 | `CONFLICT` | Duplicate resource (e.g. email already registered) |
| 422 | `UNPROCESSABLE_ENTITY` | Well-formed request that fails business rules (e.g. unsupported file type) |
| 429 | `RATE_LIMITED` | Too many requests; see `Retry-After` header |
| 500 | `INTERNAL_ERROR` | Unexpected server error |

`details` is optional and only present for validation errors (400/422); it lists every field that failed, not just the first.

## 3. Auth Endpoints

### `POST /api/auth/register`
**Auth required:** No

Request:
```json
{
  "name": "Riya Sharma",
  "email": "riya.sharma@example.com",
  "password": "SecurePass123!"
}
```

Response `201 Created`:
```json
{
  "id": "6f1c2e2a-1e3a-4b8d-9f0d-2f6a9d8c1234",
  "name": "Riya Sharma",
  "email": "riya.sharma@example.com",
  "createdAt": "2026-07-04T09:30:00Z"
}
```

Errors: `409 CONFLICT` if email already registered; `400 BAD_REQUEST` for weak password or invalid email.

### `POST /api/auth/login`
**Auth required:** No

Request:
```json
{
  "email": "riya.sharma@example.com",
  "password": "SecurePass123!"
}
```

Response `200 OK`:
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "expiresIn": 900,
  "user": {
    "id": "6f1c2e2a-1e3a-4b8d-9f0d-2f6a9d8c1234",
    "name": "Riya Sharma",
    "email": "riya.sharma@example.com"
  }
}
```

Errors: `401 UNAUTHORIZED` for wrong credentials.

### `POST /api/auth/refresh`
**Auth required:** No (refresh token instead)

Request:
```json
{ "refreshToken": "eyJhbGciOiJIUzI1NiIs..." }
```

Response `200 OK`:
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "expiresIn": 900
}
```

Errors: `401 UNAUTHORIZED` if refresh token is expired or revoked.

### `POST /api/auth/logout`
**Auth required:** Yes

Response `204 No Content`.

### `GET /api/auth/me`
**Auth required:** Yes

Response `200 OK`:
```json
{
  "id": "6f1c2e2a-1e3a-4b8d-9f0d-2f6a9d8c1234",
  "name": "Riya Sharma",
  "email": "riya.sharma@example.com",
  "targetRole": "Backend Engineer",
  "targetCompanies": ["Google", "Amazon"],
  "experienceLevel": "fresher",
  "createdAt": "2026-07-04T09:30:00Z"
}
```

## 4. Resume Endpoints

### `POST /api/resumes/upload`
**Auth required:** Yes · `Content-Type: multipart/form-data`

Request: multipart field `file` (PDF or DOCX, max 10 MB).

Response `201 Created`:
```json
{
  "id": "a3b1f9e0-1234-4d5e-9f6a-1234567890ab",
  "fileName": "riya_resume_v3.pdf",
  "version": 3,
  "isActive": true,
  "createdAt": "2026-07-04T09:31:00Z"
}
```

Errors: `422 UNPROCESSABLE_ENTITY` for unsupported file type; `400 BAD_REQUEST` if file exceeds 10 MB.

### `GET /api/resumes`
**Auth required:** Yes

Response `200 OK`:
```json
{
  "resumes": [
    {
      "id": "a3b1f9e0-1234-4d5e-9f6a-1234567890ab",
      "fileName": "riya_resume_v3.pdf",
      "version": 3,
      "isActive": true,
      "atsScore": 78,
      "createdAt": "2026-07-04T09:31:00Z"
    }
  ]
}
```

### `GET /api/resumes/{id}`
**Auth required:** Yes

Response `200 OK`:
```json
{
  "id": "a3b1f9e0-1234-4d5e-9f6a-1234567890ab",
  "fileName": "riya_resume_v3.pdf",
  "version": 3,
  "atsScore": 78,
  "parsedJson": {
    "sections": ["Summary", "Skills", "Experience", "Education", "Projects"],
    "skills": ["Java", "Spring Boot", "PostgreSQL"]
  },
  "createdAt": "2026-07-04T09:31:00Z"
}
```

Errors: `404 NOT_FOUND` if the resume doesn't exist or doesn't belong to the caller.

### `POST /api/resumes/{id}/analyze`
**Auth required:** Yes

Triggers an async ATS analysis job. No request body.

Response `202 Accepted`:
```json
{
  "resumeId": "a3b1f9e0-1234-4d5e-9f6a-1234567890ab",
  "status": "PROCESSING",
  "pollUrl": "/api/resumes/a3b1f9e0-1234-4d5e-9f6a-1234567890ab/analysis"
}
```

### `GET /api/resumes/{id}/analysis`
**Auth required:** Yes

Response `200 OK` (while processing):
```json
{ "status": "PROCESSING" }
```

Response `200 OK` (completed):
```json
{
  "status": "COMPLETED",
  "atsScore": 78,
  "breakdown": {
    "keywordMatch": 28,
    "sectionCompleteness": 18,
    "actionVerbsQuality": 11,
    "quantificationPresence": 10,
    "formattingCleanliness": 8,
    "contactInfoCompleteness": 3
  },
  "keywordGaps": ["Kubernetes", "system design"],
  "formattingIssues": ["Multi-column layout detected; may confuse ATS parsers"]
}
```

Errors: `404 NOT_FOUND` if no analysis has been triggered yet.

### `DELETE /api/resumes/{id}`
**Auth required:** Yes

Response `204 No Content`.

## 5. Interview Endpoints

### `POST /api/interviews/sessions`
**Auth required:** Yes

Request:
```json
{
  "resumeId": "a3b1f9e0-1234-4d5e-9f6a-1234567890ab",
  "interviewType": "technical",
  "targetCompany": "Google",
  "targetRole": "SDE-2",
  "difficulty": "medium",
  "durationMinutes": 30
}
```

Response `201 Created`:
```json
{
  "id": "9d8e7f6a-5b4c-3d2e-1f0a-9876543210ab",
  "status": "in_progress",
  "questions": [
    { "id": "q1", "text": "Explain how a hash map resolves collisions." }
  ],
  "createdAt": "2026-07-04T09:40:00Z"
}
```

### `GET /api/interviews/sessions/{id}`
**Auth required:** Yes

Response `200 OK`:
```json
{
  "id": "9d8e7f6a-5b4c-3d2e-1f0a-9876543210ab",
  "status": "in_progress",
  "interviewType": "technical",
  "targetCompany": "Google",
  "targetRole": "SDE-2",
  "currentQuestionIndex": 1,
  "questions": [
    { "id": "q1", "text": "Explain how a hash map resolves collisions." }
  ]
}
```

### `POST /api/interviews/sessions/{id}/answer`
**Auth required:** Yes

Request:
```json
{
  "questionId": "q1",
  "answerText": "A hash map resolves collisions using chaining or open addressing..."
}
```

Response `200 OK`:
```json
{
  "questionId": "q1",
  "scores": {
    "technicalAccuracy": 26,
    "clarityAndStructure": 20,
    "depth": 15,
    "relevance": 14,
    "starFormat": null
  },
  "feedback": "Good coverage of chaining; mention load factor and resizing for full marks.",
  "nextQuestionId": "q2"
}
```

Errors: `404 NOT_FOUND` if `questionId` doesn't belong to the session; `409 CONFLICT` if the session is already completed.

### `POST /api/interviews/sessions/{id}/hint`
**Auth required:** Yes

Request:
```json
{ "questionId": "q2" }
```

Response `200 OK`:
```json
{
  "questionId": "q2",
  "hint": "Think about what happens to time complexity as the load factor increases."
}
```

### `POST /api/interviews/sessions/{id}/complete`
**Auth required:** Yes

Response `200 OK`:
```json
{
  "id": "9d8e7f6a-5b4c-3d2e-1f0a-9876543210ab",
  "status": "completed",
  "overallScore": 82,
  "durationMinutes": 28,
  "feedback": [
    { "questionId": "q1", "summary": "Strong fundamentals, missed resizing behavior." }
  ],
  "completedAt": "2026-07-04T10:08:00Z"
}
```

### `GET /api/interviews/sessions`
**Auth required:** Yes

Response `200 OK`:
```json
{
  "sessions": [
    {
      "id": "9d8e7f6a-5b4c-3d2e-1f0a-9876543210ab",
      "interviewType": "technical",
      "targetCompany": "Google",
      "overallScore": 82,
      "status": "completed",
      "createdAt": "2026-07-04T09:40:00Z"
    }
  ]
}
```

## 6. Job Tracker Endpoints

> Derived from the Job Application data model (PRD §7.4) and Module 6 feature list (PRD §5.6). Confirm exact field names against the implementation before publishing.

### `POST /api/applications`
**Auth required:** Yes

Request:
```json
{
  "companyName": "Flipkart",
  "roleTitle": "SDE-1",
  "jdUrl": "https://flipkart.com/careers/sde-1",
  "jdText": "We are looking for an SDE-1 with strong fundamentals...",
  "resumeId": "a3b1f9e0-1234-4d5e-9f6a-1234567890ab",
  "status": "wishlist"
}
```

Response `201 Created`:
```json
{
  "id": "c1d2e3f4-5678-90ab-cdef-1234567890ab",
  "companyName": "Flipkart",
  "roleTitle": "SDE-1",
  "status": "wishlist",
  "atsScoreAtApply": null,
  "createdAt": "2026-07-04T10:15:00Z"
}
```

### `GET /api/applications`
**Auth required:** Yes

Query params: `status` (optional filter, e.g. `?status=applied`)

Response `200 OK`:
```json
{
  "applications": [
    {
      "id": "c1d2e3f4-5678-90ab-cdef-1234567890ab",
      "companyName": "Flipkart",
      "roleTitle": "SDE-1",
      "status": "wishlist",
      "appliedDate": null,
      "followUpDate": null
    }
  ]
}
```

### `GET /api/applications/{id}`
**Auth required:** Yes

Response `200 OK`:
```json
{
  "id": "c1d2e3f4-5678-90ab-cdef-1234567890ab",
  "companyName": "Flipkart",
  "roleTitle": "SDE-1",
  "jdUrl": "https://flipkart.com/careers/sde-1",
  "status": "wishlist",
  "notes": "Referral from Priya",
  "resumeId": "a3b1f9e0-1234-4d5e-9f6a-1234567890ab",
  "createdAt": "2026-07-04T10:15:00Z"
}
```

Errors: `404 NOT_FOUND` if the application doesn't exist or doesn't belong to the caller.

### `PATCH /api/applications/{id}`
**Auth required:** Yes

Request (partial update — e.g. moving a Kanban card):
```json
{
  "status": "applied",
  "appliedDate": "2026-07-04",
  "atsScoreAtApply": 78
}
```

Response `200 OK`:
```json
{
  "id": "c1d2e3f4-5678-90ab-cdef-1234567890ab",
  "status": "applied",
  "appliedDate": "2026-07-04",
  "atsScoreAtApply": 78,
  "updatedAt": "2026-07-04T10:20:00Z"
}
```

### `DELETE /api/applications/{id}`
**Auth required:** Yes

Response `204 No Content`.

### `GET /api/applications/analytics`
**Auth required:** Yes

Response `200 OK`:
```json
{
  "totalApplications": 42,
  "responseRate": 0.31,
  "averageTimeInStageDays": {
    "applied": 5.2,
    "screening": 3.1,
    "interview": 9.4
  },
  "statusBreakdown": {
    "wishlist": 10,
    "applied": 20,
    "screening": 5,
    "interview": 4,
    "offer": 1,
    "rejected": 2
  }
}
```

### `POST /api/applications/import`
**Auth required:** Yes · `Content-Type: multipart/form-data`

CSV import from LinkedIn Easy Apply history. Request: multipart field `file` (CSV).

Response `200 OK`:
```json
{
  "imported": 15,
  "skipped": 2,
  "errors": [
    { "row": 7, "issue": "missing companyName" }
  ]
}
```

## 7. Rate Limiting

All endpoints are subject to per-user rate limiting enforced at the API Gateway. Exceeding the limit returns:

```json
{
  "timestamp": "2026-07-04T10:25:00Z",
  "status": 429,
  "error": "RATE_LIMITED",
  "message": "Too many requests. Try again in 30 seconds.",
  "path": "/api/resumes/upload"
}
```

with a `Retry-After: 30` header.