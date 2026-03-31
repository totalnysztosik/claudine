# Claudine Flow Diagrams

## How to Read These Diagrams

All diagrams use Mermaid syntax. To render them:
- Paste into any Mermaid renderer (mermaid.live, GitHub markdown, VS Code plugin)
- Rectangles = processes/agents
- Diamonds = decisions
- Parallelograms = inputs/outputs
- Cylinders = data stores
- Arrows = data flow direction

---

## Table of Contents

1. [The Company Org Chart](#1-the-company-org-chart)
2. [The Primitive Hierarchy](#2-the-primitive-hierarchy)
3. [The Activation Chain](#3-the-activation-chain)
4. [The Self-Improvement Loop](#4-the-self-improvement-loop)
5. [Flow: Bug Fix (Waterfall)](#5-flow-bug-fix-waterfall)
6. [Flow: Full Audit (Swarm)](#6-flow-full-audit-swarm)
7. [Flow: New Feature (Hybrid)](#7-flow-new-feature-hybrid)
8. [Flow: Sprint Planning](#8-flow-sprint-planning)
9. [Flow: Release Preparation](#9-flow-release-preparation)
10. [Flow: Project Onboarding (/cne-cmd-core-adjust)](#10-flow-project-onboarding)
11. [Flow: Self-Improvement (/cne-cmd-core-improve)](#11-flow-self-improvement)
12. [The Master Flow Map](#12-the-master-flow-map)
13. [Data Flow: Context Propagation](#13-data-flow-context-propagation)

---

## 1. The Company Org Chart

This is Claudine as a company. Every department, every employee, every role.

```mermaid
graph TB
    CEO["CEO<br/>(Main Session / User)"]

    subgraph ENGINEERING["Engineering Division"]
        DEV_HEAD["VP Dev<br/>cne-agt-core-orchestrator"]
        DEV_IMPL["Developer<br/>cne-agt-dev-implementer"]
        DEV_REVIEW["Code Reviewer<br/>cne-agt-dev-reviewer"]
        DEV_DEBUG["Debugger<br/>cne-agt-dev-debugger"]
        DEV_FIX["Fixer<br/>cne-agt-dev-fixer"]
        DEV_REFACTOR["Refactorer<br/>cne-agt-dev-refactorer"]

        QA_TEST["Tester<br/>cne-agt-qa-tester"]
        QA_E2E["E2E Runner<br/>cne-agt-qa-e2e-runner"]
        QA_AUDIT["QA Auditor<br/>cne-agt-qa-auditor"]

        ARC_DESIGN["Architect<br/>cne-agt-arc-designer"]
        ARC_ANALYZE["Analyst<br/>cne-agt-arc-analyst"]

        DOP_DEPLOY["Deployer<br/>cne-agt-dop-deployer"]
        DOP_INFRA["Infra Checker<br/>cne-agt-dop-infra-checker"]

        SEC_AUDIT["Sec Auditor<br/>cne-agt-sec-auditor"]
        SEC_PEN["Pentester<br/>cne-agt-sec-pentester"]
        SEC_REVIEW["Sec Reviewer<br/>cne-agt-sec-reviewer"]

        DB_OPT["DBA<br/>cne-agt-db-optimizer"]
    end

    subgraph PRODUCT["Product Division"]
        PM_PLAN["Planner<br/>cne-agt-pm-planner"]
        PM_TRACK["Tracker<br/>cne-agt-pm-tracker"]
        PM_REPORT["Reporter<br/>cne-agt-pm-reporter"]

        PO_STRAT["Strategist<br/>cne-agt-po-strategist"]
        BA_ANALYZE["BA Analyst<br/>cne-agt-ba-analyst"]
    end

    subgraph CREATIVE["Creative Division"]
        UI_DESIGN["Designer<br/>cne-agt-ui-designer"]
        UI_UX["UX Guardian<br/>cne-agt-ui-ux-guardian"]

        TWR_WRITE["Writer<br/>cne-agt-twr-writer"]
        TWR_REVIEW["Doc Reviewer<br/>cne-agt-twr-reviewer"]

        SEO_AUDIT["SEO Auditor<br/>cne-agt-seo-auditor"]
        SEO_OPT["SEO Optimizer<br/>cne-agt-seo-optimizer"]

        MKT_CREATE["Content Creator<br/>cne-agt-mkt-creator"]
    end

    subgraph META["Meta Division (Self-Management)"]
        CORE_IMPROVE["Self-Improver<br/>cne-agt-core-improver"]
        CORE_VALID["Validator<br/>cne-agt-core-validator"]
        CORE_ORCH["Orchestrator<br/>cne-agt-core-orchestrator"]
    end

    CEO --> ENGINEERING
    CEO --> PRODUCT
    CEO --> CREATIVE
    CEO --> META

    DEV_HEAD --> DEV_IMPL
    DEV_HEAD --> DEV_REVIEW
    DEV_HEAD --> DEV_DEBUG
    DEV_HEAD --> DEV_FIX
    DEV_HEAD --> DEV_REFACTOR
```

---

## 2. The Primitive Hierarchy

How Claude Code primitives relate to each other and to the company metaphor.

```mermaid
graph TD
    subgraph ALWAYS_ON["Always Active (Company Culture)"]
        CLAUDE["CLAUDE.md<br/>(Constitution)"]
        RULES["Rules<br/>(Policies)"]
        HOOKS["Hooks<br/>(Quality Gates)"]
        MCP["MCP Servers<br/>(Contractors)"]
    end

    subgraph ON_DEMAND["On-Demand (Called When Needed)"]
        COMMANDS["Commands<br/>(SOPs)"]
        SKILLS["Skills<br/>(Training)"]
        AGENTS["Agents<br/>(Employees)"]
        SUBAGENTS["Subagents<br/>(Teams)"]
    end

    subgraph PERSISTENCE["Persistence (Memory)"]
        MEMORY["Memory Files<br/>(Institutional Knowledge)"]
        TASKS["Tasks<br/>(Kanban Board)"]
        LOGS["Session Logs<br/>(Execution History)"]
    end

    CLAUDE -->|routes to| COMMANDS
    CLAUDE -->|declares| AGENTS
    CLAUDE -->|triggers| SKILLS

    RULES -->|constrain| AGENTS
    RULES -->|constrain| COMMANDS

    COMMANDS -->|invoke| AGENTS
    COMMANDS -->|load| SKILLS

    SKILLS -->|guide| AGENTS

    AGENTS -->|spawn| SUBAGENTS
    AGENTS -->|use| MCP
    AGENTS -->|trigger| HOOKS
    AGENTS -->|read/write| MEMORY
    AGENTS -->|create/update| TASKS
    AGENTS -->|write| LOGS

    HOOKS -->|enforce| RULES
```

---

## 3. The Activation Chain

What happens when a user makes a request -- the complete flow from input to output.

```mermaid
sequenceDiagram
    actor User
    participant Session as Main Session
    participant CLAUDE as CLAUDE.md
    participant Rules as Rules
    participant Cmd as Command
    participant Skill as Skill
    participant Agent as Agent
    participant SubAgent as Subagent
    participant Hooks as Hooks
    participant Memory as Memory
    participant MCP as MCP Server

    Note over Session: Session Start
    CLAUDE->>Session: Load constitution
    Rules->>Session: Load policies
    Memory->>Session: Load memory index

    User->>Session: /cne-cmd-dev-fix auth bug

    Session->>CLAUDE: What kind of request?
    CLAUDE-->>Session: Route to dev-fix command

    Session->>Cmd: Load cne-cmd-dev-fix.md
    Cmd-->>Session: Procedure loaded

    Session->>Skill: Load cne-skl-core-debug
    Skill-->>Session: Debug methodology loaded

    Session->>Agent: Dispatch cne-agt-dev-debugger
    Note over Agent: Agent context isolated
    Agent->>Hooks: PreToolUse fires
    Hooks-->>Agent: Allowed
    Agent->>MCP: Search Jira for context
    MCP-->>Agent: Issue details
    Agent->>Agent: Diagnose issue
    Agent->>Hooks: PostToolUse fires
    Agent-->>Session: Diagnosis result

    Session->>Agent: Dispatch cne-agt-dev-fixer
    Note over Agent: New agent, new context
    Agent->>Agent: Implement fix
    Agent->>Hooks: PostToolUse (format)
    Agent-->>Session: Fix implemented

    Session->>Agent: Dispatch cne-agt-qa-tester
    Agent->>Agent: Write & run tests
    Agent-->>Session: Tests pass

    Session->>Memory: Save feedback
    Session->>User: Fix complete, tests pass
```

---

## 4. The Self-Improvement Loop

How Claudine gets better over time.

```mermaid
graph LR
    subgraph SESSION["Session (Work Phase)"]
        WORK["Execute User Requests"]
        LOG["Log Execution Data"]
        CORRECT["Capture User Corrections"]
    end

    subgraph CAPTURE["Capture Phase"]
        SESSION_LOG["CNE-SESSION-LOG.md"]
        FEEDBACK["CNE-FEEDBACK.md"]
        METRICS["CNE-METRICS.md"]
        ISSUES["CNE-KNOWN-ISSUES.md"]
    end

    subgraph ANALYZE["Analysis Phase"]
        IMPROVER["cne-agt-core-improver"]
        VALIDATOR["cne-agt-core-validator"]
        COVERAGE["cne-agt-core-coverage-checker"]
    end

    subgraph APPLY["Application Phase"]
        UPDATE["Update Agents/Commands/Skills"]
        VALIDATE_2["Run Validation Scripts"]
        CHANGELOG["Update CNE-CHANGELOG.md"]
    end

    WORK --> LOG
    WORK --> CORRECT
    LOG --> SESSION_LOG
    CORRECT --> FEEDBACK
    LOG --> METRICS

    SESSION_LOG --> IMPROVER
    FEEDBACK --> IMPROVER
    METRICS --> IMPROVER
    ISSUES --> IMPROVER

    SESSION_LOG --> VALIDATOR
    SESSION_LOG --> COVERAGE

    IMPROVER --> UPDATE
    VALIDATOR --> UPDATE
    COVERAGE --> UPDATE

    UPDATE --> VALIDATE_2
    VALIDATE_2 --> CHANGELOG

    CHANGELOG -.->|next session reads| SESSION
```

---

## 5. Flow: Bug Fix (Waterfall)

The `/cne-cmd-dev-fix` command -- a pure waterfall flow.

```mermaid
graph TD
    INPUT[/"User: /cne-cmd-dev-fix 'auth bug'"/]

    SKILL["Load cne-skl-core-debug<br/>(debugging methodology)"]

    STEP1["Step 1: DIAGNOSE<br/>cne-agt-dev-debugger<br/>Input: bug report<br/>Output: root cause analysis"]

    GATE1{"Diagnosis<br/>conclusive?"}

    STEP2["Step 2: FIX<br/>cne-agt-dev-fixer<br/>Input: diagnosis<br/>Output: code changes"]

    STEP3["Step 3: TEST<br/>cne-agt-qa-tester<br/>Input: code changes<br/>Output: test results"]

    GATE2{"Tests<br/>pass?"}

    STEP4["Step 4: REVIEW<br/>cne-agt-dev-reviewer<br/>Input: changes + tests<br/>Output: approval/rejection"]

    GATE3{"Approved?"}

    COMMIT["Step 5: COMMIT<br/>/cne-cmd-git-commit"]

    OUTPUT[/"Bug fixed, committed"/]
    FAIL[/"Report failure to user"/]

    INPUT --> SKILL
    SKILL --> STEP1
    STEP1 --> GATE1
    GATE1 -->|YES| STEP2
    GATE1 -->|NO| FAIL
    STEP2 --> STEP3
    STEP3 --> GATE2
    GATE2 -->|YES| STEP4
    GATE2 -->|NO, loop back| STEP2
    STEP4 --> GATE3
    GATE3 -->|YES| COMMIT
    GATE3 -->|NO, loop back| STEP2
    COMMIT --> OUTPUT
```

---

## 6. Flow: Full Audit (Swarm)

The `/cne-cmd-qa-audit` command -- a pure swarm flow with consolidation.

```mermaid
graph TD
    INPUT[/"User: /cne-cmd-qa-audit"/]

    DISPATCH["ORCHESTRATOR<br/>Dispatch all auditors<br/>in parallel (single message)"]

    subgraph PARALLEL["Parallel Execution (Swarm)"]
        A1["cne-agt-sec-auditor<br/>Security Audit"]
        A2["cne-agt-qa-auditor<br/>Code Quality Audit"]
        A3["cne-agt-dev-reviewer<br/>Pattern Audit"]
        A4["cne-agt-ui-ux-guardian<br/>UX Audit"]
        A5["cne-agt-seo-auditor<br/>SEO Audit"]
    end

    COLLECT["Collect all results<br/>(wait for all agents)"]

    CONSOLIDATE["CONSOLIDATE<br/>- Deduplicate findings<br/>- Resolve conflicts<br/>- Prioritize by severity<br/>- Format unified report"]

    OUTPUT[/"Unified Audit Report<br/>CRITICAL: n, HIGH: n,<br/>MEDIUM: n, LOW: n"/]

    INPUT --> DISPATCH
    DISPATCH --> A1
    DISPATCH --> A2
    DISPATCH --> A3
    DISPATCH --> A4
    DISPATCH --> A5
    A1 --> COLLECT
    A2 --> COLLECT
    A3 --> COLLECT
    A4 --> COLLECT
    A5 --> COLLECT
    COLLECT --> CONSOLIDATE
    CONSOLIDATE --> OUTPUT
```

---

## 7. Flow: New Feature (Hybrid)

The `/cne-cmd-dev-feature` command -- waterfall phases with swarm sub-phases.

```mermaid
graph TD
    INPUT[/"User: /cne-cmd-dev-feature 'add OAuth'"/]

    subgraph PHASE1["Phase 1: ANALYZE (Waterfall)"]
        P1["cne-agt-ba-analyst<br/>Analyze requirements"]
    end

    subgraph PHASE2["Phase 2: MULTI-REVIEW (Swarm)"]
        P2A["cne-agt-arc-designer<br/>Architecture review"]
        P2B["cne-agt-sec-reviewer<br/>Security review"]
        P2C["cne-agt-dev-reviewer<br/>Feasibility review"]
        P2MERGE["Consolidate reviews"]
    end

    subgraph PHASE3["Phase 3: IMPLEMENT (Waterfall)"]
        P3_TDD["Load cne-skl-qa-tdd"]
        P3_TEST["cne-agt-qa-tester<br/>Write tests first (RED)"]
        P3_IMPL["cne-agt-dev-implementer<br/>Implement feature (GREEN)"]
        P3_REFACTOR["Refactor if needed"]
    end

    subgraph PHASE4["Phase 4: VERIFY (Swarm)"]
        P4A["cne-agt-qa-tester<br/>Unit tests"]
        P4B["cne-agt-qa-e2e-runner<br/>E2E tests"]
        P4C["cne-agt-sec-auditor<br/>Security scan"]
        P4MERGE["Consolidate verification"]
    end

    GATE{"All<br/>verified?"}

    subgraph PHASE5["Phase 5: FINALIZE (Waterfall)"]
        P5_REVIEW["cne-agt-dev-reviewer<br/>Final review"]
        P5_DOCS["cne-agt-twr-writer<br/>Documentation"]
        P5_COMMIT["/cne-cmd-git-pr<br/>Create PR"]
    end

    OUTPUT[/"Feature complete,<br/>PR created"/]

    INPUT --> P1
    P1 --> P2A
    P1 --> P2B
    P1 --> P2C
    P2A --> P2MERGE
    P2B --> P2MERGE
    P2C --> P2MERGE
    P2MERGE --> P3_TDD
    P3_TDD --> P3_TEST
    P3_TEST --> P3_IMPL
    P3_IMPL --> P3_REFACTOR
    P3_REFACTOR --> P4A
    P3_REFACTOR --> P4B
    P3_REFACTOR --> P4C
    P4A --> P4MERGE
    P4B --> P4MERGE
    P4C --> P4MERGE
    P4MERGE --> GATE
    GATE -->|YES| P5_REVIEW
    GATE -->|NO| P3_IMPL
    P5_REVIEW --> P5_DOCS
    P5_DOCS --> P5_COMMIT
    P5_COMMIT --> OUTPUT
```

---

## 8. Flow: Sprint Planning

The `/cne-cmd-pm-sprint` command -- waterfall with information gathering.

```mermaid
graph TD
    INPUT[/"User: /cne-cmd-pm-sprint"/]

    CONTEXT["Phase 1: GATHER CONTEXT<br/>cne-agt-pm-planner<br/>- Read Jira backlog (MCP)<br/>- Read last sprint retro<br/>- Read team capacity"]

    PRIORITIZE["Phase 2: PRIORITIZE<br/>cne-agt-po-strategist<br/>- Score tickets (RICE)<br/>- Check dependencies<br/>- Assess risks"]

    ASSIGN["Phase 3: PLAN<br/>cne-agt-pm-planner<br/>- Assign to sprint<br/>- Set story points<br/>- Create subtasks"]

    REVIEW["Phase 4: REVIEW<br/>Present plan to user<br/>- Sprint goal<br/>- Committed items<br/>- Risks identified"]

    OUTPUT[/"Sprint plan document<br/>+ Jira updated"/]

    INPUT --> CONTEXT
    CONTEXT --> PRIORITIZE
    PRIORITIZE --> ASSIGN
    ASSIGN --> REVIEW
    REVIEW --> OUTPUT
```

---

## 9. Flow: Release Preparation

The `/cne-cmd-dop-release` command -- hybrid flow.

```mermaid
graph TD
    INPUT[/"User: /cne-cmd-dop-release"/]

    subgraph VERIFY["Phase 1: VERIFY (Swarm)"]
        V1["cne-agt-qa-tester<br/>Run all tests"]
        V2["cne-agt-sec-auditor<br/>Security scan"]
        V3["cne-agt-dop-infra-checker<br/>Infra readiness"]
        V4["cne-agt-seo-auditor<br/>SEO check"]
        VMERGE["Consolidate results"]
    end

    GATE1{"All<br/>green?"}

    subgraph PREPARE["Phase 2: PREPARE (Waterfall)"]
        P1["cne-agt-twr-writer<br/>Generate changelog"]
        P2["cne-agt-pm-reporter<br/>Release notes"]
        P3["Version bump<br/>Update package files"]
    end

    subgraph DEPLOY["Phase 3: DEPLOY (Waterfall)"]
        D1["cne-agt-dop-deployer<br/>Deploy to staging"]
        D2["cne-agt-qa-e2e-runner<br/>Smoke test staging"]
        D3["cne-agt-dop-deployer<br/>Deploy to production"]
        D4["cne-agt-dop-infra-checker<br/>Health check production"]
    end

    OUTPUT[/"Release complete<br/>+ changelog + notes"/]
    BLOCKED[/"Release blocked<br/>Fix issues first"/]

    INPUT --> V1
    INPUT --> V2
    INPUT --> V3
    INPUT --> V4
    V1 --> VMERGE
    V2 --> VMERGE
    V3 --> VMERGE
    V4 --> VMERGE
    VMERGE --> GATE1
    GATE1 -->|YES| P1
    GATE1 -->|NO| BLOCKED
    P1 --> P2
    P2 --> P3
    P3 --> D1
    D1 --> D2
    D2 --> D3
    D3 --> D4
    D4 --> OUTPUT
```

---

## 10. Flow: Project Onboarding

The `/cne-cmd-core-adjust` command -- adapting Claudine to a new project.

```mermaid
graph TD
    INPUT[/"User: /cne-cmd-core-adjust"/]

    SCAN["Phase 1: SCAN PROJECT<br/>- Read package.json/build files<br/>- Detect tech stack<br/>- Map directory structure<br/>- Read existing docs"]

    subgraph ANALYZE["Phase 2: ANALYZE (Swarm)"]
        A1["Detect languages<br/>& frameworks"]
        A2["Detect testing<br/>tools & patterns"]
        A3["Detect CI/CD<br/>& deployment"]
        A4["Detect coding<br/>conventions"]
    end

    CONFIGURE["Phase 3: CONFIGURE<br/>- Enable relevant skills<br/>- Disable irrelevant skills<br/>- Update CLAUDE.md<br/>- Adjust hook scripts<br/>- Set model preferences"]

    VALIDATE["Phase 4: VALIDATE<br/>- Run self-audit<br/>- Verify references<br/>- Test simple command"]

    OUTPUT[/"Claudine adapted<br/>to project"/]

    INPUT --> SCAN
    SCAN --> A1
    SCAN --> A2
    SCAN --> A3
    SCAN --> A4
    A1 --> CONFIGURE
    A2 --> CONFIGURE
    A3 --> CONFIGURE
    A4 --> CONFIGURE
    CONFIGURE --> VALIDATE
    VALIDATE --> OUTPUT
```

---

## 11. Flow: Self-Improvement

The `/cne-cmd-core-improve` command -- Claudine improving itself.

```mermaid
graph TD
    INPUT[/"User: /cne-cmd-core-improve<br/>or Stop hook triggers"/]

    GATHER["Phase 1: GATHER<br/>Read session logs,<br/>feedback, metrics,<br/>known issues"]

    subgraph ANALYZE["Phase 2: ANALYZE (Swarm)"]
        IMP["cne-agt-core-improver<br/>Find improvement<br/>opportunities"]
        VAL["cne-agt-core-validator<br/>Run self-audit"]
        COV["cne-agt-core-coverage-checker<br/>Find coverage gaps"]
    end

    PRIORITIZE["Phase 3: PRIORITIZE<br/>Rank by impact,<br/>effort, risk"]

    GATE{"High-risk<br/>changes?"}

    USER_APPROVE["Ask user to approve<br/>high-risk changes"]

    IMPLEMENT["Phase 4: IMPLEMENT<br/>Create/update files"]

    subgraph VALIDATE["Phase 5: VALIDATE (Swarm)"]
        VN["Validate names"]
        VR["Validate refs"]
        VS["Validate skills"]
        VA["Validate agents"]
    end

    RECORD["Phase 6: RECORD<br/>Update CNE-CHANGELOG.md<br/>Update CNE-METRICS.md"]

    OUTPUT[/"Improvements applied<br/>and validated"/]

    INPUT --> GATHER
    GATHER --> IMP
    GATHER --> VAL
    GATHER --> COV
    IMP --> PRIORITIZE
    VAL --> PRIORITIZE
    COV --> PRIORITIZE
    PRIORITIZE --> GATE
    GATE -->|YES| USER_APPROVE
    GATE -->|NO| IMPLEMENT
    USER_APPROVE --> IMPLEMENT
    IMPLEMENT --> VN
    IMPLEMENT --> VR
    IMPLEMENT --> VS
    IMPLEMENT --> VA
    VN --> RECORD
    VR --> RECORD
    VS --> RECORD
    VA --> RECORD
    RECORD --> OUTPUT
```

---

## 12. The Master Flow Map

All Claudine flows and how they connect to each other. This is the complete "company operations map."

```mermaid
graph TB
    USER(("USER"))

    subgraph ENTRY["Entry Points (Commands)"]
        CMD_FIX["/cne-cmd-dev-fix"]
        CMD_FEAT["/cne-cmd-dev-feature"]
        CMD_REFACTOR["/cne-cmd-dev-refactor"]
        CMD_AUDIT["/cne-cmd-qa-audit"]
        CMD_TEST["/cne-cmd-qa-test"]
        CMD_PLAN["/cne-cmd-pm-plan"]
        CMD_SPRINT["/cne-cmd-pm-sprint"]
        CMD_SEC["/cne-cmd-sec-scan"]
        CMD_DEPLOY["/cne-cmd-dop-deploy"]
        CMD_RELEASE["/cne-cmd-dop-release"]
        CMD_DOCS["/cne-cmd-twr-docs"]
        CMD_PR["/cne-cmd-git-pr"]
        CMD_COMMIT["/cne-cmd-git-commit"]
        CMD_ADJUST["/cne-cmd-core-adjust"]
        CMD_IMPROVE["/cne-cmd-core-improve"]
    end

    subgraph FLOWS["Internal Flows"]
        FLOW_BUGFIX["Bug Fix Flow<br/>(Waterfall)"]
        FLOW_FEATURE["Feature Flow<br/>(Hybrid)"]
        FLOW_AUDIT["Audit Flow<br/>(Swarm)"]
        FLOW_PLAN["Planning Flow<br/>(Waterfall)"]
        FLOW_RELEASE["Release Flow<br/>(Hybrid)"]
        FLOW_IMPROVE["Improvement Flow<br/>(Hybrid)"]
        FLOW_ADJUST["Adjustment Flow<br/>(Hybrid)"]
    end

    subgraph SHARED["Shared Steps"]
        COMMIT_STEP["Git Commit"]
        PR_STEP["Create PR"]
        REVIEW_STEP["Code Review"]
        TEST_STEP["Run Tests"]
    end

    USER --> CMD_FIX --> FLOW_BUGFIX
    USER --> CMD_FEAT --> FLOW_FEATURE
    USER --> CMD_AUDIT --> FLOW_AUDIT
    USER --> CMD_PLAN --> FLOW_PLAN
    USER --> CMD_RELEASE --> FLOW_RELEASE
    USER --> CMD_IMPROVE --> FLOW_IMPROVE
    USER --> CMD_ADJUST --> FLOW_ADJUST
    USER --> CMD_COMMIT --> COMMIT_STEP
    USER --> CMD_PR --> PR_STEP

    FLOW_BUGFIX --> TEST_STEP --> REVIEW_STEP --> COMMIT_STEP --> PR_STEP
    FLOW_FEATURE --> TEST_STEP
    FLOW_AUDIT -.->|findings feed into| FLOW_BUGFIX
    FLOW_PLAN -.->|plan feeds into| FLOW_FEATURE
    FLOW_RELEASE -.->|includes| FLOW_AUDIT
    FLOW_IMPROVE -.->|improves all| FLOWS
```

---

## 13. Data Flow: Context Propagation

How data moves between agents, showing what each agent can and cannot see.

```mermaid
graph TD
    subgraph MAIN["Main Session Context"]
        CLAUDE_MD["CLAUDE.md<br/>(always present)"]
        RULES_CTX["Rules<br/>(always present)"]
        USER_HIST["User Conversation<br/>History"]
        SKILL_CTX["Loaded Skills"]
        AGENT_RESULTS["Agent Return<br/>Messages"]
    end

    subgraph AGENT_A["Agent A Context (Isolated)"]
        A_INSTR["Agent A .md<br/>Instructions"]
        A_PROMPT["Prompt from<br/>Parent"]
        A_TOOLS["Agent A's own<br/>Tool Results"]
        A_NO["Cannot see:<br/>- CLAUDE.md<br/>- Rules<br/>- User history<br/>- Other agents"]
    end

    subgraph AGENT_B["Agent B Context (Isolated)"]
        B_INSTR["Agent B .md<br/>Instructions"]
        B_PROMPT["Prompt from<br/>Parent"]
        B_TOOLS["Agent B's own<br/>Tool Results"]
        B_NO["Cannot see:<br/>- Agent A's context<br/>- CLAUDE.md<br/>- Rules"]
    end

    MAIN -->|"prompt (carries context)"| AGENT_A
    MAIN -->|"prompt (carries context)"| AGENT_B
    AGENT_A -->|"return message"| MAIN
    AGENT_B -->|"return message"| MAIN

    AGENT_A -.-x AGENT_B

    style A_NO fill:#fee,stroke:#f66
    style B_NO fill:#fee,stroke:#f66
```

**Key Insight**: Agents are blind to each other and to the main session's accumulated context. The orchestrator (main session) is the ONLY entity that sees everything. It must explicitly include relevant context in each agent's prompt.

---

## Pattern Legend

Use this legend when creating new flow diagrams:

| Shape | Meaning |
|---|---|
| Rectangle | Process / Agent execution |
| Diamond | Decision gate |
| Parallelogram | Input / Output |
| Rounded rectangle | Subgraph / Phase boundary |
| Solid arrow | Data flow / sequence |
| Dashed arrow | Indirect relationship / feeds into |
| Circle | User / external entity |

| Color | Meaning |
|---|---|
| Default (blue) | Normal process |
| Red (#fee) | Warning / cannot-do |
| Green | Success path |
| Orange | Caution / requires approval |

---

## 14. Flow Selection Decision Tree

When a user makes a request, use this tree to determine which command/flow applies:

```mermaid
graph TD
    START(["User has a request"])

    Q1{"Is it a bug or issue?"}
    Q2{"Is it a new feature\nor enhancement?"}
    Q3{"Is it a review\nor audit?"}
    Q4{"Is it planning\nor tracking?"}
    Q5{"Is it deployment\nor release?"}
    Q6{"Is it documentation?"}
    Q7{"Is it Claudine\nmaintenance?"}

    F1["/cne-cmd-dev-fix\n(Waterfall)"]
    F2["/cne-cmd-dev-feature\n(Hybrid)"]
    F3["/cne-cmd-qa-audit\n(Swarm)"]
    F4["/cne-cmd-pm-plan\n(Waterfall)"]
    F5["/cne-cmd-dop-release\n(Hybrid)"]
    F6["/cne-cmd-twr-docs\n(Waterfall)"]
    F7["/cne-cmd-core-improve\n(Hybrid)"]
    F8["Direct main session\n(no flow needed)"]

    START --> Q1
    Q1 -->|YES| F1
    Q1 -->|NO| Q2
    Q2 -->|YES| F2
    Q2 -->|NO| Q3
    Q3 -->|YES| F3
    Q3 -->|NO| Q4
    Q4 -->|YES| F4
    Q4 -->|NO| Q5
    Q5 -->|YES| F5
    Q5 -->|NO| Q6
    Q6 -->|YES| F6
    Q6 -->|NO| Q7
    Q7 -->|YES| F7
    Q7 -->|NO| F8
```

---

## 15. Memory System Lifecycle

How institutional memory is read, written, and connects to sessions:

```mermaid
graph TD
    subgraph SESSION_N["Session N"]
        START_N["Session starts"]
        READ_MEM["Read MEMORY.md index"]
        READ_RELEVANT["Read relevant memory files\n(based on current task)"]
        DO_WORK["Execute user requests"]
        LEARN["Capture new learnings:\n- User corrections\n- Approach confirmations\n- New references"]
        WRITE_MEM["Write to memory:\n- feedback memories\n- project memories\n- reference memories"]
        END_N["Session ends"]
    end

    subgraph MEMORY_STORE["Memory Store (~/.claude/projects/*/memory/)"]
        INDEX["MEMORY.md\n(always loaded)"]
        USER_MEM["user_*.md\n(who the user is)"]
        FEEDBACK_MEM["feedback_*.md\n(what works/doesn't)"]
        PROJECT_MEM["project_*.md\n(ongoing context)"]
        REF_MEM["reference_*.md\n(where to find things)"]
    end

    subgraph SESSION_N1["Session N+1"]
        START_N1["Session starts"]
        READ_MEM_N1["Read MEMORY.md\n(sees new memories from Session N)"]
        BETTER["Better context =\nBetter responses"]
    end

    START_N --> READ_MEM
    READ_MEM --> INDEX
    INDEX --> READ_RELEVANT
    READ_RELEVANT --> USER_MEM
    READ_RELEVANT --> FEEDBACK_MEM
    READ_RELEVANT --> PROJECT_MEM
    READ_RELEVANT --> REF_MEM
    READ_RELEVANT --> DO_WORK
    DO_WORK --> LEARN
    LEARN --> WRITE_MEM
    WRITE_MEM --> FEEDBACK_MEM
    WRITE_MEM --> PROJECT_MEM
    WRITE_MEM --> REF_MEM
    WRITE_MEM --> INDEX
    WRITE_MEM --> END_N

    END_N -.->|"Time passes"| START_N1
    START_N1 --> READ_MEM_N1
    READ_MEM_N1 --> INDEX
    READ_MEM_N1 --> BETTER
```

---

## 16. The Voting Swarm Pattern

For high-stakes decisions where multiple independent opinions are needed:

```mermaid
graph TD
    INPUT[/"Architecture decision:\nShould we use microservices?"/]

    DISPATCH["Dispatch 3 independent\nreviewers (single message)"]

    R1["Reviewer A\n(cne-agt-arc-analyst)\nPerspective: Scalability"]
    R2["Reviewer B\n(cne-agt-sec-reviewer)\nPerspective: Security"]
    R3["Reviewer C\n(cne-agt-dop-deployer)\nPerspective: Operations"]

    V1["VOTE: YES\nReason: Scale needs\njustify complexity"]
    V2["VOTE: NO\nReason: Attack surface\ntoo large"]
    V3["VOTE: YES\nReason: Team size\nsupports it"]

    TALLY["TALLY: 2 YES, 1 NO\nMajority: YES"]

    DETAIL["Consolidate:\n- Summarize all perspectives\n- Note dissenting opinion\n- Include conditions/caveats"]

    OUTPUT[/"Decision: YES (microservices)\nWith conditions from\nsecurity reviewer"/]

    INPUT --> DISPATCH
    DISPATCH --> R1
    DISPATCH --> R2
    DISPATCH --> R3
    R1 --> V1
    R2 --> V2
    R3 --> V3
    V1 --> TALLY
    V2 --> TALLY
    V3 --> TALLY
    TALLY --> DETAIL
    DETAIL --> OUTPUT
```
