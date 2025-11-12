# Project Structure - AI Ambient Music Generator

**Version:** 1.0
**Date:** 2025-11-12

---

## Complete Directory Structure

```
ai-ambient-music-generator/
│
├── backend/                          # FastAPI Backend
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                   # FastAPI app entry point
│   │   ├── config.py                 # Settings (Pydantic BaseSettings)
│   │   ├── database.py               # SQLAlchemy setup + session
│   │   │
│   │   ├── models/                   # SQLAlchemy ORM models
│   │   │   ├── __init__.py
│   │   │   ├── user.py               # User model
│   │   │   ├── track.py              # Track model
│   │   │   ├── youtube_token.py      # YouTube OAuth tokens
│   │   │   ├── refresh_token.py      # JWT refresh tokens
│   │   │   ├── batch.py              # Batch model
│   │   │   └── batch_item.py         # BatchItem model
│   │   │
│   │   ├── schemas/                  # Pydantic schemas (validation)
│   │   │   ├── __init__.py
│   │   │   ├── auth.py               # LoginRequest, TokenResponse
│   │   │   ├── track.py              # TrackResponse, GenerateRequest
│   │   │   ├── batch.py              # BatchGenerateRequest, BatchStatusResponse
│   │   │   ├── upload.py             # UploadRequest
│   │   │   └── stats.py              # StatsResponse
│   │   │
│   │   ├── routers/                  # API route handlers
│   │   │   ├── __init__.py
│   │   │   ├── auth.py               # POST /api/auth/login, /refresh, /logout
│   │   │   ├── generate.py           # POST /api/generate
│   │   │   ├── render.py             # POST /api/render
│   │   │   ├── tracks.py             # GET /api/history, GET /api/tracks/{id}, DELETE /api/tracks/{id}
│   │   │   ├── stats.py              # GET /api/stats
│   │   │   ├── youtube.py            # YouTube OAuth + status endpoints
│   │   │   ├── upload.py             # POST /api/upload (YouTube upload)
│   │   │   └── batch.py              # POST /api/batch/generate, GET /api/batch/{id}
│   │   │
│   │   ├── services/                 # Business logic layer
│   │   │   ├── __init__.py
│   │   │   ├── mubert_service.py     # Mubert API integration
│   │   │   ├── openai_service.py     # DALL·E 3 integration
│   │   │   ├── ffmpeg_service.py     # Video rendering with FFmpeg
│   │   │   ├── youtube_service.py    # YouTube upload (resumable)
│   │   │   ├── storage_service.py    # File storage operations
│   │   │   ├── stats_service.py      # Statistics calculation
│   │   │   └── batch_service.py      # Batch generation logic
│   │   │
│   │   ├── middleware/               # Custom middleware
│   │   │   ├── __init__.py
│   │   │   └── auth_middleware.py    # JWT validation middleware
│   │   │
│   │   └── utils/                    # Utility functions
│   │       ├── __init__.py
│   │       ├── jwt_handler.py        # JWT create/validate
│   │       ├── password.py           # bcrypt hash/verify
│   │       └── encryption.py         # Fernet encryption for OAuth tokens
│   │
│   ├── migrations/                   # Alembic database migrations
│   │   ├── versions/
│   │   │   ├── 001_create_users_table.py
│   │   │   ├── 002_create_tracks_table.py
│   │   │   ├── 003_create_refresh_tokens_table.py
│   │   │   ├── 004_create_youtube_tokens_table.py
│   │   │   └── 005_create_batch_tables.py
│   │   ├── env.py
│   │   └── script.py.mako
│   │
│   ├── tests/                        # Unit + integration tests
│   │   ├── __init__.py
│   │   ├── conftest.py               # Pytest fixtures
│   │   │
│   │   ├── unit/
│   │   │   ├── __init__.py
│   │   │   ├── test_mubert_service.py
│   │   │   ├── test_openai_service.py
│   │   │   ├── test_ffmpeg_service.py
│   │   │   ├── test_youtube_service.py
│   │   │   ├── test_jwt_handler.py
│   │   │   └── test_encryption.py
│   │   │
│   │   └── integration/
│   │       ├── __init__.py
│   │       ├── test_auth_endpoints.py
│   │       ├── test_generate_endpoints.py
│   │       ├── test_track_endpoints.py
│   │       ├── test_batch_endpoints.py
│   │       └── test_youtube_endpoints.py
│   │
│   ├── scripts/                      # CLI/management scripts
│   │   ├── create_user.py            # Create initial user
│   │   └── backup.sh                 # Backup script (storage + DB)
│   │
│   ├── .env.example                  # Environment variables template
│   ├── requirements.txt              # Python dependencies
│   ├── Dockerfile                    # Backend Docker image
│   ├── alembic.ini                   # Alembic configuration
│   ├── pytest.ini                    # Pytest configuration
│   └── README.md                     # Backend-specific docs
│
├── frontend/                         # Next.js Frontend
│   ├── app/
│   │   ├── layout.tsx                # Root layout (providers, fonts)
│   │   ├── page.tsx                  # Landing page (/)
│   │   ├── globals.css               # Global styles (Tailwind)
│   │   │
│   │   ├── (auth)/                   # Auth layout group (no auth required)
│   │   │   ├── layout.tsx            # Auth layout (centered, minimal)
│   │   │   └── login/
│   │   │       └── page.tsx          # Login page
│   │   │
│   │   └── (protected)/              # Protected layout group (auth required)
│   │       ├── layout.tsx            # Protected layout (navbar, sidebar)
│   │       │
│   │       ├── dashboard/
│   │       │   └── page.tsx          # Dashboard (/dashboard)
│   │       │
│   │       ├── generate/
│   │       │   └── page.tsx          # Generate page (/generate)
│   │       │
│   │       ├── history/
│   │       │   └── page.tsx          # History page (/history)
│   │       │
│   │       └── batch/
│   │           └── page.tsx          # Batch generation page (/batch)
│   │
│   ├── components/
│   │   ├── ui/                       # shadcn/ui base components
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── input.tsx
│   │   │   ├── textarea.tsx
│   │   │   ├── select.tsx
│   │   │   ├── alert.tsx
│   │   │   ├── badge.tsx
│   │   │   ├── progress.tsx
│   │   │   └── ...                   # Other shadcn components
│   │   │
│   │   ├── layout/
│   │   │   ├── Navbar.tsx            # Top navigation bar
│   │   │   ├── Sidebar.tsx           # Side navigation (optional)
│   │   │   └── Footer.tsx            # Footer
│   │   │
│   │   ├── auth/
│   │   │   ├── LoginForm.tsx         # Login form component
│   │   │   └── ProtectedRoute.tsx    # Route guard wrapper
│   │   │
│   │   ├── tracks/
│   │   │   ├── TrackCard.tsx         # Track display card
│   │   │   ├── TrackList.tsx         # List of tracks (grid)
│   │   │   ├── TrackDetailsModal.tsx # Full track details modal
│   │   │   ├── DeleteConfirmDialog.tsx # Delete confirmation
│   │   │   └── AudioPlayer.tsx       # Custom audio player (optional)
│   │   │
│   │   ├── generate/
│   │   │   ├── GenerateForm.tsx      # Prompt + duration form
│   │   │   ├── GeneratingState.tsx   # Loading state during generation
│   │   │   └── TrackPreview.tsx      # Preview after generation
│   │   │
│   │   ├── render/
│   │   │   ├── RenderButton.tsx      # Trigger video rendering
│   │   │   ├── RenderProgress.tsx    # Rendering progress bar
│   │   │   └── VideoPreview.tsx      # Video player
│   │   │
│   │   ├── stats/
│   │   │   ├── StatsCards.tsx        # Dashboard stats display
│   │   │   └── RecentTracks.tsx      # Recent tracks widget
│   │   │
│   │   ├── batch/
│   │   │   ├── BatchForm.tsx         # Multi-prompt input form
│   │   │   ├── BatchProgress.tsx     # Real-time batch progress
│   │   │   └── BatchResults.tsx      # Success/failure summary
│   │   │
│   │   └── youtube/
│   │       ├── YouTubeConnectButton.tsx  # OAuth connection button
│   │       ├── YouTubeStatusBadge.tsx    # Connection status indicator
│   │       └── UploadToYouTubeModal.tsx  # Upload form modal
│   │
│   ├── hooks/                        # Custom React hooks
│   │   ├── useAuth.ts                # Authentication state
│   │   ├── useGenerate.ts            # Track generation
│   │   ├── useRender.ts              # Video rendering
│   │   ├── useHistory.ts             # Track history fetching
│   │   ├── useStats.ts               # Statistics fetching
│   │   ├── useYouTube.ts             # YouTube connection/upload
│   │   └── useBatch.ts               # Batch generation
│   │
│   ├── lib/                          # Utility libraries
│   │   ├── api.ts                    # Axios instance + interceptors
│   │   ├── auth.ts                   # Auth helpers (getToken, logout)
│   │   ├── formatters.ts             # Date/duration formatting
│   │   └── utils.ts                  # General utilities (cn, etc.)
│   │
│   ├── types/                        # TypeScript type definitions
│   │   ├── index.ts                  # Global types
│   │   ├── api.ts                    # API response types
│   │   └── components.ts             # Component prop types
│   │
│   ├── public/                       # Static assets
│   │   ├── favicon.ico
│   │   ├── logo.svg
│   │   └── images/
│   │
│   ├── .env.local.example            # Environment variables template
│   ├── package.json                  # Node.js dependencies
│   ├── tsconfig.json                 # TypeScript configuration
│   ├── tailwind.config.ts            # TailwindCSS configuration
│   ├── next.config.js                # Next.js configuration
│   ├── postcss.config.js             # PostCSS configuration
│   ├── components.json               # shadcn/ui configuration
│   ├── Dockerfile                    # Frontend Docker image
│   └── README.md                     # Frontend-specific docs
│
├── storage/                          # Generated files storage (Docker volume)
│   ├── music/
│   │   └── {uuid}.mp3                # Generated music files
│   ├── images/
│   │   └── {uuid}.png                # Generated images
│   └── videos/
│       └── {uuid}.mp4                # Rendered videos
│
├── docs/                             # Project documentation
│   ├── architecture/
│   │   ├── system-architecture.md    # This document
│   │   └── project-structure.md      # Directory structure
│   ├── api/
│   │   └── openapi.json              # Generated OpenAPI spec
│   └── guides/
│       ├── setup.md                  # Setup instructions
│       ├── deployment.md             # Deployment guide
│       └── backup.md                 # Backup/restore guide
│
├── specs/                            # Product specifications
│   └── product/
│       └── prd.md                    # Product Requirements Document
│
├── .claude/                          # Claude Code session data
│   └── sessions/
│       └── main-features/
│           ├── requirements.md
│           └── issues/
│               ├── fase-1-geracao-musica-imagem.md
│               ├── fase-2-renderizacao-video.md
│               ├── fase-3-dashboard-historico.md
│               ├── fase-4-upload-youtube.md
│               └── fase-5-geracao-em-lote.md
│
├── .git/                             # Git repository
├── .gitignore                        # Git ignore rules
├── docker-compose.yml                # Docker Compose configuration
├── docker-compose.prod.yml           # Production Docker Compose (future)
├── .env.example                      # Root environment variables template
├── README.md                         # Main project README
└── LICENSE                           # Project license

```

