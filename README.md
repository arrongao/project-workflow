# project-workflow

面向不同 Agent 的项目协作 Skill：分开管理规则、任务状态与完成证据。纯 Markdown，无运行时依赖。最初用于 DeepSeek Harness，现已抽离宿主专属约束。

## 使用

- 只读检查：“使用 project-workflow 检查当前项目的规则、状态与验证证据，不修改文件。”
- 制定方案：“使用 project-workflow 为现有项目提出治理改进，沿用已有需求表。”
- 执行：“使用 project-workflow 按已确认方案补齐文档，保留其他任务改动，不提交。”

Codex可用`$project-workflow`；Claude Code可用`/project-workflow`；其他宿主使用其技能选择器或明确要求读取SKILL.md。引用方式由宿主决定。

## 设计原则

规则写稳定约束，状态写进展，证据说明结论依据。优先复用项目现有资料，读取范围与任务相称。支持单仓库、独立文档仓库和没有Git的目录。不会自动初始化Git、接手他人工作、部署或发布。

默认模板包括AGENTS.md、BACKLOG.md、WORKFLOW.md、README.md和.gitattributes。它们是可裁剪骨架，不是必需文件清单；已有同职责文件时复用。安装Skill也不会自动创建这些项目文件。

## 安装与兼容

见[各Agent安装说明](INSTALL.md)。通用格式遵循[Agent Skills标准](https://agentskills.io/specification)。宿主路径、刷新与工具权限不属于标准统一保证；没有在所有Agent上进行运行验收。

目录：SKILL.md为执行入口；references/按需读取；templates/为生成用骨架；tests/scenarios.md为手动行为评估场景。没有绑定MCP或自动执行脚本。

## 发布与许可

采用[MIT License](LICENSE)，许可覆盖本仓库原创Skill、参考说明和模板。允许商用、修改及再分发，须保留许可要求的声明；不要求使用者把自己的项目整体改为MIT。外部链接所指内容遵守各自许可。

GitHub公开仓库并不自动赋予开源授权。发布时保留LICENSE和SKILL.md的license字段，确认拥有分发所包含内容的权利。本次版权署名使用仓库账号arrongao。

源码仓库：[arrongao/project-workflow](https://github.com/arrongao/project-workflow)。首个公开版本为 `v0.0.1`，内部迭代版本号不代表既有公开发行版。

[版本与下载](https://github.com/arrongao/project-workflow/releases) · [发布检查](PUBLISHING.md)
