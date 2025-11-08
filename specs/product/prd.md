# 🎧 AI Ambient Music Generator — Product Requirements Document (PRD)

## 🧩 Visão Geral

O **AI Ambient Music Generator** é uma plataforma web que permite criar, visualizar e gerenciar músicas ambiente geradas por Inteligência Artificial.  
O objetivo é automatizar a criação de faixas para relaxamento, meditação e concentração, permitindo o render e upload direto para o YouTube.

O sistema utiliza **Next.js + shadcn/ui** no frontend e **FastAPI (Python)** no backend, com geração de música via **Mubert API**, imagens via **OpenAI DALL·E**, e renderização de vídeo com **FFmpeg**.

---

## 🎯 Objetivos do Produto

- Facilitar a geração e publicação de músicas ambiente com IA.  
- Oferecer uma UI minimalista e intuitiva.  
- Permitir automação do fluxo completo: prompt → música → capa → vídeo → upload.  
- Gerar conteúdo original e livre de direitos autorais para uso comercial (YouTube).

---

## 👤 Público-Alvo

- Criadores de conteúdo de **música relaxante / meditativa**.  
- Desenvolvedores interessados em **IA generativa musical**.  
- Produtores que desejam **automatizar uploads para o YouTube**.

---

## 🔐 Autenticação e Autorização

### Sistema Multi-User
O sistema suporta múltiplos usuários com isolamento completo de dados e recursos.

### Método de Autenticação
- **JWT (JSON Web Tokens)** customizado para autenticação stateless.
- **Access Token**: duração de 15 minutos.
- **Refresh Token**: duração de 7 dias, armazenado em HttpOnly cookie.
- **Password Hashing**: bcrypt com salt rounds = 12.

### Fluxo de Autenticação

1. **Registro (`POST /api/auth/register`)**
   - Input: `{ email, password, username }`
   - Validações: email único, senha mínima 8 caracteres
   - Output: usuário criado + tokens JWT

2. **Login (`POST /api/auth/login`)**
   - Input: `{ email, password }`
   - Output: access_token + refresh_token (cookie)

3. **Refresh Token (`POST /api/auth/refresh`)**
   - Input: refresh_token via cookie
   - Output: novo access_token

4. **Logout (`POST /api/auth/logout`)**
   - Invalida refresh_token e limpa cookies

### Proteção de Rotas
- **Backend**: middleware JWT para validar token em todas as rotas protegidas.
- **Frontend**: HOC/middleware para redirecionar usuários não autenticados.
- Rotas públicas: `/auth/login`, `/auth/register`.
- Rotas protegidas: `/api/generate`, `/api/render`, `/api/upload`, `/api/history`.

---

## 👥 Gerenciamento de Usuários e Quotas

### Sistema de Quotas
Para controlar custos de API e uso de recursos, cada usuário possui limites diários:

- **Gerações diárias**: máximo de 10 músicas por dia (configurável).
- **Reset de quota**: automático à meia-noite UTC.
- **Tracking de uso**: contador armazenado no banco de dados.

### Informações do Usuário
- Cada usuário pode visualizar seu uso atual em dashboard.
- Campos rastreados:
  - Total de músicas geradas (lifetime)
  - Gerações restantes hoje
  - Data de registro
  - Último acesso

### Validações
- Requisições bloqueadas quando quota diária atingida.
- Mensagem de erro amigável: "Limite diário atingido. Tente novamente amanhã."

---

## 🚀 Escopo Funcional

### 1. Geração de Música (Mubert API)
- Input: prompt textual (ex: “Deep sleep ocean ambient with ethereal pads”).
- Output: URL de arquivo `.mp3` gerado via Mubert.
- Configurações: duração, gênero, BPM, tags opcionais.

### 2. Geração de Imagem de Capa (DALL·E 3)
- Input: mesmo prompt do usuário (ou ajustado).
- Output: URL da imagem `.png`.
- Resolução padrão: 1024x1024.

### 3. Renderização de Vídeo (FFmpeg)
- Combina música + imagem em um vídeo `.mp4`.
- Duração igual à da faixa.
- Opção de fade-in / fade-out automático.

### 4. Gerenciamento de Histórico
- Lista de faixas geradas (imagem + player + link de vídeo).
- Armazenamento local (SQLite/PostgreSQL) ou Supabase.
- Dados: prompt, URLs, timestamp.

### 5. Upload para YouTube (opcional)
- Login via OAuth2.
- Campos: título, descrição, tags, privacidade.
- API: YouTube Data API v3.

---

## 📱 Funcionalidades do Frontend

### Página Principal (`/`)
- Exibe formulário com:
  - Textarea para prompt.
  - Botão “Generate Music”.
  - Loader animado (shadcn/ui).
