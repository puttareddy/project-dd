# Modernization Plan

## Phase 1: Infrastructure Setup

### Task-1.1: Database Schema Extension
- [ ] Create PostgreSQL migration script for new tables: `claude_skills`, `claude_skill_configs`, `claude_skill_executions`
- [ ] Set up database indexes for performance optimization:
  - `idx_skills_user` on `claude_skills(user_id)`
  - `idx_skills_enabled` on `claude_skills(user_id, enabled)`
  - `idx_configs_user_skill` on `claude_skill_configs(user_id, skill_id)`
  - `idx_executions_user` on `claude_skill_executions(user_id)`
  - `idx_executions_task` on `claude_skill_executions(task_id)`
- [ ] Test migration on staging database with existing data integrity verification
- [ ] Document schema changes and migration rollback procedure

**File locations:**
- Migration scripts: `/backend/app/migrations/` (create new directory if needed)
- Database model definitions: `/backend/app/model/skills.py` (new file)

---

### Task-1.2: Backend Dependencies Installation
- [ ] Add new Python dependencies to `pyproject.toml`:
  - `jsonschema>=4.20.0` - Skill manifest JSON schema validation
  - `cryptography>=41.0.0` - AES-256 API key encryption (upgrade from bcrypt)
- [ ] Run `uv sync` to install new dependencies
- [ ] Verify no dependency conflicts with existing packages (FastAPI, CAMEL-AI, Pydantic)

**Reference file:**
- `/backend/pyproject.toml` (add to dependencies array)

---

### Task-1.3: Frontend Type Definitions
- [ ] Create `/src/types/skills.ts` with TypeScript interfaces:
  - `SkillManifest` - Full skill manifest definition
  - `SkillParameter` - Parameter type definition
  - `SkillConfigField` - Configuration field definition
  - `SkillExecutionSpec` - Execution specification
  - `SkillUISpec` - UI customization options
  - `SkillItem` - Installed skill database representation
  - `SkillConfig` - Skill configuration value
  - `SkillExecution` - Skill execution record
  - `SkillsState` - Zustand store state interface
- [ ] Export types for use across components

**New file:** `/src/types/skills.ts`

---

### Task-1.4: API Client Setup
- [ ] Create `/src/api/skills.ts` with API client functions:
  - `listSkills(enabledOnly?: boolean)` - GET `/api/v1/skills`
  - `discoverSkills()` - POST `/api/v1/skills/discover`
  - `installSkill(skillId: string)` - POST `/api/v1/skills/install`
  - `uninstallSkill(skillId: string)` - DELETE `/api/v1/skills/{skill_id}`
  - `enableSkill(skillId: string)` - POST `/api/v1/skills/{skill_id}/enable`
  - `disableSkill(skillId: string)` - POST `/api/v1/skills/{skill_id}/disable`
  - `getSkillConfig(skillId: string)` - GET `/api/v1/skills/{skill_id}/config`
  - `updateSkillConfig(skillId: string, config: SkillConfigRequest)` - PUT `/api/v1/skills/{skill_id}/config`
  - `executeSkill(skillId, parameters, taskId?, onMessage, onError, onClose)` - POST `/api/v1/skills/{skill_id}/execute` with SSE
  - `getSkillLogs(skillId: string)` - GET `/api/v1/skills/{skill_id}/logs`
- [ ] Use existing `proxyFetchGet`, `proxyFetchPost`, `proxyFetchPut`, `proxyFetchDelete` from `/src/api/http.ts`
- [ ] Implement SSE client using existing `@microsoft/fetch-event-source` pattern

**Reference files:**
- `/src/api/http.ts` - Use existing proxy fetch functions
- `/src/store/chatStore.ts` - Follow SSE handling pattern

---

## Phase 2: Backend Core Implementation

### Task-2.1: Skills Data Models
- [ ] Create `/backend/app/model/skills.py` with Pydantic models:
  - `SkillManifest` - Complete skill manifest with validation
  - `SkillParameter` - Parameter definition with type constraints
  - `SkillConfigField` - Configuration field with encryption flag
  - `SkillExecutionSpec` - Execution handler specification
  - `SkillUISpec` - UI customization (icon, color, category)
  - `SkillInstallRequest` - Installation request payload
  - `SkillUpdateRequest` - Update request (enable/disable)
  - `SkillConfigRequest` - Configuration update payload
  - `SkillExecutionRequest` - Execution request with parameters
  - `SkillItem` - Installed skill response model
  - `SkillConfig` - Configuration value response model
  - `SkillExecution` - Execution log response model
