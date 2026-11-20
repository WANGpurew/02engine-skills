<!-- capability_id: cap.e2.build-extension | revision: 1 | status: active -->

# 从零编写 02Engine 扩展的完整流程

## R — 原文 (Reading)

> "Make sure to always call register() exactly once. If you don't call it, your extension will never get added and we will keep waiting for it to load. If you call it multiple times, the behavior is undefined, so don't rely on it."
>
> 译文：register() 必须恰好调用一次——不调用则扩展永不加载、VM 一直等待；多次调用行为未定义，不可依赖。
>
> — 02Engine 功能文档·development/extensions/hello-world / slug: hello-world（行1367；源：candidates/case.md c14、principle.md p23、counter-example.md ce27）

> "And perhaps the biggest limitation of all is that every time a block runs, the script will pause for at least 1 frame, regardless of how simple the script is. It doesn't matter whether 'run without screen refresh' or turbo mode is enabled. It will always wait. ... running a block from an unsandboxed extension is actually instantaneous -- no forced 1 frame delay."
>
> 译文：沙箱最大的局限是每运行一次积木脚本至少暂停 1 帧，与积木多简单、是否勾选"不刷新屏幕运行"或 turbo 模式无关；而非沙箱扩展运行积木是瞬时完成的，没有这强制的 1 帧延迟。
>
> — 02Engine 功能文档·development/extensions/sandbox / slug: sandbox（行1793/1809；源：candidates/framework.md f25、counter-example.md ce31）

> "Once projects exist using your extension, it is critical that you do not change the extension in ways that will break compatibility as doing so will effectively corrupt projects. In these instances, you should create a brand new extension with an entirely new ID and leave the old version untouched."
>
> 译文：一旦有项目在用你的扩展，就绝不能做破坏兼容的改动——那等于损坏这些项目；确实无法避免时，应新建一个完全不同 ID 的扩展，把旧版本原样保留。
>
> — 02Engine 功能文档·development/extensions/compatibility / slug: compatibility（行3414-3583；源：candidates/framework.md f27、principle.md p32/p33、counter-example.md ce37）

## I — 方法论骨架 (Interpretation)

把"写一个扩展"当成一条有顺序依赖的流水线，后一页假定前一页的例子已懂，不跳步：
① **先选型**：Core（改 VM/GUI 仓库、能碰 internals、不能按 URL 加载）／沙箱自定义（iframe 隔离、可按 URL）／非沙箱自定义（两者皆可），本教程聚焦自定义扩展；
② **骨架起步**：class + `getInfo()`（id/name/blocks）+ `Scratch.extensions.register(new MyExtension())`，register 恰好一次；用文件/文本导入或 `python3 -m http.server 8080` 先验证；
③ **参数**：arguments 对象套对象、`text` 用 `[ARG]` 占位每参恰好一次、菜单 `menus` + `acceptReporters:true`（几乎总用 input 而非 field）、`type` 只是建议需自行转换；
④ **异步**：返回 Promise（setTimeout 手动 new、fetch 链式 then(text())、可 reject 必加 catch）；
⑤ **沙箱决策**：要不要 internals/变量/角色、能否接受 1 帧延迟；非沙箱走 localhost:8000 白名单 + IIFE + 'use strict' + 权限化 API（Scratch.fetch/canFetch/openWindow/redirect）、`util.target` 只能同步读；
⑥ **工具链**：迁到 `npm run dev` 热重载服务器免 hard refresh；
⑦ **丰富 API 与 hats**：color/docsURI/Cast.compare(<0/>0/===0)/hideFromPalette；EVENT / predicate HAT / edge-activated HAT 按触发需求选、startHats 块内用 util 版；
⑧ **兼容与发布**：守"只增不改"，必须破坏时新建 ID；沙箱静态 HTTPS（非 raw）、非沙箱提交官方库、自用走 Packager。
核心洞察：能力越强责任越大——非沙箱换来瞬时和 internals，同时背负"不抛错、返回合法值、不死循环"与权限化约束；兼容红线（ID/opcode/参数）一旦踩踏即损坏已有项目。

