# Data Model

## Entities

### Chat
- **Fields:**
  - task_id: str (Unique identifier for the task/workflow)
  - question: str (User's request or task description)
  - email: str (User email for authentication and file path)
  - attaches: list[str] (List of attached file paths)
  - model_platform: str (LLM provider platform, e.g., "openai", "anthropic")
  - model_type: str (Specific model, e.g., "gpt-4", "claude-3-5-sonnet")
  - api_key: str (API key for the LLM provider)
  - api_url: str | None (Optional custom API URL for cloud deployments)
  - language: str (Language setting, default "en")
  - browser_port: int (Chrome DevTools Protocol port, default 9222)
  - max_retries: int (Maximum retry attempts, default 3)
  - allow_local_system: bool (Allow local system access, default False)
  - installed_mcp: McpServers (Configured MCP tools)
  - bun_mirror: str (Mirror for Bun package manager)
  - uvx_mirror: str (Mirror for UVX package manager)
  - env_path: str | None (Custom environment path for task execution)
  - summary_prompt: str (Prompt for task completion summary generation)
  - new_agents: list[NewAgent] (Custom agent definitions for the task)
  - extra_params: dict | None (Provider-specific parameters like Azure settings)
  - auth_token: str | None (Bearer token for Nexus credits check and guardrails)
  - agent_providers: dict[str, AgentProviderConfig] (Per-agent provider configuration overrides)
  - template_mode: bool (Flag indicating template execution mode)
  - template_subtasks: list[dict] | None (Pre-defined subtasks from template)
  - template_summary: str | None (Pre-defined summary from template)
  - template_id: int | None (Reference to template being used)
  - source_task_id: str | None (Parent task this was cloned/re-executed from)
- **Relationships:**
  - One-to-many with chat_history (via task_id)
  - One-to-many with chat_step (via task_id)
  - One-to-many with chat_snapshot (via task_id)
  - One-to-many with new_agents (Agent definitions for this task)
  - Many-to-many with mcp_user (MCP tools via installed_mcp field)
- **Source:** `/backend/app/model/chat.py`

### Status
- **Fields:**
  - confirming: str (Task is awaiting confirmation from user)
  - confirmed: str (Task is confirmed and ready to start)
  - processing: str (Task is currently being processed)
  - done: str (Task is completed)
- **Relationships:**
  - Used by chat_history, TaskLock
- **Source:** `/backend/app/model/chat.py`

### ChatHistory
- **Fields:**
  - role: RoleType (Message sender role: user, assistant, system, etc.)
  - content: str (Message content)
- **Relationships:**
  - Many-to-one with chat_history (database table)
- **Source:** `/backend/app/model/chat.py`

### NewAgent
- **Fields:**
  - name: str (Custom agent name)
  - description: str (Agent description)
  - tools: list[str] (List of tool names the agent can use)
  - mcp_tools: McpServers | None (MCP tool configurations for this agent)
  - env_path: str | None (Custom environment path for this agent)
- **Relationships:**
  - Many-to-one with Chat (belongs to a task)
- **Source:** `/backend/app/model/chat.py`

### AgentProviderConfig
- **Fields:**
  - agent_name: str (Agent type name, e.g., "Developer_Agent", "Search_Agent")
  - model_platform: str (LLM provider platform for this agent)
  - model_type: str (Specific model for this agent)
  - api_key: str (API key for this agent's provider)
  - api_url: str | None (Optional custom API URL)
  - extra_params: dict | None (Provider-specific parameters)
- **Relationships:**
  - Many-to-one with Chat (agent_providers dict maps agent_name to AgentProviderConfig)
- **Source:** `/backend/app/model/chat.py`

### McpServers
- **Fields:**
  - mcpServers: dict[str, dict] (Nested dict mapping server names to configurations)
- **Relationships:**
  - Embedded in Chat.installed_mcp and NewAgent.mcp_tools
- **Source:** `/backend/app/model/chat.py`

### TaskLock
- **Fields:**
  - id: str (Task identifier)
  - status: Status (Current status: confirming, confirmed, processing, done)
  - active_agent: str (Name of currently active agent)
  - mcp: list[str] (List of MCP tools configured for this task)
  - queue: asyncio.Queue[ActionData] (SSE event queue for streaming responses)
  - human_input: dict[str, asyncio.Queue[str]] (Queue for human replies per agent)
  - created_at: datetime (Task creation timestamp)
  - last_accessed: datetime (Last activity timestamp)
  - background_tasks: set[asyncio.Task] (Background task references for cleanup)
  - stop_requested: asyncio.Event (Event signaling task should stop)
- **Relationships:**
  - One-to-one with Task (CAMEL-AI task object)
- **Source:** `/backend/app/service/task.py`

### Action
- **Fields:**
  - improve: str (User requests task improvement)
  - update_task: str (User updates task content)
  - task_state: str (Backend sends task state update to user)
  - start: str (User starts task execution)
  - create_agent: str (Backend creates agent)
  - activate_agent: str (Backend activates agent)
  - deactivate_agent: str (Backend deactivates agent)
  - assign_task: str (Backend assigns task to agent)
  - activate_toolkit: str (Backend activates toolkit)
  - deactivate_toolkit: str (Backend deactivates toolkit)
  - write_file: str (Backend writes file)
  - ask: str (Backend asks user question)
  - notice: str (Backend sends notice to user)
  - search_mcp: str (Backend searches MCP tools)
  - install_mcp: str (Backend installs MCP tool)
  - terminal: str (Backend sends terminal output)
  - end: str (Backend signals task completion)
  - stop: str (User stops task)
  - supplement: str (User supplements completed task)
  - pause: str (User takes control of task)
  - resume: str (User releases control of task)
  - new_agent: str (User creates new agent)
  - budget_not_enough: str (Backend signals insufficient budget)
  - browser_navigate: str (Backend sends browser URL change)
- **Relationships:**
  - Used in SSE streaming via ActionData union
- **Source:** `/backend/app/service/task.py`

### AbstractToolkit
- **Fields:**
  - api_task_id: str (Task identifier for context)
  - agent_name: str (Name of agent using this toolkit)
- **Relationships:**
  - Extended by concrete toolkit implementations (NotionMCPToolkit, GoogleCalendarToolkit, etc.)
- **Source:** `/backend/app/utils/toolkit/abstract_toolkit.py`

### SkillManifest
- **Fields:**
  - skill_id: str (Unique skill identifier following claude:// scheme, e.g., "claude://skills/commit")
  - name: str (Human-readable skill name, max 255 chars)
  - description: str (Skill capabilities description, max 1000 chars)
  - version: str (Semantic version, pattern ^\d+\.\d+\.\d+$)
  - author: str (Skill author or organization, default "Unknown")
  - capabilities: list[str] (List of capability keywords)
  - dependencies: list[str] (Required Python packages or system dependencies)
  - parameters: list[SkillParameter] (Parameters accepted by skill execution)
  - configuration: list[SkillConfigField] (Configurable options for skill)
  - execution: SkillExecutionSpec (Execution specification)
  - ui: SkillUISpec (UI customization options)
- **Relationships:**
  - Many-to-one with SkillItem (embedded in manifest JSONB field)
  - Contains many SkillParameter definitions
  - Contains many SkillConfigField definitions
- **Source:** `/backend/app/model/skills.py` (new)

### SkillParameter
- **Fields:**
  - name: str (Parameter name)
  - type: Literal["string", "number", "boolean", "array", "object"] (Parameter data type)
  - description: str (Parameter description)
  - required: bool (Whether parameter is required, default False)
  - default: Any (Default value for parameter)
- **Relationships:**
  - Many-to-one with SkillManifest
- **Source:** `/backend/app/model/skills.py` (new)

### SkillConfigField
- **Fields:**
  - key: str (Configuration key identifier)
  - type: Literal["string", "number", "boolean", "api_key"] (Configuration data type)
  - default: Any (Default value for configuration)
  - description: str (Configuration description)
  - encrypted: bool (Whether to encrypt this value in storage, default False)
- **Relationships:**
  - Many-to-one with SkillManifest
  - Many-to-one with SkillConfig (via config_key)
- **Source:** `/backend/app/model/skills.py` (new)

### SkillExecutionSpec
- **Fields:**
  - type: Literal["local", "remote", "mcp"] (Execution type)
  - handler: str (Python module path or MCP server URL)
- **Relationships:**
  - One-to-one with SkillManifest
- **Source:** `/backend/app/model/skills.py` (new)

### SkillUISpec
- **Fields:**
  - icon: str | None (Icon identifier or URI)
  - color: str | None (Color hex code)
  - category: str | None (Skill category for organization)
- **Relationships:**
  - One-to-one with SkillManifest
- **Source:** `/backend/app/model/skills.py` (new)

### SkillInstallRequest
- **Fields:**
  - skill_id: str (Skill identifier to install)
  - user_id: int (User ID requesting installation)
- **Relationships:**
  - Request payload for POST /api/v1/skills/install
- **Source:** `/backend/app/model/skills.py` (new)

### SkillUpdateRequest
- **Fields:**
  - enabled: bool | None (New enabled status)
- **Relationships:**
  - Request payload for PUT /api/v1/skills/{skill_id}
- **Source:** `/backend/app/model/skills.py` (new)

### SkillConfigRequest
- **Fields:**
  - config: dict[str, Any] (Configuration key-value pairs)
- **Relationships:**
  - Request payload for PUT /api/v1/skills/{skill_id}/config
- **Source:** `/backend/app/model/skills.py` (new)

### SkillExecutionRequest
- **Fields:**
  - parameters: dict[str, Any] (Execution parameters)
  - task_id: str | None (Optional task to link execution to)
- **Relationships:**
  - Request payload for POST /api/v1/skills/{skill_id}/execute
- **Source:** `/backend/app/model/skills.py` (new)

### SkillItem
- **Fields:**
  - id: int (Database primary key)
  - user_id: int (User who installed this skill)
  - skill_id: str (Unique skill identifier)
  - name: str (Skill display name)
  - description: str (Skill description)
  - version: str (Skill version)
  - manifest: SkillManifest (Full skill manifest as JSONB)
  - enabled: bool (Whether skill is available for agents)
  - installed_at: datetime (Installation timestamp)
  - updated_at: datetime (Last update timestamp)
- **Relationships:**
  - One-to-many with SkillConfig (via skill_id)
  - One-to-many with SkillExecution (via skill_id)
  - Many-to-one with user (via user_id)
- **Source:** `/backend/app/model/skills.py` (new)

### SkillConfig
- **Fields:**
  - id: int (Database primary key)
  - user_id: int (User who owns this configuration)
  - skill_id: str (Skill this configuration belongs to)
  - config_key: str (Configuration field key)
  - config_value_encrypted: str | None (Encrypted value for sensitive data like API keys)
  - config_value_text: str | None (Plain text value for non-sensitive data)
  - updated_at: datetime (Last update timestamp)
- **Relationships:**
  - Many-to-one with SkillItem (via skill_id)
  - Many-to-one with user (via user_id)
  - Many-to-one with SkillConfigField (via config_key)
- **Source:** `/backend/app/model/skills.py` (new)

### SkillExecution
- **Fields:**
  - id: int (Database primary key)
  - user_id: int (User who triggered execution)
  - skill_id: str (Skill being executed)
  - task_id: int | None (Associated task ID, references chat_history)
  - status: Literal["running", "completed", "failed", "cancelled"] (Execution status)
  - parameters: JSONB (Execution parameters)
  - started_at: datetime (Execution start timestamp)
  - completed_at: datetime | None (Completion timestamp)
  - error_message: str | None (Error details if failed)
  - output: JSONB | None (Execution output or result)
- **Relationships:**
  - Many-to-one with SkillItem (via skill_id)
  - Many-to-one with user (via user_id)
  - Many-to-one with chat_history (via task_id, optional)
- **Source:** `/backend/app/model/skills.py` (new)

## Relationships Diagram

```
┌─────────────────┐
│     user      │ (existing table)
└───────┬────────┘
        │
        ├──► chat_history (1:N)
        │     ├──► TaskLock (1:1)
        │     │     └──► Task (CAMEL-AI)
        │     │           ├──► Agent (1:N)
        │     │           └──► Toolkit (1:N)
        │     │                ├──► AbstractToolkit
        │     │                │     ├──► NotionMCPToolkit
        │     │                │     └──► GoogleCalendarToolkit
        │     │                └──► ClaudeSkillsToolkit (NEW)
        │     ├──► chat_step (1:N)
        │     └──► chat_snapshot (1:N)
        │
        ├──► mcp_user (1:N) (existing table)
        │
        └──► claude_skills (1:N) (NEW table)
              │
              ├──► claude_skill_configs (1:N) (NEW table)
              │     └──► SkillConfigField (via manifest.config)
              │
              └──► claude_skill_executions (1:N) (NEW table)
                    └──► chat_history (optional reference)
```

**Legend:**
- `──►` = One-to-many relationship
- `(1:N)` = One-to-many cardinality
- `(1:1)` = One-to-one cardinality
- `via` = Relationship path

**Key Observations:**
1. Skills are user-scoped (user_id foreign key)
2. Skills extend the existing toolkit pattern via ClaudeSkillsToolkit
3. Skills can be linked to tasks for execution tracking
4. Configuration uses dual storage (encrypted for API keys, plain text for others)
5. Skills maintain their own manifest JSON for self-containment

## Migration Notes

### Database Tables (New)

#### Table: `claude_skills`
```sql
CREATE TABLE claude_skills (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES user(id),
    skill_id VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    version VARCHAR(50),
    manifest JSONB NOT NULL,
    enabled BOOLEAN DEFAULT TRUE,
    installed_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

**Notes:**
- `user_id` references existing `user` table (StackAuth integration)
- `manifest` stores complete SkillManifest as JSONB for self-containment
- `enabled` allows disabling skills without uninstalling
- UNIQUE constraint on `skill_id` per user prevents duplicate installations

#### Table: `claude_skill_configs`
```sql
CREATE TABLE claude_skill_configs (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES user(id),
    skill_id VARCHAR(255) NOT NULL REFERENCES claude_skills(skill_id) ON DELETE CASCADE,
    config_key VARCHAR(255) NOT NULL,
    config_value_encrypted TEXT,
    config_value_text TEXT,
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, skill_id, config_key)
);
```

**Notes:**
- Dual storage pattern: `config_value_encrypted` for sensitive data (API keys), `config_value_text` for plain values
- UNIQUE constraint ensures one config value per (user, skill, key)
- ON DELETE CASCADE removes configs when skill is uninstalled
- Encryption using AES-256 (upgrade from existing bcrypt in `encrypt.py`)

#### Table: `claude_skill_executions`
```sql
CREATE TABLE claude_skill_executions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES user(id),
    skill_id VARCHAR(255) NOT NULL REFERENCES claude_skills(skill_id) ON DELETE SET NULL,
    task_id INTEGER REFERENCES chat_history(id) ON DELETE SET NULL,
    status VARCHAR(50) NOT NULL CHECK (status IN ('running', 'completed', 'failed', 'cancelled')),
    parameters JSONB,
    started_at TIMESTAMP DEFAULT NOW(),
    completed_at TIMESTAMP,
    error_message TEXT,
    output JSONB
);
```

**Notes:**
- `task_id` references existing `chat_history` table for linking executions to tasks
- ON DELETE SET NULL preserves execution records even if task is deleted
- CHECK constraint ensures valid status values
- `output` stores structured result data as JSONB

### Indexes (Performance Optimization)

```sql
-- Skills indexes
CREATE INDEX idx_skills_user ON claude_skills(user_id);
CREATE INDEX idx_skills_enabled ON claude_skills(user_id, enabled);

-- Config indexes
CREATE INDEX idx_configs_user_skill ON claude_skill_configs(user_id, skill_id);

-- Execution indexes
CREATE INDEX idx_executions_user ON claude_skill_executions(user_id);
CREATE INDEX idx_executions_task ON claude_skill_executions(task_id);
CREATE INDEX idx_executions_status ON claude_skill_executions(status);
```

### Constraints to Preserve

#### Existing Constraints (Must Not Break)
1. **User authentication flow:** StackAuth OIDC integration via `user` table
2. **Task execution isolation:** TaskLock mechanism in memory (must work with skills)
3. **SSE streaming format:** `sse_json(step, data)` format must remain unchanged
4. **MCP tool compatibility:** Existing `installed_mcp` field in Chat model
5. **Agent toolkit registration:** AbstractToolkit pattern must extend cleanly

#### New Constraints (Skills System)
1. **User-scoped skills:** All skill operations must filter by `user_id`
2. **API key encryption:** Sensitive config values (`type="api_key"`) must be encrypted
3. **Skill manifest validation:** JSON schema validation before database insertion
4. **Enable/disable isolation:** Disabled skills must not appear in agent toolkit
5. **Execution isolation:** Skill failures must not crash platform or affect other skills
6. **Unique installations:** Prevent duplicate skill installation per user via UNIQUE constraint
7. **Cascade deletion:** Uninstalling skill must cascade delete configs and set execution task_id to NULL

### Data Migration Strategy

#### Phase 1: Schema Creation (Non-Destructive)
- Run CREATE TABLE statements for new tables
- Run CREATE INDEX statements for performance
- Verify foreign key constraints with existing `user` and `chat_history` tables

#### Phase 2: Application Initialization
- Create `~/.hachiai/skills` directory on first run
- Scan directory for `manifest.json` files
- Load and validate skill manifests
- (No automatic skill installation - requires user action via UI)

#### Phase 3: Backward Compatibility Verification
- Test existing task execution without any skills installed
- Verify SSE streaming still works for agent tasks
- Verify MCP tools still function via existing endpoints
- Verify StackAuth authentication still works

#### Rollback Strategy
- Drop tables in reverse dependency order:
  1. DROP INDEX idx_executions_status;
  2. DROP INDEX idx_executions_task;
  3. DROP INDEX idx_executions_user;
  4. DROP INDEX idx_configs_user_skill;
  5. DROP INDEX idx_skills_enabled;
  6. DROP INDEX idx_skills_user;
  7. DROP TABLE IF EXISTS claude_skill_executions;
  8. DROP TABLE IF EXISTS claude_skill_configs;
  9. DROP TABLE IF EXISTS claude_skills;

### Encryption Migration

#### Current State
- Password hashing using bcrypt in `/backend/app/component/encrypt.py`
- Files:
  ```python
  from passlib.context import CryptContext
  password = CryptContext(schemes=["bcrypt"], deprecated="auto")
  ```

#### Target State (Skills API Keys)
- AES-256 encryption for API keys using `cryptography` library
- New file: `/backend/app/component/api_key_encryption.py`
- Environment variable: `SKILLS_ENCRYPTION_KEY` (generate if not set)

#### Migration Notes
- Existing password hashing remains unchanged (for user passwords)
- New AES-256 encryption is additive (only for skill API keys)
- No migration needed for existing MCP tool configs (they use different pattern)
