# Modernization Specification

## Overview

HachiAI is a multi-agent AI workforce platform built on React, FastAPI, and CAMEL-AI that enables users to orchestrate AI agents (Developer, Search, Document, Multi-Modal) with tools for task automation. The platform provides real-time task execution visibility via SSE streaming, browser automation through Chrome DevTools Protocol, terminal execution, and MCP (Model Context Protocol) tool integrations.

The Claude Skills Integration modernization extends HachiAI from a fixed-agent platform to an extensible workforce ecosystem. This enables users to discover, install, configure, and manage ANY Claude skill as a callable tool within multi-agent workflows, dynamically integrating with the existing CAMEL-AI Workforce Manager, SSE streaming infrastructure, and StackAuth authentication system.

The modernization transforms HachiAI by adding a universal plugin system for Claude skills while preserving all existing multi-agent architecture, real-time streaming, toolkits, and UI patterns. Skills become first-class citizens alongside MCP tools, appearing in the workflow designer and agent toolkit registry without requiring platform code changes.

## Current Architecture

### Frontend Layer (React + TypeScript + Electron)
**Location:** `/Users/puttaiaharugunta/codebase/hachi/ai-agent-platform/src`

**Structure:**
- **Pages:** Home (`/src/pages/Home`), History (`/src/pages/History`), Iris (`/src/pages/Iris`), Setting (`/src/pages/Setting`), Login, SignUp
- **State Management:** Zustand 5.0.4 stores in `/src/store/`:
  - `chatStore.ts` - Task execution state, SSE handling, messages, progress tracking
  - `authStore.ts` - StackAuth OIDC tokens, user session, worker list, settings
  - `globalStore.ts` - Application-wide state
  - `installationStore.ts` - Installation tracking
- **Components:** `/src/components/` includes ChatBox, TaskCard, Folder, File browser, Terminal, Workflow designer, Dialog components
- **Routing:** `/src/routers/index.tsx` with React Router DOM, protected routes pattern
- **Authentication:** `/src/auth/` using `@stackframe/react` and `react-oidc-context` for StackAuth OIDC
- **Real-time:** SSE streaming via `@microsoft/fetch-event-source`
- **Build:** Vite 5.4.11, Electron 33.2.0 desktop wrapper, electron-builder 24.13.3

### Backend Layer (FastAPI + Python)
**Location:** `/Users/puttaiaharugunta/codebase/hachi/ai-agent-platform/backend`

**Structure:**
- **API Routes:** `/backend/app/api/` - FastAPI routers organized by domain:
  - `health/` - Health check endpoints
  - `chat/` - Chat session, task streaming (`POST /api/v1/chat`)
  - `tasks/` - Task management endpoints
  - `agents/` - Agent management
  - `mcp/` - MCP tool integrations
  - `learning/` - Learning system
  - `models/` - Model configuration
- **Controllers:** `/backend/app/controller/` - HTTP request handlers:
  - `chat_controller.py` - Chat session initiation, SSE streaming response
  - `task_controller.py` - Task lifecycle management
  - `model_controller.py` - Model configuration
  - `tool_controller.py` - MCP tool installation endpoints
  - `irex_controller.py`, `learning_controller.py`, `health_controller.py`
- **Services:** `/backend/app/service/` - Business logic layer
  - `chat_service.py` - Chat processing, step solving
  - `task.py` - Task queue, SSE action handling, TaskLock management
  - `credits_service.py`, `guardrail_service.py`, `learning_service.py`
- **Models:** `/backend/app/model/chat.py` - Pydantic data models:
  - `Chat` - Main task request model with agent config, MCP tools, environment
  - `ChatHistory` - Message history
  - `Status` enum - `confirming`, `confirmed`, `processing`, `done`
  - `NewAgent` - Custom agent definitions
  - `SupplementChat`, `HumanReply`, `TaskContent`, `UpdateData`
  - `McpServers` - MCP server configurations
  - `sse_json(step, data)` - SSE event formatter
- **Components:** `/backend/app/component/` - Shared utilities
  - `app_auth.py` - Auth helpers
  - `code.py` - Code execution
  - `command.py` - Shell commands
  - `encrypt.py` - Password hashing with bcrypt
  - `environment.py` - Environment configuration
  - `pydantic/i18n.py` - Internationalized Pydantic
- **Utils:** `/backend/app/utils/`
  - `workforce.py` - CAMEL-AI Workforce Manager wrapper with custom quality evaluation
  - `single_agent_worker.py` - Single agent execution worker
  - `toolkit/abstract_toolkit.py` - Abstract toolkit base class
  - `toolkit/*.py` - 30+ toolkit implementations (search_toolkit, terminal_toolkit, hybrid_browser_toolkit, github_toolkit, notion_mcp_toolkit, google_gmail_mcp_toolkit, etc.)
- **Entry Point:** `/backend/main.py` - FastAPI app initialization with CORS middleware
- **Python Version:** 3.10.16, FastAPI ≥0.115.12, CAMEL-AI ≥0.2.76a6, <0.2.79

### AI Workforce Engine (CAMEL-AI)
**Location:** `/backend/app/utils/workforce.py`

**Architecture:**
- **Workforce Manager:** Custom `Workforce` class extending CAMEL-AI `BaseWorkforce`
- **Agent Pool:** Manages agent types (Developer, Search, Document, Multi-Modal)
- **Agent Types:** `Developer_Agent`, `Search_Agent`, `Document_Agent`, `MultiModal_Agent`
- **Toolkits:** AbstractToolkit pattern for browser, terminal, file, search, MCP tools
- **Quality Evaluation:** Custom `_analyze_task()` with LLM-based quality checking (configurable via `CAMEL_QUALITY_CHECK_ENABLED`, `CAMEL_QUALITY_THRESHOLD`)
- **Coherence Checking:** Cross-subtask dependency overlap detection

### Database (PostgreSQL)
**Existing Tables:**
- `chat_history` - Main task/workflow records with metadata
- `chat_step` - Individual execution steps with JSON data
- `chat_snapshot` - Browser screenshots and visual state
- `user` - User management (StackAuth integration)
- `mcp_user` - MCP tool configurations and API keys

**Note:** No explicit schema files found - database likely managed through ORM or direct SQL queries.

### Real-Time Streaming (SSE)
**Implementation:**
- SSE endpoint: `POST /api/v1/chat` returns `StreamingResponse` with `media_type="text/event-stream"`
- SSE format: `sse_json(step, data)` in `/backend/app/model/chat.py` produces `data: {"step": "...", "data": {...}}\n\n`
- Frontend consumer: `@microsoft/fetch-event-source` in `chatStore.ts`
- Action types in `/backend/app/service/task.py`:
  - `ActionAssignTaskData`, `ActionEndData`, `ActionNoticeData`, `ActionTaskStateData`
  - `ActionImproveData`, `ActionInstallMcpData`, `ActionStopData`, `ActionSupplementData`

### Existing MCP Tool System
**Components:**
- Model: `McpServers` type in `chat.py` - `{"mcpServers": {"server_name": {...}}}`
- Controller: `/backend/app/controller/tool_controller.py` with endpoints:
  - `POST /install/tool/{tool}` - Install and pre-authenticate MCP tools
  - `GET /tools/available` - List available MCP tools
- Toolkit Pattern: `AbstractToolkit` in `/backend/app/utils/toolkit/abstract_toolkit.py`
- Example Toolkits:
  - `NotionMCPToolkit` in `notion_mcp_toolkit.py`
  - `GoogleCalendarToolkit` in `google_calendar_toolkit.py`
  - `GoogleGmailMCPToolkit` in `google_gmail_mcp_toolkit.py`
  - `NotionMCPToolkit` in `notion_mcp_toolkit.py`

### UI Components and Patterns
**Dialog Components:**
- Radix UI dialogs in `/src/components/Dialog/` - AddWorkerDialog, AgentQuestionDialog, DeleteAgentModal, Privacy, CloseNotice
- Toast notifications via `sonner` 2.0.6

**MCP Management UI (Reference Pattern for Skills):**
- Location: `/src/pages/Setting/MCP.tsx`
- Components:
  - `MCPList.tsx` - List configured MCP tools
  - `MCPConfigDialog.tsx` - Configure MCP settings
  - `MCPAddDialog.tsx` - Add new MCP tool
  - `MCPDeleteDialog.tsx` - Delete MCP tool
