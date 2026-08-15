# Project-Iris

## 1. Overview
A backend platform (Java, Spring Boot) to create, schedule, and publish posts to Instagram from a single interface, with room to expand to other platforms later.

## 2. Goals (MVP)
- Connect one Instagram Business account via Facebook Graph API
- Create a post (image/video + caption)
- Publish immediately or schedule for a future time
- Store post history and status (draft, scheduled, published, failed)

## 3. Tech Stack
- Language: Java 17+
- Framework: Spring Boot (Web, Scheduler, Data JPA, Security)
- Database: PostgreSQL
- API Integration: Instagram Graph API (via Facebook Login OAuth2)
- Build tool: Maven or Gradle
- Optional: Docker for containerization

## 4. Project Structure
```
social-scheduler/
  src/main/java/com/example/scheduler/
    config/          -> Spring config, security, scheduler beans
    auth/            -> OAuth2 flow with Facebook/Instagram
    post/
      controller/    -> REST endpoints for creating/managing posts
      service/       -> Business logic, publishing pipeline
      repository/    -> JPA repositories
      model/         -> Entities: Post, MediaAsset, PublishLog
    scheduler/       -> Scheduled job that checks due posts and publishes
    instagram/       -> Instagram Graph API client wrapper
  src/main/resources/
    application.yml
  src/test/java/...
```

## 5. Database Schema (entities)
- User: id, name, email, created_at
- SocialAccount: id, user_id (FK), platform, ig_business_id, access_token, token_expires_at
- Post: id, user_id (FK), caption, media_url, media_type, status, scheduled_time, created_at, published_at
- PublishLog: id, post_id (FK), attempt_time, status, error_message

## 6. Entity Relationship Diagram (Mermaid)
```
erDiagram
    USER ||--o{ SOCIAL_ACCOUNT : owns
    USER ||--o{ POST : creates
    POST ||--o{ PUBLISH_LOG : has
    SOCIAL_ACCOUNT ||--o{ POST : publishes_to

    USER {
        int id PK
        string name
        string email
        datetime created_at
    }
    SOCIAL_ACCOUNT {
        int id PK
        int user_id FK
        string platform
        string ig_business_id
        string access_token
        datetime token_expires_at
    }
    POST {
        int id PK
        int user_id FK
        int social_account_id FK
        string caption
        string media_url
        string media_type
        string status
        datetime scheduled_time
        datetime created_at
        datetime published_at
    }
    PUBLISH_LOG {
        int id PK
        int post_id FK
        datetime attempt_time
        string status
        string error_message
    }
```

## 7. Class Diagram (Mermaid)
```
classDiagram
    class Post {
        Long id
        String caption
        String mediaUrl
        String mediaType
        PostStatus status
        LocalDateTime scheduledTime
        publish()
    }
    class SocialAccount {
        Long id
        String platform
        String accessToken
        LocalDateTime tokenExpiresAt
        refreshToken()
    }
    class PostService {
        createPost()
        schedulePost()
        publishPost()
    }
    class InstagramClient {
        createMediaContainer()
        publishContainer()
    }
    class SchedulerJob {
        checkDuePosts()
    }
    PostService --> Post
    PostService --> InstagramClient
    SchedulerJob --> PostService
    Post --> SocialAccount
```

## 8. Sequence Diagram - Publishing a Scheduled Post (Mermaid)
```
sequenceDiagram
    participant Scheduler as SchedulerJob
    participant Service as PostService
    participant DB as Database
    participant IG as Instagram Graph API

    Scheduler->>DB: find posts due for publishing
    DB-->>Scheduler: list of due posts
    loop for each due post
        Scheduler->>Service: publishPost(post)
        Service->>IG: create media container (image/caption)
        IG-->>Service: container_id
        Service->>IG: publish container(container_id)
        IG-->>Service: media_id (success) or error
        Service->>DB: update post status + log result
    end
```

## 9. Flow Diagram - Post Creation to Publish (Mermaid)
```
flowchart TD
    A[User creates post via API/UI] --> B{Publish now or schedule?}
    B -->|Now| C[Call InstagramClient immediately]
    B -->|Schedule| D[Save post with scheduled_time, status=SCHEDULED]
    D --> E[SchedulerJob polls DB periodically]
    E --> F{Is post due?}
    F -->|No| E
    F -->|Yes| C
    C --> G[Create media container via Graph API]
    G --> H[Publish container via Graph API]
    H --> I{Success?}
    I -->|Yes| J[Update status=PUBLISHED, save published_at]
    I -->|No| K[Update status=FAILED, log error, optional retry]
```

## 10. Facebook/Instagram App Setup Requirements
- Instagram account must be a Business or Creator account
- Must be linked to a Facebook Page
- Register app in Facebook Developer Console
- Implement OAuth2 login flow for account owner to grant access
- Submit for App Review with: app description, privacy policy URL, screen recording of the permission flow
- Handle access token storage and refresh (tokens expire)

## 11. Suggested Initial GitHub Issues
1. Set up Spring Boot project skeleton with Maven/Gradle
2. Configure PostgreSQL and JPA entities (User, SocialAccount, Post, PublishLog)
3. Implement Facebook OAuth2 login flow
4. Build InstagramClient wrapper (create container, publish container)
5. Build PostService with create/schedule/publish logic
6. Build SchedulerJob to poll and publish due posts
7. Build REST endpoints for post CRUD
8. Register app in Facebook Developer Console
9. Prepare app review materials (privacy policy, screen recording)
10. Write integration tests for publishing pipeline
11. (Stretch) Add basic web UI for connecting account and creating posts

## 12. Future Expansion Ideas (post-MVP)
- Support additional platforms (Twitter/X, TikTok, LinkedIn)
- Add analytics on posting time and engagement
- Add AI-based content generation for captions/images
- Add best-time-to-post recommendations based on engagement data
- Multi-user support for scaling to other people

