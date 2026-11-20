<!-- capability_id: cap.e2.addon-and-legacy | revision: 1 | status: active -->

# Addon 体系与旧扩展 / ScratchX 背景

## R — 原文 (Reading)

> "Open the addon menu and choose one of the custom import actions: Choose folder: install an addon from an unpacked folder. Import Zip: install an addon packaged as a zip file. The addon should contain a valid addon manifest and the userscripts, styles, assets, or settings declared by that manifest."
>
> 译文：打开 addon 菜单选自定义导入动作——"Choose folder" 从未打包文件夹安装、"Import Zip" 从 zip 包安装；addon 须含合法 manifest 及该 manifest 声明的 userscripts/styles/assets/settings。
>
> — 02Engine 功能文档·custom-addon / slug: addon-and-legacy（原文行601–607；源：candidates/framework.md f35）

> "Only the active editor window has the writable Blockly workspace. ... Use addon API helpers such as tab.waitForElement and rerun-safe observers instead of storing permanent DOM references. If your addon adds UI, make the insertion idempotent."
>
> 译文：只有活动编辑器窗口有可写 Blockly workspace；用 `tab.waitForElement` 与可重跑 observer 而非永久 DOM 引用；加 UI 时插入要幂等。
>
> — 02Engine 功能文档·development/addons / slug: addon-and-legacy（原文行3668/3674/3676；源：candidates/framework.md f36、principle.md p14、counter-example.md ce39/ce40）

> "Global variables that can be accessed using the developer console, for convenience. These can not be used in sandboxed custom extensions and should not be used in unsandboxed custom extensions. ... Any functionality related to hardware will not work. Many extensions also rely on web services that are increasingly broken due to the age of the extensions."
>
> 译文：全局变量供开发者控制台便利使用，沙箱扩展不能用、非沙箱扩展也不应使用；ScratchX 硬件相关功能不可用，许多扩展依赖的旧 web 服务因年久多已失效。
>
> — 02Engine 功能文档·development/globals / scratchx / slug: addon-and-legacy（原文行3746/3750/3779；源：counter-example.md ce41、summary-extension-dev.md §development/globals·scratchx）

## I — 方法论骨架 (Interpretation)

把"给作品加能力 vs 改编辑器本身"先分清，再谈 addon 兼容与旧体系：
① **扩展 vs addon**：给作品加积木 → 写扩展（`extension-authoring`）；改编辑器 UI（按钮/面板/右键菜单/搜索/调试/主题）→ 写 addon。addon 跑在编辑器 UI 里、不在 Scratch 项目内部；属"受信任的编辑器代码"，只装可信来源；
② **本地安装**：addon 菜单选 Choose folder（未打包文件夹）或 Import Zip（zip 包），须含合法 manifest 及其声明的 userscripts/styles/assets/settings；
③ **New UI 兼容**：New UI 把编辑器包进可移动窗口、会重挂载/移动 DOM → 用 `tab.waitForElement` 与可重跑 observer 而非永久 DOM 引用；插入 UI 要幂等（先查存在、用稳定 class/data 属性、活动编辑器变化时清理过期元素、菜单回调按对注册一次加 guard）；
④ **Active Editor Rule**：只有一个活动编辑器有可写 Blockly workspace，改积木/读选中/开菜单/定位浮层只针对活动 workspace，非活动窗口是快照不注入；浮层锚定活动 workspace，拖拽/缩放/切标签/换主题后重算；
⑤ **双 UI 测试**：覆盖 Old UI、New UI 单/双窗口、切标签、明暗主题、切角色、开关弹窗、重载扩展等场景；
⑥ **globals**：`vm`/`ScratchBlocks`/`paper`/`ReduxStore` 仅供控制台调试，沙箱扩展不能用、非沙箱扩展也不应使用（非沙箱用 `Scratch.vm`）；
⑦ **ScratchX/`.sbx`**：旧体系，Flash 移除后失效、2023 初关站，硬件功能不可用、依赖的旧 web 服务多已失效；用法是找扩展 JS 源码 URL 像普通自定义扩展加载。
核心洞察：addon 与扩展是两套作用域——addon 改编辑器 UI、扩展加积木；New UI 下最大约束是"唯一活动可写编辑器 + DOM 会被重挂载"，故须元素发现助手 + 幂等注入 + 双 UI 测试；globals/ScratchX 是背景答疑、非扩展应依赖的能力。

> 路由说明：本能力经 02engine-guide 路由入口访问（未独立成 Skill），user 问 addon/该写扩展还是 addon/ScratchX/globals 时从 02engine-guide 命中本卡。

## A1 — 书中的应用 (Past Application)

