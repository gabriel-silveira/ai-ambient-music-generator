# AI Ambient Music Generator - System Architecture

**Version:** 1.0
**Date:** 2025-11-12
**Status:** Draft
**Author:** Architecture Team

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Overview](#system-overview)
3. [Architecture Diagrams](#architecture-diagrams)
4. [Technology Stack](#technology-stack)
5. [Data Architecture](#data-architecture)
6. [API Architecture](#api-architecture)
7. [Security Architecture](#security-architecture)
8. [Storage Architecture](#storage-architecture)
9. [Deployment Architecture](#deployment-architecture)
10. [Architectural Decisions](#architectural-decisions)
11. [Patterns and Best Practices](#patterns-and-best-practices)
12. [Risks and Trade-offs](#risks-and-trade-offs)
13. [Implementation Roadmap](#implementation-roadmap)
14. [Appendix](#appendix)

---

## 1. Executive Summary

### Purpose

This document defines the complete technical architecture for the **AI Ambient Music Generator**, a single-user web application designed to automate the creation and distribution of AI-generated ambient music to YouTube for monetization.

### Scope

The system encompasses 5 major phases:
1. Music & Image Generation (Mubert API + DALL·E 3)
2. Video Rendering (FFmpeg)
3. Dashboard & History Management
4. YouTube Upload (OAuth 2.0 + YouTube Data API v3)
5. Batch Generation (up to 10 tracks)

### Key Constraints

- **Single-user system** (no public registration)
- **Local deployment** (Docker Compose)
- **Synchronous processing** (no background workers/queues)
- **Flat file storage** (no S3/CDN)
- **Budget**: $53/month (development) → $211/month (production)

### Target Users

Single content creator focused on YouTube ambient music monetization.

---

## 2. System Overview

### 2.1. High-Level Architecture

```mermaid
graph TB
    subgraph "User Layer"
        U[User Browser]
    end

    subgraph "Frontend Layer"
        FE[Next.js 16.0.1<br/>shadcn/ui + TailwindCSS]
    end

    subgraph "Backend Layer"
        API[FastAPI<br/>Python 3.11+]
        AUTH[JWT Auth<br/>Middleware]
    end

    subgraph "Data Layer"
        DB[(PostgreSQL 15<br/>Users, Tracks, Tokens)]
        STORAGE[/Flat File Storage<br/>music/ images/ videos/\]
    end

    subgraph "External Services"
        MUBERT[Mubert API 3.0<br/>Music Generation]
        DALLE[OpenAI DALL·E 3<br/>Image Generation]
        YOUTUBE[YouTube Data API v3<br/>Video Upload]
        FFMPEG[FFmpeg<br/>Video Rendering]
    end

    U -->|HTTPS| FE
    FE -->|REST API| API
    API -->|Validate| AUTH
    AUTH -->|Query| DB
    API -->|Read/Write| STORAGE
    API -->|Generate Music| MUBERT
    API -->|Generate Image| DALLE
    API -->|Upload Video| YOUTUBE
    API -->|Render Video| FFMPEG

    style U fill:#e1f5ff
    style FE fill:#fff4e1
    style API fill:#ffe1f5
    style DB fill:#e1ffe1
    style STORAGE fill:#f0f0f0
    style MUBERT fill:#ffd4d4
    style DALLE fill:#ffd4d4
    style YOUTUBE fill:#ffd4d4
    style FFMPEG fill:#d4e5ff
```

### 2.2. System Context

**Before (Current State):**
- No system exists (greenfield project)

**After (Target State - All 5 Phases Complete):**
- Fully automated music generation pipeline
- YouTube upload with OAuth integration
- Dashboard with analytics and history
- Batch processing capabilities

### 2.3. Key Components

| Component | Technology | Responsibility |
|-----------|-----------|----------------|
| **Frontend** | Next.js 16.0.1 | User interface, forms, dashboard |
| **Backend API** | FastAPI | REST endpoints, business logic |
| **Database** | PostgreSQL 15 | Persistent data storage |
| **File Storage** | Filesystem | Generated media files |
| **Auth** | JWT | Session management |
| **Music Gen** | Mubert API | Generate ambient tracks |
| **Image Gen** | DALL·E 3 | Generate cover art |
| **Video Render** | FFmpeg | Combine audio + image |
| **Upload** | YouTube API | Publish to YouTube |

---

## 3. Architecture Diagrams

### 3.1. Component Architecture

```mermaid
C4Context
    title System Context Diagram - AI Ambient Music Generator

    Person(user, "Content Creator", "Single user managing<br/>ambient music content")

    System(app, "AI Ambient Music<br/>Generator", "Web application for<br/>automated music generation")

    System_Ext(mubert, "Mubert API", "AI music generation<br/>service")
    System_Ext(openai, "OpenAI API", "DALL·E 3 image<br/>generation")
    System_Ext(youtube, "YouTube API", "Video upload and<br/>management")

    Rel(user, app, "Uses", "HTTPS")
    Rel(app, mubert, "Generates music", "REST API")
    Rel(app, openai, "Generates images", "REST API")
    Rel(app, youtube, "Uploads videos", "OAuth 2.0 + REST")
```

### 3.2. Data Flow - Track Generation

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant Backend
    participant DB
    participant Storage
    participant Mubert
    participant DALLE
    participant FFmpeg

    User->>Frontend: Enter prompt + duration
    Frontend->>Backend: POST /api/generate

    Backend->>DB: Create track record (status: generating)
    Backend->>Mubert: Generate music track
    Mubert-->>Backend: Audio file URL
    Backend->>Storage: Download & save MP3

    Backend->>DALLE: Generate cover art (1792x1024)
    DALLE-->>Backend: Image URL
    Backend->>Backend: Upscale to 1920x1080 (Pillow)
    Backend->>Storage: Save PNG

    Backend->>DB: Update track (status: generated)
    Backend-->>Frontend: Track data + URLs
    Frontend-->>User: Display track card

    Note over User,FFmpeg: Phase 2: Video Rendering

    User->>Frontend: Click "Render Video"
    Frontend->>Backend: POST /api/render {track_id}
    Backend->>FFmpeg: ffmpeg -i audio.mp3 -i image.png output.mp4
    FFmpeg-->>Backend: Video file
    Backend->>Storage: Save MP4
    Backend->>DB: Update track (status: rendered)
    Backend-->>Frontend: Video URL
    Frontend-->>User: Video preview
```

### 3.3. Authentication Flow

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant Backend
    participant DB

    Note over User,DB: Initial Login

    User->>Frontend: Enter email + password
    Frontend->>Backend: POST /api/auth/login
    Backend->>DB: Query user by email
    DB-->>Backend: User record
    Backend->>Backend: Verify password (bcrypt)
    Backend->>Backend: Generate JWT tokens<br/>(access 15min, refresh 7d)
    Backend-->>Frontend: {access_token, refresh_token}
    Frontend->>Frontend: Store tokens (localStorage)
    Frontend-->>User: Redirect to /dashboard

    Note over User,DB: Subsequent Requests

    User->>Frontend: Navigate to /history
    Frontend->>Backend: GET /api/history<br/>Header: Authorization: Bearer {access_token}
    Backend->>Backend: Validate JWT signature + expiry
    Backend->>DB: Query tracks for user_id
    DB-->>Backend: Track list
    Backend-->>Frontend: Track data
    Frontend-->>User: Display history

    Note over User,DB: Token Refresh (after 15min)

    Frontend->>Backend: POST /api/auth/refresh<br/>Body: {refresh_token}
    Backend->>DB: Validate refresh token
    DB-->>Backend: Token valid
    Backend->>Backend: Generate new access token
    Backend->>DB: Update last_used timestamp
    Backend-->>Frontend: {access_token}
    Frontend->>Frontend: Update stored token
```

### 3.4. YouTube Upload Flow (Phase 4)

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant Backend
    participant DB
    participant Google OAuth
    participant YouTube API

    Note over User,YouTube API: Step 1: Connect YouTube

    User->>Frontend: Click "Connect YouTube"
    Frontend->>Backend: GET /api/auth/youtube
    Backend->>Backend: Generate state (CSRF token)
    Backend->>DB: Store state temporarily
    Backend-->>Frontend: Redirect to Google OAuth
    Frontend->>Google OAuth: Authorization request
    Google OAuth-->>User: Consent screen
    User->>Google OAuth: Grant permission
    Google OAuth-->>Backend: GET /callback?code=XXX&state=YYY
    Backend->>DB: Validate state
    Backend->>Google OAuth: Exchange code for tokens
    Google OAuth-->>Backend: {access_token, refresh_token}
    Backend->>Backend: Encrypt tokens (Fernet)
    Backend->>DB: Store encrypted tokens
    Backend-->>Frontend: Redirect to /dashboard?youtube=connected
    Frontend-->>User: Show "YouTube Connected ✓"

    Note over User,YouTube API: Step 2: Upload Video

    User->>Frontend: Click "Upload to YouTube"
    Frontend->>Frontend: Open upload modal (title, desc, tags)
    User->>Frontend: Fill metadata + click upload
    Frontend->>Backend: POST /api/upload {track_id, metadata}
    Backend->>DB: Get track + YouTube tokens
    Backend->>Backend: Decrypt access_token
    Backend->>Backend: Check token expiry

    alt Token expired
        Backend->>Google OAuth: Refresh token
        Google OAuth-->>Backend: New access_token
        Backend->>DB: Update tokens
    end

    Backend->>YouTube API: Resumable upload (init)
    YouTube API-->>Backend: Upload session URL
    Backend->>YouTube API: Upload video chunks (256KB)
    YouTube API-->>Backend: Video ID
    Backend->>DB: Update track (youtube_url, status: uploaded)
    Backend-->>Frontend: {youtube_url}
    Frontend-->>User: Show YouTube link
```

---

## 4. Technology Stack

### 4.1. Backend Stack

| Component | Technology | Version | Justification |
|-----------|-----------|---------|---------------|
| **Runtime** | Python | 3.11+ | Modern async support, rich ecosystem |
| **Framework** | FastAPI | Latest | Fast, async, auto OpenAPI docs |
| **Database** | PostgreSQL | 15 | ACID compliance, JSON support, reliability |
| **ORM** | SQLAlchemy | 2.x | Industry standard, type-safe |
| **Migrations** | Alembic | Latest | Integrated with SQLAlchemy |
| **Auth** | PyJWT | Latest | JWT token generation/validation |
| **Password** | bcrypt | Latest | Secure password hashing |
| **HTTP Client** | httpx | Latest | Async HTTP for API calls |
| **Image Processing** | Pillow | Latest | Image upscaling (Lanczos) |
| **Video Rendering** | FFmpeg | 6.0+ | Industry standard for video encoding |
| **Environment** | python-dotenv | Latest | Environment variable management |
| **Validation** | Pydantic | 2.x | Data validation, settings management |
| **Encryption** | cryptography | Latest | Fernet encryption for OAuth tokens |

### 4.2. Frontend Stack

| Component | Technology | Version | Justification |
|-----------|-----------|---------|---------------|
| **Framework** | Next.js | 16.0.1 | React framework, SSR/SSG, App Router |
| **Runtime** | Node.js | 20 LTS | Stable LTS version |
| **Language** | TypeScript | 5.x | Type safety, better DX |
| **UI Components** | shadcn/ui | Latest | Accessible, customizable components |
| **Styling** | TailwindCSS | 3.x | Utility-first, fast development |
| **Animations** | Framer Motion | Latest | Smooth animations |
| **HTTP Client** | Axios | Latest | Promise-based HTTP client |
| **Forms** | React Hook Form | Latest | Performant form handling |
| **State** | React Context | Built-in | Sufficient for single-user app |

### 4.3. Infrastructure Stack

| Component | Technology | Version | Justification |
|-----------|-----------|---------|---------------|
| **Containerization** | Docker | 24.x | Consistent environments |
| **Orchestration** | Docker Compose | 2.x | Simple multi-container setup |
| **Web Server** | Uvicorn | Latest | ASGI server for FastAPI |
| **Reverse Proxy** | (None in dev) | N/A | Direct access in local deployment |

### 4.4. External Services

| Service | API | Purpose | Cost (Dev) | Cost (Prod) |
|---------|-----|---------|-----------|-------------|
| **Mubert** | v3.0 | Music generation | $49/mo (Trial) | $199/mo (Startup) |
| **OpenAI** | REST | DALL·E 3 images | ~$4/mo | ~$12/mo |
| **YouTube** | Data API v3 | Video upload | Free* | Free* |

\* YouTube API: 10,000 units/day quota (1 upload ≈ 1,600 units = ~6 uploads/day)

### 4.5. Development Tools

- **Version Control**: Git
- **Code Editor**: VSCode (recommended)
- **API Testing**: Postman / Thunder Client
- **Database Client**: pgAdmin / DBeaver
- **Package Manager (Python)**: pip + venv
- **Package Manager (Node)**: npm / pnpm

---

## 5. Data Architecture

### 5.1. Entity Relationship Diagram

```mermaid
erDiagram
    USERS ||--o{ TRACKS : creates
    USERS ||--o| YOUTUBE_TOKENS : has
    USERS ||--o{ BATCHES : creates
    USERS ||--o{ REFRESH_TOKENS : has

    TRACKS ||--o| BATCH_ITEMS : "belongs to"
    BATCHES ||--|{ BATCH_ITEMS : contains

    USERS {
        uuid id PK
        varchar email UK
        varchar password_hash
        timestamp created_at
        timestamp updated_at
    }

    TRACKS {
        uuid id PK
        uuid user_id FK
        text prompt
        varchar music_path
        varchar image_path
        varchar video_path
        varchar youtube_url
        int duration
        varchar status
        timestamp created_at
    }

    YOUTUBE_TOKENS {
        uuid id PK
        uuid user_id FK "UK"
        varchar access_token "encrypted"
        varchar refresh_token "encrypted"
        timestamp expires_at
        timestamp created_at
        timestamp updated_at
    }

    REFRESH_TOKENS {
        uuid id PK
        uuid user_id FK
        varchar token UK
        timestamp expires_at
        timestamp last_used
        timestamp created_at
    }

    BATCHES {
        uuid id PK
        uuid user_id FK
        int total
        int successful
        int failed
        varchar status
        int duration_seconds
        timestamp created_at
        timestamp completed_at
    }

    BATCH_ITEMS {
        uuid id PK
        uuid batch_id FK
        uuid track_id FK "nullable"
        text prompt
        varchar status
        text error_message "nullable"
        int position
        timestamp created_at
        timestamp completed_at
    }
```

### 5.2. Database Schema Details

#### **users**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
```

**Constraints:**
- Email must be unique and valid format
- Password hashed with bcrypt (cost factor 12)

---

#### **tracks**
```sql
CREATE TABLE tracks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    prompt TEXT NOT NULL CHECK (length(prompt) >= 10 AND length(prompt) <= 500),
    music_path VARCHAR(500),
    image_path VARCHAR(500),
    video_path VARCHAR(500),
    youtube_url VARCHAR(500),
    duration INT NOT NULL CHECK (duration >= 120 AND duration <= 900),
    status VARCHAR(20) NOT NULL DEFAULT 'generating',
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_tracks_user_id ON tracks(user_id);
CREATE INDEX idx_tracks_status ON tracks(status);
CREATE INDEX idx_tracks_created_at ON tracks(created_at DESC);
```

**Status values:**
- `generating` - Music/image generation in progress
- `generated` - Phase 1 complete (music + image ready)
- `rendering` - Video rendering in progress
- `rendered` - Phase 2 complete (video ready)
- `uploading` - YouTube upload in progress
- `uploaded` - Phase 4 complete (live on YouTube)
- `failed` - Error occurred

---

#### **youtube_tokens**
```sql
CREATE TABLE youtube_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
    access_token VARCHAR(1000) NOT NULL,  -- Encrypted with Fernet
    refresh_token VARCHAR(1000) NOT NULL, -- Encrypted with Fernet
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_youtube_tokens_user_id ON youtube_tokens(user_id);
```

**Security:**
- Tokens encrypted using Fernet (symmetric encryption)
- Encryption key stored in environment variable `ENCRYPTION_KEY`

---

#### **refresh_tokens**
```sql
CREATE TABLE refresh_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token VARCHAR(500) UNIQUE NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    last_used TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_refresh_tokens_token ON refresh_tokens(token);
CREATE INDEX idx_refresh_tokens_user_id ON refresh_tokens(user_id);
```

**Lifecycle:**
- Issued on login (7-day expiry)
- Used to obtain new access tokens
- Rotated on refresh (old token invalidated)

---

#### **batches**
```sql
CREATE TABLE batches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    total INT NOT NULL CHECK (total > 0 AND total <= 10),
    successful INT DEFAULT 0,
    failed INT DEFAULT 0,
    status VARCHAR(20) NOT NULL DEFAULT 'processing',
    duration_seconds INT NOT NULL CHECK (duration_seconds >= 120 AND duration_seconds <= 900),
    created_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP
);

CREATE INDEX idx_batches_user_id ON batches(user_id);
CREATE INDEX idx_batches_created_at ON batches(created_at DESC);
```

**Status values:**
- `processing` - Batch generation in progress
- `completed` - All items processed (success or failure)

---

#### **batch_items**
```sql
CREATE TABLE batch_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    batch_id UUID NOT NULL REFERENCES batches(id) ON DELETE CASCADE,
    track_id UUID REFERENCES tracks(id) ON DELETE SET NULL,
    prompt TEXT NOT NULL,
    status VARCHAR(20) NOT NULL,
    error_message TEXT,
    position INT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP
);

CREATE INDEX idx_batch_items_batch_id ON batch_items(batch_id);
CREATE INDEX idx_batch_items_position ON batch_items(batch_id, position);
```

**Status values:**
- `pending` - Waiting to be processed
- `generating` - Currently processing
- `success` - Track created successfully
- `failed` - Error occurred during generation

---

### 5.3. Data Lifecycle

```mermaid
stateDiagram-v2
    [*] --> generating: User submits prompt

    generating --> generated: Music + Image ready
    generating --> failed: API error

    generated --> rendering: User clicks "Render Video"

    rendering --> rendered: Video ready
    rendering --> failed: FFmpeg error

    rendered --> uploading: User clicks "Upload to YouTube"

    uploading --> uploaded: YouTube upload successful
    uploading --> failed: Upload error / Quota exceeded

    failed --> [*]: User deletes track
    uploaded --> [*]: User deletes track
```

---

## 6. API Architecture

### 6.1. API Design Principles

- **RESTful**: Standard HTTP methods (GET, POST, DELETE)
- **JSON**: All requests/responses use JSON
- **Stateless**: JWT-based authentication (no server-side sessions)
- **Versioned**: `/api/` prefix (future: `/api/v2/`)
- **Documented**: Auto-generated OpenAPI docs at `/docs`

### 6.2. Authentication Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/login` | None | Login with email/password |
| POST | `/api/auth/refresh` | None | Refresh access token |
| POST | `/api/auth/logout` | JWT | Invalidate refresh token |

**Request/Response Examples:**

```json
// POST /api/auth/login
{
  "email": "user@example.com",
  "password": "securePassword123"
}

// Response 200
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 900
}
```

---

### 6.3. Track Management Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/generate` | JWT | Generate music + image |
| POST | `/api/render` | JWT | Render video from track |
| GET | `/api/history` | JWT | List all tracks (paginated) |
| GET | `/api/tracks/{id}` | JWT | Get track details |
| DELETE | `/api/tracks/{id}` | JWT | Delete track + files |
| GET | `/api/stats` | JWT | Get usage statistics |

**Request/Response Examples:**

```json
// POST /api/generate
{
  "prompt": "Deep sleep ocean ambient with ethereal pads",
  "duration": 300
}

// Response 200
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "prompt": "Deep sleep ocean ambient with ethereal pads",
  "music_url": "http://localhost:8000/storage/music/550e8400-e29b-41d4-a716-446655440000.mp3",
  "image_url": "http://localhost:8000/storage/images/550e8400-e29b-41d4-a716-446655440000.png",
  "duration": 300,
  "status": "generated",
  "created_at": "2025-11-12T10:30:00Z"
}
```

---

### 6.4. YouTube Integration Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/auth/youtube` | JWT | Initiate YouTube OAuth |
| GET | `/api/auth/youtube/callback` | None | OAuth callback handler |
| GET | `/api/auth/youtube/status` | JWT | Check connection status |
| DELETE | `/api/auth/youtube` | JWT | Disconnect YouTube |
| POST | `/api/upload` | JWT | Upload video to YouTube |

---

### 6.5. Batch Generation Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/batch/generate` | JWT | Generate multiple tracks |
| GET | `/api/batch/{id}` | JWT | Get batch status (polling) |

---

### 6.6. Error Responses

Standard error format:

```json
{
  "detail": "Human-readable error message",
  "error_code": "VALIDATION_ERROR",
  "status_code": 400
}
```

**HTTP Status Codes:**
- `200` - Success
- `201` - Created
- `400` - Bad Request (validation error)
- `401` - Unauthorized (invalid/expired token)
- `403` - Forbidden (CSRF, quota exceeded)
- `404` - Not Found
- `429` - Too Many Requests (rate limit)
- `500` - Internal Server Error

---

## 7. Security Architecture

### 7.1. Authentication & Authorization

**JWT Token Structure:**

```
Header:
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload (Access Token):
{
  "sub": "user_id",
  "email": "user@example.com",
  "exp": 1699876543,  // 15 minutes from issue
  "iat": 1699875643,
  "type": "access"
}

Payload (Refresh Token):
{
  "sub": "user_id",
  "exp": 1700480443,  // 7 days from issue
  "iat": 1699875643,
  "type": "refresh"
}
```

**Token Lifecycle:**

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Backend

    User->>Frontend: Login
    Frontend->>Backend: POST /api/auth/login
    Backend-->>Frontend: access_token (15min) + refresh_token (7d)

    loop Every API call
        Frontend->>Backend: Request + Bearer {access_token}
        Backend->>Backend: Validate JWT
        alt Token valid
            Backend-->>Frontend: Response
        else Token expired (after 15min)
            Backend-->>Frontend: 401 Unauthorized
            Frontend->>Backend: POST /api/auth/refresh {refresh_token}
            Backend-->>Frontend: New access_token
            Frontend->>Backend: Retry original request
        end
    end
```

---

### 7.2. Password Security

- **Hashing**: bcrypt with cost factor 12
- **Minimum length**: 8 characters
- **Validation**: Must contain uppercase, lowercase, number, special char (frontend only)
- **No password reset flow** (single-user, manual DB update if needed)

**Example:**
```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Hash password
hashed = pwd_context.hash("mySecurePassword123")

# Verify password
is_valid = pwd_context.verify("mySecurePassword123", hashed)
```

---

### 7.3. OAuth Token Encryption

YouTube OAuth tokens stored encrypted using **Fernet (symmetric encryption)**.

**Encryption Flow:**

```python
from cryptography.fernet import Fernet

# Generate key (once, stored in .env)
key = Fernet.generate_key()  # ENCRYPTION_KEY=xxx

# Initialize cipher
cipher = Fernet(key)

# Encrypt
encrypted = cipher.encrypt(b"access_token_value")

# Decrypt
decrypted = cipher.decrypt(encrypted)
```

**Environment Variable:**
```bash
ENCRYPTION_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=
```

---

### 7.4. CSRF Protection (OAuth)

**State Parameter in YouTube OAuth:**

```python
import secrets

# Generate random state
state = secrets.token_urlsafe(32)

# Store temporarily (Redis or DB)
cache.set(f"oauth_state:{user_id}", state, expire=300)  # 5 min

# Validate in callback
saved_state = cache.get(f"oauth_state:{user_id}")
if state != saved_state:
    raise HTTPException(403, "Invalid state parameter")
```

---

### 7.5. Security Checklist

- [x] Passwords hashed with bcrypt (cost 12)
- [x] JWT tokens with short expiry (15min access, 7d refresh)
- [x] OAuth tokens encrypted (Fernet)
- [x] CSRF protection (state parameter)
- [x] HTTPS in production (requirement documented)
- [x] No sensitive data in logs
- [x] SQL injection prevention (SQLAlchemy parameterized queries)
- [x] Input validation (Pydantic models)
- [ ] Rate limiting (future enhancement)
- [ ] CORS configuration (frontend domain whitelist)

---

## 8. Storage Architecture

### 8.1. File System Structure

```
ai-ambient-music-generator/
├── storage/
│   ├── music/
│   │   └── {uuid}.mp3          # Generated music files
│   ├── images/
│   │   └── {uuid}.png          # Generated cover art (1920x1080)
│   └── videos/
│       └── {uuid}.mp4          # Rendered videos (H.264, 1080p)
```

### 8.2. File Naming Convention

All files use **UUID v4** from database `track.id`:

```
Track ID: 550e8400-e29b-41d4-a716-446655440000

Files:
- storage/music/550e8400-e29b-41d4-a716-446655440000.mp3
- storage/images/550e8400-e29b-41d4-a716-446655440000.png
- storage/videos/550e8400-e29b-41d4-a716-446655440000.mp4
```

**Benefits:**
- No name collisions
- Direct lookup from DB
- No encoding issues (pure ASCII)
- Easy cleanup (delete by UUID)

---

### 8.3. File Specifications

#### **Music Files (MP3)**
- **Format**: MP3
- **Bitrate**: 320 kbps (constant)
- **Sample Rate**: 44.1 kHz
- **Channels**: Stereo
- **Duration**: 120-900 seconds
- **Avg Size**: ~7-8 MB per 5 minutes

#### **Image Files (PNG)**
- **Format**: PNG (lossless)
- **Resolution**: 1920x1080 (Full HD)
- **Source**: DALL·E 3 (1792x1024) → Pillow upscale (LANCZOS)
- **Color**: RGB
- **Avg Size**: ~2-3 MB

#### **Video Files (MP4)**
- **Format**: MP4 (H.264 + AAC)
- **Resolution**: 1920x1080 @ 30fps
- **Video Codec**: libx264 (preset: medium, CRF: 23)
- **Audio Codec**: AAC (320k)
- **Avg Size**: ~10-15 MB per 5 minutes

---

### 8.4. Storage Capacity Planning

**Estimates per track (5 minutes):**
- Music: 8 MB
- Image: 3 MB
- Video: 12 MB
- **Total**: ~23 MB per track

**Monthly estimates:**
- **Development (Trial - 500 tracks/mo max)**: 500 × 23 MB = ~11.5 GB
- **Production (166 tracks/mo sustainable)**: 166 × 23 MB = ~3.8 GB

**Disk requirements:**
- **Minimum**: 50 GB (room for ~2,000 tracks)
- **Recommended**: 100 GB (room for ~4,000 tracks)

---

### 8.5. Storage Operations

#### **Save File**
```python
import os
import uuid

def save_track_file(track_id: uuid.UUID, file_content: bytes, file_type: str) -> str:
    """
    Save file to storage and return relative path.

    Args:
        track_id: UUID of the track
        file_content: Binary file content
        file_type: 'music', 'image', or 'video'

    Returns:
        Relative path (e.g., 'storage/music/xxx.mp3')
    """
    extensions = {
        'music': 'mp3',
        'image': 'png',
        'video': 'mp4'
    }

    directory = f"storage/{file_type}"
    os.makedirs(directory, exist_ok=True)

    filename = f"{track_id}.{extensions[file_type]}"
    filepath = os.path.join(directory, filename)

    with open(filepath, 'wb') as f:
        f.write(file_content)

    return filepath
```

#### **Delete Files**
```python
def delete_track_files(track: Track) -> list[str]:
    """
    Delete all files associated with a track.

    Returns:
        List of deleted file types
    """
    deleted = []

    for file_type, path in [
        ('music', track.music_path),
        ('image', track.image_path),
        ('video', track.video_path)
    ]:
        if path and os.path.exists(path):
            os.remove(path)
            deleted.append(file_type)
            logger.info(f"Deleted {file_type}: {path}")

    return deleted
```

---

### 8.6. Backup Strategy

**Documented approach (not automated initially):**

```bash
# Manual backup script (backup.sh)

#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)

# 1. Backup storage files
tar -czf backup_storage_$DATE.tar.gz storage/

# 2. Backup PostgreSQL database
docker compose exec -T postgres pg_dump -U postgres ai_ambient_music > backup_db_$DATE.sql

# 3. Move to backup location
mv backup_*.{tar.gz,sql} /mnt/backups/

echo "Backup completed: $DATE"
```

**Recommended schedule:**
- Daily backups (automated via cron in production)
- Keep 7 daily backups
- Keep 4 weekly backups
- Keep 3 monthly backups

**Restoration:**
```bash
# Restore storage
tar -xzf backup_storage_20251112.tar.gz

# Restore database
docker compose exec -T postgres psql -U postgres ai_ambient_music < backup_db_20251112.sql
```

---

## 9. Deployment Architecture

### 9.1. Docker Compose Structure

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: ai-music-db
    environment:
      POSTGRES_DB: ai_ambient_music
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: ai-music-backend
    environment:
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@postgres:5432/ai_ambient_music
      JWT_SECRET: ${JWT_SECRET}
      ENCRYPTION_KEY: ${ENCRYPTION_KEY}
      MUBERT_API_KEY: ${MUBERT_API_KEY}
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      YOUTUBE_CLIENT_ID: ${YOUTUBE_CLIENT_ID}
      YOUTUBE_CLIENT_SECRET: ${YOUTUBE_CLIENT_SECRET}
    volumes:
      - ./storage:/app/storage
      - ./backend:/app
    ports:
      - "8000:8000"
    depends_on:
      postgres:
        condition: service_healthy
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: ai-music-frontend
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:8000
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    depends_on:
      - backend
    command: npm run dev

volumes:
  postgres_data:
```

### 9.2. Backend Dockerfile

```dockerfile
FROM python:3.11-slim

# Install FFmpeg
RUN apt-get update && apt-get install -y \
    ffmpeg \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create storage directories
RUN mkdir -p storage/music storage/images storage/videos

# Expose port
EXPOSE 8000

# Run migrations on startup
CMD alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8000
```

### 9.3. Frontend Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Start development server
CMD ["npm", "run", "dev"]
```

### 9.4. Environment Variables

**.env.example**
```bash
# Database
DB_PASSWORD=your_secure_password

# JWT Authentication
JWT_SECRET=your_jwt_secret_key_min_32_chars
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=15
JWT_REFRESH_TOKEN_EXPIRE_DAYS=7

# Encryption (for OAuth tokens)
# Generate: python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
ENCRYPTION_KEY=your_fernet_encryption_key

# Mubert API
MUBERT_API_KEY=your_mubert_api_key

# OpenAI API
OPENAI_API_KEY=sk-your-openai-api-key

# YouTube OAuth
YOUTUBE_CLIENT_ID=your-client-id.apps.googleusercontent.com
YOUTUBE_CLIENT_SECRET=your-client-secret
YOUTUBE_REDIRECT_URI=http://localhost:8000/api/auth/youtube/callback

# Frontend
NEXT_PUBLIC_API_URL=http://localhost:8000
```

### 9.5. Deployment Steps

**Development (Local):**

```bash
# 1. Clone repository
git clone <repo-url>
cd ai-ambient-music-generator

# 2. Create .env file
cp .env.example .env
# Edit .env with your API keys

# 3. Start services
docker compose up -d

# 4. Run migrations
docker compose exec backend alembic upgrade head

# 5. Create seed user (optional)
docker compose exec backend python scripts/create_user.py

# 6. Access application
# Frontend: http://localhost:3000
# Backend API: http://localhost:8000
# API Docs: http://localhost:8000/docs
```

**Production (AWS EC2 - Future):**

```bash
# 1. Provision EC2 instance (t3.medium or larger)
# 2. Install Docker + Docker Compose
# 3. Clone repository
# 4. Configure production .env (with HTTPS URLs)
# 5. Set up Nginx reverse proxy (SSL termination)
# 6. Configure firewall (allow 80, 443)
# 7. Start services with docker compose -f docker-compose.prod.yml up -d
# 8. Set up automated backups (cron)
```

---

## 10. Architectural Decisions

### ADR-001: Monorepo Structure

**Status:** Accepted

**Context:**
- Single-user application
- Tightly coupled frontend/backend
- Local development focus

**Decision:**
Use monorepo with `backend/` and `frontend/` directories.

**Consequences:**
- ✅ Easier local development
- ✅ Shared Docker Compose
- ✅ Simplified deployment
- ❌ Larger repository size

---

### ADR-002: Synchronous Processing (No Background Workers)

**Status:** Accepted

**Context:**
- Single-user system
- Max 10 tracks per batch (~15 min total)
- No concurrent users

**Decision:**
Process generation requests synchronously (no Celery/RQ).

**Consequences:**
- ✅ Simpler architecture (no Redis, no workers)
- ✅ Fewer moving parts
- ✅ Sufficient for single-user
- ❌ Frontend blocked during processing
- ❌ Cannot scale to multi-user

**Mitigation:**
- Use polling for progress updates
- Clear UI feedback during processing

---

### ADR-003: Flat File Storage (No S3)

**Status:** Accepted

**Context:**
- Single-user local deployment
- Estimated ~3.8 GB/month (sustainable usage)
- No need for CDN

**Decision:**
Store files on local filesystem (`storage/` directory).

**Consequences:**
- ✅ No cloud storage costs
- ✅ Faster access (no network latency)
- ✅ Simpler backup (tar + pg_dump)
- ❌ Not suitable for multi-user/distributed deployment
- ❌ Manual backup required

---

### ADR-004: JWT Authentication (No Sessions)

**Status:** Accepted

**Context:**
- RESTful API design
- Stateless backend preferred
- Single-user (no complex session management needed)

**Decision:**
Use JWT tokens (access 15min, refresh 7 days).

**Consequences:**
- ✅ Stateless backend (horizontal scaling possible)
- ✅ Standard approach
- ✅ Built-in expiration
- ❌ Cannot revoke access token before expiry
- ❌ Slightly larger payload than session cookie

**Mitigation:**
- Short access token expiry (15 min)
- Refresh token rotation on use

---

### ADR-005: PostgreSQL Over SQLite

**Status:** Accepted

**Context:**
- Need for concurrent reads/writes during batch processing
- Future-proofing for analytics queries
- Production-ready database

**Decision:**
Use PostgreSQL 15 instead of SQLite.

**Consequences:**
- ✅ Better concurrency support
- ✅ JSON column support (future flexibility)
- ✅ Production-ready
- ✅ Easy to migrate to managed DB (RDS)
- ❌ Slightly more complex setup (Docker required)

---

### ADR-006: Next.js App Router Over Pages Router

**Status:** Accepted

**Context:**
- Next.js 16.0.1 includes stable App Router
- Modern React patterns (Server Components)
- Future-proof

**Decision:**
Use App Router (`app/` directory) instead of Pages Router.

**Consequences:**
- ✅ Modern architecture
- ✅ Better performance (RSC)
- ✅ Simplified routing
- ❌ Steeper learning curve

---

### ADR-007: Single-User Hardcoded Authentication

**Status:** Accepted

**Context:**
- Application designed for single content creator
- No need for public registration
- Simplified security model

**Decision:**
Create single user via seed script/migration. No registration endpoint.

**Consequences:**
- ✅ Simple, no registration UI/logic
- ✅ No spam/abuse concerns
- ✅ Reduced attack surface
- ❌ Password change requires DB access
- ❌ Cannot add users without code change

**Mitigation:**
- Document password change procedure
- Provide CLI script for password reset

---

### ADR-008: Mubert Trial for Development

**Status:** Accepted

**Context:**
- Trial plan costs $49/month (vs $199 Startup)
- Trial has 500 generations/month (vs 5000)
- **Trial cannot be monetized on YouTube**

**Decision:**
Use Trial plan during development/testing. Upgrade to Startup before production.

**Consequences:**
- ✅ Lower development costs
- ✅ Sufficient for testing (500 tracks)
- ❌ **BLOCKER: Cannot monetize content generated with Trial**
- ❌ Must upgrade before going live

**Critical Note:**
This is a **temporary decision**. Production deployment requires Startup plan ($199/mo).

---

## 11. Patterns and Best Practices

### 11.1. Backend Patterns

#### **Project Structure**
```
backend/
├── app/
│   ├── main.py                 # FastAPI app entry point
│   ├── config.py               # Settings (Pydantic BaseSettings)
│   ├── database.py             # SQLAlchemy setup
│   │
│   ├── models/                 # SQLAlchemy models
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── track.py
│   │   ├── youtube_token.py
│   │   ├── refresh_token.py
│   │   ├── batch.py
│   │   └── batch_item.py
│   │
│   ├── schemas/                # Pydantic schemas (request/response)
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── track.py
│   │   ├── batch.py
│   │   └── upload.py
│   │
│   ├── routers/                # API route handlers
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── generate.py
│   │   ├── render.py
│   │   ├── tracks.py
│   │   ├── stats.py
│   │   ├── youtube.py
│   │   └── batch.py
│   │
│   ├── services/               # Business logic
│   │   ├── __init__.py
│   │   ├── mubert_service.py
│   │   ├── openai_service.py
│   │   ├── ffmpeg_service.py
│   │   ├── youtube_service.py
│   │   ├── storage_service.py
│   │   └── batch_service.py
│   │
│   ├── middleware/             # Custom middleware
│   │   ├── __init__.py
│   │   └── auth_middleware.py
│   │
│   └── utils/                  # Utilities
│       ├── __init__.py
│       ├── jwt_handler.py
│       ├── password.py
│       └── encryption.py
│
├── migrations/                 # Alembic migrations
│   └── versions/
├── tests/                      # Unit + integration tests
│   ├── unit/
│   └── integration/
├── scripts/                    # CLI scripts
│   └── create_user.py
├── requirements.txt
├── Dockerfile
└── alembic.ini
```

#### **Dependency Injection Pattern**
```python
from fastapi import Depends
from sqlalchemy.orm import Session

# Database session dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Current user dependency
async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    user = db.query(User).filter(User.id == payload["sub"]).first()
    if not user:
        raise HTTPException(401, "Invalid credentials")
    return user

# Usage in routes
@router.get("/api/tracks")
async def get_tracks(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    return db.query(Track).filter(Track.user_id == current_user.id).all()
```

#### **Service Layer Pattern**
```python
# services/mubert_service.py
class MubertService:
    def __init__(self):
        self.api_key = settings.MUBERT_API_KEY
        self.base_url = "https://api-b2b.mubert.com/v2"

    async def generate_track(
        self,
        prompt: str,
        duration: int
    ) -> dict:
        """Generate music track using Mubert API"""
        # API call logic
        pass

# Usage in routes
@router.post("/api/generate")
async def generate_track(request: GenerateRequest):
    service = MubertService()
    result = await service.generate_track(request.prompt, request.duration)
    return result
```

---

### 11.2. Frontend Patterns

#### **Project Structure**
```
frontend/
├── app/
│   ├── (auth)/                 # Auth layout group
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── layout.tsx
│   │
│   ├── (protected)/            # Protected layout group
│   │   ├── dashboard/
│   │   │   └── page.tsx
│   │   ├── generate/
│   │   │   └── page.tsx
│   │   ├── history/
│   │   │   └── page.tsx
│   │   ├── batch/
│   │   │   └── page.tsx
│   │   └── layout.tsx
│   │
│   ├── layout.tsx              # Root layout
│   └── page.tsx                # Landing page
│
├── components/
│   ├── ui/                     # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── dialog.tsx
│   │   └── ...
│   │
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   └── ProtectedRoute.tsx
│   │
│   ├── tracks/
│   │   ├── TrackCard.tsx
│   │   ├── TrackList.tsx
│   │   └── TrackDetailsModal.tsx
│   │
│   ├── batch/
│   │   ├── BatchForm.tsx
│   │   ├── BatchProgress.tsx
│   │   └── BatchResults.tsx
│   │
│   └── youtube/
│       ├── YouTubeConnectButton.tsx
│       └── UploadToYouTubeModal.tsx
│
├── hooks/
│   ├── useAuth.ts
│   ├── useGenerate.ts
│   ├── useHistory.ts
│   └── useYouTube.ts
│
├── lib/
│   ├── api.ts                  # Axios instance
│   ├── auth.ts                 # Auth helpers
│   └── utils.ts
│
├── types/
│   └── index.ts
│
├── public/
├── package.json
├── tsconfig.json
├── tailwind.config.ts
└── next.config.js
```

#### **Custom Hook Pattern**
```typescript
// hooks/useGenerate.ts
import { useState } from 'react';
import { api } from '@/lib/api';

export function useGenerate() {
  const [isGenerating, setIsGenerating] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const generate = async (prompt: string, duration: number) => {
    setIsGenerating(true);
    setError(null);

    try {
      const response = await api.post('/api/generate', { prompt, duration });
      return response.data;
    } catch (err: any) {
      setError(err.response?.data?.detail || 'Generation failed');
      throw err;
    } finally {
      setIsGenerating(false);
    }
  };

  return { generate, isGenerating, error };
}
```

#### **API Client Pattern**
```typescript
// lib/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  timeout: 900000, // 15 minutes (for batch generation)
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor (add auth token)
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('access_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor (handle token refresh)
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Token expired, try refresh
      const refreshToken = localStorage.getItem('refresh_token');
      if (refreshToken) {
        try {
          const response = await axios.post(
            `${process.env.NEXT_PUBLIC_API_URL}/api/auth/refresh`,
            { refresh_token: refreshToken }
          );

          localStorage.setItem('access_token', response.data.access_token);

          // Retry original request
          error.config.headers.Authorization = `Bearer ${response.data.access_token}`;
          return axios(error.config);
        } catch (refreshError) {
          // Refresh failed, redirect to login
          localStorage.clear();
          window.location.href = '/login';
        }
      }
    }
    return Promise.reject(error);
  }
);

export { api };
```

---

### 11.3. Testing Patterns

#### **Backend Unit Test Example**
```python
# tests/unit/test_mubert_service.py
import pytest
from unittest.mock import patch, MagicMock
from app.services.mubert_service import MubertService

@pytest.fixture
def mubert_service():
    return MubertService()

@patch('httpx.AsyncClient.post')
async def test_generate_track_success(mock_post, mubert_service):
    # Arrange
    mock_response = MagicMock()
    mock_response.json.return_value = {
        "data": {
            "tasks": [{
                "download_link": "https://example.com/track.mp3"
            }]
        }
    }
    mock_post.return_value = mock_response

    # Act
    result = await mubert_service.generate_track(
        prompt="Deep sleep ambient",
        duration=300
    )

    # Assert
    assert result["download_url"] == "https://example.com/track.mp3"
    mock_post.assert_called_once()
```

#### **Backend Integration Test Example**
```python
# tests/integration/test_generate_endpoint.py
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_generate_track_unauthorized():
    # No auth token
    response = client.post("/api/generate", json={
        "prompt": "Test prompt",
        "duration": 300
    })
    assert response.status_code == 401

def test_generate_track_success(auth_headers):
    response = client.post(
        "/api/generate",
        json={
            "prompt": "Deep sleep ambient with ocean sounds",
            "duration": 300
        },
        headers=auth_headers
    )

    assert response.status_code == 200
    data = response.json()
    assert "id" in data
    assert data["status"] in ["generating", "generated"]
```

---

### 11.4. Code Quality Standards

- **Linting**:
  - Backend: `flake8`, `black`, `isort`
  - Frontend: `eslint`, `prettier`
- **Type Checking**:
  - Backend: `mypy`
  - Frontend: TypeScript strict mode
- **Test Coverage**: Minimum 70%
- **Documentation**: Docstrings for all public functions
- **Commit Messages**: Conventional Commits format

---

## 12. Risks and Trade-offs

### 12.1. Technical Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| **YouTube API Quota Exceeded** | Medium | High | Monitor usage, implement daily limit warning, handle 429 errors gracefully |
| **Mubert API Downtime** | Low | High | Implement retry logic, queue requests, show clear error messages |
| **FFmpeg Rendering Timeout** | Medium | Medium | 300s timeout, optimize preset ("medium"), provide progress feedback |
| **Disk Space Full** | Medium | High | Monitor storage in dashboard, implement cleanup warnings, document backup strategy |
| **OAuth Token Theft** | Low | High | Encrypt tokens (Fernet), HTTPS required, short expiry |
| **PostgreSQL Corruption** | Low | Critical | Regular backups (documented), Docker volume persistence |

---

### 12.2. Business Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| **YouTube Verification Delay (2-4 weeks)** | High | Start verification process immediately, test with private/unlisted videos during wait |
| **Mubert API Policy Change** | High | Monitor Mubert terms, have backup plan (Soundful, AIVA) |
| **DALL·E Cost Increase** | Medium | Monitor OpenAI pricing, optimize generation (single image per track) |
| **YouTube Monetization Denial** | Critical | Follow YouTube Partner Program policies, ensure content quality |

---

### 12.3. Architectural Trade-offs

#### **Single-User vs Multi-User**

**Decision:** Single-user

**Trade-offs:**
- ✅ Simpler architecture (no user management, quotas, billing)
- ✅ Lower costs (single API keys, no scaling)
- ✅ Faster development
- ❌ Cannot monetize as SaaS
- ❌ Limited market (only self-use)

**Future Path:** If multi-user needed, requires:
- User registration system
- Per-user quotas
- Background job queue (Celery)
- Cloud storage (S3)
- Billing integration (Stripe)

---

#### **Synchronous vs Asynchronous Processing**

**Decision:** Synchronous (blocking)

**Trade-offs:**
- ✅ Simpler (no Redis, Celery, workers)
- ✅ Fewer failure modes
- ✅ Sufficient for single-user (~15 min max)
- ❌ Frontend blocked during generation
- ❌ Cannot handle concurrent requests

**Mitigation:**
- Polling for progress updates (batch generation)
- Clear UI feedback (loading states, ETAs)

---

#### **Flat Storage vs S3**

**Decision:** Flat file storage

**Trade-offs:**
- ✅ No cloud costs
- ✅ Faster access (local disk)
- ✅ Simpler backup (tar + pg_dump)
- ❌ Not suitable for distributed deployment
- ❌ Manual backup required
- ❌ No CDN (slower for remote access)

**Future Path:** Migrate to S3 + CloudFront if:
- Multi-user deployment needed
- Remote access required
- CDN performance desired

---

## 13. Implementation Roadmap

### 13.1. Phase Overview

```mermaid
gantt
    title Implementation Roadmap (27 Story Points)
    dateFormat  YYYY-MM-DD
    section Phase 1
    Music + Image Generation    :p1, 2025-11-13, 40h
    section Phase 2
    Video Rendering            :p2, after p1, 25h
    section Phase 3
    Dashboard + History        :p3, after p2, 15h
    section Phase 4
    YouTube Upload             :p4, after p3, 40h
    section Phase 5
    Batch Generation           :p5, after p4, 15h
```

**Total Estimated Time:** ~135 hours (~17 working days at 8h/day)

---

### 13.2. Phase 1: Music & Image Generation (8 pts, ~40h)

**Goal:** Generate music (Mubert) + cover art (DALL·E) from prompt.

**Deliverables:**
- Backend:
  - [x] PostgreSQL setup + Docker Compose
  - [x] User model + JWT auth
  - [x] Track model
  - [x] `POST /api/auth/login`
  - [x] `POST /api/auth/refresh`
  - [x] `POST /api/generate` (Mubert + DALL·E + Pillow upscale)
  - [x] Storage service (save MP3 + PNG)
  - [x] Migrations (Alembic)

- Frontend:
  - [x] Login page
  - [x] Dashboard layout
  - [x] Generate form (prompt + duration)
  - [x] Track preview card (audio player + image)

**Acceptance Criteria:**
- User can login with email/password
- User can submit prompt + duration (120-900s)
- System generates music (MP3, 320kbps, 44.1kHz)
- System generates cover art (PNG, 1920x1080)
- Track appears in dashboard with audio player

---

### 13.3. Phase 2: Video Rendering (5 pts, ~25h)

**Goal:** Render static video (image + audio) using FFmpeg.

**Deliverables:**
- Backend:
  - [x] `POST /api/render` endpoint
  - [x] FFmpegService (H.264, 1080p, 30fps)
  - [x] Update track status (rendered)

- Frontend:
  - [x] "Render Video" button on track card
  - [x] Video preview (HTML5 video player)
  - [x] Download video button

**Acceptance Criteria:**
- User can render video from generated track
- Video is H.264, 1920x1080, 30fps, MP4
- Video includes 2s fade-in/fade-out
- Video playable in browser
- Rendering completes in < 300s

---

### 13.4. Phase 3: Dashboard & History (3 pts, ~15h)

**Goal:** Track management and usage statistics.

**Deliverables:**
- Backend:
  - [x] `GET /api/history` (paginated, filterable)
  - [x] `GET /api/tracks/{id}`
  - [x] `DELETE /api/tracks/{id}` (with file cleanup)
  - [x] `GET /api/stats` (total tracks, storage, monthly usage)

- Frontend:
  - [x] `/history` page (track list with filters)
  - [x] Track details modal
  - [x] Delete confirmation dialog
  - [x] Stats cards (total tracks, storage, monthly Mubert usage)
  - [x] Dashboard updated with recent tracks

**Acceptance Criteria:**
- User can view all tracks (paginated, 50 per page)
- User can filter by status (all, generated, rendered, uploaded)
- User can delete track (removes DB + files)
- Dashboard shows stats (total tracks, storage used, Mubert usage %)
- Alert shown when Mubert usage > 80%

---

### 13.5. Phase 4: YouTube Upload (8 pts, ~40h)

**Goal:** OAuth integration + video upload to YouTube.

**Deliverables:**
- **Pre-requisite:** Google OAuth verification (2-4 weeks)
  - [ ] Create Google Cloud project
  - [ ] Configure OAuth Consent Screen
  - [ ] Submit for verification
  - [ ] Wait for approval

- Backend:
  - [x] YouTubeToken model
  - [x] `GET /api/auth/youtube` (OAuth init)
  - [x] `GET /api/auth/youtube/callback`
  - [x] `GET /api/auth/youtube/status`
  - [x] `DELETE /api/auth/youtube` (disconnect)
  - [x] `POST /api/upload` (resumable upload)
  - [x] Token encryption (Fernet)
  - [x] Token refresh logic

- Frontend:
  - [x] YouTube connect button
  - [x] Upload modal (title, description, tags, privacy)
  - [x] YouTube link display after upload

**Acceptance Criteria:**
- User can connect YouTube account (OAuth flow)
- User can upload video with metadata
- Upload handles large files (resumable, 256KB chunks)
- YouTube URL saved to track
- Token auto-refreshed when expired
- User can disconnect YouTube

**Critical Blocker:**
Without Google verification, videos remain **private permanently**.

---

### 13.6. Phase 5: Batch Generation (3 pts, ~15h)

**Goal:** Generate up to 10 tracks in one request.

**Deliverables:**
- Backend:
  - [x] Batch + BatchItem models
  - [x] `POST /api/batch/generate` (synchronous, sequential)
  - [x] `GET /api/batch/{id}` (status polling)
  - [x] BatchService (process all prompts, continue on failure)

- Frontend:
  - [x] `/batch` page
  - [x] Batch form (textarea, 1 prompt per line, max 10)
  - [x] Progress tracker (polling every 2s)
  - [x] Results page (success + failures)

**Acceptance Criteria:**
- User can submit 1-10 prompts at once
- System processes sequentially (~90s per track)
- Progress visible in real-time
- If 1 track fails, others continue
- Results show success count + error details

---

### 13.7. Dependencies Between Phases

```mermaid
graph TD
    P1[Phase 1: Generate<br/>Music + Image] --> P2[Phase 2: Render Video]
    P2 --> P3[Phase 3: Dashboard]
    P2 --> P4[Phase 4: YouTube Upload]
    P1 --> P5[Phase 5: Batch Generation]

    GV[Google Verification<br/>2-4 weeks] -.->|Required for public videos| P4

    style P1 fill:#e1f5ff
    style P2 fill:#ffe1f5
    style P3 fill:#fff4e1
    style P4 fill:#ffe1e1
    style P5 fill:#e1ffe1
    style GV fill:#ffcccc,stroke:#ff0000
```

**Critical Path:**
1. Phase 1 (foundation)
2. Phase 2 (video needed for upload)
3. **Google Verification (parallel with Phase 3)**
4. Phase 4 (upload)
5. Phase 5 (optional enhancement)

---

## 14. Appendix

### 14.1. Glossary

- **Track**: A single generated music piece with associated image and optional video
- **Batch**: A group of tracks generated together (up to 10)
- **JWT**: JSON Web Token - authentication token format
- **OAuth 2.0**: Authorization protocol used by YouTube
- **Resumable Upload**: Upload technique for large files (chunked)
- **Fernet**: Symmetric encryption algorithm for token storage
- **FFmpeg**: Multimedia framework for video/audio processing
- **shadcn/ui**: Collection of re-usable React components

---

### 14.2. External API Documentation

- **Mubert API**: https://mubert.com/api
- **OpenAI DALL·E 3**: https://platform.openai.com/docs/guides/images
- **YouTube Data API v3**: https://developers.google.com/youtube/v3
- **FFmpeg Documentation**: https://ffmpeg.org/documentation.html

---

### 14.3. Key Repositories & Tools

- **FastAPI**: https://github.com/tiangolo/fastapi
- **Next.js**: https://github.com/vercel/next.js
- **shadcn/ui**: https://ui.shadcn.com
- **SQLAlchemy**: https://github.com/sqlalchemy/sqlalchemy
- **Alembic**: https://github.com/sqlalchemy/alembic

---

### 14.4. Cost Breakdown

#### **Development (Mubert Trial)**

| Service | Usage | Cost |
|---------|-------|------|
| Mubert Trial | 500 tracks/mo | $49/mo |
| OpenAI DALL·E 3 | 100 images @ $0.040 | $4/mo |
| **Total** | | **$53/mo** |

#### **Production (Mubert Startup)**

| Service | Usage | Cost |
|---------|-------|------|
| Mubert Startup | 5000 tracks/mo | $199/mo |
| OpenAI DALL·E 3 | 300 images @ $0.040 | $12/mo |
| AWS EC2 (t3.medium) | 730 hours | ~$30/mo* |
| **Total** | | **~$241/mo** |

\* *EC2 cost estimate for future production deployment*

---

### 14.5. Open Questions

1. **Backup Automation**: When to implement automated backups? (Not in MVP)
2. **Monitoring**: Add Sentry in production? (Recommended but not required)
3. **Rate Limiting**: Implement API rate limits? (Not needed for single-user)
4. **HTTPS**: How to handle SSL in production? (Nginx reverse proxy + Let's Encrypt)

---

### 14.6. Document Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-12 | Architecture Team | Initial draft |

---

**End of Document**