- [ ] Add JSON schema validation for manifest structure
- [ ] Follow existing Pydantic patterns from `/backend/app/model/chat.py`

**Reference file:**
- `/backend/app/model/chat.py` - Follow existing Pydantic patterns

---

### Task-2.2: Skills Plugin Loader
- [ ] Create `/backend/app/utils/skills_plugin_loader.py`:
  - `SkillsPluginLoader.__init__()` - Initialize with skills directory path (`~/.hachiai/skills`)
  - `discover_local_skills()` - Scan directory for `manifest.json` files
  - `load_skill(skill_id: str)` - Load specific skill manifest
  - `get_available_skills()` - Return all available manifests
  - `validate_manifest(manifest: dict)` - JSON schema validation
- [ ] Create skills directory structure at `~/.hachiai/skills/` on first run
- [ ] Log errors for invalid manifests without crashing

**New file:** `/backend/app/utils/skills_plugin_loader.py`

---

### Task-2.3: Skills Service Layer
- [ ] Create `/backend/app/service/skills_service.py`:
  - `get_user_skills(user_id: int, enabled_only: bool = False)` - Query database for user's skills
  - `load_skill_manifest(skill_id: str)` - Load and validate skill from plugin loader
  - `install_skill_to_db(skill_manifest: SkillManifest, user_id: int)` - Insert to `claude_skills` table
  - `delete_skill_from_db(skill_id: str, user_id: int)` - Remove from `claude_skills` and `claude_skill_configs`
  - `get_skill_config(skill_id: str, user_id: int)` - Retrieve configuration from `claude_skill_configs`
  - `update_skill_config(skill_id: str, user_id: int, config: dict)` - Update with encryption for API keys
  - `check_dependencies(skill_manifest: SkillManifest)` - Validate required Python packages
- [ ] Use existing `loguru` logger patterns from `/backend/app/service/chat_service.py`
- [ ] Follow database query patterns from existing services

**Reference files:**
- `/backend/app/service/chat_service.py` - Follow service layer patterns
- `/backend/app/component/encrypt.py` - Use encryption utilities

---

### Task-2.4: Skills Controller & API Endpoints
- [ ] Create `/backend/app/controller/skills_controller.py` with endpoints:
  - `GET /api/v1/skills` - List installed skills (with `enabled_only` query param)
  - `POST /api/v1/skills/discover` - Discover available skills from plugin loader
  - `POST /api/v1/skills/install` - Install skill with validation
  - `PUT /api/v1/skills/{skill_id}` - Update skill enable/disable status
  - `DELETE /api/v1/skills/{skill_id}` - Uninstall skill
  - `POST /api/v1/skills/{skill_id}/enable` - Enable specific skill
  - `POST /api/v1/skills/{skill_id}/disable` - Disable specific skill
  - `GET /api/v1/skills/{skill_id}/config` - Get skill configuration
  - `PUT /api/v1/skills/{skill_id}/config` - Update skill configuration
  - `POST /api/v1/skills/{skill_id}/execute` - Execute skill with SSE streaming
  - `GET /api/v1/skills/{skill_id}/logs` - Get skill execution logs
- [ ] Use existing FastAPI patterns from `/backend/app/controller/tool_controller.py`
- [ ] Apply existing CORS middleware configuration
- [ ] Include error handling using existing exception patterns

**Reference files:**
- `/backend/app/controller/tool_controller.py` - Follow FastAPI endpoint patterns
- `/backend/app/api/middleware/errors.py` - Use existing error handling

---

### Task-2.5: Skills Router Registration
- [ ] Create `/backend/app/api/skills/__init__.py`:
  - Import `skills_controller.py` router
  - Register router with prefix `/api/v1/skills` and tags `["skills"]`
- [ ] Update `/backend/main.py` to include skills router
- [ ] Verify router is included in FastAPI app initialization