- Após geração:
  - Exibe imagem da capa.
  - Player de áudio.
  - Botão “Render Video”.
  - Botão “Upload to YouTube” (opcional).

### Página de Histórico (`/history`)
- Lista de faixas já geradas.
- Exibe miniatura + áudio + status.
- Opção de “Regerar” ou “Excluir”.

### Layout e UI
- Framework: **Next.js + shadcn/ui + TailwindCSS + Framer Motion**.
- Tema: **dark minimalista** (tons de azul escuro e cinza).
- Componentes:
  - `Card`, `Button`, `Textarea`, `Dialog`, `Progress`, `AudioPlayer`.

---

## ⚙️ Funcionalidades do Backend

### API Endpoints (FastAPI)

#### Endpoints de Autenticação (Públicos)
| Método | Endpoint | Função | Autenticação |
|---------|-----------|--------|--------------|
| `POST` | `/api/auth/register` | Registra novo usuário | Não |
| `POST` | `/api/auth/login` | Realiza login | Não |
| `POST` | `/api/auth/refresh` | Renova access token | Refresh Token |
| `POST` | `/api/auth/logout` | Realiza logout | Bearer Token |
| `GET` | `/api/auth/me` | Retorna dados do usuário logado | Bearer Token |

#### Endpoints de Geração (Protegidos)
| Método | Endpoint | Função | Autenticação |
|---------|-----------|--------|--------------|
| `POST` | `/api/generate` | Gera música e imagem | Bearer Token |
| `POST` | `/api/render` | Renderiza vídeo (FFmpeg) | Bearer Token |
| `POST` | `/api/upload` | Faz upload para YouTube | Bearer Token |
| `GET` | `/api/history` | Lista faixas do usuário | Bearer Token |
| `GET` | `/api/tracks/{track_id}` | Retorna dados de uma faixa | Bearer Token |
| `DELETE` | `/api/tracks/{track_id}` | Deleta uma faixa | Bearer Token |

#### Endpoints de Quota
| Método | Endpoint | Função | Autenticação |
|---------|-----------|--------|--------------|
| `GET` | `/api/quota` | Retorna quota atual do usuário | Bearer Token |

---

### Exemplo de fluxo de requisição

1. Frontend envia:
   ```json
   { "prompt": "Calming space ambient with soft pads and rain" }
   ```
2. Backend:
   - Chama Mubert API → gera música `.mp3`
   - Chama DALL·E API → gera capa `.png`
   - Armazena metadados
   - Retorna URLs para frontend

3. Usuário visualiza prévia → aciona renderização → recebe `.mp4`.

---

## 🧠 Integrações de IA

| Recurso | Serviço | Forma de Integração |
|----------|----------|--------------------|
| Música | Mubert API | REST via `requests` |
| Imagem | OpenAI DALL·E 3 | `openai.images.generate()` |
| Render | FFmpeg | subprocess (`ffmpeg -loop 1 ...`) |
| Upload | YouTube Data API v3 | `googleapiclient.discovery` |

---

## ⚙️ Especificações Técnicas

### Configurações de Áudio
- **Formato**: MP3
- **Bitrate**: 320 kbps (alta qualidade)
- **Sample Rate**: 44.1 kHz
- **Canais**: Stereo
- **Duração padrão**: 5-10 minutos (300-600 segundos)
- **Duração mínima**: 2 minutos (120 segundos)
- **Duração máxima**: 15 minutos (900 segundos)

### Configurações de Imagem
- **Formato**: PNG
- **Resolução**: 1920x1080 pixels (16:9)
- **Qualidade**: Alta (sem compressão agressiva)
- **Color Space**: sRGB
- **DPI**: 72 (padrão web)

### Configurações de Vídeo
- **Formato**: MP4
- **Codec de vídeo**: H.264 (libx264)
- **Codec de áudio**: AAC
- **Resolução**: 1920x1080 (Full HD 1080p)
- **Frame Rate**: 30 fps
- **Bitrate vídeo**: 5000k
- **Bitrate áudio**: 320k
- **Preset FFmpeg**: `medium` (balanço entre velocidade e qualidade)
- **Efeitos**: fade-in (2s) e fade-out (2s) no áudio

### Comando FFmpeg de Referência
```bash
ffmpeg -loop 1 -i cover.png -i audio.mp3 \
  -c:v libx264 -tune stillimage -c:a aac \
  -b:a 320k -pix_fmt yuv420p -shortest \
  -vf "scale=1920:1080" -r 30 \
  -preset medium -b:v 5000k \
  -af "afade=t=in:st=0:d=2,afade=t=out:st=${duration-2}:d=2" \
  output.mp4
```

### Timeouts e Performance
- **Timeout Mubert API**: 60 segundos
- **Timeout DALL·E API**: 60 segundos
- **Timeout FFmpeg render**: 300 segundos (5 minutos)
- **Tempo estimado de geração completa**: 60-90 segundos
- **Processamento**: Síncrono com feedback de loading para o usuário

