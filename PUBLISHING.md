# 发布前检查

1. 确认本仓库原创内容及可分发权利；LICENSE署名当前为arrongao。若权利归组织或有其他权利人，先调整相应声明。外部来源不能仅靠新增MIT而重新许可。
2. 检查将公开的文件及Git历史，避免暴露项目私有材料、身份信息或凭据；本次通用化不重写历史。
3. 运行可用的Skill格式检查，并按tests/scenarios.md进行需要的行为评估；明确哪些Agent只是文档适配、哪些已实测。
4. 审阅Git diff、MIT文本及安装命令；选择版本/tag。首个公开版本为v0.0.1；metadata、tag及发行说明的版本必须一致。
5. 用户授权后再创建GitHub仓库或push；确认真实URL后更新下载链接。公开仓库本身不替代LICENSE。

MIT允许商用、修改和再分发，要求保留版权及许可声明，并按原文提供免责条款；不要求下游整体开源。完整法律条款以LICENSE为准。

依据：[OSI MIT License](https://opensource.org/license/mit)。本文件不附加额外许可限制。

## 标签发布

GitHub仓库：https://github.com/arrongao/project-workflow 。保留GitCode的origin远程；GitHub远程名为github。

取得当次发布授权并完成检查后，提交版本及 `releases/<tag>.md` 发行说明，创建对应版本标签，再推送main和该标签到github。不要强制覆盖已发布标签。

`.github/workflows/release.yml` 仅在版本标签推送时创建GitHub Release，从同一标签读取发行说明；使用GitHub提供的临时令牌，权限限定为仓库contents写入，无需个人访问令牌。发布后检查工作流结果、Release、标签对应提交及下载包，不能把推送成功当作Release已创建。
