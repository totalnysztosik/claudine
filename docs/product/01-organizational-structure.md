# Claudine Organizational Structure

## The Company Metaphor

Claudine is a digital company. Every piece of Claude Code's extensibility system maps to a role, department, or procedure within this company. This document defines every position, how each employee is hired (created), when they report for duty (activated), what tools they carry (capabilities), and how they communicate with one another (data flow).

---

## Table of Contents

1. [The Org Chart: Primitives as Roles](#1-the-org-chart-primitives-as-roles)
2. [The Constitution: CLAUDE.md](#2-the-constitution-claudemd)
3. [Company Policies: Rules](#3-company-policies-rules)
4. [The Employees: Agents](#4-the-employees-agents)
5. [Standard Operating Procedures: Commands](#5-standard-operating-procedures-commands)
6. [Training Programs: Skills](#6-training-programs-skills)
7. [Automated Quality Gates: Hooks](#7-automated-quality-gates-hooks)
8. [External Contractors: MCP Servers](#8-external-contractors-mcp-servers)
9. [Institutional Memory: The Memory System](#9-institutional-memory-the-memory-system)
10. [The Task Board: TodoWrite/Tasks](#10-the-task-board-todowritetasks)
11. [The Interaction Matrix](#11-the-interaction-matrix)
12. [Activation Hierarchy](#12-activation-hierarchy)
13. [Data Flow Between Primitives](#13-data-flow-between-primitives)
14. [Department Structure](#14-department-structure)

---

## 1. The Org Chart: Primitives as Roles

| Claude Code Primitive | Company Analogue | Always Active? | Who Activates It? | Scope |
|---|---|---|---|---|
| **CLAUDE.md** | Company Constitution & Operating Manual | YES | System (auto-loaded) | Project-wide |
| **Rules** (`.claude/rules/`) | Corporate Policies & Compliance Standards | YES | System (auto-loaded) | Global or project |
| **Agents** (`.claude/agents/`) | Specialist Employees | NO | Parent agent, command, or user | Per-invocation |
| **Commands** (`.claude/commands/`) | Standard Operating Procedures (SOPs) | NO | User (via `/command`) | Per-invocation |
| **Skills** (installed plugins) | Expert Training Programs & Certifications | NO | Skill tool or auto-trigger | Per-invocation |
| **Hooks** (`settings.json`) | Automated Quality Control Gates | YES* | System (on tool events) | Per-tool-call |
| **MCP Servers** | External Contractors & Tool Vendors | YES* | System (connected at start) | Session-wide |
| **Memory** (`.claude/projects/*/memory/`) | Institutional Knowledge Base | NO | Agent reads/writes | Cross-session |
| **Tasks** (TodoWrite) | Project Task Board / Kanban | NO | Agent creates/updates | In-session |

*Hooks are always active but only fire on matching events. MCP servers are connected at session start but tools are invoked on demand.

### The Critical Distinction: Always-On vs. On-Demand

**Always-On primitives** shape EVERY interaction. They are the air Claudine breathes:
- `CLAUDE.md` -- loaded into context window at session start, always present
- `Rules` -- appended to system prompt, always enforced
- `Hooks` -- listening for events, always ready to fire

**On-Demand primitives** are summoned when needed. They are the specialists you call:
- `Agents` -- spawned as subprocesses with their own context
- `Commands` -- triggered by user with `/slash-command`
- `Skills` -- loaded via Skill tool, providing expert guidance
- `Memory` -- read when relevant, written when learning
- `Tasks` -- created to track progress

This distinction is FUNDAMENTAL to Claudine's architecture. Always-on primitives define the culture and rules. On-demand primitives do the actual work.

---

## 2. The Constitution: CLAUDE.md

### What It Is
`CLAUDE.md` is the single most important file in any Claudine-enhanced project. It is the company's constitution -- the foundational document that everything else derives authority from. It is automatically loaded into Claude's context window at the start of every session.

### How It Works
- **Location**: Project root (`/CLAUDE.md`) and/or `.claude/CLAUDE.md`
- **Loading**: Automatic. Claude Code reads this file before any interaction begins.
- **Nesting**: Subdirectory `CLAUDE.md` files are loaded when working in those directories.
- **Priority**: CLAUDE.md instructions override default Claude behavior but yield to direct user instructions.

### What Goes In It
For Claudine, the CLAUDE.md serves as the **master routing document**:

```
CLAUDE.md
├── Project identity (what is this project?)
├── Technology stack declaration
├── Department roster (which agents exist and their roles)
├── Workflow declarations (which commands exist and when to use them)
├── Skill activation triggers (when to invoke which skill)
├── Quality standards (what hooks enforce)
├── Cross-references to rules (which policies apply)
├── MCP server declarations (what external tools are available)
└── Self-improvement protocol (how to update Claudine itself)
```

### How It Fits the Company
CLAUDE.md is the **employee handbook** that every new hire (every new session) reads on day one. It tells Claude:
- Who you are working for (project context)
- What departments exist (agent roster)
- What procedures to follow (command references)
- What standards to uphold (rule references)
- What external tools are available (MCP servers)
- How to improve the company (self-improvement protocol)

### Creation Guidelines
1. CLAUDE.md must be **concise but complete** -- it eats context window
2. Use **references** to other files rather than duplicating content
3. Structure with clear **headers** so Claude can quickly find relevant sections
4. Include a **routing table** that maps task types to departments/agents
5. Update CLAUDE.md as part of the self-improvement loop (see Task 3)

---

## 3. Company Policies: Rules

### What They Are
Rules are markdown files in `.claude/rules/` that are **always appended to the system prompt**. They are corporate policies that every employee must follow, every moment of every day. Unlike CLAUDE.md which can be project-specific, rules can be both **global** (in `~/.claude/rules/`) and **project-specific** (in `.claude/rules/`).

### How They Work
- **Location**: `~/.claude/rules/*.md` (global) or `.claude/rules/*.md` (project)
- **Loading**: Automatic. All rules are appended to the system prompt.
- **Scope**: Global rules apply to all projects. Project rules apply to one project.
- **Priority**: Rules are always active. They cannot be "turned off" within a session.
- **Format**: Plain markdown files. No frontmatter required (but Claudine adds it for consistency).

### How They Differ from CLAUDE.md
| Aspect | CLAUDE.md | Rules |
|---|---|---|
| **Scope** | Project-specific | Global or project |
| **Purpose** | Project handbook + routing | Policy enforcement |
| **Granularity** | One document | Many small files |
| **Content** | Context, routing, references | Constraints, standards, requirements |
| **Analogy** | Employee handbook | Legal compliance documents |

### What Goes In Rules
Rules define **constraints and standards** -- things that must ALWAYS be true:
- Coding style requirements (immutability, file size limits)
- Git workflow standards (commit message format, branch naming)
- Security policies (never hardcode secrets, always validate input)
- Testing requirements (80% coverage, TDD workflow)
- Performance guidelines (model selection, context management)
- Hook behavior documentation (what hooks enforce)

### How They Fit the Company
Rules are the **legal department's output** -- compliance documents that are non-negotiable. They don't tell you what to build; they tell you how everything must be built. An employee (agent) can choose which SOP (command) to follow, but they cannot choose to ignore company policy (rules).

### Creation Guidelines
1. **One rule per concern** -- `cne-rul-immutability.md`, not a monolithic policy doc
2. **Rules must be actionable** -- "Functions must be <50 lines" not "Write clean code"
3. **Rules consume context** -- keep each rule file under 100 lines
4. **Rules are always loaded** -- never put optional/situational content in rules
5. **Use rules for invariants** -- things that are true 100% of the time

---

## 4. The Employees: Agents

### What They Are
Agents are the **specialist employees** of Claudine. Each agent is a markdown file in `.claude/agents/` that defines a persona, capabilities, and operating procedures. When invoked, an agent runs as a **separate subprocess** with its own context window, isolated from the parent conversation.

### How They Work

#### Creation
An agent is defined by a markdown file:
```
.claude/agents/cne-dev-reviewer.md
```

The file contains:
- **Frontmatter** (optional): model selection, tool restrictions
- **Identity**: Who is this agent? What is their role?
- **Instructions**: Detailed operating procedures
- **Tools available**: Which tools the agent can use
- **Output format**: What the agent returns to its caller

#### Invocation
Agents are invoked via the **Agent tool**:
```
Agent tool call:
  subagent_type: "cne-dev-reviewer"  (matches filename without .md)
  prompt: "Review the changes in src/auth/"
  model: "sonnet"  (optional override)
  run_in_background: true  (optional)
  isolation: "worktree"  (optional, for git isolation)
```

#### Lifecycle
1. **Spawn**: Parent calls Agent tool → new subprocess created
2. **Context Load**: Agent's .md file loaded as system instructions
3. **Execute**: Agent reads prompt, uses tools, does work
4. **Return**: Agent returns a single result message to parent
5. **Terminate**: Agent's context is discarded

#### Agent File Frontmatter
Agent markdown files can include optional YAML frontmatter to control model and tool access:
```yaml
---
model: haiku  # or sonnet, opus (overrides default model for this agent)
allowedTools:
  - Read
  - Grep
  - Glob
  - Bash
  # Omit Write/Edit for read-only agents like reviewers
  # Omit Agent to prevent subagent spawning
---
```

If no frontmatter is provided, the agent inherits the parent session's model and has access to all standard tools.

#### Key Properties
- **Isolated context**: An agent does NOT see the parent's conversation history
- **Own tool access**: Agents can use Read, Write, Edit, Bash, Grep, Glob, and nested Agent calls (unless restricted by `allowedTools` frontmatter)
- **No user interaction**: Agents cannot ask the user questions -- they work autonomously. When an agent encounters ambiguity it cannot resolve, it must return with an "insufficient information" message explaining what it needs, so the parent can retry with better context
- **Single return**: An agent returns ONE result message -- it must be comprehensive
- **Parallelizable**: Multiple agents can run simultaneously (all Agent tool calls must be in a SINGLE message for true parallelism)
- **Background capable**: Agents can run in background (`run_in_background: true`), notifying parent on completion
- **Worktree capable**: Agents can get an isolated copy of the repo via git worktrees (`isolation: "worktree"`)

### Agent vs. Subagent

An **agent** is any specialist invoked via the Agent tool. A **subagent** is an agent invoked BY another agent. The distinction is hierarchical:

```
User
 └── Main Claude session (the "CEO")
      ├── Agent A (invoked by main session) ← "Agent"
      │    ├── Agent A1 (invoked by Agent A) ← "Subagent"
      │    └── Agent A2 (invoked by Agent A) ← "Subagent"
      └── Agent B (invoked by main session) ← "Agent"
```

Subagents inherit the same mechanics as agents -- they are just agents invoked from a non-root context. There is no technical difference; the distinction is organizational.

### Who Invokes Agents?

| Invoker | How | Example |
|---|---|---|
| **User** (via main session) | Main session decides to dispatch | "Let me use the planner agent" |
| **Command** (via prompt) | Command prompt instructs to launch agent | `/cne-pm-plan` → launches planner agent |
| **Skill** (via guidance) | Skill instructs agent dispatch | Brainstorming skill → launches parallel agents |
| **Agent** (as subagent) | Parent agent dispatches child | Orchestrator agent → launches reviewer + tester |
| **CLAUDE.md** (via routing) | CLAUDE.md instructs when to auto-dispatch | "For security tasks, always use cne-sec-auditor" |

### How They Fit the Company
Agents are **specialist employees** who are called into meetings when their expertise is needed. They don't sit in every meeting (they're not always-on). When called, they do their job in isolation (their own office), produce a deliverable (the return message), and go home (context discarded).

Some agents are **department heads** (orchestrators) who manage teams of subagents. Some are **individual contributors** who do focused work. The hierarchy mirrors a real company:

```
CEO (main session)
├── VP Engineering (cne-dev-orchestrator)
│    ├── Senior Dev (cne-dev-implementer)
│    ├── Code Reviewer (cne-dev-reviewer)
│    └── Test Writer (cne-qa-tester)
├── VP Product (cne-pm-orchestrator)
│    ├── Product Manager (cne-pm-planner)
│    └── Business Analyst (cne-ba-analyst)
└── VP Security (cne-sec-orchestrator)
     ├── Security Auditor (cne-sec-auditor)
     └── Penetration Tester (cne-sec-pentester)
```

### Creation Guidelines
1. **One agent per role** -- not one agent per task
2. **Clear identity** -- the agent must know WHO it is
3. **Explicit output format** -- define what the agent returns
4. **Minimal but sufficient context** -- agents have limited context windows
5. **Tool restrictions where appropriate** -- a reviewer shouldn't need Write
6. **Model selection** -- use Haiku for simple tasks, Sonnet for complex, Opus for reasoning

---

## 5. Standard Operating Procedures: Commands

### What They Are
Commands are markdown files in `.claude/commands/` that define **user-invocable procedures**. When a user types `/command-name` in Claude Code, the command's markdown content is loaded as a prompt. Commands are the SOPs that any employee (or the CEO/user) can trigger to start a defined workflow.

### How They Work

#### Creation
A command is a markdown file:
```
.claude/commands/cne-pm-plan.md
```

The file contains:
- **Description comment** at top (shown in command picker)
- **Prompt template** -- instructions for Claude to follow
- **$ARGUMENTS** placeholder -- replaced with user's input after the command name
- **Agent dispatch instructions** -- commands can instruct launching agents
- **Cross-references** -- commands can reference skills, rules, other commands

#### Invocation
Commands are invoked by the **user** typing in the CLI:
```
/cne-pm-plan Create a plan for the authentication module
```

This loads the command's markdown as a prompt, with `$ARGUMENTS` replaced by "Create a plan for the authentication module".

#### Key Properties
- **User-facing**: Commands are the primary interface between the user and Claudine
- **Prompt-based**: The command file IS the prompt -- it tells Claude what to do
- **Can invoke agents**: A command's prompt can instruct Claude to dispatch agents
- **Can invoke skills**: A command can reference skills to load
- **Accepts arguments**: Via the `$ARGUMENTS` placeholder
- **Nestable directories**: Commands can be organized in subdirectories (e.g., `.claude/commands/git/`)
- **Shown in picker**: Available commands appear when user types `/`

### Commands vs. Agents: The Critical Difference

| Aspect | Commands | Agents |
|---|---|---|
| **Who triggers** | User (via `/`) | Parent session, command, or another agent |
| **What it is** | A prompt template | A persistent persona definition |
| **Context** | Runs in main session context | Runs in isolated subprocess |
| **Statefulness** | Uses main conversation history | Fresh context each time |
| **User interaction** | Can ask user questions | Cannot interact with user |
| **Purpose** | Define WHAT to do | Define WHO does it |

**The relationship**: A command says "here's what we need to accomplish" and can specify which agent(s) to call to do it. An agent says "here's who I am and how I work" and executes whatever prompt it receives.

**Example flow**:
```
User types: /cne-dev-fix $ARGUMENTS="flaky test in auth module"
  → Command prompt loaded, says: "Launch cne-dev-debugger agent to diagnose,
     then cne-dev-fixer to implement the fix, then cne-qa-tester to verify"
  → Main session dispatches agents as instructed by the command
  → Results flow back to user in main session
```

### Do Commands Invoke Agents, or Do Agents Invoke Commands?

**BOTH**, but in different directions:

1. **Commands → Agents** (primary flow): A command is the entry point that orchestrates which agents to call. This is the standard flow. The user triggers a procedure (command), and the procedure calls in specialists (agents).

2. **Agents → Commands**: Technically, agents cannot invoke slash commands. However, an agent CAN be written to follow the same steps that a command would prescribe. In practice, the orchestration always flows: **User → Command → Agent(s) → Subagent(s)**.

3. **Skills → Agents**: A skill (loaded via Skill tool) provides guidance that may instruct the main session to dispatch agents. The skill doesn't invoke agents directly -- it provides the playbook.

4. **The Routing Chain**:
```
User Input
  → CLAUDE.md routing (suggests command or direct agent)
    → Command (defines the procedure)
      → Skill (provides expert methodology)
        → Agent (executes the work)
          → Subagent (handles sub-tasks)
```

### How They Fit the Company
Commands are **Standard Operating Procedures** -- the documented processes that anyone can invoke. When a project manager says "run the sprint planning procedure," they're invoking a command. The command specifies which specialists (agents) need to be in the room and what each one does.

### Creation Guidelines
1. **Action-oriented names** -- `/cne-dev-fix`, not `/cne-dev-fixer`
2. **Clear description comment** -- users see this in the picker
3. **Explicit agent dispatch** -- name which agents to call
4. **Accept $ARGUMENTS** -- commands should be parameterizable
5. **Include success criteria** -- how do we know the command completed successfully?
6. **Reference rules** -- remind which policies apply to this procedure

---

## 6. Training Programs: Skills

### What They Are
Skills are the most sophisticated primitive in Claude Code. They are **expert knowledge modules** -- detailed playbooks that tell Claude HOW to approach a specific type of work. Skills are installed as plugins and loaded on demand via the Skill tool. They are the "training programs" that give agents and the main session specialized capabilities.

### How They Work

#### Structure
A skill is a directory containing a `SKILL.md` file:
```
skills/cne-dev-typescript/
├── SKILL.md          (the skill definition)
├── references/       (optional supporting files)
│    ├── patterns.md
│    └── examples.md
└── tests/            (optional self-tests)
```

#### The SKILL.md File
```markdown
---
name: cne-dev-typescript
description: TypeScript development patterns and quality standards
---

[Detailed instructions for how to write TypeScript in this project...]
[Checklist items...]
[Examples...]
[Anti-patterns to avoid...]
```

The frontmatter includes:
- **name**: Unique identifier
- **description**: Used to determine relevance (triggers auto-loading)

#### Invocation
Skills are loaded via the **Skill tool**:
```
Skill tool call:
  skill: "cne-dev-typescript"
```

This loads the SKILL.md content into the current context. The skill then GUIDES the session -- it doesn't execute; it instructs.

#### Auto-Triggering
Skills can auto-trigger based on context. The description field is matched against the current task:
- User asks to write TypeScript → `cne-dev-typescript` skill auto-triggers
- User asks to debug → `cne-dev-debug` skill auto-triggers
- User asks to plan → `cne-pm-plan` skill auto-triggers

The superpowers framework enforces: "If there's even a 1% chance a skill applies, invoke it."

### Skills vs. Agents vs. Commands

| Aspect | Skills | Agents | Commands |
|---|---|---|---|
| **Nature** | Knowledge/methodology | Persona/worker | Procedure/workflow |
| **Analogy** | Training program | Employee | SOP document |
| **When loaded** | On demand or auto-trigger | On invocation | On user `/command` |
| **What it does** | Provides guidance | Executes work | Defines what to do |
| **Has own context?** | No (loaded into current) | Yes (isolated subprocess) | No (loaded into current) |
| **Persistence** | Until context ends | Until agent returns | Until prompt completes |
| **Output** | Instructions to follow | Deliverable result | N/A (it IS the prompt) |

**The key insight**: Skills tell you HOW. Commands tell you WHAT. Agents tell you WHO.

**Combined example**:
```
/cne-dev-fix auth bug          ← COMMAND (what to do)
  → loads cne-dev-debug skill  ← SKILL (how to debug)
  → dispatches cne-dev-debugger agent ← AGENT (who debugs)
    → agent follows skill guidance to diagnose and fix
```

### How They Fit the Company
Skills are **training certifications and methodology handbooks**. When you hire a new employee (spawn an agent), they come with baseline knowledge. But when they need to do something specific -- write TypeScript, conduct a security audit, run TDD -- they consult the relevant training manual (skill). The skill doesn't do the work; it ensures the work is done correctly.

### Creation Guidelines
1. **Skills are methodology, not tasks** -- "how to do TDD" not "run the tests"
2. **Include checklists** -- actionable items that can become TodoWrite tasks
3. **Include anti-patterns** -- what NOT to do is as important as what TO do
4. **Include examples** -- concrete code/output examples
5. **Keep SKILL.md focused** -- use `references/` for supporting material
6. **Write trigger descriptions carefully** -- they determine auto-loading
7. **Skills should be composable** -- a session might load multiple skills simultaneously

---

## 7. Automated Quality Gates: Hooks

### What They Are
Hooks are shell commands configured in `settings.json` that automatically execute **before or after** tool calls, or **when the session ends**. They are the automated quality control system -- the gates that every action must pass through.

### How They Work

#### Important Limitation: No SessionStart Hook
Claude Code's hook system supports ONLY three event types: `PreToolUse`, `PostToolUse`, and `Stop`. There is NO `SessionStart` hook. To achieve "session start" behavior, Claudine uses one of these alternatives:
1. **CLAUDE.md instructions** (recommended): CLAUDE.md instructs Claude to read catchup files before responding to the first user request
2. **Startup command**: User runs `/cne-cmd-core-catchup` as the first action in every session
3. **PreToolUse on first tool**: A PreToolUse hook that reads catchup files on the first tool call (requires state management)

#### Configuration in settings.json
Hooks are defined in `settings.json` (global: `~/.claude/settings.json`, project: `.claude/settings.json`, local: `.claude/settings.local.json` which is user-specific and NOT checked into git):
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "/path/to/pre-write-check.sh"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "command": "/path/to/post-bash-check.sh"
      }
    ],
    "Stop": [
      {
        "command": "/path/to/session-end-check.sh"
      }
    ]
  }
}
```

#### Hook Types

| Type | When It Fires | What It Can Do |
|---|---|---|
| **PreToolUse** | Before a tool executes | Validate parameters, modify input, BLOCK execution |
| **PostToolUse** | After a tool executes | Check output, auto-format, log results |
| **Stop** | When session/agent completes | Final verification, cleanup, reporting |

#### Key Properties
- **Automatic**: Hooks fire without anyone invoking them
- **Blocking**: PreToolUse hooks can prevent tool execution
- **Shell-based**: Hooks run shell scripts, not AI prompts
- **Matchable**: Hooks can target specific tools via matcher patterns
- **Environment access**: Hook scripts receive context about the tool call
- **Silent or verbose**: Hooks can report back to the conversation or run silently

### How They Fit the Company
Hooks are the **automated compliance systems** -- metal detectors at the door, smoke alarms in the ceiling, audit logs in the server room. Nobody invokes them. They just fire when conditions are met. A PreToolUse hook is the security guard checking your badge before you enter. A PostToolUse hook is the QA inspector checking your work product. A Stop hook is the exit interview.

### Use Cases in Claudine

| Hook | Trigger | Purpose |
|---|---|---|
| Format on save | PostToolUse (Write/Edit) | Auto-run linter/formatter after code changes |
| Prevent `git add .` | PreToolUse (Bash) | Block dangerous `git add -A` or `git add .` |
| PR comment check | PreToolUse (Bash on merge) | Require code review comment before merge |
| Session summary | Stop | Write session notes to SHARED_TASK_NOTES.md |
| SEO pre-commit | PreToolUse (Bash on commit) | Run SEO checks before committing |
| Skill activation | PreToolUse (any) | Log which skills are being used for analytics |

### Creation Guidelines
1. **Hooks must be FAST** -- they run on every matching tool call
2. **Hooks must be RELIABLE** -- a broken hook blocks all matching tool calls
3. **Hooks should be SILENT by default** -- only report on failures
4. **PreToolUse hooks should be CONSERVATIVE** -- only block when truly necessary
5. **Test hooks thoroughly** -- a bad hook can make Claude Code unusable
6. **Document what each hook does** -- in the rules, so Claude knows why things are being blocked

---

## 8. External Contractors: MCP Servers

### What They Are
MCP (Model Context Protocol) servers are **external tool providers** that connect to Claude Code via a standardized protocol. They give Claude access to tools that don't exist natively -- Jira, Confluence, Figma, databases, APIs, etc.

### How They Work
- **Configuration**: Defined in `settings.json` under the `mcpServers` key (project or global)
- **Connection**: Established at session start; tools available throughout session
- **Tools**: Each MCP server exposes named tools (e.g., `mcp__atlassian__searchJiraIssuesUsingJql`)
- **Invocation**: Tools are called like any other Claude Code tool; execution happens on the MCP server
- **Discovery**: Available MCP tools appear in the session context; use ToolSearch to find specific ones

Example `settings.json` MCP configuration:
```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@anthropic/atlassian-mcp"],
      "env": { "ATLASSIAN_API_TOKEN": "..." }
    }
  }
}
```

### How They Fit the Company
MCP servers are **external contractors and SaaS vendors**. They don't work in your office (they run on separate servers). They provide specialized services (Jira for tickets, Confluence for docs, Figma for designs). You communicate with them through standardized contracts (the MCP protocol).

### Claudine's MCP Strategy
Claudine doesn't define MCP servers -- they're configured per-user/per-project. But Claudine DOES:
1. **Reference MCP tools in commands** -- e.g., "use Jira MCP to create the ticket"
2. **Reference MCP tools in agents** -- e.g., "search Confluence for context"
3. **Document which MCP servers are expected** -- in CLAUDE.md
4. **Provide fallbacks** -- if MCP server unavailable, what to do instead

---

## 9. Institutional Memory: The Memory System

### What It Is
The memory system is a file-based persistent knowledge store in `.claude/projects/*/memory/`. It survives across sessions, allowing Claudine to build institutional knowledge over time.

### How It Works
- **Location**: `~/.claude/projects/{project-hash}/memory/`
- **Index**: `MEMORY.md` -- always loaded into context, lists all memory files
- **Files**: Individual `.md` files with frontmatter (type, name, description)
- **Types**: user, feedback, project, reference
- **Access**: Read when relevant, written when learning

### How It Fits the Company
Memory is the **company wiki** -- the institutional knowledge that persists when employees come and go (sessions start and end). It stores:
- Who the user is (user memories)
- What approaches work/don't work (feedback memories)
- What's happening in the project (project memories)
- Where to find things (reference memories)

### Claudine's Memory Strategy
Memory is critical for Claudine's self-improvement loop (see Task 3). After every session, Claudine should:
1. Record what went well (feedback memory)
2. Record what went poorly (feedback memory)
3. Update project context (project memory)
4. Note any new external references (reference memory)

---

## 10. The Task Board: TodoWrite/Tasks

### What They Are
Tasks (via TaskCreate/TaskUpdate tools) are an in-session progress tracking system. They create a visible task list that shows the user what's being worked on, what's done, and what's next.

### How They Work
- **Creation**: `TaskCreate` with subject, description, activeForm
- **Updates**: `TaskUpdate` to change status (pending → in_progress → completed)
- **Dependencies**: Tasks can block/be blocked by other tasks
- **Ownership**: Tasks can be assigned to specific agents
- **Visibility**: The task list is shown to the user in real-time

### How They Fit the Company
Tasks are the **Kanban board** -- the daily standup where everyone sees what's in progress. They don't persist across sessions (that's what memory is for). They organize the current session's work.

### Claudine's Task Strategy
Every command should create a task list that reflects its steps. This:
1. Shows the user progress in real-time
2. Helps Claude track its own work
3. Enables the user to steer (add/remove/reorder tasks)
4. Provides a natural checkpoint system

---

## 11. The Interaction Matrix

This matrix shows which primitives can invoke, reference, or be aware of which other primitives:

| Invoker ↓ / Target → | CLAUDE.md | Rules | Agents | Commands | Skills | Hooks | MCP | Memory | Tasks |
|---|---|---|---|---|---|---|---|---|---|
| **CLAUDE.md** | - | references | lists/routes to | lists/routes to | lists triggers | documents | declares | references | - |
| **Rules** | referenced by | - | constrains | constrains | constrains | documented in | constrains | - | - |
| **Agents** | reads | obeys | spawns (subagent) | cannot invoke | can load (Skill tool) | triggers (on tool use) | can use tools | can read/write | can create/update |
| **Commands** | reads | obeys | instructs to spawn | cannot invoke self | can load (Skill tool) | triggers (on tool use) | can use tools | can read/write | can create |
| **Skills** | reads | obeys | instructs to spawn | cannot invoke | can reference | - | can reference | - | instructs to create |
| **Hooks** | - | enforces | - | - | - | - | - | - | - |
| **MCP** | - | - | - | - | - | - | - | - | - |
| **Memory** | - | - | - | - | - | - | - | - | - |
| **Tasks** | - | - | - | - | - | - | - | - | - |

### Reading the Matrix
- **Agents can spawn subagents**: An agent can call the Agent tool to create child agents
- **Commands instruct agent spawning**: A command's prompt says "launch agent X"
- **Skills guide behavior**: A skill tells the session HOW to work, which may include dispatching agents
- **Hooks enforce rules**: Hooks run automatically to enforce policies defined in rules
- **CLAUDE.md routes everything**: It's the master document that says "for X, use Y"

---

## 12. Activation Hierarchy

When a user makes a request, here is the complete activation chain:

```
SESSION START
│
├─ CLAUDE.md loaded (constitution)
├─ Rules loaded (policies)
├─ Hooks registered (quality gates)
├─ MCP servers connected (contractors)
├─ Memory index loaded (institutional knowledge)
│
USER MAKES REQUEST
│
├─ 1. CLAUDE.md routing table consulted
│     "What kind of request is this? Which department handles it?"
│
├─ 2. Relevant command identified (if applicable)
│     User may type /command or main session may determine the appropriate workflow
│
├─ 3. Relevant skills auto-triggered or manually loaded
│     "How should this type of work be done?"
│
├─ 4. Agent(s) dispatched per command/skill guidance
│     "Who will do the work?"
│
│     ├─ 4a. Agent loads its own instructions (.claude/agents/name.md)
│     ├─ 4b. Agent receives prompt (from command or parent)
│     ├─ 4c. Agent may load additional skills
│     ├─ 4d. Agent may spawn subagents (for swarm pattern)
│     ├─ 4e. Agent executes work (Read, Write, Edit, Bash, etc.)
│     │       └── HOOKS FIRE on each tool call
│     ├─ 4f. Agent returns result to parent
│     └─ 4g. Agent context discarded
│
├─ 5. Results consolidated by main session or orchestrator agent
│
├─ 6. Tasks updated (completed, new tasks created)
│
├─ 7. Memory updated if new learnings captured
│
└─ 8. Response presented to user

SESSION END
│
├─ Stop hooks fire (final checks)
├─ Memory written (session learnings)
└─ Session context discarded
```

---

## 13. Data Flow Between Primitives

### The Prompt Chain
Data flows through Claudine as **prompts** -- text instructions that carry context from one primitive to the next:

```
CLAUDE.md context (always present)
  + Rules context (always present)
    + User request (the trigger)
      + Command template (the procedure)
        + Skill content (the methodology)
          = COMPLETE PROMPT for the main session

Main session then creates:
  Agent prompt = Agent instructions (.md) + Task description + Relevant context

Agent then creates:
  Subagent prompt = Subagent instructions (.md) + Sub-task description + Parent context
```

### Context Boundaries
**CRITICAL**: Each agent has its OWN context window. Data does NOT automatically flow between agents. The parent must EXPLICITLY include relevant context in the agent's prompt.

```
Main Session Context:
├── CLAUDE.md ✓
├── Rules ✓
├── User conversation history ✓
├── Loaded skills ✓
├── Tool results ✓
└── Agent A's return message ✓

Agent A Context:
├── Agent A's .md instructions ✓
├── Prompt from parent ✓
├── Agent A's own tool results ✓
├── CLAUDE.md ✗ (NOT automatically included)
├── Rules ✗ (NOT automatically included)
├── User conversation ✗ (NOT available)
└── Skills ✗ (must load independently)
```

This means:
1. **Agent instructions must be self-contained** -- include critical rules/context in the agent .md
2. **Prompts to agents must carry context** -- pass relevant information in the prompt
3. **Skills loaded in parent don't transfer to agents** -- agents must load their own skills
4. **Memory must be explicitly read** -- agents don't automatically know memory contents

### The Return Path
Data flows BACK from agents to the main session via the **return message** -- a single text response that contains everything the agent wants to report. This message becomes part of the main session's context.

```
Agent A returns: "Found 3 security issues: [details]"
  → This text is now in the main session's context
  → Main session can use this to dispatch Agent B: "Fix these 3 issues: [details]"
```

---

## 14. Department Structure

Based on the personas defined in v2.md and the source repositories, Claudine's departments are:

### Engineering Division
| Dept Code | Department | Agents | Commands | Skills |
|---|---|---|---|---|
| `dev` | Development | implementer, reviewer, debugger, fixer | fix, implement, refactor | typescript, react, python, etc. |
| `qa` | Quality Assurance | tester, e2e-runner, auditor | test, audit, validate | tdd, e2e, test-master |
| `arc` | Architecture | architect, analyst | optimize, design, review-arch | architecture-designer, coupling-analysis |
| `dop` | DevOps | deployer, infra-checker | deploy, check-infra | docker, k8s, ci-cd |
| `sec` | Security | auditor, pentester, reviewer | scan, audit-sec | security-best-practices, threat-model |
| `db` | Database | dba, optimizer | migrate, optimize-db | postgres, sql |

### Product Division
| Dept Code | Department | Agents | Commands | Skills |
|---|---|---|---|---|
| `pm` | Project Management | planner, tracker, reporter | plan, status, retro | planning-with-files, scrum-master |
| `po` | Product | owner, strategist, analyst | discover, prioritize | product-manager, product-discovery |
| `ba` | Business Analysis | analyst, researcher | analyze, research | feature-forge, spec-miner |

### Creative Division
| Dept Code | Department | Agents | Commands | Skills |
|---|---|---|---|---|
| `ui` | Design | designer, ux-guardian | design, audit-ux | frontend-design, figma |
| `twr` | Technical Writing | writer, reviewer | docs, changelog | docs-writer, code-documenter |
| `seo` | SEO | auditor, content-optimizer | audit-seo, optimize-seo | seo, seo-audit |
| `mkt` | Marketing | content-creator, strategist | campaign, content | marketing-strategy, copywriting |

### Operations Division
| Dept Code | Department | Agents | Commands | Skills |
|---|---|---|---|---|
| `ops` | Operations | orchestrator, monitor | status, health-check | monitoring, observability |
| `cld` | Cloud | architect, deployer | deploy-cloud, audit-cloud | aws, azure, gcp |
| `net` | Networking | engineer | audit-network | - |

### Governance Division
| Dept Code | Department | Agents | Commands | Skills |
|---|---|---|---|---|
| `cmp` | Compliance | auditor, reporter | audit-compliance | gdpr, iso27001, soc2 |
| `fin` | Finance | analyst | budget, forecast | financial-analyst |

### Meta Division (Claudine Self-Management)
| Dept Code | Department | Agents | Commands | Skills |
|---|---|---|---|---|
| `cne` | Claudine Core | self-improver, skill-builder, validator | adjust, improve, audit-self | skill-architect, skill-tester |

---

## Summary

Claudine is a company where:

1. **CLAUDE.md** is the constitution -- always read, always obeyed, routes all traffic
2. **Rules** are corporate policies -- always enforced, cannot be bypassed
3. **Agents** are specialist employees -- called when needed, work in isolation, return deliverables
4. **Commands** are SOPs -- user-triggered procedures that orchestrate agents
5. **Skills** are training programs -- loaded on demand, provide expert methodology
6. **Hooks** are automated quality gates -- fire automatically, enforce standards
7. **MCP servers** are external contractors -- provide tools not available in-house
8. **Memory** is institutional knowledge -- persists across sessions, grows over time
9. **Tasks** are the Kanban board -- track in-session progress

The flow is always: **User → Command → Skill → Agent → Subagent**, with Rules and Hooks providing guard rails, CLAUDE.md providing routing, Memory providing context, and MCP providing external capabilities.
