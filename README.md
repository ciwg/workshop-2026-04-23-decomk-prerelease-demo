class: center, middle

# Decomk Migration Workshop

## From Pre-Release Setup to Production-Ready Flow

### Layperson-friendly walkthrough of the `outline.md` topics

---

## Why This Deck Exists

This deck explains a real migration story across four repos:

- this workshop repo (slides)
- `decomk` (the bootstrap tool)
- `decomk-conf-cswg` (shared setup policy)
- `mob-sandbox` (pilot consumer repo)

Goal: make the technical outline understandable to people who do not live in this stack every day.

---

## Prerequisite Vocabulary

- **Devcontainer**: a pre-defined development environment (tools + settings) that opens in a container.
- **Codespaces**: GitHub-hosted devcontainers.
- **Bootstrap**: first steps that prepare an environment so work can start.
- **Config repo**: a shared repo that stores setup policy used by many project repos.
- **Lifecycle phases**:
  - `updateContent` = prebuild/common setup
  - `postCreate` = first-boot/user setup

---

## Repo Roles in Plain English

| Repo | Role | Why it matters |
|---|---|---|
| `decomk` | Runs setup rules | Turns policy into repeatable actions |
| `decomk-conf-cswg` | Shared policy (`decomk.conf` + `Makefile`) | One place to manage common setup |
| `mob-sandbox` | Test consumer | Safe place to prove changes before wider rollout |
| `workshop-2026-04-23-decomk-prerelease-demo` | Presentation scaffold | Communicates migration decisions |

---

## The Storyline from `outline.md`

1. A pre-release config repo existed (`workspace-config`, created by JJ).
2. Steve forked and evolved it into `decomk-conf-cswg` for post-release production use.
3. `mob-sandbox` is the guinea pig for validating the new pattern.
4. `fpga-workbench` migration is happening in parallel.
5. Underneath this migration is a bigger concept: command history should be treated as source-of-truth.

---

## What Changed: Pre-Release vs Post-Release

| Topic | Pre-release style | Post-release direction |
|---|---|---|
| Stage-0 source vars | Multiple legacy vars/modes | URI contract: `DECOMK_TOOL_URI`, `DECOMK_CONF_URI` |
| Tool install semantics | "latest" often assumed "newest commit" | `@latest` resolves to latest tagged version |
| Setup location | More mixed between Dockerfile/hooks | Prefer single-path Makefile-driven setup |
| Image model | Less explicit producer/consumer story | Config repo produces checkpointable shared state; target repos consume images |

---

## Important Detail: `@latest` Is Tag-Latest

In Go module installs:

- `go install module@latest` resolves to the latest tagged release
- it does **not** guarantee the latest untagged commit on `main`

Why this matters:

- DevOps workflows need explicit release discipline
- "what is deployed" must be predictable and auditable

---

## Lifecycle Model (Single Path)

Shared setup should follow one canonical flow:

1. `updateContent` runs common setup work.
2. Stage-0 script calls `decomk run <action>`.
3. `decomk` resolves tuples from `decomk.conf`.
4. `make` executes target graph in stamp space.

Then `postCreate` handles user/runtime-specific steps.

---

## Why `mob-sandbox` Is the Pilot

`mob-sandbox` is intentionally used as a proving ground before broader adoption.

It gives the team a place to:

- validate migration mechanics,
- test lifecycle behavior (`updateContent` and `postCreate`),
- catch regressions before applying the pattern to heavier repos.

---

## Parallel Workstream: `fpga-workbench`

While pilot validation happens in `mob-sandbox`, the team is also converting `fpga-workbench`.

Key post-release expectations there:

- avoid per-repo Dockerfile drift when possible,
- keep build/provision logic in Makefile targets,
- run decomk in both lifecycle phases.

---

## Big Concept: Event Sourcing vs Command Sourcing

- **Event sourcing** (accounting analogy): journal entries describe what happened.
- **Command sourcing** (this context): ordered setup commands create current machine state.

Simple framing:

- source documents in accounting -> transactions -> books
- source documents in infrastructure -> commands -> machine state

---

## Source Documents and History Discipline

In this model, a Makefile stanza is treated as a source document.

Practical rule:

- prefer append-only changes for versioned setup
- avoid rewriting old history unless a controlled exception is required

This keeps provenance clear and rollback reasoning easier.

---

## Blockchain Analogy (and Limits)

Useful analogy from the outline:

- blockchain records are intentionally hard to rewrite
- command history for infrastructure also benefits from tamper-evident thinking

But strict immutability has limits:

- real operations sometimes require corrective edits
- example: package ecosystem drift and cache/repository mismatch incidents

---

## Why This Is Hard in Practice

The outline highlights a structural challenge:

- many OS/package ecosystems are not designed for perfect long-term reproducibility
- repositories expire packages
- historical rebuilds can break without independent artifact capture

So teams need both:

- disciplined command history,
- and pragmatic recovery/override mechanisms.

---

## Mental Model for a Lay Audience

Think of this as "infrastructure bookkeeping":

- commands are ledger entries,
- order matters,
- edits to old entries are risky,
- reproducibility improves when the ledger is explicit and shared.

`decomk` provides the mechanism; the config repo provides policy.

---

## Current Migration Status Snapshot

- `decomk` defines the tool/lifecycle contract.
- `decomk-conf-cswg` is the active shared policy repo.
- `mob-sandbox` is the live pilot consumer.
- `fpga-workbench` conversion is in progress in parallel.

This is the concrete context behind the outline discussion.

---

## Suggested Discussion Prompts

1. Which setup steps are truly shared vs user-specific?
2. Where do we require append-only history, and where are controlled edits acceptable?
3. What release-tag policy should govern `DECOMK_TOOL_URI` in production?
4. Which repos should migrate next after pilot confidence is established?

---

class: center, middle

# Thank You

### Questions and review of concrete next migration steps
