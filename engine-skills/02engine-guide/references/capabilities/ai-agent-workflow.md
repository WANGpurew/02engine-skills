<!-- capability_id: cap.e2.agent-workflow | revision: 1 | status: active -->

# 02Agent 小步改项目与 Skill bridge

## R — 原文 (Reading)

> "对复杂项目，推荐按以下方式使用 02Agent：先让 02Agent 查看项目概览。只读取将要编辑的舞台或角色脚本文件。搜索需要使用的积木或扩展。对不熟悉的 opcode 调用积木帮助工具。每次只修改一个较小的脚本或资源范围。修改后运行诊断，确认生成的 Scratch JS DSL 可以同步回积木。……不要要求 02Agent 一次性重写整个大型项目。"
>
> 中文要点：改项目的推荐顺序是"概览→只读目标脚本→搜积木/扩展→getBlockHelp 查生疏 opcode→小步改→getDiagnostics 确认 DSL 同步"；拆成"建角色→加变量→写主脚本→补视觉"多步，禁止一次性重写大项目。
>
> — 02Engine 功能文档·02agent-addon / slug: 02agent-addon（原页中文；源：candidates/case.md c10、framework.md f16、counter-example.md ce26）

> "02Agent 可以在需要扩展积木时先搜索再安装扩展。例如需要画笔积木时，AI 应先搜索 pen，安装内置 Pen 扩展，再调用 searchBlocks 或 getBlockHelp 确认 pen.* 积木语法。……外部直接 URL 默认不会被接受。只有用户明确提供可信 URL，并且工具调用显式设置 allowExternalUrl: true 时，02Agent 才应尝试安装任意外部 URL。"
>
> 中文要点：需要某类积木时走"先搜后装再验语法"——searchExtensions→installExtension→searchBlocks/getBlockHelp；远程扩展走 unsandboxed runner，外部 URL 默认拒绝，须用户明确提供可信 URL 且工具显式 `allowExternalUrl: true`，不让 AI 自动装无关扩展。
>
> — 02Engine 功能文档·02agent-addon / slug: 02agent-addon（原页中文，行820-841；源：candidates/case.md c11、framework.md f17、counter-example.md ce23）

> "ws://127.0.0.1:40202/agent?token=<token>" ... "http://127.0.0.1:40202" ... "GET /health /manifest /tools /prompts ... POST /call /reloadManifest" ... "node src/client.js call searchBlocks --stdin" ... "--base64 <base64Json>"
>
> 中文要点：02Agent 内置浏览器侧 WebSocket 客户端连本机 bridge（WS 默认 ws://127.0.0.1:40202/agent?token=<token>，HTTP 默认 http://127.0.0.1:40202），本地接口 GET /health、/manifest、/tools、/prompts 与 POST /call、/reloadManifest；本地 bridge server 只应监听 127.0.0.1，不得暴露到公网或局域网；Windows 下含中文/非 ASCII 参数用 UTF-8 stdin（`--stdin`）或 base64（`--base64`）传参，避免 argv 编码损坏。
>
> — 02Engine 功能文档·02agent-addon / slug: 02agent-addon（原页中文，英文 URL/API 串逐字见行850/857/889-899/910/917；源：framework.md f18、principle.md p21/p22、counter-example.md ce24/ce25）

## I — 方法论骨架 (Interpretation)

02Agent 是编辑器内 AI 助手（基于 Gandi IDE AI assistant 二开），目标是"先检查→小范围修改→诊断确认"的项目内助手，而不是替代编辑器。
① **工作流**：看项目概览→只读将编辑的舞台/角色脚本→searchBlocks/getBlockHelp 搜积木与查生疏 opcode→每次只改一个小范围→getDiagnostics 确认 DSL 能同步回积木；拆成建角色→加变量→写主脚本→补视觉，禁止一次性重写大项目；
② **虚拟文件与 DSL**：项目映射为虚拟文件（`/stage.js`、`/sprites/<名>.js`、造型 SVG、`/docs/scratch-agent.md` DSL 指南、`/docs/block-catalog.md` 积木目录），脚本用 Scratch JS DSL（`// @script` 定位顶层脚本、`// @script <id>` 与 `// blockId` 注释利于同步），改后同步回 VM；
③ **工具集四类**：项目/文件（getProjectOverview/listFiles/readFile/searchFiles/getDiagnostics）、积木与 DSL（getScratchGuide/searchBlocks/getBlockHelp）、编辑（applyPatch、createSpriteWithSvg、addCostumeWithSvg 等）、扩展（searchExtensions/installExtension）；改状态工具串行排队，避免聊天与 bridge 冲突；
④ **扩展搜装**：searchExtensions→installExtension→searchBlocks/getBlockHelp 验证语法；外部 URL 默认拒绝、须用户明确可信 URL 且显式 `allowExternalUrl: true`；不让 AI 自动装无关扩展；
⑤ **Skill bridge**：浏览器侧出站 WS 连本机 bridge（127.0.0.1:40202）→HTTP /call 调用工具；manifest 公开协议/工具 schema/提示/项目概览/安全声明，不公开 API Key、模型密钥、完整 localStorage 配置；bridge 只监听本机、中文参数用 stdin/base64；
⑥ **安全边界**：AI 可增删改角色/脚本/造型/变量；远程扩展可能跑非沙箱 JS；bridge 允许本机其它进程调用；重要项目先存副本再大改。
核心洞察：这是"带诊断闭环的小步编辑"——每一步都能被 getDiagnostics 验证是否同步回积木，且所有越权路径（外部 URL、bridge 暴露网络）都被默认拒绝或限制在本机。