---

## 💾 Armazenamento

### Estratégia de Armazenamento
O sistema utiliza **armazenamento local em sistema de arquivos** organizado por usuário.

### Estrutura de Diretórios
```
storage/
├── users/
│   ├── {user_id}/
│   │   ├── music/
│   │   │   └── {track_id}.mp3
│   │   ├── images/
│   │   │   └── {track_id}.png
│   │   └── videos/
│   │       └── {track_id}.mp4
└── temp/
    └── {job_id}/
```

### Detalhes
- **Isolamento por usuário**: cada usuário tem seu próprio diretório identificado por `user_id`.
- **Organização por tipo**: arquivos separados em `music/`, `images/`, `videos/`.
- **Nomenclatura**: todos os arquivos usam `track_id` (UUID) como nome.
- **Diretório temporário**: processos de renderização usam `/temp/{job_id}/` antes de mover para local definitivo.

### Políticas de Retenção
- **Arquivos permanentes**: mantidos indefinidamente até exclusão manual pelo usuário.
- **Arquivos temporários**: limpeza automática após 24h se não movidos para diretório permanente.
- **Limite de armazenamento**: pode ser implementado posteriormente (ex: 1GB por usuário).

### Estrutura de Dados (PostgreSQL)

#### Tabela: `users`
| Campo | Tipo | Descrição |
|--------|------|------------|
| id | UUID | Identificador único do usuário (PK) |
| email | VARCHAR(255) | Email único do usuário |
| username | VARCHAR(100) | Nome de usuário |
| password_hash | VARCHAR(255) | Hash bcrypt da senha |
| created_at | TIMESTAMP | Data de criação da conta |
| last_login | TIMESTAMP | Último login |
| is_active | BOOLEAN | Conta ativa/inativa |

#### Tabela: `tracks`
| Campo | Tipo | Descrição |
|--------|------|------------|
| id | UUID | Identificador único da faixa (PK) |
| user_id | UUID | Referência ao usuário (FK) |
| prompt | TEXT | Texto descritivo usado na geração |
| music_path | VARCHAR(500) | Caminho local do arquivo MP3 |
| image_path | VARCHAR(500) | Caminho local da imagem PNG |
| video_path | VARCHAR(500) | Caminho local do vídeo MP4 (nullable) |
| duration | INTEGER | Duração em segundos |
| created_at | TIMESTAMP | Data/hora de criação |
| status | VARCHAR(50) | Status: 'generated', 'rendered', 'uploaded' |
| youtube_url | VARCHAR(500) | URL do vídeo no YouTube (nullable) |

#### Tabela: `user_quotas`
| Campo | Tipo | Descrição |
|--------|------|------------|
| id | UUID | Identificador único (PK) |
| user_id | UUID | Referência ao usuário (FK) |
| date | DATE | Data da quota (YYYY-MM-DD) |
| generations_used | INTEGER | Número de gerações usadas no dia |
| daily_limit | INTEGER | Limite diário configurado (padrão: 10) |
| created_at | TIMESTAMP | Data de criação do registro |

#### Tabela: `refresh_tokens`
| Campo | Tipo | Descrição |
|--------|------|------------|
| id | UUID | Identificador único (PK) |
| user_id | UUID | Referência ao usuário (FK) |
| token | VARCHAR(500) | Hash do refresh token |
| expires_at | TIMESTAMP | Data de expiração |
| created_at | TIMESTAMP | Data de criação |
| revoked | BOOLEAN | Token revogado? |

### Relacionamentos
- `tracks.user_id` → `users.id` (Many-to-One)
- `user_quotas.user_id` → `users.id` (Many-to-One)
- `refresh_tokens.user_id` → `users.id` (Many-to-One)

---

## 🧱 Estrutura do Projeto

