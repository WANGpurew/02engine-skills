<!-- capability_id: cap.e2.extension-debug-convert | revision: 1 | status: active -->

# 扩展热重载桥、SB3 转 CLI 与 JS 导出路径

## R — 原文 (Reading)

> "02Engine includes a WebSocket bridge for the 02Engine VSCode Toolbox. It lets a local development tool push extension code into the running editor, reload it, and synchronize comment data used by related addons. ... The code is converted into a data:application/javascript URL. 02Engine loads it through vm.extensionManager.loadExtensionURL."
>
> 译文：02Engine 为 VSCode Toolbox 提供 WebSocket 桥，让本地开发工具把扩展代码推给运行中的编辑器、重载并同步注释数据；代码转成 `data:application/javascript` URL，经 `vm.extensionManager.loadExtensionURL` 加载。
>
> — 02Engine 功能文档·extension-debugger / slug: extension-debug-convert（原文行1001/1036/1038；源：candidates/framework.md f20/f21、case.md（连接步骤 f20 source_quote））

> "The 02Engine Packager can export a Scratch project as a Node.js command line application. This is useful for automated tasks, server-side experiments, simulations, and tests that do not need a graphical stage. ... cli.getArgs() / cli.getArg(key) ... cli.exit(code)"
>
> 译文：Packager 能把 Scratch 项目导出为 Node.js 命令行程序，适合无需图形舞台的自动化、服务端实验、模拟与测试；提供 `cli.getArgs()`/`cli.getArg(key)` 读参、`cli.log` 等输出、`cli.exit(code)` 退出。
>
> — 02Engine 功能文档·convert-cli / slug: extension-debug-convert（原文行1110/1169–1175；源：candidates/framework.md f22、case.md c24、counter-example.md ce43）

> "If you want to package/HTMLify a project, use https://packager.02engine.org/ — If you want to convert Scratch projects to readable and editable JavaScript, use https://leopardjs.com/ instead of 02Engine."
>
> 译文：要打包/网页化项目用 https://packager.02engine.org/；要把 Scratch 项目转成可读可编辑的 JavaScript 用 https://leopardjs.com/，不要用 02Engine。
>
> — 02Engine 功能文档·javascript / slug: extension-debug-convert（原文行1211/1213；源：candidates/framework.md f23、principle.md p20、counter-example.md ce44）

## I — 方法论骨架 (Interpretation)

把"扩展调试桥/转换/导出"当成一条开发工具链，按目的选路径：
① **热重载桥**：为 VSCode Toolbox 提供 WS 桥（`ws://localhost:1101`），本地工具把扩展代码推给运行中的编辑器；连接步骤：启动 Toolbox server → 打开 02Engine → Advanced Settings → `02Engine VScode Toolbox` → 点 "Connect to 02Engine VScode Toolbox Server" → 确认状态 connected；
② **重载机制**：代码转 `data:application/javascript` URL → `vm.extensionManager.loadExtensionURL` → 检测扩展 ID → 若已加载先移除旧注册与积木定义 → 加载新版本并刷新 toolbox/workspace；比常规加载更激进；语法错误后可能留陈旧状态，需刷新编辑器；
③ **CLI 转换**：Packager → 选 .sb3 → Open Environment → `Other environments > Node.js CLI` → 打包下载；解压装 Node.js 18+、按包内 README 构建；用 `cli.getArg/getArgs` 读参、`cli.log/error/warn/info` 输出、`cli.exit(code)` 退出，say/ask 可映射到控制台；无图形舞台、鼠键受限、依赖浏览器 API 的扩展可能不工作；
④ **JS 导出分路径**：打包/网页化 → `packager.02engine.org`；可读可编辑 JS → `leopardjs.com`（02Engine 生成代码无格式、含私有 API 如 `listGet`/`b0.value`，不适合人工读）；仅看生成代码 → 启动前控制台 `vm.enableDebug()`；
⑤ **安全**：本地开发 WS server 不要暴露到不可信网络，推送的扩展代码信任级别等同自定义扩展。
核心洞察：三件事目的不同——热重载调试用 1101 桥、无界面运行用 Node.js CLI、拿代码先分清"打包"还是"可读 JS"（可读走 leopardjs，别读 02Engine 生成码）。

> 路由说明：本能力经 02engine-guide 路由入口访问（未独立成 Skill），user 问 02Engine 热重载/CLI 转换/JS 导出时从 02engine-guide 命中本卡。

## A1 — 书中的应用 (Past Application)

