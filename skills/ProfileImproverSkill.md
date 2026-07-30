# ProfileImproverSkill

> **触发指令：** 当用户说出 `启动 ProfileImproverSkill` 时，立即读取本文件并严格按照 Step 0 → Step 4 流程执行。每步需用户确认后方可进入下一步，否则阻塞等待。

> 流程化更新 `.github` 仓库内容的 Skill。

---

## 工作流

### Step 0 — 读取上下文

读取以下两份文档以建立全局认知：

| 顺序 | 文件 | 目的 |
|------|------|------|
| 1 | `.github/history/GENERAL.md` | 了解仓库整体变更历程概要（注意 `tints` 元数据约束：总字数 ≤ 500） |
| 2 | `.github/history/<latest_date>_record.md` | 明确上一次更改的具体任务与产出 |

---

### Step 1 — 收集意图 & PLAN MODE

向用户提问：

> **"你想对仓库进行什么增 / 删 / 改？请描述你的需求。"**

收到用户回复后，进入 **PLAN MODE**：

1. 将用户需求拆解为 **原子级任务列表**（每项可独立验收）
2. 对每项任务标注：
   - 影响范围（涉及哪些文件）
   - 预期产出（新增/修改/删除什么）
   - 依赖关系（是否有先后顺序）
3. 将完整 PLAN 展示给用户，明确询问：

> **"以上是我的理解与执行计划，是否准确？有需要调整的地方吗？"**

4. 阻塞等待用户确认。用户可能：
   - ✅ 确认 → 进入 Step 2
   - 🔄 修正 → 更新 PLAN 后再次确认
   - ❌ 取消 → 终止流程

---

### Step 2 — TODO LIST 执行

1. 将确认后的 PLAN 转化为 **序号 TODO LIST**（markdown checklist 格式）
2. 逐项执行，每完成一项：
   - 自行验收（验证产出是否符合预期）
   - 标记为 `- [x]`
   - 向用户简短反馈该项的完成情况
3. 全部完成后进入 Step 3

---

### Step 3 — SUBAGENT · Archiver（变更归档）

> ⚠️ 执行前须征得用户确认

**职责：** 扫描 `git diff`，将本次变更记录到 history 目录。

**执行逻辑：**

1. 运行 `git diff --stat` 获取变更文件列表
2. 运行 `git diff` 获取详细差异
3. 检查 `.github/history/` 下是否已有当天日期的 record 文档（格式 `YYYY-MM-DD_record.md`）：
   - **存在** → 在现有文档末尾追加本次变更内容
   - **不存在** → 新建当天 record 文档
4. Record 文档格式参照 `2026-07-30_record.md`，包含：
   - 标题：`# YYYY-MM-DD Changelog`
   - `## 新增` / `## 修改` / `## 删除` 分类记录
   - 每项注明涉及文件与变更说明
5. 更新 `.github/history/GENERAL.md`：
   - 更新 `updated` 元数据日期
   - 在「更新概要」表格中新增/更新当日条目
   - 更新「当前状态」描述
   - 确保总字数不超出 `tints` 约束（500 字）, 如果需要新增的内容会突破限制，需要对现有内容压缩精简

---

### Step 4 — SUBAGENT · Committer（提交推送）

> ⚠️ 执行前须征得用户确认

**职责：** 撰写 commit message 并执行 `git commit && git push`。

**执行逻辑：**

1. 基于 `git diff --stat` 和变更内容，生成规范的 commit message：
   - 格式：`<type>(<scope>): <简短描述>`
   - type 类型：`feat` / `fix` / `style` / `refactor` / `docs` / `chore`
   - scope：`readme` / `assets` / `skills` / `history` / `meta`
   - body：按文件逐条列出变更摘要
2. 将 commit message 展示给用户确认
3. 用户确认后执行：
   ```bash
   git add -A
   git commit -m "<message>"
   git push
   ```

---

## 约束规则

| # | 规则 |
|---|------|
| 1 | **每步确认**：Step 1~4 的入口均需用户明确确认后才执行 |
| 2 | **阻塞等待**：用户未确认前，流程在该步骤阻塞 |
| 3 | **GENERAL.md ≤ 500 字**：Archiver 写入时须检查字数，超出则精简或拆分 |
| 4 | **Record 幂等**：同一天多次修改追加到同一 record 文件 |
| 5 | **Commit 原子性**：一次完整流程对应一个 commit |