**Reference files:**
- `/backend/app/api/chat/__init__.py` - Follow router registration pattern
- `/backend/main.py` - Update to include skills router

---

### Task-2.6: Skills Toolkit Integration
- [ ] Create `/backend/app/utils/toolkit/claude_skills_toolkit.py`:
  - `ClaudeSkillsToolkit.__init__(user_id: int, task_id: str)` - Initialize with user and task context
  - `connect()` - Load enabled skills from database and register as tools
  - `disconnect()` - Cleanup resources
  - `_register_skill(skill: SkillItem)` - Create async wrapper for skill execution
  - `_execute_skill(skill: SkillItem, params: dict)` - Execute skill via skills service
  - `get_tools()` - Return all registered skills as callable tools
- [ ] Follow existing `AbstractToolkit` pattern from `/backend/app/utils/toolkit/abstract_toolkit.py`
- [ ] Integrate with existing CAMEL-AI agent toolkit registration

**Reference files:**
- `/backend/app/utils/toolkit/abstract_toolkit.py` - Follow AbstractToolkit pattern
- `/backend/app/utils/toolkit/notion_mcp_toolkit.py` - Example toolkit implementation

---

### Task-2.7: SSE Skill Execution
- [ ] Extend `/backend/app/service/skills_service.py` with async skill execution:
  - `execute_skill_async(skill_id: str, user_id: int, task_id: str, parameters: dict)` - Async generator for SSE
  - Load skill manifest and configuration
  - Send `skill_start` event using `sse_json()` from `/backend/app/model/chat.py`
  - Execute skill handler with progress tracking
  - Send `skill_progress` events during execution
  - Send `skill_complete` event on success with output
  - Send `skill_error` event on failure with error details
  - Log execution to `claude_skill_executions` table
- [ ] Follow existing SSE patterns from `/backend/app/service/task.py`
- [ ] Use existing `Action` enum and data models for SSE events

**Reference files:**
- `/backend/app/service/task.py` - Follow SSE action patterns
- `/backend/app/model/chat.py` - Use `sse_json()` function

---

## Phase 3: Frontend State Management

### Task-3.1: Skills Zustand Store
- [ ] Create `/src/store/skillsStore.ts` following existing Zustand patterns:
  - Interface definition: `SkillsState` with:
    - `installedSkills: SkillItem[]`
    - `availableSkills: SkillManifest[]`
    - `selectedSkill: SkillItem | null`
    - `isInstalling: boolean`
    - `isConfiguring: boolean`
    - `installations: Record<string, boolean>` - Installation tracking per skill ID
    - `executions: SkillExecution[]`
  - Actions:
    - `setInstalledSkills(skills: SkillItem[])`
    - `addSkill(skill: SkillItem)`
    - `removeSkill(skillId: string)`
    - `enableSkill(skillId: string)`
    - `disableSkill(skillId: string)`
    - `updateSkillConfig(skillId: string, config: SkillConfig)`
    - `setAvailableSkills(skills: SkillManifest[])`
  - Use `persist` middleware for local storage
- [ ] Follow existing store patterns from `/src/store/authStore.ts`

**Reference files:**
- `/src/store/authStore.ts` - Follow Zustand store pattern
- `/src/store/chatStore.ts` - Use `persist` middleware

---

### Task-3.2: Frontend Routing Updates
- [ ] Update `/src/routers/index.tsx`:
  - Add lazy imports for skills pages:
    ```typescript
    const SettingSkills = lazy(() => import("@/pages/Setting/Skills"));
    const SkillsCatalog = lazy(() => import("@/pages/Setting/SkillsCatalog"));
    ```
  - Add routes under `/setting`:
    ```typescript
    <Route path="skills" element={<SettingSkills />} />
    <Route path="skills_catalog" element={<SkillsCatalog />} />
    ```
- [ ] Maintain existing protected route pattern via `ProtectedRoute`

**Reference file:**
- `/src/routers/index.tsx` - Follow existing routing pattern

---

## Phase 4: Frontend Components

