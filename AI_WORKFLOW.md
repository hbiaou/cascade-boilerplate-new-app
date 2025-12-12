# Cascade - Autonomous Coding Boilerplate

## Overview

**Cascade** is a **silent scaffolding system**. It provides the intelligence to build an app, but it is designed to disappear from the final product. It only serves as a **standardized scaffolding system** for building complex software applications using autonomous AI agents.

Instead of starting from zero with every new project, Cascade provides the essential "instructional architecture" required to turn a high-level idea into a production-ready application. It implements a strictly defined **Five-Agent Workflow** that separates concerns between architectural planning, database setup (when needed), project initialization, iterative development, and release management.

## Core Philosophy

As a boilerplate, this project aims to:

1. **Kickstart Projects:** Eliminate the repetitive setup of defining how the AI should behave.
2. **Enforce Standards:** Ensure every project starts with a robust specification (`app_spec.txt`) and a rigorous testing plan (`feature_list.json`).
3. **Ensure Quality:** Mandate a "Test-First" development approach using real browser automation (Playwright) rather than fragile DOM emulation.

## The Five-Agent Workflow

Cascade defines five distinct roles for the AI to play. You will use these agents sequentially, with the Grist Database Specialist being optional (only when using Grist as your backend). The agents cascade from one to the next, each building upon the previous agent's output.

![](templates/workflow-diagram.jpg)

### 1. The Specification Writer (Architect)

- **Agent File:** `.claude/agents/architect.md`
- **Command:** `@architect`
- **Role:** Takes a vague user idea and converts it into a rigid, XML-structured technical requirement file.
- **Input:** User conversation, optional documentation/mockups
- **Output:** `app_spec.txt` - Complete technical specification in XML format
- **Why:** Prevents "scope creep" and ensures the coding agent has a single source of truth. The spec should explicitly state the database backend choice (SQLite/Postgres/etc. vs Grist hosted).

### 2. The Grist Database Specialist (Optional)

- **Agent File:** `.claude/agents/grist-database-specialist.md`
- **Command:** `@grist-database-specialist`
- **Role:** Optional specialist step for projects choosing Grist (hosted free-tier) as the backend database. Produces a Grist schema + API integration plan and helps implement BYOK settings + offline-first caching patterns.
- **Input:** `app_spec.txt` (required, to confirm Grist is specified as database backend)
- **Output:** `grist_schema.txt` - Contains the complete Grist schema definition, table structure, API configuration requirements, and integration architecture. This file is read by the Project Initializer to understand the database setup.
- **When to use:** Only when `app_spec.txt` specifies Grist as the database backend, or when the user explicitly wants to use Grist.
- **Why:** Grist requires specialized setup (REST API access, schema design, caching strategies) that differs from traditional SQL databases.

### 3. The Project Initializer (Project Lead)

- **Agent File:** `.claude/agents/project-initializer.md`
- **Command:** `@project-initializer`
- **Role:** Reads the spec (`app_spec.txt`) and Grist schema (`grist_schema.txt` if it exists) and sets up the physical project structure.
- **Input:** `app_spec.txt` (required), `grist_schema.txt` (optional, if Grist is used)
- **Output:** `feature_list.json` (25-50 test cases), `init.sh` (environment setup), `.gitignore`, git repository, and project structure
- **Key Task:** Generates `feature_list.json` containing **25-50** strictly defined test cases (functional & stylistic).

### 4. The Coding Agent (Developer)

- **Agent File:** `.claude/agents/coder.md`
- **Command:** `@coder`
- **Role:** An iterative worker that picks **one** failing test, implements the code, and verifies it using browser automation.
- **Input:** `app_spec.txt`, `feature_list.json`, `init.sh` (required), `grist_schema.txt`, `claude-progress.txt` (optional)
- **Output:** Application code files (unpredictable, depends on tech stack), updates to `feature_list.json` (pass/fail status only), updates to `claude-progress.txt`
- **Tooling:** Configured to use **Playwright MCP** for visual regression testing and end-to-end verification.
- **Constraint:** Cannot move to feature B until feature A passes.

### 5. The Release Engineer (Finalization)

- **Agent File:** `.claude/agents/release-engineer.md`
- **Command:** `@release-engineer`
- **Role:** Final step that performs SemVer versioning, changelog updates, annotated tags, and pre-flight checks. Does not deploy.
- **Input:** Version manifest files (package.json, pyproject.toml, Cargo.toml, etc.), git repository, `CHANGELOG.md` (optional, creates if missing)
- **Output:** Updated version manifests, `CHANGELOG.md` (created or updated), annotated git tags (`v{version}`), release commit
- **When to use:** When you're ready to tag a versioned release (after all required tests pass, or for pre-releases like alpha/beta/rc).
- **Why:** Ensures professional, error-free releases following semantic versioning best practices.

## Repository Structure