## A1 — 书中的应用 (Past Application)

### 案例一：hello-world 最小骨架（worked_example，slug: hello-world / c14）
- **问题**：从零写一个只报告 "Hello, world!" 的扩展积木，并确认它能被编辑器加载。
- **方法执行**：定义 `class HelloWorld`；`getInfo()` 返回 `{id:'helloworld', name:'It works!', blocks:[{opcode:'hello', blockType: Scratch.BlockType.REPORTER, text:'Hello, world!'}]}`；实现 `hello(){ return 'Hello, world!'; }`；文件末尾 `Scratch.extensions.register(new HelloWorld());`。在编辑器点 + → Custom Extension 加载该文件，此时**不勾** "Run extension without sandbox"。
- **结论**：文档所述效果——加载后侧栏出现名为 "It works!" 的扩展、积木可拖出并返回字符串；`register()` 必须且只能调用一次，否则永不加载或行为未定义。

### 案例二：async 积木的手动 Promise 与 fetch().catch()（worked_example，slug: async / c18）
- **问题**：扩展需要两个异步积木——一个近似 Scratch 的 wait（等 N 秒），一个 fetch URL 并返回文本。
- **方法执行**：wait 用 `new Promise((resolve,reject)=>{ setTimeout(()=>resolve(), args.TIME*1000); })`（setTimeout 不返回 Promise，须手动构造）；fetch 用 `fetch(args.URL).then(response=>response.text()).catch(error=>{ console.error(error); return 'Uh oh! Something went wrong.'; })`；COMMAND 积木的 resolve 不带值。
- **结论**：文档所述效果——Promise resolve 前当前脚本不继续、但其他脚本可继续；必须用 `text()` 把 Response 转成字符串以满足返回值限制；对可能 reject 的 Promise 加 catch 并返回错误提示字符串。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 要从零写一个 02Engine 扩展、给扩展加积木/参数/菜单/异步，不知从哪起步或按什么顺序学。
2. 纠结用沙箱还是非沙箱、本地怎么加载调试（localhost:8000、热重载、`?extension=`、hard refresh）。
3. 要给扩展加事件/帽积木（startHats、predicate / edge-activated HAT），或不确定触发写法。
4. 维护一个已发布的扩展，担心更新破坏兼容，或不知怎么发布（GitHub Pages / 官方库 / raw 链接无效）。

### 语言信号
- "写扩展 / 自定义积木扩展 / getInfo / register / opcode / blockType / id 怎么取"
- "加参数 / 菜单 / acceptReporters / 异步积木 / Promise / fetch / catch / 返回值"
- "沙箱 / 非沙箱 / unsandboxed / IIFE / iframe / 1 帧延迟 / localhost:8000 / 权限化 API"
- "hat 积木 / EVENT / startHats / predicate / edge-activated / isEdgeActivated"
- "向后兼容 / 别改 ID / 发布扩展 / hot reload / dev server / extension"

### 与相邻 skill 的区分
- 与 `extension-library`（also_read）：本卡管"从零写扩展"；library 卡管"选型、安装现成扩展"。只想装一个现成扩展用 library，不要来这里写 JS。
- 与 `inline-js`（also_read）：本卡产出独立可分发的扩展 JS 文件；direct-js 是在某个积木项目的注释里写 `#code` 原生 JS。改单个项目脚本用 direct-js，做可复用扩展用本卡。

## E — 可执行步骤 (Execution)

