<h1 align="center">dsh-tray-restart</h1>

<p align="center">
  <em>复制一段提示词，把“重启 DeepSeek Harness”加入官方托盘菜单——只保留一个图标。</em>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-65a30d?style=flat" alt="MIT license"></a>
  <img src="https://img.shields.io/badge/platform-Windows-0078D4?style=flat&logo=windows&logoColor=fff" alt="Windows">
  <img src="https://img.shields.io/badge/distribution-prompt--only-8b5cf6?style=flat" alt="prompt only">
  <img src="https://img.shields.io/badge/runtime_dependencies-zero-brightgreen?style=flat" alt="zero runtime dependencies">
  <img src="https://img.shields.io/badge/tray_icons-one-0ea5e9?style=flat" alt="one tray icon">
</p>

<p align="center">
  <b>中文</b> · <a href="README.en.md">English</a>
</p>

---

## 它做什么

这不是 Cordis 插件，也不会再创建第二个托盘图标。你只需把下方提示词粘贴进正在使用的 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)，Harness 会先核查本机版本与桌面端结构，再把 **“重启 DeepSeek Harness”** 加入现有的官方 Electron 托盘菜单。

最终形态：

- **一个图标**：继续使用 DSH 原生托盘，不启动托盘助手。
- **原生重启**：调用 Electron 的 `app.relaunch()`，再走 DSH 已有的安全退出流程。
- **零运行时占用**：没有插件、PowerShell 常驻进程、flag、日志、注册表项或开机启动项。
- **可回滚**：修改前创建带哈希和版本信息的原始 `app.asar` 备份。
- **拒绝盲改**：找不到唯一代码锚点、无法验证归档或备份失败时，立即停止并报告，不强行写入。

> [!IMPORTANT]
> 这是面向 Windows 桌面版的社区方案，不是 DSH 官方扩展 API。DSH 升级通常会替换 `resources/app.asar`，升级后应重新运行提示词，而不是把旧版本备份覆盖到新版本。

## 快速开始

1. 等当前 Agent 回合结束并保存重要工作。
2. 在 DSH 桌面端新开一个会话，确保该会话有权读写 DSH 的安装目录。
3. 点击下面代码块右上角的复制按钮，把**完整提示词**粘贴并发送。
4. 阅读 Harness 给出的备份路径、哈希和验证结果；完成后手动完全退出并重新打开 DSH。
5. 右键官方托盘图标，确认只有一个图标且出现“重启 DeepSeek Harness”。

### 可直接复制的完整提示词

````text
你正在本机 Windows 桌面版 DeepSeek Harness 内工作。请把“重启 DeepSeek Harness”加入现有的官方系统托盘右键菜单，并完成备份、验证与回滚信息记录。

目标结果
- 托盘中只保留 DSH 官方 Electron 图标。
- 官方托盘菜单新增本地化项：中文为“重启 DeepSeek Harness”，英文为“Restart DeepSeek Harness”。
- 点击后调用 Electron 主进程的原生重启链路：`app.relaunch();`，随后调用项目现有的 `quitWithoutConfirmation();`。
- 实现位于桌面端主进程的官方 `DesktopTray` 路径，不创建任何 Cordis 插件、额外托盘、PowerShell/脚本常驻进程、服务、计划任务、注册表项或开机启动项。
- 不联网下载依赖，不读取或修改 API Key、Token、会话内容及任何无关 profile 配置。