### Task-4.1: Skills Management Page
- [ ] Create `/src/pages/Setting/Skills.tsx` (follows MCP.tsx pattern):
  - State management: `items`, `isLoading`, `showConfig`, `configForm`, `saving`, `showAdd`, `deleting`, `deleteTarget`, `switchLoading`
  - Data fetching: `useEffect` with `listSkills()` from `/src/api/skills.ts`
  - Render skills list with enable/disable switches
  - Configuration dialog for skill settings
  - Delete confirmation dialog
  - Add/Import skill button (links to catalog)
  - Use existing Radix UI components from `/src/components/ui/`
  - Follow existing MCP page patterns for layout and styling
- [ ] Use existing API client functions from `/src/api/skills.ts`

**Reference files:**
- `/src/pages/Setting/MCP.tsx` - Follow MCP page pattern
- `/src/pages/Setting/components/MCPList.tsx` - Reference for list component
- `/src/api/skills.ts` - Use new API client

---

### Task-4.2: Skills Catalog Page
- [ ] Create `/src/pages/Setting/SkillsCatalog.tsx`:
  - State: `skills`, `installing`, `filter` for search
  - Load available skills using `discoverSkills()` from `/src/api/skills.ts`
  - Render skills grid with cards showing:
    - Skill name, description, capabilities
    - Install button (disabled if already installed)
  - Search/filter functionality by name or category
  - Show installation progress with loading state
- [ ] Use existing card component patterns from `/src/components/ui/card.tsx`

**Reference files:**
- `/src/pages/Setting/MCPMarket.tsx` - Follow marketplace pattern
- `/src/components/ui/card.tsx` - Use existing card component

---

### Task-4.3: Skill Configuration Dialog
- [ ] Create `/src/components/Skill/SkillConfiguration.tsx`:
  - Accept `skill: SkillItem` prop
  - Load configuration using `getSkillConfig()` from `/src/api/skills.ts`
  - Render form fields based on `skill.manifest.configuration`:
    - Text input for `type: "string"`
    - Password input for `type: "api_key"` (encrypted)
    - Switch for `type: "boolean"`
    - Number input for `type: "number"`
  - Save button with loading state
  - Use existing Radix UI `Dialog` and `Input` components
  - Show toast notifications on success/failure using `sonner`

**Reference files:**
- `/src/pages/Setting/components/MCPConfigDialog.tsx` - Follow config dialog pattern
- `/src/components/ui/dialog.tsx` - Use existing dialog component
- `/src/components/ui/input.tsx` - Use existing input component
- `/src/components/ui/switch.tsx` - Use existing switch component

---

### Task-4.4: Skill Execution Monitor
- [ ] Create `/src/components/Skill/SkillExecutionMonitor.tsx`:
  - Accept `skillId: string`, `taskId?: string` props
  - Use `executeSkill()` from `/src/api/skills.ts` with SSE
  - Track execution state: `running`, `completed`, `failed`
  - Render progress indicator using existing progress component
  - Display execution output/logs
  - Cancel button for running skills
  - Error handling with toast notifications
  - Follow existing SSE patterns from `/src/store/chatStore.ts`

**Reference files:**
- `/src/store/chatStore.ts` - Follow SSE handling pattern
- `/src/components/ui/progress.tsx` - Use existing progress component

---

### Task-4.5: Skill Card Component
- [ ] Create `/src/components/Skill/SkillCard.tsx`:
  - Accept `skill: SkillManifest` or `skill: SkillItem` prop
  - Display skill name, description, and icon
  - Show capabilities as badges
  - Install/Configure buttons based on context
  - Use existing card and badge components
  - Support both catalog and manager contexts

**Reference files:**
- `/src/components/ui/card.tsx` - Use existing card component
- `/src/components/ui/badge.tsx` - Use existing badge component

---

### Task-4.6: Workflow Designer Integration
- [ ] Create `/src/components/Workflow/SkillBlock.tsx`:
  - Accept `skill: SkillItem` prop
  - Render draggable block with skill name and icon
  - Display skill description on hover
  - Support connection to other blocks (agents, skills)
  - Follow existing workflow node patterns from `/src/components/WorkFlow/node.tsx`
- [ ] Update workflow designer to include Skills palette
  - Add skills to available blocks in workflow designer

