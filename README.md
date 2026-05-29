# semiformal-sdd-workflow

**面向 coding agent + 人类开发者的半形式化 SDD 工程协议**。设计先于代码、契约对照审核、修复同等严谨——适用于算法 / 验证 / 长期可维护性项目。

**`workflow.md` 本身 agent-agnostic**——抽象流程规范，可用于任何 AI coding agent（Claude / GPT / Gemini 等）。本仓库提供的部署配套文件（`CLAUDE-template.md` / `global-CLAUDE.md` / `new-project.md`）以 **Claude Code** 为参考实现，其它 agent 用户可类比适配。

包含：设计三阶段流程、半形式化规约书写规范、分层测试策略、根因分析驱动的 bug 修复流程、契约对照审核（消除自审 bias 的结构化双验证），以及 coding agent 行为准则。

适用于算法密集型项目（如数学库、编译器、形式化验证工具等），也可作为通用项目的开发规范基础。

## 文件说明

| 文件 | 用途 | 部署位置 |
|------|------|----------|
| `workflow.md` | 通用开发工作流程规范（核心文档） | 项目 `docs/workflow.md` |
| `CLAUDE-template.md` | 项目级 CLAUDE.md 模板 | 复制为项目根目录 `CLAUDE.md` |
| `global-CLAUDE.md` | 全局规则：新项目时提醒初始化 | 合并到 `~/.claude/CLAUDE.md` |
| `new-project.md` | 自定义命令：一键初始化项目规范 | `~/.claude/commands/new-project.md` |
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

# 复制核心模板
cp semiformal-sdd-workflow/CLAUDE-template.md ~/templates/
cp semiformal-sdd-workflow/workflow.md ~/templates/

# 复制契约对照审核 v2 模板（搭配 workflow.md §6 使用，按需启用）
cp -r semiformal-sdd-workflow/templates/contract-audit ~/templates/
```

### 3. 部署 Claude Code 配置

```bash
# 全局规则（追加到已有内容后）
mkdir -p ~/.claude
cat semiformal-sdd-workflow/global-CLAUDE.md >> ~/.claude/CLAUDE.md

# 自定义命令
mkdir -p ~/.claude/commands
cp semiformal-sdd-workflow/new-project.md ~/.claude/commands/new-project.md
```

### 4. 使用

在新项目目录中启动 Claude Code，它会自动询问是否初始化。也可以随时手动执行：

```
/new-project
```

命令会自动复制模板并提示你填写项目信息。

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
- **CLAUDE-template.md**：技术栈、常用命令、代码风格、参考实现、已知限制等节均为渐进填充，项目初期留空不影响使用
- **global-CLAUDE.md**：如果你已有全局 CLAUDE.md，将内容合并而非覆盖
- **templates/contract-audit/**：仅在引入 §6 v2 流程时需要部署到项目根（项目 `templates/contract-audit/`）。bug fix / refactor / 小项目可不部署。详见 `templates/contract-audit/README.md` 中的"何时建议你也用 v2"段

## 致谢

规约书写规范部分参考了 [SysSpec](https://arxiv.org/abs/2512.13047) 的形式化方法思路（Hoare-logic 风格前置/后置条件、rely-guarantee 并发规约）。

## License

MIT