不可妥协的安全边界
1. 先识别实际运行中的 DSH 桌面安装根目录、版本、`resources/app.asar` 路径；存在多个候选且无法唯一确认时停止并询问我。
2. 先检查当前实现。只有在官方托盘类/菜单构造与托盘实例创建处都找到唯一、相互对应的结构锚点时才能修改；不要仅凭全局字符串匹配 `app.relaunch()`。
3. “已安装”必须同时满足：对中英文两套消息表分别求值后，标签恰为“重启 DeepSeek Harness”**且**“Restart DeepSeek Harness”；菜单位置正确、只出现一次；调用同一托盘实例的唯一 restart 回调，且回调顺序正确。若全部满足，只对当前归档做只读验证并报告“已安装”；**跳过步骤 3–6，不创建任何临时/备份/staged 文件、不重建、不替换正式文件**。
4. 确需修改时，必须先创建原始 `app.asar` 的完整备份。备份放到 `$DSH_HOME/backups/dsh-tray-restart/<时间戳>-<随机后缀>/`；未设置 `DSH_HOME` 时使用 `~/.dsh/backups/dsh-tray-restart/<时间戳>-<随机后缀>/`。目录必须是新建且原先不存在，禁止覆盖旧备份。同时写入 `manifest.json`，至少记录：DSH 版本、安装路径、创建时间、原始/备份/修改后 app.asar SHA-256、修改前后 `lib/main.js` SHA-256、目标文件路径、补丁契约版本和本次修改说明。原始与备份哈希不同则立即停止。
5. 所有修改先在与正式文件同卷的全新临时目录或 staged 文件中完成并验证。验证通过后才替换正式 `app.asar`；任何失败都保持正式文件不变。
6. 仅在步骤 2 判定确需修改后，才证明本机已有的 ASAR 读写方式能正确重建当前格式及其 integrity 元数据：对原件副本做一次零内容改动的 round-trip，逐条验证内容和语义元数据；失败或没有可靠写入能力时停止。禁止为此联网下载或全局安装包。
7. 以只读方式检查当前 Electron/可执行文件是否启用了会拒绝修改后归档的 embedded-ASAR integrity、fuse 或签名策略。若进入修改分支且无法用本机已有的正式能力生成可接受的归档，停止；不要绕过或关闭安全策略。
8. 保留归档中除目标 `lib/main.js` 内容及其必需 integrity/尺寸/偏移更新之外的所有文件、路径、内容、unpacked 标记、可执行标记、符号链接和语义元数据；不要改动 `app.asar.unpacked`。每个条目的实际字节都必须与其 integrity hash/blocks 一致。
9. 保留 `lib/main.js` 中所有无关的本机改动。源代码差异只能落在下面补丁契约定义的两个允许区域：官方托盘菜单项区域、同一托盘实例的 restart options 区域。全新安装通常改两个区域；部分安装只能改实际缺失/错误的一个或两个区域。禁止整文件格式化、换行归一化或从旧备份重建。
10. 不自动关闭或重启当前 DSH。完成落盘与静态验证后报告“需要手动完全退出并重开”，由我决定何时重启。
11. 若发现旧的 `dsh-tray-restart` Cordis 插件、profile insert、node_modules 链接或额外托盘助手，只列出精确路径/进程并询问我是否删除；未经确认不要删除其他用户数据或第三方组件。

执行步骤与完成标准

步骤 1：建立只读基线
- 找到实际桌面 exe、版本与 `resources/app.asar`。
- 从当前归档只读提取 `lib/main.js`，定位官方 `DesktopTray`（或当前版本等价实现）的菜单构造函数，以及创建该托盘实例并传入 `open`/`quit` 回调的位置。
- 记录两处结构锚点及其上下文。每处必须唯一；否则停止。
- 只读识别归档使用的 integrity 字段，并检查桌面 exe 的 Electron embedded-ASAR integrity/fuse/签名策略；本步骤不得创建临时副本或重建归档。
完成标准：目标安装、版本、归档和两个锚点均唯一；当前归档内容/integrity 可只读验证，运行时安全策略已有只读证据。

步骤 2：判定状态并选择分支
- 在官方托盘菜单上下文中检查是否已有且仅有一次 `this.options.restart()`；分别用中文、英文消息表求值标签，结果必须**同时**恰为“重启 DeepSeek Harness”和“Restart DeepSeek Harness”，且位置在分隔线后与退出项前。
- 在同一个官方托盘实例的 options 中检查是否已有且仅有一次 `restart: () => { ... }`，并确认它执行 `app.relaunch()` 与 `quitWithoutConfirmation()`。
- 两处及两种语言的标签/位置都正确：状态为“已安装”。只验证当前归档并直接转到步骤 7；明确跳过步骤 3–6，不创建临时文件，零文件写入、零备份、零重建。
- 仅一处存在或任一语言标签不完整：把它视为部分安装，只补齐/修正缺失部分，并继续步骤 3。
- 出现重复项、歧义实现或不同重启语义：停止并报告差异，不叠加第三套逻辑。
完成标准：状态明确为未安装、部分安装或已安装，并已锁定只读分支或修改分支。

