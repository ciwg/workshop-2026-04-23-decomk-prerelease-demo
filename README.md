class: center, middle

# Decomk Migration Workshop

## From Pre-Release Setup to Production-Ready Flow

### Migration architecture and command-sourcing model

???
Open with the outcome: this is a migration from ad-hoc prerelease practices to a production operating model.

---

## Executive Summary

- We are speed-running several years of evolution and training, and
  effort originally funding by multinational trillion-dollar
  corporations, into a few weeks of work
- So this is all pretty rough but coming together -- wouldn't be
  possible at all without LLMs.
- We are standardizing devcontainer setup on a single-execution-path
  lifecycle for both builds and updates.
- We use a baseline image for all repos' codespaces so we have
  predictable tools available for all.
- Builds get checkpointed periodically to keep first-boot times short.
- Codespaces start from those checkpoints and run user/repo-specific
  setup on top.

???
State the headline early so each later slide feels like evidence for these four claims.

---

## Current Status Snapshot

- `decomk` defines and enforces the stage-0/lifecycle contract.
- `decomk-conf-cswg` is active as both shared policy repo and shared image producer repo.
- Consumer repos are converging on release version of decomk, with bug
  fixes and image-consumption validation in `mob-sandbox` pilot.

???
Use this slide to close the argument: architecture, process, and rollout are aligned.

---


## Glossary 

- **Devcontainer**: a ready-to-use development environment for a repo, running in a container.
- **Stage-0**: the bootstrap step that prepares the container before normal work starts.
- **Devcontainer Lifecycle phases**:
  - `updateContent`: runs during build or prebuild.
  - `postCreate`: runs during boot.
- **Checkpoint image**: a prebuilt image that captures current state
  of a devcontainer after `updateContent` but before `postCreate`.
- **Producer repo**: the repo that builds/publishes shared checkpoint images.
- **Consumer repo**: a repo that uses those promoted images.

???
Define terms gently and clearly so later jargon feels familiar instead of abrupt.

---

## System Map

| Repo | Primary role | Operational value |
|---|---|---|
| `decomk` | Bootstrap/runtime engine | Executes policy deterministically |
| `decomk-conf-cswg` | Shared policy + checkpoint image producer | Centralizes setup logic and image publication |
| `mob-sandbox` | Pilot image consumer | Fast validation loop before wider rollout |

???
Use this as the stable map for the rest of the deck; keep returning to these three repos.

---

## Lifecycle Contract (Single Path)

Shared setup follows one canonical execution path:

1. Stage-0 calls `decomk run <action>`.
2. `decomk` resolves tuples from `decomk.conf`.
3. `make` runs the ordered target graph in stamp space.
4. `updateContent` runs during build.
5. `postCreate` runs during boot.

???
Emphasize that producer builds and consumer prebuilds share the same `updateContent -> decomk run` path.

---

## Producer/Consumer Image Management

- `decomk-conf-cswg` is the producer repo for checkpoint images as
  well as being the location for decomk.conf and the Makefile.
- Producer freezes shared state from the same prebuild path used in normal lifecycle execution.
- Consumer repos (`mob-sandbox`, `fpga-workbench`, others) reference promoted tags in `.devcontainer/devcontainer.json`.
- User/runtime customization remains in `postCreate`, outside shared checkpoint layers.

Reference design: `decomk/doc/image-management.md`.

???
Transition line: now that the runtime path is clear, show how images are produced once and consumed many times.

---

## Migration Timeline

1. JJ created pre-release `workspace-config`.
2. Steve forked it to `decomk-conf-cswg` and aligned it to post-release decomk behavior.
3. `mob-sandbox` became the pilot consumer for migration proof.
4. `fpga-workbench` is being converted in parallel.
5. Producer/consumer image flow is becoming the default operating pattern.

???
Keep this chronological and concrete; it anchors the technical slides in real repo history.

---

## What Changed: Pre-Release vs Post-Release

| Topic | Pre-release style | Post-release direction |
|---|---|---|
| Stage-0 source vars | Multiple legacy vars/modes | URI contract: `DECOMK_TOOL_URI`, `DECOMK_CONF_URI` |
| Tool install semantics | "latest" often assumed "newest commit" | `@latest` resolves to latest tagged version |
| Setup location | Mixed Dockerfile/hook logic | Single-path Makefile-driven flow |
| Image model | Weak producer/consumer separation | `decomk-conf-cswg` produces, target repos consume promoted images |

???
This is the key comparison slide; all earlier slides define the "after" column.

---

## Release Semantics: `@latest` Is Tag-Latest

