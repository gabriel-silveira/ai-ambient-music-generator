## 🎧 AI Ambient Music Generator

O **AI Ambient Music Generator** é uma plataforma web completa projetada para automatizar a criação, renderização e publicação de músicas ambiente (para relaxamento, meditação e concentração) usando Inteligência Artificial.

A ferramenta permite que criadores de conteúdo e produtores automatizem o fluxo completo: `prompt` $\rightarrow$ `música` $\rightarrow$ `capa` $\rightarrow$ `vídeo` $\rightarrow$ `upload para o YouTube`. O objetivo é gerar conteúdo original e livre de direitos autorais para uso comercial.

### ✨ Principais Funcionalidades

* **Geração de Música com IA:** Cria faixas ambientais a partir de um prompt textual usando a **Mubert API**.
* **Geração de Capas:** Cria a imagem de capa do vídeo usando o mesmo prompt através do **OpenAI DALL·E 3**.
* **Renderização de Vídeo:** Combina a música e a imagem estática em um arquivo `.mp4` pronto para upload, utilizando **FFmpeg**.
* **Upload Automatizado:** Publicação opcional e direta para o YouTube via **YouTube Data API v3** com autenticação OAuth2.
* **Histórico e Gerenciamento:** Permite visualizar e gerenciar todas as faixas já geradas.

### 🛠️ Stack Tecnológico

A plataforma é construída com uma arquitetura moderna e modular:

| Componente | Tecnologia | Detalhes |
| :--- | :--- | :--- |
| **Frontend** | **Next.js 15** | Interface de usuário minimalista e intuitiva, construída com **shadcn/ui**, **TailwindCSS** e **Framer Motion**. |
| **Backend** | **FastAPI (Python)** | API de alta performance para orquestrar as integrações de IA e o processamento. |
| **Integrações de IA** | **Mubert API, OpenAI DALL·E 3** | Serviços principais para geração de áudio e imagem. |
| **Processamento** | **FFmpeg** | Utilizado para renderizar o vídeo final (`música + imagem`). |
| **Banco de Dados** | **PostgreSQL/SQLite** | Armazenamento de metadados das faixas geradas. |

### 🎯 Público-Alvo

* Criadores de conteúdo de música relaxante/meditativa.
* Produtores que desejam automatizar uploads para o YouTube.
* Desenvolvedores interessados em IA generativa musical.

### 🛣️ Roadmap (Fases Iniciais)

O desenvolvimento está estruturado nas seguintes fases:

1.  **MVP:** Pipeline básico de Geração de música + imagem.
2.  **Renderização:** Criação do vídeo final (FFmpeg + preview).
3.  **Dashboard:** Histórico de faixas e status de processamento.
4.  **Upload:** Integração e publicação automatizada no YouTube.