# Quick Start Guide - AI Ambient Music Generator

**For Developers Starting Implementation**

---

## 🎯 Before You Start

### ✅ Prerequisites Checklist

- [ ] Read [Executive Summary](EXECUTIVE_SUMMARY.md) (5 min)
- [ ] Review [System Architecture](architecture/system-architecture.md) (30 min)
- [ ] **CRITICAL:** Start [Google OAuth verification](#step-1-google-oauth-verification) process
- [ ] Obtain API credentials (Mubert + OpenAI)
- [ ] Install Docker 24.x + Docker Compose 2.x
- [ ] Install FFmpeg 6.0+
- [ ] Ensure 50+ GB free disk space

---

## 🚀 Setup (30 minutes)

### Step 1: Google OAuth Verification (START NOW) 🔴

**Why now?** Approval takes 2-4 weeks. Start immediately to avoid blocking Phase 4.

```bash
# 1. Create Google Cloud project
# Visit: https://console.cloud.google.com/

# 2. Enable YouTube Data API v3
# Navigation: APIs & Services > Library > Search "YouTube Data API v3" > Enable

# 3. Configure OAuth Consent Screen
# Navigation: APIs & Services > OAuth consent screen
# - Type: External
# - App name: AI Ambient Music Generator
# - User support email: your-email@example.com
# - Scopes: Add https://www.googleapis.com/auth/youtube.upload

# 4. Create OAuth 2.0 credentials
# Navigation: APIs & Services > Credentials > Create Credentials > OAuth client ID
# - Application type: Web application
# - Name: YouTube OAuth Client
# - Authorized redirect URIs: http://localhost:8000/api/auth/youtube/callback
# - Copy Client ID and Client Secret (save for .env)

# 5. Submit for verification
# Navigation: OAuth consent screen > Publish App > Submit for Verification
# - Fill detailed form explaining use case
# - Upload demo video (optional but recommended)
# - Wait 2-4 weeks for approval
```

---

### Step 2: Clone Repository

```bash
git clone https://github.com/yourusername/ai-ambient-music-generator.git
cd ai-ambient-music-generator
```

---

### Step 3: Get API Credentials

#### Mubert API (Trial - $49/month)
```bash
# 1. Visit: https://mubert.com/api
# 2. Sign up for Trial plan ($49/month)
# 3. Copy API key
# 4. Note: Trial cannot be monetized on YouTube
#    Upgrade to Startup ($199/mo) before production
```

#### OpenAI API
```bash
# 1. Visit: https://platform.openai.com/api-keys
# 2. Create new secret key
# 3. Copy key (starts with sk-)
# 4. Enable billing: https://platform.openai.com/account/billing
```

#### Generate Encryption Key (for OAuth tokens)
```bash
# Run in Python:
python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"

# Copy output (44 characters, ends with =)
# Example: vX8_Zk3qR9LmNpWsYtUvXyZaBcDeFgHiJkLmNoPqRsT=
```

---

### Step 4: Configure Environment

```bash
# 1. Copy example .env
cp .env.example .env

# 2. Edit .env with your credentials
nano .env
```

**Required .env variables:**

```bash
# Database
DATABASE_URL=postgresql://postgres:your_secure_password@postgres:5432/ai_ambient_music

# JWT Authentication
JWT_SECRET=your-super-secret-jwt-key-minimum-32-characters-long
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=15
JWT_REFRESH_TOKEN_EXPIRE_DAYS=7

# Encryption (for YouTube OAuth tokens)
ENCRYPTION_KEY=vX8_Zk3qR9LmNpWsYtUvXyZaBcDeFgHiJkLmNoPqRsT=

# Mubert API
MUBERT_API_KEY=your_mubert_api_key_here

# OpenAI API
OPENAI_API_KEY=sk-your-openai-api-key-here

# YouTube OAuth (from Step 1)
YOUTUBE_CLIENT_ID=123456789.apps.googleusercontent.com
YOUTUBE_CLIENT_SECRET=GOCSPX-your-client-secret
YOUTUBE_REDIRECT_URI=http://localhost:8000/api/auth/youtube/callback

# Frontend
NEXT_PUBLIC_API_URL=http://localhost:8000
```

---

### Step 5: Start Services

```bash
# Start all containers (Postgres + Backend + Frontend)
docker compose up -d --build

# Check logs
docker compose logs -f

# Wait for services to be healthy (~30 seconds)
```

---

### Step 6: Initialize Database

```bash
# Run migrations
docker compose exec backend alembic upgrade head

# Create seed user (optional - single user)
docker compose exec backend python scripts/create_user.py
# Enter email: admin@example.com
# Enter password: SecurePassword123!
```

---

### Step 7: Verify Installation

```bash
# Check services are running
docker compose ps

# Expected output:
# NAME                 STATUS
# ai-music-backend     Up (healthy)
# ai-music-frontend    Up
# ai-music-db          Up (healthy)

# Test API
curl http://localhost:8000/docs
# Should return Swagger UI

# Test Frontend
curl http://localhost:3000
# Should return Next.js page
```

---

### Step 8: Access Application

Open in browser:
- **Frontend:** http://localhost:3000
- **Backend API:** http://localhost:8000
- **API Docs (Swagger):** http://localhost:8000/docs
- **PostgreSQL:** localhost:5432

**Login credentials:**
- Email: admin@example.com (or what you entered in Step 6)
- Password: SecurePassword123! (or what you entered in Step 6)

---

## 📋 Implementation Phases

### Phase 1: Music + Image Generation (Week 1) ⚡ START HERE

**Goal:** Generate music (Mubert) + cover art (DALL·E) from text prompt.

**Tasks:**
```bash
# Backend
- [ ] Set up PostgreSQL models (User, Track)
- [ ] Implement JWT authentication
- [ ] Create POST /api/auth/login endpoint
- [ ] Create POST /api/generate endpoint
  - [ ] Integrate Mubert API
  - [ ] Integrate DALL·E 3 API
  - [ ] Implement image upscaling (Pillow, LANCZOS)
  - [ ] Save files to storage/ (UUID-based)
- [ ] Write unit tests (70%+ coverage)

# Frontend
- [ ] Create login page (/login)
- [ ] Create dashboard layout
- [ ] Create generate form (prompt + duration)
- [ ] Display generated track (audio player + image)
- [ ] Add loading states

# Testing
- [ ] Manual: Generate 5 different prompts
- [ ] Verify: MP3 (320kbps, 44.1kHz), PNG (1920x1080)
- [ ] Check: Files saved correctly in storage/
```

**Estimated time:** ~40 hours

**Files to create:**
```
backend/app/
├── models/user.py
├── models/track.py
├── schemas/auth.py
├── schemas/track.py
├── routers/auth.py
├── routers/generate.py
├── services/mubert_service.py
├── services/openai_service.py
└── services/storage_service.py

frontend/
├── app/(auth)/login/page.tsx
├── app/(protected)/dashboard/page.tsx
├── app/(protected)/generate/page.tsx
├── components/generate/GenerateForm.tsx
├── components/tracks/TrackCard.tsx
└── hooks/useGenerate.ts
```

---

### Phase 2: Video Rendering (Week 2)

**Goal:** Render static video (image + audio) with FFmpeg.

**Tasks:**
```bash
# Backend
- [ ] Create POST /api/render endpoint
- [ ] Implement FFmpegService
  - [ ] H.264 codec (CRF 23, preset medium)
  - [ ] 1920x1080 @ 30fps
  - [ ] Add fade-in/fade-out (2s)
- [ ] Update track status (rendered)
- [ ] Write unit tests for FFmpeg service

# Frontend
- [ ] Add "Render Video" button to track card
- [ ] Show rendering progress
- [ ] Display video player (HTML5)
- [ ] Add download video button

# Testing
- [ ] Manual: Render 5 different tracks
- [ ] Verify: MP4 playable in browser
- [ ] Check: Video quality (1080p, 30fps)
- [ ] Measure: Rendering time < 60s per 5-min track
```

**Estimated time:** ~25 hours

---

### Phase 3: Dashboard + History (Week 2-3)

**Goal:** Track management and usage statistics.

**Tasks:**
```bash
# Backend
- [ ] Create GET /api/history endpoint (paginated)
- [ ] Create GET /api/tracks/{id} endpoint
- [ ] Create DELETE /api/tracks/{id} endpoint
  - [ ] Delete DB record
  - [ ] Delete files (music, image, video)
- [ ] Create GET /api/stats endpoint
  - [ ] Calculate storage used
  - [ ] Calculate monthly Mubert usage
  - [ ] Count tracks by status

# Frontend
- [ ] Create /history page
  - [ ] Track list with filters (status)
  - [ ] Pagination (50 per page)
- [ ] Create track details modal
- [ ] Create delete confirmation dialog
- [ ] Add stats cards to dashboard
  - [ ] Total tracks
  - [ ] Storage used
  - [ ] Monthly Mubert usage (with alert at 80%)

# Testing
- [ ] Manual: Generate 20 tracks
- [ ] Test: Filter by status (all, generated, rendered)
- [ ] Test: Delete track (verify files removed)
- [ ] Verify: Stats calculated correctly
```

**Estimated time:** ~15 hours

---

### Phase 4: YouTube Upload (Week 3-4) ⚠️ Requires Google Approval

**Prerequisites:**
- ✅ Google OAuth verification approved (2-4 weeks wait)
- ✅ Phase 2 complete (videos ready for upload)

**Tasks:**
```bash
# Backend
- [ ] Create YouTubeToken model (encrypted storage)
- [ ] Create GET /api/auth/youtube endpoint (OAuth init)
- [ ] Create GET /api/auth/youtube/callback endpoint
  - [ ] Validate state (CSRF protection)
  - [ ] Exchange code for tokens
  - [ ] Encrypt tokens (Fernet)
  - [ ] Store in database
- [ ] Create POST /api/upload endpoint
  - [ ] Implement resumable upload (256KB chunks)
  - [ ] Handle token refresh
  - [ ] Update track status (uploaded)
- [ ] Create GET /api/auth/youtube/status endpoint
- [ ] Create DELETE /api/auth/youtube endpoint

# Frontend
- [ ] Add YouTube connect button
- [ ] Create upload modal (title, description, tags, privacy)
- [ ] Show YouTube link after upload
- [ ] Display connection status badge

# Testing
- [ ] Manual: Connect YouTube account
- [ ] Verify: Tokens encrypted in DB
- [ ] Test: Upload video (< 100 MB)
- [ ] Test: Upload video (> 100 MB, resumable)
- [ ] Verify: Video appears on YouTube
- [ ] Test: Disconnect and reconnect
```

**Estimated time:** ~40 hours

**Important:** Without Google verification, videos will remain **private permanently**.

---

### Phase 5: Batch Generation (Week 4)

**Goal:** Generate up to 10 tracks in one request.

**Tasks:**
```bash
# Backend
- [ ] Create Batch and BatchItem models
- [ ] Create POST /api/batch/generate endpoint
  - [ ] Process prompts sequentially
  - [ ] Continue on failure (resilient)
  - [ ] Update status per item
- [ ] Create GET /api/batch/{id} endpoint (status polling)

# Frontend
- [ ] Create /batch page
- [ ] Create batch form (textarea, max 10 prompts)
- [ ] Create progress tracker (polling every 2s)
- [ ] Create results page (success + failures)

# Testing
- [ ] Manual: Generate batch of 5 prompts
- [ ] Manual: Generate batch of 10 prompts
- [ ] Test: 1 prompt fails, others continue
- [ ] Verify: Progress updates in real-time
```

**Estimated time:** ~15 hours

---

## 🧪 Testing Strategy

### Unit Tests (Backend)
```bash
cd backend
pytest tests/unit/ -v --cov=app --cov-report=html

# Target: 70%+ coverage
```

### Integration Tests (Backend)
```bash
pytest tests/integration/ -v

# Test all API endpoints with real DB
```

### Frontend Tests
```bash
cd frontend
npm test

# Component tests with React Testing Library
```

### Manual Testing Checklist
- [ ] Generate track with various prompts
- [ ] Render video
- [ ] Upload to YouTube (Phase 4)
- [ ] Delete track (verify files removed)
- [ ] Batch generation (5 tracks)
- [ ] Error handling (invalid prompt, API timeout)

---

## 🐛 Troubleshooting

### Docker containers won't start
```bash
# Check logs
docker compose logs backend
docker compose logs frontend
docker compose logs postgres

# Rebuild from scratch
docker compose down -v
docker compose up -d --build
```

### Database connection failed
```bash
# Verify DATABASE_URL in .env
# Check Postgres is healthy
docker compose ps postgres

# Check Postgres logs
docker compose logs postgres
```

### FFmpeg not found
```bash
# Install FFmpeg in backend container
docker compose exec backend apt-get update
docker compose exec backend apt-get install -y ffmpeg

# Or rebuild container (Dockerfile includes FFmpeg)
docker compose up -d --build backend
```

### YouTube upload fails with 403
```bash
# Check:
# 1. YouTube OAuth tokens exist in DB
# 2. Tokens not expired (check youtube_tokens.expires_at)
# 3. Google OAuth verification approved
# 4. App not in "Testing" mode (must be published)
```

### Mubert API returns 401
```bash
# Check:
# 1. MUBERT_API_KEY in .env is correct
# 2. Trial plan active (check Mubert dashboard)
# 3. Monthly quota not exceeded (500 generations/month)
```

---

## 📊 Monitoring

### Key Metrics Dashboard

Add to dashboard (Phase 3):
- **Total tracks generated:** Count
- **Storage used:** GB (monitor weekly)
- **Monthly Mubert usage:** X/500 (Trial) or X/5000 (Startup)
- **Average generation time:** Seconds
- **Average rendering time:** Seconds
- **Failed generations:** Count (investigate if > 5%)

### Logs

```bash
# Backend logs (JSON structured)
docker compose logs -f backend | grep ERROR

# Frontend logs
docker compose logs -f frontend

# All logs
docker compose logs -f
```

---

## 🚀 Production Deployment (Future)

### When Ready
- ✅ All 5 phases complete
- ✅ 70%+ test coverage achieved
- ✅ Google OAuth verification approved
- ✅ Mubert upgraded to Startup ($199/mo)

### AWS EC2 Setup
```bash
# 1. Provision EC2 instance
# Type: t3.medium (2 vCPUs, 4GB RAM)
# OS: Ubuntu 22.04 LTS
# Storage: 100 GB SSD

# 2. Install Docker
sudo apt update
sudo apt install -y docker.io docker-compose

# 3. Clone repository
git clone <repo-url>
cd ai-ambient-music-generator

# 4. Configure production .env
cp .env.example .env
nano .env
# Update with production values (HTTPS URLs, strong passwords)

# 5. Start services
docker compose -f docker-compose.prod.yml up -d --build

# 6. Configure SSL (Let's Encrypt)
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com

# 7. Set up automated backups
crontab -e
# Add: 0 2 * * * /home/ubuntu/ai-ambient-music-generator/scripts/backup.sh
```

---

## 📞 Support

### Documentation
- [System Architecture](architecture/system-architecture.md) - Complete technical spec
- [Project Structure](architecture/project-structure.md) - Directory structure
- [Executive Summary](EXECUTIVE_SUMMARY.md) - Business overview
- [Phase Issues](.claude/sessions/main-features/issues/) - Detailed phase specs

### External Resources
- [FastAPI Docs](https://fastapi.tiangolo.com)
- [Next.js Docs](https://nextjs.org/docs)
- [Mubert API Docs](https://mubert.com/api)
- [OpenAI API Docs](https://platform.openai.com/docs)
- [YouTube Data API v3](https://developers.google.com/youtube/v3)

---

## ✅ Pre-Launch Checklist

Before production launch:

**Technical**
- [ ] All 5 phases implemented
- [ ] 70%+ test coverage
- [ ] Performance targets met (< 90s generation, < 60s render)
- [ ] Error handling comprehensive
- [ ] Logs structured (JSON)
- [ ] Backups automated

**External Services**
- [ ] Google OAuth verification APPROVED ✅
- [ ] Mubert upgraded to Startup ($199/mo)
- [ ] OpenAI billing active
- [ ] YouTube channel ready

**Security**
- [ ] HTTPS enabled (Let's Encrypt)
- [ ] Strong JWT secret (32+ chars)
- [ ] Database password secure
- [ ] Firewall configured (ports 80, 443 only)
- [ ] .env not in git (verify .gitignore)

**Documentation**
- [ ] README updated
- [ ] API docs generated (OpenAPI)
- [ ] Backup/restore procedure documented
- [ ] Troubleshooting guide complete

---

## 🎉 You're Ready!

Start with **Phase 1** (Music + Image Generation) and work sequentially through phases.

**Remember:** Start Google OAuth verification **NOW** to avoid blocking Phase 4.

Good luck! 🚀

---

**Last Updated:** November 12, 2025
**Version:** 1.0
