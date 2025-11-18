# CLAUDE.md - AI Assistant Guide for Khoj Development

> Comprehensive guide for AI assistants working with the Khoj codebase

---

## Table of Contents
1. [Repository Overview](#repository-overview)
2. [Codebase Architecture](#codebase-architecture)
3. [Technology Stack](#technology-stack)
4. [Development Setup](#development-setup)
5. [Code Conventions & Style](#code-conventions--style)
6. [Database & Models](#database--models)
7. [Testing Strategy](#testing-strategy)
8. [API Structure](#api-structure)
9. [Frontend Development](#frontend-development)
10. [Deployment](#deployment)
11. [Git Workflow](#git-workflow)
12. [Key Patterns & Conventions](#key-patterns--conventions)

---

## Repository Overview

**Project Name**: Khoj
**Description**: Your AI second brain - Personal AI app for semantic search, chat, and automation
**Version**: 2.0.0-beta.18
**License**: AGPL-3.0-or-later
**Repository**: https://github.com/khoj-ai/khoj
**Documentation**: https://docs.khoj.dev

### What is Khoj?

Khoj is a personal AI application that seamlessly scales from an on-device personal AI to a cloud-scale enterprise AI. It enables users to:
- Chat with local or online LLMs (llama3, qwen, gemma, mistral, gpt, claude, gemini, deepseek)
- Get answers from the internet and personal documents (PDF, Markdown, Org-mode, Word, Notion, Images)
- Access via Browser, Obsidian, Emacs, Desktop, Phone, or WhatsApp
- Create custom agents with knowledge, persona, and tools
- Automate research with newsletters and smart notifications
- Perform advanced semantic search across documents

---

## Codebase Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Clients                               │
│  Web │ Desktop │ Mobile │ Obsidian │ Emacs │ WhatsApp       │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                    FastAPI Backend                           │
│  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Routers  │  │Processors │  │  Tools   │  │ Operators│  │
│  └──────────┘  └───────────┘  └──────────┘  └──────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                   Django Layer                               │
│  ┌──────────┐  ┌───────────┐  ┌──────────┐                 │
│  │   ORM    │  │Migrations │  │  Admin   │                 │
│  └──────────┘  └───────────┘  └──────────┘                 │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│         PostgreSQL + pgvector (Embeddings)                   │
└─────────────────────────────────────────────────────────────┘
```

### Directory Structure

```
/home/user/khoj/
├── src/
│   ├── khoj/                      # Core Python backend
│   │   ├── app/                   # Django settings, ASGI, URLs
│   │   ├── database/              # Django models, migrations, adapters
│   │   ├── processor/             # Content processing pipeline
│   │   │   ├── content/           # Document processors (PDF, MD, Images, etc.)
│   │   │   ├── conversation/      # Chat/conversation logic
│   │   │   ├── operator/          # Agent operators (code, browser, computer)
│   │   │   ├── tools/             # MCP, code execution, web search
│   │   │   ├── speech/            # Text-to-speech
│   │   │   └── image/             # Image generation
│   │   ├── routers/               # FastAPI route handlers
│   │   ├── search_type/           # Search implementations
│   │   ├── search_filter/         # Query filtering
│   │   ├── utils/                 # Utility functions
│   │   └── interface/             # Email and web interface configs
│   ├── interface/                 # All UI implementations
│   │   ├── web/                   # Next.js web app (main UI)
│   │   ├── desktop/               # Electron desktop app
│   │   ├── android/               # Android app
│   │   ├── obsidian/              # Obsidian plugin
│   │   └── emacs/                 # Emacs plugin
│   └── telemetry/                 # Analytics
├── tests/                         # Pytest test suite
├── documentation/                 # Docusaurus documentation site
├── scripts/                       # Development/deployment scripts
├── .github/workflows/             # CI/CD pipelines
├── pyproject.toml                 # Python project config
├── docker-compose.yml             # Local development stack
├── Dockerfile                     # Local deployment image
├── prod.Dockerfile                # Production deployment image
└── computer.Dockerfile            # Operator service image
```

---

## Technology Stack

### Backend Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| **Python** | 3.10-3.12 | Core backend language |
| **FastAPI** | >=0.110.0 | REST API framework |
| **Django** | 5.1.14 | ORM, admin interface, migrations |
| **Uvicorn** | 0.30.6 | ASGI server (development) |
| **Gunicorn** | 22.0.0 | WSGI server (production) |
| **PostgreSQL** | 15 + pgvector | Vector database for embeddings |
| **Pydantic** | >=2.0.0 | Data validation |

#### AI/ML Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| **LangChain** | 0.3.27 | LLM integration framework |
| **Sentence-Transformers** | 3.4.1 | Semantic embeddings |
| **Torch** | 2.6.0 | ML framework |
| **Transformers** | >=4.51.0 | HuggingFace models |
| **OpenAI SDK** | >=2.0.0, <3.0.0 | OpenAI API client |
| **Anthropic SDK** | 0.52.0 | Claude API client |
| **Google GenAI** | 1.11.0 | Gemini API client |

#### Document Processing

| Technology | Version | Purpose |
|-----------|---------|---------|
| **PyMuPDF** | 1.24.11 | PDF processing |
| **Pillow** | ~10.0.0 | Image processing |
| **Beautiful Soup 4** | ~4.12.3 | Web scraping |
| **docx2txt** | 0.8 | Word document processing |
| **markdown-it-py** | ~3.0.0 | Markdown parsing |

### Frontend Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| **Next.js** | 14.2.32 | React framework (SSR, optimization) |
| **React** | 18.3.1 | UI library |
| **TypeScript** | 5.9.2 | Type-safe JavaScript |
| **Tailwind CSS** | 3.4.17 | Utility-first CSS |
| **shadcn-ui** | 0.9.5 | Component library |
| **Radix UI** | Latest | Accessible primitives |
| **Bun** | Latest | JavaScript runtime/bundler |
| **React Hook Form** | 7.62.0 | Form state management |
| **SWR** | 2.3.6 | Data fetching |
| **Zod** | 3.25.76 | Schema validation |

---

## Development Setup

### Prerequisites

- Python 3.10-3.12
- PostgreSQL 15+ with pgvector extension
- Bun (JavaScript runtime)
- Git
- [UV](https://github.com/astral-sh/uv) package manager (recommended)

### Quick Start (Local Development)

#### 1. Clone Repository

```bash
git clone https://github.com/khoj-ai/khoj && cd khoj
```

#### 2. Backend Setup

```bash
# Create and activate virtual environment
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies with all extras
uv sync --all-extras

# Alternative: pip install -e ".[dev]"
```

#### 3. Database Setup

```bash
# Create PostgreSQL database
createdb khoj -U postgres --password

# Set environment variables (optional, defaults work for most)
export POSTGRES_DB=khoj
export POSTGRES_USER=postgres
export POSTGRES_PASSWORD=postgres
export POSTGRES_HOST=localhost
export POSTGRES_PORT=5432

# Run migrations
python3 src/khoj/manage.py migrate
```

#### 4. Frontend Setup

```bash
cd src/interface/web
bun install
bun export  # Builds and collects static assets for Django
```

#### 5. Run Development Server

```bash
# From repository root
khoj -vv  # Runs on http://localhost:42110

# Alternative: python3 src/khoj/main.py --anonymous-mode
```

### Docker Setup (Recommended for Full Stack)

```bash
# Starts PostgreSQL, Terrarium (code sandbox), SearxNG (search), and Khoj
docker-compose up -d

# View logs
docker-compose logs -f

# Rebuild after code changes
docker-compose build --no-cache
```

### Frontend Development Server

```bash
cd src/interface/web
bun dev  # Runs on http://localhost:3000
```

**Important**: Always run `bun export` and test on http://localhost:42110 before creating a PR, as streaming doesn't work on the dev server.

---

## Code Conventions & Style

### Python Style Guide

#### Formatting & Linting

**Tool**: Ruff (replaces Black, isort, flake8)
**Line Length**: 120 characters
**Config**: `pyproject.toml`

```toml
[tool.ruff]
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "I"]  # Error, Warning, Import checks
ignore = [
    "E501",  # Line length (handled by formatter)
    "F405",  # Star imports
    "E402",  # Module level import not at top
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

#### Type Checking

**Tool**: MyPy
**Config**: Strict mode disabled, ignore missing imports

```bash
mypy src/khoj  # Run type checker
```

#### Import Order

1. Standard library imports
2. Third-party imports
3. Local/application imports

Use `from khoj.xxx import yyy` for internal imports.

#### Example Code Structure

```python
"""Module docstring describing purpose."""

import asyncio
import logging
from typing import Optional, List, Dict

from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel

from khoj.database.adapters import ConversationAdapters
from khoj.database.models import KhojUser
from khoj.routers.helpers import CommonQueryParams
from khoj.utils.helpers import is_none_or_empty

logger = logging.getLogger(__name__)

# Constants at module level
DEFAULT_TIMEOUT = 30

# Pydantic models for validation
class RequestModel(BaseModel):
    query: str
    user_id: Optional[str] = None

# Router initialization
router = APIRouter()

# Route handlers with type hints
@router.post("/api/endpoint")
async def endpoint_handler(
    request: RequestModel,
    user: KhojUser = Depends(get_current_user),
) -> Dict[str, Any]:
    """Docstring explaining endpoint behavior."""
    logger.info(f"Processing request for user {user.id}")
    # Implementation
    return {"status": "success"}
```

### TypeScript/React Conventions

#### File Naming
- Components: PascalCase (`ChatInput.tsx`)
- Utilities: camelCase (`formatDate.ts`)
- Hooks: camelCase with `use` prefix (`useChat.ts`)

#### Component Structure

```typescript
'use client';  // If using Next.js client components

import React, { useState, useEffect } from 'react';
import { Button } from '@/components/ui/button';

interface ChatInputProps {
    onSubmit: (message: string) => void;
    disabled?: boolean;
}

export function ChatInput({ onSubmit, disabled = false }: ChatInputProps) {
    const [message, setMessage] = useState('');

    const handleSubmit = () => {
        if (message.trim()) {
            onSubmit(message);
            setMessage('');
        }
    };

    return (
        <div className="flex gap-2">
            <input
                value={message}
                onChange={(e) => setMessage(e.target.value)}
                disabled={disabled}
                className="flex-1"
            />
            <Button onClick={handleSubmit}>Send</Button>
        </div>
    );
}
```

### Pre-commit Hooks

The repository uses pre-commit hooks for automatic validation:

```bash
# Install hooks
./scripts/dev_setup.sh

# Or manually
pip install pre-commit
pre-commit install

# Run manually on all files
pre-commit run --all-files

# Run manual hooks (like mypy)
pre-commit run --hook-stage manual --all
```

**Hooks Run**:
- `ruff check --fix` (linting)
- `ruff format` (formatting)
- `mypy` (type checking, pre-push only)
- Standard file checks (trailing whitespace, EOF, JSON/YAML/TOML validation)

---

## Database & Models

### Database Configuration

**Database**: PostgreSQL 15+ with pgvector extension
**ORM**: Django ORM
**Location**: `src/khoj/database/models/__init__.py`

### Key Models

#### Base Model

All models inherit from `DbBaseModel`:

```python
class DbBaseModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True
```

#### User Model

```python
class KhojUser(AbstractUser):
    # Extends Django's AbstractUser
    # Contains authentication, profile data
```

#### Conversation Models

```python
class Conversation(DbBaseModel):
    user = models.ForeignKey(KhojUser)
    # Stores conversation metadata

class ChatMessageModel(PydanticBaseModel):
    # Pydantic model for message validation
    by: str  # 'khoj' or 'you'
    message: str | list[dict]
    context: List[Context] = []
    onlineContext: Dict[str, OnlineContext] = {}
    codeContext: Dict[str, CodeContextData] = {}
```

#### Entry Models (Document Storage)

```python
class Entry(DbBaseModel):
    # Base class for document entries
    embeddings = VectorField(dimensions=...)  # pgvector field
    raw = models.TextField()
    compiled = models.TextField()
    user = models.ForeignKey(KhojUser)
```

### Migrations

#### Creating Migrations

```bash
# After modifying models in src/khoj/database/models/
python3 src/khoj/manage.py makemigrations

# Apply migrations
python3 src/khoj/manage.py migrate

# Check migration status
python3 src/khoj/manage.py showmigrations
```

#### Migration Best Practices

1. **Always review generated migrations** before committing
2. **Test migrations** on a copy of production data
3. **Use data migrations** for complex changes
4. **Squash migrations** periodically to reduce complexity
5. **Never modify applied migrations** in production

### Database Adapters

Location: `src/khoj/database/adapters/`

Adapters provide a clean interface to database operations:

```python
from khoj.database.adapters import ConversationAdapters

# Get or create conversation
conversation = await ConversationAdapters.aget_conversation_by_user(user)

# Save message
await ConversationAdapters.asave_conversation(user, conversation_log)
```

---

## Testing Strategy

### Test Framework

**Primary**: pytest
**Location**: `/home/user/khoj/tests/`
**Configuration**: `pytest.ini`, `pyproject.toml`

### Running Tests

```bash
# All tests
pytest

# Specific test file
pytest tests/test_agents.py

# Specific test function
pytest tests/test_agents.py::test_agent_creation

# Skip quality evaluation tests (slow)
pytest -m "not chatquality"

# Parallel execution
pytest -n auto

# With verbose output
pytest -vv

# Reuse database (faster)
pytest --reuse-db
```

### Test Structure

```python
import pytest
from freezegun import freeze_time
from factory import Factory

from khoj.database.models import KhojUser
from khoj.processor.conversation.utils import save_to_conversation_log

@pytest.fixture
def user_factory():
    """Fixture providing user factory."""
    class UserFactory(Factory):
        class Meta:
            model = KhojUser
        username = "testuser"
    return UserFactory

@freeze_time("2025-01-15")
def test_conversation_save(user_factory):
    """Test saving conversation with timestamp."""
    user = user_factory.create()
    result = save_to_conversation_log(user, "Test message")
    assert result.created_at.year == 2025
```

### Test Categories

1. **Unit Tests**: Individual functions/methods
   - `tests/test_conversation_utils.py`
   - `tests/test_text_search.py`

2. **Integration Tests**: API endpoints
   - `tests/test_online_chat_*.py`
   - `tests/test_client.py`

3. **Content Processing Tests**: Document processors
   - `tests/test_pdf_to_entries.py`
   - `tests/test_markdown_to_entries.py`

4. **Quality Evaluation Tests**: LLM performance benchmarks
   - `tests/evals/` (marked with `@pytest.mark.chatquality`)

### Testing Best Practices

1. **Use fixtures** for common setup (see `tests/conftest.py`)
2. **Mock external APIs** (OpenAI, Anthropic, etc.)
3. **Use factory-boy** for test data generation
4. **Test both sync and async** code paths
5. **Clean up test data** with pytest fixtures
6. **Use freezegun** for time-dependent tests

---

## API Structure

### FastAPI Routers

Location: `src/khoj/routers/`

#### Router Organization

| Router | Purpose | File |
|--------|---------|------|
| **api_chat.py** | Chat/conversation endpoints | WebSocket + HTTP |
| **api_agents.py** | Agent management | CRUD for agents |
| **api_content.py** | Document indexing | Upload, sync, delete |
| **api_automation.py** | Automations/schedules | Recurring tasks |
| **api_model.py** | LLM model config | Chat/search models |
| **api_subscription.py** | Payments/subscription | Stripe integration |
| **auth.py** | Authentication | OAuth, login, logout |
| **research.py** | Research mode | Deep research with citations |
| **web_client.py** | Static file serving | Next.js assets |

#### Example Router Pattern

```python
# src/khoj/routers/api_example.py
import logging
from typing import Optional

from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel

from khoj.database.adapters import ExampleAdapters
from khoj.database.models import KhojUser
from khoj.routers.helpers import CommonQueryParams, ApiUserRateLimiter

logger = logging.getLogger(__name__)
router = APIRouter()

# Request/Response models
class ExampleRequest(BaseModel):
    query: str
    limit: Optional[int] = 10

class ExampleResponse(BaseModel):
    results: list[str]
    count: int

# Dependency for authentication
async def get_current_user() -> KhojUser:
    # Authentication logic
    pass

# Rate limiting
rate_limiter = ApiUserRateLimiter(requests=100, window=60)

# Endpoint definition
@router.post("/api/example", response_model=ExampleResponse)
async def example_endpoint(
    request: ExampleRequest,
    common: CommonQueryParams = Depends(),
    user: KhojUser = Depends(get_current_user),
) -> ExampleResponse:
    """
    Example endpoint with proper structure.

    Args:
        request: Request body with query and limit
        common: Common query parameters (pagination, etc.)
        user: Authenticated user from dependency

    Returns:
        ExampleResponse with results and count
    """
    try:
        # Business logic
        results = await ExampleAdapters.aget_results(
            user=user,
            query=request.query,
            limit=request.limit
        )

        logger.info(f"Retrieved {len(results)} results for user {user.id}")

        return ExampleResponse(
            results=results,
            count=len(results)
        )
    except Exception as e:
        logger.error(f"Error in example endpoint: {e}", exc_info=True)
        raise HTTPException(status_code=500, detail=str(e))
```

### Authentication & Authorization

**Library**: Authlib
**Pattern**: OAuth 2.0 + Session cookies

```python
from starlette.authentication import requires

@router.get("/protected")
@requires(["authenticated"])
async def protected_route(request: Request):
    user = request.user
    # Access authenticated user
```

### Rate Limiting

```python
from khoj.routers.helpers import ApiUserRateLimiter

rate_limiter = ApiUserRateLimiter(requests=10, window=60)  # 10 req/min

@router.post("/limited")
async def limited_endpoint(
    user: KhojUser = Depends(get_current_user),
    _rate_limit: None = Depends(rate_limiter.check_rate_limit),
):
    # Endpoint logic
    pass
```

---

## Frontend Development

### Next.js Application Structure

Location: `src/interface/web/`

```
src/interface/web/
├── app/                    # Next.js 14 app directory
│   ├── (app)/             # App routes
│   │   ├── chat/          # Chat page
│   │   ├── search/        # Search page
│   │   ├── settings/      # Settings page
│   │   └── layout.tsx     # App layout
│   ├── api/               # API routes (proxies to backend)
│   └── layout.tsx         # Root layout
├── components/            # React components
│   ├── ui/               # shadcn-ui components
│   └── ...               # Custom components
├── public/               # Static assets
├── styles/               # Global styles
├── utils/                # Utility functions
├── package.json          # Dependencies
├── tsconfig.json         # TypeScript config
└── tailwind.config.js    # Tailwind config
```

### Build Process

```bash
# Development server (with hot reload)
bun dev

# Production build
bun run build

# Export static assets + Django static collection
bun run export  # This is what you should use before PRs
```

**Critical**: The `bun export` command:
1. Builds the Next.js app
2. Exports static assets to `out/` directory
3. Copies to `src/khoj/interface/built/`
4. Runs Django's `collectstatic` to move to `src/khoj/interface/compiled/`

### Component Development

Use shadcn-ui components as building blocks:

```typescript
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card } from "@/components/ui/card";

export function MyComponent() {
    return (
        <Card>
            <Input placeholder="Type here..." />
            <Button>Submit</Button>
        </Card>
    );
}
```

### State Management

- **Server state**: SWR for data fetching
- **Form state**: React Hook Form + Zod validation
- **Local state**: React useState/useContext

```typescript
import useSWR from 'swr';

function ChatComponent() {
    const { data, error, mutate } = useSWR('/api/chat/history', fetcher);

    if (error) return <div>Failed to load</div>;
    if (!data) return <div>Loading...</div>;

    return <ChatHistory messages={data} />;
}
```

### API Calls to Backend

```typescript
// Use relative paths - proxied to backend in production
async function sendMessage(message: string) {
    const response = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ q: message }),
        credentials: 'include',  // Important for cookies
    });
    return response.json();
}
```

---

## Deployment

### Docker Deployment

#### Local Development Stack

```bash
docker-compose up -d
```

Services included:
- **Khoj server** (port 42110)
- **PostgreSQL + pgvector** (port 5432)
- **Terrarium** (code sandbox, port 8080)
- **SearxNG** (web search, port 8090)

#### Production Docker Image

```bash
# Build production image
docker build -f prod.Dockerfile -t khoj:latest .

# Run production container
docker run -d \
  -p 42110:42110 \
  -e POSTGRES_DB=khoj \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=secretpassword \
  -e POSTGRES_HOST=db.example.com \
  -e KHOJ_DOMAIN=app.khoj.dev \
  -v khoj-data:/root/.khoj \
  khoj:latest
```

#### Environment Variables

Key environment variables:

```bash
# Database
POSTGRES_DB=khoj
POSTGRES_USER=postgres
POSTGRES_PASSWORD=<password>
POSTGRES_HOST=localhost
POSTGRES_PORT=5432

# Django
KHOJ_DJANGO_SECRET_KEY=<secret>
KHOJ_DOMAIN=khoj.dev
KHOJ_ALLOWED_DOMAIN=khoj.dev
KHOJ_NO_HTTPS=false  # Set to true for local dev

# LLM API Keys
OPENAI_API_KEY=<key>
ANTHROPIC_API_KEY=<key>
GEMINI_API_KEY=<key>

# Production
GUNICORN_WORKERS=6
GUNICORN_TIMEOUT=180
```

### PyPI Package Deployment

The project is published to PyPI as `khoj`:

```bash
# Install from PyPI
pip install khoj

# Run as CLI
khoj --anonymous-mode
```

Release process handled by `.github/workflows/pypi.yml` on git tags.

### Kubernetes/Cloud Deployment

Use `prod.Dockerfile` with environment-specific configurations:
- External PostgreSQL with pgvector
- Load balancer in front of Gunicorn
- Redis for caching (optional)
- S3 or cloud storage for user uploads

---

## Git Workflow

### Branch Naming

- `master` - Main branch (stable releases)
- `feature/<name>` - New features
- `fix/<name>` - Bug fixes
- `refactor/<name>` - Code refactoring
- `docs/<name>` - Documentation updates

### Commit Messages

Follow conventional commits:

```
type(scope): description

[optional body]

[optional footer]
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

Examples:
```
feat(chat): add support for Claude 4 models
fix(search): resolve unicode handling in PDF parsing
docs(api): update authentication flow documentation
```

### PR Workflow

1. **Create GitHub Issue**: Discuss design before coding
2. **Create Feature Branch**: `git checkout -b feature/my-feature`
3. **Make Changes**: Follow code conventions
4. **Run Tests**: `pytest`
5. **Run Linter**: `mypy` (or use pre-commit hooks)
6. **Commit**: Use conventional commit messages
7. **Push**: `git push origin feature/my-feature`
8. **Create PR**: Link to GitHub issue
9. **Address Review**: Make requested changes
10. **Merge**: Squash and merge after approval

### Pre-PR Checklist

- [ ] Linked to GitHub issue
- [ ] Tests pass (`pytest`)
- [ ] Type checking passes (`mypy`)
- [ ] Linting passes (`ruff check`)
- [ ] Frontend builds (`bun export`)
- [ ] Manual testing completed
- [ ] Documentation updated if needed
- [ ] Conventional commit messages used

---

## Key Patterns & Conventions

### 1. Async/Await Pattern

Khoj uses async extensively for I/O operations:

```python
# Use async for DB operations
async def get_user_conversations(user: KhojUser):
    return await ConversationAdapters.aget_conversations_by_user(user)

# Use async for LLM calls
async def generate_response(query: str):
    response = await anthropic_client.messages.create(...)
    return response
```

### 2. Logging Pattern

```python
import logging

logger = logging.getLogger(__name__)

# Use appropriate log levels
logger.debug("Detailed diagnostic info")
logger.info("General informational messages")
logger.warning("Warning messages")
logger.error("Error messages", exc_info=True)  # Include traceback
```

### 3. Error Handling

```python
from fastapi import HTTPException

try:
    result = await some_operation()
except ValueError as e:
    logger.error(f"Invalid input: {e}")
    raise HTTPException(status_code=400, detail=str(e))
except Exception as e:
    logger.error(f"Unexpected error: {e}", exc_info=True)
    raise HTTPException(status_code=500, detail="Internal server error")
```

### 4. Pydantic Validation

Use Pydantic for request/response validation:

```python
from pydantic import BaseModel, Field, field_validator

class ChatRequest(BaseModel):
    query: str = Field(..., min_length=1, max_length=5000)
    model: Optional[str] = None

    @field_validator('query')
    def validate_query(cls, v):
        if not v.strip():
            raise ValueError('Query cannot be empty')
        return v.strip()
```

### 5. Database Adapter Pattern

Always use adapters for database operations:

```python
# Good
from khoj.database.adapters import ConversationAdapters
conversations = await ConversationAdapters.aget_conversations_by_user(user)

# Avoid direct model queries in routers
# conversations = await Conversation.objects.filter(user=user)
```

### 6. Content Processor Pattern

All content processors inherit from base class:

```python
from khoj.processor.content.text_to_entries import TextToEntries

class CustomToEntries(TextToEntries):
    def __init__(self):
        super().__init__()

    async def process(self, files: list) -> list[Entry]:
        # Processing logic
        pass
```

### 7. Frontend Data Fetching

Use SWR for server state:

```typescript
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

function useConversations() {
    const { data, error, mutate } = useSWR('/api/chat/sessions', fetcher, {
        refreshInterval: 30000,  // Refresh every 30s
        revalidateOnFocus: true,
    });

    return {
        conversations: data,
        isLoading: !error && !data,
        isError: error,
        refresh: mutate,
    };
}
```

### 8. Environment Configuration

```python
import os
from khoj.utils.helpers import is_env_var_true

# Boolean flags
DEBUG_MODE = is_env_var_true("DEBUG")

# Required values
API_KEY = os.getenv("OPENAI_API_KEY")
if not API_KEY:
    raise ValueError("OPENAI_API_KEY environment variable required")

# Optional with defaults
MAX_WORKERS = int(os.getenv("MAX_WORKERS", "4"))
```

### 9. LLM Integration Pattern

```python
# Use standardized interface for different providers
from khoj.processor.conversation.anthropic.anthropic_chat import converse_anthropic
from khoj.processor.conversation.openai.gpt import converse_openai
from khoj.processor.conversation.google.gemini_chat import converse_gemini

# Select based on config
if model.startswith("claude"):
    response = await converse_anthropic(...)
elif model.startswith("gpt"):
    response = await converse_openai(...)
elif model.startswith("gemini"):
    response = await converse_gemini(...)
```

### 10. Vector Embeddings Pattern

```python
from khoj.processor.embeddings import EmbeddingsModel

# Generate embeddings
embeddings_model = EmbeddingsModel()
embeddings = embeddings_model.embed_query(query_text)

# Search with pgvector
from pgvector.django import CosineDistance

results = Entry.objects.annotate(
    distance=CosineDistance('embeddings', embeddings)
).order_by('distance')[:10]
```

---

## Important Conventions for AI Assistants

### File Operations

1. **Always read before edit**: Use Read tool before Edit/Write
2. **Preserve exact indentation**: Match existing code style
3. **Prefer Edit over Write**: Modify existing files rather than overwriting
4. **Test after frontend changes**: Run `bun export` after modifying web code

### Code Modifications

1. **Follow existing patterns**: Match the style of surrounding code
2. **Update tests**: Add/modify tests for new functionality
3. **Update type hints**: All functions should have type annotations
4. **Add logging**: Use logger for debugging and error tracking
5. **Handle errors**: Never let exceptions bubble up unhandled

### Database Changes

1. **Create migrations**: Always run `makemigrations` after model changes
2. **Test migrations**: Apply on test database before committing
3. **Use adapters**: Never bypass the adapter pattern
4. **Async operations**: Use `a` prefixed methods for async (e.g., `aget`, `asave`)

### Testing Requirements

1. **Write tests first**: TDD when possible
2. **Test async code**: Use pytest-asyncio markers
3. **Mock external APIs**: Don't make real API calls in tests
4. **Clean up**: Use fixtures for setup/teardown

### Documentation

1. **Docstrings**: Add to all public functions/classes
2. **Type hints**: Use everywhere for better IDE support
3. **Comments**: Explain complex logic, not obvious code
4. **Update docs**: Modify `/documentation` for user-facing changes

### Performance Considerations

1. **Use async**: For all I/O operations (DB, API calls, file ops)
2. **Batch operations**: Don't query in loops
3. **Index properly**: Add database indexes for frequent queries
4. **Cache when appropriate**: Use Redis or in-memory caching
5. **Stream responses**: Use streaming for long-running LLM calls

### Security

1. **Validate input**: Use Pydantic models
2. **Sanitize output**: Escape user content in HTML
3. **Use parameterized queries**: Django ORM handles this
4. **Check permissions**: Verify user access before operations
5. **Environment secrets**: Never commit API keys

---

## Common Development Tasks

### Adding a New API Endpoint

1. Create route handler in appropriate router file
2. Define Pydantic request/response models
3. Add database adapter methods if needed
4. Write unit tests in `tests/`
5. Update API documentation

### Adding a New Document Processor

1. Create processor in `src/khoj/processor/content/<type>/`
2. Inherit from `TextToEntries` base class
3. Implement `process()` method
4. Add tests in `tests/test_<type>_to_entries.py`
5. Register processor in content router

### Adding a New LLM Provider

1. Create module in `src/khoj/processor/conversation/<provider>/`
2. Implement `converse_<provider>()` function
3. Add provider-specific configuration in Django admin
4. Update model selection logic in routers
5. Add integration tests

### Modifying Database Schema

1. Edit models in `src/khoj/database/models/__init__.py`
2. Run `python3 src/khoj/manage.py makemigrations`
3. Review generated migration file
4. Test migration: `python3 src/khoj/manage.py migrate`
5. Update adapters to use new fields
6. Add tests for new schema

### Adding Frontend Feature

1. Create component in `src/interface/web/components/`
2. Add TypeScript types
3. Implement UI with shadcn-ui components
4. Add API calls to backend
5. Run `bun dev` for testing
6. Run `bun export` before committing
7. Test on `http://localhost:42110`

---

## Troubleshooting

### Common Issues

**Issue**: `ModuleNotFoundError: No module named 'khoj'`
**Solution**: Activate virtual environment and reinstall: `uv sync --all-extras`

**Issue**: `Database connection refused`
**Solution**: Check PostgreSQL is running and environment variables are set correctly

**Issue**: `Frontend changes not reflecting`
**Solution**: Run `bun export` to rebuild and collect static files

**Issue**: `Migration conflicts`
**Solution**: Delete migration file, run `makemigrations` again, or resolve conflicts manually

**Issue**: Type checking fails
**Solution**: Run `mypy src/khoj` and fix reported issues

**Issue**: Tests failing in CI but passing locally
**Solution**: Check Python version (CI tests on 3.10, 3.11, 3.12), ensure database is clean

---

## Resources

- **Documentation**: https://docs.khoj.dev
- **Repository**: https://github.com/khoj-ai/khoj
- **Discord**: https://discord.gg/WaxF3SkFPU
- **Issues**: https://github.com/khoj-ai/khoj/issues
- **Good First Issues**: https://github.com/khoj-ai/khoj/contribute

---

**Last Updated**: 2025-01-18
**Khoj Version**: 2.0.0-beta.18

---

*This guide is maintained for AI assistants to effectively navigate and contribute to the Khoj codebase. For user-facing documentation, see https://docs.khoj.dev.*