**输入契约**：一个扩展需求（积木功能清单 + blockType + 是否异步/帽积木）+ 已确认的开发环境（浏览器 devtools、Node.js/Git、可选官方 dev server）+ 沙箱模式需求（是否需访问 VM internals/变量/角色）+ 是否已有用户项目在用该扩展（决定兼容档位）。
1. **选型**：判断 Core / 沙箱自定义 / 非沙箱自定义，并定分发方式（URL / 文件导入 / 官方库）。完成标准：类型与分发明确；判停：需求属 Core 能力且要改 VM/GUI 仓库 → 超出本教程范围。
2. **hello-world 骨架**：class + `getInfo(){id,name,blocks}` + `Scratch.extensions.register(new ...)` 恰好一次；用文件/文本导入或 `python3 -m http.server 8080` 起步。完成标准：侧栏出现扩展、积木可拖出并返回合法值；判停：不加载先看 devtools 报错与 register 调用次数。
3. **inputs 参数与菜单**：`arguments` 每参定义 type/defaultValue/menu；`text` 用 `[ARG]` 占位、每参恰好一次；菜单在 `menus` 定义 items + 显式 `acceptReporters:true`；代码内自行转换类型。完成标准：参数/菜单可拖入且读取正确；判停：菜单 items 为空或写成数组 → 修正（空则无默认值、数组隐式 field）。
4. **async 异步**：返回 Promise；setTimeout 手动 new Promise、fetch 链式 then + `text()`；可能 reject 必加 catch 返回错误提示；COMMAND resolve 不带值。完成标准：等待期间其他脚本可继续、错误有提示字符串。
5. **沙箱 vs 非沙箱决策**：需访问 internals/变量/角色或不能接受 1 帧延迟 → 选非沙箱；否则沙箱。完成标准：模式明确；判停：沙箱里写高频/即时循环 → 改非沙箱。
6. **非沙箱改造（如需）**：整体包进 `(function(Scratch){ 'use strict'; ... })(Scratch)`、可加 `if(!Scratch.extensions.unsandboxed) throw ...` 守卫；`const vm = Scratch.vm`；`util.target` 仅同步读、异步回调前先存值；远程访问换 `Scratch.fetch/canFetch/openWindow/redirect` 并优雅处理用户拒绝。完成标准：URL 以 `https://extensions.02engine.org/` 或 `http://localhost:8000/` 精确开头加载成功、无全局泄漏、无抛错/死循环；判停：URL 用 127.0.0.1/0.0.0.0 → 加载失败。
7. **热重载开发服务器**：`git clone .../extensions && npm ci && npm run dev` → `http://localhost:8000`；扩展放 `extensions/<用户名>/`、ID 含用户名。完成标准：改代码免 hard refresh 自动生效、模拟真实站点。
8. **assorted-apis 外观与工具**：color1/2/3、docsURI、disableMonitor、`Scratch.Cast.compare` 用 `<0/>0/===0`、hideFromPalette 隐藏积木、filter 仍用 `util.target.isStage` 兜底、isTerminal 仅影响外形。完成标准：外观/隐藏/目标校验符合预期。
9. **hats（如需）**：EVENT（仅非沙箱、不写函数、`isEdgeActivated:false`、用 startHats 触发，块外用 `vm.runtime`、块内用 `util.startHats`）；predicate HAT（定义返回 true/false 的函数、自行 startHats 如 BEFORE_EXECUTE）；edge-activated HAT（`isEdgeActivated:true`、Scratch 每帧自动 startHats、不用 shouldRestartExistingThreads、任何扩展含沙箱可用）。完成标准：触发时机与过滤（按菜单 value、target）符合预期。
10. **兼容与发布**：对照"绝不改"清单逐项确认；要改就新建积木 + 旧积木 `hideFromPalette:true` 委托；必须破坏时新建全新 ID 扩展、旧版不动。发布：沙箱 → 静态 HTTPS（GitHub Pages，**不用 raw.githubusercontent.com**）；非沙箱 → 提交 extensions.02engine.org；自用 + Packager → 无需提交。完成标准：既有项目不损坏、发布链接可用。
**输出契约**：一份可运行的扩展 JS（getInfo 元数据 + 积木定义 + 方法实现 + 恰好一次的 register 调用）+ 沙箱模式与 URL/权限化 API 核对记录 + 向后兼容检查表（绝不改项逐项打勾）+ 发布方式与实际链接。

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill
- 只想安装一个现成扩展 → `extension-library`；让 AI 代为搜装 → `ai-agent-workflow`。
- 改编辑器 UI/按钮/面板 → 写 addon，不在本 skill 范围；在积木项目注释里写原生 JS → `inline-js`。

