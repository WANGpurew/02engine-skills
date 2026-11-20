# 02Engine 文档概览（积木搭建与扩展使用）

> 阶段 0 整书理解 + 参考内容映射。供概览/术语/背景类查询。具体任务按 SKILL.md 路由表加载能力卡。

## 它是什么

02Engine 是兼容 Scratch 的编辑器 + 运行时 + 桌面应用 + 打包器，基于 Scratch 与 TurboWarp 代码库。保留 Scratch 项目模型，为大型项目、重度扩展、直接 JS、额外积木提供工具。本 skill 按用户要求**只聚焦积木搭建与扩展使用**，不覆盖编辑器界面修改（New UI/窗口/主题/背景）。文档站 docs.02engine.org，CC BY 4.0，英文为主。

## 骨架：从用积木到造扩展

| 主题 | slug | 对应能力 |
|---|---|---|
| 额外积木与兼容分界 | blocks | extra-blocks |
| 自定义报告积木/递归 | return | custom-reporter |
| 注释内嵌 JS | direct-javascript-coding | inline-js |
| 云变量与 bot | cloud-variables | cloud-variables |
| 新编译器/禁用编译器/OPF | new-compiler, disable-compiler, opf | 02engine-guide（路由） |
| 扩展库选型安装 | extension-library | extension-library |
| 02Agent AI 助手 | 02agent-addon | ai-agent-workflow |
| 扩展调试桥/CLI转换/JS导出 | extension-debugger, convert-cli, javascript | 02engine-guide（路由） |
| 从零写扩展 12 步 | development/extensions/* | extension-authoring |
| addon/globals/ScratchX | development/* | 02engine-guide（路由） |

## 核心术语速览（完整 41 条见 glossary.md）

- **Extra Blocks**：继承 TurboWarp 的额外积木，warning 线前兼容 Scratch、线后不兼容（项目不能上传官网）。
- **Custom Reporter / return**：拖入 return 积木把自定义积木变成可嵌入的报告积木，支持递归；应无副作用、编译器下递归超几千层栈溢出。
- **#code marker**：帽子积木注释行首标记，之后内容作为原生 JS body；用 target/runtime/block/vars/wait，yield 防冻结、retire()+return 结束。
- **Cloud Variables**：独立云服、10 万字符上限但只存数字、会重置、编辑器打开时禁用；bot 须合规。
- **warp timer**：不刷新屏幕积木跑 500ms 后暂停防丢工作，抵消新编译器在编辑器内的提速。
- **OPF / FPS**：FPS 控产帧频率、OPF 控每帧 VM 工作量；先调 FPS 再调 OPF。
- **扩展七类来源**：Official Scratch / 02Engine / TurboWarp / PenguinMod / Mist / SharkPool / Other。
- **Compatibility badge**：Native / Batch / Not Scratch-compatible，后者的项目可能无法上传 scratch.mit.edu。
- **Scratch JS DSL / Virtual file**：02Agent 把项目映射成 /stage.js、/sprites/<名>.js 等虚拟文件，用 DSL 表达积木脚本。
- **Skill bridge**：02Agent 的 WS/HTTP 桥（127.0.0.1:40202），让本机 AI 客户端调用编辑器工具。
- **Sandbox / Unsandboxed**：沙箱是 iframe（1 帧延迟、不能访问 internals）；非沙箱是同上下文 script（瞬时、全权限、须 IIFE + 权限化 API）。
- **getInfo() / register()**：扩展元数据方法 / 注册 API（只能调一次）。
- **blockType**：COMMAND / REPORTER / BOOLEAN / HAT / EVENT。
- **startHats / hats**：EVENT（外部事件，仅非沙箱）、predicate HAT、edge-activated HAT（自动启动，任何扩展可用）。
- **Backward compatibility**：扩展"只增不改"原则——破坏兼容等于损坏已有项目。

## 重要限制与信任边界（来自文档自述）

- **Scratch 兼容割裂**：额外积木警告线后的积木、自定义报告积木、直接 JS、自定义扩展都会让项目无法上传 scratch.mit.edu。看徽章/警告判断。
- **信任模型偏激进**：外部 URL 扩展默认经非沙箱 runner 加载，可访问强浏览器/VM API——只装可信来源；Skill bridge 只监听 127.0.0.1、中文参数用 stdin/base64。
- **云变量**：编辑器打开时禁用、会重置、只存数字；用云变量或自定义扩展会禁用视频颜色侦测。
- **扩展开发**：register 只调一次、非沙箱 URL 必须 localhost:8000 精确开头、非沙箱死循环会冻结窗口丢数据、util.target 只能同步访问、参数 type 只是建议需自行转换、field↔input 互换破坏兼容、Cast.compare 只能用 <0/>0/===0。
- **编译器**：新编译器可能破坏少数自定义扩展（故障回退 old-compiler URL），编辑器内因 warp timer 无显著提升。
- **JS 导出**：02Engine 生成的 JS 为兼容/性能生成、无格式、含私有 API，不适合人工阅读——可读 JS 用 leopardjs.com。

## 不在本 skill 范围（用户明确不要）
New UI/编辑器窗口/背景设置/自定义默认项目/界面主题等编辑器修改；打包器完整文档与 Web 兼容杂项（CORS/账号/翻译/捐赠）；服务端部署运维。

## 批判性提示
文档偏英文与开发向，自定义扩展教程要求熟悉 JS 和 devtools；新编译器 2025-09 才发布，URL 与细节可能随版本变，以当前文档为准。