- State management: `items`, `showConfig`, `configForm`, `saving`, `showAdd`, `installing`, `deleting`
- API calls: `proxyFetchGet`, `proxyFetchDelete`, `proxyFetchPost`, `proxyFetchPut` from `/src/api/http.ts`

**Routing Structure:**
- Protected routes via `ProtectedRoute` component (checks `authStore.token`)
- IREX route guarded by role check (`role !== "business"`)
- Setting sub-routes: `/setting/general`, `/setting/privacy`, `/setting/models`, `/setting/api`, `/setting/mcp`, `/setting/toolkit`, `/setting/billing`, `/setting/tools`, `/setting/mcp_market`, `/setting/toolkit_market`, etc.

## Target Architecture

### Extension Strategy: Hybrid Integration
The Skills system extends the existing architecture without breaking changes. New components are added alongside existing ones, following established patterns.

### Frontend Extensions
**New Pages:**
- `/src/pages/Setting/Skills.tsx` - Main Skills management page (follows MCP.tsx pattern)
- `/src/pages/Setting/SkillsCatalog.tsx` - Browse and discover available skills
- `/src/pages/Setting/SkillsMarket.tsx` - Skills marketplace (future)

**New Components:**
- `/src/components/Skill/SkillsCatalog.tsx` - Skills discovery and browsing
- `/src/components/Skill/SkillsManager.tsx` - Manage installed skills
- `/src/components/Skill/SkillConfiguration.tsx` - Configure skill-specific settings
- `/src/components/Skill/SkillExecutionMonitor.tsx` - Monitor skill execution with SSE
- `/src/components/Skill/SkillCard.tsx` - Skill card in catalog/manager
- `/src/components/Workflow/SkillBlock.tsx` - Skill block in workflow designer (alongside agent blocks)

**New State Management:**
- `/src/store/skillsStore.ts` - Skills state following Zustand pattern:
  ```typescript
  interface SkillsState {
    installedSkills: SkillItem[];
    availableSkills: SkillManifest[];
    selectedSkill: SkillItem | null;
    isInstalling: boolean;
    isConfiguring: boolean;
    executions: SkillExecution[];
    setInstalledSkills: (skills: SkillItem[]) => void;
    addSkill: (skill: SkillItem) => void;
    removeSkill: (skillId: string) => void;
    enableSkill: (skillId: string) => void;
    disableSkill: (skillId: string) => void;
    updateSkillConfig: (skillId: string, config: SkillConfig) => void;
  }
  ```

**Routing Updates:**
- Add to `/src/routers/index.tsx`:
  ```typescript
  const SettingSkills = lazy(() => import("@/pages/Setting/Skills"));
  const SkillsCatalog = lazy(() => import("@/pages/Setting/SkillsCatalog"));

  // In routes:
  <Route path="skills" element={<SettingSkills />} />
  <Route path="skills_catalog" element={<SkillsCatalog />} />
  ```

### Backend Extensions
**New API Routes:**
- `/backend/app/api/skills/` - Skills domain router
  - `__init__.py` - Router definition with prefix `/api/v1/skills`
  - `router.py` or `__init__.py` - Endpoint definitions

**New Controller:**
- `/backend/app/controller/skills_controller.py` - Skills CRUD endpoints:
  ```python
  @router.get("/", name="list installed skills")
  async def list_skills(user_id: int):
      """Get all installed skills for user"""

  @router.post("/discover", name="discover available skills")
  async def discover_skills():
      """Get available skills from catalog (local or remote)"""

  @router.post("/install", name="install skill")
  async def install_skill(request: SkillInstallRequest):
      """Install a skill, validate manifest, save to database"""

  @router.put("/{skill_id}", name="update skill")
  async def update_skill(skill_id: str, request: SkillUpdateRequest):
      """Update skill configuration"""

  @router.delete("/{skill_id}", name="uninstall skill")
  async def uninstall_skill(skill_id: str, user_id: int):
      """Uninstall a skill"""

  @router.post("/{skill_id}/enable", name="enable skill")
  async def enable_skill(skill_id: str, user_id: int):

  @router.post("/{skill_id}/disable", name="disable skill")
  async def disable_skill(skill_id: str, user_id: int):

  @router.get("/{skill_id}/config", name="get skill config")
  async def get_skill_config(skill_id: str, user_id: int):

  @router.put("/{skill_id}/config", name="update skill config")
  async def update_skill_config(skill_id: str, user_id: int, config: SkillConfigRequest):

  @router.post("/{skill_id}/execute", name="execute skill")
  async def execute_skill(skill_id: str, request: SkillExecutionRequest):
      """Execute skill with SSE streaming"""

  @router.get("/{skill_id}/logs", name="get skill logs")
  async def get_skill_logs(skill_id: str, user_id: int):
  ```

**New Service:**
- `/backend/app/service/skills_service.py` - Skills business logic:
  - `load_skill_manifest(skill_id)` - Load and validate skill manifest
  - `install_skill_to_db(skill_manifest, user_id)` - Persist to database
  - `get_user_skills(user_id)` - Retrieve installed skills
  - `get_skill_config(skill_id, user_id)` - Retrieve configuration
  - `update_skill_config(skill_id, user_id, config)` - Update with encryption
  - `execute_skill_async(skill_id, parameters)` - Async skill execution with SSE
  - `validate_manifest(manifest)` - JSON schema validation

**New Models:**
- `/backend/app/model/skills.py` - Skills data models:
  ```python
  from pydantic import BaseModel, Field
  from typing import Optional, List, Dict, Any, Literal

  class SkillManifest(BaseModel):
      """Claude skill manifest definition"""
      skill_id: str = Field(..., description="Unique skill identifier (e.g., 'claude://skills/commit')")
      name: str = Field(..., description="Display name")
      description: str = Field(..., description="Skill description")
      version: str = Field(..., description="Semantic version")
      author: str = Field(default="Unknown", description="Skill author")
      capabilities: List[str] = Field(default_factory=list, description="Skill capabilities")
      dependencies: List[str] = Field(default_factory=list, description="Required dependencies")
      parameters: List[SkillParameter] = Field(default_factory=list, description="Parameters")
      configuration: List[SkillConfigField] = Field(default_factory=list, description="Configuration options")
      execution: SkillExecutionSpec = Field(..., description="Execution specification")
      ui: SkillUISpec = Field(default_factory=lambda: {}, description="UI customization")

  class SkillParameter(BaseModel):
      """Skill parameter definition"""
      name: str
      type: Literal["string", "number", "boolean", "array", "object"]
      description: str
      required: bool = False
      default: Any = None

  class SkillConfigField(BaseModel):
      """Skill configuration field"""
      key: str
      type: Literal["string", "number", "boolean", "api_key"]
      default: Any
      description: str
      encrypted: bool = Field(default=False, description="Whether to encrypt this value")

  class SkillExecutionSpec(BaseModel):
      """Skill execution specification"""
      type: Literal["local", "remote", "mcp"]
      handler: str = Field(..., description="Execution handler path")

  class SkillUISpec(BaseModel):
      """Skill UI customization"""
      icon: Optional[str] = None
      color: Optional[str] = None
      category: Optional[str] = None

  # Database Models
  class SkillInstallRequest(BaseModel):
      skill_id: str
      user_id: int

  class SkillUpdateRequest(BaseModel):
      enabled: Optional[bool] = None

  class SkillConfigRequest(BaseModel):
      config: Dict[str, Any]

  class SkillExecutionRequest(BaseModel):
      parameters: Dict[str, Any]
      task_id: Optional[str] = None  # For linking to task

  # API Response Models
  class SkillItem(BaseModel):
      id: int
      user_id: int
      skill_id: str
      name: str
      description: str
      version: str
      manifest: SkillManifest
      enabled: bool
      installed_at: str
      updated_at: str

  class SkillConfig(BaseModel):
      config_key: str
      config_value: Any
      is_encrypted: bool
  ```

