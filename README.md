# Arboleda — Engineering Agent Harness

**An evidence-driven engineering-agent experiment built on Pi and Claude Opus 5.5.**

This repository packages a coding-agent harness for the Physical-Engineering's Last Exam (PLE). The aim is not simply to make a model produce plausible engineering prose: it is to study whether an agent can inspect a real task workspace, use engineering software and tools, create the exact requested deliverables, and verify its work under a fixed evaluation protocol.

The current baseline runs the [Pi](https://github.com/badlogic/pi-mono) coding-agent framework with `google-cloud/claude-opus-5-5`. Pi supplies the agent loop and tool execution; Claude Opus 5.5 is the model. The competition harness connects that agent to the isolated task environment through the PLE WebSocket protocol.

> **Research integrity:** measured baseline results are separated from hypotheses and planned experiments. No improvement is claimed until a controlled run and its artifacts support it.

## Baseline snapshot

The committed `artifacts/` directory contains a 29-task dev-set run. The manifest records the harness/model identity, task-level rewards, generated deliverables, and run metadata.

| Baseline measure | Recorded value |
|---|---:|
| Model | `google-cloud/claude-opus-5-5` |
| Agent framework | Pi |
| Dev tasks | 29 |
| Mean task reward (simple arithmetic mean of manifest rewards) | **0.4912** |
| Tasks with reward 0 | 4 |
| Tasks with reward at least 0.80 | 7 |
| Manifest creation time | 2026-10-20 03:33:56 UTC |

These figures describe the committed manifest, not a claim about a current leaderboard position. Task rewards are not interchangeable with engineering sign-off or proof that a design is safe.

### What the baseline already tells us

The results are heterogeneous. Several tasks score highly, including `backfeed-energization-reply` and `data-center-site-model` (1.0 each), while four tasks have zero reward: `foundation-strategy`, `solar-witness-test-package`, `data-hall-mep-routing`, and `data-hall-mep-routing-offset`.

That spread makes the dev set useful as a diagnostic corpus. It argues against treating one aggregate score as an explanation of agent capability. The next step is to inspect task instructions, source files, produced artifacts, and rubric feedback together before assigning causes to failures.

## Research questions

The engineering focus is on failure mechanisms that generic coding-agent benchmarks can hide:

1. **Evidence extraction:** Does the agent correctly reconcile drawings, schedules, specifications, calculations, correspondence, and existing project files?
2. **Engineering interpretation:** Does it distinguish a drawing note from a governing requirement, identify the applicable code or standard, and preserve assumptions and units?
3. **Quantitative verification:** Are calculations independently recomputed, dimensional consistency checked, and solver or software outputs sanity-checked?
4. **Deliverable fidelity:** Does the agent create every requested file at the exact path, name, revision, and format, rather than stopping after a convincing narrative?
5. **Uncertainty and reviewability:** Does it expose missing information, conflicting evidence, limitations, and decisions that require a licensed engineer?
6. **Time-budget discipline:** Does it create a usable first deliverable early, verify it, and finish cleanly before the task deadline?

These are research questions, not claims that the current baseline already solves them.

## Experimental plan

The baseline should remain reproducible while changes are tested one at a time.

| Experiment | Intervention | What to measure |
|---|---|---|
| Evidence-first workflow | Require an inventory of instructions, source files, and constraints before substantive work | Task reward; missed requirements; unsupported assumptions |
| Calculation verification | Add independent recomputation and unit/dimensional checks where appropriate | Numerical discrepancies caught; reward; time cost |
| Deliverable checklist | Track every requested output path, format, and revision | Missing/incorrect deliverables; reward |
| Code-and-standard traceability | Require each consequential criterion to be tied to supplied evidence or an identified governing reference | Incorrect or unsupported code claims; review effort |
| Time-aware execution | Reserve explicit time for validation and finalization | Timeout/unfinished rate; deliverable completeness |
| Engineer comparison | Have a qualified engineer independently review the same task packet and rubric | Agreement, omissions, correction time, and risk-weighted error categories |

For each experiment, keep the dev-set snapshot, model, prompt, harness code, and evaluation procedure fixed except for the named intervention. Report per-task deltas as well as the aggregate. Preserve negative results: an intervention that adds latency or complexity without improving reliability is useful evidence.

### Failure-analysis protocol

For every low-scoring or zero-reward task:

- Read the complete task instruction and identify the requested deliverables and acceptance criteria.
- Inspect the starting workspace and relevant source material.
- Compare the generated files and final message with the rubric and manifest.
- Classify the failure only when evidence supports it: for example, source interpretation, missing requirement, calculation, code applicability, file/path/format, tool execution, or premature termination.
- Record the evidence, likely mechanism, confidence, and a testable intervention.
- Re-run the task after changing one factor; do not present a plausible explanation as an observed root cause.

A useful engineering-agent result is not only a score. It is a traceable account of what the system read, what it inferred, what it calculated, what it produced, and what a human reviewer still needs to decide.

## Architecture

```text
PLE evaluator
    │ WebSocket task protocol
    ▼
serve.py ── taskbridge.py
    │
    ▼
harness.py ── Pi agent loop ── Claude Opus 5.5
    │
    ├── task-tools.ts: read / write / edit / bash
    └── taskbridge relay: task filesystem, commands, task MCP servers
    │
    ▼
Isolated task workspace → requested deliverables + final summary
```

- **Pi** manages the model/agent loop, tool calls, retries, and conversation compaction.
- **Claude Opus 5.5** performs the reasoning and generates tool calls and responses.
- **`harness.py`** configures the provider, model limits, reasoning level, MCP servers, and task prompt, then runs one headless Pi session per task.
- **`task-tools.ts`** replaces Pi's local file and shell tools with operations against the task container.
- **`taskbridge.py`** implements the competition protocol and relays task operations and MCP traffic.
- **`serve.py`** exposes the WebSocket endpoint expected by the evaluator.

The task container has no general internet access. The harness should rely on the supplied task evidence and declared MCP tools, not assume it can browse external sources.

## Run it

This is a PLE submission repository, not a standalone engineering calculator. Follow the competition repository's current instructions and use its kit from the competition repository root.

1. Clone this repository beside your PLE competition repository.
2. Set a valid entry name in `submission.yaml`.
3. Ensure the required local dependencies are available (`uv`, Docker, and Harbor for dev-set runs). A Google Cloud model route may require the local model proxy described in the competition documentation.
4. From the competition repository root, run:

   ```bash
   uv run python -m kit check ../Arboleda-Google---Agent-Harness-on-Phygital-Engineering
   uv run python eval/run.py ../Arboleda-Google---Agent-Harness-on-Phygital-Engineering
   ```

5. Inspect the new `artifacts/` and task-level results. Commit the harness and corresponding artifacts together.

The exact commands and requirements can change with the competition kit; consult its current documentation and the detailed [AGENTS.md](AGENTS.md) in this repository. The checked-in artifacts are a historical baseline and will be replaced by a new run when the harness is evaluated again.

## Repository map

| Path | Purpose |
|---|---|
| `submission.yaml` | Entry name, model declaration, install and serve commands |
| `serve.py` | Competition WebSocket server |
| `harness.py` | Pi session, provider configuration, task prompt, reasoning and model limits |
| `task-tools.ts` | Task-container file and shell tools |
| `taskbridge.py` | PLE protocol and task/MCP relay |
| `package.json`, `package-lock.json` | Pinned Pi package and dependencies |
| `requirements.txt` | Python dependencies |
| `AGENTS.md` | Detailed competition and harness operating instructions |
| `artifacts/` | Per-task dev-set outputs and `manifest.json` |

## Engineering responsibility

This project is an experimental AI-agent system. Its outputs are not a substitute for review, approval, or professional responsibility by a qualified and appropriately licensed engineer. Any real-world use requires independent verification against the project requirements, applicable codes and standards, authoritative source documents, and the responsible engineer's judgment.

## License

See [LICENSE](LICENSE).
