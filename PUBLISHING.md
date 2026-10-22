# 发布前检查

1. 确认本仓库原创内容及可分发权利；LICENSE署名当前为arrongao。若权利归组织或有其他权利人，先调整相应声明。外部来源不能仅靠新增MIT而重新许可。
2. 检查将公开的文件及Git历史，避免暴露项目私有材料、身份信息或凭据；本次通用化不重写历史。
3. 运行可用的Skill格式检查，并按tests/scenarios.md进行需要的行为评估；明确哪些Agent只是文档适配、哪些已实测。
4. 审阅Git diff、MIT文本及安装命令；选择版本/tag。首个公开版本为v0.0.1；metadata、tag及发行说明的版本必须一致。
5. 用户授权后再创建GitHub仓库或push；确认真实URL后更新下载链接。公开仓库本身不替代LICENSE。

MIT允许商用、修改和再分发，要求保留版权及许可声明，并按原文提供免责条款；不要求下游整体开源。完整法律条款以LICENSE为准。

依据：[OSI MIT License](https://opensource.org/license/mit)。本文件不附加额外许可限制。

## 双仓库同步发布

本项目每个版本都同步发布到以下两个仓库；这是本项目的发布约定，不是通用 Skill 模板规则。

- GitHub： https://github.com/arrongao/project-workflow ，远程名 `github`。
- GitCode： https://gitcode.com/arrongao/project-workflow ，远程名 `origin`。

取得当次发布授权并完成检查后，提交版本及 `releases/<tag>.md` 发行说明，创建对应版本标签。两个平台必须使用相同的提交、版本标签、发行标题及说明；不要分别生成内容不同的标签，也不要强制覆盖已发布标签。

依次向两个远程推送 main 和该版本标签，分别核对返回结果及远程标签解析后的提交 SHA。跨平台推送不是原子操作；一侧失败时保留已成功结果，修复后补齐另一侧，不通过删改已发布标签伪装成同时成功。

`.github/workflows/release.yml` 仅在版本标签推送时创建GitHub Release，从同一标签读取发行说明；使用GitHub提供的临时令牌，权限限定为仓库contents写入，无需个人访问令牌。发布后检查工作流结果、Release、标签对应提交及下载包，不能把推送成功当作Release已创建。

GitCode Release须通过其网页或官方API单独创建；SSH推送标签不会自动创建Release。目前GitHub工作流不负责GitCode发布，也不配置跨平台凭据。API授权凭据仅通过本地安全存储或环境注入，不写入仓库或发行说明。

发布完成条件：两个远程的main同步；对应版本标签指向同一提交；两个平台的Release均已发布且说明一致；下载内容包含正确版本、许可证和必需文件。若只有源码或标签同步，报告“Release待完成”，不能报告双平台发布成功。后续补充发布流程文档可以推进main，但不得移动已发布版本标签。