**New Plugin Loader:**
- `/backend/app/utils/skills_plugin_loader.py` - Dynamic skill loading:
  ```python
  import json
  from pathlib import Path
  from typing import Dict, Optional
  from loguru import logger

  class SkillsPluginLoader:
      """Load and manage Claude skill plugins"""

      def __init__(self, skills_dir: Path = None):
          self.skills_dir = skills_dir or (Path.home() / ".hachiai" / "skills")
          self.skills_dir.mkdir(parents=True, exist_ok=True)
          self._manifests: Dict[str, SkillManifest] = {}

      def discover_local_skills(self) -> Dict[str, SkillManifest]:
          """Scan skills directory for manifest.json files"""
          self._manifests.clear()
          for skill_path in self.skills_dir.iterdir():
              manifest_file = skill_path / "manifest.json"
              if manifest_file.exists():
                  try:
                      with open(manifest_file) as f:
                          manifest = json.load(f)
                      validated = SkillManifest(**manifest)
                      self._manifests[validated.skill_id] = validated
                  except Exception as e:
                      logger.error(f"Failed to load skill manifest from {manifest_file}: {e}")
          return self._manifests

      def load_skill(self, skill_id: str) -> Optional[SkillManifest]:
          """Load specific skill manifest"""
          return self._manifests.get(skill_id)

      def get_available_skills(self) -> List[SkillManifest]:
          """Get all available skill manifests"""
          return list(self._manifests.values())
  ```

**New Toolkit Integration:**
- `/backend/app/utils/toolkit/claude_skills_toolkit.py` - Skills toolkit for agents:
  ```python
  from app.utils.toolkit.abstract_toolkit import AbstractToolkit

  class ClaudeSkillsToolkit(AbstractToolkit):
      """Toolkit exposing installed Claude skills as agent tools"""

      def __init__(self, user_id: int, task_id: str):
          super().__init__()
          self.user_id = user_id
          self.task_id = task_id
          self._skills: Dict[str, Callable] = {}

      async def connect(self):
          """Load enabled skills for user"""
          from app.service.skills_service import get_user_skills
          skills = await get_user_skills(self.user_id)
          for skill in skills:
              if skill.enabled:
                  self._register_skill(skill)

      def _register_skill(self, skill: SkillItem):
          """Register skill as callable tool"""
          self._skills[skill.skill_id] = self._create_skill_wrapper(skill)

      def _create_skill_wrapper(self, skill: SkillItem) -> Callable:
          """Create async wrapper for skill execution"""
          async def wrapper(**kwargs):
              return await self._execute_skill(skill, kwargs)
          wrapper.__name__ = skill.skill_id
          return wrapper

      async def _execute_skill(self, skill: SkillItem, params: Dict):
          """Execute skill with SSE streaming"""
          from app.service.skills_service import execute_skill_async
          return await execute_skill_async(
              skill.skill_id,
              self.user_id,
              self.task_id,
              params
          )

      def get_tools(self) -> List[Callable]:
          """Return all registered skills as tools"""
          return list(self._skills.values())
  ```

### Database Schema Extensions

**New Tables:**

```sql
-- Skills catalog (installed skills)
CREATE TABLE claude_skills (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES user(id),
    skill_id VARCHAR(255) UNIQUE NOT NULL,  -- Skill identifier (e.g., 'claude://skills/commit')
    name VARCHAR(255) NOT NULL,
    description TEXT,
    version VARCHAR(50),
    author VARCHAR(255),
    manifest JSONB NOT NULL,  -- Full skill manifest (SkillManifest)
    enabled BOOLEAN DEFAULT TRUE,
    installed_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Skill configurations (encrypted values)
CREATE TABLE claude_skill_configs (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES user(id),
    skill_id VARCHAR(255) NOT NULL REFERENCES claude_skills(skill_id),
    config_key VARCHAR(255) NOT NULL,
    config_value_encrypted TEXT,  -- Encrypted value for sensitive data
    config_value_text TEXT,  -- Plain text for non-sensitive data
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, skill_id, config_key)
);

-- Skill execution logs
CREATE TABLE claude_skill_executions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES user(id),
    skill_id VARCHAR(255) NOT NULL REFERENCES claude_skills(skill_id),
    task_id VARCHAR(255) REFERENCES chat_history(id),
    status VARCHAR(50),  -- 'running', 'completed', 'failed', 'cancelled'
    parameters JSONB,  -- Execution parameters
    started_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP,
    error_message TEXT,
    output JSONB
);

-- Performance indexes
CREATE INDEX idx_skills_user ON claude_skills(user_id);
CREATE INDEX idx_skills_enabled ON claude_skills(user_id, enabled);
CREATE INDEX idx_configs_user_skill ON claude_skill_configs(user_id, skill_id);
CREATE INDEX idx_executions_user ON claude_skill_executions(user_id);
CREATE INDEX idx_executions_task ON claude_skill_executions(task_id);
```

### Workforce Integration

**Skill-to-Agent Mapping:**
- Extend `/backend/app/utils/workforce.py` `Workforce.__init__()`:
  ```python
  def __init__(
      self,
      *,
      claude_skills_enabled: bool = True,
      **kwargs
  ):
      super().__init__(**kwargs)
      self.claude_skills_enabled = claude_skills_enabled
      if self.claude_skills_enabled:
          self._load_claude_skills_toolkit()

  def _load_claude_skills_toolkit(self):
      """Load Claude Skills toolkit into agent toolkits"""
      from app.utils.toolkit.claude_skills_toolkit import ClaudeSkillsToolkit
      self.claude_skills_toolkit = ClaudeSkillsToolkit(
          user_id=self.user_id,
          task_id=self.api_task_id
      )
      # Add to agent's available toolkits
      self.task_agent.toolkits.append(self.claude_skills_toolkit)
  ```

**SSE Integration for Skill Execution:**
- Follow existing SSE pattern in `/backend/app/service/task.py`:
  ```python
  async def execute_skill_async(
      skill_id: str,
      user_id: int,
      task_id: str,
      parameters: Dict
  ) -> AsyncGenerator[str, None, None]:
      """Execute skill with SSE streaming"""

      # Load skill manifest
      manifest = await load_skill_manifest(skill_id)

      # Load skill configuration
      config = await get_skill_config(skill_id, user_id)

      # Send starting event
      yield sse_json("skill_start", {
          "skill_id": skill_id,
          "name": manifest.name,
          "status": "running"
      })

      try:
          # Execute skill handler
          handler = load_handler(manifest.execution.handler)
          result = await handler(**parameters, config=config)

          # Send progress events during execution
          yield sse_json("skill_progress", {
              "skill_id": skill_id,
              "progress": result.get("progress", 100),
              "step": result.get("step", "")
          })

          # Send completion event
          yield sse_json("skill_complete", {
              "skill_id": skill_id,
              "status": "completed",
              "output": result.get("output")
          })

      except Exception as e:
          # Send error event
          yield sse_json("skill_error", {
              "skill_id": skill_id,
              "status": "failed",
              "error": str(e)
          })
  ```

### Skill Manifest Schema

