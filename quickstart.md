# Quick Start Guide

## Prerequisites

**Frontend Requirements:**
- **Node.js** 18.0.0+ (Required per package.json engines)
- **npm** or **pnpm** (Package manager)
- **Git** (For version control)

**Backend Requirements:**
- **Python** 3.10.16 (Exact version required per pyproject.toml)
- **uv** (Python package manager, used in project)
- **PostgreSQL** (Database for chat history, MCP tools, and Skills)

**Additional Tools:**
- **Electron** 33.2.0 (For desktop builds)
- **TypeScript** 5.4.2 (Type safety for frontend)
- **Vite** 5.4.11 (Frontend build tool)

## Installation

### 1. Clone Repository

```bash
git clone https://github.com/hachiai/ai-agent-platform.git
cd ai-agent-platform
```

### 2. Install Frontend Dependencies

```bash
npm install
```

This installs all frontend dependencies including:
- React 18.3.1, TypeScript 5.4.2
- Vite 5.4.11, Electron 33.2.0
- Zustand 5.0.4 (state management)
- Radix UI components, Tailwind CSS 3.4.15
- @stackframe/react, react-oidc-context (StackAuth)
- @microsoft/fetch-event-source (SSE client)

### 3. Install Backend Dependencies

```bash
cd backend
uv sync
```

This installs Python dependencies including:
- FastAPI ≥0.115.12 (API framework)
- CAMEL-AI ≥0.2.76a6, <0.2.79 (Multi-agent framework)
- Uvicorn ≥0.34.2 (ASGI server)
- Pydantic-i18n ≥0.4.5 (Internationalized data models)
- Loguru ≥0.7.3 (Logging)
- OpenAI ≥1.99.3, <2 (LLM client)

**Note:** Python 3.10.16 is strictly required. The project uses uv as the package manager.

### 4. Configure Environment

```bash
# Copy environment template
cp .env.example .env
# Or use development template
cp .env.development .env
```

Edit `.env` with required configuration:

**Frontend Environment Variables:**
```bash
# Backend API URL (local or remote)
VITE_BASE_URL=https://dev-agent-backend.hachiai.com
VITE_PROXY_URL=https://dev-agent-backend.hachiai.com

# StackAuth (Keycloak) Configuration
VITE_AUTHORITY=https://keycloak-auth-dev.hachiai.com/realms/bank-a
VITE_CLIENT_ID=ai-agent-platform

# Nexus Usage Reporting (Optional)
VITE_NEXUS_BASE_URL=https://nexus.hachiai.com
VITE_CREDIT_TYPE=organization_then_personal
```

**Backend Environment Variables:**
```bash
# Python Encoding
PYTHONIOENCODING=utf-8

# Environment
ENVIRONMENT=development

# URL Prefix (optional)
url_prefix=
```

### 5. Set Up Database

**Prerequisites:**
- PostgreSQL must be installed and running
- Database name, user, and password configured

**Database Schema:**
The project uses the following tables (managed through ORM or direct SQL):
- `chat_history` - Main task/workflow records
- `chat_step` - Individual execution steps with JSON data
- `chat_snapshot` - Browser screenshots and visual state
- `user` - User management (StackAuth integration)
- `mcp_user` - MCP tool configurations and API keys

**Note:** The project does not include explicit migration scripts. Database schema is managed dynamically through the ORM layer.

### 6. Start Development Server

**Start Frontend (Vite + Electron):**
```bash
# Terminal 1 - Frontend development server
npm run dev
```
This compiles Python babel translations and starts Vite dev server at http://localhost:5174

**Start Backend (FastAPI + Uvicorn):**
```bash
# Terminal 2 - Backend API server
cd backend
uvicorn app.main:app --reload --host 0.0.0.0 --port 3001
```

Or use the provided CLI:
```bash
cd backend
python main.py
```

**Note:** Backend runs on port 3001 by default. Frontend proxies API calls to this port via `VITE_PROXY_URL`.

### 7. Access Application

- **Frontend:** http://localhost:5174
- **Backend API:** http://localhost:3001
- **API Documentation:** http://localhost:3001/docs (FastAPI auto-generated)

## Project Structure

