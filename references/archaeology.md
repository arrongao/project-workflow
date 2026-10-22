# 按需分析项目历史

仅在需要查明重复故障、状态漂移或规则依据时使用。先限定模块和时间；短历史、浅克隆、复制代码或squash记录不足时说明限制，不硬凑结论。

跨Shell可直接使用的只读Git命令：

```text
git status --short
git log -n 20 --oneline -- path/to/module
git log -n 20 --format="%h %ad %s" --date=short -- path/to/module
git show --stat COMMIT
git show COMMIT -- path/to/file
git log -Ssymbol --oneline -- path/to/module
```

path/to/module、COMMIT、symbol为示例，执行前替换为目标真实路径和标识。命令参数按当前Shell正确引用；不依赖grep/sed/awk管道。发现具体问题后再增加读取范围，不默认扫描数百个完整diff。

文件修改频次或fix标题只是线索，不足以证明重复根因。阅读对应diff、问题和验证记录，确认触发条件及机制是否相同。不同问题碰巧修改同一文件不应形成一条通用禁令。

对照文档时区分：声明所指版本、当前版本、实际实现和验收状态。存在代码不意味着需求已完成，记录过命令也不意味着当前版本已验证。

输出：发现、涉及版本/文件、证据、影响、置信度、建议规则及适用范围。没有证据就说明未证实，不按提交数评价团队或作者。已有足够证据回答任务时停止。