**JSON Schema for Skill Definition:**

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Claude Skill Manifest",
  "type": "object",
  "required": ["skill_id", "name", "description", "version", "execution"],
  "properties": {
    "skill_id": {
      "type": "string",
      "pattern": "^claude://[a-z0-9_\\-]+/[a-z0-9_\\-]+$",
      "description": "Unique skill identifier following claude:// scheme"
    },
    "name": {
      "type": "string",
      "maxLength": 255,
      "description": "Human-readable skill name"
    },
    "description": {
      "type": "string",
      "maxLength": 1000,
      "description": "Detailed description of skill capabilities"
    },
    "version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$",
      "description": "Semantic version (e.g., 1.0.0)"
    },
    "author": {
      "type": "string",
      "default": "Unknown",
      "description": "Skill author or organization"
    },
    "capabilities": {
      "type": "array",
      "items": { "type": "string" },
      "default": [],
      "description": "List of capability keywords"
    },
    "dependencies": {
      "type": "array",
      "items": { "type": "string" },
      "default": [],
      "description": "Required Python packages or system dependencies"
    },
    "parameters": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "type": { "enum": ["string", "number", "boolean", "array", "object"] },
          "description": { "type": "string" },
          "required": { "type": "boolean", "default": false },
          "default": {}
        },
        "required": ["name", "type", "description"]
      },
      "default": [],
      "description": "Parameters accepted by skill execution"
    },
    "configuration": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "key": { "type": "string" },
          "type": { "enum": ["string", "number", "boolean", "api_key"] },
          "default": {},
          "description": { "type": "string" },
          "encrypted": { "type": "boolean", "default": false }
        },
        "required": ["key", "type", "description"]
      },
      "default": [],
      "description": "Configurable options for the skill"
    },
    "execution": {
      "type": "object",
      "properties": {
        "type": { "enum": ["local", "remote", "mcp"] },
        "handler": { "type": "string", "format": "uri" }
      },
      "required": ["type", "handler"],
      "description": "Execution specification"
    },
    "ui": {
      "type": "object",
      "properties": {
        "icon": { "type": "string", "format": "uri" },
        "color": { "type": "string", "format": "color" },
        "category": { "type": "string" }
      },
      "default": {},
      "description": "UI customization for the skill"
    }
  }
}
```

**Example Skill Manifest:**

```json
{
  "skill_id": "claude://skills/commit",
  "name": "Commit",
  "description": "Create git commits with intelligent message generation",
  "version": "1.0.0",
  "author": "Anthropic",
  "capabilities": [
    "git_operations",
    "message_generation"
  ],
  "dependencies": [],
  "parameters": [
    {
      "name": "message",
      "type": "string",
      "description": "Commit message (optional, will be generated if not provided)",
      "required": false
    },
    {
      "name": "files",
      "type": "array",
      "description": "Specific files to commit (optional, commits all staged if not provided)",
      "required": false
    }
  ],
  "configuration": [
    {
      "key": "sign_commits",
      "type": "boolean",
      "default": false,
      "description": "Enable GPG signing"
    }
  ],
  "execution": {
    "type": "local",
    "handler": "hachiai.skills.commit.execute"
  },
  "ui": {
    "icon": "git-commit",
    "color": "#00D9FF",
    "category": "Development"
  }
}
```

### Skill Storage and Loading

**Skills Directory Structure:**
```
~/.hachiai/skills/
├── claude://skills/commit/
│   ├── manifest.json
│   ├── __init__.py
│   └── execute.py
├── claude://skills/pdf/
│   ├── manifest.json
│   ├── __init__.py
│   └── execute.py
└── claude://skills/simplify/
    ├── manifest.json
    ├── __init__.py
    └── execute.py
```

**Skill Execution Handler Template:**
```python
# File: hachiai/skills/commit/execute.py
import subprocess
from pathlib import Path

async def execute(
    message: str | None = None,
    files: list | None = None,
    sign_commits: bool = False,
    config: dict = {}
) -> dict:
    """Execute git commit skill

    Args:
        message: Commit message (will be generated if not provided)
        files: Specific files to commit (None = all staged)
        sign_commits: Enable GPG signing
        config: Skill configuration (includes API keys if needed)

    Returns:
        Dict with execution result and progress
    """
    # Generate message if not provided
    if not message:
        message = await _generate_commit_message()

    # Build git command
    cmd = ["git", "commit", "-m", message]
    if sign_commits:
        cmd.append("-S")

    # Execute command
    result = subprocess.run(cmd, capture_output=True, text=True)

    return {
        "status": "completed",
        "output": result.stdout,
        "progress": 100,
        "step": "Created commit"
    }

async def _generate_commit_message() -> str:
    """Generate intelligent commit message using LLM"""
    from openai import OpenAI
    client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))

    diff = subprocess.run(["git", "diff", "--cached"], capture_output=True, text=True).stdout

    response = await client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "Generate concise git commit message"},
            {"role": "user", "content": f"Changes:\n{diff}"}
        ]
    )

    return response.choices[0].message.content
```

### Authentication Integration

**Existing StackAuth Pattern:**
- Frontend: `@stackframe/react` + `react-oidc-context` in `/src/auth/`
- Token management: `authStore.ts` stores `token`, `user_id`, `email`, `role`
- Protected routes: `ProtectedRoute` component checks `authStore.token`
- Backend: No explicit auth enforcement (to be added for skills endpoints)

**Skills-Specific Auth:**
- API key encryption: Use existing `bcrypt` from `/backend/app/component/encrypt.py` or upgrade to AES-256 for API keys
- User-level skills: Filter by `user_id` in all skill queries
- Configuration encryption: Sensitive config values (type="api_key") encrypted in `claude_skill_configs.config_value_encrypted`

### External Dependencies

**New Dependencies:**
- Python (backend):
  - `jsonschema` - For skill manifest validation
  - `cryptography` or `pycryptodome` - For AES-256 encryption of API keys (upgrade from bcrypt)
  - `aiofiles` - Already present for async file operations

- TypeScript (frontend):
  - No new dependencies (use existing Radix UI, Zustand, react-oidc-context)

**Existing Dependencies to Preserve:**
- `camel-ai[eigent]>=0.2.76a6, <0.2.79` - Multi-agent framework
- `fastapi>=0.115.12` - API framework
- `loguru>=0.7.3` - Logging
- `@microsoft/fetch-event-source` - SSE client
- `@stackframe/react`, `react-oidc-context` - Authentication
- `zustand` - State management
- Radix UI components - Dialogs, forms, switches

## Data Models

### Existing Data Models

**Location:** `/backend/app/model/chat.py`

```python
# Status Enum
class Status(str, Enum):
    confirming = "confirming"
    confirmed = "confirmed"
    processing = "processing"
    done = "done"

# Message History
class ChatHistory(BaseModel):
    role: RoleType  # from camel.types
    content: str

# MCP Servers Configuration
McpServers = dict[Literal["mcpServers"], dict[str, dict]]

# Main Chat/Task Request Model
class Chat(BaseModel):
    task_id: str
    question: str
    email: str
    attaches: list[str] = []
    model_platform: str
    model_type: str
    api_key: str
    api_url: str | None = None
    language: str = "en"
    browser_port: int = 9222
    max_retries: int = 3
    allow_local_system: bool = False
    installed_mcp: McpServers = {"mcpServers": {}}
    bun_mirror: str = ""
    uvx_mirror: str = ""
    env_path: str | None = None
    summary_prompt: str = (...)
    new_agents: list["NewAgent"] = []
    extra_params: dict | None = None
    auth_token: str | None = None  # Bearer token for Nexus

    # Per-agent provider configuration
    agent_providers: dict[str, AgentProviderConfig] = Field(default_factory=dict)

    # Template execution fields
    template_mode: bool = False
    template_subtasks: list[dict] | None = None
    template_summary: str | None = None
    template_id: int | None = None
    source_task_id: str | None = None  # For re-execute tracking

# Agent Provider Configuration
class AgentProviderConfig(BaseModel):
    agent_name: str
    model_platform: str
    model_type: str
    api_key: str
    api_url: str | None = None
    extra_params: dict | None = None

# Custom Agent Definition
class NewAgent(BaseModel):
    name: str
    description: str
    tools: list[str]
    mcp_tools: McpServers | None
    env_path: str | None = None

# Supplemental Chat Input
class SupplementChat(BaseModel):
    question: str

# Human Reply for Agent
class HumanReply(BaseModel):
    agent: str
    reply: str

# Task Content
class TaskContent(BaseModel):
    id: str
    content: str

# Update Data
class UpdateData(BaseModel):
    task: list[TaskContent]

# SSE Event Formatter
def sse_json(step: str, data):
    res_format = {"step": step, "data": data}
    return f"data: {json.dumps(res_format, ensure_ascii=False)}\n\n"
```

### New Data Models (Skills)

**Location:** `/backend/app/model/skills.py`

```python
from pydantic import BaseModel, Field
from typing import Optional, List, Dict, Any, Literal
from datetime import datetime

# Skill Manifest Definition
class SkillManifest(BaseModel):
    """Claude skill manifest (JSON schema)"""
    skill_id: str = Field(..., description="Unique skill identifier")
    name: str = Field(..., max_length=255)
    description: str = Field(..., max_length=1000)
    version: str = Field(..., pattern="^\\d+\\.\\d+\\.\\d+$")
    author: str = Field(default="Unknown", max_length=255)
    capabilities: List[str] = Field(default_factory=list)
    dependencies: List[str] = Field(default_factory=list)
    parameters: List[SkillParameter] = Field(default_factory=list)
    configuration: List[SkillConfigField] = Field(default_factory=list)
    execution: SkillExecutionSpec
    ui: SkillUISpec = Field(default_factory=dict)

class SkillParameter(BaseModel):
    """Skill parameter definition"""
    name: str
    type: Literal["string", "number", "boolean", "array", "object"]
    description: str
    required: bool = Field(default=False)
    default: Any = Field(default=None)

class SkillConfigField(BaseModel):
    """Skill configuration field"""
    key: str
    type: Literal["string", "number", "boolean", "api_key"]
    default: Any
    description: str
    encrypted: bool = Field(default=False, description="Encrypt this value in storage")

