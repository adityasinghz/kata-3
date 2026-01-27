# LearnCraft - API Specification

> **⚠️ Core Requirements**: APIs support the core requirements defined in [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md).

## Table of Contents
1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Course APIs](#course-apis)
4. [Enrollment APIs](#enrollment-apis)
5. [Content APIs](#content-apis)
6. [Lab APIs](#lab-apis)
7. [Progress APIs](#progress-apis)
8. [AI APIs](#ai-apis)
9. [Assessment APIs](#assessment-apis)
10. [User APIs](#user-apis)
11. [Admin APIs](#admin-apis)
12. [Webhooks](#webhooks)
13. [Error Handling](#error-handling)
14. [Rate Limiting](#rate-limiting)

---

## Overview

### Base URL
```
Production: https://api.learncraft.io/v1
Staging: https://api-staging.learncraft.io/v1
```

### API Conventions
- **REST**: RESTful API design
- **JSON**: All requests and responses use JSON
- **UTC**: All timestamps in ISO 8601 format (UTC)
- **Pagination**: Cursor-based pagination for list endpoints
- **Versioning**: API version in URL path (`/v1/`)

### Common Headers
```
Authorization: Bearer <token>
Content-Type: application/json
Accept: application/json
X-Request-ID: <uuid>
```

---

## Authentication

### POST /auth/register
Register a new user account.

**Request:**
```json
{
  "email": "learner@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe"
}
```

**Response (201 Created):**
```json
{
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "learner@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "learner",
    "subscription": "free"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### POST /auth/login
Authenticate user and get access token.

**Request:**
```json
{
  "email": "learner@example.com",
  "password": "SecurePass123!"
}
```

**Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
  "expiresIn": 3600,
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "learner@example.com",
    "role": "learner"
  }
}
```

---

### POST /auth/refresh
Refresh access token.

**Request:**
```json
{
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4..."
}
```

**Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600
}
```

---

## Course APIs

### GET /courses
Get catalog of courses with filtering.

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| category | string | Filter by category |
| level | string | Filter by skill level |
| search | string | Search query |
| instructor | uuid | Filter by instructor |
| featured | boolean | Featured courses only |
| cursor | string | Pagination cursor |
| limit | integer | Results per page (max 50) |

**Response (200 OK):**
```json
{
  "courses": [
    {
      "id": "course-uuid",
      "title": "Python for Beginners",
      "slug": "python-for-beginners",
      "shortDescription": "Learn Python from scratch",
      "instructor": {
        "id": "instructor-uuid",
        "name": "Jane Smith"
      },
      "category": "programming_languages",
      "level": "beginner",
      "durationMinutes": 480,
      "price": 49.99,
      "thumbnailUrl": "https://cdn.learncraft.io/...",
      "rating": 4.8,
      "enrollmentCount": 15234,
      "modulesCount": 8,
      "lessonsCount": 42,
      "hasLabs": true
    }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6Imxhc3QtaWQifQ==",
    "hasMore": true
  }
}
```

---

### GET /courses/{id}
Get detailed course information.

**Response (200 OK):**
```json
{
  "id": "course-uuid",
  "title": "Python for Beginners",
  "slug": "python-for-beginners",
  "description": "Complete Python course...",
  "instructor": {
    "id": "instructor-uuid",
    "name": "Jane Smith",
    "avatarUrl": "https://...",
    "bio": "Software engineer with 10+ years..."
  },
  "category": "programming_languages",
  "level": "beginner",
  "durationMinutes": 480,
  "price": 49.99,
  "thumbnailUrl": "https://cdn.learncraft.io/...",
  "previewVideoUrl": "https://cdn.learncraft.io/...",
  "rating": 4.8,
  "ratingCount": 2341,
  "enrollmentCount": 15234,
  "modules": [
    {
      "id": "module-uuid",
      "title": "Getting Started",
      "description": "Introduction to Python",
      "orderIndex": 1,
      "lessons": [
        {
          "id": "lesson-uuid",
          "title": "Installing Python",
          "type": "video",
          "durationMinutes": 12,
          "isFree": true,
          "hasLab": false
        },
        {
          "id": "lesson-uuid-2",
          "title": "Your First Python Program",
          "type": "mixed",
          "durationMinutes": 25,
          "isFree": false,
          "hasLab": true
        }
      ]
    }
  ],
  "isEnrolled": false,
  "requirements": ["Basic computer skills"],
  "objectives": ["Write Python programs", "Understand OOP"],
  "publishedAt": "2026-01-15T10:00:00Z"
}
```

---

## Enrollment APIs

### POST /courses/{id}/enroll
Enroll in a course.

**Response (201 Created):**
```json
{
  "enrollment": {
    "id": "enrollment-uuid",
    "courseId": "course-uuid",
    "status": "active",
    "enrolledAt": "2026-01-27T10:00:00Z"
  },
  "message": "Successfully enrolled in the course"
}
```

**Response (402 Payment Required):**
```json
{
  "error": "payment_required",
  "message": "This course requires payment",
  "checkoutUrl": "https://checkout.learncraft.io/..."
}
```

---

### GET /enrollments
Get user's enrollments.

**Response (200 OK):**
```json
{
  "enrollments": [
    {
      "id": "enrollment-uuid",
      "course": {
        "id": "course-uuid",
        "title": "Python for Beginners",
        "thumbnailUrl": "https://..."
      },
      "status": "active",
      "progress": {
        "percentComplete": 45,
        "completedLessons": 19,
        "totalLessons": 42,
        "lastAccessedAt": "2026-01-26T15:30:00Z"
      },
      "enrolledAt": "2026-01-10T10:00:00Z"
    }
  ]
}
```

---

## Content APIs

### GET /lessons/{id}
Get lesson content.

**Response (200 OK):**
```json
{
  "id": "lesson-uuid",
  "title": "Your First Python Program",
  "description": "Write and run your first Python code",
  "type": "mixed",
  "moduleId": "module-uuid",
  "orderIndex": 2,
  "durationMinutes": 25,
  "video": {
    "id": "video-uuid",
    "durationSeconds": 720,
    "thumbnailUrl": "https://cdn.learncraft.io/...",
    "qualities": ["360p", "720p", "1080p"]
  },
  "lab": {
    "id": "lab-uuid",
    "title": "Hello World Lab",
    "durationMinutes": 30,
    "difficulty": "easy"
  },
  "resources": [
    {
      "type": "pdf",
      "title": "Python Cheatsheet",
      "url": "https://cdn.learncraft.io/..."
    }
  ],
  "progress": {
    "status": "in_progress",
    "videoPosition": 120,
    "videoCompleted": false,
    "labCompleted": false
  }
}
```

---

### GET /videos/{id}/stream
Get video streaming URL.

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| quality | string | Video quality (360p, 720p, 1080p) |

**Response (200 OK):**
```json
{
  "streamingUrl": "https://cdn.learncraft.io/videos/abc123/manifest.m3u8?token=...",
  "expiresAt": "2026-01-27T12:00:00Z",
  "quality": "720p",
  "durationSeconds": 720,
  "subtitles": [
    {
      "language": "en",
      "url": "https://cdn.learncraft.io/videos/abc123/en.vtt"
    }
  ]
}
```

---

### GET /videos/{id}/summary
Get AI-generated video summary.

**Response (200 OK):**
```json
{
  "videoId": "video-uuid",
  "summary": "This lesson covers the basics of writing your first Python program...",
  "keyPoints": [
    "Python uses print() function for output",
    "Indentation is significant in Python",
    "Variables don't need type declarations"
  ],
  "timestamps": [
    {
      "time": 0,
      "topic": "Introduction"
    },
    {
      "time": 120,
      "topic": "Writing your first program"
    },
    {
      "time": 360,
      "topic": "Running the program"
    }
  ]
}
```

---

## Lab APIs

### POST /labs/{id}/sessions
Start a new lab session.

**Response (201 Created):**
```json
{
  "session": {
    "id": "session-uuid",
    "labId": "lab-uuid",
    "status": "provisioning",
    "createdAt": "2026-01-27T10:00:00Z"
  },
  "message": "Lab environment is being prepared"
}
```

**Response (200 OK) - After provisioning:**
```json
{
  "session": {
    "id": "session-uuid",
    "labId": "lab-uuid",
    "status": "running",
    "terminalUrl": "wss://labs.learncraft.io/terminals/session-uuid",
    "ideUrl": "https://labs.learncraft.io/ide/session-uuid",
    "startedAt": "2026-01-27T10:00:05Z",
    "expiresAt": "2026-01-27T11:00:05Z",
    "timeRemainingSeconds": 3600
  },
  "instructions": "## Lab: Hello World\n\n1. Create a file called `hello.py`\n2. ...",
  "starterFiles": {
    "hello.py": "# Write your code here\n"
  }
}
```

---

### GET /labs/sessions/{id}
Get lab session status.

**Response (200 OK):**
```json
{
  "id": "session-uuid",
  "labId": "lab-uuid",
  "status": "running",
  "terminalUrl": "wss://labs.learncraft.io/terminals/session-uuid",
  "ideUrl": "https://labs.learncraft.io/ide/session-uuid",
  "startedAt": "2026-01-27T10:00:05Z",
  "expiresAt": "2026-01-27T11:00:05Z",
  "timeRemainingSeconds": 2400,
  "resetCount": 0
}
```

---

### POST /labs/sessions/{id}/reset
Reset lab environment to initial state.

**Response (200 OK):**
```json
{
  "session": {
    "id": "session-uuid",
    "status": "running",
    "resetCount": 1
  },
  "message": "Lab environment has been reset"
}
```

---

### POST /labs/sessions/{id}/extend
Extend lab session time.

**Request:**
```json
{
  "minutes": 30
}
```

**Response (200 OK):**
```json
{
  "session": {
    "id": "session-uuid",
    "expiresAt": "2026-01-27T11:30:05Z",
    "timeRemainingSeconds": 3600
  },
  "message": "Session extended by 30 minutes"
}
```

---

### POST /labs/sessions/{id}/validate
Validate lab completion.

**Response (200 OK):**
```json
{
  "passed": true,
  "score": 100,
  "feedback": "Excellent work! All tasks completed correctly.",
  "criteria": [
    {
      "name": "File created",
      "passed": true,
      "message": "hello.py file exists"
    },
    {
      "name": "Correct output",
      "passed": true,
      "message": "Program outputs 'Hello, World!'"
    }
  ]
}
```

---

### DELETE /labs/sessions/{id}
End lab session.

**Response (204 No Content)**

---

## Progress APIs

### POST /progress/video
Update video progress.

**Request:**
```json
{
  "videoId": "video-uuid",
  "positionSeconds": 240,
  "completed": false
}
```

**Response (200 OK):**
```json
{
  "progress": {
    "videoId": "video-uuid",
    "positionSeconds": 240,
    "durationSeconds": 720,
    "percentComplete": 33,
    "completed": false
  }
}
```

---

### GET /progress/dashboard
Get learner progress dashboard.

**Response (200 OK):**
```json
{
  "summary": {
    "totalCoursesEnrolled": 5,
    "completedCourses": 2,
    "inProgressCourses": 3,
    "totalLearningTimeMinutes": 1240,
    "currentStreak": 7,
    "longestStreak": 14
  },
  "recentCourses": [
    {
      "courseId": "course-uuid",
      "title": "Python for Beginners",
      "percentComplete": 45,
      "lastAccessedAt": "2026-01-26T15:30:00Z"
    }
  ],
  "achievements": [
    {
      "id": "achievement-uuid",
      "name": "First Steps",
      "description": "Complete your first lesson",
      "iconUrl": "https://...",
      "earnedAt": "2026-01-10T10:30:00Z"
    }
  ],
  "weeklyActivity": [
    {"day": "Mon", "minutes": 45},
    {"day": "Tue", "minutes": 30},
    {"day": "Wed", "minutes": 60}
  ]
}
```

---

### GET /progress/courses/{id}
Get detailed course progress.

**Response (200 OK):**
```json
{
  "courseId": "course-uuid",
  "percentComplete": 45,
  "completedLessons": 19,
  "totalLessons": 42,
  "completedLabs": 8,
  "totalLabs": 15,
  "timeSpentMinutes": 320,
  "lastAccessedAt": "2026-01-26T15:30:00Z",
  "modules": [
    {
      "moduleId": "module-uuid",
      "title": "Getting Started",
      "percentComplete": 100,
      "lessons": [
        {
          "lessonId": "lesson-uuid",
          "title": "Installing Python",
          "status": "completed",
          "completedAt": "2026-01-10T11:00:00Z"
        }
      ]
    }
  ]
}
```

---

## AI APIs

### POST /ai/chat
Ask a question to AI assistant.

**Request:**
```json
{
  "message": "What is a Python list?",
  "context": {
    "courseId": "course-uuid",
    "lessonId": "lesson-uuid",
    "sessionId": "session-uuid"
  }
}
```

**Response (200 OK):**
```json
{
  "response": "A Python list is an ordered, mutable collection that can hold items of different types. Lists are defined using square brackets `[]`. For example:\n\n```python\nfruits = ['apple', 'banana', 'cherry']\n```\n\nLists support indexing, slicing, and various methods like `append()`, `remove()`, and `sort()`.",
  "sources": [
    {
      "lessonId": "lesson-uuid",
      "title": "Working with Lists",
      "relevance": 0.95
    }
  ],
  "followUpQuestions": [
    "How do I add items to a list?",
    "What's the difference between a list and a tuple?"
  ]
}
```

---

### POST /ai/hint
Get a hint for current lab context.

**Request:**
```json
{
  "sessionId": "session-uuid",
  "context": {
    "currentFile": "hello.py",
    "fileContent": "print('Hello')",
    "lastError": null
  }
}
```

**Response (200 OK):**
```json
{
  "hint": "You're on the right track! The lab expects the output to be exactly 'Hello, World!' with the comma and exclamation mark. Try updating your print statement.",
  "hintLevel": 1,
  "maxHints": 3
}
```

---

### GET /ai/recommendations
Get personalized course recommendations.

**Response (200 OK):**
```json
{
  "recommendations": [
    {
      "courseId": "course-uuid",
      "title": "Advanced Python",
      "reason": "Based on your completion of Python for Beginners",
      "score": 0.95,
      "thumbnailUrl": "https://..."
    }
  ],
  "learningPath": {
    "nextCourse": {
      "courseId": "course-uuid",
      "title": "Python Data Structures"
    },
    "suggestedPath": [
      "Python for Beginners",
      "Python Data Structures",
      "Advanced Python"
    ]
  }
}
```

---

## Assessment APIs

### POST /assessments/{id}/start
Start an assessment attempt.

**Response (200 OK):**
```json
{
  "attempt": {
    "id": "attempt-uuid",
    "assessmentId": "assessment-uuid",
    "status": "in_progress",
    "startedAt": "2026-01-27T10:00:00Z",
    "expiresAt": "2026-01-27T10:30:00Z"
  },
  "questions": [
    {
      "id": "question-uuid",
      "type": "multiple_choice",
      "content": "What is the output of print(2 + 3)?",
      "options": ["23", "5", "2 + 3", "Error"],
      "points": 1
    }
  ]
}
```

---

### POST /assessments/attempts/{id}/submit
Submit assessment answers.

**Request:**
```json
{
  "answers": [
    {
      "questionId": "question-uuid",
      "response": "5"
    }
  ]
}
```

**Response (200 OK):**
```json
{
  "attempt": {
    "id": "attempt-uuid",
    "status": "completed",
    "score": 85,
    "totalPoints": 100,
    "percentScore": 85,
    "isPassing": true,
    "completedAt": "2026-01-27T10:25:00Z"
  },
  "results": [
    {
      "questionId": "question-uuid",
      "isCorrect": true,
      "pointsEarned": 1,
      "explanation": "The + operator adds numbers, so 2 + 3 = 5"
    }
  ]
}
```

---

## User APIs

### GET /users/me
Get current user profile.

**Response (200 OK):**
```json
{
  "id": "user-uuid",
  "email": "learner@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "avatarUrl": "https://...",
  "role": "learner",
  "subscription": {
    "tier": "pro",
    "status": "active",
    "expiresAt": "2027-01-27T00:00:00Z"
  },
  "preferences": {
    "language": "en",
    "theme": "dark",
    "emailNotifications": true
  },
  "createdAt": "2025-06-15T10:00:00Z"
}
```

---

### PATCH /users/me
Update user profile.

**Request:**
```json
{
  "firstName": "Jonathan",
  "bio": "Learning Python enthusiast"
}
```

**Response (200 OK):**
```json
{
  "id": "user-uuid",
  "firstName": "Jonathan",
  "bio": "Learning Python enthusiast",
  "updatedAt": "2026-01-27T10:00:00Z"
}
```

---

### GET /users/me/certificates
Get user's certificates.

**Response (200 OK):**
```json
{
  "certificates": [
    {
      "id": "cert-uuid",
      "courseId": "course-uuid",
      "courseTitle": "Python for Beginners",
      "certificateNumber": "LC-2026-000123",
      "pdfUrl": "https://cdn.learncraft.io/certs/...",
      "verificationUrl": "https://learncraft.io/verify/LC-2026-000123",
      "issuedAt": "2026-01-20T10:00:00Z"
    }
  ]
}
```

---

## Admin APIs

### GET /admin/courses/pending
Get courses pending review (Content Admin only).

**Response (200 OK):**
```json
{
  "courses": [
    {
      "id": "course-uuid",
      "title": "React Fundamentals",
      "instructor": {
        "id": "instructor-uuid",
        "name": "Jane Smith"
      },
      "submittedAt": "2026-01-26T10:00:00Z",
      "modulesCount": 6,
      "lessonsCount": 28,
      "labsCount": 12
    }
  ]
}
```

---

### POST /admin/courses/{id}/approve
Approve and publish a course (Content Admin only).

**Response (200 OK):**
```json
{
  "course": {
    "id": "course-uuid",
    "status": "published",
    "publishedAt": "2026-01-27T10:00:00Z"
  },
  "message": "Course has been published"
}
```

---

### POST /admin/courses/{id}/reject
Reject a course with feedback (Content Admin only).

**Request:**
```json
{
  "reason": "Video quality needs improvement in Module 3",
  "details": "Please re-record lessons 3.2 and 3.4 with better audio"
}
```

**Response (200 OK):**
```json
{
  "course": {
    "id": "course-uuid",
    "status": "revision_needed"
  },
  "message": "Feedback sent to instructor"
}
```

---

## Webhooks

### Supported Events
| Event | Description |
|-------|-------------|
| `enrollment.created` | User enrolled in a course |
| `enrollment.completed` | User completed a course |
| `certificate.issued` | Certificate was issued |
| `subscription.created` | New subscription started |
| `subscription.cancelled` | Subscription was cancelled |
| `course.published` | New course was published |

### Webhook Payload
```json
{
  "id": "webhook-uuid",
  "event": "enrollment.completed",
  "timestamp": "2026-01-27T10:00:00Z",
  "data": {
    "userId": "user-uuid",
    "courseId": "course-uuid",
    "completedAt": "2026-01-27T10:00:00Z"
  }
}
```

---

## Error Handling

### Error Response Format
```json
{
  "error": {
    "code": "invalid_request",
    "message": "The request was invalid",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

### HTTP Status Codes
| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Unprocessable Entity |
| 429 | Too Many Requests |
| 500 | Internal Server Error |

---

## Rate Limiting

### Default Limits
| Endpoint Category | Limit |
|-------------------|-------|
| Authentication | 10 requests/minute |
| Course APIs | 100 requests/minute |
| Lab APIs | 20 requests/minute |
| AI APIs | 30 requests/minute |
| Admin APIs | 50 requests/minute |

### Rate Limit Headers
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1706356800
```

---

**Last Updated**: January 2026
**Version**: 1.0
**Status**: Design Complete, Implementation Pending