## A1 — 书中的应用 (Past Application)

### 案例一：02Agent 小步改项目的推荐工作流（scenario，slug: 02agent-addon / c10）
- **问题**：复杂项目想让 AI 可靠修改而不破坏——一次性整体重写不稳定。
- **方法执行**：先让 02Agent 看项目概览 → 只读将编辑的舞台/角色脚本 → 搜要用的积木/扩展 → 不熟悉 opcode 调 getBlockHelp → 每次只改一个小范围 → 改后跑 getDiagnostics 确认 DSL 能同步回积木 → 涉造型优先用 SVG 工具；把任务拆成"建角色→加变量→写主脚本→补视觉"多步。
- **结论**：文档所述效果——拆成小步骤更稳定，并要求不要一次性重写大型项目；改后用诊断确认同步成功。

### 案例二：searchExtensions → installExtension 获取画笔积木（scenario，slug: 02agent-addon / c11）
- **问题**：项目需要画笔积木但尚未加载 Pen 扩展，AI 不知 `pen.*` 语法。
- **方法执行**：文档示例——先 `searchExtensions` 搜 pen → `installExtension` 安装内置 Pen → 再 `searchBlocks`/`getBlockHelp` 确认 `pen.*` 积木语法。外部直接 URL 默认拒绝，须用户明确提供可信 URL 且工具显式 `allowExternalUrl: true` 才尝试安装。
- **结论**：文档所述效果——先搜后装再验语法可获得正确积木；远程扩展走非沙箱 runner，只装可信扩展，不让 AI 自动装无关扩展。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 想让 AI / 02Agent 帮忙改一个 02Engine 项目（加角色/变量/脚本/造型），又怕一次性改坏。
2. AI 改了脚本却没同步回积木、诊断报错（如漏 `$field_`、把 Boolean 写成普通变量、用了未加载扩展的 opcode），想排查 DSL。
3. 想让 AI 帮忙找并装扩展，但担心它乱装不可信的外部 URL。
4. 想用本机 AI 客户端（OpenCode 等）经 bridge 调用编辑器里的 02Agent 工具，或连接一直等待/中文参数乱码。

### 语言信号
- "02Agent / AI 改项目 / 让 AI 写脚本 / 小步改 / 诊断"
- "虚拟文件 / stage.js / Scratch JS DSL / @script / blockId / applyPatch / getDiagnostics"
- "让 AI 装扩展 / searchExtensions / installExtension / allowExternalUrl"
- "Skill bridge / 40202 / WebSocket / manifest / --stdin / --base64 / 连接等待 / 改脚本没同步"

### 与相邻 skill 的区分
- 与 `extension-library`（also_read）：本卡是"让 AI 搜装扩展 + 改项目"；library 卡是"用户手动选型安装扩展"。手动装扩展用 library，让 AI 代劳或改项目用本卡。
- 与 `custom-reporter`（also_read）：本卡讲让 AI 改项目的工作流与安全边界；custom-reporter 讲"把自定义积木变报告积木/递归"的积木设计技巧。要写报告积木本身用 custom-reporter。

## E — 可执行步骤 (Execution)

