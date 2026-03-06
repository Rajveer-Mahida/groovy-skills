---
name: groovy-skills
description: A suite of specialized agents for planning, executing, debugging, and verifying software projects in structured phases. Use these agents when working on project roadmaps, phase-based execution, debugging, code analysis, or post-execution verification.
---

# Groovy Skills

A collection of agents that power the `/groovy:*` command workflow — from project research and roadmap creation through phase planning, execution, verification, and debugging.

## Agent Index

### groovy-codebase-mapper
**File:** `references/groovy-codebase-mapper.md`

Explores a codebase for a specific focus area and writes structured analysis documents to `.planning/codebase/`. Spawned by `/groovy:map-codebase`.

**Focus areas:**
- `tech` → writes `STACK.md`, `INTEGRATIONS.md`
- `arch` → writes `ARCHITECTURE.md`, `STRUCTURE.md`
- `quality` → writes `CONVENTIONS.md`, `TESTING.md`
- `concerns` → writes `CONCERNS.md`

**Use when:** You need a structured analysis of an existing codebase before planning or executing changes.

---

### groovy-debugger
**File:** `references/groovy-debugger.md`

Investigates bugs using scientific method (hypothesis testing, evidence gathering, falsifiability). Manages persistent debug sessions via `.planning/debug/` files that survive context resets. Spawned by `/groovy:debug`.

**Modes:**
- Default: Interactive — gathers symptoms, investigates, fixes, verifies
- `goal: find_root_cause_only` — diagnose only, no fix
- `goal: find_and_fix` — full cycle with human verification checkpoint
- `symptoms_prefilled: true` — skip symptom gathering

**Use when:** A bug needs systematic investigation, especially complex or intermittent issues.

---

### groovy-executor
**File:** `references/groovy-executor.md`

Executes `PLAN.md` files atomically — one commit per task, handles deviations automatically, pauses at checkpoints, creates `SUMMARY.md` and updates `STATE.md`. Spawned by `/groovy:execute-phase`.

**Deviation rules:**
- Rule 1: Auto-fix bugs
- Rule 2: Auto-add missing critical functionality (security, correctness)
- Rule 3: Auto-fix blocking issues
- Rule 4: Ask before architectural changes

**Use when:** A phase plan is ready and needs to be implemented.

---

### groovy-integration-checker
**File:** `references/groovy-integration-checker.md`

Verifies cross-phase integration and E2E flows. Checks that exports are imported, APIs have consumers, auth is applied, and user workflows complete end-to-end. Spawned by the milestone auditor.

**Checks:**
- Export/import wiring between phases
- API route coverage (routes have callers)
- Auth protection on sensitive areas
- E2E flow completeness (form → API → DB → display)
- Requirements integration map

**Use when:** Multiple phases are complete and you need to verify the system works as a whole.

---

### groovy-phase-researcher
**File:** `references/groovy-phase-researcher.md`

Researches how to implement a specific phase before planning begins. Produces `RESEARCH.md` that the planner consumes. Uses Context7, official docs, and WebSearch with verified confidence levels. Spawned by `/groovy:plan-phase`.

**Output sections:** Standard Stack, Architecture Patterns, Don't Hand-Roll, Common Pitfalls, Code Examples, Validation Architecture (if nyquist enabled).

**Use when:** Starting to plan a new phase and technical research is needed first.

---

### groovy-plan-checker
**File:** `references/groovy-plan-checker.md`

Verifies that plans **will** achieve the phase goal before execution burns context. Goal-backward analysis across 8 dimensions. Spawned by `/groovy:plan-phase` after the planner creates plans.

**Verification dimensions:**
1. Requirement Coverage — every req has tasks
2. Task Completeness — files, action, verify, done all present
3. Dependency Correctness — no cycles, valid references
4. Key Links Planned — artifacts are wired, not just created
5. Scope Sanity — 2-3 tasks/plan, within context budget
6. Verification Derivation — must_haves trace to phase goal
7. Context Compliance — plans honor CONTEXT.md decisions
8. Nyquist Compliance — automated verify commands present (if enabled)

