# Groovy Skills

> A suite of specialized agents for planning, executing, debugging, and verifying software projects in structured phases.

## Overview

**Groovy Skills** is a collection of AI agent definitions that power the `/groovy:*` command workflow. Together, they guide a project through its entire lifecycle — from initial research and roadmap creation through phase-by-phase planning, execution, integration verification, and debugging.

Each agent is a focused, single-responsibility worker that writes its output directly to disk and returns a concise status to the orchestrator.

---

## Workflow Commands

| Command | Purpose |
|---|---|
| `/groovy:new-project` | Research the domain and generate a roadmap |
| `/groovy:map-codebase` | Analyze an existing codebase across 4 focus areas |
| `/groovy:plan-phase` | Research, plan, and quality-check a phase |
| `/groovy:execute-phase` | Execute a ready `PLAN.md` atomically |
| `/groovy:verify-work` | Verify a phase achieved its goal |
| `/groovy:debug` | Investigate and fix a bug systematically |

---

## Agent Index

### `groovy-project-researcher`
Researches the domain ecosystem before roadmap creation. Runs in parallel across 4 focus areas (stack, features, architecture, pitfalls). Produces `STACK.md`, `FEATURES.md`, `ARCHITECTURE.md`, and `PITFALLS.md` in `.planning/research/`.

### `groovy-research-synthesizer`
Reads the 4 parallel research outputs and synthesizes them into a unified `SUMMARY.md` with roadmap implications. Also commits all research files.

### `groovy-roadmapper`
Transforms requirements into a phase structure with observable success criteria. Maps every requirement to exactly one phase, and initializes `STATE.md`. Produces `ROADMAP.md`.

### `groovy-codebase-mapper`
Explores a codebase for a specific focus area and writes structured analysis documents to `.planning/codebase/`.

| Focus | Output Files |
|---|---|
| `tech` | `STACK.md`, `INTEGRATIONS.md` |
| `arch` | `ARCHITECTURE.md`, `STRUCTURE.md` |
| `quality` | `CONVENTIONS.md`, `TESTING.md` |
| `concerns` | `CONCERNS.md` |

### `groovy-phase-researcher`
Researches how to implement a specific phase. Produces `RESEARCH.md` (with verified confidence levels) consumed by the planner. Uses Context7, official docs, and WebSearch.

### `groovy-planner`
Creates executable `PLAN.md` files with task breakdown, dependency analysis, and goal-backward `must_haves`. Decomposes phases into parallel-optimized plans (2–3 tasks each) with wave assignments.

**Task types:** `auto`, `checkpoint:human-verify`, `checkpoint:decision`, `tdd`

### `groovy-plan-checker`
Verifies that plans **will** achieve the phase goal before execution. Performs goal-backward analysis across 8 dimensions:

1. Requirement Coverage
2. Task Completeness
3. Dependency Correctness
4. Key Links Planned
5. Scope Sanity
6. Verification Derivation
7. Context Compliance
8. Nyquist Compliance

### `groovy-executor`
Executes `PLAN.md` files atomically — one commit per task. Handles deviations automatically, pauses at checkpoints, and creates `SUMMARY.md` while updating `STATE.md`.

**Deviation rules:**
- Auto-fix bugs
- Auto-add missing critical functionality (security, correctness)
- Auto-fix blocking issues
- Ask before architectural changes

### `groovy-verifier`
Verifies that a phase achieved its **goal**, not just completed tasks. Produces `VERIFICATION.md` structured for `/groovy:plan-phase --gaps`.

**Status outcomes:** `passed` · `gaps_found` · `human_needed`

### `groovy-integration-checker`
Verifies cross-phase integration and end-to-end flows. Checks exports/imports wiring, API route coverage, auth protection, and E2E flow completeness.

### `groovy-debugger`
Investigates bugs using the scientific method (hypothesis testing, evidence gathering, falsifiability). Manages persistent debug sessions via `.planning/debug/` that survive context resets.

**Modes:** Interactive · `find_root_cause_only` · `find_and_fix`

---

## Workflow Diagram

```
/groovy:new-project
  ├── groovy-project-researcher (×4 parallel: stack, features, arch, pitfalls)
  ├── groovy-research-synthesizer
  └── groovy-roadmapper

/groovy:map-codebase
  └── groovy-codebase-mapper (×4 parallel: tech, arch, quality, concerns)

/groovy:plan-phase
  ├── groovy-phase-researcher
  ├── groovy-planner
  └── groovy-plan-checker (revision loop up to 3×)

/groovy:execute-phase
  └── groovy-executor (one per plan, waves respected)

/groovy:verify-work
  ├── groovy-verifier
  └── groovy-integration-checker (milestone scope)

/groovy:debug
  └── groovy-debugger
```

---

## Key Principles

- **Mandatory Initial Read** — All agents load their `<files_to_read>` blocks before doing anything else.
- **Write files directly** — Agents write to disk (never return large content to the orchestrator) to reduce context load.
- **Structured returns** — Agents return concise status messages (`COMPLETE`, `BLOCKED`, `CHECKPOINT REACHED`), not full content.
- **Persistent state** — Debug sessions and phase state survive context resets via `.planning/` files.
- **Never `git add .`** — Executors always stage files individually.

---

## File Structure

```
Groovy Skills/
├── README.md               # This file
├── SKILLS.md               # Machine-readable skill manifest
└── agents/
    ├── groovy-codebase-mapper.md
    ├── groovy-debugger.md
    ├── groovy-executor.md
    ├── groovy-integration-checker.md
    ├── groovy-phase-researcher.md
    ├── groovy-plan-checker.md
    ├── groovy-planner.md
    ├── groovy-project-researcher.md
    ├── groovy-research-synthesizer.md
    ├── groovy-roadmapper.md
    └── groovy-verifier.md
```