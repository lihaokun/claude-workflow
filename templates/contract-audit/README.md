# Contract Audit Templates

> **流程规范**: `workflow.md §6 契约对照审核 v2`

## 这是什么

新增子计划（带契约文档的实施任务）的强制审核流程模板。**取代**单纯依赖 §5.2 五维度自审核的旧做法。

## 为什么需要

**经验教训**：传统"凭经验直觉读代码"的自审核流程容易**漏检字段名 typo 与 forward-compat 字段缺陷**。根本原因：

1. **方向错了**：从代码"事后回填"expectations，被代码现状污染
2. **同一审核者 bias**：写代码的人审自己的代码，confirmation bias 不可消除
3. **维度不全**：仅看 schema/枚举，漏行为/时序/不变量/性能/安全契约
4. **manual checklist 易疲劳**：长清单准确率下降

v2 流程的核心改进：**Step 0 提前抽 expectations + Step 5 双验证（自动 + 独立 subagent）**。

## 文件清单

```
templates/contract-audit/
├── README.md                         # 本文件
├── expectations-template.md          # Step 0 用：抽契约 expectations 的 10 节模板
└── subagent-prompt-template.md       # Step 5 用：启动 subagent 独立审的 prompt 模板
```

**配套——项目自备**（每个下游项目自行实现，不在本仓库提供）：
```
scripts/contract_audit/                # 自动检查工具（机械化部分，workflow.md §6.5 列出应覆盖的检查项；脚本名 / 入口 / 语言项目自选）
```

**配套——运行时产出**（流程执行过程中自动生成，每子计划一份）：
```
docs/audits/<subplan-id>/
  ├── expectations.md                  # Step 0 复制 expectations-template 填写
  ├── audit-report.md                  # Step 5 路径 B subagent 产出
  └── decisions.md                     # ⚠️ 项的处置记录
```

## 完整使用流程

### Step 0 — 规划阶段必产出 expectations.md

```bash
# 1. 复制模板到 docs/audits/<subplan-id>/
mkdir -p docs/audits/<subplan-id>
cp templates/contract-audit/expectations-template.md docs/audits/<subplan-id>/expectations.md

# 2. 编辑填写 §1-§10
#    关键纪律: 必须先读契约不读现有代码
```

填写优先级：
- **§1 契约源**：列出所有相关 doc + 章节
- **§2-§4 机械化部分**：字段 / 枚举 / 流程，配合自动校验脚本
- **§5-§6 人审部分**：行为 / 时序契约，配测试断言
- **§7 不变量**：用 property-based 测试框架覆盖（如 Python `hypothesis`、Haskell `QuickCheck`、Rust `proptest`、Java `jqwik`、JavaScript `fast-check`）
- **§8-§10**：性能 / 安全 / 跨实现一致性

### Step 1-3 — 实施时遵循 Step 0 的机械化约束

代码必须：
- **引用共享常量**（不重新定义）：例 `from <module> import <SHARED_ENUM>`
- **加 `# Step Pn:` 注释**：每个流程步骤代码处标注，与 expectations §4 对照
- **写 property-based 测试覆盖 §7 不变量**：自动 fuzz 验证

### Step 5 路径 A — 自动检查（机械化）

```bash
# 具体脚本由项目自备，脚本名 / 入口 / 语言项目自选。本文档示例用 run_all.py，仅作占位：
python scripts/contract_audit/run_all.py <subplan-id>
```

应覆盖（workflow.md §6.5 路径 A）：
- JSON Schema 验证 JSON 输出
- `# Step Pn:` 注释 grep（验证 P1-Pn 全在）
- 共享枚举 import 检查（无重复定义）
- git diff 检查（声明「零修改」的核心模块未被改动）
- 性能测试时长断言

### Step 5 路径 B — Subagent 独立审

按 `subagent-prompt-template.md` 启动 subagent：

1. 主 agent 填占位符（`{{SUBPLAN_ID}}` / `{{CONTRACT_DOCS}}` / `{{COMMIT_RANGE}}` 等）
2. 用 Agent tool（subagent_type=`general-purpose`）启动
3. subagent 看契约 + diff（**不看完整代码 + 不看 expectations 的"实现位置"列**）
4. subagent 独立列 expectations + 与主 agent 对比 + 产出 `audit-report.md`

### 收尾 — 修 + 决策

主 agent 收到 audit-report.md 后：

