# Reference

Detailed specifications for evals.json format and assertion types.

---

## evals.json Format

### Quality Compare

```json
{
  "skill_name": "my-skill",
  "evals": [
    {
      "id": 1,
      "name": "onboarding-fresh",
      "prompt": "Check the weather in Tokyo",
      "context": "Clean machine, no prior setup. For grader only.",
      "expected_output": "Install → configure → verify profile",
      "assertions": [
        {
          "id": "a1-1",
          "description": "Install command executed",
          "type": "output_contains",
          "value": "pip install"
        },
        {
          "id": "a1-2",
          "description": "Profile verified after setup",
          "type": "output_contains",
          "value": "profile current",
          "priority": true
        }
      ]
    }
  ]
}
```

### Trigger Rate

```json
{
  "id": 1,
  "name": "direct-weather",
  "query": "What's the weather in Singapore?",
  "expected": true,
  "category": "positive"
}
```

---

## Assertion Types

| Type | Detection |
|------|-----------|
| `output_contains` | Value appears in conversation or tool output |
| `output_not_contains` | Value does not appear |
| `output_count_max` | Occurrences ≤ max |
| `tool_called` | Specific tool called at least once |
| `tool_not_called` | Specific tool not called |
| `conversation_contains` | Value appears anywhere in conversation |
| `conversation_contains_any` | At least one value appears |

**Priority assertions** (`"priority": true`): any failure → overall=FAIL.
**Gap assertions** (`"note": "Best practice..."`): failure = skill design gap.

---

## Issue Priority

```
🔴 P0 Critical  — Core functionality broken
🟠 P1 High      — Significantly impacts usability
🟡 P2 Medium    — Room for improvement
🟢 P3 Low       — Minor polish
```

---

## agents/ Files

| File | Purpose | When to Use |
|------|---------|-------------|
| `grader.md` | Check assertions, record behavior anomalies, give priority recommendations | Required for every Quality Compare eval |
| `comparator.md` | Blind A/B comparison without assertions | When unbiased comparison is needed |
| `analyzer.md` | Analyze cross-eval patterns after all evals complete | Post-analysis |

---

## Directory Structure

```
eval-workspace/<skill-name>/
├── evals.json                    ← Eval definition (shared across iterations)
└── iteration-1/
    ├── raw/
    │   ├── histories/            ← Trigger test session histories
    │   └── transcripts/          ← Quality compare transcripts
    ├── trigger_results.json      ← analyze_triggers output
    ├── quality_results.json      ← analyze_quality output
    └── diagnostics/
        └── RECOMMENDATIONS.md
```
