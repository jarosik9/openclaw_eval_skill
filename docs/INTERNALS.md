# Internals

Technical details for contributors and advanced users.

---

## Architecture

**Two-layer design**:

```
Layer 1: Agent (main OpenClaw session)
  → Reads evals.json
  → Calls sessions_spawn to run subagents
  → Calls sessions_history to collect results
  → Writes raw data to workspace/

Layer 2: Python analysis scripts (run via exec)
  → Read the raw data from workspace/
  → Compute statistics
  → Generate reports
```

Python scripts (`analyze_*.py`) are data processors — they cannot call `sessions_spawn`. The agent drives the workflow.

---

## Runtime Actions

This skill performs the following actions during evaluation:

| Action | Purpose | When |
|--------|---------|------|
| Read `~/.openclaw/openclaw.json` | Find skill directories (extraDirs) | Path resolution |
| Write to `eval-workspace/` | Store evaluation results | Every eval run |
| Call `sessions_spawn` | Run test queries in isolated sessions | Trigger & quality tests |
| Call `sessions_history` | Collect conversation data for analysis | After each spawn |
| Persist `cleanup="keep"` sessions | Required for trigger detection | Trigger rate tests |

**NOT performed automatically**: Gateway restart, config modification, skill installation.

---

## Evaluation Tiers

### Tier 1: Core (Always Run)

| Scenario | What It Tests | Output |
|----------|---------------|--------|
| **Trigger Rate** | Does description trigger SKILL.md reads? | recall, specificity, precision, F1 |
| **Quality Compare** | Does skill improve output vs no-skill baseline? | quality_score, assertion pass rate |
| **Description Diagnosis** | Why did triggers fail? | gap analysis, recommendations |

### Tier 2: Optional

| Scenario | What It Tests | When to Use |
|----------|---------------|-------------|
| **Model Comparison** | Quality + speed across haiku/sonnet/opus | Before deployment |
| **Efficiency Profile** | Response time + retry patterns | When skill feels slow |

### Tier 3: Future (Roadmap)

| Scenario | Status |
|----------|--------|
| Cross-skill Conflict | Planned |
| Error Recovery | Planned |

---

## Key Constraints

- **`sandbox="inherit"`** — subagents inherit skill registration environment
- **`cleanup="keep"`** — history must be retained for trigger detection
- Skill must be in a real directory under `skills.load.extraDirs` (symlinks rejected)

---

## Behavior Anomaly Tracking

Grader records these signals beyond assertions:

| Field | Trigger |
|-------|---------|
| `path_corrections` | Wrong path then self-corrected |
| `retry_count` | Same command executed multiple times |
| `missing_file_reads` | Attempted to read non-existent files |
| `skipped_steps` | Steps required by skill were not executed |
| `hallucinations` | Fabricated non-existent commands/APIs |