### 案例一：VSCode Toolbox 热重载连接（worked_example，slug: extension-debugger / f20）
- **问题**：在 VSCode 写完扩展代码后，想实时推给运行中的 02Engine 编辑器重载调试。
- **方法执行**：启动 02Engine VSCode Toolbox server → 打开 02Engine → Advanced Settings → 找 `02Engine VScode Toolbox` → 点 "Connect to 02Engine VScode Toolbox Server" → 确认状态变 connected → 推送代码（代码转 data: URL 经 `loadExtensionURL` 加载、移除旧注册加载新版本）。VM 可用时编辑器也会自动初始化调试服务，toolbox 晚于编辑器启动时用手动按钮。
- **结论**：文档所述效果——连接后代码改动可热重载到编辑器、分类与 workspace 刷新；该重载比常规加载激进。

### 案例二：SB3 转 CLI 的命令行用法（worked_example，slug: convert-cli / c24）
- **问题**：项目打包成 Node.js CLI 后，如何用命令行参数运行、在积木/扩展代码里读参数与输出。
- **方法执行**：运行 `app.exe your-project.sb3 --mode test --count 5 --flag`；代码里 `const args = cli.getArgs(); cli.log('Arguments:', args); if (cli.getArg('mode') === 'test') {...} cli.exit(0);`；用 `cli.log/error/warn/info` 输出、`cli.exit(code)` 带码退出。
- **结论**：文档所述效果——CLI 项目可通过 helper API 读参、写控制台、带码退出，say/ask 也可能映射到控制台；须 Node.js 18+、构建命令以包内 README 为准。

### 案例三：为项目选 JS 导出路径（scenario，slug: javascript / f23+ce44）
- **问题**：想拿到项目的 JavaScript，但打包/网页化与可读代码是不同目的。
- **方法执行**：打包/HTMLify → `packager.02engine.org`；可读可编辑 JS → `leopardjs.com`（不用 02Engine，其生成代码为兼容/性能生成、无格式、含 `listGet` 等 runtime 私有函数）；仅调试看生成代码 → 启动前控制台 `vm.enableDebug()`。
- **结论**：文档所述效果——可读代码须用 leopardjs；`vm.enableDebug()` 只是在控制台打印编译输出、不是导出方案。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 在 VSCode/本地写扩展，想热重载推给编辑器调试，或调试桥连不上（1101）。
2. 想把 .sb3 项目转成 Node.js 命令行程序做自动化/服务端实验/CI。
3. 想拿到项目的 JavaScript——打包网页化、还是转成可读可编辑代码、或只看生成代码。
4. 重载后积木不刷新、旧扩展残留、或某扩展代码推过去报错。

### 语言信号
- "热重载 / 调试桥 / VSCode Toolbox / 1101 / Connect / 连不上 / unavailable"
- "SB3 转 CLI / Node.js CLI / Packager / cli.getArg / cli.log / cli.exit"
- "JS 导出 / 下载 JavaScript / 打包 / HTMLify / leopardjs / 可读 JS / vm.enableDebug"
- "data: URL / loadExtensionURL / 陈旧状态 / 刷新编辑器"

### 与相邻 skill 的区分
- 与 `extension-authoring`（also_read）：本卡是"调试/转换/导出"工具链；build 卡是"写扩展本体"。要写扩展代码用 build，要把写好的代码热重载/转 CLI/导出用本卡。
- 与 `compiler-and-perf`（router）：编译器故障回退/OPF 调优归 compiler-and-perf；本卡只管扩展推送、项目转换与代码导出。
- 路由说明：本卡经 02engine-guide 路由入口访问（未独立成 Skill），非独立入口。

## E — 可执行步骤 (Execution)

**输入契约**：一个明确目的——(A) 热重载调试扩展（含本地 1101 server 与编辑器）/ (B) 把 .sb3 转 Node.js CLI（含 .sb3 与 Node.js 18+ 环境）/ (C) 拿项目 JS（区分打包/可读/看代码）。
1. **分清目的**：确定是热重载调试、CLI 转换还是 JS 导出。完成标准：目的唯一且明确；判停：目的是"写扩展本体" → 转 `extension-authoring`。
2. **（A 热重载）连桥**：启动 Toolbox server → 打开 02Engine → Advanced Settings → `02Engine VScode Toolbox` → 点 Connect → 确认状态 connected。完成标准：状态为 connected；判停：unavailable → 确认 1101 服务在跑；连接被拦 → 查本地网络/WebSocket 权限。
3. **（A 热重载）推送与刷新**：推送扩展代码（转 data: URL 经 `loadExtensionURL`、移除旧注册加载新版本）；积木不刷新就切换扩展分类再切回或重开扩展库；旧扩展残留/语法错误后留陈旧状态 → 刷新编辑器。完成标准：新代码生效、workspace 正确刷新。
4. **（B CLI）打包构建**：Packager → 选 .sb3 → Open Environment → `Other environments > Node.js CLI` → 下载；解压、装 Node.js 18+、按包内 README 装依赖构建。完成标准：生成可执行文件并能用参数运行；判停：项目依赖图形舞台/鼠键/浏览器 API → 不适合 CLI，换方案。
5. **（B CLI）读写参数**：用 `cli.getArg/getArgs` 读参、`cli.log/error/warn/info` 输出、`cli.exit(code)` 退出。完成标准：命令行参数被正确读取、输出与退出码符合预期。
6. **（C 导出）按目的选路径**：打包/网页化 → `packager.02engine.org`；可读可编辑 JS → `leopardjs.com`；仅看生成代码 → 控制台 `vm.enableDebug()`。完成标准：拿到符合目的的产物或输出；判停：想人工读改 02Engine 生成 JS → 劝阻（无格式、含私有 API），改用 leopardjs。