### 案例一：本地安装自定义 addon（worked_example，slug: custom-addon / f35）
- **问题**：想从本地安装一个自定义 addon，给编辑器 UI 加按钮/面板/右键菜单/主题等工作流功能。
- **方法执行**：准备合法 manifest 及声明的 userscripts/styles/assets/settings → addon 菜单选 "Choose folder"（未打包文件夹）或 "Import Zip"（zip 包）→ 验证功能在编辑器 UI 生效。GUI 源码自带示例：pause、mouse-pos、block-count、find-bar、variable-manager、editor-searchable-dropdowns、comment-vscode-sync。
- **结论**：文档所述效果——addon 跑在编辑器 UI 而非 Scratch 项目内部；属"受信任的编辑器代码"，涉及项目文件/localStorage/远程 URL/扩展调试时尤其只装可信来源。

### 案例二：为 New UI 编写兼容 addon 的幂等注入（scenario，slug: development/addons / f36+ce39+ce40）
- **问题**：New UI 窗口化会重挂载/移动编辑器 DOM，addon 的 DOM 引用失效、控件重复或定位错位。
- **方法执行**：用 `tab.waitForElement` 与可重跑 observer 而非永久 DOM 引用；只写活动 Blockly workspace、非活动快照不注入；插入 UI 幂等（先查存在、稳定 class/data 属性、活动窗变化时清理过期元素、菜单回调按对注册加 guard）；浮层锚定活动 workspace 并在拖拽/缩放/切标签/换主题后重算；按测试清单覆盖 Old UI、New UI 单/双窗口、切角色、开关弹窗等。
- **结论**：文档所述效果——New UI 下始终只有一个活动可写编辑器，遵循幂等 DOM 发现与双 UI 测试清单可避免失效/重复/错位。

### 案例三：ScratchX 旧扩展的加载与限制（scenario，slug: development/scratchx）
- **问题**：手上有 ScratchX 旧扩展/`.sbx` 项目，问还能不能用。
- **方法执行**：找到扩展 JS 源码 URL，像普通自定义扩展一样加载；须知道 Flash 移除后 ScratchX 已失效、网站 2023 初关闭，硬件功能不可用、依赖的旧 web 服务多已失效。
- **结论**：文档所述效果——02Engine 对 ScratchX/`.sbx` 有初级（primitive）支持，但硬件功能不可用、旧 web 服务多失效，不应作为可靠能力来源。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 想判断该写扩展还是 addon（改编辑器 UI vs 给作品加积木）。
2. 要从本地文件夹/zip 安装 addon，或问 addon 怎么在新界面里不失效。
3. addon 在 New UI 下控件重复、定位错位、DOM 引用失效、写到非活动窗口。
4. 问 ScratchX/`.sbx` 旧扩展还能不能用、`vm`/`ReduxStore` 等 globals 能否在扩展里用。

### 语言信号
- "addon / 扩展还是 addon / 编辑器插件 / 改编辑器 UI / manifest"
- "New UI / 新界面 / 活动编辑器 / Active Editor Rule / Blockly workspace / tab.waitForElement"
- "DOM 引用失效 / 幂等 / 重复控件 / 浮层定位 / Old UI 双测"
- "globals / vm / ScratchBlocks / paper / ReduxStore / ScratchX / sbx / 旧扩展"

### 与相邻 skill 的区分
- 与 `extension-authoring`（also_read）：本卡是"写 addon 改编辑器 UI + 旧体系背景"；build 卡是"写扩展给作品加积木"。加积木用 build，改 UI 用本卡思路（但具体界面修改不在本 skill 聚焦范围）。
- 与 `extension-library`：从库里装扩展/看可信来源归 extension-library；本卡是 addon 的安装与兼容结构。二者都强调"只装可信来源"。
- 路由说明：本卡经 02engine-guide 路由入口访问（未独立成 Skill），非独立入口。

## E — 可执行步骤 (Execution)

