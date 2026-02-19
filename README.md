# claude-workflow

面向 Claude Code 的通用开发工作流程规范模板。

包含设计三阶段流程、半形式化规约书写规范、分层测试策略、根因分析驱动的 bug 修复流程，以及 Claude 行为准则。适用于算法密集型项目（如数学库、编译器、形式化验证工具等），也可作为通用项目的开发规范基础。

## 文件说明

| 文件 | 用途 | 部署位置 |
|------|------|----------|
| `workflow.md` | 通用开发工作流程规范（核心文档） | 项目 `docs/workflow.md` |
| `CLAUDE-template.md` | 项目级 CLAUDE.md 模板 | 复制为项目根目录 `CLAUDE.md` |
| `global-CLAUDE.md` | 全局规则：新项目时提醒初始化 | 合并到 `~/.claude/CLAUDE.md` |
| `new-project.md` | 自定义命令：一键初始化项目规范 | `~/.claude/commands/new-project.md` |

## 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/<你的用户名>/claude-workflow.git
```

### 2. 部署模板文件

```bash
# 创建模板目录
mkdir -p ~/templates

# 复制模板
cp claude-workflow/CLAUDE-template.md ~/templates/
cp claude-workflow/workflow.md ~/templates/
```

### 3. 部署 Claude Code 配置

```bash
# 全局规则（追加到已有内容后）
mkdir -p ~/.claude
cat claude-workflow/global-CLAUDE.md >> ~/.claude/CLAUDE.md

# 自定义命令
mkdir -p ~/.claude/commands
cp claude-workflow/new-project.md ~/.claude/commands/new-project.md
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

Bug 修复：
  局部错误     → 直接修复 + 补测试 + 回归
  算法内部错误 → 修正方案文档 → [确认] → 修复 + 补测试 + 回归
  接口/架构问题 → 修正方案文档 → [确认] → 更新设计 → 按实现流程执行 + 回归

Claude 每步操作：
  说明计划 → [等待确认] → 执行单步 → 报告结果 → [等待反馈]
```

详见 [workflow.md](workflow.md)。

## 定制指南

这套模板是通用基础，建议根据项目特点定制：

- **workflow.md**：如果项目不涉及并发，可忽略 §3.3 并发规约。如果不是算法类项目，§5.1 的参考实现对照可简化
- **CLAUDE-template.md**：技术栈、常用命令、代码风格、参考实现、已知限制等节均为渐进填充，项目初期留空不影响使用
- **global-CLAUDE.md**：如果你已有全局 CLAUDE.md，将内容合并而非覆盖

## 致谢

规约书写规范部分参考了 [SysSpec](https://arxiv.org/abs/2512.13047) 的形式化方法思路（Hoare-logic 风格前置/后置条件、rely-guarantee 并发规约）。

## License

MIT
