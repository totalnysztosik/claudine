# Claudine Execution Patterns: Waterfall and Swarm

## Overview

Claudine supports two fundamental execution patterns for multi-step work: **Waterfall** (sequential, context-carrying) and **Swarm** (parallel, consolidating). This document defines both patterns exhaustively -- the technical mechanics, when to use each, how to implement them in Claude Code, and how to determine which pattern applies to any future flow.

---

## Table of Contents

1. [The Two Patterns at a Glance](#1-the-two-patterns-at-a-glance)
2. [Waterfall Pattern: Deep Dive](#2-waterfall-pattern-deep-dive)
3. [Swarm Pattern: Deep Dive](#3-swarm-pattern-deep-dive)
4. [Hybrid Pattern: Waterfall with Swarm Phases](#4-hybrid-pattern-waterfall-with-swarm-phases)
5. [The Decision Framework](#5-the-decision-framework)
6. [Implementation Mechanics in Claude Code](#6-implementation-mechanics-in-claude-code)
7. [Error Handling and Recovery](#7-error-handling-and-recovery)
8. [Pattern Examples for Every Department](#8-pattern-examples-for-every-department)
9. [Anti-Patterns and Pitfalls](#9-anti-patterns-and-pitfalls)
10. [Template for Defining New Flows](#10-template-for-defining-new-flows)

---

## 1. The Two Patterns at a Glance

| Aspect | Waterfall | Swarm |
|---|---|---|
| **Execution** | Sequential (A → B → C) | Parallel (A + B + C) then merge |
| **Context** | Each step receives output of previous | All steps receive same initial input |
| **Dependencies** | Step N depends on Step N-1 | Steps are independent of each other |
| **Speed** | Slower (serial) | Faster (parallel) |
| **Accuracy** | Higher (full context chain) | Variable (needs consolidation) |
| **Complexity** | Simple to implement | Complex to orchestrate |
| **Agent type** | Chain of specialists | Team of specialists + consolidator |
| **Company analogy** | Assembly line | War room / brainstorm |
| **Best for** | Dependent tasks, workflows | Independent analysis, reviews |

---

## 2. Waterfall Pattern: Deep Dive

### Definition
The Waterfall pattern executes tasks **sequentially**, where each step MUST complete before the next begins, and each subsequent step receives the **full output** of the previous step as input context.

### Visual Model
```
INPUT
  │
  ▼
┌─────────────────┐
│ Step 1: Diagnose │ (cne-agt-dev-debugger)
│ Input: bug report│
│ Output: diagnosis│
└────────┬────────┘
         │ diagnosis passed forward
         ▼
┌─────────────────┐
│ Step 2: Fix      │ (cne-agt-dev-fixer)
│ Input: diagnosis │
│ Output: fix diff │
└────────┬────────┘
         │ fix diff passed forward
         ▼
┌─────────────────┐
│ Step 3: Test     │ (cne-agt-qa-tester)
│ Input: fix diff  │
│ Output: test     │
│         results  │
└────────┬────────┘
         │ test results passed forward
         ▼
┌─────────────────┐
│ Step 4: Review   │ (cne-agt-dev-reviewer)
│ Input: fix diff  │
│   + test results │
│ Output: approval │
│   or rejection   │
└────────┬────────┘
         │
         ▼
       OUTPUT
```

### Technical Mechanics

#### How Context Carries Forward
In Claude Code, the Waterfall pattern is implemented in the **main session** (or an orchestrator agent), which calls agents sequentially and passes each agent's return message to the next:

```
MAIN SESSION:

1. result_1 = Agent(cne-agt-dev-debugger, prompt="Diagnose: {bug_report}")
   // Wait for result_1

2. result_2 = Agent(cne-agt-dev-fixer, prompt="Fix based on diagnosis: {result_1}")
   // Wait for result_2

3. result_3 = Agent(cne-agt-qa-tester, prompt="Test this fix: {result_2}")
   // Wait for result_3

4. result_4 = Agent(cne-agt-dev-reviewer, prompt="Review fix and tests: {result_2} {result_3}")
   // Wait for result_4

5. Return result_4 to user
```

#### The Context Accumulation Problem
Each agent returns a result that becomes part of the main session's context. As the waterfall progresses, context accumulates:

```
After Step 1: main context = [original request] + [result_1]
After Step 2: main context = [original request] + [result_1] + [result_2]
After Step 3: main context = [original request] + [result_1] + [result_2] + [result_3]
After Step 4: main context = [original request] + [result_1] + [result_2] + [result_3] + [result_4]
```

**CRITICAL**: This means long waterfalls consume significant context window. Mitigations:
1. **Summarize between steps**: Have the main session summarize key findings before passing to next agent
2. **Only pass relevant context**: Don't pass ALL previous results -- only what the next step needs
3. **Use tasks for tracking**: Create tasks for each step so the user sees progress without the main session needing to remember everything

#### Agent Return Quality
Each agent in a waterfall chain must return **structured, comprehensive output** because:
- The next agent depends on this output as its primary input
- The main session needs to extract the right information to pass forward
- The final output depends on every intermediate output being complete

**Agent return format for waterfall steps**:
```markdown
## Summary
[One paragraph of what was done]

## Findings
[Detailed findings/results]

## Key Data for Next Step
[Explicitly structured data that the next step needs]

## Concerns/Blockers
[Anything that should stop or redirect the waterfall]
```

### When to Use Waterfall

Use Waterfall when ANY of these are true:
1. **Step N cannot begin without Step N-1's output** -- e.g., can't fix a bug without first diagnosing it
2. **Quality compounds** -- each step builds on and improves the previous step's work
3. **Order matters for correctness** -- reviewing before testing makes no sense
4. **Rollback is needed** -- if Step 3 fails, you need to go back to Step 2 with that information
5. **The task is inherently sequential** -- write → review → fix → re-review

### Waterfall Variants

#### Linear Waterfall
The standard A → B → C → D pattern. Each step happens once.

#### Waterfall with Loops
A step can send the flow back to a previous step:
```
Diagnose → Fix → Test → [FAIL] → Fix (again, with test failure info) → Test → [PASS] → Review
```

Implementation: The main session checks each step's output for success/failure signals and routes accordingly.

#### Waterfall with Gates
A quality gate between steps that can halt the flow:
```
Diagnose → [GATE: is diagnosis conclusive?] → Fix → [GATE: is fix safe?] → Test → ...
```

Implementation: The main session evaluates each result before dispatching the next agent. If a gate fails, the flow stops and the user is notified.

---

## 3. Swarm Pattern: Deep Dive

### Definition
The Swarm pattern dispatches **multiple agents simultaneously** to work on independent aspects of a task, then **consolidates** their results into a unified output. An orchestrator (either the main session or a dedicated orchestrator agent) manages the dispatch and consolidation.

### Visual Model
```
INPUT
  │
  ▼
┌─────────────────────────────────────┐
│         ORCHESTRATOR                 │
│    (cne-agt-core-orchestrator        │
│     or main session)                 │
└──┬────────┬────────┬────────┬───────┘
   │        │        │        │
   ▼        ▼        ▼        ▼        (PARALLEL)
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│Agent │ │Agent │ │Agent │ │Agent │
│  A   │ │  B   │ │  C   │ │  D   │
│(sec) │ │(perf)│ │(code)│ │(a11y)│
└──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘
   │        │        │        │
   ▼        ▼        ▼        ▼        (RESULTS)
┌─────────────────────────────────────┐
│         CONSOLIDATION                │
│    (orchestrator merges all results) │
│    - Deduplicates                    │
│    - Resolves conflicts              │
│    - Prioritizes                     │
│    - Formats unified output          │
└──────────────┬──────────────────────┘
               │
               ▼
            OUTPUT
```

### Technical Mechanics

#### Parallel Dispatch
In Claude Code, parallel dispatch uses **multiple Agent tool calls in a single message**:

```
MAIN SESSION (single message with multiple tool calls):

Agent(cne-agt-sec-auditor, prompt="Audit security of: {codebase}", run_in_background=true)
Agent(cne-agt-qa-auditor, prompt="Audit code quality of: {codebase}", run_in_background=true)
Agent(cne-agt-dev-reviewer, prompt="Review code patterns in: {codebase}", run_in_background=true)
Agent(cne-agt-ui-ux-guardian, prompt="Audit UX of: {codebase}", run_in_background=true)

// All four agents run SIMULTANEOUSLY
// Main session is notified as each completes
```

**CRITICAL IMPLEMENTATION DETAIL**: To run agents truly in parallel, ALL Agent tool calls must be in a **single message**. If you put them in separate messages, they execute sequentially.

#### Background vs. Foreground Dispatch
| Mode | How | When |
|---|---|---|
| **Foreground (parallel)** | Multiple Agent calls in one message, no `run_in_background` | When you need all results before proceeding |
| **Background** | `run_in_background=true` | When you have other work to do while agents run |
| **Mixed** | Some foreground, some background | When some results are needed immediately, others can wait |

#### The Consolidation Phase
After all swarm agents return, the orchestrator must **consolidate** their results. This is the hardest part of the swarm pattern. The consolidation must:

1. **Collect all results**: Wait for every agent to complete
2. **Parse results**: Extract structured findings from each agent's output
3. **Deduplicate**: Multiple agents may find the same issue
4. **Resolve conflicts**: Agent A says "this is fine," Agent B says "this is a problem"
5. **Prioritize**: Rank findings by severity/importance
6. **Format**: Produce a single, coherent output

**Consolidation approaches**:

| Approach | How | Best For |
|---|---|---|
| **Main session consolidates** | Main session reads all results, synthesizes | Simple swarms (2-3 agents) |
| **Dedicated consolidator agent** | A separate `cne-agt-core-consolidator` agent receives all results | Complex swarms (4+ agents) |
| **Structured output merge** | Each agent returns JSON/structured data, merge programmatically | Data-heavy results |

#### The Input Uniformity Requirement
In a swarm, **every agent receives the same input** (or a defined subset). This is fundamentally different from waterfall where each step transforms the input:

```
WATERFALL: Input → [Transform A] → Transformed → [Transform B] → ...
SWARM:     Input → [Analyze A] → Result A
           Input → [Analyze B] → Result B    (same input, different lens)
           Input → [Analyze C] → Result C
```

Each swarm agent looks at the SAME thing from a DIFFERENT perspective. They don't modify the input; they analyze it.

### When to Use Swarm

Use Swarm when ALL of these are true:
1. **Steps are independent** -- Agent A doesn't need Agent B's output
2. **Same input, different perspectives** -- each agent analyzes the same thing differently
3. **No ordering requirement** -- it doesn't matter who finishes first
4. **Speed matters** -- parallel execution is significantly faster
5. **Consolidation is feasible** -- results can be meaningfully merged

### Swarm Variants

#### Full Swarm (All Parallel)
Every agent runs simultaneously. All results consolidated at once.

#### Staged Swarm
Agents are dispatched in groups. Group 1 runs in parallel. After Group 1 completes, Group 2 runs in parallel (possibly using Group 1's consolidated output):
```
Stage 1 (parallel): [Audit Security] + [Audit Code] + [Audit Perf]
  → Consolidate Stage 1 findings
Stage 2 (parallel): [Fix Security Issues] + [Fix Code Issues] + [Fix Perf Issues]
  → Consolidate Stage 2 fixes
```

This is actually a **Hybrid** pattern (see Section 4).

#### Competitive Swarm
Multiple agents work on the **same task** independently, and the best result is selected:
```
[Agent A writes solution]  +  [Agent B writes solution]  +  [Agent C writes solution]
  → Consolidator picks the best solution (or merges best parts)
```

Use this for creative tasks where multiple approaches should be explored.

#### Voting Swarm
Multiple agents independently evaluate the same thing, and decisions are made by majority:
```
[Reviewer A: APPROVE]  +  [Reviewer B: REJECT]  +  [Reviewer C: APPROVE]
  → Majority: APPROVE (2-1)
```

Use this for high-stakes decisions (security approval, architecture review).

---

## 4. Hybrid Pattern: Waterfall with Swarm Phases

### Definition
The most powerful pattern combines both: a **waterfall** of sequential phases, where one or more phases use the **swarm** pattern internally.

### Visual Model
```
INPUT
  │
  ▼
┌─────────────────┐
│ Phase 1: Analyze │ ← WATERFALL STEP (single agent)
│ (cne-agt-ba-    │
│  analyst)        │
└────────┬────────┘
         │ analysis results
         ▼
┌─────────────────────────┐
│ Phase 2: Multi-Review    │ ← SWARM PHASE (parallel)
│ ┌─────┐ ┌─────┐ ┌─────┐│
│ │ Sec │ │Perf │ │Code ││
│ │Audit│ │Audit│ │Audit││
│ └──┬──┘ └──┬──┘ └──┬──┘│
│    └────┬───┘───────┘   │
│    Consolidate           │
└────────┬────────────────┘
         │ consolidated review
         ▼
┌─────────────────┐
│ Phase 3: Fix     │ ← WATERFALL STEP (single agent, uses review)
│ (cne-agt-dev-   │
│  fixer)          │
└────────┬────────┘
         │ fixes applied
         ▼
┌─────────────────────────┐
│ Phase 4: Verify          │ ← SWARM PHASE (parallel)
│ ┌─────┐ ┌─────┐ ┌─────┐│
│ │Unit │ │E2E  │ │Sec  ││
│ │Test │ │Test │ │Scan ││
│ └──┬──┘ └──┬──┘ └──┬──┘│
│    └────┬───┘───────┘   │
│    Consolidate           │
└────────┬────────────────┘
         │
         ▼
       OUTPUT
```

### Implementation
The main session (or orchestrator) manages the high-level waterfall, switching between single-agent steps and multi-agent swarm phases:

```
MAIN SESSION:

// Phase 1: Waterfall step (single agent)
analysis = Agent(cne-agt-ba-analyst, prompt="Analyze: {input}")

// Phase 2: Swarm phase (parallel agents in ONE message)
sec_review = Agent(cne-agt-sec-auditor, prompt="Review: {analysis}")
perf_review = Agent(cne-agt-qa-auditor, prompt="Review: {analysis}")
code_review = Agent(cne-agt-dev-reviewer, prompt="Review: {analysis}")
// Wait for all three
consolidated_review = synthesize(sec_review, perf_review, code_review)

// Phase 3: Waterfall step (uses consolidated swarm output)
fixes = Agent(cne-agt-dev-fixer, prompt="Fix issues: {consolidated_review}")

// Phase 4: Swarm phase (verify fixes in parallel)
unit_test = Agent(cne-agt-qa-tester, prompt="Unit test: {fixes}")
e2e_test = Agent(cne-agt-qa-e2e-runner, prompt="E2E test: {fixes}")
sec_scan = Agent(cne-agt-sec-auditor, prompt="Re-scan: {fixes}")
// Wait for all three
final_verdict = synthesize(unit_test, e2e_test, sec_scan)
```

---

## 5. The Decision Framework

When designing a new flow, use this decision tree to determine which pattern to use:

### Decision Tree

```
START: You have a multi-step task
│
├── Q1: Can ALL steps run with only the original input (no inter-step dependencies)?
│   ├── YES → SWARM (all parallel)
│   └── NO → Continue to Q2
│
├── Q2: Can SOME steps run independently while others depend on previous results?
│   ├── YES → HYBRID (waterfall with swarm phases)
│   │   └── Group independent steps into swarm phases
│   │       Chain dependent steps as waterfall
│   └── NO → Continue to Q3
│
├── Q3: Does EVERY step depend on the previous step's output?
│   ├── YES → WATERFALL (fully sequential)
│   └── NO → Re-examine -- you likely have a hybrid case
│
└── Q4: Are there steps that explore ALTERNATIVES (not perspectives)?
    ├── YES → COMPETITIVE SWARM within a waterfall
    └── NO → Standard pattern from Q1-Q3
```

### Heuristic Rules

| Signal | Pattern |
|---|---|
| "First do X, then based on X do Y" | Waterfall |
| "Analyze X from multiple angles" | Swarm |
| "Try multiple approaches to X" | Competitive Swarm |
| "Review from multiple perspectives, then fix" | Hybrid |
| "Write, then test, then fix, then re-test" | Waterfall with loops |
| "Audit everything in parallel" | Full Swarm |
| "Diagnose, then fix in parallel, then verify" | Hybrid (W → S → W) |
| "Get approval from multiple reviewers" | Voting Swarm |

### The Independence Test

For any two steps A and B, ask: **"Can Step B produce a correct, complete result WITHOUT knowing what Step A found?"**

- **YES for ALL pairs** → Swarm
- **YES for some pairs** → Hybrid (group independent steps)
- **NO for ALL pairs** → Waterfall

### The Ordering Test

For any two steps A and B, ask: **"Would the result change if we swapped the order?"**

- **NO for ALL pairs** → Swarm (order doesn't matter)
- **YES for some pairs** → Those pairs must be in waterfall order
- **YES for ALL pairs** → Waterfall

---

## 6. Implementation Mechanics in Claude Code

### Waterfall Implementation: The Sequential Dispatch Pattern

```markdown
# In a command file (cne-cmd-dev-fix.md):

## Procedure (Waterfall)
Execute the following steps in strict sequence. Each step MUST complete
before the next begins. Pass the relevant output from each step to the next.

### Step 1: Diagnose
Dispatch `cne-agt-dev-debugger` with the following prompt:
"Diagnose this issue: $ARGUMENTS. Identify root cause, affected files,
and severity. Return structured findings."

Wait for the agent to return. Extract the diagnosis.

### Step 2: Plan Fix
Dispatch `cne-agt-dev-implementer` with:
"Based on this diagnosis: [paste diagnosis from Step 1],
plan and implement a fix. Return the fix diff and explanation."

Wait for the agent to return. Extract the fix details.

### Step 3: Test
Dispatch `cne-agt-qa-tester` with:
"Verify this fix: [paste fix from Step 2].
Write tests, run them, confirm the fix works. Return test results."

Wait for the agent to return. Check if tests pass.

### Step 4: Gate
If tests FAIL: Go back to Step 2 with test failure details.
If tests PASS: Proceed to Step 5.

### Step 5: Review
Dispatch `cne-agt-dev-reviewer` with:
"Review this fix and tests: [paste fix from Step 2] [paste tests from Step 3].
Check code quality, security, performance. Return approval or issues."
```

### Swarm Implementation: The Parallel Dispatch Pattern

```markdown
# In a command file (cne-cmd-qa-full-audit.md):

## Procedure (Swarm)
Execute ALL of the following audits SIMULTANEOUSLY by dispatching
all agents in a SINGLE message (this is critical for true parallelism).

### Parallel Agents
Launch all in one message:
1. `cne-agt-sec-auditor`: "Audit security of the codebase. Focus on OWASP Top 10."
2. `cne-agt-qa-auditor`: "Audit code quality. Check complexity, duplication, standards."
3. `cne-agt-dev-reviewer`: "Review code patterns. Check architecture, SOLID, naming."
4. `cne-agt-ui-ux-guardian`: "Audit UX. Check accessibility, responsiveness, usability."
5. `cne-agt-seo-auditor`: "Audit SEO. Check meta tags, performance, structure."

### Consolidation
After ALL agents have returned, consolidate their findings:
1. Merge all findings into a single list
2. Deduplicate (multiple agents may find the same issue)
3. Categorize by severity (CRITICAL, HIGH, MEDIUM, LOW)
4. Sort by severity descending
5. Format as a unified audit report
```

### Hybrid Implementation: Waterfall + Swarm Phases

```markdown
# In a command file (cne-cmd-dev-feature.md):

## Procedure (Hybrid: Waterfall with Swarm Phases)

### Phase 1: Requirements Analysis (Waterfall - Single Agent)
Dispatch `cne-agt-ba-analyst` to analyze the feature request.
Wait for completion. Extract requirements spec.

### Phase 2: Architecture Review (Swarm - Parallel Agents)
In a SINGLE message, dispatch:
1. `cne-agt-arc-designer`: "Design architecture for: [requirements]"
2. `cne-agt-sec-reviewer`: "Evaluate security implications of: [requirements]"
3. `cne-agt-dev-reviewer`: "Assess implementation feasibility of: [requirements]"
Wait for all three. Consolidate into an architecture plan.

### Phase 3: Implementation (Waterfall - Sequential)
Dispatch `cne-agt-dev-implementer` with the architecture plan.
Wait for completion. Extract the implementation.

### Phase 4: Verification (Swarm - Parallel Agents)
In a SINGLE message, dispatch:
1. `cne-agt-qa-tester`: "Write and run unit tests for: [implementation]"
2. `cne-agt-qa-e2e-runner`: "Run E2E tests for: [implementation]"
3. `cne-agt-sec-auditor`: "Scan implementation for vulnerabilities"
Wait for all three. Consolidate into a verification report.

### Phase 5: Final Review (Waterfall - Gate)
If verification PASSES: Dispatch `cne-agt-dev-reviewer` for final review.
If verification FAILS: Go back to Phase 3 with failure details.
```

### Git Worktree Isolation for Swarm Agents

When swarm agents need to MODIFY files (not just read), use git worktrees to prevent conflicts:

```markdown
Launch agents with isolation:
1. Agent(cne-agt-dev-fixer, isolation="worktree", prompt="Fix security issues")
2. Agent(cne-agt-dev-fixer, isolation="worktree", prompt="Fix performance issues")
3. Agent(cne-agt-dev-fixer, isolation="worktree", prompt="Fix code quality issues")

Each agent gets its OWN copy of the repo. After all complete:
- Review each worktree's changes
- Merge non-conflicting changes
- Resolve conflicts manually
```

---

## 7. Error Handling and Recovery

### Waterfall Error Handling

| Error Type | Recovery Strategy |
|---|---|
| Agent timeout | Retry once with simplified prompt. If fails again, stop waterfall and report. |
| Agent returns low-quality result | Loop back to same step with feedback: "Previous attempt was insufficient because..." |
| Agent identifies blocker | Stop waterfall, report blocker to user, suggest alternatives. |
| Context window exhaustion | Summarize accumulated context, continue with summary. |
| Step produces unexpected output | Have main session validate output structure before passing to next step. |

### Swarm Error Handling

| Error Type | Recovery Strategy |
|---|---|
| One agent fails, others succeed | Consolidate available results. Note the gap. Optionally retry failed agent. |
| Multiple agents fail | Retry failed agents once. If still failing, consolidate what's available. |
| Agents return conflicting results | Flag conflicts explicitly in consolidation. Let user decide. |
| One agent takes much longer | Don't wait indefinitely. Set timeout. Consolidate available results. |
| Results are too large to consolidate | Have a dedicated consolidator agent process results. |

### Hybrid Error Handling
Apply waterfall error handling to waterfall phases and swarm error handling to swarm phases. Additionally:
- If a swarm phase fails: The waterfall can retry the swarm phase or skip to the next waterfall step with a warning
- If a waterfall step fails after a swarm phase: Don't re-run the swarm -- use cached results

---

## 8. Pattern Examples for Every Department

### Development (dev)

| Flow | Pattern | Steps |
|---|---|---|
| Bug Fix | Waterfall | Diagnose → Fix → Test → Review |
| Code Review | Swarm | Security + Quality + Patterns + Performance audits in parallel |
| New Feature | Hybrid | Analyze(W) → Multi-Review(S) → Implement(W) → Multi-Verify(S) |
| Refactoring | Waterfall | Identify → Plan → Refactor → Verify → Review |

### Quality Assurance (qa)

| Flow | Pattern | Steps |
|---|---|---|
| Full Audit | Swarm | Security + Code + UX + SEO + A11y audits in parallel |
| TDD Cycle | Waterfall | Write Test → Run (RED) → Implement → Run (GREEN) → Refactor |
| Pre-Deploy | Hybrid | Build(W) → Multi-Test(S) → Gate(W) |

### Project Management (pm)

| Flow | Pattern | Steps |
|---|---|---|
| Sprint Planning | Waterfall | Gather Context → Prioritize → Assign → Document |
| Retrospective | Swarm | Gather from multiple sources in parallel → Consolidate |
| Feature Discovery | Hybrid | Research(W) → Multi-Perspective(S) → Synthesize(W) |

### Security (sec)

| Flow | Pattern | Steps |
|---|---|---|
| Vulnerability Scan | Swarm | OWASP + Dependencies + Secrets + Config scans in parallel |
| Threat Model | Waterfall | Enumerate Assets → Map Threats → Assess Risk → Document |
| Penetration Test | Waterfall | Recon → Identify → Exploit → Report |

### Architecture (arc)

| Flow | Pattern | Steps |
|---|---|---|
| Design Review | Voting Swarm | Multiple reviewers evaluate independently → Majority decides |
| Migration Plan | Waterfall | Analyze Current → Design Target → Plan Migration → Validate |
| Technology Choice | Competitive Swarm | Multiple agents evaluate different options → Best selected |

---

## 9. Anti-Patterns and Pitfalls

### Waterfall Anti-Patterns

| Anti-Pattern | Problem | Solution |
|---|---|---|
| **Too-Long Waterfall** | Context window exhaustion after 6+ steps | Break into hybrid with swarm phases |
| **Unnecessary Sequencing** | Making independent steps sequential "just in case" | Apply the independence test (Section 5) |
| **Missing Gates** | Not checking intermediate results | Add validation between steps |
| **Context Dumping** | Passing ALL previous output to every step | Only pass what the next step needs |
| **No Loop Back** | Proceeding even when a step fails | Add failure detection and loop-back |

### Swarm Anti-Patterns

| Anti-Pattern | Problem | Solution |
|---|---|---|
| **Missing Consolidation** | Returning raw parallel results to user | Always have a consolidation step |
| **Dependent Agents in Swarm** | Agents that need each other's output | Move to waterfall or hybrid |
| **Too Many Agents** | 10+ parallel agents overwhelm consolidation | Max 5-6 parallel agents per swarm phase |
| **Sequential Dispatch** | Putting Agent calls in separate messages | ALL parallel agents in ONE message |
| **Conflicting Write Access** | Multiple agents editing same files | Use worktree isolation or make agents read-only |

### Hybrid Anti-Patterns

| Anti-Pattern | Problem | Solution |
|---|---|---|
| **Unnecessary Complexity** | Using hybrid when simple waterfall/swarm suffices | Start simple, add complexity only when needed |
| **Swarm Phase Too Large** | 8+ agents in a swarm phase | Split into multiple smaller swarm phases |
| **Missing Phase Boundaries** | Unclear where waterfall ends and swarm begins | Document phase boundaries explicitly |

---

## 10. Template for Defining New Flows

When creating a new Claudine flow (workflow), use this template to determine and document the pattern:

```markdown
# Flow: cne-wfl-{dept}-{name}

## Purpose
[What does this flow accomplish?]

## Input
[What does the flow receive?]

## Output
[What does the flow produce?]

## Dependency Analysis
For each pair of steps, answer: "Can B run without A's output?"

| Step A → Step B | Independent? | Reason |
|---|---|---|
| [Step 1] → [Step 2] | YES/NO | [why] |
| [Step 1] → [Step 3] | YES/NO | [why] |
| [Step 2] → [Step 3] | YES/NO | [why] |

## Pattern Selection
Based on dependency analysis:
- [ ] All independent → SWARM
- [ ] All dependent → WATERFALL
- [ ] Mixed → HYBRID

Selected pattern: [WATERFALL / SWARM / HYBRID]

## Flow Definition

### Phase 1: [Name] ([Waterfall Step / Swarm Phase])
Agent(s): [list agents]
Input: [what this phase receives]
Output: [what this phase produces]
Error handling: [what to do if it fails]

### Phase 2: ...
[continue for each phase]

## Consolidation Strategy (if swarm phases exist)
[How are parallel results merged?]

## Error Recovery
[What happens when things go wrong at each phase?]

## Context Management
[How is context carried forward? What gets summarized? What gets dropped?]
```

---

## Summary

The two patterns are complementary tools:

- **Waterfall** is your **assembly line** -- use it when order matters and each step transforms the output
- **Swarm** is your **war room** -- use it when you need multiple perspectives on the same input simultaneously
- **Hybrid** is your **managed project** -- use it when real work requires both sequential logic and parallel analysis

The Decision Framework (Section 5) is the universal algorithm for choosing between them. Apply it to every new flow before writing the implementation. When in doubt, start with waterfall (simpler, more predictable) and optimize to hybrid/swarm when speed becomes important.
