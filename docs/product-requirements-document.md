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

| Método | Endpoint | Função |
|---------|-----------|--------|
| `POST` | `/api/generate` | Gera música e imagem |
| `POST` | `/api/render` | Renderiza vídeo (FFmpeg) |
| `POST` | `/api/upload` | Faz upload para YouTube |
| `GET` | `/api/history` | Lista faixas geradas |

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

## 💾 Armazenamento

### Estrutura de Dados (PostgreSQL ou SQLite)

| Campo | Tipo | Descrição |
|--------|------|------------|
| id | UUID | Identificador |
| prompt | TEXT | Texto descritivo |
| music_url | TEXT | URL do áudio gerado |
| image_url | TEXT | URL da capa |
| video_path | TEXT | Caminho local ou S3 |
| created_at | TIMESTAMP | Data/hora |
| status | TEXT | (generated / rendered / uploaded) |

---

## 🧱 Estrutura do Projeto

```
ai-music-generator/
├── backend/
│   ├── main.py
│   ├── routers/
│   │   ├── generate.py
│   │   ├── render.py
│   │   ├── upload.py
│   ├── services/
│   │   ├── mubert_service.py
│   │   ├── dalle_service.py
│   │   ├── ffmpeg_service.py
│   │   └── youtube_service.py
│   ├── models/
│   └── database/
│       └── db.py
│
├── frontend/
│   ├── app/
│   │   ├── page.tsx
│   │   ├── generate/page.tsx
│   │   └── history/page.tsx
│   └── components/
│       ├── ui/ (shadcn)
│       ├── PromptForm.tsx
│       ├── TrackCard.tsx
│       ├── AudioPlayer.tsx
│       ├── VideoPreview.tsx
│       └── Loader.tsx
│
└── docker-compose.yml
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

## 🐳 Deploy

### Docker Compose
```yaml
version: "3.8"

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    env_file:
      - .env
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
```

---

## 🔒 Requisitos Não-Funcionais

- **Performance:** resposta do backend < 5s para geração inicial.  
- **UX:** interface fluida, minimalista e responsiva.  
- **Licenciamento:** apenas faixas royalty-free (Mubert License).  
- **Segurança:** OAuth2 para upload, CORS habilitado no backend.  
- **Manutenção:** código modular e extensível.

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
