---
name: skill-eval
description: "Skill evaluation framework. Use when: testing trigger rate, quality compare (with/without skill), or model comparison. Runs via sessions_spawn + sessions_history. Trigger words: evaluate skill, benchmark, trigger rate, A/B compare. NOT for: debugging conversations, general testing unrelated to skill evaluation."
metadata: { "openclaw": { "emoji": "🔬" } }
---

# skill-eval

Evaluate any OpenClaw skill's trigger accuracy and output quality.

---

## Quick Start

Just say:
```
evaluate weather
```

The agent will:
1. Resolve skill path automatically
2. Run trigger rate + quality compare tests
3. Output results to `eval-workspace/weather/iter-N/`

**Options**:
- `evaluate weather trigger` — trigger rate only
- `evaluate weather quality` — quality compare only

---

## What It Tests

| Test | Question | Output |
|------|----------|--------|
| **Trigger Rate** | Does description trigger SKILL.md reads at the right times? | recall, precision, F1 |
| **Quality Compare** | Does skill improve output vs no-skill baseline? | quality_score, assertions |
| **Model Comparison** | Which model is enough for this skill? | haiku/sonnet/opus scores |

---

## Bundled Test Skill: fake-tool

A test skill is included for validating trigger detection. It simulates a fictional "Zephyr API" that models cannot know from training.

**Setup** (manual):
```bash
cp -r test-skills/fake-tool ~/.openclaw/workspace/skills/
openclaw gateway restart
```

**Verify**:
```bash
python scripts/resolve_paths.py fake-tool
```

---

## Creating Evals

### Trigger Rate (`evals/{skill}/triggers.json`)

```json
[
  { "id": 1, "query": "What's the weather?", "expected": true },
  { "id": 2, "query": "What's 2+2?", "expected": false }
]
```

### Quality Compare (`evals/{skill}/quality.json`)

```json
{
  "skill_name": "weather",
  "evals": [
    {
      "id": 1,
      "prompt": "Weather in Tokyo",
      "assertions": [
        { "type": "tool_called", "value": "exec" },
        { "type": "output_contains", "value": "Tokyo" }
      ]
    }
  ]
}
```

---

## Workflows

See **`USAGE.md`** for step-by-step instructions:

| Workflow | Section |
|----------|---------|
| Trigger Rate | Workflow 1 |
| Quality Compare | Workflow 2 |
| Model Comparison | Workflow 3 |

---

## Output

Results go to `eval-workspace/{skill}/iter-{n}/`:

```
eval-workspace/weather/iter-1/
├── raw/histories/          ← Session data
├── trigger_results.json    ← Trigger analysis
├── quality_results.json    ← Quality scores
└── RECOMMENDATIONS.md      ← Improvement suggestions
```

---

## Principles

1. **Never modify the skill** — observe only
2. **Keep full records** — save complete session histories
3. **Iterate** — each run creates `iter-N` for comparison

---

## More Details

- `docs/REFERENCE.md` — evals.json format, assertion types
- `docs/INTERNALS.md` — architecture, advanced scenarios
- `agents/grader.md` — grading prompt for quality compare