```
ai-ambient-music-generator/
├── backend/
│   ├── main.py                          # Entry point FastAPI
│   ├── config.py                        # Configurações e variáveis de ambiente
│   ├── dependencies.py                  # Dependency injection (DB, Auth)
│   │
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── auth.py                      # Registro, login, refresh, logout
│   │   ├── generate.py                  # Geração de música + imagem
│   │   ├── render.py                    # Renderização de vídeo
│   │   ├── upload.py                    # Upload para YouTube
│   │   ├── tracks.py                    # CRUD de tracks
│   │   └── quota.py                     # Consulta de quota do usuário
│   │
│   ├── services/
│   │   ├── __init__.py
│   │   ├── mubert_service.py            # Integração Mubert API
│   │   ├── dalle_service.py             # Integração DALL·E (OpenAI)
│   │   ├── ffmpeg_service.py            # Renderização de vídeo
│   │   ├── youtube_service.py           # Upload YouTube
│   │   ├── storage_service.py           # Gerenciamento de arquivos
│   │   └── quota_service.py             # Lógica de quotas
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py                      # Modelo User (SQLAlchemy)
│   │   ├── track.py                     # Modelo Track
│   │   ├── user_quota.py                # Modelo UserQuota
│   │   └── refresh_token.py             # Modelo RefreshToken
│   │
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py                      # Pydantic schemas (User, UserCreate, UserResponse)
│   │   ├── track.py                     # Pydantic schemas (Track, TrackCreate)
│   │   ├── auth.py                      # Schemas de login/registro
│   │   └── quota.py                     # Schema de quota
│   │
│   ├── middleware/
│   │   ├── __init__.py
│   │   ├── auth_middleware.py           # Middleware JWT
│   │   ├── cors_middleware.py           # Configuração CORS
│   │   └── rate_limit_middleware.py     # Rate limiting
│   │
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── jwt_handler.py               # Criação e validação de JWT
│   │   ├── password.py                  # Hash e verificação de senha
│   │   ├── validators.py                # Validadores customizados
│   │   └── logger.py                    # Configuração de logging
│   │
│   ├── database/
│   │   ├── __init__.py
│   │   ├── db.py                        # Configuração SQLAlchemy
│   │   └── migrations/                  # Alembic migrations
│   │       └── versions/
│   │
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── test_auth.py
│   │   ├── test_generate.py
│   │   └── test_quota.py
│   │
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
│
├── frontend/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── register/
│   │   │       └── page.tsx
│   │   ├── (protected)/
│   │   │   ├── layout.tsx               # Layout com AuthGuard
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx             # Dashboard principal
│   │   │   ├── generate/
│   │   │   │   └── page.tsx             # Geração de música
│   │   │   └── history/
│   │   │       └── page.tsx             # Histórico de faixas
│   │   ├── layout.tsx
│   │   └── page.tsx                     # Landing page
│   │
│   ├── components/
│   │   ├── ui/                          # shadcn/ui components
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── input.tsx
│   │   │   └── ...
│   │   ├── auth/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── AuthGuard.tsx
│   │   ├── generate/
│   │   │   ├── PromptForm.tsx
│   │   │   ├── GenerationLoader.tsx
│   │   │   └── ResultPreview.tsx
│   │   ├── tracks/
│   │   │   ├── TrackCard.tsx
│   │   │   ├── TrackList.tsx
│   │   │   └── AudioPlayer.tsx
│   │   ├── video/
│   │   │   └── VideoPreview.tsx
│   │   └── layout/
│   │       ├── Navbar.tsx
│   │       ├── Sidebar.tsx
│   │       └── QuotaDisplay.tsx
│   │
│   ├── lib/
│   │   ├── api.ts                       # Axios client configurado
│   │   ├── auth.ts                      # Funções de autenticação
│   │   └── utils.ts                     # Utilidades
│   │
│   ├── hooks/
│   │   ├── useAuth.ts                   # Hook de autenticação
│   │   ├── useGenerate.ts               # Hook de geração
│   │   └── useQuota.ts                  # Hook de quota
│   │
│   ├── contexts/
│   │   └── AuthContext.tsx              # Context de autenticação
│   │
│   ├── types/
│   │   ├── user.ts
│   │   ├── track.ts
│   │   └── api.ts
│   │
│   ├── package.json
│   ├── next.config.js
│   ├── tailwind.config.ts
│   ├── tsconfig.json
│   ├── Dockerfile
│   └── .env.local.example
│
├── storage/                             # Diretório de armazenamento (git ignored)
│   ├── users/
│   │   └── {user_id}/
│   │       ├── music/
│   │       ├── images/
│   │       └── videos/
│   └── temp/
│
├── nginx/
│   ├── nginx.conf                       # Configuração Nginx
│   └── ssl/                             # Certificados SSL
│
├── docs/
│   ├── product-requirements-document.md # Este arquivo
│   ├── api-documentation.md             # Documentação da API
│   └── setup-guide.md                   # Guia de configuração
│
├── .github/
│   └── workflows/
│       └── ci-cd.yml                    # GitHub Actions
│
├── docker-compose.yml
├── docker-compose.prod.yml
├── .env.example
├── .gitignore
├── README.md
└── LICENSE
```

---

## 🧰 Tecnologias e Bibliotecas

### Frontend
- `next@15`
- `tailwindcss`
- `shadcn/ui`
- `framer-motion`
- `axios`
- `react-icons`

### Backend
- `fastapi`
- `httpx` / `requests`
- `pydantic`
- `openai`
- `ffmpeg-python`
- `sqlalchemy`
- `google-api-python-client`

---

## 🛠️ Setup e Desenvolvimento Local

