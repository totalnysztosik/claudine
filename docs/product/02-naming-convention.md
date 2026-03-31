# Claudine Naming Convention (CNE Standard)

## Preamble

Every artifact in Claudine -- every agent, command, skill, rule, hook, script, document -- follows a single, predictable naming convention. This convention is not cosmetic. It is the taxonomy that makes Claudine navigable, debuggable, and self-documenting. When you see a name, you should immediately know: what type of thing is it, which department owns it, and what it does.

---

## Table of Contents

1. [The Prefix: cne](#1-the-prefix-cne)
2. [The Anatomy of a Name](#2-the-anatomy-of-a-name)
3. [Type Codes](#3-type-codes)
4. [Department Codes](#4-department-codes)
5. [Action/Role Naming Rules](#5-actionrole-naming-rules)
6. [Complete Naming Patterns by Primitive](#6-complete-naming-patterns-by-primitive)
7. [Directory Structure](#7-directory-structure)
8. [Examples Registry](#8-examples-registry)
9. [Anti-Patterns](#9-anti-patterns)
10. [Migration Guide](#10-migration-guide)

---

## 1. The Prefix: cne

**Every** Claudine artifact begins with `cne`.

| Question | Answer |
|---|---|
| What does `cne` stand for? | **C**laudi**ne** -- a contraction of the project name |
| Why not `claudine`? | Too long. File names must be scannable. `cne` is 3 characters -- minimal yet unique |
| Why not `cln` or `cdn`? | `cne` preserves the beginning and ending of "Claudine" making it phonetically recognizable |
| Is it case-sensitive? | Always **lowercase**. Never `CNE`, `Cne`, or `CnE` |
| Is it mandatory? | **YES**. No exceptions. Every file, every identifier, every hook name. If it doesn't start with `cne`, it's not part of Claudine |

The prefix serves three purposes:
1. **Namespace isolation**: Claudine artifacts never collide with other plugins, projects, or system files
2. **Instant recognition**: You can `grep cne-` to find every Claudine artifact in any project
3. **Ownership declaration**: The prefix says "this belongs to Claudine, not to the project itself"

---

## 2. The Anatomy of a Name

Every Claudine name follows this pattern:

```
cne-{type}-{dept}-{name}
```

| Segment | Required? | Description | Example |
|---|---|---|---|
| `cne` | ALWAYS | The Claudine prefix | `cne` |
| `-` | ALWAYS | Hyphen separator (never underscore, never dot, never slash) | `-` |
| `{type}` | ALWAYS | 2-3 letter code identifying the primitive type | `agt`, `cmd`, `skl` |
| `-` | ALWAYS | Hyphen separator | `-` |
| `{dept}` | ALWAYS* | 2-3 letter department code | `dev`, `pm`, `sec` |
| `-` | ALWAYS | Hyphen separator | `-` |
| `{name}` | ALWAYS | Descriptive action or role name (kebab-case) | `reviewer`, `fix-flaky`, `audit-deps` |

*Exceptions to the 4-segment pattern:

| Artifact Type | Pattern | Reason |
|---|---|---|
| **Hooks** | `cne-hk-{when}-{action}` | Hooks are event-driven, not department-owned. `{when}` is one of: `pre`, `post`, `stop`. |
| **Rules** | `cne-rul-{topic}-{constraint}` | Rules are cross-cutting; topic codes replace department codes. |
| **CNE tracking files** | `CNE-{PURPOSE}.md` (UPPERCASE) | Project-root tracking files use uppercase to visually distinguish from Claudine source artifacts. These are data files, not definitions. |
| **Memory files** | Free naming | Memory files in `~/.claude/projects/*/memory/` do not require `cne-` prefix as they are system-managed, outside the project repo. |

### Separator Rules
- **ALWAYS use hyphens** (`-`) between segments
- **NEVER use underscores** (`_`) -- this is a hard break from the user's original trials
- **NEVER use dots** (`.`) -- reserved for file extensions
- **NEVER use slashes** (`/`) -- reserved for directory paths
- **Multi-word names use hyphens**: `fix-flaky-test`, not `fixFlakyTest` or `fix_flaky_test`

### Length Guidelines
- **Total name** (without extension): 15-40 characters target, 50 characters maximum
- **Name segment** ({name}): 3-25 characters, prefer shorter
- **Be descriptive but concise**: `cne-agt-dev-reviewer` not `cne-agt-dev-code-quality-reviewer-for-pull-requests`

---

## 3. Type Codes

The type code identifies WHAT kind of Claude Code primitive this artifact is.

| Type Code | Full Name | Primitive | File Location | Example |
|---|---|---|---|---|
| `agt` | Agent | Agent definition | `.claude/agents/` | `cne-agt-dev-reviewer.md` |
| `cmd` | Command | Slash command | `.claude/commands/` | `cne-cmd-pm-plan.md` |
| `skl` | Skill | Skill package | `.claude/skills/` | `cne-skl-dev-typescript/` |
| `rul` | Rule | Always-on rule | `.claude/rules/` | `cne-rul-immutability.md` |
| `hk` | Hook | Automated trigger | `settings.json` (name field) | `cne-hk-post-edit-format` |
| `scr` | Script | Shell/Python script | `scripts/` | `cne-scr-validate-skills.sh` |
| `doc` | Document | Internal document | `docs/` | `cne-doc-architecture.md` |
| `tpl` | Template | Reusable template | `templates/` | `cne-tpl-agent-base.md` |
| `ref` | Reference | Supporting material | `references/` | `cne-ref-style-guide.md` |
| `tst` | Test | Test definition | `tests/` | `cne-tst-skill-trigger.md` |
| `wfl` | Workflow | Multi-step flow def | `workflows/` | `cne-wfl-dev-bugfix.md` |
| `std` | Standard | Quality standard | `standards/` | `cne-std-communication.md` |

### Why Include Type in the Name?

The directory already tells you the type (agents are in `.claude/agents/`). So why duplicate it in the name? Three reasons:

1. **Out-of-context recognition**: When you see `cne-agt-dev-reviewer` in a log, error message, CLAUDE.md reference, or conversation, you immediately know it's an agent -- without knowing its file path
2. **Cross-reference clarity**: When a command says "dispatch `cne-agt-dev-reviewer`", the `agt` prefix makes it unambiguous that we're talking about an agent, not a command or skill with a similar name
3. **Grep safety**: `grep -r "cne-agt-"` finds ALL agents. `grep -r "cne-cmd-"` finds ALL commands. This is invaluable for auditing and self-improvement

---

## 4. Department Codes

Department codes identify WHICH department owns this artifact. They map directly to the organizational structure defined in Document 01.

### Engineering Division

| Code | Department | Description | Examples of artifacts |
|---|---|---|---|
| `dev` | Development | Core software development | `cne-agt-dev-reviewer`, `cne-cmd-dev-fix` |
| `qa` | Quality Assurance | Testing, quality, validation | `cne-agt-qa-tester`, `cne-cmd-qa-audit` |
| `arc` | Architecture | System design, structure | `cne-agt-arc-designer`, `cne-skl-arc-coupling` |
| `dop` | DevOps | CI/CD, infrastructure | `cne-agt-dop-deployer`, `cne-cmd-dop-deploy` |
| `sec` | Security | InfoSec, auditing, compliance | `cne-agt-sec-auditor`, `cne-cmd-sec-scan` |
| `db` | Database | DBA, queries, migrations | `cne-agt-db-optimizer`, `cne-cmd-db-migrate` |

### Product Division

| Code | Department | Description | Examples of artifacts |
|---|---|---|---|
| `pm` | Project Management | Planning, tracking, reporting | `cne-agt-pm-planner`, `cne-cmd-pm-plan` |
| `po` | Product | Product strategy, ownership | `cne-agt-po-strategist`, `cne-cmd-po-discover` |
| `ba` | Business Analysis | Requirements, analysis | `cne-agt-ba-analyst`, `cne-cmd-ba-research` |

### Creative Division

| Code | Department | Description | Examples of artifacts |
|---|---|---|---|
| `ui` | Design | UI/UX, visual design | `cne-agt-ui-designer`, `cne-skl-ui-frontend` |
| `twr` | Technical Writing | Documentation, changelogs | `cne-agt-twr-writer`, `cne-cmd-twr-docs` |
| `seo` | SEO | Search optimization | `cne-agt-seo-auditor`, `cne-cmd-seo-audit` |
| `mkt` | Marketing | Content, campaigns | `cne-agt-mkt-creator`, `cne-cmd-mkt-campaign` |

### Operations Division

| Code | Department | Description | Examples of artifacts |
|---|---|---|---|
| `ops` | Operations | Monitoring, health | `cne-agt-ops-monitor`, `cne-cmd-ops-status` |
| `cld` | Cloud | Cloud infrastructure | `cne-agt-cld-architect`, `cne-skl-cld-aws` |
| `net` | Networking | Network infrastructure | `cne-agt-net-engineer` |

### Governance Division

| Code | Department | Description | Examples of artifacts |
|---|---|---|---|
| `cmp` | Compliance | Regulatory, legal | `cne-agt-cmp-auditor`, `cne-skl-cmp-gdpr` |
| `fin` | Finance | Budget, analysis | `cne-agt-fin-analyst`, `cne-cmd-fin-budget` |

### Meta Division (Claudine Self-Management)

| Code | Department | Description | Examples of artifacts |
|---|---|---|---|
| `core` | Claudine Core | System management, self-improvement | `cne-agt-core-improver`, `cne-cmd-core-adjust` |
| `git` | Git Operations | Version control workflows | `cne-cmd-git-commit`, `cne-cmd-git-pr` |

### Special Codes

| Code | Usage | Example |
|---|---|---|
| `core` | Cross-department or Claudine-internal | `cne-agt-core-orchestrator` |
| `ext` | External stakeholder-facing | `cne-cmd-ext-report` |

### Rules Exception

Rules don't belong to departments -- they apply globally. For rules, the `{dept}` segment is replaced with a **topic code**:

| Topic Code | Description | Example |
|---|---|---|
| `code` | Coding standards | `cne-rul-code-immutability.md` |
| `git` | Git workflow | `cne-rul-git-commit-format.md` |
| `test` | Testing requirements | `cne-rul-test-coverage.md` |
| `sec` | Security policies | `cne-rul-sec-no-hardcoded-secrets.md` |
| `perf` | Performance standards | `cne-rul-perf-context-management.md` |
| `style` | Style and formatting | `cne-rul-style-file-size.md` |
| `flow` | Workflow standards | `cne-rul-flow-gates.md` |
| `self` | Self-improvement rules | `cne-rul-self-improvement.md` |

---

## 5. Action/Role Naming Rules

The `{name}` segment is the most variable part. It must follow these rules:

### For Agents ({name} = role)
Agents are **people**. Name them by their **role**, not their task:
- `reviewer` (person who reviews) -- not `review` (an action)
- `debugger` (person who debugs) -- not `debug` (an action)
- `planner` (person who plans) -- not `plan` (an action)
- `orchestrator` (person who orchestrates) -- not `orchestrate` (an action)

**Compound roles** use hyphens:
- `code-reviewer` (more specific than just `reviewer`)
- `e2e-runner` (specific type of runner)
- `brand-guardian` (specific type of guardian)

### For Commands ({name} = action)
Commands are **actions**. Name them by what they **do**, in imperative form:
- `fix` -- not `fixer` or `fixing`
- `plan` -- not `planner` or `planning`
- `audit` -- not `auditor` or `auditing`
- `deploy` -- not `deployer` or `deployment`

**Compound actions** use hyphens:
- `fix-flaky` (fix flaky tests)
- `create-pr` (create a pull request)
- `audit-deps` (audit dependencies)
- `run-e2e` (run end-to-end tests)

### For Skills ({name} = capability)
Skills are **knowledge domains**. Name them by the **capability** they provide:
- `typescript` -- not `typescript-writer` or `write-typescript`
- `tdd` -- not `test-driven-development-methodology`
- `react` -- not `react-expert`
- `security-review` -- not `security-reviewer`

**Compound capabilities** use hyphens:
- `coupling-analysis`
- `frontend-design`
- `threat-model`

### For Rules ({name} = constraint)
Rules are **constraints**. Name them by what they **enforce**:
- `immutability` -- not `immutable` or `use-immutability`
- `no-hardcoded-secrets` -- not `secrets-policy`
- `commit-format` -- not `how-to-commit`
- `file-size` -- not `keep-files-small`

### For Hooks ({name} = trigger-action)
Hooks are **reactions**. Name them by **when they fire** and **what they do**:
- `post-edit-format` (after edit, format the file)
- `pre-commit-lint` (before commit, run linter)
- `pre-push-test` (before push, run tests)
- `stop-write-notes` (on session end, write notes)

### For Scripts ({name} = task)
Scripts are **utilities**. Name them by what they **accomplish**:
- `validate-skills` (validate all skill files)
- `update-docs` (update documentation)
- `session-catchup` (catch up on session state)

### Banned Words
Never use these in names (they add no information):
- `claudine`, `claude` (already implied by `cne`)
- `new`, `old`, `v2`, `legacy` (use dates or versions in separate metadata)
- `helper`, `util`, `misc` (be specific about what it helps with)
- `my`, `custom` (everything is custom in Claudine)
- `main`, `base`, `default` (everything has a specific purpose)

---

## 6. Complete Naming Patterns by Primitive

### Agents

**Pattern**: `cne-agt-{dept}-{role}.md`
**Location**: `.claude/agents/`

| Name | Department | Role | Purpose |
|---|---|---|---|
| `cne-agt-dev-reviewer.md` | Development | reviewer | Reviews code changes |
| `cne-agt-dev-implementer.md` | Development | implementer | Writes implementation code |
| `cne-agt-dev-debugger.md` | Development | debugger | Diagnoses bugs |
| `cne-agt-dev-fixer.md` | Development | fixer | Fixes identified issues |
| `cne-agt-dev-refactorer.md` | Development | refactorer | Refactors code |
| `cne-agt-qa-tester.md` | Quality | tester | Writes and runs tests |
| `cne-agt-qa-e2e-runner.md` | Quality | e2e-runner | Runs E2E test suites |
| `cne-agt-qa-auditor.md` | Quality | auditor | Audits code quality |
| `cne-agt-arc-designer.md` | Architecture | designer | Designs system architecture |
| `cne-agt-arc-analyst.md` | Architecture | analyst | Analyzes architectural patterns |
| `cne-agt-pm-planner.md` | Project Mgmt | planner | Creates implementation plans |
| `cne-agt-pm-tracker.md` | Project Mgmt | tracker | Tracks project progress |
| `cne-agt-pm-reporter.md` | Project Mgmt | reporter | Generates status reports |
| `cne-agt-sec-auditor.md` | Security | auditor | Audits for vulnerabilities |
| `cne-agt-sec-pentester.md` | Security | pentester | Penetration testing |
| `cne-agt-sec-reviewer.md` | Security | reviewer | Reviews security posture |
| `cne-agt-twr-writer.md` | Tech Writing | writer | Writes documentation |
| `cne-agt-twr-reviewer.md` | Tech Writing | reviewer | Reviews documentation |
| `cne-agt-ui-designer.md` | Design | designer | Creates UI components |
| `cne-agt-ui-ux-guardian.md` | Design | ux-guardian | Guards UX quality |
| `cne-agt-seo-auditor.md` | SEO | auditor | Audits SEO health |
| `cne-agt-seo-optimizer.md` | SEO | optimizer | Optimizes content for SEO |
| `cne-agt-dop-deployer.md` | DevOps | deployer | Manages deployments |
| `cne-agt-dop-infra-checker.md` | DevOps | infra-checker | Checks infrastructure |
| `cne-agt-db-optimizer.md` | Database | optimizer | Optimizes queries/schema |
| `cne-agt-po-strategist.md` | Product | strategist | Product strategy |
| `cne-agt-ba-analyst.md` | Business Analysis | analyst | Requirements analysis |
| `cne-agt-mkt-creator.md` | Marketing | creator | Creates marketing content |
| `cne-agt-cmp-auditor.md` | Compliance | auditor | Compliance auditing |
| `cne-agt-fin-analyst.md` | Finance | analyst | Financial analysis |
| `cne-agt-cld-architect.md` | Cloud | architect | Cloud architecture |
| `cne-agt-core-orchestrator.md` | Core | orchestrator | Orchestrates multi-dept work |
| `cne-agt-core-improver.md` | Core | improver | Self-improvement agent |
| `cne-agt-core-validator.md` | Core | validator | Validates Claudine integrity |

### Commands

**Pattern**: `cne-cmd-{dept}-{action}.md`
**Location**: `.claude/commands/`

| Name | Department | Action | Purpose |
|---|---|---|---|
| `cne-cmd-dev-fix.md` | Development | fix | Fix a bug or issue |
| `cne-cmd-dev-implement.md` | Development | implement | Implement a feature |
| `cne-cmd-dev-refactor.md` | Development | refactor | Refactor code |
| `cne-cmd-dev-repro.md` | Development | repro | Reproduce an issue |
| `cne-cmd-qa-test.md` | Quality | test | Run test suite |
| `cne-cmd-qa-audit.md` | Quality | audit | Run quality audit |
| `cne-cmd-qa-validate.md` | Quality | validate | Validate changes |
| `cne-cmd-arc-optimize.md` | Architecture | optimize | Optimize architecture |
| `cne-cmd-arc-design.md` | Architecture | design | Design a component |
| `cne-cmd-arc-review.md` | Architecture | review | Review architecture |
| `cne-cmd-pm-plan.md` | Project Mgmt | plan | Create implementation plan |
| `cne-cmd-pm-status.md` | Project Mgmt | status | Get project status |
| `cne-cmd-pm-retro.md` | Project Mgmt | retro | Run retrospective |
| `cne-cmd-pm-epic.md` | Project Mgmt | epic | Manage an epic |
| `cne-cmd-pm-sprint.md` | Project Mgmt | sprint | Manage a sprint |
| `cne-cmd-pm-discover.md` | Project Mgmt | discover | Run feature discovery |
| `cne-cmd-sec-scan.md` | Security | scan | Security scan |
| `cne-cmd-sec-audit.md` | Security | audit | Security audit |
| `cne-cmd-twr-docs.md` | Tech Writing | docs | Write documentation |
| `cne-cmd-twr-changelog.md` | Tech Writing | changelog | Generate changelog |
| `cne-cmd-seo-audit.md` | SEO | audit | SEO audit |
| `cne-cmd-seo-optimize.md` | SEO | optimize | Optimize for SEO |
| `cne-cmd-dop-deploy.md` | DevOps | deploy | Deploy application |
| `cne-cmd-dop-check.md` | DevOps | check | Check infrastructure |
| `cne-cmd-dop-release.md` | DevOps | release | Prepare release |
| `cne-cmd-db-migrate.md` | Database | migrate | Run migrations |
| `cne-cmd-db-optimize.md` | Database | optimize | Optimize database |
| `cne-cmd-po-discover.md` | Product | discover | Product discovery |
| `cne-cmd-po-prd.md` | Product | prd | Generate PRD |
| `cne-cmd-ba-research.md` | Business Analysis | research | Research analysis |
| `cne-cmd-ba-analyze.md` | Business Analysis | analyze | Analyze requirements |
| `cne-cmd-mkt-campaign.md` | Marketing | campaign | Create campaign |
| `cne-cmd-mkt-content.md` | Marketing | content | Create content |
| `cne-cmd-git-commit.md` | Git | commit | Smart commit |
| `cne-cmd-git-pr.md` | Git | pr | Create pull request |
| `cne-cmd-git-branch.md` | Git | branch | Manage branches |
| `cne-cmd-git-clean.md` | Git | clean | Clean up branches |
| `cne-cmd-core-adjust.md` | Core | adjust | Adjust Claudine for project |
| `cne-cmd-core-improve.md` | Core | improve | Run self-improvement |
| `cne-cmd-core-audit.md` | Core | audit | Audit Claudine integrity |

### Skills

**Pattern**: `cne-skl-{dept}-{capability}/SKILL.md`
**Location**: `.claude/skills/` (or installed plugin location)

| Directory Name | Department | Capability | Purpose |
|---|---|---|---|
| `cne-skl-dev-typescript/` | Development | typescript | TypeScript patterns |
| `cne-skl-dev-react/` | Development | react | React patterns |
| `cne-skl-dev-python/` | Development | python | Python patterns |
| `cne-skl-dev-golang/` | Development | golang | Go patterns |
| `cne-skl-qa-tdd/` | Quality | tdd | Test-driven development |
| `cne-skl-qa-e2e/` | Quality | e2e | End-to-end testing |
| `cne-skl-arc-design/` | Architecture | design | Architecture design |
| `cne-skl-arc-coupling/` | Architecture | coupling | Coupling analysis |
| `cne-skl-sec-review/` | Security | review | Security review |
| `cne-skl-sec-threat-model/` | Security | threat-model | Threat modeling |
| `cne-skl-pm-planning/` | Project Mgmt | planning | Project planning |
| `cne-skl-ui-frontend/` | Design | frontend | Frontend design |
| `cne-skl-seo-audit/` | SEO | audit | SEO auditing |
| `cne-skl-dop-docker/` | DevOps | docker | Docker patterns |
| `cne-skl-dop-k8s/` | DevOps | k8s | Kubernetes patterns |
| `cne-skl-db-postgres/` | Database | postgres | PostgreSQL patterns |
| `cne-skl-core-brainstorm/` | Core | brainstorm | Brainstorming methodology |
| `cne-skl-core-debug/` | Core | debug | Systematic debugging |

### Rules

**Pattern**: `cne-rul-{topic}-{constraint}.md`
**Location**: `.claude/rules/`

| Name | Topic | Constraint | Purpose |
|---|---|---|---|
| `cne-rul-code-immutability.md` | code | immutability | Enforce immutable patterns |
| `cne-rul-code-file-size.md` | code | file-size | Max 800 lines per file |
| `cne-rul-code-function-size.md` | code | function-size | Max 50 lines per function |
| `cne-rul-git-commit-format.md` | git | commit-format | Conventional commits |
| `cne-rul-git-no-force-push.md` | git | no-force-push | Prevent force push to main |
| `cne-rul-test-coverage.md` | test | coverage | Minimum 80% coverage |
| `cne-rul-test-tdd.md` | test | tdd | Mandatory TDD workflow |
| `cne-rul-sec-no-secrets.md` | sec | no-secrets | No hardcoded secrets |
| `cne-rul-sec-validate-input.md` | sec | validate-input | Always validate user input |
| `cne-rul-perf-model-select.md` | perf | model-select | Model selection guidelines |
| `cne-rul-perf-context.md` | perf | context | Context window management |
| `cne-rul-style-naming.md` | style | naming | This naming convention |
| `cne-rul-flow-gates.md` | flow | gates | Quality gate requirements |
| `cne-rul-self-loop.md` | self | loop | Self-improvement protocol |

### Hooks

**Pattern**: `cne-hk-{when}-{action}`
**Location**: Defined in `settings.json` under `hooks`

| Name | When | Action | Purpose |
|---|---|---|---|
| `cne-hk-pre-edit-validate` | pre-edit | validate | Validate before editing |
| `cne-hk-post-edit-format` | post-edit | format | Auto-format after editing |
| `cne-hk-pre-commit-lint` | pre-commit | lint | Lint before committing |
| `cne-hk-pre-push-test` | pre-push | test | Run tests before pushing |
| `cne-hk-pre-merge-review` | pre-merge | review | Require review before merge |
| `cne-hk-post-write-check` | post-write | check | Check newly written files |
| `cne-hk-stop-write-notes` | stop | write-notes | Write session notes on end |
| `cne-hk-stop-self-improve` | stop | self-improve | Run self-improvement on end |
| `cne-hk-pre-bash-safety` | pre-bash | safety | Block dangerous commands |
| `cne-hk-post-bash-log` | post-bash | log | Log bash command history |

### Scripts

**Pattern**: `cne-scr-{dept}-{task}.{ext}`
**Location**: `scripts/`

| Name | Department | Task | Purpose |
|---|---|---|---|
| `cne-scr-core-validate-skills.py` | Core | validate-skills | Validate all skill files |
| `cne-scr-core-validate-markdown.py` | Core | validate-markdown | Validate markdown formatting |
| `cne-scr-core-update-docs.py` | Core | update-docs | Update documentation |
| `cne-scr-core-session-catchup.py` | Core | session-catchup | Catch up on session state |
| `cne-scr-seo-capture-screenshot.py` | SEO | capture-screenshot | Capture page screenshots |
| `cne-scr-seo-fetch-page.py` | SEO | fetch-page | Fetch page content |
| `cne-scr-seo-analyze-visual.py` | SEO | analyze-visual | Analyze visual elements |

### Workflows

**Pattern**: `cne-wfl-{dept}-{flow-name}.md`
**Location**: `workflows/`

| Name | Department | Flow | Purpose |
|---|---|---|---|
| `cne-wfl-dev-bugfix.md` | Development | bugfix | Complete bug fix workflow |
| `cne-wfl-dev-feature.md` | Development | feature | New feature workflow |
| `cne-wfl-qa-pre-commit.md` | Quality | pre-commit | Pre-commit checks workflow |
| `cne-wfl-qa-pre-deploy.md` | Quality | pre-deploy | Pre-deploy checks workflow |
| `cne-wfl-qa-full-audit.md` | Quality | full-audit | Complete audit workflow |
| `cne-wfl-dop-release.md` | DevOps | release | Release preparation workflow |

### Documents

**Pattern**: `cne-doc-{topic}.md`
**Location**: `docs/`

| Name | Topic | Purpose |
|---|---|---|
| `cne-doc-architecture.md` | architecture | System architecture |
| `cne-doc-philosophy.md` | philosophy | Design philosophy |
| `cne-doc-roadmap.md` | roadmap | Development roadmap |
| `cne-doc-creative-ideas.md` | creative-ideas | Ideas for future |

### Templates

**Pattern**: `cne-tpl-{type}-{purpose}.md`
**Location**: `templates/`

| Name | Type | Purpose |
|---|---|---|
| `cne-tpl-agt-base.md` | agt | Base agent template |
| `cne-tpl-cmd-base.md` | cmd | Base command template |
| `cne-tpl-skl-base.md` | skl | Base skill template |

---

## 7. Directory Structure

The complete Claudine directory tree:

```
project-root/
├── CLAUDE.md                           # Constitution (uses cne- references)
├── .claude/
│   ├── settings.json                   # Hooks, permissions, MCP servers
│   ├── agents/                         # All agent definitions
│   │   ├── cne-agt-dev-reviewer.md
│   │   ├── cne-agt-dev-implementer.md
│   │   ├── cne-agt-dev-debugger.md
│   │   ├── cne-agt-qa-tester.md
│   │   ├── cne-agt-pm-planner.md
│   │   ├── cne-agt-sec-auditor.md
│   │   ├── cne-agt-core-orchestrator.md
│   │   └── ...
│   ├── commands/                       # All slash commands
│   │   ├── cne-cmd-dev-fix.md
│   │   ├── cne-cmd-dev-implement.md
│   │   ├── cne-cmd-pm-plan.md
│   │   ├── cne-cmd-qa-audit.md
│   │   ├── cne-cmd-git-commit.md
│   │   ├── cne-cmd-git-pr.md
│   │   ├── cne-cmd-core-adjust.md
│   │   └── ...
│   ├── rules/                          # Always-on rules
│   │   ├── cne-rul-code-immutability.md
│   │   ├── cne-rul-git-commit-format.md
│   │   ├── cne-rul-test-coverage.md
│   │   ├── cne-rul-sec-no-secrets.md
│   │   ├── cne-rul-style-naming.md
│   │   └── ...
│   └── skills/                         # Skill packages
│       ├── cne-skl-dev-typescript/
│       │   ├── SKILL.md
│       │   └── references/
│       ├── cne-skl-qa-tdd/
│       │   ├── SKILL.md
│       │   └── references/
│       └── ...
├── scripts/                            # Utility scripts
│   ├── cne-scr-core-validate-skills.py
│   ├── cne-scr-core-validate-markdown.py
│   └── ...
├── workflows/                          # Workflow definitions
│   ├── cne-wfl-dev-bugfix.md
│   ├── cne-wfl-dev-feature.md
│   └── ...
├── templates/                          # Reusable templates
│   ├── cne-tpl-agt-base.md
│   ├── cne-tpl-cmd-base.md
│   └── ...
├── standards/                          # Quality standards
│   ├── cne-std-communication.md
│   ├── cne-std-documentation.md
│   └── ...
├── docs/                               # Internal documents
│   ├── cne-doc-architecture.md
│   └── ...
├── tests/                              # Self-tests
│   ├── cne-tst-skill-trigger.md
│   └── ...
└── research/                           # Research & analysis
    ├── cne-ref-source-repos.md
    └── ...
```

---

## 8. Examples Registry

### Full Name Deconstruction Examples

```
cne-agt-dev-reviewer.md
│    │    │    └── role: reviewer (person who reviews)
│    │    └── dept: dev (Development department)
│    └── type: agt (Agent)
└── prefix: cne (Claudine)

cne-cmd-pm-plan.md
│    │    │   └── action: plan (create a plan)
│    │    └── dept: pm (Project Management)
│    └── type: cmd (Command)
└── prefix: cne (Claudine)

cne-skl-qa-tdd/SKILL.md
│    │    │   └── capability: tdd (test-driven development)
│    │    └── dept: qa (Quality Assurance)
│    └── type: skl (Skill)
└── prefix: cne (Claudine)

cne-rul-sec-no-secrets.md
│    │    │    └── constraint: no-secrets (no hardcoded secrets)
│    │    └── topic: sec (Security)
│    └── type: rul (Rule)
└── prefix: cne (Claudine)

cne-hk-post-edit-format
│    │    │      └── action: format (run formatter)
│    │    └── when: post-edit (after Edit tool)
│    └── type: hk (Hook)
└── prefix: cne (Claudine)
```

### Cross-Reference Examples

In CLAUDE.md:
```markdown
## Bug Fix Workflow
When a bug is reported, use `/cne-cmd-dev-fix` which will:
1. Load `cne-skl-core-debug` for systematic debugging methodology
2. Dispatch `cne-agt-dev-debugger` to diagnose the issue
3. Dispatch `cne-agt-dev-fixer` to implement the fix
4. Dispatch `cne-agt-qa-tester` to verify the fix
5. Run `/cne-cmd-git-commit` to commit with proper format
```

In a command (`cne-cmd-dev-fix.md`):
```markdown
# Fix a Bug
Use the Skill tool to load `cne-skl-core-debug`.
Follow the systematic debugging methodology from the skill.
Dispatch `cne-agt-dev-debugger` with the bug details.
Once diagnosed, dispatch `cne-agt-dev-fixer` with the diagnosis.
Finally, dispatch `cne-agt-qa-tester` to verify.
```

---

## 9. Anti-Patterns

| Anti-Pattern | Why It's Wrong | Correct Form |
|---|---|---|
| `claudine-dev_reviewer` | Wrong prefix, underscore separator | `cne-agt-dev-reviewer` |
| `cne_agt_dev_reviewer` | Underscores instead of hyphens | `cne-agt-dev-reviewer` |
| `cne-reviewer` | Missing type and department | `cne-agt-dev-reviewer` |
| `cne-agt-code-reviewer` | `code` is not a department code | `cne-agt-dev-reviewer` |
| `cne-agent-dev-reviewer` | `agent` should be `agt` | `cne-agt-dev-reviewer` |
| `cne-cmd-dev-fixing` | Command actions must be imperative | `cne-cmd-dev-fix` |
| `cne-agt-dev-fix` | Agent roles must be nouns | `cne-agt-dev-fixer` |
| `cne-agt-DEV-reviewer` | Department codes must be lowercase | `cne-agt-dev-reviewer` |
| `cne-agt-development-reviewer` | Department codes must be abbreviated | `cne-agt-dev-reviewer` |
| `cne-skl-typescript` | Missing department | `cne-skl-dev-typescript` |
| `cne-rul-always-use-immutability` | Rule names should be concise | `cne-rul-code-immutability` |
| `my-custom-agent.md` | Missing `cne` prefix entirely | `cne-agt-{dept}-{role}.md` |

---

## 10. Migration Guide

### From v2.md Naming to CNE Standard

| v2.md Name | CNE Standard Name |
|---|---|
| `claudine-dev_fix-flaky-test` | `cne-cmd-qa-fix-flaky` |
| `claudine-dev_repro-issue` | `cne-cmd-dev-repro` |
| `claudine-arch_optimize` | `cne-cmd-arc-optimize` |
| `claudine-pm_complete-epic` | `cne-cmd-pm-epic-complete` |
| `claudine-pm_complete-sprint` | `cne-cmd-pm-sprint-complete` |
| `claudine-pm_create-epic-plan` | `cne-cmd-pm-epic-plan` |
| `claudine-pm_create-implementation-plan` | `cne-cmd-pm-plan` |
| `claudine-dev_execute-ticket` | `cne-cmd-dev-execute` |
| `claudine-pm_common-ground` | `cne-cmd-core-common-ground` |
| `claudine-qa_complete-ticket` | `cne-cmd-qa-complete` |
| `claudine-arch_feature-discovery` | `cne-cmd-pm-discover` |
| `claudine-dev_add-analytics-event` | `cne-skl-dev-analytics` |
| `claudine-dev_clojure-review` | `cne-skl-dev-clojure-review` |
| `claudine-dev_ts-write` | `cne-skl-dev-typescript` |
| `claudine-qa_cypress-e2e-create` | `cne-skl-qa-cypress-e2e` |
| `claudine-rules_locality` | `cne-rul-flow-locality` |
| `claudine-rules_basics` | `cne-rul-flow-basics` |
| `claudine-rules_repo-autonomy` | `cne-rul-flow-repo-autonomy` |
| `claudine-rules_coding-principles` | `cne-rul-code-principles` |
| `claudine-rules_git-workflow` | `cne-rul-git-workflow` |

---

## Validation

A Claudine name is valid if and only if:

1. It starts with `cne-`
2. The second segment is a valid type code from Section 3
3. The third segment is a valid department code from Section 4 (or topic code for rules)
4. The fourth segment follows the naming rules from Section 5
5. All segments are lowercase
6. All separators are hyphens
7. Total length (without extension) is <= 50 characters
8. No banned words from Section 5 are used

### Validation Regex

For standard artifacts (agents, commands, skills, scripts, docs, templates, workflows, standards, tests, references):
```regex
^cne-(agt|cmd|skl|scr|doc|tpl|ref|tst|wfl|std)-[a-z]{2,4}-[a-z][a-z0-9-]{1,24}$
```

For hooks:
```regex
^cne-hk-(pre|post|stop)-[a-z][a-z0-9-]{1,20}$
```

For rules:
```regex
^cne-rul-[a-z]{2,5}-[a-z][a-z0-9-]{1,24}$
```

For CNE tracking files (uppercase):
```regex
^CNE-[A-Z][A-Z-]{2,20}\.md$
```

### Workflows vs. Commands Disambiguation

A **workflow** (`cne-wfl-*`) is a documentation artifact that describes a multi-step flow, its pattern (waterfall/swarm/hybrid), and all phases. A **command** (`cne-cmd-*`) is the user-facing trigger that executes that workflow. The relationship is:

```
cne-wfl-dev-bugfix.md  → defines the flow (phases, agents, patterns, error handling)
cne-cmd-dev-fix.md     → triggers the flow (user-facing, loads the workflow)
```

Commands reference workflows. Workflows don't reference commands. A workflow may exist without a command (internal orchestration), but a command should always reference its workflow.

The `cne-scr-core-validate-skills.py` script (and future `cne-scr-core-validate-names.py`) will enforce this convention programmatically.
