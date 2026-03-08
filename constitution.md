# Modernization Constitution

## Project Identity
- **Project Name:** HachiAI Claude Skills Integration
- **Source Path:** /Users/puttaiaharugunta/codebase/hachi/ai-agent-platform
- **Strategy:** hybrid

## Governing Principles
1. **Preserve Multi-Agent Architecture** — The existing CAMEL-AI workforce manager and agent orchestration system must remain intact
2. **Maintain Real-Time Streaming** — All existing SSE (Server-Sent Events) streaming infrastructure for task execution visibility must be preserved
3. **StackAuth Integration** — Existing OIDC-based authentication system must continue to work without changes
4. **Extend MCP Pattern** — Leverage existing MCP tool system as the pattern for Claude Skills integration
5. **No Breaking Changes** — All existing workflows, agents, and UI patterns must continue to function
6. **Type Safety** — All new code must maintain TypeScript (frontend) and Python type safety (backend)
7. **Data Encryption** — Skill API keys and credentials must be encrypted at rest

## Current State
- **Languages:** TypeScript 5.4.2 (Frontend), Python 3.10.16 (Backend)
- **Frontend:** React 18.3.1 with Vite 5.4.11, Electron 33.2.0 desktop wrapper
- **Backend:** FastAPI ≥0.115.12 with CAMEL-AI ≥0.2.76a6, <0.2.79
- **Database:** PostgreSQL (for chat_history, chat_step, chat_snapshot, user, mcp_user tables)
- **Styling:** Tailwind CSS 3.4.15 with Radix UI components
- **State Management:** Zustand 5.0.4
- **Authentication:** StackAuth (OIDC) via @stackframe/react and react-oidc-context
- **Real-time:** SSE streaming via @microsoft/fetch-event-source

## Target State
- **Frontend:** React 18.3.1 + TypeScript 5.4.2 + Zustand 5.0.4 (extend with Skills pages)
- **Backend:** FastAPI + Python 3.10.16 + CAMEL-AI (extend with Skills API endpoints and plugin loader)
- **Database:** PostgreSQL with new tables: `claude_skills`, `claude_skill_configs`, `claude_skill_executions`
- **Styling:** Tailwind CSS 3.4.15 + Radix UI (extend with Skills UI components)
- **Skills Plugin System:** Dynamic JSON manifest-based skill loader with isolated execution context

## Migration Boundaries

### MUST Preserve
- **Multi-agent workforce architecture** — CAMEL-AI Workforce Manager, agent orchestration, and task routing
- **Real-time SSE streaming** — All existing SSE endpoints and event types for task execution
- **Existing agent types** — Developer, Search, Document, Multi-Modal agents and their capabilities
- **StackAuth authentication** — OIDC integration, user sessions, and role-based access
- **MCP tool system** — Existing MCP tool configuration model (`mcp_user`), controllers, and services
- **Zustand stores** — chatStore, authStore, globalStore, installationStore patterns
- **Database schemas** — Existing `chat_history`, `chat_step`, `chat_snapshot`, `user`, `mcp_user` tables
- **UI patterns** — Layout components, Settings page structure, Dialog components, Toast notifications
- **API contract** — Existing `/api/chat/*`, `/api/task/*`, `/api/tool/*` endpoints
- **Electron wrapper** — Desktop app model, IPC communication, auto-updater

### MAY Change
- **Tool system extension** — Add skills plugin loader alongside existing MCP tool system
- **Settings UI** — Add Skills management tab/page under `/setting/skills`
- **Database schema** — Add new tables for skills storage (claude_skills, claude_skill_configs, claude_skill_executions)
- **Agent toolkit registration** — Allow dynamic skill-to-tool mapping in workforce.py
- **API endpoints** — Add skills CRUD endpoints (`/api/skills/*`)
- **Frontend routing** — Add Skills management routes to existing router structure
- **Workflow designer** — Extend to support skill blocks as draggable components
- **State management** — Add `skillsStore.ts` following existing Zustand patterns

### WILL Drop
- None — This is an additive feature; no existing functionality will be removed

## Agent Guidelines

### Code Reading Requirements
- Agents **must read** the following files before implementing any changes:
  - `backend/app/model/chat.py` — Understand existing Chat data models and status enums
  - `backend/app/service/task.py` — Understand TaskLock, SSE actions, and agent lifecycle
  - `backend/app/controller/tool_controller.py` — Understand existing MCP tool installation pattern
  - `backend/app/utils/workforce.py` — Understand CAMEL-AI workforce orchestration
  - `src/store/chatStore.ts` — Understand existing state management and SSE handling patterns
  - `src/store/authStore.ts` — Understand authentication integration with Zustand
  - `src/routers/index.tsx` — Understand routing structure and protected routes pattern
  - `src/pages/Setting/MCP.tsx` — Understand existing MCP management UI as reference for Skills UI

### Verification Requirements
- Each Skills API endpoint must include test cases in `backend/tests/`
- Skills installation must be verified to correctly persist to `claude_skills` table
- Skill execution must produce real-time SSE updates matching existing task streaming format
- Skill configuration encryption must be verified using existing encryption patterns
- Skills must appear as available tools in agent toolkit registration

### Database Migration Requirements
- Create migration scripts for new tables in `backend/app/migrations/` (if using Alembic)
- Include data migration scripts if any schema changes to existing tables
- Test migrations on staging database before production deployment

### API Endpoint Requirements
- All skills endpoints must maintain the same authentication patterns (StackAuth token validation)
- Skills endpoints must use existing CORS middleware configuration
- Skills endpoints must return error responses matching existing error format
- All new SSE streams must follow the `sse_json()` format from `app/model/chat.py`

### Frontend Development Requirements
- Use existing Radix UI components for dialogs, dropdowns, and forms
- Follow existing Zustand middleware patterns (`persist`, `subscribeWithSelector`)
- Use existing Toast notification system (`sonner`) for user feedback
- Follow existing code style (Prettier, ESLint configuration)
- Maintain responsive design patterns with Tailwind CSS

### Agent Integration Requirements
- Skills must be registered in CAMEL-AI agent toolkit using existing `AbstractToolkit` pattern
- Skill execution must use existing `TaskLock` queue mechanism for SSE communication
- Skills must not modify global state — all state must be task-scoped or persisted
- Skill failures must be isolated and logged using existing `loguru` patterns

### Testing Requirements
- Write unit tests for all new services using `pytest`
- Write integration tests for API endpoints
- Write frontend component tests using `vitest` and `@testing-library/react`
- Verify backward compatibility by running existing test suite

### Security Requirements
- All skill API keys must be encrypted using existing encryption utilities in `backend/app/component/encrypt.py`
- Skill manifests must be validated before installation (JSON schema validation)
- Skill execution must run in isolated context (no shared mutable state)
- Audit log all skill installations, configurations, and executions

### Performance Requirements
- Skills loading must not increase application startup time by more than 5 seconds
- Skill installation API calls must complete within 30 seconds
- Support at least 100 installed skills per user
- Support at least 10 concurrent skill executions

### Documentation Requirements
- Update API documentation (FastAPI auto-generated docs) for new endpoints
- Add inline comments for complex skill execution logic
- Document skill manifest schema and validation rules
- Update user-facing documentation for skills management UI
- save