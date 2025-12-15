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
- **Output:** `feature_list.json` (50-75 test cases: 30-45 functional + 20-30 visual), `init.sh` (environment setup), `.gitignore`, git repository, and project structure
- **Key Task:** Generates `feature_list.json` containing **50-75** strictly defined test cases: 30-45 functional tests and 20-30 visual/design tests ensuring pixel-perfect design system compliance.

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

### Standard Workflow (New Projects)

- **`@architect`**: Use when starting a new project. Always run this first to create `app_spec.txt`.
- **`@grist-database-specialist`**: Use only if `app_spec.txt` specifies Grist as the database backend, or if you explicitly want to use Grist's hosted free-tier API. This agent helps design the schema and implement the integration layer.
- **`@project-initializer`**: Use after `app_spec.txt` exists (and optionally after Grist setup which creates `grist_schema.txt`). This creates the project foundation and `feature_list.json`.
- **`@coder`**: Use repeatedly during development. Run this agent to implement features one at a time from `feature_list.json`.
- **`@release-engineer`**: Called AUTOMATICALLY by the coder agent after each feature completion. Can also be called manually for milestone releases. Requires a clean git working directory and passing tests. Handles SemVer versioning, changelog updates, and git tagging (does not deploy). Designed to be called multiple times during development.

### Improvement Workflow (Adding Features to Existing Apps)

- **`@architect`** (improvement mode): Use when `app_spec.txt` already exists and you want to add new features. Creates `improvement_spec.txt` instead of modifying `app_spec.txt`.
- **`@project-initializer`** (improvement mode): Use after `improvement_spec.txt` exists. Creates `improvement_list.json` with tests for new features only.
- **`@coder`** (improvement mode): Automatically detects `improvement_list.json` and prioritizes improvements over original features. Merges improvement files after all improvements pass.
- **`@release-engineer`**: Works identically in both workflows. Called automatically after each improvement completion.

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

---

## Iterative Improvement Workflow

After your initial app development is complete, you may want to add new features or make improvements to your deployed application. Cascade supports an **iterative improvement workflow** that allows you to enhance your existing application without disrupting the original feature tracking.

### When to Use Improvement Workflow

Use this workflow when:

- Your app is already deployed/completed and `feature_list.json` exists
- You want to add **NEW features** not in the original feature list
- You want to make **enhancements** or **improvements** to existing functionality
- You want to **refactor** or **optimize** existing features
- The user comes back with additional requirements after the initial build

### Workflow Sequence

The improvement workflow follows a similar but specialized path:

1. **@architect** (improvement mode) → `improvement_spec.txt`
2. **@project-initializer** (improvement mode) → `improvement_list.json`
3. **@coder** (improvement mode) → implements/verifies improvements
4. **@release-engineer** → versioning (automatic, no change)

### How It Works

#### Step 1: Define Improvements

Start by engaging the architect in improvement mode:

```bash
@architect I want to add [describe new features/improvements]
```

**The architect will:**

- **Detect improvement mode** by checking if `app_spec.txt` already exists
- Read your existing `app_spec.txt` to understand current architecture
- Analyze your improvement request
- Ask clarifying questions about the new features
- Create `improvement_spec.txt` with the improvement requirements

**Key difference from initial workflow:**

