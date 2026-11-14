# 安装、升级与落后检测

本文件是版本与升级机制的**执行规范**，供 Agent 在安装、升级或回答"哪些项目模板落后"时按需读取。设计背景与取舍见 [VERSIONING.md](../VERSIONING.md)。不要默认加载本文件；只在安装、升级、或用户询问版本/落后时读取。

纯 Markdown 加两个 JSON 文件，无运行时依赖：所有步骤都可用 Agent 已有的读写文件与计算哈希能力完成。

**哈希一律按 LF 归一化后计算。** 本文件所有 `sha256:` 基准与比较都要求先把内容里的 `\r\n` 换成 `\n` 再取哈希。原因：同一份模板在 Windows 上检出、被编辑器另存、或被带 `core.autocrlf` 的仓库检入后，工作区字节会变成 CRLF，直接按原始字节算哈希会与基准全部不符——于是每个文件都被误判成「用户改过」，升级退化为「一律不覆盖」。归一化只影响换行，不影响任何其他字节：真实的内容修改仍然会改变哈希，保护不丢失。

## 一、三个版本事实源

| 事实源 | 位置 | 谁写 | 含义 |
|---|---|---|---|
| `skill-version` | `SKILL.md` frontmatter 的 `metadata.version` | 维护者发布时 | skill 本体版本 |
| `template-version` | `templates/manifest.json` 的 `template_version` | 维护者改模板时 | 当前最新模板版本 |
| 项目标记 marker | 项目内被植入文件首行注释 | 植入/升级该项目的 Agent | 该项目**实际**使用的模板版本 |

**判定规则**：判断某项目模板是否落后，只比 `项目 marker` 与 `manifest.json 的 template_version`。registry 里缓存的版本不参与判定。

## 二、marker 格式（必须逐字匹配）

每个由本 skill 植入项目的文件，**第一行**写版本标记，使用该文件类型的原生注释语法：

- Markdown / HTML：`<!-- project-workflow: template-version=<版本> installed=<YYYY-MM-DD> -->`
- 其他按行注释的文件（如 `.gitattributes`、`.gitignore`）：`# project-workflow: template-version=<版本> installed=<YYYY-MM-DD>`

规则：

- 单行、独立成行、位于文件最前（先于任何标题或规则）。
- `<版本>` 与 `manifest.json` 的 `template_version` 取值一致。
- 读取时对两种语法都匹配，统一提取版本号与日期：`project-workflow:\s*template-version=([0-9A-Za-z._-]+)(?:\s+installed=(\d{4}-\d{2}-\d{2}))?`
- 一个项目内多个模板文件版本不一致时，取**最低**版本作为该项目版本，并在报告中列出不一致的文件。
- **无 marker 的老项目**：0.0.1 及以前植入的项目没有 marker。读不到 marker 但存在本 skill 同名模板文件（`AGENTS.md` 等）时，**按 `pre-marker` 版本处理**（基准见 `templates/manifest-history/pre-marker.json`），据此走 §七 的正常升级判定：文件哈希命中基准的视为未被改动、可替换；不命中的视为用户改过、不覆盖并报告。升级后为该文件补写当前 marker。
  - `pre-marker.json` 的每个文件值是**哈希数组**，收录该 tag 之前历史上出现过的所有模板内容，任一命中即视为未改动。这覆盖「项目植入的是更早的 4fd6d78/1900ab0 版本」的情况。
- **非植入物：`legacy`（模板祖先或自述文档）**。若项目中的同名文件哈希与任何历史基准**全部不符**，说明它从来不是本 skill 的产物。此时**不写 marker、不升级、不纳入模板管理**，只在 registry 记为 `legacy`。
  - 判为 `legacy` 需以下信号**至少两条**成立，并必须把证据写进报告：① 哈希不命中任何版本基准；② 目标文件体量与对应模板悬殊（本机实测：`AGENTS.md` 9–34KB 对模板 4.5KB，`BACKLOG.md` 20–293KB 对模板 678B，`README.md` 23–46KB 对模板 371B）；③ 文件的 Git 首次提交时间早于 skill 首次发布（`v0.0.1`，2026-10-22）。
  - 只成立一条时不下结论，报告「疑似非本模板植入，请确认」，交用户裁决。
  - **为什么必须拦住**：本 skill 的模板是从已有项目的治理文档抽象出来的，项目是模板的**祖先**而非后代。对祖先做「升级」等于用待填骨架清空它的实操细节，是本机制最严重的一类破坏。详见 [VERSIONING.md](../VERSIONING.md) §11。
  - `legacy` 项目此后每次被检测都只做「存在性提示」（告知当前模板版本、说明该项目不适用模板升级），不再提「是否升级」。

