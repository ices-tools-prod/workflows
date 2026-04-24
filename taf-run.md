# TAF Results Branch Model (`taf`)

This repository uses a structured and governed approach to publishing
Transparent Assessment Framework (TAF) results using a dedicated **`taf`**
branch.

The model is designed to balance:

- scientific reproducibility
- operational efficiency
- clean version history
- auditability for review and advice tracking

---

## Overview

- **`main` / `master`**
  Contains the analytical intent: code, inputs, and configuration.

- **`taf`**
  Contains *only* generated results (data, model outputs, reports) and
  machine‑readable metadata describing how those results were produced.

The `taf` branch is **fully automated** and **never edited manually**.

---

## Key principles

1. **Semantic tagging**
   - Results are tagged **only when explicitly requested**:
     - `--tag-advice`
     - `--tag-final`
   - Tags represent **official analytical milestones**.

2. **Rolling untagged results**
   - Between tags, the `taf` branch contains **a single rolling commit**
     representing the most recent run.
   - Intermediate runs do *not* accumulate history noise.

3. **Explicit provenance**
   - Each run writes a `taf-metadata.json` file describing:
     - input hashes
     - reuse decisions
     - squash behaviour
     - software versions
     - triggering commit and context

4. **Controlled history rewriting**
   - History is squashed **only on the generated `taf` branch**.
   - Tagged commits are never rewritten.

---

## TAF history evolution

The following diagram shows how the `taf` branch evolves over time.

```mermaid
gitGraph
  commit id: "Initial run"
  commit id: "Untagged run (rolling)"
  branch advice
  commit id: "Advice tag" tag: "advice-2026"
  checkout main
  checkout advice
  commit id: "Post-advice run (rolling)"
  commit id: "Updated run (squashed)"
  branch final
  commit id: "Final tag" tag: "final-2026"