**Reference files:**
- `/src/components/WorkFlow/node.tsx` - Follow workflow node pattern
- `/src/pages/Home.tsx` - Update workflow designer integration

---

## Phase 5: Workforce Integration

### Task-5.1: Skills Toolkit Registration in Workforce
- [ ] Update `/backend/app/utils/workforce.py`:
  - Import `ClaudeSkillsToolkit` from `/backend/app/utils/toolkit/claude_skills_toolkit.py`
  - Add `claude_skills_enabled` parameter to `Workforce.__init__()` (default: `True`)
  - Add `_load_claude_skills_toolkit()` method to:
    - Create `ClaudeSkillsToolkit` instance with `user_id` and `api_task_id`
    - Call `connect()` to load enabled skills
    - Add to agent's `toolkits` list
  - Call `_load_claude_skills_toolkit()` in `Workforce.__init__()` if enabled
- [ ] Maintain existing quality evaluation and coherence checking

**Reference file:**
- `/backend/app/utils/workforce.py` - Extend existing workforce initialization

---

### Task-5.2: Agent Toolkit Integration
- [ ] Update agent initialization to include skills toolkit:
  - Pass `user_id` and `task_id` to `ClaudeSkillsToolkit`
  - Register skills toolkit alongside existing toolkits (browser, terminal, search, etc.)
  - Ensure skills are discoverable by agents during task execution
- [ ] Test agent-to-skill invocation in multi-agent workflows

**Reference files:**
- `/backend/app/utils/single_agent_worker.py` - Update agent toolkit registration
- `/backend/app/utils/toolkit/` - Reference existing toolkit patterns

---

## Phase 6: Testing & Verification

### Task-6.1: Backend Unit Tests
- [ ] Create `/backend/tests/test_skills_controller.py`:
  - Test `GET /api/v1/skills` - List installed skills
  - Test `POST /api/v1/skills/discover` - Discover available skills
  - Test `POST /api/v1/skills/install` - Install skill with valid/invalid manifest
  - Test `PUT /api/v1/skills/{skill_id}` - Enable/disable skill
  - Test `DELETE /api/v1/skills/{skill_id}` - Uninstall skill
  - Test `GET /api/v1/skills/{skill_id}/config` - Get configuration
  - Test `PUT /api/v1/skills/{skill_id}/config` - Update configuration with encryption
- [ ] Create `/backend/tests/test_skills_service.py`:
  - Test `get_user_skills()` - Filter by user_id and enabled status
  - Test `load_skill_manifest()` - Load and validate manifest
  - Test `install_skill_to_db()` - Database insertion with duplicate handling
  - Test `execute_skill_async()` - SSE streaming with skill_start, skill_progress, skill_complete events
- [ ] Use existing `pytest` patterns from `/backend/tests/`
- [ ] Verify encryption/decryption of API keys

**Reference files:**
- `/backend/tests/` - Follow existing test patterns
- `/backend/pyproject.toml` - Uses `pytest` and `pytest-asyncio`

---

### Task-6.2: Frontend Unit Tests
- [ ] Create `/src/store/__tests__/skillsStore.test.ts`:
  - Test store initialization with default state
  - Test `setInstalledSkills()` action
  - Test `addSkill()` and `removeSkill()` actions
  - Test `enableSkill()` and `disableSkill()` actions
  - Test `updateSkillConfig()` action
  - Test `persist` middleware stores state correctly
- [ ] Create `/src/pages/Setting/__tests__/Skills.test.tsx`:
  - Test component renders without crashing
  - Test skills list displays correctly
  - Test enable/disable switch triggers API calls
  - Test configuration dialog opens and saves
  - Test delete confirmation dialog works
- [ ] Use existing `vitest` patterns from `/test/`

**Reference files:**
- `/test/` - Follow existing frontend test patterns
- `package.json` - Uses `vitest` for testing

---

### Task-6.3: Integration Tests
- [ ] Create `/backend/tests/test_skills_integration.py`:
  - End-to-end skill installation flow
  - Skill execution with SSE streaming
  - Skills toolkit registration and agent invocation
  - Configuration encryption and retrieval