## 三、registry 格式

`registry.jsonl` 位于 skill 安装目录内（与 `SKILL.md` 同级），**不放进任何项目**。每行一个 JSON 对象：

```json
{"path":"D:/work/shop","template_version":"1.0","skill_version":"0.0.1","last_seen":"2026-11-14T12:00:00+08:00","source":"install"}
```

| 字段 | 说明 |
|---|---|
| `path` | 项目根绝对路径，正斜杠分隔 |
| `template_version` | 最后已知版本（缓存，不作判定依据）。`legacy` 项目填 `null`，因为它们从不属于任何模板版本 |
| `skill_version` | 写入该行时的 skill 版本 |
| `last_seen` | 最后确认时间，ISO 8601 含时区 |
| `source` | `install`（安装时登记）、`seen`（调用时刷新）或 `legacy`（确认为非本模板植入，不升级） |

写入规则：同一 `path` 只保留一行，刷新时整行替换。registry 是**索引**，不是真相；可在丢失后由已记录项目重建。

## 四、安装时（植入模板到某项目）

1. 按 `templates/manifest.json` 复制模板到项目，逐个文件在首行插入 marker，`template-version` 取 manifest 当前值。
2. 目标文件已存在时**停止并报告**，不覆盖；由用户决定合并方式（见 §六）。
3. 若 skill 安装目录尚无 `snapshot.json`，生成它（见 §五）。
4. 在 `registry.jsonl` 登记该项目：`source:"install"`。
5. 报告：写入的文件、marker 版本、registry 路径。

## 五、skill 本体的哈希快照（`snapshot.json`）

位于 skill 安装目录内，与 `SKILL.md` 同级，在**安装 skill 时**生成：

```json
{
  "skill_version": "0.0.1",
  "template_version": "1.0",
  "installed_at": "2026-11-14",
  "files": { "SKILL.md": "sha256:...", "references/upgrade.md": "sha256:..." }
}
```

计算方式（在 skill 安装目录执行，任选其一）。必须排除 `.git/`，否则快照会混入版本库内部对象而永远不匹配。**两条命令都先做 LF 归一化再取哈希**（见文首说明）：

```sh
# Bash
find . -type f ! -name snapshot.json ! -name registry.jsonl ! -name '*.log' \
  ! -path './.git/*' ! -path './.memsearch/*' \
  | sed 's|^\./||' | sort | while read -r f; do
      h=$(sed 's/\r$//' < "$f" | sha256sum | cut -d' ' -f1)
      printf '%s  %s\n' "$h" "$f"
    done
```

```powershell
# PowerShell
Get-ChildItem -Recurse -File |
  Where-Object { $_.Name -notin @('snapshot.json','registry.jsonl') -and $_.Extension -ne '.log' -and
                 $_.FullName -notmatch '[\\/]\.git[\\/]' -and $_.FullName -notmatch '[\\/]\.memsearch[\\/]' } |
  Sort-Object FullName | ForEach-Object {
    $bytes = [IO.File]::ReadAllBytes($_.FullName)
    $text = [Text.Encoding]::UTF8.GetString($bytes) -replace "`r`n", "`n"
    $sha = [Security.Cryptography.SHA256]::Create()
    $h = -join ($sha.ComputeHash([Text.Encoding]::UTF8.GetBytes($text)) | ForEach-Object { $_.ToString('x2') })
    "$h  $($_.FullName.Substring((Get-Location).Path.Length + 1).Replace('\','/'))"
  }