**输出契约**：按目的给出——(A) 桥连接状态与热重载验证记录（含连不上/陈旧状态的排查结论）/ (B) 构建好的 CLI 可执行文件 + 参数读写实现 + "项目适合 CLI"的判定 / (C) 对应路径的导出产物或生成代码输出 + "02Engine 生成 JS 不适合人工读"的说明；附本地 server 不外暴露的安全确认。

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill
- 写扩展本体（getInfo/register/参数/发布）→ `extension-authoring`。
- 编译器故障回退、OPF/FPS 调优 → `compiler-and-perf`。
- 想人工阅读/编辑 02Engine 生成的 JS → 本卡不给该方案（改用 leopardjs）。

### 作者在文档中警告的失败模式
- **调试桥连不上**：状态 unavailable → 确认 1101 端口服务在跑；浏览器拦连接 → 查本地网络/WebSocket 权限。
- **重载后陈旧状态**：开发重载会碰 VM 内部，语法错误后可能留旧注册/积木 → 切分类刷新或刷新编辑器。
- **CLI 局限**：无图形舞台、鼠键交互受限/不可用、音频视环境受限、依赖浏览器 API 的扩展可能不工作；须 Node.js 18+。
- **02Engine 生成 JS 不适合人读**：为兼容/性能生成、无格式、含 runtime 私有函数（如 `listGet`、`b0.value`），强行阅读妨害学习；要可读代码必须用 leopardjs。`vm.enableDebug()` 只用于看编译输出，不是导出方案。
- **本地 server 安全**：WS server 是本地开发功能，不要暴露到不可信网络；推送的扩展代码信任级别等同自定义扩展。

### 文档的盲点 / 局限
- 热重载机制专为开发设计、比常规加载激进（追求重载速度），稳定性不如常规加载。
- Packager 总以非沙箱运行扩展，CLI 打包命令随模板版本变化，以包内 README 为准。
- JS 生成代码写法随上下文变化（`myList[i]` 可能被生成为 `(b1.value[(b0.value|0)-1] ?? "")` 或 `listGet(b0, b1.value)`），不能当作稳定可读输出。
- 桥的 comment 数据同步用于相关 addon 监听，非通用数据通道。

### 容易混淆的邻近方法论
- 热重载桥（推送代码到编辑器调试）vs CLI 转换（把项目打包成命令行程序）——目的完全不同。
- 打包/网页化（packager）vs 可读 JS（leopardjs）：前者要可运行产物，后者要可编辑源码。
- `vm.enableDebug()`（看生成代码）vs JS 导出（拿可读代码）——前者是调试手段，不是导出方案。
- 连桥状态 connected（正常）vs unavailable（1101 服务未跑）——排查入口不同。

---

## 相关 skills
- composes-with: `extension-authoring`
- depends-on: 无（经 02engine-guide 路由入口访问，未独立成 Skill）

## 审计信息
- 来源: summary-extensions.md §4（extension-debugger）/§6（convert-cli）/§7（javascript）；candidates/framework.md f20–f23、principle.md p20、case.md c24、counter-example.md ce43/ce44；slug: extension-debug-convert（原文行1001/1018/1036/1038/1054/1110/1169–1180/1211/1213/1226）
- 引用回查: R 段三处英文引用逐字核对 `02engine-focused.txt` extension-debugger/convert-cli/javascript 页，均 ≤100 词；中文翻译忠实原意。
- 元数据: verified.yaml `cap.e2.extension-debug-convert`（status active / importance medium / destination **router**）；also_read: extension-authoring；A2 已注明经 02engine-guide 路由访问。
