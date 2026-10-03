# 安全回滚

回滚的核心不是“找到一个旧 `app.asar` 覆盖回去”，而是证明**当前文件正是那次补丁产生的文件**，并在 DSH 完全退出后再次验证这一事实。证明依据是备份目录中的 `manifest.json`。

> [!IMPORTANT]
> DSH 内的 Agent 不能在桌面端退出后继续执行。因此，直接恢复分为两个阶段：Harness 内只读生成并审阅计划；用户退出 DSH 后，在普通 Windows PowerShell 中手动执行一次性离线恢复块。该命令块不是常驻助手，不创建服务、计划任务、注册表项或开机启动项。

## Manifest 最低字段

```json
{
  "dshVersion": "<实际版本>",
  "installRoot": "C:\\path\\to\\DeepSeek Harness",
  "targetPath": "C:\\path\\to\\resources\\app.asar",
  "createdAt": "<RFC 3339 时间>",
  "originalSha256": "...",
  "backupSha256": "...",
  "patchedSha256": "...",
  "mainJsOriginalSha256": "...",
  "mainJsPatchedSha256": "...",
  "patchContractVersion": "1",
  "change": "Add restart item to the official DesktopTray menu"
}
```

`originalSha256` 与 `backupSha256` 必须相同。路径必须是本机实际探测结果，文档中的路径只是占位示例。

## 哈希决策表

先只读计算当前正式 `app.asar` 的 SHA-256，再按表处理：

| 当前哈希 | 含义 | 动作 |
| --- | --- | --- |
| 等于 `patchedSha256` | 当前看起来仍是该次补丁产物 | 可以生成离线恢复计划；**执行前仍须再次检查** |
| 等于 `originalSha256` | 已回滚 | 不做任何写入 |
| 两者都不等 | DSH 已更新或文件又被修改 | 禁止用该旧备份覆盖，也禁止按相似字符串自动删代码 |
| manifest 缺失/字段不全 | 无法证明对应关系 | 禁止直接覆盖 |

## 两阶段直接恢复

仅当初次检查满足以下全部条件时，Harness 才能生成离线恢复块：

- 当前哈希等于唯一候选 manifest 的 `patchedSha256`；
- manifest 的 DSH 版本和 `targetPath` 与当前安装完全一致；
- 备份 SHA-256 同时等于 `backupSha256` 和 `originalSha256`；
- 先用“当前 live 哈希等于 `patchedSha256` + DSH 版本一致 + `targetPath` 一致”筛选 manifest，筛选结果恰好一份；同一路径下其他版本或其他 `patchedSha256` 的历史 manifest 不构成冲突；
- 用户已经看过目标、备份、三个归档哈希和影响范围。

### 阶段 A：Harness 内只读准备

1. 不退出 DSH，不写正式归档。
2. 展示唯一 `targetPath`、备份路径、exe 路径、DSH 版本和全部预期哈希。
3. 生成一段**完全展开字面量、无需继续依赖 Agent**的 Windows PowerShell 命令块，但不要代替用户执行。
4. 命令块必须使用全新随机后缀的 staged 与取证文件名；若任一目标已存在则停止，禁止覆盖。
5. 命令块必须包含阶段 B 的每个检查，尤其是紧邻替换前的第二次 live hash 检查。

### 阶段 B：用户在 DSH 外手动执行

1. 保存工作并完全退出 DSH，确认相关桌面进程已结束。
2. 打开普通 Windows PowerShell；不要从 DSH Agent 工具中运行。
3. 粘贴阶段 A 生成的一次性命令块。该命令块必须按顺序：
   - 设置 `$ErrorActionPreference = 'Stop'`；
   - 拒绝仍在运行且可执行路径等于目标安装的 DSH 桌面进程；无法可靠判定时停止；
   - 验证 live target、备份与 manifest 路径均仍是阶段 A 的唯一字面量路径；
   - 重新计算备份哈希，并验证它同时等于 `backupSha256` 与 `originalSha256`；
   - 把备份复制为与 target 同卷的全新 staged restore，并验证 staged 哈希等于 `originalSha256`；
   - **在安全替换的紧邻前一刻**重新计算 live target 哈希；若不等于 `patchedSha256`，立即停止且不覆盖；
   - 使用同卷原子替换能力（例如 `.NET File.Replace`），同时把刚被替换的 patched 归档保存到全新取证路径；
   - 重读正式文件并确认 SHA-256 等于 `originalSha256`；否则报告失败，不删除取证副本。
4. 命令成功后，由用户手动启动 DSH。
5. 确认官方托盘仍工作且本项目添加的“重启”项已移除。

任何检查失败都必须停止。不要用 `Copy-Item -Force` 直接覆盖 live target，也不要在哈希检查与替换之间加入等待、确认提示或其他长操作。

## 当前哈希不匹配时

不要恢复旧备份，也不要仅凭 `this.options.restart()`、标签文本或两段相似代码自动做“手术式删除”。这些结构可能已经由 DSH 新版本正式提供，或被其他本机修改共同拥有。

安全选项只有：

- 使用 DSH 官方安装器修复/重装**当前版本**；或
- 升级到目标版本后重新评估是否仍需本项目；或
- 单独进行专家级只读归属审查。只有 manifest 记录的修改前后 `lib/main.js` 哈希、精确补丁 hunk 指纹、当前版本和唯一结构上下文都能证明归属时，才可另行提出最小删除计划；该计划仍需用户再次确认，不能由本回滚流程自动执行。

旧备份是版本绑定的恢复材料，不是通用安装包。

## 可复制的回滚评估提示词

```text
请只读检查当前 DeepSeek Harness 桌面安装与 `$DSH_HOME/backups/dsh-tray-restart/` 下的备份 manifest。列出当前 app.asar SHA-256、候选 manifest 的 original/backup/patched SHA-256、DSH 版本、唯一 targetPath、备份路径和桌面 exe 路径。

先以“当前哈希等于 patchedSha256 + 版本一致 + targetPath 一致”筛选 manifest；只有筛选结果恰好一份，且对应备份哈希同时等于 backupSha256 与 originalSha256 时，才可生成直接恢复计划。同一路径下其他版本或其他 patchedSha256 的历史 manifest 不计为候选冲突。若筛选后候选不唯一、哈希不匹配、版本不同或 manifest 不完整，禁止覆盖和自动删代码；建议用官方安装器修复当前版本，并说明原因。

如果门控全部通过，请不要在当前 DSH 会话中退出或替换文件。先向我展示路径、版本、三个归档哈希和影响范围，然后生成一段可由我在完全退出 DSH 后，于普通 Windows PowerShell 中手动执行的一次性离线恢复命令块。命令块必须：使用全新 staged/取证路径；拒绝目标 DSH 进程仍在运行；复验备份哈希；验证 staged；在原子替换的紧邻前一刻重新验证 live target 哈希仍等于 patchedSha256；用同卷原子替换并保留被替换文件；最后验证 live target 哈希等于 originalSha256。任一检查失败立即停止。不要创建常驻进程、服务、计划任务、注册表项或启动项，不要删除会话、profile、插件或其他用户数据。
```

该提示词只负责只读判定与生成离线命令。真正替换由用户退出 DSH 后在外部 PowerShell 中手动执行。