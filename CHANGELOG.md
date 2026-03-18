# CHANGELOG

## v0.6 — Public Release (2026-03-19)

### Changed
- Single recommended usage path: agent-driven workflow via `USAGE.md`
- Legacy v1 scripts (`run_*.py`) remain for reference but are not the primary path

### Added
- Unit tests for `analyze_triggers.py` and `analyze_quality.py` (24 tests)
- `.gitignore`, `LICENSE` (MIT), `CONTRIBUTING.md`

### Fixed
- False positive trigger detection when `skill_path` has no parent directory
- Removed hardcoded domain-specific trigger detection

---

## v0.5 — Description Diagnostics + Latency Profiling (2026-03-18)

- Description quality diagnosis with prioritized recommendations
- Latency profiling with p50/p90/std_dev statistics

---

## v0.4 — Model Comparison (2026-03-18)

- Cross-model Quality + Speed comparison (haiku/sonnet/opus)
- Three-dimension evaluation framework (Quality/Speed/Cost)

---

## v0.3 — Parallel Execution (2026-03-18)

- Parallel evaluation orchestrator (5-10x speedup)
- Concurrent transcript extraction

---

## v0.2 — Visualization (2026-03-17)

- HTML report generation (`viewer/`)
- Side-by-side comparison UI

---

## v0.1 — Core Framework (2026-03-08)

- Initial release
- Trigger rate detection
- Quality comparison (with vs without skill)
