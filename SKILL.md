---
name: project-workflow
description: Establish or audit a project's rules, task status, handoff and completion evidence. Use when the user asks to initialize project governance, reconcile documentation with reality, or improve an existing workflow. Preserve existing document locations and authorization; do not invoke just because an ordinary coding task has a repository.
license: MIT
metadata:
  version: "0.0.1"
---

# Project workflow

帮助项目明确规则、维护任务状态，并让完成结论有可复核依据。适用于支持 Agent Skills 的不同 Agent；不依赖特定产品、工具名称或安装目录。

## 1. 确定任务模式与边界

先从用户请求和已有上下文确定模式，缺少会影响行动的信息时才询问：

| 模式 | 允许的行为 |
|---|---|
| 检查 | 只读检查，报告事实、差异和建议；不创建文件或提交 |
| 方案 | 给出拟修改文件、理由及影响；不执行未授权修改 |
| 执行 | 在用户已授权范围内创建或更新；不因项目已有历史而重复请求同一授权 |

用户及宿主的有效指令优先于本Skill。Skill被调用不等于获得提交、推送、删除、发布或接管他人工作的权限。停止或只读要求立即生效。

## 2. 有限探测

1. 确认目标目录、适用规则与实际文档位置。代码/文档分仓时分别确认分支、工作区和事实源。
2. 有Git时读取status、暂存/未暂存差异摘要及少量近期相关历史；识别未跟踪文件，但不盲读凭据或私有数据。无Git时检查目录，不自动git init。
3. 读取规则入口、项目介绍、状态索引及当前模块资料；按发现的矛盾扩展阅读，不要求遍历全部文档。
4. 有未提交修改时确认归属与任务关系。只读检查和不相交的授权工作可继续；内容吻合不等于接手授权。不覆盖、stash、reset或提交无关改动。

## 3. 选择最小方案

- 新项目：从上下文明确范围、验收人和交付物。确有必要才补问；不为已有答案重复提问。
- 已有项目：优先沿用现有规则、需求表、Issue或文档仓库；先说明映射，不建立重复状态源。
- 新目录中的旧代码、monorepo子应用和双仓库都属于已有上下文；不要仅凭目录新就重建流程。
- 默认骨架见templates/，文件名可按项目适配。若宿主不自动读取AGENTS.md，复用其规则入口或加入短链接，不复制两份规则正文。
- 按任务选择参考资料：基本判断见[core-judgments](references/core-judgments.md)；变更边界见[boundaries](references/boundaries.md)；明确需要历史根因分析时读[archaeology](references/archaeology.md)；多人交接或并行工作时读[multi-agent](references/multi-agent.md)。不要默认加载所有参考文件。

## 4. 维护规则、状态与证据

- 规则：稳定约束、适用范围及原因。依据可来自明确需求、规范、一次严重事故或重复问题；区分已验证事实与建议，不机械等待第二次事故。
- 状态：目标、负责人、当前进展、下一步、关联版本与证据。人工验收和开发自检分别记录；Agent可按用户明确结论代记状态，不得自行制造批准。
- 证据：适用版本、环境、操作或命令、关键结果及未验证边界。不要求每个文档任务都运行代码；不要伪造执行结果。
- 验证按变更风险选择。构建通过不能证明无动态引用、无运行错误或所有场景通过；删除前检查实际入口、配置、外部调用和兼容需求。
- 提交遵守用户和项目规则，只包含本任务改动。允许不提交的交接，记录文件、差异及下一步；commit不等于共享、push或验收。

## 5. 输出并结束

说明：读取范围、发现及证据、已做/建议的修改、验证结果、未解决事项与下一步。指出保留或复用的现有入口；不得把未创建新文件误写成未完成。
检查模式交付报告即结束；执行模式完成授权内容后结束。只有真实缺失的产品决策、冲突或权限才需要暂停。

安装方式见[INSTALL.md](INSTALL.md)，许可见[LICENSE](LICENSE)。安装文档只在安装或迁移Skill时阅读，不影响日常项目流程。