### 作者在文档中警告的失败模式
- **register 调用次数**：0 次则永不加载、VM 一直等待；>1 次行为未定义、不可依赖。
- **返回值与类型**：REPORTER 只返回 string/number/boolean、BOOLEAN 只返回 boolean；null/undefined/列表/对象非法。参数 `type` 只是建议、从不强制，不自行 toString/toNumber 会在真实实参下报错；defaultValue 不用于布尔。
- **菜单**：items 不能为空、几乎总显式 `acceptReporters:true`；把菜单写成数组会隐式 field；field↔input 互换破坏兼容。
- **沙箱 1 帧延迟**：每积木至少暂停 1 帧、turbo/"不刷新屏幕运行"都无效，高频循环几乎不可用；沙箱不能访问 internals/变量/角色。
- **非沙箱三大责任**：不得抛错、输入/布尔积木必须返回合法值、不得陷入死循环（会冻结整个窗口、导致数据丢失）；`util` 是共享对象，只能在积木同步执行瞬间读，setTimeout/Promise 回调前须先存值。
- **API 契约**：`Scratch.Cast.compare` 只能用 `<0/>0/===0`（不能 `===1/===-1`）；隐藏积木用 hideFromPalette 而非 `filter:[]`；filter 可被拖放/背包绕过，代码仍须 `util.target.isStage` 兜底。
- **兼容/发布**：破坏兼容等于损坏项目；必须破坏时新建 ID；`raw.githubusercontent.com` 链接无效，沙箱托管要用 GitHub Pages。

### 文档的盲点 / 局限
- 自定义扩展不兼容 Scratch 官网：项目不能上传 scratch.mit.edu，但可在 02Engine 打开并用 Packager 打包。
- 三种"扩展"体系不同：Core 需改 VM/GUI 仓库（本教程不覆盖）；沙箱几乎不可用；非沙箱责任大。
- 非沙箱 URL 白名单只精确匹配 `https://extensions.02engine.org/` 与 `http://localhost:8000/`；文件/源码手动加载每次都要勾 "Run extension without sandbox"。
- globals（vm/ScratchBlocks/paper/ReduxStore）仅供开发控制台调试，沙箱扩展不能用、非沙箱也不应使用。
- 官方 dev server 用 npm ci/npm run dev；而引擎仓库本身用 Bun 作脚本运行器（除非 README 另有说明），别混用包管理器。

### 容易混淆的邻近方法论
- Core / 沙箱自定义 / 非沙箱自定义是三种体系，选型依据不同（是否碰 internals、是否按 URL 分发）。
- 沙箱 iframe（隔离、1 帧延迟）vs 非沙箱 script 标签（同上下文、瞬时、全权限）。
- EVENT / predicate HAT / edge-activated HAT 的触发者与启动方式不同：前者外部 startHats、谓词帽需自行 startHats、边沿帽由 Scratch 每帧自动 startHats。
- getInfo 的"可改白名单"（name/docsURI/color/icons/text 顺序/disableMonitor/defaultValue/增菜单项）vs"绝不改清单"（ID/opcode/类型/参数/isTerminal/acceptReporters）。

---

## 相关 skills
- composes-with: `extension-library`、`inline-js`
- depends-on: 无

## 审计信息
- 来源: summary-extension-dev.md §1-3（introduction→wrapping-up 12 页）；candidates/framework.md f24-f32、principle.md p23-p33、case.md c14-c23、counter-example.md ce27-ce38；slug: hello-world / inputs / async / sandbox / unsandboxed / better-development-server / assorted-apis / hats / compatibility / share（原文行1367/1793/1809/1933/1949/2110/2514/3414/3604）
- 引用回查: R 段三处英文引用逐字核对 `02engine-focused.txt` 行1367、1793/1809、3414-3583，均 ≤100 词；中文翻译忠实原意。
- 元数据: verified.yaml `cap.e2.build-extension`（status active / importance critical / destination promoted）；also_read: extension-library, inline-js。
