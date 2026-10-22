# 安装到不同Agent

核对日期：2026-10-22。安装整个project-workflow目录，至少保留SKILL.md、references/、templates/及LICENSE。下载后不要多套一层目录。托管在GitHub、GitCode或本地不改变Skill格式；不同安装器支持的URL可能不同。

## 目录与调用

| Agent | 用户级位置 | 项目级位置 | 调用 |
|---|---|---|---|
| Codex | ~/.agents/skills/project-workflow/ | .agents/skills/project-workflow/ | $project-workflow，或宿主技能选择器 |
| Claude Code（本地） | ~/.claude/skills/project-workflow/ | .claude/skills/project-workflow/ | /project-workflow |
| DeepSeek Harness | 有效agentsHome/skills/project-workflow/（默认~/.agents/skills/）；或有效dshHome/skills/project-workflow/ | .agents/skills/project-workflow/或.dsh/skills/project-workflow/ | 宿主技能列表或显式指定名称 |
| 其他Agent | 查当前产品文档中的Agent Skills目录 | 查产品文档 | 不能假定支持上述路径或命令 |

~表示用户主目录。只选择一个作用域，避免同名副本产生冲突。Codex使用官方当前本地发现路径；已有.codex/skills安装不必在本次自动迁移，先确认实际来源。Claude云端/Cowork不等同本地Claude Code，不能靠复制本机目录完成云端安装。

## 获取指定版本

下载 [v0.0.1 源码 ZIP](https://github.com/arrongao/project-workflow/archive/refs/tags/v0.0.1.zip) 并解压，或执行：

```sh
git clone --branch v0.0.1 --depth 1 https://github.com/arrongao/project-workflow.git
cd project-workflow
```

然后按下面的复制步骤安装。版本标签用于固定内容；不要把下载目录的版本后缀带入最终 Skill 目录名。

## 从已下载仓库安装

以下在下载后的仓库根目录运行，目标已存在就停止；更新现有安装前先核对其来源和本地修改。不要用强制覆盖解决同名冲突。

PowerShell，默认安装给Codex。Claude Code将目标改为$HOME/.claude/skills/project-workflow；DSH先确认有效根目录。

```powershell
$skillSource = (Get-Location).Path
$skillTarget = Join-Path $HOME '.agents/skills/project-workflow'
if (!(Test-Path -LiteralPath (Join-Path $skillSource 'SKILL.md'))) { throw '请在Skill仓库根目录执行' }
if (Test-Path -LiteralPath $skillTarget) { throw '目标已存在，请先核对现有安装' }
New-Item -ItemType Directory -Path $skillTarget -Force | Out-Null
foreach ($entry in @('SKILL.md','LICENSE','README.md','INSTALL.md','PUBLISHING.md','references','templates','tests')) {
    Copy-Item -LiteralPath (Join-Path $skillSource $entry) -Destination $skillTarget -Recurse
}
```

macOS/Linux（Bash），同样从Skill仓库根目录执行：

```bash
set -eu
skill_source="$PWD"
skill_target="$HOME/.agents/skills/project-workflow"
test -f "$skill_source/SKILL.md"
if test -e "$skill_target" || test -L "$skill_target"; then
  printf '%s\n' '目标已存在，请先核对现有安装' >&2
  exit 1
fi
mkdir -p "$skill_target"
for entry in SKILL.md LICENSE README.md INSTALL.md PUBLISHING.md references templates tests; do
  cp -R "$skill_source/$entry" "$skill_target/"
done
```

这些命令仅复制文件，不安装依赖、不启动Agent。符号链接/junction适合维护单一源码，但是否跟随链接取决于宿主；更新源码会同时影响所有链接安装。默认提供复制方式，不自动更换已有链接。

## DeepSeek Harness专属说明

dshHome可由配置或DSH_HOME改变；agentsHome也可配置，不能宣称~/.agents永远生效。设置DSH_HOME只改变相应DSH根，不表示其他有效技能根全部失效。先核对当前安装的配置、技能来源路径与版本。

本条根据2026-10-22本地源码版本`d743267388641bc76f17c45ce8b4c231aed1d32c` 的 `packages/skill/skill-filesystem/src/index.ts`的discoverRoots相关实现（project-dsh、project-agents、user-dsh、user-agents）及packages/util/home-paths核对，不保证所有发行版完全相同。它是宿主说明，不是通用执行规则。

## 验证与更新

1. 检查目标根下直接存在SKILL.md，frontmatter中的name为project-workflow。
2. 在宿主技能列表确认名称和实际来源路径；未出现时按产品支持的方式刷新或开启新会话，不保证所有Agent即时刷新。
3. 用“只读检查，不修改”请求做一次试用，确认未产生意外文件、提交或外部操作。
4. 更新时比较安装版本及本地修改，再同步已选定版本；有多个Agent副本时分别核对，避免版本漂移。

## 依据与验证边界

- [Codex官方本地技能文档](https://learn.chatgpt.com/docs/build-skills)
- [Claude Code官方技能文档](https://code.claude.com/docs/en/skills)
- [Agent Skills标准](https://agentskills.io/specification)

Codex本机能识别现有Skill目录；Claude Code按官方文档适配，DSH按本地源码核对。本次未在Claude Code、DSH及其他Agent逐一执行安装/运行验收。