步骤 3：能力证明与备份（仅修改分支）
- 识别本机现有的 ASAR 读写能力；在全新临时目录中的原件副本上完成零内容改动 round-trip，证明所有条目内容与语义元数据等价、全部 integrity hash/blocks 可由实际字节重新验证。失败或运行时策略不允许修改归档时停止并删除临时产物，不触碰正式文件。
- 计算正式 `app.asar` 的 SHA-256。
- 创建上述带随机后缀的全新备份目录；若目录已存在立即停止，禁止复用或覆盖。复制原始归档，重新计算备份 SHA-256，并先写入 manifest 的基线字段。
完成标准：ASAR 写入能力和运行时策略门通过；原始与备份 SHA-256 完全相同；备份可重新读取并成功提取 `lib/main.js`。

步骤 4：按补丁契约最小修改
A. 在官方托盘右键菜单中，把重启项放在分隔线之后、退出项之前。沿用当前文件的缩进与代码风格，语义应为：

~~~javascript
{
  label: (/^打开/.test(messages.openApplication) ? '重启' : 'Restart')
    + String(messages.openApplication).replace(/^\S+/, ''),
  click: () => {
    this.options.restart();
  }
},
~~~

只有当当前版本的正式本地化字段求值后已经包含完整产品名（中文恰为“重启 DeepSeek Harness”、英文恰为“Restart DeepSeek Harness”）时，才可直接复用。`messages.restartApplication` 若仅为“重启”/“Restart”，不能单独作为标签；此时使用上面的 `openApplication` 派生方式或当前版本等价的完整本地化组合。修改后必须对中英文两套消息表验证最终完整标签。

B. 在创建同一个官方托盘实例时，为 options 增加：

~~~javascript
restart: () => {
  if (quitting) return;
  app.relaunch();
  quitWithoutConfirmation();
}
~~~

若当前上下文没有 `quitting` 状态，不要凭空创建不可靠变量；保留核心顺序 `app.relaunch();` → `quitWithoutConfirmation();`，并在报告中说明缺少防重入守卫。若当前 DSH 已有等价且可证明安全的 restart 回调，复用它而不是新增重复实现。

完成标准：只修改官方托盘菜单项区域和同一托盘实例的 restart options 区域；全新安装通常改两个区域，部分安装只改实际缺失/错误的区域；菜单项和回调各恰好一个；中英文完整标签与位置正确；没有任何额外进程方案。

步骤 5：验证 staged 产物（仅修改分支）
- 对修改后的 `lib/main.js` 做 JavaScript 语法检查。
- 对修改前后的 `lib/main.js` 做逐行源代码 diff：每一条新增、删除或替换行都必须属于“官方托盘菜单项”或“同一托盘实例 restart options”两个允许区域之一；全新安装预期覆盖两个区域，部分安装允许只覆盖实际修正的一个区域。不要用 unified-diff 的 hunk 数量作为判据；不得出现整文件格式化、换行/编码归一化或其他代码差异。
- 重新打开 staged ASAR 并再次提取 `lib/main.js`；确认完整中英文标签、菜单位置、菜单调用、restart 回调和两条核心调用都在正确上下文中。
- 比较原始与 staged 的完整归档：除 `lib/main.js` 内容及由其尺寸引起的物理 offset 变化外，所有路径、类型、内容哈希、unpacked/可执行标记、链接目标及其他语义元数据相同；`app.asar.unpacked` 未改变。
- 对 staged ASAR 的**每个条目**从实际 payload 重新计算并核对 integrity algorithm/hash/blockSize/blocks；目标 `lib/main.js` 的 integrity 必须对应新字节，非目标条目的完整性值与语义保持不变。
- 检查 staged 归档头、文件偏移和所有条目均可读取；再次确认 Electron embedded-ASAR integrity/fuse/签名策略不会拒绝该 staged 产物。若只能证明“可解析”而不能证明“可被当前 Electron 接受”，停止应用。
- 检查幂等性：用步骤 2 的完整检测逻辑再跑一次，必须分别求值并验证中英文完整标签、位置、唯一调用与回调，判定“已安装”，且不产生新的文件写入、临时产物或重建。
完成标准：语法、允许区域内的最小源码差异、全归档语义、逐条 integrity、运行时策略与幂等性全部通过。

