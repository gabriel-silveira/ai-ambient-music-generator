# Architecture Documentation

Welcome to the AI Ambient Music Generator architecture documentation.

---

## 📚 Documents

### [System Architecture](./system-architecture.md)
**Complete technical architecture specification** covering:
- High-level system design
- Technology stack
- Data architecture (ERD, schemas)
- API architecture (REST endpoints)
- Security architecture (JWT, OAuth, encryption)
- Storage architecture (flat file system)
- Deployment architecture (Docker Compose)
- Architectural decisions (ADRs)
- Risks and trade-offs
- Implementation roadmap (5 phases)

**Length:** ~150 pages equivalent
**Audience:** Developers, architects, technical stakeholders

---

### [Project Structure](./project-structure.md)
**Detailed directory structure and file organization** including:
- Complete monorepo structure
- Backend (FastAPI) structure
- Frontend (Next.js) structure
- Configuration files
- Naming conventions
- Git ignore rules

**Length:** ~20 pages equivalent
**Audience:** Developers starting implementation

---

## 🎯 Quick Reference

### System Overview

```
┌─────────────┐
│   Browser   │
└──────┬──────┘
       │ HTTPS
       ↓
┌─────────────────────────────────────────┐
│         Next.js Frontend (3000)         │
│  - React 19 + TypeScript                │
│  - shadcn/ui + TailwindCSS              │
└──────────────┬──────────────────────────┘
               │ REST API
               ↓
┌─────────────────────────────────────────┐
│        FastAPI Backend (8000)           │
│  - Python 3.11+                         │
│  - SQLAlchemy + Alembic                 │
│  - JWT Auth                             │
└─────┬───────────────────────┬───────────┘
      │                       │
      ↓                       ↓
┌─────────────┐      ┌─────────────────┐
│ PostgreSQL  │      │  File Storage   │
│   (5432)    │      │  (storage/*)    │
└─────────────┘      └─────────────────┘
      │
      └─ Tracks, Users, Tokens, Batches
```

---

### External APIs

| Service | Purpose | Cost (Dev) | Cost (Prod) |
|---------|---------|-----------|-------------|
| **Mubert API 3.0** | Music generation | $49/mo | $199/mo |
| **OpenAI DALL·E 3** | Image generation | ~$4/mo | ~$12/mo |
| **YouTube Data API v3** | Video upload | Free | Free |
| **FFmpeg** | Video rendering | Free | Free |

**Total Cost:** $53/mo (dev) → $211/mo (prod)

---

### 5-Phase Roadmap

```
Phase 1: Music + Image Generation (8 pts, ~40h)
         ↓
Phase 2: Video Rendering (5 pts, ~25h)
         ↓
Phase 3: Dashboard + History (3 pts, ~15h)
         ↓
Phase 4: YouTube Upload (8 pts, ~40h) ⚠️ Requires Google verification
         ↓
Phase 5: Batch Generation (3 pts, ~15h)

Total: 27 points, ~135 hours
```

---

### Key Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| **Monorepo** | Simpler for single-user, local development |
| **Synchronous processing** | No need for Redis/Celery (single-user) |
| **Flat file storage** | No cloud costs, faster access |
| **JWT auth** | Stateless, scalable |
| **PostgreSQL** | Production-ready, better concurrency |
| **Single-user hardcoded** | No registration UI needed |

---

### Data Models (ERD Summary)

```
users (1) ─┬─→ tracks (N)
           ├─→ youtube_tokens (1)
           ├─→ refresh_tokens (N)
           └─→ batches (N)

batches (1) ─→ batch_items (N)

batch_items (N) ─→ tracks (1) [optional]
```

**Key Tables:**
- `users` - Single user account
- `tracks` - Generated music/image/video
- `youtube_tokens` - OAuth tokens (encrypted)
- `refresh_tokens` - JWT refresh tokens
- `batches` - Batch generation requests
- `batch_items` - Individual tracks in batch

---

### API Endpoints Summary

#### **Authentication**
- `POST /api/auth/login` - Login
- `POST /api/auth/refresh` - Refresh token
- `POST /api/auth/logout` - Logout

#### **Track Management**
- `POST /api/generate` - Generate music + image
- `POST /api/render` - Render video
- `GET /api/history` - List tracks (paginated)
- `GET /api/tracks/{id}` - Get track details
- `DELETE /api/tracks/{id}` - Delete track + files
- `GET /api/stats` - Usage statistics

#### **YouTube Integration**
- `GET /api/auth/youtube` - OAuth init
- `GET /api/auth/youtube/callback` - OAuth callback
- `GET /api/auth/youtube/status` - Connection status
- `DELETE /api/auth/youtube` - Disconnect
- `POST /api/upload` - Upload to YouTube

#### **Batch Generation**
- `POST /api/batch/generate` - Generate multiple tracks
- `GET /api/batch/{id}` - Batch status (polling)

---

### Security Features

- ✅ **JWT Authentication** (access 15min, refresh 7d)
- ✅ **Password Hashing** (bcrypt, cost 12)
- ✅ **OAuth Token Encryption** (Fernet symmetric encryption)
- ✅ **CSRF Protection** (state parameter in OAuth)
- ✅ **Input Validation** (Pydantic schemas)
- ✅ **SQL Injection Prevention** (SQLAlchemy parameterized queries)

---

### File Specifications

#### **Music (MP3)**
- Bitrate: 320 kbps
- Sample Rate: 44.1 kHz
- Duration: 120-900s
- Avg Size: ~8 MB per 5 min

#### **Images (PNG)**
- Resolution: 1920x1080
- Source: DALL·E 3 (1792x1024) → Pillow upscale
- Avg Size: ~3 MB

