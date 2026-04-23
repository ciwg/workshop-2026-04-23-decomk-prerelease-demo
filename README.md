class: center, middle

# Decomk Migration Workshop

## From Pre-Release Setup to Production-Ready Flow

### Migration architecture and command-sourcing model

???
Open with the outcome: this is a migration from ad-hoc prerelease practices to a production operating model.

---

## Executive Summary

- We are standardizing devcontainer setup on a single-path lifecycle.
- `decomk-conf-cswg` now serves two roles: policy repo and image producer repo.
- Consumer repos move faster by consuming promoted checkpoint images.
- Ordering discipline is a design requirement, not an implementation preference.

???
State the headline early so each later slide feels like evidence for these four claims.

---

## System Map

| Repo | Primary role | Operational value |
|---|---|---|
| `decomk` | Bootstrap/runtime engine | Executes policy deterministically |
| `decomk-conf-cswg` | Shared policy + checkpoint image producer | Centralizes setup logic and image publication |
| `mob-sandbox` | Pilot consumer | Fast validation loop before wider rollout |

???
Use this as the stable map for the rest of the deck; keep returning to these three repos.

---

## Lifecycle Contract (Single Path)

Shared setup follows one canonical execution path:

1. `updateContent` runs common setup.
2. Stage-0 calls `decomk run <action>`.
3. `decomk` resolves tuples from `decomk.conf`.
4. `make` runs the ordered target graph in stamp space.
5. `postCreate` handles runtime/user-specific changes.

???
Emphasize that producer builds and consumer prebuilds share the same `updateContent -> decomk run` path.

---

## Producer/Consumer Image Management

- `decomk-conf-cswg` is the producer repo for checkpoint images.
- Producer builds freeze shared state from the same prebuild path used in normal lifecycle execution.
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

## Infrastructures.Org Lineage

- Infrastructures.Org emerged from enterprise infrastructure work in the late 1990s and early 2000s.
- `isconf` and later `decomk` share this lineage.
- Two papers frame the model used here:
  - *Bootstrapping an Infrastructure* (1998)
  - *Why Order Matters* (2002)

???
Position these papers as the conceptual foundation for the operating model now being applied.

---

## Paper Lens: Bootstrapping an Infrastructure (1998)

Key contributions used in this migration:

- infrastructure as a policy-driven virtual machine,
- bootstrap through explicit dependency ordering,
- centralized source documents over host-local drift,
- one method that scales from one host to many hosts.

???
Tie this directly to config repo policy + ordered Makefile execution.

---

## Paper Lens: Why Order Matters (2002)

Key contributions used in this migration:

- host administration is computationally expressive and self-modifying,
- circular dependencies are unavoidable,
- test environments are mandatory,
- production must replay the tested order to control risk.

???
Connect this to checkpoint promotion and deterministic rollout practices.

---

## Command Sourcing in Practice

- Event sourcing tracks events that occurred.
- Command sourcing tracks ordered instructions that create state.

In this stack:

- Makefile stanzas are source documents,
- replay order is part of correctness,
- state is the result of executed command history.

???
Keep this practical: this is not abstract theory, it is how reproducibility is enforced.

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

- package repositories expire artifacts,
- historical rebuilds can fail without local capture,
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

## Current Status Snapshot

- `decomk` defines and enforces the stage-0/lifecycle contract.
- `decomk-conf-cswg` is active as both shared policy repo and shared image producer repo.
- Consumer repos are converging on promoted-tag consumption.
- The migration is active, not theoretical.

???
Use this slide to close the argument: architecture, process, and rollout are aligned.

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