步骤 6：应用（仅修改分支）
- 在安全替换的紧邻前一刻重新计算正式 `app.asar` 与备份的 SHA-256；它们必须分别仍等于步骤 3 的原始哈希。若任一变化，停止，保留 staged 与备份，不覆盖。
- 采用同卷安全替换方式写入验证后的归档，不留下半写文件。
- 重新读取正式归档并重复关键验证；确认正式文件 SHA-256 等于 staged；补全 `manifest.json` 中修改后 SHA-256、修改前后 `lib/main.js` SHA-256、integrity/运行时策略与验证结果。
完成标准：正式文件等于已验证的 staged 归档，备份与 manifest 完整。

步骤 7：给出最终报告
请逐项报告：
- DSH 版本与安装路径
- 正式 `app.asar` 路径
- 状态：新安装 / 修复部分安装 / 原本已安装
- 两个结构锚点、完整中英文标签、菜单位置及每项出现次数
- 只读 integrity 与 Electron fuse/签名策略检查结果；修改分支另报告 ASAR 读写能力与零改动 round-trip，已安装分支明确写“不适用，未创建临时产物”
- 修改分支的 JavaScript 语法、允许区域源码 diff、ASAR 全量语义差异与 `app.asar.unpacked` 检查结果；已安装分支明确写“不适用”
- 修改分支：备份目录、manifest 路径，以及原始/备份/修改后 SHA-256；已安装只读分支：明确写“不适用，零文件写入”
- 是否发现旧插件/第二托盘残留（只报告，不擅自删除）
- 仅修改分支提示：现在需要我手动完全退出并重新打开 DSH，再进行托盘实测；已安装只读分支不要求为本次检查重启
- 回滚条件：仅当离线替换前当前 `app.asar` 哈希再次等于 manifest 的“修改后哈希”时，才可直接恢复该备份；若 DSH 已升级或哈希不同，禁止用旧备份覆盖

最终验收条件
- 右键 DSH 官方托盘图标可见“重启 DeepSeek Harness”或“Restart DeepSeek Harness”。
- 系统通知区域只有一个 DSH 托盘图标。
- 保存工作后点击重启，桌面端退出并由 Electron 自动重新拉起。
- 不存在为本功能新增的 Cordis 插件、PowerShell 常驻助手、flag/log、服务、计划任务、注册表或开机启动项。
- 任一步骤的安全条件不成立时，保持正式文件不变并清楚报告阻塞点；不要为了“完成”而降级安全边界。
````

## 预期结果

完成并重新打开 DSH 后：

| 检查项 | 预期 |
| --- | --- |
| 托盘图标 | 只有官方 DSH 图标 |
| 右键菜单 | 打开 / 重启 / 退出（以及当前版本已有项目） |
| 重启机制 | Electron `app.relaunch()` + DSH 退出流程 |
| 常驻进程 | 不新增 |
| profile 插件 | 不新增 |
| 补丁流程网络与凭据 | 不额外下载、不发起额外网络请求、不读取凭据；Harness 正常模型通信不变 |
| 更新后 | 可能需重新运行提示词 |

重启会中断正在执行的任务，语义等同于手动完全退出再打开。请在 Agent 空闲时测试。

## 为什么不用插件

