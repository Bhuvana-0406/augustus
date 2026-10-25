<div align="center">

# 🎬 Augustus

### AI-Powered YouTube Learning Agent

Chat with any YouTube video. Augustus retrieves answers from the transcript with RAG and decides on its own when to search the web for more.

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![LangChain](https://img.shields.io/badge/LangChain-Agent-1C3C3C?logo=langchain&logoColor=white)](https://www.langchain.com)
[![Pinecone](https://img.shields.io/badge/Pinecone-Vector%20DB-000000?logo=pinecone&logoColor=white)](https://www.pinecone.io)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-NeonDB-4169E1?logo=postgresql&logoColor=white)](https://neon.tech)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[Features](#-features) · [Quick Start](#-quick-start) · [API](#-api-reference) · [Architecture](#-architecture) · [Deployment](#-deployment) · [Roadmap](#-roadmap)

</div>

---

## 📌 Overview

Augustus looks like a simple Q&A tool, but under the hood it is a **LangChain ReAct agent**. For every question it decides whether the video transcript is enough or whether it should also search the web. It remembers the conversation, so follow-up questions just work.

---

## ✨ Features

| | Feature | Details |
|---|---|---|
| 🤖 | **Agentic reasoning** | The agent orchestrates YouTube RAG and falls back to Google Custom Search only when needed |
| 🧠 | **Conversation memory** | Persistent sessions with follow-up question handling |
| 🌐 | **Multi-language** | Handles Hindi and English video transcripts |
| 📚 | **Advanced RAG** | Multi-query retrieval with contextual compression over OpenAI `text-embedding-3-large` embeddings |
| 🗂️ | **Isolated user data** | Every user gets their own Pinecone index |
| 🔐 | **Secure auth** | OAuth2 + JWT with scopes, bcrypt hashing, role-based access |
| ☁️ | **Cloud-native** | NeonDB (serverless PostgreSQL), Alembic migrations, SSL/TLS, optimized connection pooling |

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| **API** | FastAPI, Pydantic, Uvicorn |
| **Agent / LLM** | LangChain (ReAct agent), OpenAI GPT-4o |
| **Retrieval** | Pinecone, OpenAI `text-embedding-3-large` |
| **Web search** | Google Custom Search |
| **Database** | PostgreSQL (NeonDB), SQLAlchemy, Alembic |
| **Auth** | OAuth2, JWT, bcrypt |

---

## 🚀 Quick Start

### Prerequisites

- Python 3.10+
- OpenAI API key
- Pinecone API key
- NeonDB account (or a local PostgreSQL instance)

### 1. Install

```bash
# Clone the repository
git clone https://github.com/Bhuvana-0406/augustus.git
cd augustus

# Create a virtual environment
python -m venv myvenv
source myvenv/bin/activate        # Windows: myvenv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Configure

```bash
cp env.example .env
```

Then edit `.env`:

```env
# Database (NeonDB or local PostgreSQL)
DATABASE_URL=postgresql://user:password@host:5432/database?sslmode=require

# Security (generate with: openssl rand -hex 32)
SECRET_KEY=your-64-char-random-key

# API keys
OPENAI_API_KEY=sk-your-openai-key
PINECONE_API_KEY=your-pinecone-key

# Environment
ENVIRONMENT=production
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

### 3. Set up the database

```bash
alembic upgrade head
```

### 4. Run

```bash
uvicorn main:app --reload
```

Open **http://localhost:8000/docs** for the interactive Swagger UI.

---

## 📖 API Reference

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/signup` | ❌ | Register a new user |
| `POST` | `/token` | ❌ | Sign in and get a JWT |
| `POST` | `/` | ✅ | Ask a question about a YouTube video |
| `GET` | `/health` | ❌ | Health check |

<details>
<summary><b>🔑 Sign up</b></summary>

```http
POST /signup
Content-Type: application/json

{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "SecurePassword123!"
}
```

</details>

<details>
<summary><b>🔑 Sign in (get JWT token)</b></summary>

```http
POST /token
Content-Type: application/x-www-form-urlencoded

username=johndoe&password=SecurePassword123!
```

**Response**

```json
{
  "access_token": "eyJ0eXAiOiJKV1...",
  "token_type": "bearer",
  "expires_in": 1800
}
```

</details>

<details>
<summary><b>💬 YouTube Q&A</b></summary>

```http
POST /
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "query": "What is this video about?",
  "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
  "session_id": "optional-session-id-for-continuity"
}
```

**Response**

```json
{
  "answer": "Based on the video transcript, this video discusses...",
  "session_id": "uuid-of-conversation-session",
  "video_context": "video-id"
}
```

</details>

---

## 🏗️ Architecture

```mermaid
flowchart LR
    A[Client<br/>JWT Auth] --> B[FastAPI App<br/>Augustus]
    B --> C[(NeonDB<br/>PostgreSQL)]
    B --> D[OpenAI<br/>GPT-4o]
    B --> E[(Pinecone<br/>Vector Store)]
    B --> F[Google<br/>Custom Search]
```

### Data flow

1. **Authenticate**: validate the JWT
2. **Session**: retrieve or create the user's session
3. **Ingest**: extract the video transcript and translate if needed
4. **Embed**: store embeddings in the user's own Pinecone index
5. **Reason**: the agent retrieves from the transcript and, if needed, Google Custom Search
6. **Respond**: GPT-4o generates the answer
7. **Persist**: save the conversation in PostgreSQL

---

## 🗄️ Database Schema

| Table | Purpose |
|---|---|
| **users** | UUID primary key, bcrypt password hashing, OAuth2 scopes (`read`, `write`, `admin`) |
| **user_sessions** | Conversation context, video ID tracking for follow-ups, 24-hour expiry with auto-cleanup |
| **messages** | Sliding-window history for Human / AI / System / Tool messages, JSONB metadata |

---

## 🔒 Security

- ✅ JWT authentication with OAuth2 scopes
- ✅ bcrypt password hashing
- ✅ Strong `SECRET_KEY` validation
- ✅ SSL/TLS encryption (NeonDB)
- ✅ CORS protection
- ✅ Input validation (Pydantic)
- ✅ SQL injection protection (SQLAlchemy)
- ✅ Per-user data isolation (separate Pinecone indexes)

---

## ☁️ Deployment

<details>
<summary><b>Railway</b></summary>

```bash
railway init
railway variables set DATABASE_URL="your-neondb-url"
railway variables set SECRET_KEY="$(openssl rand -hex 32)"
railway variables set OPENAI_API_KEY="your-key"
railway variables set PINECONE_API_KEY="your-key"
railway variables set ENVIRONMENT="production"
railway up
```

</details>

<details>
<summary><b>Render</b></summary>

1. Connect your GitHub repository
2. Set the environment variables in the dashboard
3. Deploy

</details>

### Environment variables

| Variable | Required | Description | Default |
|---|:---:|---|---|
| `DATABASE_URL` | ✅ | NeonDB connection string | - |
| `SECRET_KEY` | ✅ | 64-char random key | - |
| `OPENAI_API_KEY` | ✅ | OpenAI API key | - |
| `PINECONE_API_KEY` | ✅ | Pinecone API key | - |
| `ENVIRONMENT` | ❌ | `production` or `development` | `development` |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | ❌ | JWT expiry | `30` |
| `CORS_ORIGINS` | ❌ | Allowed origins | `*` |

---

## 🛠️ Development

### Project structure

```text
augustus/
├── main.py              # FastAPI application
├── database.py          # Database models and configuration
├── yt_to_id.py          # YouTube ID extraction utility
├── requirements.txt     # Python dependencies
├── alembic/             # Database migrations
│   ├── env.py
│   └── versions/
├── alembic.ini          # Alembic configuration
├── env.example          # Environment variable template
└── docs/                # Documentation
```

### Running tests

```bash
python test_auth_system.py          # Authentication
python test_oauth2_endpoints.py     # OAuth2 endpoints
python test_deployment_ready.py     # Deployment readiness
```

### Database migrations

```bash
alembic revision --autogenerate -m "Description"   # Create
alembic upgrade head                                # Apply
alembic downgrade -1                                # Roll back
```

---

## 📚 Documentation

- [Deployment Guide](DEPLOYMENT_READY_GUIDE.md): complete deployment instructions
- [NeonDB Setup](README_NEONDB.md): cloud database configuration
- [Authentication Guide](AUTH_README.md): auth system details
- [Quick Start](QUICK_START.md): getting started guide
- [Search Tool Update](README_TOOL_CHANGE.md): summary of the DuckDuckGo removal
- [API Docs](http://localhost:8000/docs): interactive Swagger UI (when running locally)

---

## 🗺️ Roadmap

- [ ] Email verification for user registration
- [ ] Password reset
- [ ] Rate limiting on auth endpoints
- [ ] Webhook support for real-time notifications
- [ ] Multi-video conversations
- [ ] Export conversation history
- [ ] Analytics dashboard

---

## 🤝 Contributing

Contributions are welcome!

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📝 License

Distributed under the MIT License. See [LICENSE](LICENSE) for details.

## 🙏 Acknowledgments

[FastAPI](https://fastapi.tiangolo.com) · [LangChain](https://www.langchain.com) · [Pinecone](https://www.pinecone.io) · [OpenAI](https://openai.com) · [NeonDB](https://neon.tech) · [Alembic](https://alembic.sqlalchemy.org)

---

<div align="center">

**Built with ❤️ using FastAPI, LangChain, and NeonDB**

</div>