class SkillExecutionSpec(BaseModel):
    """Skill execution specification"""
    type: Literal["local", "remote", "mcp"]
    handler: str = Field(..., description="Python module path or MCP server URL")

class SkillUISpec(BaseModel):
    """UI customization"""
    icon: Optional[str] = None
    color: Optional[str] = None
    category: Optional[str] = None

# API Request Models
class SkillInstallRequest(BaseModel):
    """Request to install a skill"""
    skill_id: str
    user_id: int

class SkillUpdateRequest(BaseModel):
    """Request to update skill"""
    enabled: Optional[bool] = None

class SkillConfigRequest(BaseModel):
    """Request to update skill configuration"""
    config: Dict[str, Any]

class SkillExecutionRequest(BaseModel):
    """Request to execute skill"""
    parameters: Dict[str, Any]
    task_id: Optional[str] = None

# API Response Models
class SkillItem(BaseModel):
    """Installed skill (database representation)"""
    id: int
    user_id: int
    skill_id: str
    name: str
    description: str
    version: str
    manifest: SkillManifest
    enabled: bool
    installed_at: datetime
    updated_at: datetime

class SkillConfig(BaseModel):
    """Skill configuration value"""
    config_key: str
    config_value: Any
    is_encrypted: bool

class SkillExecution(BaseModel):
    """Skill execution record"""
    id: int
    user_id: int
    skill_id: str
    task_id: Optional[str]
    status: Literal["running", "completed", "failed", "cancelled"]
    parameters: Dict[str, Any]
    started_at: datetime
    completed_at: Optional[datetime]
    error_message: Optional[str]
    output: Optional[Dict[str, Any]]
```

### Frontend TypeScript Models

**Location:** `/src/types/skills.ts` (new file)

```typescript
export interface SkillManifest {
  skill_id: string;
  name: string;
  description: string;
  version: string;
  author?: string;
  capabilities: string[];
  dependencies: string[];
  parameters: SkillParameter[];
  configuration: SkillConfigField[];
  execution: SkillExecutionSpec;
  ui: SkillUISpec;
}

export interface SkillParameter {
  name: string;
  type: 'string' | 'number' | 'boolean' | 'array' | 'object';
  description: string;
  required: boolean;
  default?: any;
}

export interface SkillConfigField {
  key: string;
  type: 'string' | 'number' | 'boolean' | 'api_key';
  default?: any;
  description: string;
  encrypted?: boolean;
}

export interface SkillExecutionSpec {
  type: 'local' | 'remote' | 'mcp';
  handler: string;
}

export interface SkillUISpec {
  icon?: string;
  color?: string;
  category?: string;
}

export interface SkillItem {
  id: number;
  user_id: number;
  skill_id: string;
  name: string;
  description: string;
  version: string;
  manifest: SkillManifest;
  enabled: boolean;
  installed_at: string;
  updated_at: string;
}

export interface SkillConfig {
  config_key: string;
  config_value: any;
  is_encrypted: boolean;
}

export interface SkillExecution {
  id: number;
  user_id: number;
  skill_id: string;
  task_id: string | null;
  status: 'running' | 'completed' | 'failed' | 'cancelled';
  parameters: Record<string, any>;
  started_at: string;
  completed_at: string | null;
  error_message: string | null;
  output: Record<string, any> | null;
}

