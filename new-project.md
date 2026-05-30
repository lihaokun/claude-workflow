初始化新项目的规范文件：

1. 从 ~/templates/CLAUDE-template.md 复制为当前项目根目录的 CLAUDE.md
2. 创建 docs/ 目录（如不存在）
3. 从 ~/templates/workflow.md 复制到 docs/workflow.md
4. 询问用户：本项目是否预计有"新增子计划（带契约文档）"的工作？
   - 如是：从 ~/templates/contract-audit/ 复制到项目根 templates/contract-audit/（启用 workflow.md §6 v2 流程）
   - 如否或不确定：跳过本步，后续如需启用，再手动复制
5. 提示用户填写 CLAUDE.md 中的以下占位信息：
   - 项目名称
   - 项目描述
   - 项目结构（如已有文件则根据实际情况调整）
6. 告知用户：技术栈、常用命令、代码风格、参考实现、已知限制等节可在开发过程中逐步补充