### Pré-requisitos
- **Git**: controle de versão
- **Docker** e **Docker Compose**: 20.10+
- **Python**: 3.11+ (se rodar backend sem Docker)
- **Node.js**: 18+ e npm/yarn (se rodar frontend sem Docker)
- **FFmpeg**: instalado no sistema (se rodar sem Docker)

### Instalação - Opção 1: Docker (Recomendado)

#### 1. Clone o repositório
```bash
git clone https://github.com/yourusername/ai-ambient-music-generator.git
cd ai-ambient-music-generator
```

#### 2. Configure variáveis de ambiente
```bash
cp .env.example .env
# Edite o arquivo .env com suas credenciais
nano .env
```

**Variáveis obrigatórias para desenvolvimento:**
```bash
# Database
POSTGRES_DB=ai_music_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=localdev123

# JWT
JWT_SECRET_KEY=dev_secret_key_minimum_32_characters_long_please

# API Keys
OPENAI_API_KEY=sk-your-openai-key
MUBERT_API_KEY=your-mubert-key  # ou mock para desenvolvimento
```

#### 3. Inicialize os containers
```bash
docker-compose up -d --build
```

#### 4. Execute migrações do banco
```bash
docker-compose exec backend alembic upgrade head
```

#### 5. Acesse a aplicação
- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs (Swagger UI)

### Instalação - Opção 2: Manual (Sem Docker)

#### Backend

```bash
cd backend

# Criar ambiente virtual
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Instalar dependências
pip install -r requirements.txt

# Configurar .env
cp .env.example .env
# Editar com suas credenciais

# Executar migrações
alembic upgrade head

# Rodar servidor
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

#### Frontend

```bash
cd frontend

# Instalar dependências
npm install

# Configurar variáveis de ambiente
cp .env.local.example .env.local
# Editar NEXT_PUBLIC_API_URL=http://localhost:8000/api

# Rodar em desenvolvimento
npm run dev
```

#### PostgreSQL Local
```bash
# Instalar PostgreSQL
sudo apt install postgresql postgresql-contrib  # Ubuntu/Debian
brew install postgresql@15                      # macOS

# Criar banco de dados
createdb ai_music_db
```

### Desenvolvimento

#### Comandos Úteis

**Backend:**
```bash
# Rodar testes
pytest

# Criar nova migration
alembic revision --autogenerate -m "description"

# Aplicar migrations
alembic upgrade head

# Reverter migration
alembic downgrade -1

# Linting e formatação
black .
flake8 .
```

**Frontend:**
```bash
# Rodar testes
npm test

# Build de produção
npm run build

# Linting
npm run lint

# Type checking
npm run type-check
```

**Docker:**
```bash
# Ver logs
docker-compose logs -f

# Rebuild após mudanças
docker-compose up -d --build

# Parar containers
docker-compose down

# Limpar volumes (cuidado!)
docker-compose down -v

# Acessar shell do container
docker-compose exec backend bash
docker-compose exec frontend sh
```

### Estrutura de Branches

```
main            # Branch principal (produção)
├── develop     # Branch de desenvolvimento
│   ├── feature/user-auth
│   ├── feature/music-generation
│   └── feature/video-rendering
└── hotfix/     # Correções urgentes
```

### Workflow de Desenvolvimento

1. **Criar branch a partir de develop**
   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b feature/nome-da-feature
   ```

2. **Desenvolver e testar localmente**
   ```bash
   # Fazer commits incrementais
   git add .
   git commit -m "feat: adiciona endpoint de autenticação"
   ```

3. **Push e criar Pull Request**
   ```bash
   git push origin feature/nome-da-feature
   # Abrir PR no GitHub para develop
   ```

4. **Code Review e Merge**
   - Aguardar aprovação
   - Merge para develop
   - Deploy automático em staging (se configurado)

### Troubleshooting

#### Problema: FFmpeg não encontrado
```bash
# Ubuntu/Debian
sudo apt install ffmpeg

# macOS
brew install ffmpeg

# Verificar instalação
ffmpeg -version
```

#### Problema: Erro de conexão com banco de dados
```bash
# Verificar se PostgreSQL está rodando
docker-compose ps
# ou
sudo systemctl status postgresql

# Verificar credenciais no .env
cat .env | grep POSTGRES
```

#### Problema: Porta já em uso
```bash
# Verificar processos na porta 8000
lsof -i :8000

# Matar processo
kill -9 <PID>

# Ou alterar porta no docker-compose.yml
```

#### Problema: Erro de CORS
- Verificar `CORS_ORIGINS` no `.env` do backend
- Adicionar `http://localhost:3000` se necessário

### Ferramentas Recomendadas

- **IDE**: VSCode com extensões Python e TypeScript
- **API Testing**: Postman ou Insomnia
- **Database Client**: DBeaver ou pgAdmin
- **Git GUI**: GitKraken ou GitHub Desktop (opcional)

