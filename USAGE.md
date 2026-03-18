# 使用指南（Mo 驱动）

**版本**：v2（2026-03-18）

**核心原则**：Mo 负责所有 OpenClaw API 调用（spawn/history），Python 脚本只做纯数据分析。

---

## 两层架构

```
Layer 1: Mo（主 session）
  ├─ 读 evals.json
  ├─ sessions_spawn → subagents
  ├─ sessions_history → 提取数据
  └─ 写文件到 raw/ 目录

Layer 2: Python 脚本（exec 运行）
  ├─ 读 raw/ 目录里的 JSON/txt
  ├─ 做统计分析
  └─ 输出报告
```

---

## 工作流一：Trigger Rate 测试

### Step 1：Mo 执行 spawn

```
对每个 eval query，spawn 一个 subagent：

task = f"""
你是 OpenClaw assistant。
判断下面这个 query 是否需要读 {skill_path}，如果是就读并使用它，不是就直接回答。

Query: {query}
"""

sessions_spawn(task=task, mode="run", cleanup="keep", label=f"trigger-eval-{id}")
```

等所有 subagent 完成（收到 announce 信号）。

### Step 2：Mo 提取 history，写文件

```
对每个 session key，调：
sessions_history(sessionKey=sk, includeTools=True)

把结果写到：
workspace/{skill}/iter-{n}/raw/histories/eval-{id}.json
格式：{"eval_id": 1, "query": "...", "expected": true, "messages": [...]}
```

### Step 3：脚本分析

```bash
python3 scripts/analyze_triggers.py \
    --evals evals/{skill}/triggers.json \
    --histories workspace/{skill}/iter-{n}/raw/histories/ \
    --output workspace/{skill}/iter-{n}/trigger_results.json
```

### Step 4（可选）：诊断

```bash
python3 scripts/run_diagnostics.py \
    --evals evals/{skill}/triggers.json \
    --skill-path /path/to/SKILL.md \
    --trigger-results workspace/{skill}/iter-{n}/trigger_results.json \
    --output-dir workspace/{skill}/iter-{n}/diagnostics/
```

---

## 工作流二：Quality Compare（with vs without skill）

### Step 1：Mo spawn（两组）

```
对每个 eval，spawn 两个 subagent：

# With skill
sessions_spawn(
    task=f"Read {skill_path}. Then: {prompt}",
    mode="run", cleanup="keep", label=f"q-eval-{id}-with"
)

# Without skill
sessions_spawn(
    task=prompt,
    mode="run", cleanup="keep", label=f"q-eval-{id}-without"
)
```

### Step 2：Mo 提取 transcript，写文件

```
sessions_history(sessionKey=sk_with, includeTools=False)
→ 提取 assistant 的最后一条文本消息
→ 写到 workspace/{skill}/iter-{n}/raw/transcripts/eval-{id}-with.txt

同样处理 without。
```

### Step 3：脚本分析

```bash
python3 scripts/analyze_quality.py \
    --evals evals/{skill}/quality.json \
    --transcripts workspace/{skill}/iter-{n}/raw/transcripts/ \
    --output workspace/{skill}/iter-{n}/quality_results.json
```

---

## 工作流三：Model Comparison

### Step 1：Mo spawn（多模型 × 多 eval × N 次）

```
对每个 (model, eval, run)：
sessions_spawn(
    task=f"Read {skill_path}. Then: {prompt}",
    model=full_model_name,  # e.g. "anthropic/claude-haiku-4-5"
    mode="run", cleanup="keep",
    label=f"mc-eval-{id}-{model}-run-{n}"
)

记录 spawn 时间：start = time.now()
收到 announce 后：elapsed = time.now() - start
```

### Step 2：Mo 写数据文件

```
Transcript:
workspace/{skill}/iter-{n}/raw/model-compare/eval-{id}-{model}-run-{n}-transcript.txt

Timing:
workspace/{skill}/iter-{n}/raw/model-compare/eval-{id}-{model}-run-{n}-timing.json
格式：{"eval_id": 1, "model": "haiku", "run": 1, "elapsed_seconds": 9.2}
```

### Step 3：脚本分析

```bash
python3 scripts/analyze_model_compare.py \
    --evals evals/{skill}/quality.json \
    --data-dir workspace/{skill}/iter-{n}/raw/model-compare/ \
    --models haiku,sonnet \
    --dimensions quality,speed \
    --output-dir workspace/{skill}/iter-{n}/model-compare/
```

---

## 工作流四：Latency Profile

### Step 1：Mo spawn（同一 eval 重复 N 次）

```
# N-runs per eval
for run in range(1, n_runs+1):
    sessions_spawn(
        task=f"Read {skill_path}. Then: {prompt}",
        model=model,
        mode="run", cleanup="keep",
        label=f"lat-eval-{id}-{model}-run-{run}"
    )
    # 记录 elapsed
```

### Step 2：Mo 写 timing 文件

```
workspace/{skill}/iter-{n}/raw/timings/eval-{id}-{model}-run-{r}.json
{"eval_id": 1, "model": "sonnet", "run": 1, "elapsed_seconds": 12.3}
```

### Step 3：脚本分析

```bash
python3 scripts/analyze_latency.py \
    --evals evals/{skill}/quality.json \
    --timings-dir workspace/{skill}/iter-{n}/raw/timings/ \
    --models haiku,sonnet \
    --output-dir workspace/{skill}/iter-{n}/latency/
```

---

## 目录结构约定

```
workspace/
└── {skill}/
    └── iter-{n}/
        ├── raw/
        │   ├── histories/              ← trigger test session histories
        │   │   └── eval-{id}.json
        │   ├── transcripts/            ← quality compare transcripts
        │   │   ├── eval-{id}-with.txt
        │   │   └── eval-{id}-without.txt
        │   ├── model-compare/          ← model comparison data
        │   │   ├── eval-{id}-{model}-run-{n}-transcript.txt
        │   │   └── eval-{id}-{model}-run-{n}-timing.json
        │   └── timings/                ← latency profile timings
        │       └── eval-{id}-{model}-run-{n}.json
        │
        ├── trigger_results.json        ← analyze_triggers 输出
        ├── quality_results.json        ← analyze_quality 输出
        ├── model-compare/              ← analyze_model_compare 输出
        │   ├── compare_matrix.json
        │   └── model_comparison_report.md
        ├── latency/                    ← analyze_latency 输出
        │   ├── latency_report.json
        │   └── latency_report.md
        └── diagnostics/                ← run_diagnostics 输出
            ├── diagnosis.json
            └── RECOMMENDATIONS.md
```

---

## 命名规范

| 变量 | 例子 |
|------|------|
| `{skill}` | `weather`, `cobo-wallet` |
| `{n}` | `1`, `2`, `3`（每次迭代递增） |
| `{id}` | eval 的 id 字段 |
| `{model}` | `haiku`, `sonnet`, `opus` |
| `{r}` / `{run}` | 从 1 开始 |

---

## 注意事项

- **每次迭代独立文件夹**：`iter-1/`, `iter-2/`... 不覆盖历史数据
- **先 spawn 再等**：Mo spawn 完所有 subagent 后，等待所有 announce 信号，再批量提取 history
- **timing 从 spawn 开始计时**：`start = time.now()` 在调 sessions_spawn 前，`elapsed` 在收到 announce 后
- **transcript 提取**：取 sessions_history 中 role=assistant 的最后一条 text 消息