```
ai-agent-platform/
├── src/                      # Frontend source code
│   ├── api/                  # API client layer
│   ├── auth/                  # StackAuth (OIDC) configuration
│   ├── components/            # UI components (Dialog, Terminal, Workflow, etc.)
│   ├── config/                # Application configuration
│   ├── hooks/                 # React custom hooks
│   ├── i18n/                 # Internationalization
│   ├── lib/                   # Utility libraries
│   ├── pages/                 # Page components
│   │   ├── Home.tsx          # Workflow designer
│   │   ├── History.tsx        # Task history
│   │   ├── Iris.tsx           # Agent visualization
│   │   ├── Login.tsx          # Authentication
│   │   ├── SignUp.tsx        # Registration
│   │   └── Setting/           # Settings pages
│   │       ├── General.tsx
│   │       ├── MCP.tsx          # MCP tool management
│   │       └── ...             # Other settings
│   ├── routers/               # React Router DOM configuration
│   ├── stack/                 # Zustand state stores
│   │   ├── authStore.ts      # Authentication state
│   │   ├── chatStore.ts      # Task/chat state
│   │   ├── globalStore.ts     # Application-wide state
│   │   └── ...              # Other stores
│   ├── store/                 # Additional stores
│   ├── style/                 # Global styles
│   ├── types/                 # TypeScript type definitions
│   ├── utils/                 # Utility functions
│   ├── App.tsx                # Root React component
│   └── main.tsx               # Application entry point
├── backend/                  # Backend source code
│   ├── app/
│   │   ├── api/              # API endpoints and routers
│   │   │   ├── health/        # Health check endpoints
│   │   │   ├── chat/         # Chat/session endpoints
│   │   │   ├── tasks/        # Task management endpoints
│   │   │   ├── agents/       # Agent management endpoints
│   │   │   ├── mcp/          # MCP tool endpoints
│   │   │   ├── learning/     # Learning system endpoints
│   │   │   └── models/       # Model configuration endpoints
│   │   ├── controller/       # HTTP request handlers
│   │   │   ├── chat_controller.py
│   │   │   ├── task_controller.py
│   │   │   ├── model_controller.py
│   │   │   └── tool_controller.py
│   │   ├── service/          # Business logic layer
│   │   │   ├── chat_service.py
│   │   │   ├── task.py         # Task queue, SSE, TaskLock
│   │   │   └── ...           # Other services
│   │   ├── model/            # Pydantic data models
│   │   │   ├── chat.py       # Chat, Status, NewAgent models
│   │   │   └── ...           # Other models
│   │   ├── utils/            # Utility functions
│   │   │   ├── workforce.py   # CAMEL-AI Workforce Manager
│   │   │   ├── single_agent_worker.py
│   │   │   └── toolkit/       # Toolkit implementations
│   │   ├── component/        # Shared components
│   │   │   ├── encrypt.py     # Password hashing (bcrypt)
│   │   │   └── environment.py # Configuration helpers
│   │   └── ...             # Other modules
│   ├── tests/                # Backend unit and integration tests
│   ├── main.py               # FastAPI application entry point
│   └── pyproject.toml         # Python dependencies
├── electron/                  # Electron main process
│   ├── main.ts               # Electron entry point
│   └── preload.ts            # Preload scripts
├── specs/                    # Specifications (this project)
│   └── modernization/        # Modernization specs
├── test/                     # Frontend tests (Vitest, Playwright)
├── public/                   # Static assets
├── package.json               # Frontend dependencies and scripts
├── tsconfig.json             # TypeScript configuration
├── vite.config.ts            # Vite configuration
├── tailwind.config.js        # Tailwind CSS configuration
└── electron-builder.json       # Electron build configuration
```

## Common Commands

### Development

**Frontend:**
- `npm run dev` - Start development server (Vite)
- `npm run compile-babel` - Compile Python translations
- `npm run type-check` - Run TypeScript type checking
- `npm run lint` - Lint code with ESLint
- `npm run lint:fix` - Fix linting issues automatically
- `npm run format` - Check code formatting with Prettier
- `npm run format:fix` - Format code with Prettier

**Backend:**
- `cd backend && uv run pybabel compile -d lang` - Compile Python translations
- `cd backend && python main.py` - Start FastAPI server
- `cd backend && uvicorn app.main:app --reload` - Start with auto-reload

### Testing