**输入契约**：一个明确意图——(A) 判断写扩展还是 addon / (B) 本地安装并兼容 addon / (C) 问 globals 或 ScratchX 旧体系；含（B 时）合法 manifest 与资源、（C 时）具体疑问。
1. **分清作用域**：加积木给作品 → `extension-authoring`；改编辑器 UI → addon。完成标准：明确写扩展还是 addon；判停：目的是具体界面美化/布局操作 → 不在本 skill 范围，说明边界。
2. **（B 安装）准备与导入**：准备合法 manifest 及声明的 userscripts/styles/assets/settings → addon 菜单选 Choose folder 或 Import Zip → 验证生效。完成标准：功能在编辑器 UI 出现；判停：manifest 不合法或缺声明资源 → 补全后重试；来源不可信 → 不装。
3. **（B 兼容）幂等 DOM 发现**：用 `tab.waitForElement` 与可重跑 observer 代替永久 DOM 引用；插入 UI 先查存在、用稳定 class/data 属性、活动窗变化时清理过期元素、菜单回调按对注册加 guard。完成标准：切窗/重挂载后控件不重复不失效。
4. **（B 兼容）守 Active Editor Rule**：改积木/读选中/开菜单/定位浮层只针对活动 workspace，非活动快照不注入；浮层锚定活动 workspace 并在拖拽/缩放/切标签/换主题后重算。完成标准：多窗口下只写活动编辑器、浮层定位正确。
5. **（B 兼容）双 UI 测试**：覆盖 Old UI、New UI 单/双窗口（含一锁定）、切标签、明暗主题、切角色、开关弹窗、重载扩展等场景。完成标准：清单各场景均通过。
6. **（C 答疑）给结论**：globals 仅供控制台调试、扩展不应使用（非沙箱用 `Scratch.vm`）；ScratchX/`.sbx` 是旧体系、硬件功能不可用、旧 web 服务多失效。完成标准：明确回答并给出现代替代路径。

**输出契约**：作用域判定结论（扩展 vs addon）+（B 时）合法 manifest 与安装验证 + 幂等 DOM 注入与 Active Editor Rule 的实现 + 双 UI 测试清单结果 +（C 时）globals/ScratchX 的答疑结论；附"只装可信来源 addon"的确认。

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill
- 给作品加积木 → `extension-authoring`；从扩展库选型安装 → `extension-library`。
- 编辑器界面美化/布局的具体操作 → 不在本 skill 聚焦范围，明确说明本 skill 聚焦积木与扩展。
- 问编译器性能/热重载/JS 导出 → `compiler-and-perf` / `extension-debug-convert`。

### 作者在文档中警告的失败模式
- **New UI 硬编码 DOM 引用失效**：New UI 重挂载/移动 DOM，永久引用、一次性查询、重复菜单回调会失效或错位 → 用 waitForElement + 幂等注入。
- **只有一个活动可写编辑器**：非活动窗口是快照，给每个窗口注入可写监听/在非活动窗改积木会写错窗口或重复 hook。
- **addon 是受信任编辑器代码**：涉及项目文件/localStorage/远程 URL/扩展调试时尤其只装可信来源。
- **globals 不应在扩展使用**：沙箱扩展不能用、非沙箱也不应使用，非沙箱应走 `Scratch.vm`。
- **ScratchX 硬件功能不可用**：Flash 移除后失效、旧 web 服务多失效，作可靠能力来源会踩坑。

### 文档的盲点 / 局限
- 02Engine 对 ScratchX/`.sbx` 仅"初级（primitive）支持"，非完整兼容。
- New UI 下主题/样式变化需 addon 自行监听更新 DOM，不能等整页刷新。
- globals 中的 `ScratchBlocks`/`paper` 分别只在打开编辑器/造型编辑器后可用，非稳定扩展 API。
- 本卡为路由内背景答疑，不覆盖 addon 开发的完整 GUI/VM 架构与仓库开发细节。

### 容易混淆的邻近方法论
- 扩展（加积木、进作品）vs addon（改编辑器 UI、不进项目）——作用域不同，别混用。
- Active Editor（唯一可写 workspace）vs 非活动快照（只读、不注入）。
- `tab.waitForElement`（元素发现助手）vs 永久 DOM 引用（New UI 下失效）——前者才是正确写法。
- globals（控制台调试便利）vs `Scratch.vm`（非沙箱扩展正规入口）。
- ScratchX/`.sbx`（失效旧体系）vs 现代自定义扩展（按 URL 加载、可信来源）。

---

## 相关 skills
- composes-with: `extension-authoring`
- depends-on: 无（经 02engine-guide 路由入口访问，未独立成 Skill）

## 审计信息
- 来源: summary-extensions.md §2（custom-addon）/development/addons·globals·scratchx；candidates/framework.md f35/f36、principle.md p14、case.md（安装/兼容演示见 f35/f36 source_quote）、counter-example.md ce39–ce41；slug: addon-and-legacy（原文行601–607/3668–3682/3746–3779）
- 引用回查: R 段三处英文引用逐字核对 `02engine-focused.txt` custom-addon/development-addons/globals/scratchx 页，均 ≤100 词；中文翻译忠实原意。
- 元数据: verified.yaml `cap.e2.addon-and-legacy`（status active / importance low / destination **router**）；also_read: extension-authoring；A2 已注明经 02engine-guide 路由访问。