In Go module installs:

- `go install module@latest` resolves to the latest tagged release,
- not necessarily the latest untagged commit on `main`.

Operational implication:

- release/tag policy must be explicit,
- deployment provenance must be auditable.

???
This is where engineering process meets tooling semantics; do not skip this nuance.

---

## Why Ordering Is a Hard Requirement

Infrastructure changes are self-referential:

- tools run inside the systems they modify,
- each change can alter the behavior of later changes,
- identical operations in different orders can produce different outcomes.

Therefore: deterministic ordering is required for predictable replay from test to production.

???
Bridge to the papers: the migration choices are grounded in this property, not just team preference.

---

## Event Sourcing (Accounting Journal Analogy)

- Event sourcing records what happened as a time-ordered event stream.
- Accounting example: journal entries.
  - Each debit/credit entry records a completed business event.
  - Current balances are derived by replaying entries in order.
- Source of truth: event history.

???
Set up the contrast first: event sourcing captures outcomes that already happened.

---

## Command Sourcing (Accounting Source-Document Analogy)

- Command sourcing records the instructions that produce state.
- Accounting source-document example:
  - purchase order, invoice, and receipt are the command-like inputs,
  - bookkeeping executes from those documents,
  - resulting journal entries are the events.

In this stack:

- Makefile stanzas and lifecycle actions are source documents,
- replay order is part of correctness,
- state is produced by executing that command history.

???
Tie the analogy explicitly: source documents drive actions; actions create events.

---

## House-Rebuild Analogy

- Imagine keeping every plan, permit, receipt, procedure, and work record for a building.
- Then the building burns down.
- You could theoretically rebuild it to match the original, including paint layers, if you had detailed records.
- How well reconstruction works depends on the level of detail you kept.
- Command sourcing works the same way: recovery quality depends on the fidelity of command history.

???
Keep this visual and concrete: reconstruction is possible when history is detailed enough.

---

## Source Documents and History Discipline

Practical operating rule:

- treat versioned setup stanzas as append-only history,
- add new stanzas for upgrades,
- avoid rewriting prior history except controlled corrective actions.

Result:

- clearer provenance,
- safer rollback reasoning,
- lower ambiguity during incident response.

???
This slide defines day-to-day editing behavior and change governance.

---

## Limits and Real-World Friction

Even with disciplined ordering:

- package repositories expire artifacts
- historical rebuilds can fail without local capture
  - so we keep old images as well as old config and Makefile stanzas
- occasional corrective intervention is still necessary.

So the model is:

- strict by default,
- explicit about exceptions.

???
Acknowledge operational reality so the model stays credible and usable.

---

## Execution Streams Right Now

- `mob-sandbox`: pilot consumer validating lifecycle and image-consumption behavior.
- `fpga-workbench`: parallel migration stream applying the same post-release model.
- `decomk-conf-cswg`: active policy and image producer for shared baseline evolution.

???
This turns the conceptual model back into current project execution status.

---

## Discussion Prompts

1. Which steps must remain shared-image responsibilities vs `postCreate` responsibilities?
2. Which changes require append-only stanzas, and what qualifies as an exception?
3. What tag/channel policy should govern `DECOMK_TOOL_URI` and produced checkpoint images?
4. Which repos should be next in migration order after current pilot confidence?

???
Invite decisions, not general commentary; these questions drive implementation planning.

---

## References

- Infrastructures.Org (project site): http://www.infrastructures.org
- Steve Traugott, Joel Huddleston. *Bootstrapping an Infrastructure* (LISA '98, original): https://www.usenix.org/legacy/publications/library/proceedings/lisa98/full_papers/traugott/traugott_html/traugott.html
- *Bootstrapping an Infrastructure* (Infrastructures.Org updated version): http://www.infrastructures.org/papers/bootstrap/bootstrap.html
- Steve Traugott, Lance Brown. *Why Order Matters: Turing Equivalence in Automated Systems Administration* (LISA '02 original): https://www.usenix.org/conference/lisa-02/why-order-matters-turing-equivalence-automated-systems-administration
- Steve Traugott, Lance Brown. *Why Order Matters: Turing Equivalence in Automated Systems Administration* (Infrastructures.Org): http://www.infrastructures.org/papers/turing/turing.html
- `decomk` image-management design note: `decomk/doc/image-management.md`

???
Keep this slide visible for Q&A; it provides provenance for both conceptual and implementation claims.

---

class: center, middle

# Thank You

### Questions and review of concrete next migration steps

???
Close by moving from Q&A into concrete next actions and owners.