---

## 🐳 Deploy

### Ambiente de Produção: AWS EC2 com Docker

#### Arquitetura
- **Servidor**: AWS EC2 (recomendado: t3.medium ou superior)
- **OS**: Ubuntu 22.04 LTS
- **Orquestração**: Docker Compose
- **Reverse Proxy**: Nginx
- **SSL**: Let's Encrypt (Certbot)
- **Banco de Dados**: PostgreSQL em container Docker com volume persistente

#### Requisitos de Infraestrutura
- **CPU**: 2+ vCPUs (para processamento FFmpeg)
- **RAM**: 4GB+ (8GB recomendado para múltiplas gerações simultâneas)
- **Storage**: 50GB+ SSD (para armazenar músicas, imagens e vídeos)
- **Security Group**:
  - Porta 22 (SSH)
  - Porta 80 (HTTP)
  - Porta 443 (HTTPS)

### Docker Compose (Production)
```yaml
version: "3.8"

services:
  postgres:
    image: postgres:15-alpine
    container_name: ai-music-db
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network
    restart: unless-stopped

  backend:
    build: ./backend
    container_name: ai-music-backend
    ports:
      - "8000:8000"
    env_file:
      - .env
    volumes:
      - ./storage:/app/storage
    depends_on:
      - postgres
    networks:
      - app-network
    restart: unless-stopped

  frontend:
    build: ./frontend
    container_name: ai-music-frontend
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_URL: ${NEXT_PUBLIC_API_URL}
    depends_on:
      - backend
    networks:
      - app-network
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    container_name: ai-music-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    networks:
      - app-network
    restart: unless-stopped

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

### Configuração Nginx (Reverse Proxy)
```nginx
upstream backend {
    server backend:8000;
}

upstream frontend {
    server frontend:3000;
}

