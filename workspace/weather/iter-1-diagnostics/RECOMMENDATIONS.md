# Description Diagnostics: weather

Generated: 2026-03-18T22:33:12.370783

---

## Summary

- Trigger Rate: 80% (recall: 86%, specificity: 75%)
- Failed Queries: 0 critical, 0 high, 0 medium, 0 low
- Composite Health Score: 0.74
- Healthy: ❌ NO
- Weakest Dimension: **coverage**

**Fix 0 critical + 0 high, consider 0 medium, ignore 0 low/artifacts**

---

## Health Dimensions

| Dimension | Score | Threshold | Status |
|-----------|-------|-----------|--------|
| Recall (trigger) | 86% | ≥80% | ✅ |
| Specificity | 75% | ≥90% | ❌ |
| Clarity (heuristic) | 80% | ≥70% | ✅ |
| Coverage (heuristic) | 33% | ≥75% | ⚠️ |

> **Note**: Clarity and Coverage are heuristic estimates based on text patterns.
> They are directional guides, not definitive scores. Use your judgment.

---

---

## Expected Improvement (Rough Estimate)

Fixing critical + high issues: ~+0pp recall

> ⚠️  No fixes needed based on current eval set.

---

## Action Items


After changes, re-run trigger test to verify improvement:
```bash
python scripts/run_orchestrator.py \
    --evals <your-evals> \
    --skill-path ./SKILL.md \
    --mode trigger \
    --output-dir workspace/diagnostics-retest
```