#### **Videos (MP4)**
- Resolution: 1920x1080 @ 30fps
- Codec: H.264 (CRF 23) + AAC
- Avg Size: ~12 MB per 5 min

**Total per track:** ~23 MB

---

### Deployment (Docker Compose)

```yaml
services:
  postgres:    # PostgreSQL 15
  backend:     # FastAPI (Python 3.11)
  frontend:    # Next.js 16.0.1

volumes:
  postgres_data
  ./storage (bind mount)
```

**Ports:**
- Frontend: `http://localhost:3000`
- Backend: `http://localhost:8000`
- API Docs: `http://localhost:8000/docs`
- PostgreSQL: `localhost:5432`

---

## 🚀 Getting Started

### Prerequisites
- Docker 24.x + Docker Compose 2.x
- FFmpeg 6.0+ (for video rendering)
- 50+ GB free disk space

### Quick Start

```bash
# 1. Clone repository
git clone <repo-url>
cd ai-ambient-music-generator

# 2. Set up environment
cp .env.example .env
# Edit .env with your API keys

# 3. Start services
docker compose up -d

# 4. Run migrations
docker compose exec backend alembic upgrade head

# 5. Create user (optional)
docker compose exec backend python scripts/create_user.py

# 6. Access application
open http://localhost:3000
```

---

## 📋 Implementation Checklist

### Phase 1: Foundation (8 pts)
- [ ] Backend: Database setup + Docker
- [ ] Backend: User model + JWT auth
- [ ] Backend: Track model
- [ ] Backend: POST /api/generate (Mubert + DALL·E)
- [ ] Frontend: Login page
- [ ] Frontend: Dashboard layout
- [ ] Frontend: Generate form
- [ ] Tests: Unit + integration (70%+ coverage)

### Phase 2: Video Rendering (5 pts)
- [ ] Backend: POST /api/render (FFmpeg)
- [ ] Frontend: Render button + video preview
- [ ] Tests: FFmpeg service tests

### Phase 3: Dashboard (3 pts)
- [ ] Backend: History + stats endpoints
- [ ] Frontend: /history page with filters
- [ ] Frontend: Stats cards
- [ ] Tests: History + stats tests

### Phase 4: YouTube Upload (8 pts)
- [ ] **PRE-REQUISITE: Google OAuth verification** ⚠️
- [ ] Backend: YouTube OAuth flow
- [ ] Backend: Resumable upload
- [ ] Backend: Token encryption
- [ ] Frontend: YouTube connect + upload modal
- [ ] Tests: OAuth + upload tests

### Phase 5: Batch Generation (3 pts)
- [ ] Backend: Batch models + service
- [ ] Frontend: Batch form + progress
- [ ] Tests: Batch generation tests

---

## ⚠️ Critical Blockers

### 1. YouTube Verification (Phase 4)
**Issue:** Without Google verification, videos remain **private permanently**.

**Action Required:**
1. Create Google Cloud project
2. Configure OAuth Consent Screen
3. Submit for verification
4. **Wait 2-4 weeks for approval**

**Timeline:** Start immediately in parallel with Phase 1-3.

---

### 2. Mubert Plan for Production
**Issue:** Trial plan ($49/mo) cannot be monetized on YouTube.

**Action Required:**
- Use Trial for development/testing
- **Upgrade to Startup ($199/mo) before production launch**

---

## 📊 Monitoring & Metrics

### Key Metrics to Track
- **Monthly Mubert Usage**: X/500 (Trial) or X/5000 (Startup)
- **Storage Used**: GB (monitor via dashboard)
- **Daily YouTube Uploads**: < 6 per day (API quota limit)
- **Track Generation Time**: Target < 90s per track
- **Video Rendering Time**: Target < 60s per 5-min video

### Logging
- Backend: Python `logging` module (JSON format)
- Frontend: Console errors (development)
- Docker: `docker compose logs -f [service]`

---

## 🔒 Security Considerations

### Production Checklist
- [ ] Enable HTTPS (Nginx + Let's Encrypt)
- [ ] Rotate JWT secret
- [ ] Use strong database password
- [ ] Restrict CORS origins
- [ ] Enable rate limiting (optional)
- [ ] Set up firewall rules (ports 80, 443 only)
- [ ] Configure automated backups

---

## 🐛 Troubleshooting

### Common Issues

**Issue:** Docker containers won't start
- **Solution:** Check logs with `docker compose logs [service]`

**Issue:** Database connection failed
- **Solution:** Verify `DATABASE_URL` in `.env`

**Issue:** FFmpeg rendering timeout
- **Solution:** Increase timeout in `ffmpeg_service.py`

**Issue:** YouTube upload quota exceeded
- **Solution:** Wait until next day (quota resets at midnight PST)

**Issue:** DALL·E images look pixelated
- **Solution:** Verify upscale using LANCZOS filter in Pillow

---

## 📞 Support

### Resources
- **System Architecture**: [system-architecture.md](./system-architecture.md)
- **Project Structure**: [project-structure.md](./project-structure.md)
- **API Documentation**: http://localhost:8000/docs (when running)

### External Documentation
- [Mubert API Docs](https://mubert.com/api)
- [OpenAI DALL·E 3 Guide](https://platform.openai.com/docs/guides/images)
- [YouTube Data API v3](https://developers.google.com/youtube/v3)
- [FastAPI Documentation](https://fastapi.tiangolo.com)
- [Next.js Documentation](https://nextjs.org/docs)

---

**Last Updated:** 2025-11-12
**Version:** 1.0