**Frontend Tests:**
- `npm run test` - Run Vitest unit tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:e2e` - Run Playwright end-to-end tests
- `npm run test:e2e:ui` - Run Playwright tests with UI
- `npm run test:e2e:debug` - Debug Playwright tests
- `npm run test:coverage` - Generate test coverage report
- `npm run playwright:install` - Install Playwright browsers

**Backend Tests:**
```bash
cd backend
uv run pytest
uv run pytest tests/unit/  # Run unit tests only
```

### Building

**Frontend:**
- `npm run build` - Build for production (Electron bundle)
- `npm run build:mac` - Build for macOS only
- `npm run build:win` - Build for Windows only
- `npm run build:all` - Build for all platforms
- `npm run preview` - Preview production build

**Electron Builds:**
- `npm run build` - Uses electron-builder to create installers
- Output in `dist-electron/` directory

### Database

**Note:** The project does not include explicit migration scripts. Database schema is managed dynamically through the ORM layer.

**Manual Database Setup (if needed):**
1. Create PostgreSQL database
2. Configure connection string in backend (if applicable)
3. Application auto-creates required tables on first run

**Tables Created:**
- `chat_history` - Task/workflow records
- `chat_step` - Execution steps
- `chat_snapshot` - Browser screenshots
- `user` - User accounts
- `mcp_user` - MCP tool configurations

## Troubleshooting

### Backend Won't Start

**Symptom:** `uvicorn app.main:app` fails with import errors

**Solutions:**
- Verify Python 3.10.16 is installed: `python --version`
- Run `cd backend && uv sync` to ensure dependencies are installed
- Check that `PYTHONIOENCODING=utf-8` is set in environment
- Verify PostgreSQL is accessible if database connection is required

### Frontend Won't Start

**Symptom:** `npm run dev` fails or blank page

**Solutions:**
- Clear node_modules and reinstall: `rm -rf node_modules package-lock.json && npm install`
- Verify VITE_BASE_URL and VITE_PROXY_URL in `.env`
- Check backend is running on port 3001
- Verify Node.js version: `node --version` (must be >= 18.0.0 < 23.0.0)

### Database Connection Error

**Symptom:** Application fails to connect to PostgreSQL

**Solutions:**
- Verify PostgreSQL is running: `pg_isready` or check service status
- Check database credentials in configuration
- Ensure database exists: `createdb hachiai`
- Verify network connectivity to database host

### Port Already in Use

**Symptom:** Error "Address already in use" for port 3001 or 5174

**Solutions:**
- Kill existing process:
  ```bash
  # macOS/Linux
  lsof -ti:3001 | xargs kill -9
  lsof -ti:5174 | xargs kill -9
  ```
- Change port in environment variables
- Use different ports for frontend/backend in development

### CORS Errors

**Symptom:** Browser shows CORS errors when accessing API

**Solutions:**
- Verify `VITE_PROXY_URL` in `.env` matches backend URL
- Check backend CORS middleware configuration in `main.py`
- Ensure both frontend and backend are using the same protocol (http/https)

### TypeScript Errors

**Symptom:** Type errors in IDE or build

**Solutions:**
- Run `npm run type-check` to identify issues
- Check `tsconfig.json` configuration
- Ensure all dependencies are installed: `npm install`
- Clear TypeScript cache: `rm -rf node_modules/.vite`

## Next Steps

After setup:
1. **Review Implementation Plan** - See `specs/modernization/plan.md` for detailed task breakdown
2. **Check Task List** - See `specs/modernization/tasks.md` for actionable tasks
3. **Start Implementing** - Use `/speckit.implement` to begin implementation

### Learning Resources

- **Project README** - See `/README.md` for project overview
- **Documentation** - Visit https://docs.hachiai.com/v2 for comprehensive docs
- **Contribution Guide** - See `/CONTRIBUTING.md` for development guidelines
- **System Architecture** - See `/system-architecture.md` for deep technical details

### Skills Integration Quick Reference

For the Claude Skills Integration feature:

1. **Database Tables** - `claude_skills`, `claude_skill_configs`, `claude_skill_executions`
2. **Backend Extensions** - Skills controller, service, plugin loader, toolkit
3. **Frontend Extensions** - Skills pages, components, Zustand store
4. **API Endpoints** - `/api/v1/skills/*` for CRUD and execution
5. **Skills Directory** - `~/.hachiai/skills/` for local skill manifests

**Skills Workflow:**
1. Install skill → Validate manifest → Save to database
2. Configure skill → Set API keys and options
3. Enable skill → Make available to agents
4. Execute skill → Real-time SSE streaming
5. Monitor → View execution logs and output