```text
cascade/
├── .claude/
│   └── agents/
│       ├── architect.md                    # Agent 1: Creates the Blueprint (app_spec.txt)
│       ├── grist-database-specialist.md    # Agent 2 (Optional): Grist DB setup & integration
│       ├── project-initializer.md           # Agent 3: Sets the Foundation (feature_list.json, init.sh)
│       ├── coder.md                         # Agent 4: Builds the House (iterative development)
│       └── release-engineer.md              # Agent 5: Finalizes releases (versioning & tagging)
├── templates/
│   └── app_spec_example.txt                 # Reference implementation of a spec file
└── AI_WORKFLOW.md
```

## Directory Info

- **`.claude/`**: Contains the agent definitions. Ignored by git.

- **`templates/`**: Reference files. Ignored by git.

- **`AI_WORKFLOW.md`**: This guide. Ignored by git.

- **`Root`**: The actual application code lives here.

To use Cascade effectively, you need:

1. **An LLM Interface:** (e.g., Claude Code, Google Antigravity, Cursor, or a CLI).
2. **Playwright MCP Server:** The Coding Agent is configured to use the Model Context Protocol (MCP) to control a headless browser.
   - *Required Tools:* `browser_navigate`, `browser_click`, `browser_take_screenshot`, etc.

## When to Use Which Agent

Each agent has specific trigger conditions:

- **`@architect`**: Use when starting a new project. Always run this first to create `app_spec.txt`.
- **`@grist-database-specialist`**: Use only if `app_spec.txt` specifies Grist as the database backend, or if you explicitly want to use Grist's hosted free-tier API. This agent helps design the schema and implement the integration layer.
- **`@project-initializer`**: Use after `app_spec.txt` exists (and optionally after Grist setup which creates `grist_schema.txt`). This creates the project foundation and `feature_list.json`.
- **`@coder`**: Use repeatedly during development. Run this agent to implement features one at a time from `feature_list.json`.
- **`@release-engineer`**: Called AUTOMATICALLY by the coder agent after each feature completion. Can also be called manually for milestone releases. Requires a clean git working directory and passing tests. Handles SemVer versioning, changelog updates, and git tagging (does not deploy). Designed to be called multiple times during development.

## Usage Guide

### Step 1: Scaffolding

Clone this repository to your local machine. When you want to start a NEW app.

### Step 2: Define the Spec

Run this command to start the Architect session:

```bash
@architect. I want to build a [Your Idea Here]...
```

Wait for the agent to generate `app_spec.txt`. The spec will explicitly state the database backend choice.

### Step 3: (Optional) Grist Database Setup

**Only if using Grist as your backend database:**

```bash
@grist-database-specialist. Set up Grist database schema and API integration
```

The agent will guide you through creating the Grist document, designing the schema, and implementing the API client/service layer with BYOK configuration.

### Step 4: Initialize

```bash
@project-initializer. Please setup the project based on the app_spec.txt
```

The agent will generate the folder structure, install dependencies, and create the test plan (`feature_list.json`).

### Step 5: Build the App (The Loop)

Now, you simply iterate with the Coder. Run this command repeatedly:

```bash
@coder. Fix the next failing test.
```

**Important:** After each feature is completed and verified, the Coder agent will AUTOMATICALLY call the Release Engineer to create an incremental release. This means:

- Each completed feature gets its own version tag (typically PATCH bumps: 1.2.3 → 1.2.4)
- The changelog is updated incrementally with each feature
- Version history is maintained throughout development
- You don't need to manually call the release-engineer during the build loop

*Repeat Step 5 until all tests pass. The release-engineer will be called automatically after each feature completion.*

### Step 6: Final Release (Optional)

If you want to create a milestone release (e.g., moving from 0.x.x to 1.0.0, or creating a beta/rc release), you can manually call:

```bash
@release-engineer. Create a milestone release
```

This is optional - the automatic incremental releases during development are sufficient for most cases.

## Contributing

Cascade is a living boilerplate. If you find that the agents consistently fail at a specific task (e.g., database migration), update the relevant agent file in `.claude/agents/` to provide better "gap-filling" logic.

## Acknowledgments & Inspiration

Cascade was inspired by research and best practices from the AI agent development community. We gratefully acknowledge the following sources:

### Primary Inspiration

- **"AI Agents That Actually Work"** by Nate B Jones  
  YouTube Video: [https://www.youtube.com/watch?v=xNcEgqzlPqs](https://www.youtube.com/watch?v=xNcEgqzlPqs)  
  YouTube Channel: [AI News & Strategy Daily | Nate B Jones](https://www.youtube.com/@NateBJones)  
  *This video provided the primary inspiration for Cascade's multi-agent workflow architecture.*

### Additional References

- **"Effective harnesses for long-running agents"** by Anthropic  
  Blog Post: [https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)  
  *This research informed our approach to incremental progress tracking, feature lists, and session management across multiple context windows.*

- **Anthropic Autonomous Coding Quickstart**  
  GitHub Repository: [https://github.com/anthropics/claude-quickstarts/tree/main/autonomous-coding](https://github.com/anthropics/claude-quickstarts/tree/main/autonomous-coding)  
  *This quickstart provided valuable insights into initializer and coding agent patterns, testing strategies, and environment management.*

These resources collectively shaped Cascade's design philosophy of using specialized agents, structured feature tracking, and incremental development to enable long-running autonomous coding workflows.