**Use when:** Plans are written and need quality assurance before execution.

---

### groovy-planner
**File:** `references/groovy-planner.md`

Creates executable `PLAN.md` files with task breakdown, dependency analysis, and goal-backward must_haves. Decomposes phases into parallel-optimized plans (2-3 tasks each) with wave assignments. Spawned by `/groovy:plan-phase`.

**Outputs:** One or more `XX-NN-PLAN.md` files per phase, each with frontmatter (wave, depends_on, must_haves, requirements) and typed tasks (auto, checkpoint:human-verify, checkpoint:decision, tdd).

**Use when:** A phase needs to be broken down into an executable plan.

---

### groovy-project-researcher
**File:** `references/groovy-project-researcher.md`

Researches the domain ecosystem before roadmap creation. Runs in parallel with other researchers. Produces files in `.planning/research/` consumed by the roadmapper. Spawned by `/groovy:new-project` or `/groovy:new-milestone`.

**Research modes:** Ecosystem (default), Feasibility, Comparison

**Output files:** `STACK.md`, `FEATURES.md`, `ARCHITECTURE.md`, `PITFALLS.md`, optionally `COMPARISON.md` or `FEASIBILITY.md`

**Use when:** Starting a new project or milestone and domain research is needed before roadmapping.

---

### groovy-research-synthesizer
**File:** `references/groovy-research-synthesizer.md`

Reads the 4 parallel research outputs (STACK, FEATURES, ARCHITECTURE, PITFALLS) and synthesizes them into a unified `SUMMARY.md` with roadmap implications. Also commits all research files (researchers write but don't commit). Spawned by `/groovy:new-project` after researchers complete.

**Use when:** All parallel researcher agents have completed and their findings need to be unified.

---

### groovy-roadmapper
**File:** `references/groovy-roadmapper.md`

Transforms requirements into a phase structure with observable success criteria. Maps every v1 requirement to exactly one phase, validates 100% coverage, derives goal-backward success criteria (2-5 per phase), and initializes `STATE.md`. Spawned by `/groovy:new-project`.

**Outputs:** `ROADMAP.md` (summary checklist + detail sections + progress table), `STATE.md`, updated `REQUIREMENTS.md` traceability.

**Use when:** Requirements are defined and need to be organized into an executable project roadmap.

---

### groovy-verifier
**File:** `references/groovy-verifier.md`

Verifies that a phase achieved its **goal**, not just completed its tasks. Goal-backward analysis: derives must-haves (truths, artifacts, key links), checks all three levels (exists, substantive, wired), scans for stubs and anti-patterns, identifies human verification needs. Spawned by `/groovy:verify-work`.

**Status outcomes:** `passed`, `gaps_found`, `human_needed`

**Output:** `VERIFICATION.md` with YAML frontmatter gaps structured for `/groovy:plan-phase --gaps`.

**Use when:** A phase is complete and needs to be verified before moving to the next phase.

---

## Workflow Overview

```
/groovy:new-project
  └── groovy-project-researcher (x4 parallel: stack, features, arch, pitfalls)
  └── groovy-research-synthesizer
  └── groovy-roadmapper

/groovy:map-codebase
  └── groovy-codebase-mapper (x4 parallel: tech, arch, quality, concerns)

/groovy:plan-phase
  └── groovy-phase-researcher
  └── groovy-planner
  └── groovy-plan-checker (revision loop up to 3x)

/groovy:execute-phase
  └── groovy-executor (one per plan, waves respected)

/groovy:verify-work
  └── groovy-verifier
  └── groovy-integration-checker (milestone scope)

/groovy:debug
  └── groovy-debugger
```

## Key Principles

- **Mandatory Initial Read:** All agents check for `<files_to_read>` blocks in their prompt and load those files before doing anything else.
- **Write files directly:** Agents write to disk (never return large content to orchestrator) to reduce context load.
- **Structured returns:** Agents return concise status messages (COMPLETE, BLOCKED, CHECKPOINT REACHED) not full content.
- **Persistent state:** Debug sessions and phase state survive context resets via `.planning/` files.
- **Never `git add .`:** Executors always stage files individually.
