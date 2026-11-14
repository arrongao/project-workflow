# 通用版检查记录

## 2026-11-14：两套版本体系（template 1.0 → 1.1）

范围：`VERSIONING.md`、`references/upgrade.md`、`templates/manifest.json` 与 `templates/manifest-history/`、marker 植入、`legacy` 判定。含真实项目实测，不含独立模型前向评估。

- 4 个 JSON 合法：`manifest.json`、`manifest-history/{1.0,1.1,pre-marker}.json`。
- `manifest.json` 的 5 个 sha256 与 `templates/` 实际文件逐条匹配；5 个 marker 全部可被统一正则提取且值均为 `1.1`。
- history 三条基准：`pre-marker.json` 16 条（哈希数组）、`1.0.json` 5 条、`1.1.json` 5 条；提升版本前已校验 `1.0.json` 与旧 manifest 的 `files` 完全一致。
- `legacy` 判定实测（本机 5 个既有项目，名称不在此列出）：4 个 Git 项目各 3/3 信号 → `legacy`；1 个非 Git 项目 2/3（无年代信号，其余两条成立）→ `legacy`。
- 反向实测（临时目录，防误判）：v0.0.1 原样植入 → `pre-marker`（5/5 命中，可升级）；植入后改 `AGENTS.md` → `pre-marker`（4/5）；模板内容删去 marker 行 → 疑似非模板植入（1/3，交用户确认）；自制文档体量相当 → 疑似非模板植入（1/3）；空目录 → 不涉及。无误判为 `legacy`。
- 17 个 Markdown 文件本地链接检查：0 断链。
- `registry.jsonl` 5 行，路径均存在；`snapshot.json`、`registry.jsonl` 已被 `.gitignore` 排除。
- `git diff --check`：通过。
- 未提交、未打 tag、未推送。

## 2026-10-22

日期：2026-10-22。范围：2.0.0通用化改动，非全Agent认证。

- Codex skill-creator quick_validate.py：Skill is valid!（源码与临时安装副本均通过）。
- git diff --check：通过。
- INSTALL.md中的PowerShell块在仓库根执行，目标仅替换为临时目录：复制成功，templates/.gitattributes存在；重复执行被“目标已存在”保护中止。
- 通用主说明、references及templates没有DSH_HOME/agentsHome/dshHome产品专属配置。
- Markdown本地链接检查：9个本地链接，0个断链。
- tests/scenarios.md是待执行的行为评估清单，不声明已经通过；未进行独立模型前向评估。
- Bash示例未执行；Claude Code及DSH未进行安装/运行验收。Codex现有junction继续指向本仓库，改动已反映到安装路径。
- 未创建GitHub仓库、tag或release，未push；未审计全部历史的公开适宜性。MIT已添加，版权署名arrongao；发布前由权利人核实。