- [ ] Create `/test/e2e/skills.spec.ts`:
  - Browse skills catalog and install a skill
  - Configure skill with API key
  - Enable/disable skill in settings
  - Execute skill and monitor progress
  - Delete skill
- [ ] Use existing `playwright` patterns from `/test/e2e/`

**Reference files:**
- `/test/e2e/` - Follow existing Playwright e2e patterns
- `package.json` - Uses `@playwright/test` for e2e testing

---

### Task-6.4: Verification Tests
- [ ] Verify backward compatibility:
  - Run existing agent workflows with no skills installed
  - Test existing MCP tools continue to work
  - Verify SSE streaming for tasks works unchanged
  - Check StackAuth authentication still functions
- [ ] Verify skills integration:
  - Skills appear in agent toolkit
  - Agents can call skills during execution
  - Skill execution produces real-time SSE updates
  - Skill failures are isolated and don't crash platform
- [ ] Verify data integrity:
  - Skills are user-scoped (filtered by `user_id`)
  - API keys are encrypted in database
  - Skill configurations persist correctly
  - Execution logs are stored with correct task_id references

---

## Phase 7: Documentation & Polish

### Task-7.1: API Documentation
- [ ] Update FastAPI auto-generated docs:
  - Add description and example payloads for all skills endpoints
  - Document SSE event formats for skill execution
  - Include skill manifest schema documentation
  - Add authentication requirements (Bearer token from StackAuth)
- [ ] Verify docs at `/docs` endpoint in development mode

---

### Task-7.2: Inline Code Documentation
- [ ] Add docstrings to all new Python files:
  - `/backend/app/model/skills.py` - Model descriptions and validation rules
  - `/backend/app/service/skills_service.py` - Service layer function docs
  - `/backend/app/controller/skills_controller.py` - Endpoint descriptions and parameters
  - `/backend/app/utils/skills_plugin_loader.py` - Plugin loader documentation
  - `/backend/app/utils/toolkit/claude_skills_toolkit.py` - Toolkit integration docs
- [ ] Add JSDoc comments to TypeScript files:
  - `/src/types/skills.ts` - Type definitions documentation
  - `/src/api/skills.ts` - API client function documentation
  - `/src/store/skillsStore.ts` - Store state and action documentation

---

### Task-7.3: User Documentation
- [ ] Create skill manifest schema guide:
  - JSON schema structure
  - Required fields and validation rules
  - Example skill manifest with all fields
  - Parameter and configuration field types
- [ ] Document skills management UI:
  - How to browse and install skills
  - How to configure skill settings
  - How to enable/disable skills
  - How to use skills in workflows
- [ ] Document skill development:
  - Packaging skills as manifest + handler
  - Local skills directory structure
  - Testing skills before installation

---

## Phase 8: Security & Performance

### Task-8.1: Security Implementation
- [ ] Implement API key encryption:
  - Use `cryptography` library for AES-256 encryption
  - Generate or load encryption key from `SKILLS_ENCRYPTION_KEY` environment variable
  - Encrypt values with `type: "api_key"` before storing to database
  - Decrypt values when retrieving for skill execution
- [ ] Add skill manifest validation:
  - JSON schema validation on installation
  - Sanitize skill handler paths to prevent path traversal
  - Validate dependencies don't contain malicious packages
- [ ] Add audit logging:
  - Log all skill installations with `user_id`, `skill_id`, timestamp
  - Log skill executions with `task_id`, `parameters`, status
  - Log configuration changes with before/after values

**Reference file:**
- `/backend/app/component/encrypt.py` - Existing encryption utilities

---

### Task-8.2: Performance Optimization
- [ ] Database query optimization:
  - Verify indexes are created for all foreign keys
  - Test queries with 100+ installed skills
  - Optimize configuration retrieval with JOIN queries
- [ ] Skills loading optimization:
  - Cache available skills manifest in memory
  - Lazy load skill configurations
  - Limit concurrent skill executions to prevent resource exhaustion
- [ ] Frontend performance:
  - Virtualize skills list if > 100 items
  - Lazy load skill configurations
  - Debounce search/filter inputs in catalog

---

## Phase 9: Deployment

### Task-9.1: Build Configuration
- [ ] Update Electron build config:
  - Ensure skills storage directory is included in app bundle
  - Skills configurations persist across app updates
  - Auto-updater preserves skills data