- `app_spec.txt` remains **unchanged** (it's the original specification)
- `improvement_spec.txt` is created **separately** to track the new additions
- The improvement spec focuses on **what's changing** rather than re-describing the entire app

#### Step 2: Create Improvement Tests

After the improvement specification is ready, initialize the testing framework:

```bash
@project-initializer Set up tests for the improvements
```

**The project-initializer will:**

- **Detect improvement mode** by finding `improvement_spec.txt`
- Read `improvement_spec.txt` for new feature requirements
- Read `app_spec.txt` for existing app context
- Review existing `feature_list.json` to avoid duplicate tests
- Generate `improvement_list.json` with **5-25 test cases** (fewer than initial because it's incremental)

**Key difference from initial workflow:**

- Only creates `improvement_list.json` (no `init.sh`, `.gitignore`, or git repo setup)
- Tests focus on **new/changed functionality** only
- Includes **regression tests** to verify existing features still work
- **Does NOT modify** `feature_list.json` at this stage

#### Step 3: Implement Improvements

Now engage the coder to build the improvements:

```bash
@coder Work on the improvements
```

**The coder will:**

- **Detect improvement mode** by finding `improvement_list.json`
- **Prioritize improvements** - work on `improvement_list.json` FIRST before touching `feature_list.json`
- Implement each improvement one by one
- Test with browser automation
- Mark improvements as passing in `improvement_list.json`
- **Automatically call release-engineer** after each completed improvement (same as standard workflow)

**Automatic Merge When Complete:**

Once ALL improvements are passing (all tests in `improvement_list.json` have `"passes": true`), the coder will **automatically**:

1. Merge `improvement_list.json` into `feature_list.json`
2. Delete `improvement_list.json`
3. Delete `improvement_spec.txt`
4. Commit the merge with a descriptive message

After the merge, the coder returns to **standard mode** and can continue with any remaining features in the now-unified `feature_list.json`.

#### Step 4: Release Management (Automatic)

The **@release-engineer** agent works identically for both workflows:

- Called automatically by the coder after each improvement is completed
- Creates incremental version tags (typically MINOR bumps for new features, PATCH for enhancements)
- Updates changelog with improvement descriptions
- No changes needed to the agent itself

### File Structure During Improvements

**Before Improvement Workflow:**

```text
your-app/
├── app_spec.txt              # Original specification
├── feature_list.json         # Original features (some or all passing)
├── init.sh                   # Environment setup
└── [app code files...]
```

**During Improvement Workflow:**

```text
your-app/
├── app_spec.txt              # Original specification (unchanged)
├── improvement_spec.txt      # New improvements specification
├── feature_list.json         # Original features (unchanged during improvements)
├── improvement_list.json     # New improvement tests (separate)
├── init.sh                   # Environment setup
└── [app code files...]
```

**After Improvements Complete:**

```text
your-app/
├── app_spec.txt              # Original specification
├── feature_list.json         # Merged: original + improvements
├── init.sh                   # Environment setup
└── [app code files...]
```

Note: `improvement_spec.txt` and `improvement_list.json` are automatically deleted after the merge.

### Example Improvement Scenario

**Scenario:** You've built a task management app and now want to add collaboration features.

```bash
# Step 1: Define the improvement
@architect I want to add task sharing and real-time collaboration features

# The architect creates improvement_spec.txt with:
# - Task sharing functionality
# - Real-time updates via WebSocket
# - Activity feed
# - Dependencies on existing features

# Step 2: Set up improvement tests
@project-initializer Set up tests for the collaboration features

# The project-initializer creates improvement_list.json with 15 tests:
# - 10 functional tests for new features
# - 3 integration tests for how improvements work with existing features
# - 2 regression tests to ensure existing task creation still works

# Step 3: Build the improvements
@coder Work on the collaboration features

# The coder:
# - Implements task sharing API and UI
# - Adds WebSocket server for real-time sync
# - Builds activity feed component
# - Tests each feature with browser automation
# - Creates releases: v1.3.0, v1.4.0, v1.5.0 (MINOR bumps for new features)
# - After all 15 tests pass, automatically merges improvement_list.json into feature_list.json
```

### Multiple Improvement Cycles

You can run the improvement workflow **multiple times** on the same project:

1. Complete first set of improvements → Files merge
2. User requests more improvements → Create new `improvement_spec.txt`
3. Run improvement workflow again → New `improvement_list.json` created
4. Complete second set → Files merge again

Each cycle is independent and follows the same pattern.

### Benefits of Separate Improvement Tracking

1. **Clear Separation:** Original features vs. new improvements remain distinct during development
2. **Easier Rollback:** If improvements don't work out, you can abandon them without affecting the original feature list
3. **Focused Testing:** Improvement tests specifically target new functionality and integration points
4. **Single Source of Truth:** After merge, `feature_list.json` contains the complete history of all features
5. **Flexible Re-entry:** Start improvements at any time without disrupting existing work

---

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
