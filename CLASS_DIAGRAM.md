# LearnCraft - Class Diagram

> **⚠️ Core Requirements**: Classes are designed around the core requirements defined in [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md).

## Table of Contents
1. [Overview](#overview)
2. [Domain Layer Classes](#domain-layer-classes)
3. [Service Layer Classes](#service-layer-classes)
4. [Infrastructure Layer Classes](#infrastructure-layer-classes)
5. [Complete Class Diagram](#complete-class-diagram)
6. [Class Relationships](#class-relationships)

---

## Overview

The LearnCraft platform follows a **layered architecture** with clear separation of concerns:

- **Domain Layer**: Core business entities and value objects
- **Service Layer**: Business logic and orchestration
- **Infrastructure Layer**: External integrations and data access

---

## Domain Layer Classes

### Core Domain Classes

```mermaid
classDiagram
    class User {
        +UUID id
        +String email
        +String passwordHash
        +String firstName
        +String lastName
        +UserRole role
        +SubscriptionTier subscription
        +DateTime createdAt
        +DateTime lastLoginAt
        +Boolean isActive
        +getFullName() String
        +hasActiveSubscription() Boolean
        +canAccessLabs() Boolean
    }
    
    class Course {
        +UUID id
        +String title
        +String slug
        +String description
        +UUID instructorId
        +Category category
        +SkillLevel level
        +CourseStatus status
        +Integer durationMinutes
        +Decimal price
        +DateTime publishedAt
        +List~Module~ modules
        +getTotalLessons() Integer
        +getTotalLabs() Integer
        +isPublished() Boolean
    }
    
    class Module {
        +UUID id
        +UUID courseId
        +String title
        +String description
        +Integer orderIndex
        +List~Lesson~ lessons
        +getDurationMinutes() Integer
    }
    
    class Lesson {
        +UUID id
        +UUID moduleId
        +String title
        +LessonType type
        +Integer orderIndex
        +Integer durationMinutes
        +Boolean isFree
        +getContent() LessonContent
    }
    
    class Video {
        +UUID id
        +UUID lessonId
        +String title
        +Integer durationSeconds
        +String hlsUrl
        +String thumbnailUrl
        +VideoStatus status
        +String transcript
        +List~VideoQuality~ qualities
        +getStreamingUrl(quality) String
    }
    
    class Lab {
        +UUID id
        +UUID lessonId
        +String title
        +String description
        +UUID templateId
        +Integer durationMinutes
        +LabDifficulty difficulty
        +String instructions
        +JSON successCriteria
        +getTemplate() LabTemplate
    }
    
    class LabTemplate {
        +UUID id
        +String name
        +String baseImage
        +String initScript
        +JSON resourceLimits
        +List~Integer~ ports
        +TechStack techStack
        +provision() LabSession
    }
    
    class LabSession {
        +UUID id
        +UUID labId
        +UUID userId
        +SessionStatus status
        +String containerId
        +String terminalUrl
        +DateTime startedAt
        +DateTime expiresAt
        +Integer resetCount
        +isActive() Boolean
        +extend(minutes) void
        +reset() void
        +terminate() void
    }

    Course "1" --> "*" Module
    Module "1" --> "*" Lesson
    Lesson "1" --> "0..1" Video
    Lesson "1" --> "0..1" Lab
    Lab "1" --> "1" LabTemplate
    Lab "1" --> "*" LabSession
    LabSession "*" --> "1" User
```

### Enrollment & Progress Classes

```mermaid
classDiagram
    class Enrollment {
        +UUID id
        +UUID userId
        +UUID courseId
        +EnrollmentStatus status
        +DateTime enrolledAt
        +DateTime completedAt
        +Decimal pricePaid
        +isActive() Boolean
        +getProgress() CourseProgress
    }
    
    class CourseProgress {
        +UUID id
        +UUID enrollmentId
        +Integer completedLessons
        +Integer totalLessons
        +Integer completedLabs
        +Integer totalLabs
        +Integer timeSpentMinutes
        +DateTime lastAccessedAt
        +getPercentComplete() Integer
    }
    
    class LessonProgress {
        +UUID id
        +UUID userId
        +UUID lessonId
        +ProgressStatus status
        +Integer videoPosition
        +Integer videoDuration
        +Boolean labCompleted
        +DateTime startedAt
        +DateTime completedAt
        +getPercentComplete() Integer
    }
    
    class Achievement {
        +UUID id
        +String name
        +String description
        +String iconUrl
        +AchievementType type
        +JSON criteria
    }
    
    class UserAchievement {
        +UUID id
        +UUID userId
        +UUID achievementId
        +DateTime earnedAt
    }
    
    class Certificate {
        +UUID id
        +UUID userId
        +UUID courseId
        +String certificateNumber
        +String pdfUrl
        +String verificationUrl
        +DateTime issuedAt
        +verify() Boolean
        +generatePdf() String
    }

    Enrollment "1" --> "1" CourseProgress
    User "1" --> "*" Enrollment
    User "1" --> "*" LessonProgress
    User "1" --> "*" UserAchievement
    UserAchievement "*" --> "1" Achievement
    User "1" --> "*" Certificate
    Certificate "*" --> "1" Course
```

### AI & Interaction Classes

```mermaid
classDiagram
    class AIInteraction {
        +UUID id
        +UUID userId
        +UUID sessionId
        +InteractionType type
        +String inputText
        +String outputText
        +JSON context
        +Integer feedbackRating
        +DateTime createdAt
        +logFeedback(rating) void
    }
    
    class ContentEmbedding {
        +UUID id
        +UUID contentId
        +ContentType contentType
        +Integer chunkIndex
        +Vector embedding
        +String chunkText
        +searchSimilar(query, limit) List
    }
    
    class Recommendation {
        +UUID id
        +UUID userId
        +UUID courseId
        +RecommendationType type
        +Float score
        +String reason
        +DateTime createdAt
    }
    
    class LearningPath {
        +UUID id
        +UUID userId
        +String name
        +List~UUID~ courseIds
        +PathStatus status
        +DateTime createdAt
        +getCourses() List~Course~
        +getProgress() Integer
    }

    User "1" --> "*" AIInteraction
    User "1" --> "*" Recommendation
    User "1" --> "*" LearningPath
```

### Assessment Classes

```mermaid
classDiagram
    class Assessment {
        +UUID id
        +UUID lessonId
        +String title
        +AssessmentType type
        +Integer passingScore
        +Integer timeLimitMinutes
        +Boolean isProctored
        +List~Question~ questions
        +getTotalPoints() Integer
    }
    
    class Question {
        +UUID id
        +UUID assessmentId
        +QuestionType type
        +String content
        +List~String~ options
        +String correctAnswer
        +Integer points
        +String explanation
    }
    
    class AssessmentAttempt {
        +UUID id
        +UUID userId
        +UUID assessmentId
        +Integer score
        +Integer totalPoints
        +AttemptStatus status
        +DateTime startedAt
        +DateTime completedAt
        +List~Answer~ answers
        +isPassing() Boolean
    }
    
    class Answer {
        +UUID id
        +UUID attemptId
        +UUID questionId
        +String response
        +Boolean isCorrect
        +Integer pointsEarned
    }

    Assessment "1" --> "*" Question
    AssessmentAttempt "*" --> "1" Assessment
    AssessmentAttempt "*" --> "1" User
    AssessmentAttempt "1" --> "*" Answer
```

---

## Service Layer Classes

### Core Services

```mermaid
classDiagram
    class UserService {
        -UserRepository userRepo
        -AuthService authService
        +register(dto) User
        +authenticate(email, password) AuthToken
        +getProfile(userId) UserProfile
        +updateProfile(userId, dto) User
        +changePassword(userId, dto) void
        +deleteAccount(userId) void
    }
    
    class CourseService {
        -CourseRepository courseRepo
        -EnrollmentService enrollmentService
        +getCatalog(filters) Page~Course~
        +getCourse(courseId) CourseDetail
        +searchCourses(query) List~Course~
        +getRecommendations(userId) List~Course~
    }
    
    class ContentService {
        -VideoRepository videoRepo
        -CDNService cdnService
        -TranscodingService transcoder
        +uploadVideo(file, metadata) Video
        +getStreamingUrl(videoId, quality) String
        +processVideo(videoId) Job
        +generateThumbnails(videoId) List~String~
    }
    
    class LabService {
        -LabRepository labRepo
        -ResourceManager resourceManager
        -SessionManager sessionManager
        +startSession(userId, labId) LabSession
        +getSession(sessionId) LabSession
        +resetSession(sessionId) LabSession
        +extendSession(sessionId, minutes) void
        +terminateSession(sessionId) void
        +validateCompletion(sessionId) ValidationResult
    }
    
    class AIService {
        -LLMClient llmClient
        -VectorDB vectorDb
        -ContextBuilder contextBuilder
        +generateSummary(videoId) Summary
        +getHint(sessionId, context) Hint
        +answerQuestion(query, context) Response
        +getRecommendations(userId) List~Course~
    }
    
    class ProgressService {
        -ProgressRepository progressRepo
        -EventPublisher eventPublisher
        +updateVideoProgress(userId, videoId, position) void
        +updateLabProgress(userId, labId, status) void
        +getCourseProgress(userId, courseId) CourseProgress
        +getDashboard(userId) ProgressDashboard
        +checkCompletionAndCertify(userId, courseId) Certificate
    }

    CourseService --> ContentService
    LabService --> ProgressService
    AIService --> ContentService
```

### Supporting Services

```mermaid
classDiagram
    class EnrollmentService {
        -EnrollmentRepository enrollmentRepo
        -PaymentService paymentService
        +enroll(userId, courseId) Enrollment
        +unenroll(userId, courseId) void
        +getEnrollments(userId) List~Enrollment~
        +isEnrolled(userId, courseId) Boolean
    }
    
    class PaymentService {
        -PaymentGateway gateway
        -SubscriptionRepository subRepo
        +createCheckout(userId, plan) CheckoutSession
        +processPayment(sessionId) Payment
        +cancelSubscription(userId) void
        +getSubscription(userId) Subscription
    }
    
    class NotificationService {
        -EmailProvider emailProvider
        -PushProvider pushProvider
        +sendEnrollmentConfirmation(userId, courseId) void
        +sendCourseReminder(userId) void
        +sendCertificateNotification(userId, certId) void
        +sendLabExpirationWarning(sessionId) void
    }
    
    class AnalyticsService {
        -AnalyticsRepository analyticsRepo
        +trackEvent(event) void
        +getInstructorAnalytics(instructorId) InstructorStats
        +getLearnerAnalytics(userId) LearnerStats
        +getContentAnalytics(courseId) ContentStats
    }
    
    class CertificateService {
        -CertificateRepository certRepo
        -PDFGenerator pdfGenerator
        -StorageService storage
        +issueCertificate(userId, courseId) Certificate
        +getCertificate(certId) Certificate
        +verifyCertificate(certNumber) VerificationResult
    }
    
    class SearchService {
        -ElasticsearchClient esClient
        +search(query, filters) SearchResults
        +indexCourse(course) void
        +indexLesson(lesson) void
        +suggest(partialQuery) List~String~
    }

    EnrollmentService --> PaymentService
    ProgressService --> CertificateService
```

---

## Infrastructure Layer Classes

```mermaid
classDiagram
    class LabProvisioner {
        -KubernetesClient k8sClient
        -ContainerPool containerPool
        +provision(template) Container
        +deprovision(containerId) void
        +getStatus(containerId) ContainerStatus
    }
    
    class ContainerPool {
        -Map~TechStack, Queue~ warmContainers
        +acquire(techStack) Container
        +release(container) void
        +warmUp(techStack, count) void
        +getPoolSize(techStack) Integer
    }
    
    class TerminalService {
        -WebSocketServer wsServer
        +connect(sessionId, userId) WebSocketConnection
        +sendInput(sessionId, input) void
        +receiveOutput(sessionId) Stream~Output~
        +disconnect(sessionId) void
    }
    
    class CDNService {
        -CloudFrontClient cfClient
        +getSignedUrl(path, expiry) String
        +invalidateCache(paths) void
        +uploadFile(file, path) String
    }
    
    class TranscodingService {
        -FFmpegRunner ffmpeg
        -JobQueue jobQueue
        +transcode(videoId, formats) Job
        +extractAudio(videoId) String
        +generateSubtitles(videoId) List~Subtitle~
    }
    
    class LLMClient {
        -OpenAIClient openai
        -PromptBuilder promptBuilder
        +complete(prompt, options) Completion
        +embed(text) Vector
        +stream(prompt, callback) void
    }
    
    class VectorDB {
        -PineconeClient pinecone
        +upsert(id, vector, metadata) void
        +query(vector, topK, filter) List~Match~
        +delete(id) void
    }

    LabProvisioner --> ContainerPool
    AIService --> LLMClient
    AIService --> VectorDB
```

---

## Complete Class Diagram

```mermaid
classDiagram
    %% Domain Layer
    class User {
        +UUID id
        +String email
        +UserRole role
        +SubscriptionTier subscription
    }
    
    class Course {
        +UUID id
        +String title
        +UUID instructorId
        +CourseStatus status
    }
    
    class Module {
        +UUID id
        +UUID courseId
        +Integer orderIndex
    }
    
    class Lesson {
        +UUID id
        +UUID moduleId
        +LessonType type
    }
    
    class Video {
        +UUID id
        +UUID lessonId
        +String hlsUrl
    }
    
    class Lab {
        +UUID id
        +UUID lessonId
        +UUID templateId
    }
    
    class LabSession {
        +UUID id
        +UUID userId
        +UUID labId
        +SessionStatus status
    }
    
    class Enrollment {
        +UUID id
        +UUID userId
        +UUID courseId
    }
    
    class Certificate {
        +UUID id
        +UUID userId
        +UUID courseId
    }
    
    %% Relationships
    User "1" --> "*" Enrollment
    User "1" --> "*" LabSession
    User "1" --> "*" Certificate
    Course "1" --> "*" Module
    Course "1" --> "*" Enrollment
    Module "1" --> "*" Lesson
    Lesson "1" --> "0..1" Video
    Lesson "1" --> "0..1" Lab
    Lab "1" --> "*" LabSession
    
    %% Service Layer
    class CourseService {
        +getCatalog()
        +getCourse()
    }
    
    class LabService {
        +startSession()
        +validateCompletion()
    }
    
    class AIService {
        +generateSummary()
        +getHint()
    }
    
    class ProgressService {
        +updateProgress()
        +getDashboard()
    }
    
    CourseService --> Course
    LabService --> Lab
    LabService --> LabSession
    ProgressService --> Enrollment
```

---

## Class Relationships

### Relationship Types

| Relationship | Description | Example |
|--------------|-------------|---------|
| **Inheritance** | Is-a relationship | VideoLesson → Lesson |
| **Composition** | Strong ownership, lifecycle dependent | Course → Module → Lesson |
| **Aggregation** | Weak ownership, independent lifecycle | User → Enrollment |
| **Association** | Uses / interacts with | LabService → LabSession |
| **Dependency** | Temporary usage | AIService → LLMClient |

### Key Relationships

1. **Course → Module → Lesson**: Composition - deleting a course deletes all modules and lessons
2. **User → Enrollment**: Aggregation - enrollments reference users but have independent lifecycle
3. **Lab → LabSession**: One lab template can spawn many sessions
4. **LabService → LabProvisioner**: Dependency for infrastructure operations
5. **AIService → VectorDB**: Dependency for semantic search

---

**Last Updated**: January 2026
**Version**: 1.0
**Status**: Design Complete, Implementation Pending
