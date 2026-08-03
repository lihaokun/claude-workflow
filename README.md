# semiformal-sdd-workflow

**面向 coding agent + 人类开发者的半形式化 SDD 工程协议**。设计先于代码、契约对照审核、修复同等严谨——适用于算法 / 验证 / 长期可维护性项目。

**`workflow.md` 本身 agent-agnostic**——抽象流程规范，可用于任何 AI coding agent（Claude / GPT / Gemini 等）。项目规则模板和初始化 skill 只有一份，通过安装时复制到不同发现目录，同时支持 **Claude Code、Codex 与 OpenCode**。

包含：设计三阶段流程、半形式化规约书写规范、分层测试策略、根因分析驱动的 bug 修复流程、契约对照审核（消除自审 bias 的结构化双验证），以及 coding agent 行为准则。

适用于算法密集型项目（如数学库、编译器、形式化验证工具等），也可作为通用项目的开发规范基础。

## 文件说明

| 文件 | 用途 | 部署位置 |
|------|------|----------|
| `workflow.md` | 通用开发工作流程规范（核心文档） | 项目 `docs/workflow.md` |
| `AGENTS-template.md` | 唯一的项目规则模板 | 新项目复制为 `AGENTS.md`，并创建 `CLAUDE.md -> AGENTS.md` |
| `skills/new-project/SKILL.md` | 跨平台初始化 / 旧项目升级 skill | 安装到各 coding agent 的 skills 目录 |
| `templates/contract-audit/` | 契约对照审核 v2 模板（搭配 `workflow.md §6` 使用） | 项目 `templates/contract-audit/` |

## 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/lihaokun/semiformal-sdd-workflow.git
```

### 2. 部署模板文件

```bash
# 创建模板目录
mkdir -p ~/templates

# 复制唯一的项目规则模板和最新版 workflow
cp semiformal-sdd-workflow/AGENTS-template.md ~/templates/
cp semiformal-sdd-workflow/workflow.md ~/templates/

# 复制契约对照审核 v2 模板（搭配 workflow.md §6 使用，按需启用）
cp -r semiformal-sdd-workflow/templates/contract-audit ~/templates/
```

### 3. 安装同一个 new-project skill

不写入任何全局 `CLAUDE.md` 或 `AGENTS.md`。只把同一个 `SKILL.md` 复制到所用平台的发现目录。

#### Claude Code

```bash
mkdir -p ~/.claude/skills/new-project
cp semiformal-sdd-workflow/skills/new-project/SKILL.md \
  ~/.claude/skills/new-project/SKILL.md
```

#### Codex / OpenCode

Codex 和 OpenCode 都会读取 `~/.agents/skills/`：

```bash
mkdir -p ~/.agents/skills/new-project
cp semiformal-sdd-workflow/skills/new-project/SKILL.md \
  ~/.agents/skills/new-project/SKILL.md
```

### 4. 使用

| 平台 | 调用方式 |
|------|----------|
| Claude Code | `/new-project` |
| Codex | `$new-project` |
| OpenCode | 请求“使用 new-project skill 初始化 / 升级当前项目” |

skill 支持两条路径：

- **全新项目**：创建 `AGENTS.md` 实体和 `CLAUDE.md -> AGENTS.md` 相对软链接
- **旧项目升级**：保留已有的 `CLAUDE.md` 或 `AGENTS.md` 实体，创建另一文件名指向它的相对软链接

两条路径都会把 `docs/workflow.md` 同步到当前安装的最新版；发现项目定制时先展示 diff，不静默覆盖。

### 5. 更新模板并升级旧项目

先更新本仓库，再刷新本机安装的唯一模板和 skill：

```bash
git -C semiformal-sdd-workflow pull --ff-only

cp semiformal-sdd-workflow/AGENTS-template.md ~/templates/
cp semiformal-sdd-workflow/workflow.md ~/templates/
cp -r semiformal-sdd-workflow/templates/contract-audit ~/templates/

# 按已安装的平台执行一个或两个复制命令
cp semiformal-sdd-workflow/skills/new-project/SKILL.md \
  ~/.claude/skills/new-project/SKILL.md
cp semiformal-sdd-workflow/skills/new-project/SKILL.md \
  ~/.agents/skills/new-project/SKILL.md
```

然后在旧项目中再次调用 `new-project` skill。它会保留现有规则实体、补齐软链接，并以最新 `~/templates/workflow.md` 为基线升级项目 workflow。

## 适配依据

- Claude Code：[Skills](https://code.claude.com/docs/en/slash-commands)、[`CLAUDE.md` 与 `AGENTS.md`](https://code.claude.com/docs/en/memory)
- Codex：[`AGENTS.md`](https://developers.openai.com/codex/guides/agents-md)、[Skills](https://developers.openai.com/codex/skills)
- OpenCode：[Rules](https://opencode.ai/docs/rules/)、[Skills](https://opencode.ai/docs/skills/)

## 工作流程概览

```
新功能开发：
  调研 → [确认] → 架构 → [确认] → 细化 → [确认] → 审查 → 逐模块实现+测试+审核

新增子计划（带契约文档）—— §6 v2 强制流程（硬约束仅 coding agent）：
  Step 0   抽契约 expectations + 识别机械化部分 + 列 property-based invariants
  Step 1-3 引用共享常量 + 加 # Step Pn 注释 + property-based 测试覆盖
  Step 5   路径 A 自动检查（项目自备） + 路径 B subagent 独立审 → 决策 ⚠️ 项

Bug 修复（§7.1 修正方案文档 8 部分 + §7 主流程 6 步：测试同步 + 代码同步 + 文档同步）：
  局部错误     → 直接修复 + 测试同步 + 代码同步 + 文档同步
  算法内部错误 → 修正方案文档 → [确认] → 修复 + 测试同步 + 代码同步 + 文档同步
  接口/架构问题 → 修正方案文档 → [确认] → 更新设计 → 按实现流程执行
                                  + 测试同步 + 代码同步 + 文档同步

Coding agent 每步操作：
  说明计划 → [等待确认] → 执行单步 → 报告结果 → [等待反馈]
```

详见 [workflow.md](workflow.md)。

## 定制指南

这套模板是通用基础，建议根据项目特点定制：

- **workflow.md**：如果项目不涉及跨进程/跨机器通信，可整段跳过附录 A 分布式接口契约扩展。如果项目不涉及并发，可忽略 §3.4 并发规约。如果不是算法类项目，§7.1 的参考实现对照可简化
- **AGENTS-template.md**：技术栈、常用命令、代码风格、参考实现、已知限制等节均为渐进填充，项目初期留空不影响使用
- **skills/new-project/SKILL.md**：同时负责全新初始化与旧项目升级；已有普通文件或异常软链接绝不静默覆盖
- **templates/contract-audit/**：仅在引入 §6 v2 流程时需要部署到项目根（项目 `templates/contract-audit/`）。bug fix / refactor / 小项目可不部署。详见 `templates/contract-audit/README.md` 中的"何时建议你也用 v2"段

## 致谢

规约书写规范部分参考了 [SysSpec](https://arxiv.org/abs/2512.13047) 的形式化方法思路（Hoare-logic 风格前置/后置条件、rely-guarantee 并发规约）。

## License

MIT