```bash
# 1. 修所有 ❌ Critical
# 2. 决策所有 ⚠️ Warning, 记录到 decisions.md
echo "# Decisions — <subplan-id>" > docs/audits/<subplan-id>/decisions.md
# 写每个 ⚠️ 项的处置: 修 / 文档化 / 接受偏离 + 理由
```

## decisions.md 格式

```markdown
# Decisions — <subplan-id>

## ⚠️ #1 <issue 简述（来自 audit-report.md）>

- **subagent 报告**: docs/audits/<id>/audit-report.md ⚠️#1
- **决策**: 修代码对齐契约 / 改契约对齐代码 / 接受偏离
- **理由**: <为什么这么决策>
- **行动**: <具体改了什么 + commit hash>

## ⚠️ #2 ...
```

## 退出准则（每个子计划）

进入下一阶段或 commit/push 前必须 4 个 ✅：

- [ ] `expectations.md` §1-§10 全部填完
- [ ] 路径 A 自动检查全过
- [ ] `audit-report.md` 0 个 critical / 全部 ⚠️ 都已 decisions.md 处置
- [ ] §5.2 五维度审核通过（兜底）

## FAQ

**Q1: 为什么 Step 0 不能从代码抽 expectations？**

避免被代码现状污染。代码可能有字段名 typo、行为偏离契约——如果从代码抽 expectations，就把 bug 当成"应该如此"了。必须从契约文档独立抽。

**Q2: Subagent 看不全代码会不会判断错？**

会有"⚠️ 需补充上下文"项，subagent 会标出，让主 agent 补充信息或人工判断。这比"看全代码后被代码影响"好。

**Q3: 这套流程会不会太重？**

只用于**新增子计划（带契约文档）**。bug fix / 小修补可仅用 §5.2。`expectations.md` 通常 200-500 行，subagent audit 是一次性 cost。相比漏检后的修复成本（fix commit + 文档同步 + 解释），v2 流程的开销更低。

**Q4: 主 agent 已经有 §5.2 自审核，subagent 独立审是不是冗余？**

不是。§5.2 是**人审兜底**（5 维度直觉判断），subagent 独立审是**结构化对照**（10 节 checklist 逐条 grep）。两者互补：subagent 负责"严谨"，§5.2 负责"灵活"。

**Q5: Step 0 expectations 与设计文档里的规约重复吗？**

不重复。设计 / impl-plan 是**自然语言任务清单**（"实现 X 函数"），expectations 是**结构化契约对照表**（"X 函数的字段名 / 枚举 / 流程 / 不变量"）。expectations 更细粒度，可机械化验证。

**Q6: 我是协作者（不是主开发者），需要遵守 v2 流程吗？**

不强制。v2 硬约束**仅针对 AI coding agent**，权威定义见 `workflow.md §6.1 约束对象声明`。

简短版：
- 你看到 `docs/audits/<subplan-id>/` 目录 = v2 审核产物，与日常开发无关
- 你的 PR 不需要填 expectations.md，除非你**也在做新增子计划**
- bug fix / refactor / 文档 / 测试 / 基建 = 仍按 §5.2 五维度自审核

**何时建议你也用 v2**:
- 大型新功能（涉及跨多文件 + 引入新 schema / 枚举 / 流程契约）
- 跨语言一致性场景（如在多语言 runtime 间保持算法 bit-exact）
- 你想确保审核质量，主动用机械化工具

**何时不要用 v2**:
- 小 bug fix
- refactor 不改契约
- 工程基建（Makefile / CI / 依赖升级）

**Q7: 怎么把这套流程引入到我的项目？**

1. 复制 `templates/contract-audit/`、`workflow.md §6` 到目标项目
2. 在 `scripts/contract_audit/` 下实现 §6.5 路径 A 列出的检查项（项目自备，可用 Python / Shell / 任意语言）
3. 在目标项目根创建 `docs/audits/` 目录
4. 修改 `CLAUDE.md` / `AGENTS.md` 引入"契约对照审核 v2"段（保持硬约束语义）

**项目内默认值需调整**:
- 共享 enum 的 import 路径
- "零修改"保护文件清单
- 自动检查脚本所需依赖（如 JSON Schema 校验库）

## 引用关系

- 流程定义: `workflow.md §6`
- 模板使用方: 各子计划 `docs/audits/<id>/expectations.md`
- 自动工具: `scripts/contract_audit/`（由项目自备）
- AI agent 约束: `CLAUDE.md` / `AGENTS.md`