server {
    listen 80;
    server_name yourdomain.com;

    location /api {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        proxy_pass http://frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Passos de Deploy

1. **Configurar EC2 Instance**
   ```bash
   # Instalar Docker e Docker Compose
   sudo apt update
   sudo apt install -y docker.io docker-compose
   sudo usermod -aG docker $USER
   ```

2. **Clonar repositório e configurar**
   ```bash
   git clone <repository-url>
   cd ai-ambient-music-generator
   cp .env.example .env
   # Editar .env com credenciais reais
   ```

3. **Build e executar containers**
   ```bash
   docker-compose up -d --build
   ```

4. **Configurar SSL (Let's Encrypt)**
   ```bash
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx -d yourdomain.com
   ```

5. **Verificar status**
   ```bash
   docker-compose ps
   docker-compose logs -f
   ```

---

## 🔐 Variáveis de Ambiente

### Arquivo `.env` (Backend)

```bash
# === Database ===
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=ai_music_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_secure_password_here

# === JWT Configuration ===
JWT_SECRET_KEY=your_super_secret_jwt_key_min_32_chars
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=15
JWT_REFRESH_TOKEN_EXPIRE_DAYS=7

# === API Keys ===
MUBERT_API_KEY=your_mubert_api_key_here
MUBERT_API_LICENSE=your_mubert_license_key
OPENAI_API_KEY=sk-your-openai-api-key-here

# === YouTube API ===
YOUTUBE_CLIENT_ID=your_google_oauth_client_id
YOUTUBE_CLIENT_SECRET=your_google_oauth_client_secret
YOUTUBE_REDIRECT_URI=http://localhost:8000/api/auth/youtube/callback

# === Storage ===
STORAGE_PATH=/app/storage
TEMP_PATH=/app/storage/temp
MAX_STORAGE_PER_USER_GB=5

# === Quotas ===
DEFAULT_DAILY_GENERATION_LIMIT=10

# === Application ===
BACKEND_HOST=0.0.0.0
BACKEND_PORT=8000
CORS_ORIGINS=http://localhost:3000,https://yourdomain.com
ENVIRONMENT=production

# === FFmpeg ===
FFMPEG_TIMEOUT_SECONDS=300
```

### Arquivo `.env.local` (Frontend)

```bash
# === API Configuration ===
NEXT_PUBLIC_API_URL=http://localhost:8000/api

# === Environment ===
NODE_ENV=development
```

### Arquivo `.env.example`
Criar um arquivo `.env.example` no repositório com valores placeholder:

```bash
# Database
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=ai_music_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=change_me

# JWT
JWT_SECRET_KEY=change_me_min_32_characters_long
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=15
JWT_REFRESH_TOKEN_EXPIRE_DAYS=7

# API Keys
MUBERT_API_KEY=your_mubert_api_key
MUBERT_API_LICENSE=your_mubert_license
OPENAI_API_KEY=your_openai_api_key

# Storage
STORAGE_PATH=/app/storage
TEMP_PATH=/app/storage/temp
MAX_STORAGE_PER_USER_GB=5

# Quotas
DEFAULT_DAILY_GENERATION_LIMIT=10

# Application
BACKEND_HOST=0.0.0.0
BACKEND_PORT=8000
CORS_ORIGINS=http://localhost:3000
ENVIRONMENT=development
```

### Segurança de Variáveis de Ambiente
- **NUNCA** commitar arquivo `.env` no Git (adicionar ao `.gitignore`)
- Usar ferramentas como `python-dotenv` para carregar variáveis
- Em produção, considerar usar AWS Secrets Manager ou similar
- Rotacionar chaves JWT periodicamente

---

## ⚠️ Tratamento de Erros e Validações

### Validações de Input

#### Registro de Usuário
- **Email**: formato válido, único no sistema
- **Password**: mínimo 8 caracteres, máximo 100 caracteres
- **Username**: 3-50 caracteres, alfanumérico + underscore

#### Geração de Música
- **Prompt**:
  - Mínimo: 10 caracteres
  - Máximo: 500 caracteres
  - Sanitização: remover HTML/scripts
- **Duração**: entre 120-900 segundos
- **Quota**: verificar limite diário não atingido

### Tratamento de Erros por Serviço

#### Erros de API Externa

**Mubert API**
- **Timeout**: 60s → Erro 504 Gateway Timeout
- **Rate Limit**: Retry após 1 minuto com exponential backoff
- **API Key inválida**: Erro 500 com log, não expor detalhes ao usuário
- **Response**:
  ```json
  {
    "error": "Falha ao gerar música. Tente novamente em alguns instantes.",
    "code": "MUSIC_GENERATION_FAILED"
  }
  ```

**OpenAI DALL·E**
- **Timeout**: 60s → Erro 504
- **Content Policy Violation**: Retornar erro amigável sugerindo modificar prompt
- **Rate Limit**: Retry com backoff
- **Response**:
  ```json
  {
    "error": "Não foi possível gerar a imagem. Por favor, reformule seu prompt.",
    "code": "IMAGE_GENERATION_FAILED",
    "suggestion": "Evite termos que possam violar políticas de conteúdo"
  }
  ```

**FFmpeg Rendering**
- **Timeout**: 300s → Erro 504
- **Arquivo inválido**: Verificar integridade antes de renderizar
- **Falta de espaço**: Verificar espaço em disco antes de iniciar
- **Response**:
  ```json
  {
    "error": "Falha ao renderizar vídeo. Tente novamente.",
    "code": "VIDEO_RENDERING_FAILED"
  }
  ```

#### Erros de Autenticação
- **401 Unauthorized**: Token inválido ou expirado
- **403 Forbidden**: Permissão negada para recurso
- **Response padrão**:
  ```json
  {
    "error": "Autenticação necessária. Faça login novamente.",
    "code": "UNAUTHORIZED"
  }
  ```

#### Erros de Quota
- **429 Too Many Requests**: Limite diário atingido
- **Response**:
  ```json
  {
    "error": "Limite diário de gerações atingido.",
    "code": "QUOTA_EXCEEDED",
    "reset_time": "2025-10-23T00:00:00Z",
    "current_usage": 10,
    "limit": 10
  }
  ```

### Retry Logic

#### Configuração de Retry
```python
# Retry para APIs externas
MAX_RETRIES = 3
RETRY_DELAY = [1, 2, 4]  # segundos (exponential backoff)
RETRY_ON = [408, 429, 500, 502, 503, 504]  # HTTP status codes
```

#### Operações que NÃO devem ter retry automático
- Validações de input (400 Bad Request)
- Erros de autenticação (401, 403)
- Recursos não encontrados (404)
- Violação de políticas de conteúdo

### Logging

#### Níveis de Log
- **INFO**: Requisições bem-sucedidas, início/fim de processos
- **WARNING**: Retry de operação, quota próxima do limite
- **ERROR**: Falha em API externa, erro de renderização
- **CRITICAL**: Falha de banco de dados, storage inacessível

#### Formato de Log
```python
{
  "timestamp": "2025-10-22T15:30:00Z",
  "level": "ERROR",
  "service": "mubert_service",
  "user_id": "uuid-here",
  "track_id": "uuid-here",
  "message": "Mubert API timeout after 60s",
  "stack_trace": "..."
}
```

### Mensagens de Erro Amigáveis (Frontend)

| Código de Erro | Mensagem ao Usuário |
|----------------|---------------------|
| `MUSIC_GENERATION_FAILED` | "Não conseguimos gerar sua música. Por favor, tente novamente." |
| `IMAGE_GENERATION_FAILED` | "Erro ao criar a imagem. Tente modificar seu prompt." |
| `VIDEO_RENDERING_FAILED` | "Falha ao criar o vídeo. Nossa equipe foi notificada." |
| `QUOTA_EXCEEDED` | "Você atingiu o limite de gerações hoje. Volte amanhã!" |
| `UNAUTHORIZED` | "Sessão expirada. Por favor, faça login novamente." |
| `INVALID_INPUT` | "Verifique os dados informados e tente novamente." |

---

## 🔒 Requisitos Não-Funcionais

### Performance
- **Resposta inicial**: < 5 segundos para geração de música/imagem
- **Renderização de vídeo**: < 60 segundos para faixas de 5-10 minutos
- **Latência API**: < 200ms para endpoints de autenticação
- **Throughput**: suportar 10+ usuários simultâneos

### Experiência do Usuário (UX)
- **Interface**: fluida, minimalista e responsiva (mobile-first)
- **Loading states**: feedback visual em todas as operações longas
- **Acessibilidade**: seguir WCAG 2.1 Level AA
- **Internacionalização**: preparado para i18n (português como padrão)

### Licenciamento
- **Música**: apenas faixas royalty-free (Mubert License)
- **Imagens**: geradas via DALL·E com direitos de uso comercial
- **Compliance**: respeitar termos de uso de todas as APIs

### Segurança

#### Autenticação e Autorização
- **JWT**: tokens assinados com algoritmo HS256 ou RS256
- **Password hashing**: bcrypt com salt rounds = 12
- **Refresh tokens**: armazenados hasheados no banco, rotação automática
- **Session management**: logout invalida tokens no servidor

#### Proteção de APIs
- **CORS**: configurado para permitir apenas origins autorizados
  ```python
  CORS_ORIGINS = ["https://yourdomain.com", "http://localhost:3000"]
  ```
- **Rate Limiting**: máximo de requisições por IP/usuário
  - Login: 5 tentativas por 15 minutos
  - Registro: 3 tentativas por hora
  - Geração: 10 por dia por usuário (quota)
- **CSRF Protection**: tokens CSRF em formulários sensíveis

#### Validação e Sanitização
- **Input validation**: Pydantic models no backend para validação de schemas
- **SQL Injection**: uso de ORM (SQLAlchemy) com parameterized queries
- **XSS Protection**: sanitização de HTML em prompts do usuário
- **Path Traversal**: validação rigorosa de caminhos de arquivo

#### Proteção de Dados
- **Sensitive data**: nunca logar senhas, tokens ou API keys
- **Environment variables**: nunca commitar .env no repositório
- **HTTPS**: obrigatório em produção (certificado Let's Encrypt)
- **Headers de segurança**:
  ```python
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  X-XSS-Protection: 1; mode=block
  Strict-Transport-Security: max-age=31536000; includeSubDomains
  ```

#### Isolamento de Recursos
- **Multi-tenancy**: isolamento completo de arquivos e dados por usuário
- **Permissions**: usuário só acessa seus próprios recursos
- **File access**: validar user_id antes de servir arquivos

#### Monitoramento e Auditoria
- **Logging**: registrar todas as ações críticas (login, geração, exclusão)
- **Alertas**: notificar em caso de múltiplas tentativas de login falhadas
- **Audit trail**: manter histórico de ações por usuário

#### Backup e Recuperação
- **Database**: backup diário automático do PostgreSQL
- **Arquivos**: considerar backup incremental do storage
- **RTO**: Recovery Time Objective < 4 horas
- **RPO**: Recovery Point Objective < 24 horas

### Manutenção e Extensibilidade
- **Código modular**: seguir princípios SOLID
- **Documentação**: docstrings em todas as funções
- **Testes**: cobertura mínima de 70%
- **CI/CD**: pipeline automatizado para testes e deploy
- **Versionamento**: semantic versioning (semver)

---

## 🪶 Futuras Extensões

- Geração em lote (playlist automática).  
- Customização de parâmetros sonoros (BPM, duração, gênero).  
- Sistema de tags e curadoria de faixas.  
- Dashboard de estatísticas (views, engajamento no YouTube).  
- Integração com Supabase Auth e Storage.

---

## 📅 Roadmap (Fases)

| Fase | Entrega | Descrição |
|------|----------|------------|
| **1. MVP** | Geração música + imagem | Pipeline básico funcionando |
| **2. Renderização** | FFmpeg + preview | Criação do vídeo final |
| **3. Dashboard** | Histórico e status | Armazenamento e listagem |
| **4. Upload** | Integração YouTube | Publicação automatizada |
| **5. Automação** | Lotes + agendamento | Escalabilidade |

---

## 🧑‍💻 Autor
**Gabriel Silveira de Souza**  
Software Developer / Music Composer  
2025 © All Rights Reserved
