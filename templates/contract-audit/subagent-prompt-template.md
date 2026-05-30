<!--
  workflow.md §6 路径 B: Subagent 独立审 prompt 模板
  使用方法: 主 agent 复制下方 prompt 填入 {{占位符}}, 用 Agent tool (subagent_type: general-purpose) 启动
  目的: 消除主 agent 自审的 confirmation bias
-->

# Subagent Audit Prompt — `<subplan-id>`

## 主 agent 启动方式

1. 主 agent 完成 Step 1-3 实施 + Step 5 路径 A（自动检查）后启动
2. 复制下方 "## Prompt" 段落的全部内容，填入 `{{占位符}}`
3. 用 Agent tool 启动 `general-purpose` subagent，将 prompt 传给它
   - **若你的 agent 环境没有 `general-purpose` 类型**，替换为等效的"独立审核 / 通用研究"类型（关键是该 subagent 不继承主 agent 的对话上下文）
4. subagent 产出 `docs/audits/{{SUBPLAN_ID}}/audit-report.md` 后返回
5. 主 agent review 报告，修 ❌ + 决策 ⚠️ → 记录到 `decisions.md`

## 占位符清单（启动前填）

| 占位符 | 含义 | 示例 |
|--------|------|------|
| `{{SUBPLAN_ID}}` | 子计划 ID | `<A-X>` |
| `{{SUBPLAN_TITLE}}` | 子计划标题（一句话） | `<feature-name 简述>` |
| `{{CONTRACT_DOCS}}` | 契约文档绝对路径列表（多个，每行一个） | 见底部启动示例 |
| `{{COMMIT_RANGE}}` | git diff 范围（用于 subagent 看 diff） | `HEAD~3..HEAD` 或 `<base_sha>..HEAD` |
| `{{IMPL_FILES}}` | 本子计划新增/修改文件相对路径（每行一个） | 见底部启动示例 |
| `{{EXPECTATIONS_PATH}}` | 主 agent 写的 expectations.md 路径 | `docs/audits/<subplan-id>/expectations.md` |

---

## Prompt（**复制以下全部内容**给 subagent）

````
你是独立契约审核者。任务: 审核 {{SUBPLAN_ID}} ({{SUBPLAN_TITLE}}) 的实现是否符合契约。

## 你的角色定位

你**不知道主 agent 写的代码**。你只**看契约**和**diff**, 然后**独立列出 expectations**, 再对比 diff 验证。

这是一次反向审核 — 不要"看代码总结契约", 要"看契约找代码漏什么"。

## 你必须读的文件

**契约文档（按重要性顺序）**:
{{CONTRACT_DOCS}}

**实施 diff**: 运行 `git diff {{COMMIT_RANGE}} -- {{IMPL_FILES}}` 获取（仅看 diff，不要 cat 完整文件）。

**主 agent 的 expectations**: `{{EXPECTATIONS_PATH}}`（**只读 §1-§10 的"契约"列**，**忽略**"实现位置"和"一致性"列——避免被主 agent 的判断污染你的独立审）。

## 你不应该读的文件

- 实施代码的完整内容（只能看 diff，不能 cat 整个 .py / .c / .java 等）
- 测试代码（除非 diff 里出现）
- 主 agent 的 commit message
- 任何在 docs/audits/{{SUBPLAN_ID}}/decisions.md 的旧决策（你做独立判断，不被旧决策影响）
- 任何旧的 docs/audits/{{SUBPLAN_ID}}/audit-report.md（若本次是迭代审核，独立做新判断，不被前次结论影响）

如果中途发现不看完整代码无法判断某项, 在 audit-report.md 中标注 "⚠️ 需主 agent 补充上下文" 并继续, 不要主动去读。

## 你的工作流程（4 步）

### Step 1: 独立列 expectations checklist

按 `templates/contract-audit/expectations-template.md` 的 10 节结构, 从契约文档**独立**抽出 expectations。

**不要**抄主 agent 的 expectations.md, 而是用契约原文重新抽。完成后**与**主 agent 的 expectations.md 对比, 检查:

- 主 agent 是否漏列契约项？(critical: 漏掉的 expectations)
- 主 agent 是否多列了契约文档没有的"伪 expectations"？(warning: 误增)
- 字段名/枚举值/流程步骤是否与契约文档**逐字符**一致？(critical: typo / 偏离)