**输入契约**：一个要交给 AI 的项目改动意图 + 已确认重要项目先存副本 + （用外部客户端时）本机 bridge 已启动、只监听 127.0.0.1、token 已配置 + 若需装扩展，用户已给出可信 URL 或接受显式 `allowExternalUrl: true`。
1. **看概览**：`getProjectOverview` 掌握项目结构。完成标准：明确要改的角色/脚本所在。
2. **只读目标**：`readFile` 只读将编辑的 `/stage.js` 或 `/sprites/<名>.js`，编辑时保留 `// @script` 与 `// blockId` 标记。完成标准：只读、不盲目全量改。
3. **搜积木/扩展**：`searchBlocks`、生疏 opcode 用 `getBlockHelp`；需扩展走 `searchExtensions`→`installExtension`→`searchBlocks/getBlockHelp` 验证语法。完成标准：opcode 与 DSL 写法已确认；判停：外部 URL 默认拒绝——须用户明确给出可信 URL 且工具显式 `allowExternalUrl: true`，否则停在此步并告知信任边界。
4. **小步改**：`applyPatch` 每次只改一个较小的脚本/资源范围；拆"建角色→加变量→写主脚本→补视觉"；涉造型用 `createSpriteWithSvg`/`addCostumeWithSvg` 等 SVG 工具（`createSpriteWithSvg` 只建新角色，加造型用 addCostume 系列）。完成标准：每步改动可独立验证；判停：禁止一次性重写大项目。
5. **诊断确认同步**：`getDiagnostics` 查 DSL 语法、字段名、输入类型、菜单值；常见根因：漏 `$field_` 前缀、把 Boolean 写成普通变量、用了未加载扩展的 opcode。完成标准：诊断通过、DSL 同步回积木；否则回第4步修正后再诊断。
6. **（如需 bridge）连本机桥**：本机启动 bridge server（默认 40202，只监听 127.0.0.1）→ 浏览器侧建立出站 WS（带 token）→ `GET /health`、`/manifest`、`/tools` 确认可用 → `POST /call` 调用工具；中文参数用 `--stdin` 或 `--base64 <base64Json>`。完成标准：调用成功、调用过程已记录；判停：bridge 被配置成监听 0.0.0.0/公网/局域网则立刻停并改为仅 127.0.0.1。
**输出契约**：改动后的项目（小步差分，每步可回溯）+ `getDiagnostics` 通过记录 + 扩展安装清单（含来源可信性确认）+（如用 bridge）调用日志与"仅本机监听"确认 + 重要项目改动前的副本。

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill
- 手动写一个扩展 → `extension-authoring`；手动选型安装扩展 → `extension-library`。
- 纯编辑器界面操作/UI 美化 → 不在本 skill 聚焦的"积木与扩展"范围内。

### 作者在文档中警告的失败模式
- **一次性重写大项目会失同步/出错**：破 `@script`/`blockId` 锚点、放大出错面；文档明确要求拆成多步小范围改。
- **改脚本没同步**：常见漏 `$field_`、把 Boolean 写成普通变量、用了未加载扩展的 opcode → 用 getDiagnostics 定位，不要凭感觉重试。
- **让 AI 自动装任意外部 URL / 无关扩展**：远程扩展走 unsandboxed runner、权限高；外部 URL 默认拒绝，须用户明确给出可信 URL 且工具显式 `allowExternalUrl: true`。
- **bridge 暴露到公网/局域网**：bridge 工具有项目写权限，token 只挡无关 WS、不能替代网络隔离；只应监听 127.0.0.1、浏览器侧只做出站 WS。
- **Windows 中文参数直接放 argv 会乱码**：用 `--stdin`（UTF-8）或 `--base64` 传参。
- **造型工具边界**：`createSpriteWithSvg` 只建新角色，加造型用 addCostume 系列；重名/建角色失败按 FAQ 排查。

### 文档的盲点 / 局限
- API Key 不通过 manifest 暴露，但仍保存在浏览器侧配置中，共享电脑勿存敏感 key。
- 重要项目建议先保存副本再让 02Agent 做大范围修改。
- 改项目状态的工具串行排队（聊天与 bridge 不冲突，但也不是并行加速）。
- 连接一直等待 → 查 bridge 是否启动、端口是否一致（默认 40202，稍后启动会自动重连）。
- manifest 公开协议/工具 schema/提示/项目概览/安全声明，但不公开 API Key、模型密钥、完整 agent 配置。

### 容易混淆的邻近方法论
- 内置 02Agent 聊天 vs 外部客户端经 bridge 调用——共用同一套动态生成 schema 的工具，但入口/传参不同。
- 虚拟文件（`/stage.js`、`/sprites/<名>.js`）vs 真实磁盘项目文件；改的是虚拟文件、同步回 VM。
- `searchExtensions`（搜内置/已知可信扩展）vs `installExtension`（可装外部 URL，触发信任边界）。
- Skill bridge（本机 40202、HTTP `/call`）vs 扩展调试桥（VSCode Toolbox 1101，见 extension-debug-convert），两者端口与用途不同。

---

## 相关 skills
- composes-with: `extension-library`、`custom-reporter`
- depends-on: 无

## 审计信息
- 来源: summary-extensions.md §3（02agent-addon）；candidates/framework.md f16-f19、principle.md p21/p22、case.md c08-c11、counter-example.md ce23-ce26；slug 02agent-addon（原文中文页，行816-937，含英文 API/URL 串）
- 引用回查: R 段引用逐字核对 `02engine-focused.txt` 行816-937（原页中文）；英文 URL/API 串 `ws://127.0.0.1:40202/agent?token=<token>`、`http://127.0.0.1:40202`、`GET /health`、`POST /call`、`--stdin`、`--base64` 均真实存在，且逐段 ≤100 词。
- 元数据: verified.yaml `cap.e2.agent-workflow`（status active / importance high / destination promoted）；also_read: extension-library, custom-reporter。