```

`files` 只收录 skill 本体，**不收录** `snapshot.json`、`registry.jsonl`，也不收录被复制进别的项目的模板实例。

## 六、升级 skill 本体

1. 读当前安装目录的 `snapshot.json`，重新计算现有文件哈希。
2. 逐文件比对：
   - 哈希一致 → 本文件未被本地修改，**可直接覆盖**为新版本。
   - 哈希不一致 → 本地已改，**不覆盖**，报告文件名与差异，由用户决定保留或合并。
   - 新版本新增的文件 → 直接写入。
   - 旧版本有、新版本已删的文件 → 不自动删除，报告后由用户确认。
3. 覆盖完成后重写 `snapshot.json`（新 `skill_version`、新哈希）。
4. 模板源（`templates/`）随本体升级一起带入，**不主动推送到任何项目**。项目模板升级只能由 §七 触发。

未改动是常态，可直接整体覆盖；只有本地 fork 过的文件需要人工裁决，因此不做完整三方合并。

## 七、升级某项目的模板实例

**只在用户确认后执行**，绝不静默覆盖。

1. 读项目文件 marker，得到项目当前版本。**读不到 marker 但存在同名模板文件时，当前版本按 `pre-marker` 处理**（基准见 `templates/manifest-history/pre-marker.json`）。
2. 目标版本取 `templates/manifest.json` 的 `template_version`。相同则报告"已是最新"并结束。
3. 取基准哈希：按项目当前版本读 `templates/manifest-history/<版本>.json`。该文件不存在则跳至第 6 步的安全降级。文件值可以是单个哈希字符串，也可以是哈希数组（`pre-marker` 用数组收录历史多个版本的内容）。
4. 对每个模板文件判定：
   - 项目文件哈希 **命中**该版本基准（值为哈希数组时任一命中即可） → 未被用户改动，可安全替换为新模板。
   - 不命中 → **用户改过**：不覆盖，列出差异，让用户选择「保留 / 用新版覆盖 / 逐段合并」。
5. 替换完成后更新（无 marker 的则补写）该文件 marker 的 `template-version` 与 `installed` 日期，并刷新 registry 对应行。
6. **安全降级**：当前版本对应的历史清单缺失，或项目全部同名文件哈希均不匹配该基准（疑似自制文件）时，不替换任何文件，只报告目标版本与建议的手工比对方式。
7. **`legacy` 项目不进入本流程**：一旦按 §二 判为 `legacy`，无论当前版本是什么都不做替换，只在报告中说明「该项目文档早于模板体系，不适用自动升级」。任何强制替换必须由用户在知悉内容将被覆盖后当次明确授权。

## 八、落后检测

### 场景 A：在某项目中调用本 skill（被动检测）

1. 读当前目录已有模板文件的 marker。
2. 判定项目归属：无 marker 时按 §二 检查同名文件哈希——命中历史基准的是 `pre-marker`（落后，提示升级）；全部不命中且满足 `legacy` 信号两条以上的是 `legacy`（不提示升级，只说明「本项目文档早于模板体系，当前模板为 `<Y>`，不适用自动升级」）。
3. 落后则一句话提示：「本项目模板为 `<X>`（无标记的显示为 `pre-marker`），当前最新 `<Y>`，是否升级？」不擅自升级。
4. 顺带把项目刷进 registry（`source` 按第 2 步判定填 `seen` 或 `legacy`，更新 `last_seen`）。**即使版本已最新也刷新**，让 registry 自增长，覆盖手工复制的项目。

### 场景 B：用户问"哪些项目模板落后"

1. 读 registry 得到候选路径。
2. 逐个读 marker：路径失效 → 记为 stale；`source:"legacy"` 或经 §二 判定为 `legacy` → 单列第三组；有 marker → 与最新值比较；无 marker 但哈希命中历史基准 → 记为 `pre-marker`（落后）。
3. 输出四组：落后、最新、`legacy`（文档早于模板体系，不参与升级）、stale（建议用户确认是否从 registry 移除）。
4. 明确说明：结果**仅覆盖 registry 中记录的项目**，registry 之外的项目需用户自行指定路径再扫。

## 九、边界

- **不做全局枚举**：被调用的 skill 只看得到当前工作目录，无法枚举整台机器。
- **registry 是索引，marker 是真相**：registry 缓存的版本可能因人工改动失真，不作判定依据。
- **不做后台主动推送**：所有提示在调用时或用户主动询问时触发。
- **不自动升级**：本项目模板的任何覆盖都必须有用户当次确认。
- 路径记录本机绝对路径，换机器或移动项目后需按 stale 处理。
- **不反向升级祖先**：文档早于模板体系的项目是模板的来源，不是模板的实例。对它们执行「升级」会以骨架覆盖实操内容，属禁止操作（见 §二 `legacy`）。
- **模板质量优先于覆盖率**：存量项目不纳入模板管理不是缺口，而是避免制造破坏。模板的改进应通过 `template_version` 递增与反哺来源项目的实践来推进。