- [ ] Update backend startup script:
  - Create `~/.hachiai/skills` directory on first run
  - Load skills plugin loader on startup
  - Log skills initialization status

**Reference files:**
- `electron-builder.yml` - Update build configuration
- `/backend/main.py` - Add skills initialization

---

### Task-9.2: Production Deployment
- [ ] Run database migrations on production database
- [ ] Verify all endpoints return correct responses
- [ ] Test skills integration with CAMEL-AI agents
- [ ] Monitor for errors in skill execution logs
- [ ] Set up logging aggregation for skills API

---

## Rollback Plan

If issues arise during deployment, execute rollback in reverse order:

1. **Rollback Phase 9:**
   - Revert `/backend/main.py` to remove skills router
   - Revert Electron build config changes

2. **Rollback Phase 8:**
   - Remove encryption key from environment
   - Disable skills endpoints in database

3. **Rollback Phase 7:**
   - Remove new documentation files

4. **Rollback Phase 6:**
   - Remove test files from `/backend/tests/` and `/test/`

5. **Rollback Phase 5:**
   - Revert `/backend/app/utils/workforce.py` changes
   - Remove `ClaudeSkillsToolkit` import

6. **Rollback Phase 4:**
   - Remove frontend component files from `/src/components/Skill/`
   - Remove frontend page files from `/src/pages/Setting/`
   - Revert `/src/routers/index.tsx` changes

7. **Rollback Phase 3:**
   - Remove `/src/store/skillsStore.ts`
   - Revert `/src/routers/index.tsx` changes

8. **Rollback Phase 2:**
   - Remove `/backend/app/api/skills/` directory
   - Remove `/backend/app/controller/skills_controller.py`
   - Remove `/backend/app/service/skills_service.py`
   - Remove `/backend/app/model/skills.py`
   - Remove `/backend/app/utils/skills_plugin_loader.py`
   - Remove `/backend/app/utils/toolkit/claude_skills_toolkit.py`
   - Revert `/backend/main.py` router registration

9. **Rollback Phase 1:**
   - Run database migration to drop new tables:
     - `DROP TABLE IF EXISTS claude_skill_executions;`
     - `DROP TABLE IF EXISTS claude_skill_configs;`
     - `DROP TABLE IF EXISTS claude_skills;`
   - Remove new dependencies from `/backend/pyproject.toml`
   - Remove `/src/types/skills.ts`
   - Remove `/src/api/skills.ts`

---

## Success Criteria

Phase completion is verified when:

- [ ] Database migration runs successfully on staging and production
- [ ] All skills API endpoints return correct responses with test data
- [ ] Skills can be installed, configured, enabled, disabled, and deleted from UI
- [ ] Skills appear in agent toolkit and can be invoked during task execution
- [ ] Skill execution produces real-time SSE updates matching existing task format
- [ ] API keys are encrypted in database and correctly decrypted for execution
- [ ] Frontend components render correctly and pass all unit tests
- [ ] Integration tests verify end-to-end skill workflow
- [ ] Existing agents, workflows, and MCP tools continue to work unchanged
- [ ] All code follows existing style guides (Prettier, ESLint, Ruff)
- [ ] Documentation is complete and accurate

---

## Notes & Considerations

- **Backward Compatibility:** All changes are additive. No existing functionality is removed or modified.
- **Database Migrations:** Use incremental migration files if using Alembic, or direct SQL scripts.
- **Error Handling:** Follow existing error response format with `code`, `text`, `error` fields.
- **SSE Format:** Use `sse_json(step, data)` from `/backend/app/model/chat.py` for consistency.
- **State Management:** Follow existing Zustand patterns with `persist` middleware and `subscribeWithSelector`.
- **UI Components:** Use existing Radix UI components and Tailwind CSS classes.
- **Toast Notifications:** Use `sonner` library for all user feedback messages.
- **Type Safety:** All new Python code uses type hints; all new TypeScript uses interfaces.
- **Testing:** Run existing test suite before and after changes to verify no regressions.
- **Cross-Platform:** Ensure skills directory works on macOS and Windows Electron builds.
