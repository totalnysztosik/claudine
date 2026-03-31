# Claudine Iterative Self-Improvement System

## Core Principle

Claudine must get better every time it is used. Every session is both a work session AND a training session. The system captures what worked, what failed, what was missing, and feeds those learnings back into itself. This document defines the complete lifecycle of how Claudine improves itself -- what files exist, how they're generated, when, and how they're referenced.

---

## Table of Contents

1. [The Improvement Loop](#1-the-improvement-loop)
2. [The Files That Drive Improvement](#2-the-files-that-drive-improvement)
3. [Session Lifecycle: Before, During, After](#3-session-lifecycle-before-during-after)
4. [The Feedback Capture System](#4-the-feedback-capture-system)
5. [The Self-Audit System](#5-the-self-audit-system)
6. [The Evolution Pipeline](#6-the-evolution-pipeline)
7. [Version Control for Claudine Itself](#7-version-control-for-claudine-itself)
8. [The /cne-cmd-core-improve Command](#8-the-cne-cmd-core-improve-command)
9. [The /cne-cmd-core-adjust Command](#9-the-cne-cmd-core-adjust-command)
10. [Metrics and Measurement](#10-metrics-and-measurement)
11. [Safeguards Against Regression](#11-safeguards-against-regression)

---

## 1. The Improvement Loop

Every Claudine session follows a closed loop:

```
┌──────────────────────────────────────────────────┐
│                                                    │
│   ┌───────────┐    ┌───────────┐    ┌──────────┐  │
│   │  SESSION   │    │  CAPTURE  │    │  APPLY   │  │
│   │  (work)    │───▶│ (learn)   │───▶│ (evolve) │  │
│   └───────────┘    └───────────┘    └──────────┘  │
│        ▲                                    │      │
│        │                                    │      │
│        └────────────────────────────────────┘      │
│                   NEXT SESSION                     │
└──────────────────────────────────────────────────┘
```

### Phase 1: SESSION (Do the work)
- Load Claudine's current state (CLAUDE.md, rules, agents, skills)
- Execute user's requests using the organizational structure
- Track what happens via tasks and execution logs

### Phase 2: CAPTURE (Record what we learned)
- At session end (Stop hook + /cne-cmd-core-improve)
- Record what worked well (feedback memories)
- Record what failed or was missing (feedback memories)
- Record skill invocations and outcomes (execution log)
- Record user corrections and preferences (feedback memories)

### Phase 3: APPLY (Make Claudine better)
- At session start or via /cne-cmd-core-improve
- Read captured feedback and execution logs
- Identify patterns in failures/successes
- Update agents, commands, skills, rules as needed
- Validate changes don't break existing functionality

---

## 2. The Files That Drive Improvement

### File Inventory

| File | Location | Purpose | Generated When | Referenced When |
|---|---|---|---|---|
| `CNE-SESSION-LOG.md` | Project root | Session execution log | During/after each session | Start of next session, improvement runs |
| `CNE-FEEDBACK.md` | Project root | Accumulated feedback | End of each session | Improvement runs |
| `CNE-METRICS.md` | Project root | Performance metrics | End of each session | Improvement runs, audits |
| `CNE-CHANGELOG.md` | Project root | Claudine version history | After improvements applied | Audit, troubleshooting |
| `CNE-KNOWN-ISSUES.md` | Project root | Known bugs/limitations | When issues discovered | Before each session, during improvement |
| `memory/*.md` | `.claude/projects/*/memory/` | Cross-session memory | During sessions | Start of each session |
| `CLAUDE.md` | Project root | Constitution (evolves) | Initial setup + improvements | Every session (always loaded) |

### File Relationships

```
CNE-SESSION-LOG.md ──┐
                     ├──▶ /cne-cmd-core-improve ──▶ Updated agents/commands/skills/rules
CNE-FEEDBACK.md ─────┤                                       │
                     │                                       ▼
CNE-METRICS.md ──────┤                              CNE-CHANGELOG.md (record changes)
                     │
CNE-KNOWN-ISSUES.md ─┘
```

### File Size Management

CNE tracking files are append-based and will grow over time. To prevent unbounded growth:

| File | Rotation Strategy | Max Size |
|---|---|---|
| `CNE-SESSION-LOG.md` | Keep last 20 sessions. Archive older entries to `archive/CNE-SESSION-LOG-{YYYY-MM}.md` | ~2000 lines |
| `CNE-FEEDBACK.md` | No rotation (curated content). Prune resolved/outdated patterns quarterly | ~500 lines |
| `CNE-METRICS.md` | Keep last 30 sessions of data. Archive older data to `archive/` | ~1000 lines |
| `CNE-CHANGELOG.md` | No rotation (historical record). This is Claudine's version history | Unlimited |
| `CNE-KNOWN-ISSUES.md` | Move resolved issues to "Resolved" section. Archive after 30 days resolved | ~200 lines |

The `/cne-cmd-core-improve` command includes a maintenance phase that handles archival when files exceed their size targets.

### Detailed File Specifications

#### CNE-SESSION-LOG.md

**Purpose**: Raw execution data from each session. What was attempted, what succeeded, what failed.

**Format**:
```markdown
# Claudine Session Log

## Session: 2026-03-31 14:30

### Commands Invoked
| Command | Arguments | Outcome | Duration |
|---|---|---|---|
| /cne-cmd-dev-fix | "auth token expiry" | SUCCESS | 12 min |
| /cne-cmd-qa-audit | - | PARTIAL (sec agent failed) | 8 min |

### Agents Dispatched
| Agent | Invoked By | Pattern | Outcome | Notes |
|---|---|---|---|---|
| cne-agt-dev-debugger | cne-cmd-dev-fix | Waterfall Step 1 | SUCCESS | Found root cause |
| cne-agt-dev-fixer | cne-cmd-dev-fix | Waterfall Step 2 | SUCCESS | Fixed in 1 attempt |
| cne-agt-qa-tester | cne-cmd-dev-fix | Waterfall Step 3 | SUCCESS | 4/4 tests pass |
| cne-agt-sec-auditor | cne-cmd-qa-audit | Swarm | TIMEOUT | Agent exceeded context |

### Skills Loaded
| Skill | Triggered By | Relevant? | Notes |
|---|---|---|---|
| cne-skl-core-debug | cne-cmd-dev-fix | YES | Guided effective diagnosis |
| cne-skl-dev-typescript | auto-trigger | YES | Needed for TS fix |

### User Corrections
| What User Said | What Claude Was Doing | Correction Applied |
|---|---|---|
| "Don't mock the database" | Writing mock-based tests | Switched to integration tests |

### Issues Encountered
| Issue | Severity | Resolved? | How |
|---|---|---|---|
| sec-auditor timeout | MEDIUM | NO | Skipped, needs investigation |
| Context window pressure at step 3 | LOW | YES | Summarized before step 3 |

---
[Previous sessions below, newest first]
```

**Generated**: During the session (appended in real-time) and at session end (final summary)
**Referenced**: At the start of the next session, during /cne-cmd-core-improve

#### CNE-FEEDBACK.md

**Purpose**: Accumulated, actionable feedback from all sessions. Distilled from session logs into patterns.

**Format**:
```markdown
# Claudine Feedback Registry

## Confirmed Patterns (things that reliably work)
- Waterfall pattern for bug fixes works well (confirmed 5/5 sessions)
- cne-agt-dev-debugger produces accurate diagnoses (confirmed 4/5)
- Auto-triggering cne-skl-core-debug for fix commands adds value

## Known Failure Patterns (things that reliably fail)
- cne-agt-sec-auditor times out on large codebases (3/5 sessions)
  → Action: Split into smaller focused security agents
- Swarm consolidation loses nuance when >4 agents (2/3 sessions)
  → Action: Add dedicated consolidator agent for large swarms

## User Preferences (learned from corrections)
- Prefer integration tests over mocks for database operations
- Want concise output, not verbose explanations
- Prefer waterfall for fixes, swarm for reviews

## Missing Capabilities (things users wanted but Claudine couldn't do)
- No agent for API documentation generation
- No command for database migration planning
- No skill for GraphQL schema design

## Improvement Suggestions (generated by self-audit)
- cne-agt-dev-reviewer should include performance analysis
- cne-cmd-pm-plan should auto-load cne-skl-pm-planning
- New rule needed: cne-rul-test-no-mocks-for-db
```

**Generated**: End of each session (by Stop hook or /cne-cmd-core-improve)
**Referenced**: During /cne-cmd-core-improve

#### CNE-METRICS.md

**Purpose**: Quantitative performance data for tracking improvement over time.

**Format**:
```markdown
# Claudine Performance Metrics

## Agent Success Rates (last 10 sessions)
| Agent | Invocations | Success | Failure | Timeout | Rate |
|---|---|---|---|---|---|
| cne-agt-dev-debugger | 8 | 7 | 1 | 0 | 87.5% |
| cne-agt-dev-fixer | 6 | 5 | 1 | 0 | 83.3% |
| cne-agt-sec-auditor | 5 | 2 | 0 | 3 | 40.0% |

## Command Completion Rates
| Command | Invocations | Full Success | Partial | Failure |
|---|---|---|---|---|
| /cne-cmd-dev-fix | 5 | 4 | 1 | 0 |
| /cne-cmd-qa-audit | 3 | 1 | 2 | 0 |

## Skill Relevance
| Skill | Loaded | Relevant | Irrelevant | Rate |
|---|---|---|---|---|
| cne-skl-core-debug | 5 | 5 | 0 | 100% |
| cne-skl-dev-typescript | 8 | 6 | 2 | 75% |

## Pattern Usage
| Pattern | Used | Succeeded | Failed |
|---|---|---|---|
| Waterfall | 12 | 10 | 2 |
| Swarm | 6 | 4 | 2 |
| Hybrid | 3 | 3 | 0 |

## Session Duration Average: 25 min
## Context Compaction Events: 1.2 per session average
## User Corrections per Session: 0.8 average (trending down)
```

**Generated**: End of each session (appended)
**Referenced**: During /cne-cmd-core-improve, during audits

#### CNE-CHANGELOG.md

**Purpose**: Version history of Claudine changes. Track what was modified, why, and what the impact was.

**Format**:
```markdown
# Claudine Changelog

## [2026-03-31] Improvement Run #14
### Changes
- UPDATED: cne-agt-sec-auditor -- reduced scope to prevent timeouts
- ADDED: cne-agt-sec-dep-scanner -- new agent for dependency scanning
- ADDED: cne-rul-test-no-mock-db -- rule against mocking database
- UPDATED: cne-cmd-qa-audit -- split security into 3 parallel agents

### Reason
sec-auditor was timing out on large codebases (3/5 sessions).
User feedback: prefer integration tests over mocks.

### Impact
Security audit success rate: 40% → 85% (next 5 sessions)

---
[Previous entries below]
```

**Generated**: After /cne-cmd-core-improve applies changes
**Referenced**: During audits, troubleshooting, rollback decisions

#### CNE-KNOWN-ISSUES.md

**Purpose**: Living document of known bugs, limitations, and workarounds.

**Format**:
```markdown
# Claudine Known Issues

## Active Issues

### CNE-001: sec-auditor timeout on large codebases
- Severity: MEDIUM
- First seen: 2026-03-15
- Status: MITIGATED (split into focused agents)
- Workaround: Use /cne-cmd-sec-scan instead of full audit

### CNE-002: Swarm consolidation loses nuance >4 agents
- Severity: LOW
- First seen: 2026-03-20
- Status: OPEN
- Workaround: Limit swarm to 4 agents, use staged swarm for more

## Resolved Issues
[moved from Active when fixed]
```

**Generated**: When issues are discovered during sessions
**Referenced**: Before each session, during improvement runs

---

## 3. Session Lifecycle: Before, During, After

### BEFORE Session (Instructed via CLAUDE.md)

**Important**: Claude Code does NOT have a `SessionStart` hook type. The "before session" catchup is implemented by embedding a startup protocol directly in CLAUDE.md:

```markdown
# In CLAUDE.md, add this section:
## Session Startup Protocol
Before responding to the user's first request in every session:
1. Read CNE-SESSION-LOG.md (last 3 session entries)
2. Read CNE-KNOWN-ISSUES.md (Active Issues section only)
3. Read CNE-FEEDBACK.md (last 5 entries)
4. Incorporate context silently -- do not output a summary unless the user asks
```

The actual startup sequence:

```
Session starts
├── CLAUDE.md loaded (constitution -- contains startup protocol)
├── Rules loaded (policies)
├── Hooks registered (quality gates -- PreToolUse, PostToolUse, Stop only)
├── Memory index loaded (MEMORY.md)
│
├── FIRST USER REQUEST arrives
│   └── CLAUDE.md startup protocol triggers:
│       ├── Reads CNE-SESSION-LOG.md (last 3 sessions)
│       ├── Reads CNE-KNOWN-ISSUES.md (active issues)
│       ├── Reads CNE-FEEDBACK.md (last 5 entries)
│       └── Context incorporated silently
└── Claude responds with full institutional memory
```

Alternative: User can run `/cne-cmd-core-catchup` manually as their first action.

**Key Files Referenced at Start**:
1. `CLAUDE.md` -- always (system loads it, contains startup protocol)
2. `.claude/rules/*` -- always (system loads them)
3. `CNE-SESSION-LOG.md` -- via CLAUDE.md startup protocol (last 3 sessions)
4. `CNE-KNOWN-ISSUES.md` -- via CLAUDE.md startup protocol (active issues)
5. `CNE-FEEDBACK.md` -- via CLAUDE.md startup protocol (recent entries)
6. `memory/MEMORY.md` -- always (system loads index)

### DURING Session (Continuous)

```
For each user request:
├── Task created (tracking)
├── Command/agent/skill invoked (work)
├── Session log updated in real-time:
│   ├── Command invoked, arguments, outcome
│   ├── Agents dispatched, pattern used, results
│   ├── Skills loaded, relevance
│   ├── User corrections (CRITICAL for feedback)
│   └── Issues encountered
├── Memory updated if new learnings
└── Task updated (progress)
```

**Key Behavior**: The main session (or a dedicated logging mechanism) MUST track:
- What command was invoked
- Which agents were dispatched and their outcomes
- What the user corrected or was unhappy with
- Any errors, timeouts, or unexpected behaviors

### AFTER Session (Stop Hook + Optional /cne-cmd-core-improve)

```
Session ending
├── Stop hook fires (cne-hk-stop-capture):
│   ├── Finalize CNE-SESSION-LOG.md entry
│   ├── Extract user corrections → append to CNE-FEEDBACK.md
│   ├── Update CNE-METRICS.md with session stats
│   ├── Check for new known issues → append to CNE-KNOWN-ISSUES.md
│   └── Write relevant memory entries
│
├── Optional: /cne-cmd-core-improve runs:
│   ├── Analyze recent feedback patterns
│   ├── Identify improvement opportunities
│   ├── Propose changes to agents/commands/skills/rules
│   ├── Apply approved changes
│   └── Record changes in CNE-CHANGELOG.md
│
└── Session context discarded (but files persist)
```

---

## 4. The Feedback Capture System

### Automatic Capture (Always-On)

The following are captured automatically:

| What | How | Stored In |
|---|---|---|
| Command outcomes | Stop hook evaluates task list | CNE-SESSION-LOG.md |
| Agent success/failure | Agent return messages parsed | CNE-SESSION-LOG.md |
| Skill relevance | Was loaded skill actually used? | CNE-SESSION-LOG.md |
| Errors and timeouts | Exception/timeout detection | CNE-SESSION-LOG.md |

### User-Triggered Capture

The following require the user's participation:

| What | How | Stored In |
|---|---|---|
| "That was wrong" | User corrects Claude → feedback memory | CNE-FEEDBACK.md + memory/ |
| "That was great" | User praises approach → feedback memory | CNE-FEEDBACK.md + memory/ |
| "I wish you could..." | User requests missing capability | CNE-FEEDBACK.md |
| "Don't do that" | User blocks behavior → feedback memory | CNE-FEEDBACK.md + memory/ |

### Distillation: From Raw to Actionable

Raw session log entries are distilled into actionable feedback:

```
RAW (session log):
"cne-agt-sec-auditor: TIMEOUT after 120s. Codebase: 450 files."

DISTILLED (feedback):
"cne-agt-sec-auditor times out on codebases >300 files.
Action: Split into focused sub-scanners or add file-count pre-check."

APPLIED (improvement):
- Created cne-agt-sec-dep-scanner (dependency-focused)
- Created cne-agt-sec-code-scanner (code-focused)
- Updated cne-cmd-sec-scan to dispatch both in swarm
```

---

## 5. The Self-Audit System

### What It Audits

The self-audit (triggered by `/cne-cmd-core-audit`) checks:

| Audit Area | What It Checks | Tools Used |
|---|---|---|
| **Naming compliance** | All files follow CNE naming convention | cne-scr-core-validate-names.py |
| **File integrity** | All referenced files exist, no orphans | cne-scr-core-validate-refs.py |
| **Skill quality** | Skills have proper SKILL.md, frontmatter, examples | cne-scr-core-validate-skills.py |
| **Agent quality** | Agents have identity, instructions, output format | cne-scr-core-validate-agents.py |
| **Command quality** | Commands have description, $ARGUMENTS, agent references | cne-scr-core-validate-commands.py |
| **Rule consistency** | Rules don't contradict each other | cne-agt-core-validator |
| **Metric trends** | Success rates, completion rates, user corrections | cne-agt-core-analyst |
| **Coverage** | All departments have agents, commands, skills | cne-agt-core-coverage-checker |
| **CLAUDE.md sync** | CLAUDE.md references match actual files | cne-scr-core-validate-refs.py |

### Audit Output

The audit produces a structured report:

```markdown
# Claudine Self-Audit Report
Date: 2026-03-31

## Naming Compliance: 95%
- 2 files don't follow convention: [list]

## File Integrity: 100%
- All referenced files exist
- 1 orphan file found: [file]

## Skill Quality: 88%
- 3 skills missing examples section

## Agent Performance: 82%
- 2 agents below 70% success rate: [list]

## Missing Coverage:
- No agent for: API documentation
- No command for: database migration
- No skill for: GraphQL

## Recommendations:
1. Rename non-compliant files
2. Add examples to 3 skills
3. Investigate low-performance agents
4. Create missing agents/commands/skills
```

---

## 6. The Evolution Pipeline

### How Changes Get Made

```
Feedback/Audit identifies opportunity
  │
  ▼
Proposal generated (what to change, why)
  │
  ▼
Impact assessment (what does this affect?)
  │
  ▼
Change implemented (file created/updated)
  │
  ▼
Validation (does it break anything?)
  │
  ▼
Changelog updated (what changed, why)
  │
  ▼
Metrics baseline reset (track impact)
```

### Change Types

| Type | Trigger | Approval |
|---|---|---|
| **Bug fix** | Agent/command fails consistently | Auto-approved |
| **Enhancement** | Feedback suggests improvement | Auto-approved if low-risk |
| **New capability** | Missing coverage identified | Requires user confirmation |
| **Breaking change** | Renaming, restructuring | Requires user confirmation |
| **Rule change** | Feedback contradicts existing rule | Requires user confirmation |

### The Improvement Agent (cne-agt-core-improver)

This is the most important agent in Claudine's meta division. It:
1. Reads CNE-SESSION-LOG.md, CNE-FEEDBACK.md, CNE-METRICS.md
2. Identifies patterns (recurring failures, missing capabilities, user preferences)
3. Proposes specific changes (new agent, updated command, new rule)
4. Implements changes (creates/updates files)
5. Validates changes (runs self-audit)
6. Records changes (updates CNE-CHANGELOG.md)

---

## 7. Version Control for Claudine Itself

### Claudine's Own Git History

Claudine files live in the project repo. Changes to Claudine artifacts should be committed separately from project changes:

```
git commit -m "feat(cne): add sec-dep-scanner agent for focused dependency scanning

Split security auditor into focused sub-scanners to prevent timeout
on large codebases. cne-agt-sec-auditor was timing out in 60% of
sessions on codebases >300 files."
```

### Commit Message Convention for Claudine Changes

```
{type}(cne): {description}

Types:
- feat(cne): New agent, command, skill, or rule
- fix(cne): Fix to existing Claudine artifact
- refactor(cne): Restructure without behavior change
- docs(cne): Documentation update
- perf(cne): Performance improvement
- test(cne): Self-test additions
```

### Branching Strategy

For major Claudine improvements:
1. Create branch: `cne/improvement-{description}`
2. Implement changes on branch
3. Run self-audit
4. Merge when validated

For minor fixes:
1. Commit directly to working branch
2. Prefix commit with `fix(cne):`

---

## 8. The /cne-cmd-core-improve Command

### Purpose
The primary mechanism for triggering Claudine self-improvement. Can be run:
- Manually by user (`/cne-cmd-core-improve`)
- Automatically via Stop hook (lighter version)
- On a schedule via cron trigger

### Procedure

```markdown
# /cne-cmd-core-improve

## Phase 1: Gather Intelligence (Waterfall)
1. Read CNE-SESSION-LOG.md (last 5 sessions)
2. Read CNE-FEEDBACK.md (all entries)
3. Read CNE-METRICS.md (current stats)
4. Read CNE-KNOWN-ISSUES.md (active issues)

## Phase 2: Analyze (Swarm)
Launch in parallel:
1. cne-agt-core-improver: "Identify improvement opportunities from session data"
2. cne-agt-core-validator: "Run self-audit, find inconsistencies"
3. cne-agt-core-coverage-checker: "Find missing agents/commands/skills"

## Phase 3: Prioritize (Waterfall)
Consolidate Phase 2 results.
Rank improvements by:
1. Impact (how many sessions affected)
2. Effort (how complex is the change)
3. Risk (could it break existing functionality)

## Phase 4: Implement (Waterfall, with user approval for high-risk)
For each approved improvement:
1. Create/update the relevant file(s)
2. Update CLAUDE.md references if needed
3. Run validation scripts
4. Update CNE-CHANGELOG.md

## Phase 5: Validate (Swarm)
Run all validation scripts in parallel:
1. cne-scr-core-validate-names.py
2. cne-scr-core-validate-refs.py
3. cne-scr-core-validate-skills.py
4. cne-scr-core-validate-agents.py
```

---

## 9. The /cne-cmd-core-adjust Command

### Purpose
Adapt Claudine to a NEW project. When you fork Claudine into a new codebase, this command rewrites relevant artifacts to match the project's technology stack, coding style, and workflow.

### Procedure

```markdown
# /cne-cmd-core-adjust

## Phase 1: Scan Project
1. Read project's package.json/build.gradle/Cargo.toml/etc.
2. Detect technology stack
3. Read existing documentation
4. Identify project structure

## Phase 2: Configure
Based on detected stack:
1. Enable relevant technology-specific skills (e.g., cne-skl-dev-typescript)
2. Disable irrelevant skills (e.g., cne-skl-dev-python if no Python)
3. Update CLAUDE.md with project-specific context
4. Adjust commands for project's build/test/deploy tools
5. Configure hooks for project's linting/formatting tools

## Phase 3: Validate
1. Run self-audit
2. Verify all referenced files exist
3. Test a simple command execution
```

---

## 10. Metrics and Measurement

### Key Performance Indicators (KPIs)

| KPI | Target | How Measured |
|---|---|---|
| Agent success rate | >80% | CNE-METRICS.md |
| Command completion rate | >85% | CNE-METRICS.md |
| Skill relevance rate | >70% | CNE-METRICS.md |
| User corrections per session | <1.0 | CNE-METRICS.md |
| Context compaction events | <2 per session | CNE-METRICS.md |
| Known issues resolved | Decreasing trend | CNE-KNOWN-ISSUES.md |
| Self-audit score | >90% | Self-audit report |

### Trend Tracking

Each metric should show:
- Current value
- 5-session moving average
- Trend direction (improving/stable/declining)
- Target comparison

---

## 11. Safeguards Against Regression

### The Golden Rule
**Never auto-apply changes that modify rules or delete capabilities.** These require user confirmation.

### Validation Gates

Before any improvement is applied:
1. **Naming validation**: Does the new/updated file follow CNE convention?
2. **Reference validation**: Do all cross-references resolve?
3. **Consistency check**: Do updated rules contradict existing rules?
4. **Rollback plan**: Can this change be easily reverted?

### Rollback Procedure

If an improvement causes problems:
1. Check CNE-CHANGELOG.md for the change
2. Revert the specific file(s) via git
3. Update CNE-KNOWN-ISSUES.md with the failure
4. Update CNE-FEEDBACK.md with the lesson learned

### The Ratchet Principle

Improvements should be **monotonically increasing** in quality:
- Never remove a working capability to add a new one
- Never weaken a rule to fix a convenience issue
- Never reduce test coverage to speed up a workflow
- If something was confirmed working, don't change it unless there's a concrete problem

---

## Summary

Claudine improves itself through a closed loop:
1. **WORK** -- execute user's requests, track everything
2. **CAPTURE** -- record outcomes, corrections, and insights
3. **ANALYZE** -- identify patterns in successes and failures
4. **PROPOSE** -- generate specific improvement opportunities
5. **IMPLEMENT** -- make targeted changes with validation
6. **MEASURE** -- track KPIs to confirm improvement

The files that drive this:
- `CNE-SESSION-LOG.md` -- raw execution data
- `CNE-FEEDBACK.md` -- distilled actionable feedback
- `CNE-METRICS.md` -- quantitative performance tracking
- `CNE-CHANGELOG.md` -- version history of improvements
- `CNE-KNOWN-ISSUES.md` -- living bug/limitation tracker
- `memory/*.md` -- cross-session institutional knowledge

The principle: every session leaves Claudine at least as good as it found it, and usually better.
