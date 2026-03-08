# Claude Skills Integration Tasks

## Database Migration (3 tasks)
- [ ] **Task-1** Create PostgreSQL migration script for new skills tables: `claude_skills`, `claude_skill_configs`, `claude_skill_executions`
- [ ] **Task-2** [P] Set up database indexes for performance optimization on `user_id`, `enabled`, and `task_id` columns
- [ ] **Task-3** [P] Test migration on development database with existing data integrity verification

## Backend Core Implementation (12 tasks)
- [ ] **Task-4** [P] Add new Python dependencies to `/backend/pyproject.toml`: `jsonschema>=4.20.0`, `cryptography>=41.0.0`
- [ ] **Task-5** [P] Create `/backend/app/model/skills.py` with Pydantic models: `SkillManifest`, `SkillParameter`, `SkillConfigField`, `SkillExecutionSpec`, `SkillUISpec`, `SkillInstallRequest`, `SkillUpdateRequest`, `SkillConfigRequest`, `SkillExecutionRequest`, `SkillItem`, `SkillConfig`, `SkillExecution`
- [ ] **Task-6** [P] Create `/backend/app/utils/skills_plugin_loader.py` with `SkillsPluginLoader` class for dynamic skill discovery and validation
- [ ] **Task-7** Create `/backend/app/service/skills_service.py` with business logic: `get_user_skills()`, `load_skill_manifest()`, `install_skill_to_db()`, `delete_skill_from_db()`, `get_skill_config()`, `update_skill_config()`, `check_dependencies()`
- [ ] **Task-8** Create `/backend/app/controller/skills_controller.py` with FastAPI endpoints: `GET /api/v1/skills`, `POST /api/v1/skills/discover`, `POST /api/v1/skills/install`, `PUT /api/v1/skills/{skill_id}`, `DELETE /api/v1/skills/{skill_id}`, `POST /api/v1/skills/{skill_id}/enable`, `POST /api/v1/skills/{skill_id}/disable`, `GET /api/v1/skills/{skill_id}/config`, `PUT /api/v1/skills/{skill_id}/config`, `POST /api/v1/skills/{skill_id}/execute`, `GET /api/v1/skills/{skill_id}/logs`
- [ ] **Task-9** Create `/backend/app/api/skills/__init__.py` to register skills router with prefix `/api/v1/skills`
- [ ] **Task-10** [P] Create `/backend/app/utils/toolkit/claude_skills_toolkit.py` with `ClaudeSkillsToolkit` class extending `AbstractToolkit`
- [ ] **Task-11** Extend `/backend/app/service/skills_service.py` with `execute_skill_async()` for SSE streaming (following `sse_json()` pattern from `/backend/app/model/chat.py`)
- [ ] **Task-12** Update `/backend/app/utils/workforce.py` to integrate skills toolkit: add `claude_skills_enabled` parameter, `_load_claude_skills_toolkit()` method, and call it in `Workforce.__init__()`
- [ ] **Task-13** [P] Create `/backend/app/component/api_key_encryption.py` with AES-256 encryption/decryption functions for secure storage
- [ ] **Task-14** Verify skills router is auto-discovered by `auto_include_routers()` in `/backend/app/component/environment.py`

## Frontend Type Definitions & API Client (2 tasks)
- [ ] **Task-15** [P] Create `/src/types/skills.ts` with TypeScript interfaces: `SkillManifest`, `SkillParameter`, `SkillConfigField`, `SkillExecutionSpec`, `SkillUISpec`, `SkillItem`, `SkillConfig`, `SkillExecution`, `SkillsState`
- [ ] **Task-16** [P] Create `/src/api/skills.ts` with API client functions: `listSkills()`, `discoverSkills()`, `installSkill()`, `uninstallSkill()`, `enableSkill()`, `disableSkill()`, `getSkillConfig()`, `updateSkillConfig()`, `executeSkill()` with SSE, `getSkillLogs()`

## Frontend State Management (1 task)
- [ ] **Task-17** Create `/src/store/skillsStore.ts` following Zustand pattern from `/src/store/authStore.ts` with `persist` middleware and actions: `setInstalledSkills()`, `addSkill()`, `removeSkill()`, `enableSkill()`, `disableSkill()`, `updateSkillConfig()`, `setAvailableSkills()`

