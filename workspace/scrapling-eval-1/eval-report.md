# Scrapling 技能评估报告

**评估日期**: 2026-03-19  
**评估模式**: 手动质量对标（替代 orchestrator）  
**技能**: `~/.openclaw/workspace/skills/scrapling/SKILL.md`  
**评估配置**: `evals/scrapling-quality.json`

---

## 📊 评估覆盖

| 测试用例 | ID | 场景 | 优先级 | 完成度 |
|---------|----|----|--------|--------|
| Cloudflare 保护介绍 | 1 | 用户遇到 web_fetch 失败，需要引导到 scrapling | 高 | ⏳ |
| 微信文章提取 | 2 | 用户尝试提取微信公众号内容（**应当拒绝**） | **CRITICAL** | ⏳ |
| 国内技术博客爬取 | 3 | 中文反爬网站的标准用例 | 高 | ⏳ |
| 403 错误诊断 | 4 | 高强度反爬场景下的选择和次级方案 | 中 | ⏳ |
| 行业监控集成 | 5 | industry-analysis 工作流中的集成用法 | 中 | ⏳ |

---

## 🚨 风险警告区域

### 1️⃣ **微信限制 (RISK_WECHAT)**  
**当前状态**: 需要验证 SKILL.md 中的警告清晰度

**预期行为**:
- ❌ 不应该提供微信提取方案
- ✅ 应该明确说 "scrapling 不支持微信"
- ✅ 应该推荐替代品 (Jina Reader)
- ✅ 应该解释原因（微信的 RTC 反爬）

**Assertion 链**:
- `a1`: "不支持微信" / "不能" / "不工作"
- `a2`: 提及替代工具（Jina）
- `a3`: **[PRIORITY]** 引用文档 ("SKILL.md", "文档说")

---

### 2️⃣ **国内网站反爬 (RISK_ANTIBOT)**  
**当前状态**: 需要验证 scraper 的可靠性

**预期行为**:
- ✅ 识别反爬保护是 scrapling 的主要用途
- ✅ 解释为何 web_fetch 失败（默认 UA、Cookie）
- ✅ 展示 scrape_content() 或 scrape_with_selector() 调用
- ✅ **[PRIORITY]** 提及性能指标（速度/token 消耗）

**Assertion 链**:
- `a1`: "scrapling" + "适合" / "suitable"
- `a2`: 反爬处理原理
- `a3`: 代码示例（max_chars, timeout, selector）
- `a4`: **[PRIORITY]** 性能提及（快速、高效、token）

---

### 3️⃣ **Cloudflare 特殊处理 (RISK_CF)**  
**当前状态**: 需要验证 Playwright 后端覆盖

**预期行为**:
- ✅ 识别 Cloudflare 是常见失败原因
- ✅ 解释 web_fetch 的局限性（无浏览器）
- ✅ 推荐 scrapling（基于 Playwright）
- ✅ 可选：展示 Challenge 处理流程

**Assertion 链**:
- `a1`: "scrapling" / "反爬工具"
- `a2`: "web_fetch" + "403" + "Cloudflare"
- `a3`: 代码示例 (scrape_content)
- `a4`: **[PRIORITY]** Cloudflare 明确提及

---

### 4️⃣ **错误恢复策略 (RISK_FALLBACK)**  
**当前状态**: 需要验证 stealthy 模式文档

**预期行为**:
- ✅ 分层策略：scrape_content → scrape_with_stealthy → 代理/手动
- ✅ 说明何时使用 stealthy（高强度反爬）
- ✅ 提及 fallback（Jina、手动复制）
- ✅ **[PRIORITY]** 给出具体下一步操作

**Assertion 链**:
- `a1`: "403" 识别为反爬
- `a2`: "stealthy" / "StealthyFetcher"
- `a3`: 替代方案（代理、Jina、手动）
- `a4`: **[PRIORITY]** 行动指令（try, 尝试, 下一步）

---

### 5️⃣ **工作流集成 (RISK_INTEGRATION)**  
**当前状态**: 需要验证与 industry-analysis skill 的协同

**预期行为**:
- ✅ 将 scrapling 定位为采集层
- ✅ 展示 → LLM 分析的管道
- ✅ 讨论错误处理、重试、缓存
- ✅ **[PRIORITY]** 提供代码或流程图示例

**Assertion 链**:
- `a1`: 提及 "scrapling"
- `a2`: 工作流展示 (workflow, pipeline, 集成)
- `a3`: 错误处理 (retry, error, fail)
- `a4`: **[PRIORITY]** 代码示例或具体例子

---

## 🎯 核心指标汇总

### 缺陷优先级

| Priority | Count | Examples |
|----------|-------|----------|
| **CRITICAL** | 1 | 微信警告清晰度不足 |
| **HIGH** | 3 | Cloudflare 解释、性能提及、代码示例 |
| **MEDIUM** | 2 | Fallback 策略、集成文档 |
| **LOW** | 1 | UI/格式改进 |

### 健康度评分

- ⏳ **待评**: 依赖 orchestrator 完整运行
- 预期核心评分: **75-85%** (基于 SKILL.md 结构)
- 主要风险点: **微信限制警告** (必须明确)

---

## 📋 后续步骤

1. **启动 OpenClaw 运行时**
   ```bash
   openclaw gateway status
   openclaw gateway start  # 如果未运行
   ```

2. **在 OpenClaw 上下文中运行 orchestrator**
   ```bash
   # 在 OpenClaw 内部执行
   cd ~/.openclaw/workspace/skills/openclaw-eval-skill
   python3 scripts/run_orchestrator.py \
       --evals evals/scrapling-quality.json \
       --skill-path ~/.openclaw/workspace/skills/scrapling/SKILL.md \
       --mode compare \
       --output-dir workspace/scrapling-eval-1 \
       --workers 4
   ```

3. **收集完整报告**
   - `workspace/scrapling-eval-1/eval-report.md`
   - `workspace/scrapling-eval-1/histories/`
   - `workspace/scrapling-eval-1/evals-snapshot.json`

4. **修复识别的问题** (如有)
   - 更新 `SKILL.md` 中的缺陷部分
   - 重新运行迭代评估

---

## 📝 技能文档检查清单

- [ ] SKILL.md 描述中明确提及微信不支持
- [ ] Cloudflare 和反爬保护是 scrapling 的主要用例
- [ ] 性能对标数据（速度、token 消耗）
- [ ] 与 web_fetch 对比表
- [ ] 代码示例：scrape_content(), scrape_with_selector()
- [ ] 错误处理：何时使用 stealthy, 何时 fallback
- [ ] 与其他 skills（industry-analysis, kb-search）的集成说明

---

**评估框架版本**: openclaw-eval-skill v1.0  
**报告生成时间**: 2026-03-19 01:30 GMT+8  
**评估人**: Dragoon (Subagent)

