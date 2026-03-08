# Project Dd

![SDD](https://img.shields.io/badge/methodology-Spec--Driven%20Development-blue?style=flat-square)

> A software project built with Spec-Driven Development.

## Features

*Feature tracking will appear here once implementation begins.*

## Project Structure

| Directory | Purpose |
|-----------|--------|
| `specs/` | Specification files (SDD artifacts) |
| `.claude/` | Claude Code configuration |

## Getting Started

```bash
# See project documentation for setup instructions
```

## Development Process

This project uses **Spec-Driven Development (SDD)** — a methodology where structured specification files guide every stage of development, from requirements gathering through implementation and verification.

SDD wraps around existing delivery processes rather than replacing them. This approach minimizes disruption while maximizing value delivery and allows teams to maintain their current tools and workflows.

### Speckit Workflow

The feature lifecycle follows a structured pipeline from specification through implementation:

```mermaid
flowchart LR
    subgraph const [" "]
        C["🛡️ /constitution\nProject principles\n& standards"]
    end

    subgraph lifecycle ["Feature Life Cycle"]
        direction LR
        S["/specify\nDefine what\nto build"] --> CL["/clarify\nFollow-up\nquestions"]
        CL --> P["/plan\nImplementation\nplan"]
        P --> T["/tasks\nBreak into\nactionable steps"]
        T --> A["/analyze\nValidate docs"]
        A --> I["/implement\nExecute tasks\n& test"]
        I --> PR["Create PR\n/ Merge"]
    end

    C -.-> S

    style const fill:#1a2332,stroke:#4a7c9b,color:#fff
    style lifecycle fill:#1a2332,stroke:#4a7c9b,color:#fff
    style C fill:#2d4a2d,stroke:#6b8e6b,color:#fff
    style S fill:#1e3a5f,stroke:#4a90d9,color:#fff
    style CL fill:#1e3a5f,stroke:#4a90d9,color:#fff
    style P fill:#1e3a5f,stroke:#4a90d9,color:#fff
    style T fill:#1e3a5f,stroke:#4a90d9,color:#fff
    style A fill:#1e3a5f,stroke:#4a90d9,color:#fff
    style I fill:#1e3a5f,stroke:#4a90d9,color:#fff
    style PR fill:#2a2a3e,stroke:#8888aa,color:#fff
```

| Stage | Command | Purpose |
|-------|---------|---------|
| Define | `/specify` | Capture requirements — what to build, without technical detail |
| Clarify | `/clarify` | Follow-up questions to resolve ambiguity |
| Plan | `/plan` | Create an implementation strategy |
| Tasks | `/tasks` | Break the plan into actionable, trackable steps |
| Validate | `/analyze` | Verify consistency across all spec documents |
| Build | `/implement` | Execute tasks, write code, run tests |
| Ship | PR / Merge | Code review and integration |

### Spec Store

SDD files define the **WHAT**, **HOW**, **DOING**, **VERIFYING**, and **WHY** to build software safely and predictably with AI assistance:

```mermaid
flowchart TD
    CONST["🛡️ <b>constitution.md</b><br/>Supreme Rules · Non-Negotiable Laws"]

    CONST --> SPEC["📋 <b>spec.md</b><br/>WHAT · Requirements"]
    CONST --> PLAN["📋 <b>plan.md</b><br/>HOW · Strategy"]
    CONST --> TASKS["✅ <b>tasks.md</b><br/>DOING · Execution"]
    CONST --> CONTRACTS["🔗 <b>contracts/</b><br/>Interaction"]

    SPEC --> DM["💾 datamodel.md<br/>Data"]
    PLAN --> CONSIST["📏 consistency.md<br/>Rules"]
    TASKS --> CHECK["☑️ checklists/<br/>Verifying"]

    DM --> DM2["💾 datamodel.md<br/>Data"]
    CONSIST --> QS1["🚀 quickstart.md<br/>Rules"]
    CHECK --> QS2["🚀 quickstart.md<br/>Onboarding"]
    CONTRACTS --> RES["📚 research.md<br/>Context"]

    style CONST fill:#c0392b,stroke:#922b21,color:#fff
    style SPEC fill:#2e86c1,stroke:#1a5276,color:#fff
    style PLAN fill:#27ae60,stroke:#1e8449,color:#fff
    style TASKS fill:#27ae60,stroke:#1e8449,color:#fff
    style CONTRACTS fill:#8e44ad,stroke:#6c3483,color:#fff
    style DM fill:#f39c12,stroke:#d68910,color:#000
    style CONSIST fill:#f39c12,stroke:#d68910,color:#000
    style CHECK fill:#f39c12,stroke:#d68910,color:#000
    style DM2 fill:#e8daef,stroke:#8e44ad,color:#000
    style QS1 fill:#abebc6,stroke:#27ae60,color:#000
    style QS2 fill:#aed6f1,stroke:#2e86c1,color:#000
    style RES fill:#fadbd8,stroke:#c0392b,color:#000
```

| File | Role | Description |
|------|------|-------------|
| `constitution.md` | Governance | Supreme rules and non-negotiable project constraints |
| `spec.md` | Requirements | **What** to build — features, user stories, acceptance criteria |
| `plan.md` | Strategy | **How** to build it — architecture decisions, phasing |
| `tasks.md` | Execution | **Doing** — ordered, trackable implementation steps |
| `contracts/` | Interaction | API contracts, interface definitions |
| `datamodel.md` | Data | Entity schemas, relationships, validation rules |
| `consistency.md` | Rules | Cross-cutting constraints and invariants |
| `checklists/` | Verifying | Quality gates and verification criteria |

## Built With

Built with **[InfiniOS](https://github.com/infiniai-tech/infiniai-os-v2)** — an autonomous software development platform that uses Spec-Driven Development to build production-quality applications.

---

*Generated by [InfiniOS](https://github.com/infiniai-tech/infiniai-os-v2)*