---

## Key Files Detail

### Backend Key Files

#### **app/main.py**
Entry point for FastAPI application. Sets up routers, middleware, CORS.

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import auth, generate, render, tracks, stats, youtube, upload, batch

app = FastAPI(title="AI Ambient Music Generator API", version="1.0.0")

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(auth.router)
app.include_router(generate.router)
app.include_router(render.router)
app.include_router(tracks.router)
app.include_router(stats.router)
app.include_router(youtube.router)
app.include_router(upload.router)
app.include_router(batch.router)
```

---

#### **app/config.py**
Application settings using Pydantic BaseSettings.

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # Database
    DATABASE_URL: str

    # JWT
    JWT_SECRET: str
    JWT_ALGORITHM: str = "HS256"
    JWT_ACCESS_TOKEN_EXPIRE_MINUTES: int = 15
    JWT_REFRESH_TOKEN_EXPIRE_DAYS: int = 7

    # Encryption
    ENCRYPTION_KEY: str

    # External APIs
    MUBERT_API_KEY: str
    OPENAI_API_KEY: str
    YOUTUBE_CLIENT_ID: str
    YOUTUBE_CLIENT_SECRET: str
    YOUTUBE_REDIRECT_URI: str

    class Config:
        env_file = ".env"

settings = Settings()
```

