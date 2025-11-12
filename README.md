# AI Ambient Music Generator

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)
[![Next.js](https://img.shields.io/badge/Next.js-16.0.1-black.svg)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Latest-green.svg)](https://fastapi.tiangolo.com/)

Uma aplicação web **single-user** para criar e gerenciar músicas ambiente geradas por IA, com renderização de vídeo e upload automatizado para YouTube. Sistema completo de produção de conteúdo musical para monetização no YouTube.

## Visão Geral

O **AI Ambient Music Generator** combina múltiplas APIs de IA generativa para criar um fluxo completo de produção de conteúdo musical:

- **Geração de Música**: Mubert API
- **Criação de Capas**: OpenAI DALL·E 3
- **Renderização de Vídeo**: FFmpeg
- **Upload Automático**: YouTube Data API v3

### Principais Features

- **Sistema single-user** com autenticação JWT
- Geração de música ambiente via prompt textual (Mubert API 3.0)
- Criação automática de capas artísticas com IA (DALL·E 3)
- Renderização de vídeos Full HD (1080p) com fade-in/fade-out (FFmpeg)
- Histórico completo de faixas geradas com filtros
- Dashboard com estatísticas de uso e storage
- **Geração em lote** (até 10 tracks por vez)
- Upload direto para YouTube com OAuth 2.0
- Interface minimalista dark theme (shadcn/ui + TailwindCSS)

## Stack Tecnológica

### Backend
- **Framework**: FastAPI (Python 3.11+)
- **Database**: PostgreSQL 15
- **ORM**: SQLAlchemy
- **Autenticação**: JWT (JSON Web Tokens) com bcrypt
- **APIs**: Mubert, OpenAI, YouTube Data API v3
- **Processamento**: FFmpeg para renderização de vídeo

### Frontend
- **Framework**: Next.js 16.0.1 (App Router)
- **Runtime**: Node.js 20 LTS
- **Language**: TypeScript 5.x
- **UI**: shadcn/ui + TailwindCSS 3.x
- **Animações**: Framer Motion
- **HTTP Client**: Axios com interceptors (auto token refresh)

### Infraestrutura
- **Containerização**: Docker 24.x + Docker Compose 2.x
- **Banco de Dados**: PostgreSQL 15
- **Storage**: Flat file system (UUID-based)
- **Deploy**: Local (desenvolvimento) → AWS EC2 (produção futura)
- **SSL**: Let's Encrypt via Nginx (produção)

## Arquitetura

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│   Next.js   │─────▶│   FastAPI    │─────▶│ PostgreSQL  │
│  Frontend   │      │   Backend    │      │  Database   │
└─────────────┘      └──────────────┘      └─────────────┘
                            │
                ┌───────────┼───────────┐
                ▼           ▼           ▼
           ┌────────┐  ┌────────┐  ┌────────┐
           │ Mubert │  │ DALL·E │  │FFmpeg  │
           │  API   │  │  API   │  │ Render │
           └────────┘  └────────┘  └────────┘
```

## Pré-requisitos

- **Docker** e **Docker Compose** 20.10+
- **Git**
- **Chaves de API**:
  - OpenAI API Key (para DALL·E 3)
  - Mubert API Key (para geração de música)
  - YouTube API Credentials (opcional, para upload)

## Instalação Rápida

### 1. Clone o repositório

```bash
git clone https://github.com/yourusername/ai-ambient-music-generator.git
cd ai-ambient-music-generator
```

### 2. Configure as variáveis de ambiente

```bash
cp .env.example .env
```

Edite o arquivo `.env` e adicione suas credenciais:

```bash
# API Keys
OPENAI_API_KEY=sk-your-openai-key
MUBERT_API_KEY=your-mubert-key

# JWT
JWT_SECRET_KEY=your-secret-key-min-32-chars

# Database
POSTGRES_DB=ai_music_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your-secure-password
```

### 3. Inicie os containers

```bash
docker-compose up -d --build
```

### 4. Execute as migrações do banco de dados

```bash
docker-compose exec backend alembic upgrade head
```

### 5. Acesse a aplicação

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **API Docs (Swagger)**: http://localhost:8000/docs

## Estrutura do Projeto

```
ai-ambient-music-generator/
├── backend/                    # FastAPI backend
│   ├── routers/               # Endpoints da API
│   ├── services/              # Lógica de negócio
│   ├── models/                # Modelos SQLAlchemy
│   ├── schemas/               # Schemas Pydantic
│   ├── middleware/            # Autenticação, CORS, etc.
│   ├── utils/                 # Funções auxiliares
│   └── database/              # Configuração DB + migrations
│
├── frontend/                  # Next.js frontend
│   ├── app/                   # App Router (Next.js 15)
│   │   ├── (auth)/           # Rotas públicas (login/register)
│   │   └── (protected)/      # Rotas protegidas
│   ├── components/            # Componentes React
│   ├── hooks/                 # React hooks customizados
│   ├── lib/                   # Utilitários e API client
│   └── contexts/              # React contexts
│
├── storage/                   # Armazenamento de arquivos
│   └── users/{user_id}/
│       ├── music/
│       ├── images/
│       └── videos/
│
├── nginx/                     # Configuração Nginx
├── docs/                      # Documentação
└── docker-compose.yml         # Orquestração Docker
```

## API Endpoints

### Autenticação

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/api/auth/register` | Registrar novo usuário |
| POST | `/api/auth/login` | Fazer login |
| POST | `/api/auth/refresh` | Renovar access token |
| POST | `/api/auth/logout` | Fazer logout |
| GET | `/api/auth/me` | Obter dados do usuário |

### Geração de Conteúdo

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/api/generate` | Gerar música + imagem |
| POST | `/api/render` | Renderizar vídeo |
| POST | `/api/upload` | Upload para YouTube |
| GET | `/api/history` | Listar faixas do usuário |
| GET | `/api/tracks/{id}` | Obter detalhes da faixa |
| DELETE | `/api/tracks/{id}` | Deletar faixa |
| GET | `/api/quota` | Consultar quota diária |

## Especificações Técnicas

### Áudio
- **Formato**: MP3, 320 kbps, 44.1 kHz, Stereo
- **Duração**: 5-10 minutos (configurável entre 2-15 min)

### Imagem
- **Formato**: PNG, 1920x1080 pixels (16:9)
- **Geração**: DALL·E 3 via OpenAI API

### Vídeo
- **Formato**: MP4, H.264 codec
- **Resolução**: 1080p (Full HD)
- **Frame Rate**: 30 fps
- **Efeitos**: Fade-in/fade-out automático (2s)

## Sistema de Quotas

Para controlar custos de API e uso de recursos:

- **Limite padrão**: 10 gerações por dia por usuário
- **Reset automático**: Meia-noite UTC
- **Configurável**: Ajustável por variável de ambiente

## Deploy em Produção

### AWS EC2

1. **Provisionar instância EC2**
   - Tipo recomendado: t3.medium (2 vCPUs, 4GB RAM)
   - OS: Ubuntu 22.04 LTS
   - Storage: 50GB+ SSD

2. **Instalar Docker**
   ```bash
   sudo apt update
   sudo apt install -y docker.io docker-compose
   ```

3. **Clonar e configurar**
   ```bash
   git clone <repo-url>
   cd ai-ambient-music-generator
   cp .env.example .env
   # Editar .env com credenciais de produção
   ```

4. **Executar containers**
   ```bash
   docker-compose -f docker-compose.prod.yml up -d --build
   ```

5. **Configurar SSL**
   ```bash
   sudo certbot --nginx -d yourdomain.com
   ```

Para instruções detalhadas, consulte a [documentação completa](docs/product-requirements-document.md).

## Desenvolvimento

### Executar testes

```bash
# Backend
cd backend
pytest

# Frontend
cd frontend
npm test
```

### Comandos úteis

```bash
# Ver logs dos containers
docker-compose logs -f

# Acessar shell do backend
docker-compose exec backend bash

# Criar nova migration
docker-compose exec backend alembic revision --autogenerate -m "description"

# Aplicar migrations
docker-compose exec backend alembic upgrade head
```

## Segurança

- Autenticação JWT com tokens de curta duração (15 min)
- Refresh tokens em HttpOnly cookies
- Passwords hasheados com bcrypt (salt rounds = 12)
- Rate limiting em endpoints críticos
- CORS configurado para origins específicos
- Validação e sanitização de todos os inputs
- Proteção contra SQL Injection, XSS e Path Traversal
- HTTPS obrigatório em produção

## Tratamento de Erros

Sistema robusto de tratamento de erros com:

- Retry logic com exponential backoff para APIs externas
- Timeouts configuráveis (Mubert: 60s, FFmpeg: 300s)
- Mensagens de erro amigáveis ao usuário
- Logging estruturado de todas as operações
- Alertas em caso de múltiplas falhas

## Licenciamento

- **Música**: Royalty-free via Mubert License
- **Imagens**: Geradas via DALL·E com direitos de uso comercial
- **Código**: MIT License (ver [LICENSE](LICENSE))

## Roadmap

- [ ] Geração em lote (playlists automáticas)
- [ ] Customização avançada de parâmetros (BPM, gênero, mood)
- [ ] Sistema de tags e curadoria de faixas
- [ ] Dashboard de estatísticas do YouTube
- [ ] Integração com Spotify e outras plataformas
- [ ] API pública para terceiros
- [ ] Mobile app (React Native)

## Documentação

### 📐 Arquitetura
- **[System Architecture](docs/architecture/system-architecture.md)** - Documentação técnica completa (~150 páginas)
  - Diagramas de componentes, fluxos e dados
  - Stack tecnológico detalhado
  - Modelagem de dados (ERD)
  - API Architecture (REST endpoints)
  - Security architecture
  - Decisões arquiteturais (ADRs)
  - Roadmap de implementação (5 fases)

- **[Project Structure](docs/architecture/project-structure.md)** - Estrutura de diretórios e convenções
- **[Architecture README](docs/architecture/README.md)** - Visão geral e quick reference

### 📋 Produto
- **[Product Requirements Document (PRD)](specs/product/prd.md)** - Requisitos de produto
- **[Phase 1: Music + Image Generation](.claude/sessions/main-features/issues/fase-1-geracao-musica-imagem.md)**
- **[Phase 2: Video Rendering](.claude/sessions/main-features/issues/fase-2-renderizacao-video.md)**
- **[Phase 3: Dashboard + History](.claude/sessions/main-features/issues/fase-3-dashboard-historico.md)**
- **[Phase 4: YouTube Upload](.claude/sessions/main-features/issues/fase-4-upload-youtube.md)**
- **[Phase 5: Batch Generation](.claude/sessions/main-features/issues/fase-5-geracao-em-lote.md)**

### 🚀 Início Rápido
Para documentação completa de setup, consulte [System Architecture - Deployment](docs/architecture/system-architecture.md#9-deployment-architecture).

## Contribuindo

Contribuições são bem-vindas! Por favor:

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'feat: Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

## Autor

**Gabriel Silveira de Souza**
Software Developer / Music Composer
[GitHub](https://github.com/yourusername)

## Suporte

Para reportar bugs ou solicitar features, abra uma [issue](https://github.com/yourusername/ai-ambient-music-generator/issues).

---

© 2025 Gabriel Silveira de Souza. All Rights Reserved.