DSH 桌面端的官方托盘属于 Electron 主进程；Cordis 插件运行在独立的纯 Node 宿主中，不能直接调用主进程的 `Tray`/`Menu` API。用插件绕过该边界只能再启动一个托盘助手，于是出现两个图标和额外生命周期管理。

本项目选择单一所有者：

```text
Electron 主进程
└─ 官方 DesktopTray
   ├─ 打开
   ├─ 重启  ← 本提示词添加
   └─ 退出
```

它把运行时占用降到零，但代价是 DSH 更新可能覆盖构建产物。详细取舍见 [设计说明](docs/DESIGN.md)。

## 兼容性

- 平台：Windows 10 / 11 桌面版。
- 结构观察基线：DeepSeek Harness `0.2.0-rc.2` 的官方托盘锚点、标签资源与 ASAR integrity 元数据已检查；这不等于承诺任意机器都具备安全的 ASAR 写入能力。
- 所有版本（包括 `0.2.0-rc.2`）：必须现场通过唯一锚点、当前归档的只读逐条 integrity 与 Electron 运行时策略门；若进入修改分支，还必须通过零改动 ASAR round-trip 和 writer 能力门。任一适用能力缺失即安全停止。
- `dsh web`：没有 Electron 系统托盘，不适用。
- DSH 更新：重新执行提示词；不要把旧版备份覆盖到新版安装。

## 验证与回滚

- 完整验收清单：[docs/VERIFICATION.md](docs/VERIFICATION.md)
- 安全回滚规则：[docs/ROLLBACK.md](docs/ROLLBACK.md)
- 架构与风险边界：[docs/DESIGN.md](docs/DESIGN.md)

最重要的回滚规则：**只有当前 `app.asar` 的 SHA-256 仍等于 manifest 中记录的“修改后哈希”，才能直接恢复对应备份。** 哈希不同通常意味着 DSH 已升级或又被修改，此时旧备份不能直接覆盖。

## 隐私与权限

- 提示词只需要读取 DSH 版本、安装路径与桌面主进程归档。
- 不需要 API Key、Token，也不读取会话内容。
- 补丁流程不额外下载依赖或发起额外网络请求；Harness 自身正常的模型通信不属于补丁行为。
- 唯一持久变更是当前版本的 `resources/app.asar`；备份与 manifest 存放在用户的 DSH home 下。
- 修改应用安装文件可能触发安全软件或完整性检查；请保留验证报告与哈希。

## 项目结构

```text
dsh-tray-restart/
├─ README.md              # 中文说明与唯一权威安装提示词
├─ README.en.md           # English overview
├─ docs/
│  ├─ DESIGN.md           # 架构、约束与更新模型
│  ├─ VERIFICATION.md     # 手动与静态验收清单
│  └─ ROLLBACK.md         # 哈希门控的安全回滚
├─ CHANGELOG.md
├─ CONTRIBUTING.md
├─ SECURITY.md
└─ LICENSE
```

仓库没有 `package.json`、安装脚本或可执行代码——**README 中的提示词就是发布物**。

## 常见问题

### 为什么升级 DSH 后菜单消失？

更新器替换了 `app.asar`。这是预期行为；在新版本上重新运行提示词，让 Harness 重新探测结构并创建新备份。

### 能否把备份直接复制回去？

只有哈希门控通过时可以。若当前归档不是 manifest 记录的修改后哈希，按 [回滚文档](docs/ROLLBACK.md) 处理，避免把新版本降回旧构建。

### 为什么不提供一键 PowerShell 脚本？

固定脚本很容易把版本差异当成文本差异并盲改。由 Harness 先检查本机结构、唯一锚点、完整归档和哈希，再做最小修改，更适合不同 DSH 版本。

### 需要管理员权限吗？

取决于 DSH 的安装目录权限。不要以管理员身份重开整个工作流；优先只为目标安装目录授予必要写权限。

## 致谢

README 的工程化呈现参考了 [dsh-auto-continue](https://github.com/HsiangNianian/dsh-auto-continue)。

## License

[![MIT](https://img.shields.io/badge/license-MIT-65a30d)](LICENSE)

MIT © 2026 CAOGGL