---

#### **app/database.py**
SQLAlchemy database setup.

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from app.config import settings

engine = create_engine(settings.DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

### Frontend Key Files

#### **app/layout.tsx**
Root layout with providers.

```typescript
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'AI Ambient Music Generator',
  description: 'Generate ambient music with AI',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className="dark">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

---

#### **lib/api.ts**
Axios client with JWT interceptors.

```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8000',
  timeout: 900000, // 15 minutes
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor (add token)
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('access_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor (handle refresh)
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      const refreshToken = localStorage.getItem('refresh_token');
      if (refreshToken) {
        try {
          const { data } = await axios.post(
            `${process.env.NEXT_PUBLIC_API_URL}/api/auth/refresh`,
            { refresh_token: refreshToken }
          );
          localStorage.setItem('access_token', data.access_token);
          error.config.headers.Authorization = `Bearer ${data.access_token}`;
          return axios(error.config);
        } catch {
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

## Configuration Files

### **.env.example** (Root)

```bash
# Backend
DATABASE_URL=postgresql://postgres:postgres@postgres:5432/ai_ambient_music
JWT_SECRET=your-super-secret-jwt-key-min-32-chars
ENCRYPTION_KEY=your-fernet-encryption-key-44-chars
MUBERT_API_KEY=your-mubert-api-key
OPENAI_API_KEY=sk-your-openai-api-key
YOUTUBE_CLIENT_ID=your-client-id.apps.googleusercontent.com
YOUTUBE_CLIENT_SECRET=your-client-secret
YOUTUBE_REDIRECT_URI=http://localhost:8000/api/auth/youtube/callback

# Frontend
NEXT_PUBLIC_API_URL=http://localhost:8000
```

---

### **docker-compose.yml**

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: ai-music-db
    environment:
      POSTGRES_DB: ai_ambient_music
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
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
    container_name: ai-music-backend
    env_file:
      - .env
    volumes:
      - ./storage:/app/storage
      - ./backend:/app
    ports:
      - "8000:8000"
    depends_on:
      postgres:
        condition: service_healthy

  frontend:
    build:
      context: ./frontend
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

volumes:
  postgres_data:
```

---

### **backend/requirements.txt**

```txt
# Core
fastapi==0.109.0
uvicorn[standard]==0.27.0
python-multipart==0.0.9

# Database
sqlalchemy==2.0.25
alembic==1.13.1
psycopg2-binary==2.9.9

# Authentication & Security
pyjwt==2.8.0
passlib[bcrypt]==1.7.4
cryptography==42.0.1

# HTTP Client
httpx==0.26.0

# Image Processing
pillow==10.2.0

# Validation
pydantic==2.5.3
pydantic-settings==2.1.0

# Environment
python-dotenv==1.0.0
```

---

### **frontend/package.json**

```json
{
  "name": "ai-ambient-music-frontend",
  "version": "1.0.0",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "16.0.1",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "axios": "^1.6.5",
    "framer-motion": "^10.18.0",
    "lucide-react": "^0.309.0",
    "@radix-ui/react-dialog": "^1.0.5",
    "@radix-ui/react-select": "^2.0.0",
    "@radix-ui/react-alert-dialog": "^1.0.5"
  },
  "devDependencies": {
    "typescript": "^5.3.3",
    "@types/node": "^20.11.0",
    "@types/react": "^18.2.48",
    "@types/react-dom": "^18.2.18",
    "tailwindcss": "^3.4.1",
    "postcss": "^8.4.33",
    "autoprefixer": "^10.4.17",
    "eslint": "^8.56.0",
    "eslint-config-next": "16.0.1"
  }
}
```

---

## File Naming Conventions

### Backend (Python)
- **Files**: `snake_case.py`
- **Classes**: `PascalCase`
- **Functions**: `snake_case()`
- **Constants**: `UPPER_SNAKE_CASE`

### Frontend (TypeScript)
- **Components**: `PascalCase.tsx`
- **Hooks**: `camelCase.ts` (prefix with `use`)
- **Utilities**: `camelCase.ts`
- **Types**: `PascalCase` (interfaces, types)

### Storage Files
- **Format**: `{uuid}.{ext}` (lowercase UUID v4)
- **Examples**:
  - `550e8400-e29b-41d4-a716-446655440000.mp3`
  - `550e8400-e29b-41d4-a716-446655440000.png`
  - `550e8400-e29b-41d4-a716-446655440000.mp4`

---

## Git Ignore Rules

### **.gitignore**

```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv/
*.egg-info/

# Node.js
node_modules/
.next/
out/
dist/
build/

# Environment
.env
.env.local

# Storage (generated files)
storage/music/*.mp3
storage/images/*.png
storage/videos/*.mp4

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Database
*.db
*.sqlite

# Logs
*.log
logs/

# Docker
docker-compose.override.yml
```

---

## Next Steps

1. **Initialize Project Structure**:
   ```bash
   mkdir -p backend/app/{models,schemas,routers,services,middleware,utils}
   mkdir -p backend/{migrations/versions,tests/{unit,integration},scripts}
   mkdir -p frontend/{app,components,hooks,lib,types,public}
   mkdir -p storage/{music,images,videos}
   mkdir -p docs/{architecture,api,guides}
   ```

2. **Set Up Git**:
   ```bash
   git init
   cp .env.example .env
   # Edit .env with your credentials
   ```

3. **Install Dependencies**:
   ```bash
   # Backend
   cd backend
   python -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt

   # Frontend
   cd ../frontend
   npm install
   ```

4. **Start Development**:
   ```bash
   docker compose up -d
   ```

---

**End of Document**