### Step 2: 用 diff 验证 expectations 落地

逐条 expectations 在 git diff 里 grep:

- 字段名: diff 里是否出现完全相同的字段名？(grep `"<field_name>"`)
- 枚举值: diff 里是否引用共享常量, 还是重新定义 enum？
- 流程步骤: diff 里是否有 `# Step Pn:` 注释覆盖每步？
- 性能契约: 是否有单元测试时长断言？
- 副作用: 是否有 git diff 检查（如声明"零修改"的核心模块未被改）？

发现不一致, 标注**具体行号**（diff 行号或 file:line）。

### Step 3: 写 audit-report.md

输出到 `docs/audits/{{SUBPLAN_ID}}/audit-report.md`, 格式:

```markdown
# Audit Report — {{SUBPLAN_ID}}

**审核者**: Subagent (独立, 仅读契约 + diff)
**时间**: <YYYY-MM-DD>
**契约 review 范围**: <列契约文档 + 章节>
**diff 范围**: {{COMMIT_RANGE}}

## 总体结论
- ❌ Critical (must fix): N 项
- ⚠️ Warning (decide): M 项
- ✅ Verified: K 项

## ❌ Critical Issues

### #1 <issue 简述>
- **契约**: <文档 §X.Y 行号 + 原文引用>
- **实现**: <diff 中的偏离, 含 file:line>
- **建议修复**: <具体改动>

### #2 ...

## ⚠️ Warnings (建议主 agent 决策)

### #1 <issue 简述>
- **契约**: ...
- **实现**: ...
- **可选处置**:
  - 选项 A: 修代码对齐契约
  - 选项 B: 改契约对齐代码
  - 选项 C: 接受偏离, 加文档说明
- **subagent 倾向**: <A/B/C + 理由>

## ✅ Verified Expectations

### Schema 字段
- <字段名>: 契约 §X.Y → diff:LL ✅
- ...

### 枚举值 / 流程步骤 / 行为契约 / 时序 / 不变量 / 性能 / 副作用 / 跨实现 / 反向扩展
- <逐条同上>

## 主 agent expectations 对比

- 主 agent 的 expectations.md (§1-§10) 与 subagent 独立抽出的 checklist 差异:
  - 主 agent **漏列**: <列出 + 是否 critical>
  - 主 agent **误增**: <列出 + 是否 critical>
  - 主 agent **字段名 typo**: <列出>

## 需主 agent 补充上下文 (⚠️ 信息不足以判断)

- <如有: 哪些项 subagent 看不全 diff 不能判断>
```

### Step 4: 报告

返回给主 agent:

1. audit-report.md 的绝对路径
2. critical / warning / verified 数量
3. 最值得主 agent 立即修复的 top 3 critical 项

## 反偏置守则

- **禁止 echo bias**: 不要因为主 agent expectations 写了 X, 你就标 ✅。要从契约原文独立判断
- **禁止 anchor bias**: 不要被 diff 第一处看到的实现影响整体判断, 每条 expectation 独立 grep
- **禁止 confirmation bias**: 找不到证据时不要"应该没问题", 标 ⚠️ "需补充上下文"

## 输出长度建议

audit-report.md ~200-500 行。**不要**逐行解读 diff (那是代码 review 的工作), **只**对比契约 expectations 与 diff 的一致性。

完成后简要回报 (≤ 200 字), 关键信息: 报告路径 + critical 数 + top 3 critical 一句话。
````

---

## 启动示例（占位符填法）

```
{{SUBPLAN_ID}} = <A-X>
{{SUBPLAN_TITLE}} = <feature-name 一句话简述>
{{CONTRACT_DOCS}} =
  /abs/path/to/docs/research/<feature>-design.md §X.Y
  /abs/path/to/src/<module>/<DATA_FORMAT_SPEC>.md
  /abs/path/to/docs/research/<feature>-implementation-plan.md §X.Y §X.Z
{{COMMIT_RANGE}} = HEAD~1..HEAD
{{IMPL_FILES}} =
  src/<module>/file.py
  tests/test_<feature>.py
{{EXPECTATIONS_PATH}} = docs/audits/<A-X>/expectations.md
```

主 agent 启动语 (Agent tool):

```
description: <A-X> 独立契约审核
subagent_type: general-purpose
prompt: <填入填好占位符的 Prompt>
```