## Frontend Routing Updates (1 task)
- [ ] **Task-18** Update `/src/routers/index.tsx` to add lazy imports for skills pages: `SettingSkills`, `SkillsCatalog` and routes: `/setting/skills`, `/setting/skills_catalog`

## Frontend Components (6 tasks)
- [ ] **Task-19** Create `/src/pages/Setting/Skills.tsx` following MCP.tsx pattern with state management, enable/disable switches, configuration dialog, delete confirmation dialog
- [ ] **Task-20** [P] Create `/src/pages/Setting/SkillsCatalog.tsx` with skills browsing, search/filter functionality, and install buttons
- [ ] **Task-21** [P] Create `/src/components/Skill/SkillConfiguration.tsx` using Radix UI Dialog with form fields based on `skill.manifest.configuration` (text input, password input for API keys, switch for boolean)
- [ ] **Task-22** [P] Create `/src/components/Skill/SkillExecutionMonitor.tsx` with SSE handling using `@microsoft/fetch-event-source`, progress tracking, status display, and cancel functionality
- [ ] **Task-23** [P] Create `/src/components/Skill/SkillCard.tsx` using existing card component from `/src/components/ui/card.tsx` with skill name, description, capabilities badges, and action buttons
- [ ] **Task-24** [P] Create `/src/components/Workflow/SkillBlock.tsx` as draggable component following `/src/components/WorkFlow/node.tsx` pattern for workflow designer integration

## Backend Unit Tests (2 tasks)
- [ ] **Task-25** [P] Create `/backend/tests/unit/controller/test_skills_controller.py` following patterns from `/backend/tests/unit/controller/test_tool_controller.py` for all skills endpoints
- [ ] **Task-26** [P] Create `/backend/tests/unit/service/test_skills_service.py` following patterns from `/backend/tests/unit/service/test_chat_service.py` for skills service functions and SSE streaming

## Frontend Unit Tests (2 tasks)
- [ ] **Task-27** [P] Create `/src/store/__tests__/skillsStore.test.ts` following existing vitest patterns for store actions and `persist` middleware
- [ ] **Task-28** [P] Create `/src/pages/Setting/__tests__/Skills.test.tsx` using `@testing-library/react` for component rendering, enable/disable functionality, configuration dialog, and delete confirmation

## Integration Tests (1 task)
- [ ] **Task-29** Create `/backend/tests/integration/test_skills_integration.py` with end-to-end tests for skill installation, execution with SSE, skills toolkit registration, and configuration encryption

## Security Implementation (2 tasks)
- [ ] **Task-30** [P] Implement API key encryption using AES-256 in `/backend/app/component/api_key_encryption.py` with environment variable `SKILLS_ENCRYPTION_KEY`
- [ ] **Task-31** Add JSON schema validation in `/backend/app/utils/skills_plugin_loader.py` for skill manifests on installation with sanitization for handler paths and dependencies

## Documentation (2 tasks)
- [ ] **Task-32** [P] Add docstrings to all new Python files: `/backend/app/model/skills.py`, `/backend/app/service/skills_service.py`, `/backend/app/controller/skills_controller.py`, `/backend/app/utils/skills_plugin_loader.py`, `/backend/app/utils/toolkit/claude_skills_toolkit.py`
- [ ] **Task-33** [P] Add JSDoc comments to TypeScript files: `/src/types/skills.ts`, `/src/api/skills.ts`, `/src/store/skillsStore.ts`

## Total: 33 tasks

---
**Note:**
- Tasks marked with [P] can be executed in parallel. Others must run sequentially.
- Each task has a unique **Task-N** identifier for better tracking and reference.
- Follow existing patterns from MCP implementation (`/backend/app/controller/tool_controller.py`, `/src/pages/Setting/MCP.tsx`) for consistency.
- Use existing Radix UI components from `/src/components/ui/` for all dialogs, forms, and inputs.
- Use existing API client functions from `/src/api/http.ts` (`proxyFetchGet`, `proxyFetchPost`, `proxyFetchPut`, `proxyFetchDelete`).
- Follow existing SSE pattern from `/backend/app/model/chat.py` (`sse_json()` function) and `/src/store/chatStore.ts` (`@microsoft/fetch-event-source`).
- Use existing Zustand pattern from `/src/store/authStore.ts` with `persist` middleware.
- All new code must follow existing style guides: Prettier for frontend, Ruff for backend.
- Skills directory structure: `~/.hachiai/skills/` with subdirectories for each skill containing `manifest.json`.