export interface SkillsState {
  installedSkills: SkillItem[];
  availableSkills: SkillManifest[];
  selectedSkill: SkillItem | null;
  isInstalling: boolean;
  isConfiguring: boolean;
  installations: Record<string, boolean>;  // Installation tracking
  executions: SkillExecution[];
}
```

## API Endpoints

### Existing Endpoints

**Base URL:** `/api/v1`

| Method | Endpoint | Purpose | Source |
|--------|-----------|---------|--------|
| POST | `/chat` | Start chat session with SSE streaming | `chat_controller.py` |
| POST | `/chat/{id}` | Improve chat (re-execute with new input) | `chat_controller.py` |
| PUT | `/chat/{id}` | Supplement completed task | `chat_controller.py` |
| DELETE | `/chat/{id}` | Stop task | `chat_controller.py` |
| POST | `/tasks` | Create task | `tasks_router` |
| GET | `/tasks/{id}` | Get task details | `tasks_router` |
| GET | `/agents` | List available agents | `agents_router` |
| POST | `/agents` | Create custom agent | `agents_router` |
| DELETE | `/agents/{id}` | Delete agent | `agents_router` |
| POST | `/install/tool/{tool}` | Install MCP tool | `tool_controller.py` |
| GET | `/tools/available` | List available MCP tools | `tool_controller.py` |
| GET | `/health` | Health check | `health_router` |
| GET/POST | `/learning/*` | Learning system endpoints | `learning_router` |
| GET/POST | `/models/*` | Model configuration | `models_router` |

### New Endpoints (Skills)

**Base URL:** `/api/v1/skills`

| Method | Endpoint | Purpose | Source |
|--------|-----------|---------|--------|
| GET | `/` | List all installed skills for current user | `skills_controller.py` |
| POST | `/discover` | Discover available skills from catalog | `skills_controller.py` |
| POST | `/install` | Install a skill (validate, save to DB) | `skills_controller.py` |
| PUT | `/{skill_id}` | Update skill (enable/disable) | `skills_controller.py` |
| DELETE | `/{skill_id}` | Uninstall a skill | `skills_controller.py` |
| POST | `/{skill_id}/enable` | Enable a skill | `skills_controller.py` |
| POST | `/{skill_id}/disable` | Disable a skill | `skills_controller.py` |
| GET | `/{skill_id}/config` | Get skill configuration | `skills_controller.py` |
| PUT | `/{skill_id}/config` | Update skill configuration (encrypted) | `skills_controller.py` |
| POST | `/{skill_id}/execute` | Execute skill with SSE streaming | `skills_controller.py` |
| GET | `/{skill_id}/logs` | Get skill execution logs | `skills_controller.py` |

**Endpoint Details:**

```python
# GET /api/v1/skills
@router.get("/", name="list installed skills")
async def list_skills(
    user_id: int = Depends(get_current_user_id),
    enabled_only: bool = False
) -> List[SkillItem]:
    """Get all installed skills for user

    Query Params:
        enabled_only: If true, only return enabled skills
    """
    return await get_user_skills(user_id, enabled_only=enabled_only)

# POST /api/v1/skills/discover
@router.post("/discover", name="discover available skills")
async def discover_skills() -> Dict[str, List[SkillManifest]]:
    """Get available skills from catalog

    Returns skills from local ~/.hachiai/skills directory
    Future: Could fetch from remote skills registry
    """
    loader = SkillsPluginLoader()
    manifests = loader.discover_local_skills()
    return {"skills": list(manifests.values())}

# POST /api/v1/skills/install
@router.post("/install", name="install skill")
async def install_skill(
    request: SkillInstallRequest,
    user_id: int = Depends(get_current_user_id)
) -> SkillItem:
    """Install a skill

    Validates manifest, saves to database, enables by default
    """
    # Load and validate manifest
    loader = SkillsPluginLoader()
    manifest = loader.load_skill(request.skill_id)
    if not manifest:
        raise HTTPException(status_code=404, detail="Skill not found")

    # Save to database
    skill = await install_skill_to_db(manifest, user_id)
    return skill

# DELETE /api/v1/skills/{skill_id}
@router.delete("/{skill_id}", name="uninstall skill")
async def uninstall_skill(
    skill_id: str,
    user_id: int = Depends(get_current_user_id)
):
    """Uninstall a skill

    Deletes from database, removes configuration
    """
    await delete_skill_from_db(skill_id, user_id)
    return {"message": "Skill uninstalled"}

# POST /api/v1/skills/{skill_id}/execute
@router.post("/{skill_id}/execute", name="execute skill")
async def execute_skill(
    skill_id: str,
    request: SkillExecutionRequest,
    user_id: int = Depends(get_current_user_id)
) -> StreamingResponse:
    """Execute skill with SSE streaming

    Follows same SSE pattern as /api/v1/chat
    """
    return StreamingResponse(
        execute_skill_async(skill_id, user_id, request.parameters),
        media_type="text/event-stream"
    )
```

### Frontend API Client

**Location:** `/src/api/skills.ts` (new file)

```typescript
import {
  proxyFetchGet,
  proxyFetchPost,
  proxyFetchPut,
  proxyFetchDelete,
  getBaseURL,
} from '@/api/http';
import { fetchEventSource } from '@microsoft/fetch-event-source';
import type {
  SkillItem,
  SkillManifest,
  SkillConfigRequest,
  SkillExecutionRequest,
} from '@/types/skills';

const SKILLS_BASE_URL = `${getBaseURL()}/api/v1/skills`;

// List installed skills
export async function listSkills(enabledOnly = false): Promise<SkillItem[]> {
  const response = await proxyFetchGet(
    `${SKILLS_BASE_URL}?enabled_only=${enabledOnly}`
  );
  return response.skills || [];
}

// Discover available skills
export async function discoverSkills(): Promise<SkillManifest[]> {
  const response = await proxyFetchPost(`${SKILLS_BASE_URL}/discover`);
  return response.skills || [];
}

// Install skill
export async function installSkill(skillId: string): Promise<SkillItem> {
  const response = await proxyFetchPost(`${SKILLS_BASE_URL}/install`, {
    skill_id: skillId,
  });
  return response;
}

// Uninstall skill
export async function uninstallSkill(skillId: string): Promise<void> {
  return proxyFetchDelete(`${SKILLS_BASE_URL}/${skillId}`);
}

// Enable skill
export async function enableSkill(skillId: string): Promise<void> {
  return proxyFetchPost(`${SKILLS_BASE_URL}/${skillId}/enable`);
}

// Disable skill
export async function disableSkill(skillId: string): Promise<void> {
  return proxyFetchPost(`${SKILLS_BASE_URL}/${skillId}/disable`);
}

// Get skill config
export async function getSkillConfig(skillId: string): Promise<SkillConfig[]> {
  const response = await proxyFetchGet(`${SKILLS_BASE_URL}/${skillId}/config`);
  return response.configs || [];
}

// Update skill config
export async function updateSkillConfig(
  skillId: string,
  config: SkillConfigRequest
): Promise<void> {
  return proxyFetchPut(`${SKILLS_BASE_URL}/${skillId}/config`, config);
}

// Execute skill (SSE)
export function executeSkill(
  skillId: string,
  parameters: Record<string, any>,
  taskId?: string,
  onMessage: (message: any) => void,
  onError: (error: any) => void,
  onClose: () => void
): () => void {
  const url = new URL(`${SKILLS_BASE_URL}/${skillId}/execute`);
  url.searchParams.set('task_id', taskId || '');

  const eventSource = fetchEventSource(url.toString(), {
    method: 'POST',
    body: JSON.stringify({ parameters }),
    headers: { 'Content-Type': 'application/json' },
  });

  eventSource.addEventListener('message', (event) => {
    try {
      const data = JSON.parse(event.data);
      onMessage(data);
    } catch (e) {
      console.error('Failed to parse SSE message:', e);
    }
  });

  eventSource.addEventListener('error', onError);
  eventSource.addEventListener('close', onClose);

  // Return cleanup function
  return () => eventSource.close();
}

// Get skill logs
export async function getSkillLogs(skillId: string): Promise<any[]> {
  const response = await proxyFetchGet(`${SKILLS_BASE_URL}/${skillId}/logs`);
  return response.logs || [];
}
```

## Business Rules

### Existing Business Rules

**Source:** `/backend/app/model/chat.py`, `/backend/app/service/task.py`, `/backend/app/utils/workforce.py`

1. **Task Status Lifecycle:**
   - Tasks move through: `confirming` → `confirmed` → `processing` → `done`
   - Source: `Status` enum in `chat.py`

2. **Task Lock Queue:**
   - Only one task execution per task_id
   - Source: `TaskLock` in `task.py`

3. **SSE Event Flow:**
   - Events must follow `sse_json(step, data)` format
   - Source: `sse_json()` in `chat.py`

4. **Agent Toolkit Registration:**
   - Agents must register toolkits before execution
   - Source: `AbstractToolkit` pattern in `toolkit/abstract_toolkit.py`

5. **MCP Tool Configuration:**
   - MCP tools configured in `Chat.installed_mcp`
   - Source: `McpServers` type in `chat.py`

6. **Quality Check Configuration:**
   - CAMEL quality check enabled via `CAMEL_QUALITY_CHECK_ENABLED` environment variable
   - Threshold configurable via `CAMEL_QUALITY_THRESHOLD` (default: 70)
   - Custom eval prompt via `CAMEL_USE_CUSTOM_EVAL_PROMPT` (default: true)
   - Source: `workforce.py` lines 164-201

7. **Credits Check:**
   - Pre-flight check via Nexus API if `VITE_NEXUS_BASE_URL` set
   - Credit types: `organization_then_personal`, `organization`, `personal`
   - Source: Frontend credits check in `chatStore.ts` lines 23-106

8. **Protected Routes:**
   - Routes require `authStore.token` to access
   - Source: `ProtectedRoute` in `routers/index.tsx`

9. **Environment Path:**
   - User-specific env path: `~/.hachiai/<email>/task_<task_id>`
   - Source: `Chat.file_save_path()` in `chat.py`

10. **Browser Port:**
    - Default Chrome DevTools Protocol port: 9222
    - Configurable per task
    - Source: `Chat.browser_port` in `chat.py`

### New Business Rules (Skills)

1. **Skill Manifest Validation:**
   - All skills must have valid `manifest.json` following JSON schema
   - Required fields: `skill_id`, `name`, `description`, `version`, `execution`
   - Source: New `skills_service.py::validate_manifest()`

2. **Skill Installation:**
   - Skills install to database with `enabled = True` by default
   - Skills can be installed only once per user (unique `user_id, skill_id`)
   - Source: New `skills_service.py::install_skill_to_db()`

3. **Skill Configuration Security:**
   - Configuration values with `type="api_key"` must be encrypted at rest
   - Encryption uses AES-256 (upgrade from bcrypt)
   - Source: New `skills_service.py::update_skill_config()`

4. **Skill Enable/Disable:**
   - Disabled skills are not available to agents
   - Skills can be disabled without uninstalling
   - Source: New `skills_controller.py::enable_skill()`, `disable_skill()`

5. **Skill Execution:**
   - Skill execution follows same SSE format as task execution
   - Skill execution events: `skill_start`, `skill_progress`, `skill_complete`, `skill_error`
   - Skill failures are isolated (don't crash platform)
   - Source: New `skills_service.py::execute_skill_async()`

6. **Skill-to-Agent Mapping:**
   - Only enabled skills appear in agent toolkit
   - Skills registered via `ClaudeSkillsToolkit`
   - Source: New `toolkit/claude_skills_toolkit.py`

7. **Skill Discovery:**
   - Skills discovered from local `~/.hachiai/skills` directory
   - Each skill must have `manifest.json` in its folder
   - Future: Remote skills registry integration
   - Source: New `utils/skills_plugin_loader.py`

8. **Skill Dependencies:**
   - Skills declare required Python packages in `dependencies` array
   - Dependencies checked during installation
   - Source: New `skills_service.py::check_dependencies()`

9. **Skill Execution Logging:**
   - All skill executions logged to `claude_skill_executions` table
   - Execution includes status, parameters, output, error
   - Source: New `skills_service.py::log_execution()`

10. **User-Scoped Skills:**
    - Skills installed per user (not global)
    - All skill operations filtered by `user_id`
    - Source: All skill queries with `user_id` filter

## UI Components

### Existing UI Components

**Location:** `/src/components/`

| Component | Location | Purpose |
|-----------|----------|---------|
| `Layout/index.tsx` | Main app layout with sidebar |
| `ChatBox/index.tsx` | Chat interface with messages and tasks |
| `ChatBox/TaskCard.tsx` | Task display card |
| `ChatBox/TaskItem.tsx` | Individual task item |
| `AddWorker/index.tsx` | Add worker to workflow |
| `Folder/index.tsx` | File/folder browser |
| `FilesSection/index.tsx` | File attachment section |
| `Iris/index.tsx` | Agent visualization (business only) |
| `HistorySidebar/index.tsx` | Task history sidebar |
| `Dialog/AddWorkerDialog.tsx` | Add worker dialog |
| `Dialog/DeleteAgentModal.tsx` | Delete agent confirmation |
| `Dialog/Privacy.tsx` | Privacy settings |
| `GlobalSearch/index.tsx` | Global search |

### Existing Setting Pages

**Location:** `/src/pages/Setting/`

| Page | Purpose |
|------|---------|
| `General.tsx` | General application settings |
| `Privacy.tsx` | Privacy and data management |
| `Models.tsx` | Model configuration |
| `API.tsx` | API key settings |
| `MCP.tsx` | MCP tool management |
| `Toolkit.tsx` | Toolkit management |
| `Billing.tsx` | Billing information |
| `Tools.tsx` | Tool settings |
| `MCPMarket.tsx` | MCP marketplace |
| `ToolkitMarket.tsx` | Toolkit marketplace |
| `HooksMarket.tsx` | Hooks marketplace |
| `GuardrailsMarket.tsx` | Guardrails marketplace |
| `MemoryMarket.tsx` | Memory marketplace |
| `KnowledgeBaseMarket.tsx` | Knowledge base marketplace |
| `ToolsMarket.tsx` | Tools marketplace |
| `A2AMarketplace.tsx` | A2A marketplace |
| `ProvidersMarket.tsx` | Providers marketplace |

### MCP Management UI (Reference Pattern)

**Location:** `/src/pages/Setting/MCP.tsx`

```typescript
// State management
const [items, setItems] = useState<MCPUserItem[]>([]);
const [showConfig, setShowConfig] = useState<MCPUserItem | null>(null);
const [configForm, setConfigForm] = useState<MCPConfigForm | null>(null);
const [saving, setSaving] = useState(false);
const [showAdd, setShowAdd] = useState(false);
const [installing, setInstalling] = useState(false);
const [deleting, setDeleting] = useState(false);
const [deleteTarget, setDeleteTarget] = useState<MCPUserItem | null>(null);
```

**Sub-Components:**
- `MCPList.tsx` - List MCP items with enable/disable switches
- `MCPConfigDialog.tsx` - Configure MCP with form fields
- `MCPAddDialog.tsx` - Add new MCP (local JSON or remote URL)
- `MCPDeleteDialog.tsx` - Confirmation dialog for deletion

### New UI Components (Skills)

**Location:** `/src/pages/Setting/Skills.tsx` (follows MCP.tsx pattern)

```typescript
export default function SettingSkills() {
  const navigate = useNavigate();
  const { t } = useTranslation();

  // State (similar to MCP.tsx)
  const [items, setItems] = useState<SkillItem[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [showConfig, setShowConfig] = useState<SkillItem | null>(null);
  const [configForm, setConfigForm] = useState<Record<string, any>>({});
  const [saving, setSaving] = useState(false);
  const [showAdd, setShowAdd] = useState(false);
  const [deleting, setDeleting] = useState(false);
  const [deleteTarget, setDeleteTarget] = useState<SkillItem | null>(null);
  const [switchLoading, setSwitchLoading] = useState<Record<number, boolean>>({});

  // Data fetching
  useEffect(() => {
    loadSkills();
  }, []);

  const loadSkills = async () => {
    setIsLoading(true);
    const skills = await listSkills();
    setItems(skills);
    setIsLoading(false);
  };

  // Handlers
  const handleEnable = async (skillId: string) => {
    setSwitchLoading(prev => ({ ...prev, [skillId]: true }));
    await enableSkill(skillId);
    await loadSkills();
    setSwitchLoading(prev => ({ ...prev, [skillId]: false }));
  };

  const handleDisable = async (skillId: string) => {
    setSwitchLoading(prev => ({ ...prev, [skillId]: true }));
    await disableSkill(skillId);
    await loadSkills();
    setSwitchLoading(prev => ({ ...prev, [skillId]: false }));
  };

  const handleDelete = async (skillId: string) => {
    await uninstallSkill(skillId);
    await loadSkills();
    setDeleteTarget(null);
  };

  // Render
  return (
    <div className="...">
      {/* Header with Add button */}
      {/* Skills list with enable/disable switches */}
      {/* Configuration dialog */}
      {/* Delete confirmation dialog */}
    </div>
  );
}
```

**Location:** `/src/pages/Setting/SkillsCatalog.tsx`

```typescript
export default function SkillsCatalog() {
  const [skills, setSkills] = useState<SkillManifest[]>([]);
  const [installing, setInstalling] = useState<Record<string, boolean>>({});
  const [filter, setFilter] = useState('');

  useEffect(() => {
    loadAvailableSkills();
  }, []);

  const loadAvailableSkills = async () => {
    const response = await discoverSkills();
    setSkills(response);
  };

  const handleInstall = async (skillId: string) => {
    setInstalling(prev => ({ ...prev, [skillId]: true }));
    try {
      await installSkill(skillId);
      toast.success(`Installed ${skillId}`);
    } catch (error) {
      toast.error('Installation failed');
    } finally {
      setInstalling(prev => ({ ...prev, [skillId]: false }));
    }
  };

  const filteredSkills = skills.filter(skill =>
    skill.name.toLowerCase().includes(filter.toLowerCase())
  );

  return (
    <div className="...">
      {/* Search/filter bar */}
      {/* Skills grid with install buttons */}
      {/* Skill cards showing name, description, capabilities */}
    </div>
  );
}
```

**Location:** `/src/components/Skill/SkillConfiguration.tsx`

```typescript
export default function SkillConfiguration({ skill }: { skill: SkillItem }) {
  const [config, setConfig] = useState<Record<string, any>>({});
  const [saving, setSaving] = useState(false);

  useEffect(() => {
    loadConfig();
  }, [skill]);

  const loadConfig = async () => {
    const configs = await getSkillConfig(skill.skill_id);
    const configMap = configs.reduce((acc, c) => ({ ...acc, [c.config_key]: c }), {});
    setConfig(configMap);
  };

  const handleSave = async () => {
    setSaving(true);
    try {
      await updateSkillConfig(skill.skill_id, { config });
      toast.success('Configuration saved');
    } catch (error) {
      toast.error('Failed to save configuration');
    } finally {
      setSaving(false);
    }
  };

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Configure {skill.name}</DialogTitle>
        </DialogHeader>
        <div className="space-y-4">
          {skill.manifest.configuration.map(field => (
            <div key={field.key}>
              <Label>{field.description}</Label>
              {field.type === 'api_key' ? (
                <Input
                  type="password"
                  value={config[field.key] || ''}
                  onChange={e => setConfig({ ...config, [field.key]: e.target.value })}
                />
              ) : field.type === 'boolean' ? (
                <Switch
                  checked={config[field.key] || field.default}
                  onCheckedChange={checked => setConfig({ ...config, [field.key]: checked })}
                />
              ) : (
                <Input
                  value={config[field.key] || field.default}
                  onChange={e => setConfig({ ...config, [field.key]: e.target.value })}
                />
              )}
            </div>
          ))}
        </div>
        <DialogFooter>
          <Button onClick={handleSave} disabled={saving}>
            {saving ? 'Saving...' : 'Save'}
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

**Location:** `/src/components/Skill/SkillExecutionMonitor.tsx`

```typescript
export default function SkillExecutionMonitor({ skillId, taskId }: Props) {
  const [status, setStatus] = useState<'running' | 'completed' | 'failed'>('running');
  const [progress, setProgress] = useState(0);
  const [output, setOutput] = useState('');
  const [logs, setLogs] = useState<string[]>([]);

  useEffect(() => {
    const cleanup = executeSkill(
      skillId,
      {}, // parameters
      taskId,
      (message) => {
        if (message.step === 'skill_start') {
          setStatus('running');
        } else if (message.step === 'skill_progress') {
          setProgress(message.data.progress);
        } else if (message.step === 'skill_complete') {
          setStatus('completed');
          setOutput(message.data.output);
        } else if (message.step === 'skill_error') {
          setStatus('failed');
          toast.error(message.data.error);
        }
      },
      (error) => {
        setStatus('failed');
        console.error('Skill execution error:', error);
      },
      () => {
        // Connection closed
      }
    );

    return cleanup;
  }, [skillId, taskId]);

  return (
    <div className="...">
      {/* Progress bar */}
      {/* Status indicator */}
      {/* Output display */}
      {/* Logs */}
    </div>
  );
}
```

**Location:** `/src/components/Workflow/SkillBlock.tsx` (workflow designer integration)

```typescript
export default function SkillBlock({ skill }: { skill: SkillItem }) {
  return (
    <div className="...">
      <div className="flex items-center gap-2">
        {skill.manifest.ui.icon && <Icon name={skill.manifest.ui.icon} />}
        <span>{skill.name}</span>
      </div>
      {/* Can be dragged into workflow */}
      {/* Connects to other blocks */}
    </div>
  );
}
```

### Dialog Components (Radix UI)

**Pattern:** Following existing MCP dialogs

- `SkillConfigDialog.tsx` - Configure skill settings
- `SkillInstallDialog.tsx` - Confirm skill installation
- `SkillDeleteDialog.tsx` - Confirm skill deletion

### Toast Notifications

**Existing Pattern:** Using `sonner` 2.0.6

```typescript
import { toast } from 'sonner';

// Success
toast.success('Skill installed successfully');

// Error
toast.error('Failed to install skill');

// Info
toast.info('Skill is already enabled');

// Warning
toast.warning('Skill execution timed out');
```

## Authentication & Authorization

### Current Auth System

**Frontend:**
- **Library:** `@stackframe/react` + `react-oidc-context` in `/src/auth/`
- **Configuration:** `/src/auth/config.ts`
- **Token Management:** `authStore.ts` stores `token`, `user_id`, `email`, `username`, `role`
- **Session Guard:** `/src/auth/sessionGuard.ts`
- **Protected Routes:** `ProtectedRoute` component in `/src/routers/index.tsx` checks `authStore.token`

**Backend:**
- **Current State:** Minimal auth enforcement in existing endpoints
- **Token Type:** OIDC Bearer token from StackAuth
- **Token Source:** Sent in `Chat.auth_token` field
- **Token Usage:** Credits check via Nexus API (`VITE_NEXUS_BASE_URL`)

**User Roles:**
- `basic` - Standard user
- `business` - Business user with IREX access

### Target Auth for Skills

**Requirements:**
1. **User-Level Skills:** All skill operations require `user_id`
2. **API Key Encryption:** Skill configurations with `type="api_key"` encrypted at rest
3. **Token-Based Auth:** Skills endpoints use StackAuth token for `user_id` extraction

**Implementation:**

**Frontend Token Injection:**
```typescript
// All skill API calls include Bearer token
export async function listSkills(): Promise<SkillItem[]> {
  const authStore = useAuthStore.getState();
  const response = await proxyFetchGet(`${SKILLS_BASE_URL}`, {
    headers: {
      'Authorization': `Bearer ${authStore.token}`,
    },
  });
  return response.skills || [];
}
```

**Backend Token Validation:**
```python
# New dependency function in backend/app/api/middleware.py
from fastapi import Header, HTTPException

async def get_current_user_id(authorization: str = Header(...)) -> int:
    """Extract user_id from StackAuth Bearer token"""
    if not authorization:
        raise HTTPException(status_code=401, detail="Missing authorization header")

    if not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Invalid authorization format")

    token = authorization[7:]  # Remove "Bearer " prefix

    # Validate token with StackAuth (or decode if JWT)
    user_id = validate_stackauth_token(token)

    if not user_id:
        raise HTTPException(status_code=401, detail="Invalid or expired token")

    return user_id

# Use in skills endpoints
@router.get("/")
async def list_skills(user_id: int = Depends(get_current_user_id)):
    return await get_user_skills(user_id)
```

### API Key Encryption

**Current:** Password hashing with bcrypt in `/backend/app/component/encrypt.py`

**Upgrade to AES-256:**
```python
# New file: backend/app/component/api_key_encryption.py
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64
import os

# Generate or load encryption key from environment
ENCRYPTION_KEY = os.getenv("SKILLS_ENCRYPTION_KEY")
if not ENCRYPTION_KEY:
    ENCRYPTION_KEY = Fernet.generate_key()
    print(f"Generated new encryption key: {ENCRYPTION_KEY}")
    print("Save this to SKILLS_ENCRYPTION_KEY environment variable")

cipher_suite = Fernet(ENCRYPTION_KEY.encode())

def encrypt_api_key(value: str) -> str:
    """Encrypt API key value"""
    return cipher_suite.encrypt(value.encode()).decode()

def decrypt_api_key(encrypted_value: str) -> str:
    """Decrypt API key value"""
    return cipher_suite.decrypt(encrypted_value.encode()).decode()

# Usage in skills_service.py
if config_field.encrypted:
    config_value_encrypted = encrypt_api_key(config_value)
    config_value_text = None
else:
    config_value_encrypted = None
    config_value_text = config_value
```

## External Dependencies

### Existing Dependencies to Preserve

| Dependency | Version | Purpose | Location |
|------------|----------|---------|----------|
| React | 18.3.1 | UI framework | `package.json` |
| TypeScript | 5.4.2 | Type safety | `package.json` |
| Vite | 5.4.11 | Build tool | `package.json` |
| Electron | 33.2.0 | Desktop wrapper | `package.json` |
| Zustand | 5.0.4 | State management | `package.json` |
| Radix UI | Various | UI components | `package.json` |
| Tailwind CSS | 3.4.15 | Styling | `package.json` |
| FastAPI | ≥0.115.12 | API framework | `pyproject.toml` |
| Python | 3.10.16 | Runtime | `pyproject.toml` |
| CAMEL-AI | ≥0.2.76a6, <0.2.79 | Multi-agent framework | `pyproject.toml` |
| loguru | ≥0.7.3 | Logging | `pyproject.toml` |
| @microsoft/fetch-event-source | 2.0.1 | SSE client | `package.json` |
| @stackframe/react | file:../package/@stackframe/react | Auth | `package.json` |
| react-oidc-context | 3.3.0 | OIDC | `package.json` |
| openai | ≥1.99.3, <2 | OpenAI client | `pyproject.toml` |
| httpx | ≥0.28.1 | HTTP client | `pyproject.toml` |
| pydantic-i18n | ≥0.4.5 | Internationalized Pydantic | `pyproject.toml` |
| traceroot | ≥0.0.4a9 | Distributed tracing | `pyproject.toml` |

### New Dependencies

| Dependency | Purpose | Location |
|------------|---------|----------|
| jsonschema | Skill manifest JSON schema validation | `pyproject.toml` |
| cryptography | AES-256 API key encryption (upgrade from bcrypt) | `pyproject.toml` |

### Updated pyproject.toml

```toml
[project]
name = "backend"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = "==3.10.16"
dependencies = [
    "camel-ai[eigent]>=0.2.76a6, <0.2.79",
    "fastapi>=0.115.12",
    "fastapi-babel>=1.0.0",
    "uvicorn[standard]>=0.34.2",
    "pydantic-i18n>=0.4.5",
    "python-dotenv>=1.1.0",
    "httpx[socks]>=0.28.1",
    "loguru>=0.7.3",
    "pydash>=8.0.5",
    "inflection>=0.5.1",
    "aiofiles>=24.1.0",
    "openai>=1.99.3,<2",
    "traceroot>=0.0.4a9",
    "langfuse==3.10.1",
    "pyyaml>=6.0.0",
    # NEW DEPENDENCIES FOR SKILLS
    "jsonschema>=4.20.0",
    "cryptography>=41.0.0",
]
```

### External Services to Preserve

| Service | Purpose | Integration |
|---------|---------|------------|
| StackAuth | OIDC authentication | Frontend `@stackframe/react`, backend token validation |
| Nexus | Credits management | Frontend credits check in `chatStore.ts` |
| Chrome DevTools Protocol | Browser automation | Existing browser toolkits |
| Google Search API | Search capability | Environment variable configuration |
| Exa AI API | Search capability | Environment variable configuration |
| OpenAI API | LLM provider | Configurable in settings |
| Langfuse | LLM evaluation/tracing | Existing integration |

### External Services (New Skills Integration)

| Service | Purpose | Integration |
|---------|---------|------------|
| Claude Skills Registry (Local) | Skill discovery | `~/.hachiai/skills` directory |
| Claude Skills Registry (Remote - Future) | Skill discovery | HTTP API to skills catalog |
| Skill-specific APIs | Skill functionality | Per-skill configuration (API keys) |

### MCP Tools to Preserve

All existing MCP toolkits must continue to work:

- `NotionMCPToolkit` - Notion integration
- `GoogleCalendarToolkit` - Google Calendar
- `GoogleGmailMCPToolkit` - Gmail integration
- All other MCP toolkits in `/backend/app/utils/toolkit/`

These serve as the reference pattern for Skills integration.
