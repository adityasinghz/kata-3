# LearnCraft - Sequence Diagrams

> **⚠️ Core Requirements**: Each sequence diagram maps to the core requirements defined in [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md).

## Table of Contents
1. [Course Enrollment Flow](#course-enrollment-flow)
2. [Video Streaming Flow](#video-streaming-flow)
3. [Lab Session Lifecycle](#lab-session-lifecycle)
4. [AI Assistance Request Flow](#ai-assistance-request-flow)
5. [Progress Tracking Flow](#progress-tracking-flow)
6. [Content Publishing Flow](#content-publishing-flow)

---

## Course Enrollment Flow

**Requirement**: REQ-1 (Learning Content Delivery)
**Use Case**: UC2 (Enroll in Course)

```mermaid
sequenceDiagram
    autonumber
    participant L as Learner
    participant UI as Web App
    participant GW as API Gateway
    participant CS as Course Service
    participant US as User Service
    participant PS as Payment Service
    participant NS as Notification Service
    participant DB as Database

    L->>UI: Click "Enroll" on course
    UI->>GW: POST /api/courses/{id}/enroll
    GW->>GW: Authenticate token
    GW->>CS: enrollInCourse(userId, courseId)
    CS->>US: getUserSubscription(userId)
    US->>DB: SELECT subscription FROM users
    DB-->>US: Subscription details
    
    alt Course included in subscription
        US-->>CS: Subscription active
        CS->>DB: INSERT INTO enrollments
        DB-->>CS: Enrollment created
        CS->>NS: sendEnrollmentNotification(userId, courseId)
        NS-->>CS: Notification queued
        CS-->>GW: Enrollment success
        GW-->>UI: 201 Created
        UI-->>L: "Successfully enrolled!"
    else Requires purchase
        US-->>CS: Payment required
        CS-->>GW: Payment required
        GW-->>UI: 402 Payment Required
        UI->>L: Show payment options
        L->>UI: Select payment method
        UI->>GW: POST /api/payments/checkout
        GW->>PS: createCheckoutSession()
        PS-->>GW: Checkout URL
        GW-->>UI: Redirect to payment
        UI-->>L: Redirect to payment gateway
    end
```

---

## Video Streaming Flow

**Requirement**: REQ-1 (Learning Content Delivery)
**Use Case**: UC3 (Watch Video Lesson)

```mermaid
sequenceDiagram
    autonumber
    participant L as Learner
    participant UI as Web App
    participant VP as Video Player
    participant GW as API Gateway
    participant CS as Content Service
    participant PS as Progress Service
    participant CDN as CDN
    participant AI as AI Service

    L->>UI: Navigate to lesson
    UI->>GW: GET /api/lessons/{id}
    GW->>CS: getLessonContent(lessonId)
    CS-->>GW: Lesson metadata + video reference
    GW-->>UI: Lesson data
    UI->>VP: Initialize player
    
    VP->>GW: GET /api/videos/{id}/stream
    GW->>CS: getStreamingUrl(videoId, userId)
    CS->>CS: Generate signed CDN URL
    CS-->>GW: Signed streaming URL
    GW-->>VP: HLS manifest URL
    
    VP->>CDN: GET manifest.m3u8
    CDN-->>VP: HLS manifest
    VP->>CDN: GET segment_001.ts
    CDN-->>VP: Video segment
    VP->>L: Play video
    
    loop Every 30 seconds
        VP->>GW: POST /api/progress/video
        GW->>PS: updateVideoProgress(userId, videoId, position)
        PS-->>GW: Progress saved
    end
    
    VP->>GW: POST /api/progress/video (complete)
    GW->>PS: markVideoComplete(userId, videoId)
    PS-->>GW: Marked complete
    
    GW->>AI: getVideoSummary(videoId)
    AI-->>GW: AI-generated summary
    GW-->>UI: Show summary
    UI-->>L: Display video summary
```

---

## Lab Session Lifecycle

**Requirement**: REQ-2 (Hands-On Lab Environment)
**Use Case**: UC4, UC5 (Start Lab, Complete Lab)

```mermaid
sequenceDiagram
    autonumber
    participant L as Learner
    participant UI as Web App
    participant GW as API Gateway
    participant LS as Lab Service
    participant RM as Resource Manager
    participant K8s as Kubernetes
    participant TS as Terminal Service
    participant PS as Progress Service
    participant AI as AI Service

    L->>UI: Click "Start Lab"
    UI->>GW: POST /api/labs/{id}/sessions
    GW->>LS: createLabSession(userId, labId)
    
    LS->>LS: Check session limits
    LS->>RM: allocateResources(labSpec)
    RM->>K8s: Request pod from pool
    K8s-->>RM: Pod assigned
    RM->>K8s: Configure environment
    K8s-->>RM: Environment ready
    RM-->>LS: Container ID + terminal URL
    
    LS-->>GW: Session created (sessionId, terminalUrl)
    GW-->>UI: Lab session details
    UI->>TS: WebSocket connect
    TS-->>UI: Terminal connected
    UI-->>L: Lab environment ready
    
    loop During lab session
        L->>UI: Type command
        UI->>TS: Send input (WebSocket)
        TS->>K8s: Execute in container
        K8s-->>TS: Command output
        TS-->>UI: Output (WebSocket)
        UI-->>L: Display output
    end
    
    alt Request hint
        L->>UI: Click "Get Hint"
        UI->>GW: POST /api/ai/hint
        GW->>AI: getContextualHint(sessionId, context)
        AI-->>GW: Hint response
        GW-->>UI: Display hint
        UI-->>L: Show hint
    end
    
    L->>UI: Click "Check Solution"
    UI->>GW: POST /api/labs/sessions/{id}/validate
    GW->>LS: validateCompletion(sessionId)
    LS->>K8s: Run validation script
    K8s-->>LS: Validation result
    LS->>PS: updateLabProgress(userId, labId, result)
    PS-->>LS: Progress updated
    LS-->>GW: Validation result
    GW-->>UI: Show result
    UI-->>L: "Lab completed!" or feedback
    
    Note over LS,K8s: Session cleanup on timeout/complete
    LS->>RM: releaseResources(sessionId)
    RM->>K8s: Terminate pod
    K8s-->>RM: Pod terminated
```

---

## AI Assistance Request Flow

**Requirement**: REQ-3 (AI-Driven Learning Assistance)
**Use Case**: UC6 (Get AI Assistance)

```mermaid
sequenceDiagram
    autonumber
    participant L as Learner
    participant UI as Web App
    participant GW as API Gateway
    participant AI as AI Service
    participant VDB as Vector DB
    participant LLM as OpenAI API
    participant DB as Database

    L->>UI: Type question in chat
    UI->>GW: POST /api/ai/chat
    GW->>AI: processQuery(userId, query, context)
    
    AI->>AI: Extract learning context
    Note over AI: Context includes: current course,<br/>lesson, lab state, history
    
    AI->>VDB: searchSimilar(query, courseId)
    VDB-->>AI: Relevant content chunks
    
    AI->>AI: Build RAG prompt
    Note over AI: System prompt + context +<br/>relevant content + user query
    
    AI->>LLM: Generate response
    LLM-->>AI: AI response
    
    AI->>AI: Post-process response
    Note over AI: Add citations, format code,<br/>filter inappropriate content
    
    AI->>DB: Log interaction
    DB-->>AI: Logged
    
    AI-->>GW: Formatted response
    GW-->>UI: AI response
    UI-->>L: Display response
    
    opt Learner feedback
        L->>UI: Rate response (👍/👎)
        UI->>GW: POST /api/ai/feedback
        GW->>AI: logFeedback(interactionId, rating)
        AI->>DB: Store feedback
    end
```

---

## Progress Tracking Flow

**Requirement**: REQ-7 (Progress Tracking & Analytics)
**Use Case**: UC8 (View Progress)

```mermaid
sequenceDiagram
    autonumber
    participant L as Learner
    participant UI as Web App
    participant GW as API Gateway
    participant PS as Progress Service
    participant AS as Analytics Service
    participant Cache as Redis
    participant DB as Database

    L->>UI: Open Progress Dashboard
    UI->>GW: GET /api/progress/dashboard
    GW->>PS: getDashboard(userId)
    
    PS->>Cache: GET progress:{userId}
    alt Cache hit
        Cache-->>PS: Cached progress data
    else Cache miss
        PS->>DB: SELECT progress, enrollments, achievements
        DB-->>PS: Progress data
        PS->>Cache: SET progress:{userId}
    end
    
    PS->>AS: getLearningStats(userId)
    AS->>DB: SELECT time_spent, streaks, etc.
    DB-->>AS: Learning statistics
    AS-->>PS: Stats data
    
    PS-->>GW: Dashboard data
    GW-->>UI: Progress response
    UI-->>L: Display dashboard
    
    Note over L,UI: Dashboard shows:<br/>- Overall progress %<br/>- Per-course progress<br/>- Learning streak<br/>- Time spent<br/>- Achievements
    
    L->>UI: Click on specific course
    UI->>GW: GET /api/progress/courses/{id}
    GW->>PS: getCourseProgress(userId, courseId)
    PS->>DB: SELECT detailed progress
    DB-->>PS: Module/lesson progress
    PS-->>GW: Detailed progress
    GW-->>UI: Course progress
    UI-->>L: Show per-module breakdown
```

---

## Content Publishing Flow

**Requirement**: REQ-8 (Content Management)
**Use Case**: UC13 (Publish Course)

```mermaid
sequenceDiagram
    autonumber
    participant I as Instructor
    participant UI as Instructor Portal
    participant GW as API Gateway
    participant CMS as Content Management
    participant VS as Video Service
    participant LS as Lab Service
    participant NS as Notification Service
    participant CA as Content Admin

    I->>UI: Click "Submit for Review"
    UI->>GW: POST /api/courses/{id}/submit
    GW->>CMS: submitForReview(courseId)
    
    CMS->>CMS: Validate completeness
    Note over CMS: Check: all modules have content,<br/>videos processed, labs configured
    
    alt Validation fails
        CMS-->>GW: Validation errors
        GW-->>UI: 400 Validation failed
        UI-->>I: Show missing requirements
    else Validation passes
        CMS->>CMS: Update status to "PENDING_REVIEW"
        CMS->>NS: notifyReviewers(courseId)
        NS-->>CMS: Notification sent
        CMS-->>GW: Review submitted
        GW-->>UI: 200 Submitted
        UI-->>I: "Course submitted for review"
    end
    
    NS->>CA: Review notification
    CA->>UI: Open review queue
    UI->>GW: GET /api/admin/courses/{id}/review
    GW->>CMS: getCourseForReview(courseId)
    CMS-->>GW: Full course details
    GW-->>UI: Course content
    UI-->>CA: Display course for review
    
    CA->>UI: Test lab environment
    UI->>GW: POST /api/labs/{id}/test
    GW->>LS: testLabTemplate(labId)
    LS-->>GW: Lab test result
    GW-->>UI: Test results
    
    alt Review approved
        CA->>UI: Click "Approve"
        UI->>GW: POST /api/admin/courses/{id}/approve
        GW->>CMS: publishCourse(courseId)
        CMS->>CMS: Update status to "PUBLISHED"
        CMS->>VS: activateCDN(courseVideos)
        CMS->>LS: activateLabs(courseLabs)
        CMS->>NS: notifyInstructor(courseId, "approved")
        CMS-->>GW: Course published
        GW-->>UI: 200 Published
        NS->>I: "Your course is now live!"
    else Review rejected
        CA->>UI: Click "Request Changes"
        CA->>UI: Enter feedback
        UI->>GW: POST /api/admin/courses/{id}/reject
        GW->>CMS: rejectCourse(courseId, feedback)
        CMS->>CMS: Update status to "REVISION_NEEDED"
        CMS->>NS: notifyInstructor(courseId, feedback)
        NS->>I: Review feedback email
    end
```

---

## Certificate Issuance Flow

**Requirement**: REQ-7 (Progress Tracking)
**Use Case**: UC9 (Earn Certificate)

```mermaid
sequenceDiagram
    autonumber
    participant L as Learner
    participant UI as Web App
    participant GW as API Gateway
    participant PS as Progress Service
    participant CS as Certificate Service
    participant DB as Database
    participant S3 as Storage

    L->>UI: Complete final lesson/lab
    UI->>GW: POST /api/progress/complete
    GW->>PS: markComplete(userId, lessonId)
    PS->>DB: Update progress
    
    PS->>PS: Check course completion
    PS->>DB: SELECT COUNT(*) FROM completed_items
    DB-->>PS: All items completed?
    
    alt Course not fully complete
        PS-->>GW: Progress updated
        GW-->>UI: Progress saved
    else Course completed
        PS->>CS: issueCertificate(userId, courseId)
        CS->>CS: Generate certificate ID
        CS->>CS: Render certificate PDF
        CS->>S3: Upload certificate
        S3-->>CS: Certificate URL
        CS->>DB: INSERT INTO certificates
        DB-->>CS: Certificate record created
        CS-->>PS: Certificate issued
        PS-->>GW: Course completed + certificate
        GW-->>UI: Completion + certificate data
        UI-->>L: 🎉 "Congratulations! Certificate earned"
    end
    
    L->>UI: View certificate
    UI->>GW: GET /api/certificates/{id}
    GW->>CS: getCertificate(certId)
    CS->>DB: SELECT certificate
    DB-->>CS: Certificate data
    CS-->>GW: Certificate details + PDF URL
    GW-->>UI: Certificate
    UI-->>L: Display certificate preview
    
    L->>UI: Share on LinkedIn
    UI-->>L: LinkedIn share dialog
    Note over L,UI: Certificate includes<br/>verification URL
```

---

**Last Updated**: January 2026
**Version**: 1.0
**Status**: Design Complete, Implementation Pending
