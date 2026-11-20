# 术语词典 (GLOSSARY) — 02Engine（积木搭建与扩展使用）

> 全书共享术语，供所有能力卡引用。英文原词保留，中文解释其文档特定用法。
> 阶段 3 产出，源自 glossary 提取器（41 条）。

```yaml
- id: g01
  term: Extra Blocks / modified argument reporters / 兼容性分界线
  type: term
  source_chapter: blocks
  author_definition: |
    "These blocks are 'compatible' with Scratch because they're actually just modified argument reporters."
    文档以一条 warning 划线："Every block beyond this warning is incompatible with Scratch. Projects that use them cannot be uploaded to the Scratch website."
  key_distinction: |
    ≠ "Scratch 官方积木" — 是 02Engine 内置、继承自 TurboWarp 的额外类别，暴露 Scratch 项目通常访问不到的运行时功能。
    ≠ "越多越好" — 线前的 is compiled?/is 02Engine? 只是"改过的参数报告积木"才兼容；线后每个积木都会让项目无法上传官网。
    = 以警告线为硬边界，用之前先判断是否专属积木。
  why_it_matters: |
    下游 skill 若把 Extra Blocks 当普通 Scratch 积木推荐，会产出无法上传 scratch.mit.edu 的项目；必须先做兼容分界自检。
  tags: [term, blocks, compatibility]

- id: g02
  term: Custom Reporter / return block（自定义报告积木 / 返回积木）
  type: term
  source_chapter: return
  author_definition: |
    "Custom Reporters let you use your own custom blocks as reporter blocks." 技术上"没有做到以前做不到的事，只是更易读写"；启用方式是把一个 "return" 积木拖入自定义积木，执行到它时"效果类似 stop this script"。
  key_distinction: |
    ≠ "新增了一种积木能力" — 文档明说只是让重复逻辑更易读写的语法糖。
    ≠ "需要在设置里勾选某种报告选项" — 只需拖入 return 积木，形状由 02Engine 自动猜（猜错可右键手动改）。
    = 可嵌入别的积木输入的自定义积木；布尔形状仅视觉辅助，任何自定义报告积木都能放进任何输入。
  why_it_matters: |
    skill 要正确指导"拖 return 积木即可"，并提醒此功能项目不与 Scratch 兼容、积木形状可能被猜错需手动改。
  tags: [term, blocks, reporter]

- id: g03
  term: Recursion / base case / stack overflow（递归 / 基线条件 / 栈溢出）
  type: term
  source_chapter: return
  author_definition: |
    "blocks can run themselves. If you can take a large problem and continuously break it down into smaller problems until you reach a known 'base case', many algorithms can be written very elegantly."
    "When using the compiler, beware of stack overflow errors from too much recursion. Recursing more than a few thousand calls deep typically will throw errors."
  key_distinction: |
    ≠ "递归层数是无限的" — 解释器支持无限递归；编译器受 JS 栈限制，约几千层即 stack overflow。
    ≠ "递归必须靠变量" — 可不用任何变量改写 repeat 积木，循环与递归可互相改写。
    = base case 是递归拆解的已知终止点；编译器下深度受限，是自定义报告积木的硬约束。
  why_it_matters: |
    skill 推荐递归方案时必须提示编译器下的栈深上限，避免产出在编译器模式崩溃的深层递归积木。
  tags: [term, blocks, recursion, compiler]

- id: g04
  term: short circuiting / side effects / compiler vs interpreter（短路 / 副作用 / 编译器与解释器）
  type: term
  source_chapter: return
  author_definition: |
    "the compiler has a feature known as short circuiting"：如 `<(1=2) and (3=3)>`，算到前半为 false 就跳过后半。"if your blocks have side effects such as moving sprites, different behaviors will occur as the interpreter always evaluates all blocks."
  key_distinction: |
    ≠ "短路只是免费性能优化" — 对有副作用（移动角色、说文字、等待）的积木，编译器会跳过，解释器却总全算，两者行为不一致。
    ≠ "两者执行顺序一致" — 深层嵌套报告积木的执行顺序在编译器/解释器之间也可能不同。
    = 自定义报告积木应"只算算法、输出最终答案"，无副作用，才能保证编译器与解释器一致。
  why_it_matters: |
    skill 必须把"纯函数式报告积木"作为硬规范，否则同一项目在编辑器和编译器下结果不同，用户难以排查。
  tags: [term, blocks, compiler, pitfall]

- id: g05
  term: #code marker / Direct JavaScript Coding（#code 标记 / 直接 JS 编码）
  type: term
  source_chapter: direct-javascript-coding
  author_definition: |
    "02Engine can compile a hat block from JavaScript written inside a comment. This is an advanced escape hatch." 给顶层帽子积木加注释，某行以 #code 开头，"Everything after the #code line is used as the JavaScript body for that script."
  key_distinction: |
    ≠ "普通注释" — 写在该脚本注释行首的 #code 会把注释后续内容变成该脚本的原生 JS body；标记必须在行首。
    ≠ "影响全部脚本" — 只影响含 #code 的那一个脚本，其他脚本正常编译。
    = 高级逃生舱，可访问 VM 内部；语法/不安全调用以 JS 或编译器错误暴露，而非 Scratch 积木错误。
  why_it_matters: |
    skill 指导写 #code 时必须强调"行首标记、只作用于本脚本、绕过积木编译"，否则用户会误把注释当普通笔记或以为全局生效。
  tags: [term, direct-js, escape-hatch]

- id: g06
  term: hat block / escape hatch（帽子积木 / 逃生舱）
  type: term
  source_chapter: direct-javascript-coding
  author_definition: |
    "hat block（帽子积木）"：脚本的起始积木（如"当绿旗被点击"），#code 通过它把注释 JS 编译成脚本 body。
    "escape hatch（逃生舱）"：文档对 Direct JavaScript Coding 的定位——绕过积木语法、直接写原生 JS 的高级出口。
  key_distinction: |
    ≠ 普通"脚本开头积木" — 在 #code 语境下它是 JS body 的挂载点；在扩展语境下 HAT 是一种 blockType（见 g28/g37）。
    ≠ "官方推荐的主流做法" — escape hatch 意味着高级、高风险、需自担让出与结束责任，纯积木用户不应轻易使用。
    = 帽子积木是脚本触发入口；逃生舱是"积木不够用时"的最后手段。
  why_it_matters: |
    下游 skill 需区分"帽子积木"在直接 JS（JS 挂载点）与扩展 blockType（HAT/EVENT）两处的不同含义，避免术语混用。
  tags: [term, direct-js, extension]

- id: g07
  term: opcode / generator / yield / retire / return（操作码 / 生成器 / 让出 / 结束）
  type: term
  source_chapter: direct-javascript-coding
  author_definition: |
    opcode 是积木的操作码名，`block` 是按 opcode 名解析的代理：`block.looks_sayforsecs({MESSAGE:'Hello',SECS:1})`。
    "代码含 yield 时按 generator 脚本编译"；`yield* wait(1000)` 实现延迟；"非常驻脚本须显式 retire(); return;，长运行 generator 可一直 yield"。
  key_distinction: |
    ≠ "yield/retire/return 是 Scratch 积木" — 它们是 #code 原生 JS 里的控制原语：yield 让出防冻结、retire 注销线程、return 退出脚本。
    ≠ "opcode 只是内部编号" — 它是 #code 里 `block.<opcode>({参数})` 调用积木的键，也是扩展积木对应方法名（见 g28）。
    = 直接 JS 脚本必须自己管理让出与结束，否则编辑器冻结或线程泄漏。
  why_it_matters: |
    skill 写 #code 脚本时必须生成 retire()+return() 收尾与长循环 yield，否则产出会冻结编辑器；opcode 是跨 #code 与扩展的通用概念。
  tags: [term, direct-js, api]

- id: g08
  term: Cloud Variables（云变量）
  type: term
  source_chapter: cloud-variables
  author_definition: |
    "The variable length limit has been increased to 100000 characters." 云变量"仍只能存数字"；服务器重启或短时无人会重置；编辑器打开时云变量被禁用；使用云变量或自定义扩展时视频侦测的颜色侦测被禁用。
  key_distinction: |
    ≠ "等价于 Scratch 官方云变量" — 这是 02Engine 独立服务器（wss://clouddata.02engine.org），协议相同但上限更高、行为有差异。
    ≠ "能存文本、能持久保存" — 只能存数字、10 万字符上限、会重置，排行榜存不久。
    = 独立于 Scratch 的多人数据能力；编辑器打开时禁用、影视侦测颜色侦测会被连带禁用。
  why_it_matters: |
    skill 设计排行榜/多人功能时必须提示"只存数字、会重置、编辑器内禁用"，否则方案在数据模型上就是错的。
  tags: [term, cloud-variables, multiplayer]

- id: g09
  term: User-Agent / Project ID（用户代理 / 项目 ID）
  type: term
  source_chapter: cloud-variables
  author_definition: |
    "Bots must provide a valid User-Agent header in their connection"：含联系方式与库名/版本，"human readable"；伪装浏览器"not okay and is easy to detect"。
    "Project IDs are not limited to just numbers -- they can be any text you want"，非 Scratch 项目用 `example.com/my-project` 以便验证，库选项"should be strings, not an integer"。
  key_distinction: |
    ≠ "User-Agent 是可选浏览器头" — 在云变量 bot 里是强制合规项，缺失/信息不全会被封禁；库须强制暴露设置 API。
    ≠ "Project ID 是数字" — 02Engine 中可为任意文本且库选项必须是字符串；无意义 ID 大量活动会被禁用。
    = 二者是 bot 合规与可追溯性的核心约束，不是可选配置。
  why_it_matters: |
    skill 生成云变量 bot 代码时必须带有效 User-Agent、用字符串 Project ID，否则 bot 会被拒连或封禁。
  tags: [term, cloud-variables, bot, compliance]

- id: g10
  term: event-driven API / buffering（事件驱动 API / 缓冲）
  type: term
  source_chapter: cloud-variables
  author_definition: |
    "libraries should offer event-driven APIs. Instead of constantly asking the server for the latest values, open exactly one WebSocket, and let the server send updates as they happen."
    "the server will buffer up several cloud variable updates to send out as one group. Updates are not guaranteed to be sent in the same order... some updates may be skipped entirely... sending updated variables more than 10 times per second is completely redundant."
  key_distinction: |
    ≠ "get_var 轮询是正常模式" — 文档推荐只开一个 WebSocket、由服务器推送；get_var 只能在内部用单连接实现。
    ≠ "更新有序、每条必达" — 缓冲后不保序、可能跳过；每秒超过 10 次更新完全冗余。
    = 事件驱动 + 单连接 + 回 pong + 限频是云变量 bot 的协议纪律。
  why_it_matters: |
    skill 生成 bot 若用轮询、频繁开关连接或高频更新，会造成连接被禁与更新丢失；必须内建事件驱动与限频规范。
  tags: [term, cloud-variables, bot, pitfall]

- id: g11
  term: Compiler / type conversion（编译器 / 类型转换）
  type: term
  source_chapter: new-compiler
  author_definition: |
    "the compiler analyzes scripts to determine what kinds of values each variable may have at each point in the script. This allows it to remove unnecessary type conversions and generate more specialized code."
  key_distinction: |
    ≠ "编译器只是把积木翻译成 JS" — 新编译器（2025-09-20 发布）核心是类型分析去除冗余类型转换，是史上最大单项改动。
    ≠ "一定更快" — 提速因项目而异（有的约两倍、有的不变），但不会变慢；编辑器内因 warp timer 无显著提升。
    = 提速来自静态类型推断去掉冗余转换；若 warp timer 启用或积木未标"不刷新屏幕运行"，生成代码与老编译器相同。
  why_it_matters: |
    skill 评估性能与解释"编辑器内没提速"时必须归因到 warp timer 抵消优化，并给出故障回退老编译器的路径。
  tags: [term, compiler, performance]

- id: g12
  term: warp timer / run without screen refresh（warp 计时器 / 不刷新屏幕运行）
  type: term
  source_chapter: new-compiler
  author_definition: |
    "The warp timer forces blocks marked as 'run without screen refresh' to briefly pause after running for 500 ms to prevent an infinite loop from causing you to lose unsaved work. This is why the warp timer is automatically enabled when you open the 02Engine editor."
    "the warp timer breaks many of the assumptions that the new compiler uses to optimize projects."
  key_distinction: |
    ≠ "纯性能优化开关" — 它是防无限循环丢失未保存工作的保护机制，打开编辑器时自动启用。
    ≠ "与新编译器协同提速" — 它破坏编译器优化假设，是编辑器内无显著提升的根因。
    = 强制"不刷新屏幕运行"积木跑 500ms 后暂停；要体现新编译器提速需在网站/打包环境。
  why_it_matters: |
    skill 解释"为什么编辑器里没变快"和"要不要关 warp timer"时必须指出它同时是保护机制与编译器提速的抵消项。
  tags: [term, compiler, performance, pitfall]

- id: g13
  term: OPF / Custom OPF / FPS / Turbo mode（每帧操作数 / 自定义每帧操作数 / 每秒帧数 / 极速模式）
  type: term
  source_chapter: opf
  author_definition: |
    "Operations Per Frame, or OPF, controls how much Scratch VM work can run during one visual frame. In 02Engine it is exposed as Custom OPF in Advanced Settings."
    FPS 控制运行时尝试产帧的频率，OPF 控制每帧内工作量；"同时提高二者会很快拉高 CPU"，"先调 FPS，不够再调 OPF"。
  key_distinction: |
    ≠ "OPF 就是 FPS" — FPS 是产帧频率，OPF 是每帧内 VM 工作量，二者是不同维度。
    ≠ "Turbo mode 等同于 OPF" — Turbo mode 是极速模式开关，调高 OPF 可让 Turbo 下脚本更快推进，但不是同一设置。
    = 调优顺序固定：先 FPS、再 OPF；重视觉/动画项目调高 OPF 会降低流畅度与编辑器响应。
  why_it_matters: |
    skill 做性能调优必须按"先 FPS 后 OPF"顺序，并提示二者同时拉高会迅速吃满 CPU，避免错误调参。
  tags: [term, performance, opf]

- id: g14
  term: Extension sources / 扩展七类来源
  type: term
  source_chapter: extension-library
  author_definition: |
    扩展库来源侧栏按元数据分组：Official Scratch、02Engine、TurboWarp、PenguinMod、Mist、SharkPool、Other（不匹配上述分组的已加载项）。默认按原列表/画廊顺序，不按字母重排，保留各来源策展顺序。
  key_distinction: |
    ≠ "全集只有一个扩展市场" — 是七类来源合并的统一工作台，保留各来源策展顺序。
    ≠ "Other 是第三方杂项" — Other 特指不匹配前六类的"已加载项"。
    = 来源分组是选型与排查扩展的第一维度，找不到的扩展先切来源、清过滤。
  why_it_matters: |
    skill 指导"按来源排查扩展"时须用七类分组而非字母序，否则用户在合并列表中定位不到目标扩展。
  tags: [term, extension-library, sources]

- id: g15
  term: Favorites / Batch import（收藏 / 批量导入）
  type: term
  source_chapter: extension-library
  author_definition: |
    Favorites：有 URL 时按扩展 URL 存储，否则按扩展 ID；重复条目在收藏区与来源区共享收藏状态。
    Batch import：可排队卡片角落有复选框，选后点 "Batch import X extensions"；逐个处理、单个失败不中断队列，结束给成功/失败汇总。
  key_distinction: |
    ≠ "收藏只是本地书签" — 有 URL 按 URL、无 URL 按 ID 存，跨来源区共享状态。
    ≠ "批量导入是并发" — 是逐个处理、单个失败不中断队列；原生扩展始终走 normal import；Custom Extension 卡故意不可批量选。
    = 收藏用于快速筛选，批量导入用于一次安装多个且有失败汇总。
  why_it_matters: |
    skill 批量安装扩展时要提示"Custom Extension 需手动加载、原生扩展强制 normal import、看失败汇总"，避免误判安装结果。
  tags: [term, extension-library, workflow]

- id: g16
  term: Compatibility Badge / Native extension（兼容性徽章 / 原生内置扩展）
  type: term
  source_chapter: extension-library
  author_definition: |
    "Native"：按 ID 加载的内置扩展；"Batch"：可入批量队列；"Not Scratch-compatible: the extension or feature creates projects that the Scratch website may not support."
    原生扩展不支持文本导入，始终走 VM 常规扩展路径。
  key_distinction: |
    ≠ "Not Scratch-compatible = 一定不能上传" — 文档措辞是 Scratch 官网"may not support"，仅提示可能不支持，不等于已验证必然失败。
    ≠ "Native 是可文本导入的扩展" — 原生扩展按 ID 加载、不支持文本导入，批量导入时即便选 text import 仍走 normal import。
    = 徽章是快速判断加载方式与兼容风险的视觉标记。
  why_it_matters: |
    skill 解释徽章时须用"可能不支持"而非"必然失败"，并区分 Native 的加载路径，避免给出错误兼容性断言。
  tags: [term, extension-library, compatibility]

- id: g17
  term: unsandboxed extension runner（非沙盒扩展运行器）
  type: term
  source_chapter: extension-library
  author_definition: |
    "02Engine's VM currently loads custom extension URLs through the unsandboxed extension runner. Only install extensions from sources you trust. Unsandboxed extensions can access powerful browser and VM APIs that normal Scratch projects cannot."
  key_distinction: |
    ≠ "扩展默认在沙箱里更安全" — 02Engine 当前通过非沙箱 runner 加载自定义扩展 URL，扩展可触及普通 Scratch 项目碰不到的强 API。
    ≠ "只是性能选项" — 这是信任/安全边界：加载外部 URL 等于赋予强权限，故只装可信来源。
    = 非沙箱运行器是"能力与风险同源"的设计，外部 URL 扩展须显式信任。
  why_it_matters: |
    skill 安装/推荐外部扩展 URL 时必须先做来源可信判断，并提示用户此举等同于授予强浏览器/VM 权限。
  tags: [term, extension-library, security, trust]

- id: g18
  term: Scratch JS DSL / Virtual file（Scratch JS 表达法 / 虚拟文件）
  type: term
  source_chapter: 02agent-addon
  author_definition: |
    项目被映射为虚拟文件，AI 改后同步回 VM。常见路径 /stage.js、/sprites/<角色名>.js、/sprites/<角色名>/costumes/<编号>-<造型名>.svg 等。脚本用 Scratch JS DSL 表达，如 `// @script new-hello` + `event.whenflagclicked(...)`；编辑已有脚本须保留 `// @script` 标记与 `// blockId` 注释以利同步。
  key_distinction: |
    ≠ "真实磁盘文件" — 虚拟文件是项目在编辑器内的映射视图，AI 改的是它再同步回 VM，不是本地文件系统。
    ≠ "DSL 可随意改写脚本" — 须保留 // @script 标记与 // blockId 注释，否则改后无法同步回积木。
    = 虚拟文件 + DSL 是 02Agent 读写项目脚本的中间表示。
  why_it_matters: |
    skill 用 02Agent 改脚本时必须保留标记注释并理解虚拟文件路径，否则改动无法同步回积木工作区。
  tags: [term, agent, dsl, virtual-file]

- id: g19
  term: getDiagnostics / applyPatch（诊断 / 补丁编辑）
  type: term
  source_chapter: 02agent-addon
  author_definition: |
    getDiagnostics：诊断 DSL 能否同步回积木，查语法、字段名、输入类型、菜单值；常见失败如漏 `$field_`、把 Boolean 写成普通变量、用未加载扩展的 opcode。
    applyPatch：02Agent 的编辑类工具，Codex 风格 patch。改项目状态的工具会串行排队，避免聊天与 bridge 冲突。
  key_distinction: |
    ≠ "getDiagnostics 是通用lint" — 专用于查 Scratch JS DSL → 积木的同步失败，错误往往在字段/类型/菜单值映射上。
    ≠ "applyPatch 是文本替换" — 是按 Codex 风格 patch 改项目，且改状态工具串行排队以防并发冲突。
    = 二者构成 02Agent"小步改 + 改后诊断确认"工作流的核心工具。
  why_it_matters: |
    skill 必须把"改后跑 getDiagnostics 确认同步"作为强制步骤，否则 AI 改动可能静默失败、不同步回积木。
  tags: [term, agent, tool, workflow]

- id: g20
  term: Skill bridge / WebSocket / Token（技能桥 / WebSocket / 令牌）
  type: term
  source_chapter: 02agent-addon
  author_definition: |
    浏览器侧 WebSocket 客户端连本机 bridge server，让本地 AI 客户端经 HTTP 调用编辑器工具。WS 默认 `ws://127.0.0.1:40202/agent?token=<token>`，HTTP 默认 `http://127.0.0.1:40202`。manifest 公开工具 schema 等，但不公开 API Key/模型密钥；bridge 只应监听 127.0.0.1。
  key_distinction: |
    ≠ "远程云服务" — bridge 是本机 WebSocket 桥，只应监听 127.0.0.1；浏览器侧不开 HTTP 端口，只做出站 WS 连接。
    ≠ "token 是完整鉴权" — token 只挡无关 WS，不防本机其它进程；不公开 API Key，别暴露到非本机网络。
    = 让外部 AI 客户端复用编辑器里同一套工具，中文参数推荐 stdin/base64 而非直接放 argv。
  why_it_matters: |
    skill 指导连接外部 AI 时必须强调只监听本机、token 边界与中文参数编码，避免安全暴露与传参失败。
  tags: [term, agent, bridge, security]

- id: g21
  term: Manifest（清单）
  type: term
  source_chapter: 02agent-addon
  author_definition: |
    清单文件。自定义 addon 须含合法 manifest 并声明 userscripts/styles/assets/settings。02Agent 的 bridge manifest 公开协议/版本、工具 schema、系统提示、项目概览、安全声明；不公开 API Key、模型密钥、localStorage 完整配置。有 `GET /manifest`、`POST /reloadManifest`。
  key_distinction: |
    ≠ "只是元数据说明" — manifest 是资源声明与对外能力边界：决定哪些工具/信息对外可见，哪些密钥必须隐藏。
    ≠ "一次性静态文件" — 支持 reloadManifest 热更新；改 schema/bridge/prompt 要同时手验聊天与 bridge 调用。
    = 清单既定义 addon 组成，也定义 bridge 的公开/隐藏边界。
  why_it_matters: |
    skill 配置 bridge 或写 addon 时必须按 manifest 声明资源、保护密钥，否则会泄露密钥或加载失败。
  tags: [term, agent, addon, api]

- id: g22
  term: Extension Debugger / VSCode Toolbox / Hot reload（扩展调试器 / VSCode 工具箱 / 热重载）
  type: term
  source_chapter: extension-debugger
  author_definition: |
    GUI 加载 `ScratchExtensionDebug`，连接 `ws://localhost:1101`；本地工具发扩展源码，编辑器作为自定义扩展加载、刷新分类与工作区。代码转 `data:application/javascript` URL 经 `vm.extensionManager.loadExtensionURL` 加载，检测 ID、移除旧注册与积木定义后加载新版——专为开发设计，比常规加载更激进。
  key_distinction: |
    ≠ "VSCode 普通调试插件" — 是经 1101 端口 WebSocket 把扩展代码热推到运行中的 02Engine 编辑器的桥。
    ≠ "安全的常规加载" — 热重载会碰 VM 内部，语法错误后可能留陈旧状态，需切分类或刷新编辑器。
    = 本地开发专用的快速迭代通道，不要把 WS server 暴露到不可信网络。
  why_it_matters: |
    skill 指导扩展调试须给出"启动 Toolbox(1101)→编辑器 Connect→推送→切分类刷新"流程及连不上/残留的排障。
  tags: [term, extension-debug, hot-reload, tool]

- id: g23
  term: "loadExtensionURL / data: URL（按 URL 加载扩展 / data URL）"
  type: term
  source_chapter: extension-debugger
  author_definition: |
    热重载机制：代码转 `data:application/javascript` URL → `vm.extensionManager.loadExtensionURL` 加载 → 检测扩展 ID → 若已加载先移除旧注册与积木定义 → 加载新版并刷新 toolbox/workspace。
  key_distinction: |
    ≠ "只是从网址加载扩展" — loadExtensionURL 也接受 data: URL（把源码内联成 URL），用于调试桥免文件直推代码。
    ≠ "加载即完成" — 调试重载需先移除旧注册/积木定义，否则残留陈旧状态。
    = data: URL 是热重载把源码变成可加载扩展的载体；loadExtensionURL 是 VM 的加载入口。
  why_it_matters: |
    skill 解释热重载原理与"旧扩展残留"排障时须点明 loadExtensionURL + data: URL 的机制，而非笼统说"刷新即可"。
  tags: [term, extension-debug, api]

- id: g24
  term: CLI App / CLI Helper API（命令行应用 / 命令行辅助 API）
  type: term
  source_chapter: convert-cli
  author_definition: |
    用 Packager 把 .sb3 打包成 Node.js CLI，在 Node.js 环境用 scratch-vm 运行。CLI Helper API：`cli.log/error/warn/info(message)` 输出，`cli.exit(code)` 带状态码退出，`cli.getArgs()` 取全部参数，`cli.getArg(key)` 按 key 取一个。
  key_distinction: |
    ≠ "GUI 项目的命令行版" — 无图形舞台输出，鼠标键盘交互受限、音频视环境受限、依赖浏览器 API 的扩展可能无法工作。
    ≠ "用 process.argv 直接读参" — 文档提供专门的 cli.getArg/getArgs 辅助 API 读取运行时参数。
    = CLI 化适合自动化/服务端/模拟/测试，须 Node.js 18+，构建命令以包内 README 为准。
  why_it_matters: |
    skill 判断项目能否 CLI 化、并生成读参代码时须用 CLI Helper API 且提示无舞台依赖，否则产出的 CLI 工具无法运行。
  tags: [term, cli, packaging]

- id: g25
  term: LeopardJS / HTMLify / vm.enableDebug()（可读 JS / 网页化 / 启用调试打印）
  type: term
  source_chapter: javascript
  author_definition: |
    想打包/HTMLify 项目用 packager.02engine.org；想把 Scratch 转成"可读可编辑"的 JavaScript 用 leopardjs.com，"不要用 02Engine"。02Engine 生成的代码不是给人读或编辑的，无格式化、含 runtime 私有函数。真要看生成代码：启动前在控制台先跑 `vm.enableDebug();`。
  key_distinction: |
    ≠ "02Engine 能导出可读 JS" — 文档明确反对：生成代码为兼容/性能非常规处理、无格式、含私有 API，强行阅读妨害学习。
    ≠ "vm.enableDebug() 是导出方案" — 它只把编译时 JS 打到控制台，仅用于调试查看。
    = 三条路径按目的分：网页化→packager、可读 JS→leopardjs、看生成代码→enableDebug。
  why_it_matters: |
    skill 必须按用户目的选对 JS 路径，并解释"为何不改 02Engine 生成 JS"，否则会给用户不可维护的代码。
  tags: [term, javascript, tooling, pitfall]

- id: g26
  term: Packager（打包器）
  type: term
  source_chapter: convert-cli
  author_definition: |
    02Engine Packager：选 .sb3 → Open Environment → Other environments > Node.js CLI 打包下载。文档另注：仅自用且用 Packager 分享项目时无需提交扩展，因为"Packager always runs extensions unsandboxed"。
  key_distinction: |
    ≠ "只是导出文件" — Packager 打包时总是非沙箱运行扩展，因此自用打包可绕过官方扩展库提交。
    ≠ "等于 LeopardJS" — Packager 产出是可运行产物（网页/CLI/桌面），不是给人读的源码。
    = 打包路径选择影响扩展运行方式（非沙箱）与分发方式。
  why_it_matters: |
    skill 在发布/分发决策中须说明 Packager 的非沙箱特性，避免用户误以为打包后扩展仍在沙箱或获得可读源码。
  tags: [term, packaging, distribution]

- id: g27
  term: getInfo() / Scratch.extensions.register
  type: term
  source_chapter: extensions/hello-world
  author_definition: |
    标准骨架 `class MyExtension { getInfo(){...} hello(){...} }` + `Scratch.extensions.register(new MyExtension());`。
    "Make sure to always call register() exactly once. If you don't call it, your extension will never get added... If you call it multiple times, the behavior is undefined."
    getInfo() 返回元数据 id/name/blocks/menus 等。
  key_distinction: |
    ≠ "register 可多次调用以注册多个积木" — 必须且只能调用一次，多次行为未定义。
    ≠ "getInfo 只是起说明作用" — 它是引擎读取扩展元数据的关键方法，决定 id/积木列表/菜单。
    = 最小扩展骨架的两个必备部件；不调 register 永远加载不上。
  why_it_matters: |
    skill 生成扩展骨架必须保证 register 恰好一次、getInfo 完整返回元数据，这是扩展能加载的硬性前提。
  tags: [term, extension-dev, api]

- id: g28
  term: id / opcode / blockType（COMMAND/REPORTER/BOOLEAN/HAT/EVENT）
  type: term
  source_chapter: extensions/hello-world
  author_definition: |
    id：a-z/0-9、唯一、不能含空格/特殊字符；name 缺省=id。opcode：对应方法名、扩展内唯一。blockType：COMMAND（不返回值）、REPORTER（圆角，返回 string/number/boolean）、BOOLEAN（尖角，返回布尔）、HAT、EVENT（后两者另述）。"null/undefined/列表/对象都不是合法返回值"。
  key_distinction: |
    ≠ "blockType 可随意选" — 只有这五种被讨论，其他 BlockType 效果差；REPORTER/BOOLEAN 返回值类型严格受限。
    ≠ "id/opcode 可含任意字符" — id 仅限 a-z/0-9 且唯一，opcode 扩展内唯一，二者是兼容性的关键不变量（见 g38）。
    = 五种 blockType 决定积木形状与行为；id/opcode 是扩展身份标识。
  why_it_matters: |
    skill 生成扩展定义必须遵守 id 字符限制、opcode 唯一、blockType 合法与返回值约束，否则扩展无法加载或报错。
  tags: [term, extension-dev, api, compatibility]

- id: g29
  term: arguments / text / [ARG] / menus / acceptReporters / Scratch.ArgumentType.*（参数与菜单）
  type: term
  source_chapter: extensions/inputs
  author_definition: |
    arguments 是"对象套对象"，text 中用 `[ARG_NAME]` 占位，每个参数恰好出现一次。
    "Remember: the argument's 'type' is just a suggestion for the editor; it's never enforced." ArgumentType 含 STRING/NUMBER/BOOLEAN/COLOR/ANGLE/MATRIX/NOTE/IMAGE/COSTUME/SOUND。
    静态菜单：type 设 STRING、menu 指向菜单名，顶层 menus 含 items 与 acceptReporters（几乎总设 true）。
  key_distinction: |
    ≠ "type 会强制输入类型" — 只是建议、从不强制，实际值可能是 string/number/boolean，代码须自行转换（如 args.TEXT.toString()）。
    ≠ "field（固定字符串）够用" — 几乎总要用 input + acceptReporters:true；写菜单对象为数组会隐式 acceptReporters:false，field↔input 互换是破坏兼容的改动。
    = 参数定义三件套：arguments 定义、text 占位、menus 下拉；type 不强制是关键。
  why_it_matters: |
    skill 生成参数/菜单时必须自行做类型转换、几乎总设 acceptReporters:true，否则积木行为错误且可能破坏兼容。
  tags: [term, extension-dev, api, pitfall]

- id: g30
  term: Promise / async / catch()（异步积木）
  type: term
  source_chapter: extensions/async
  author_definition: |
    异步块靠 Promise 告知 Scratch 正在等待："Promise resolve 前当前脚本不继续，但其他脚本可继续"。setTimeout 不返回 Promise 需手动 new Promise；fetch() 自带 Promise 可直接链式。Promise 可能 reject 时必须加 `.catch()`：记录日志并返回错误提示字符串。
  key_distinction: |
    ≠ "异步会阻塞整个编辑器" — Promise 等待时只暂停当前脚本，其他脚本继续运行。
    ≠ "fetch 返回的 Response 可直接当返回值" — 须用 text() 转成字符串以满足 REPORTER 返回值限制；COMMAND 块 resolve 不带值。
    = 异步块定义方式与同步块相同，唯一区别是返回 Promise，且必须处理 reject。
  why_it_matters: |
    skill 生成异步积木必须加 catch、把 Response 转字符串，否则未捕获 reject 或非法返回值导致积木出错。
  tags: [term, extension-dev, async, api]

- id: g31
  term: Sandbox(iframe) / Unsandboxed(script) / IIFE / 'use strict'
  type: term
  source_chapter: extensions/sandbox
  author_definition: |
    "The sandbox is actually an <iframe>"（跨域隔离），不能访问 internals/未传入的项目部分/角色，且"every time a block runs, the script will pause for at least 1 frame... It doesn't matter whether 'run without screen refresh' or turbo mode is enabled"。
    非沙箱"run in the same context as Scratch itself as <script> tags"，瞬时无 1 帧延迟。非沙箱模板整体包进 IIFE：`(function(Scratch){ 'use strict'; ... })(Scratch);`，防止变量泄漏与扩展间同名干扰。
  key_distinction: |
    ≠ "沙箱只是安全隔离" — 它还有性能代价：每积木至少 1 帧延迟，turbo/"不刷新屏幕运行"都无效，使自定义扩展几乎不可用。
    ≠ "非沙箱只是去掉限制" — 它意味着更大责任（不得抛错/死循环，否则冻结窗口丢数据），且 URL 须精确为 localhost:8000 或 extensions.02engine.org 开头，127.0.0.1/0.0.0.0 无效。
    = 沙箱=iframe(安全+1帧延迟)、非沙箱=script(瞬时+全权限)；IIFE+严格模式是非沙箱的隔离骨架。
  why_it_matters: |
    skill 帮用户做沙箱决策必须讲清"要不要访问 internals/变量/角色"与"能否接受 1 帧延迟"，并给出 IIFE 骨架与 URL 白名单。
  tags: [term, extension-dev, sandbox, security]

- id: g32
  term: Scratch.vm / BlockUtility / util.target
  type: term
  source_chapter: extensions/unsandboxed
  author_definition: |
    访问 internals：`const vm = Scratch.vm;`，所有变量声明必须在 IIFE 内。积木函数第二个参数 `util`（BlockUtility），`util.target` 是当前真实角色对象。
    "该对象是所有脚本/积木共享的，每次只更新属性：只能在积木同步执行时访问，在 setTimeout/Promise 回调中不可靠——需提前把属性存入变量。"
  key_distinction: |
    ≠ "util.target 是可随时读取的角色引用" — 它是共享对象、每次只更新属性，异步回调中不可靠，须提前存值。
    ≠ "Scratch.vm 是全局可直接用" — 非沙箱中变量声明必须在 IIFE 内，且须用各自 Scratch API 副本。
    = util.target 是非沙箱访问"当前角色"的入口，但只能在同步执行时读。
  why_it_matters: |
    skill 生成非沙箱积木访问角色时，必须提示在异步前先缓存 util.target 属性，否则读到错误角色数据。
  tags: [term, extension-dev, api, pitfall]

- id: g33
  term: Permissioned APIs（权限化 API）
  type: term
  source_chapter: extensions/unsandboxed
  author_definition: |
    非沙箱扩展访问远程服务前应请求许可（extensions.02engine.org 强制要求，运行时无技术强制），需优雅处理用户拒绝（表现如同断网）。用 `Scratch.fetch(url)` 代替 fetch()；WebSocket/Image/Audio 前先 `await Scratch.canFetch(url)`；`Scratch.openWindow/redirect` 代替 window.open / location.href，配合 canOpenWindow/canRedirect。
  key_distinction: |
    ≠ "非沙箱就没有限制" — 远程访问走权限化 API，访问前须 canFetch/canOpenWindow/canRedirect 请求许可。
    ≠ "权限只是形式" — 这些 API 也阻止 `javascript:` URL 等任意 JS 执行；用户拒绝时须表现得像断网一样优雅降级。
    = 权限化 API 是非沙箱"能力换责任"的安全闸门，扩展须处理拒绝。
  why_it_matters: |
    skill 生成非沙箱远程请求代码必须改用 Scratch.* 权限化 API 并处理拒绝，否则在官方库无法提交或触发安全拦截。
  tags: [term, extension-dev, security, api]

- id: g34
  term: Scratch.Cast.*（类型转换与比较）
  type: term
  source_chapter: extensions/assorted-apis
  author_definition: |
    "The way that Scratch handles operations such as type conversions or comparisons has numerous quirks. Instead of trying to write them yourself, you can use the Scratch.Cast.* APIs." 含 toNumber、toString、toBoolean、compare；compare 返回与 0 的比较，必须用 `<0/ >0/ ===0`，不能用 `===1/ ===-1`。
  key_distinction: |
    ≠ "JS 原生转换/比较即可" — Scratch 的类型转换有大量怪癖，应用 Scratch.Cast.* 而非自己写。
    ≠ "compare 返回 1/0/-1" — 它返回与 0 的比较，必须用 <0/>0/===0 判断，用 ===1/===-1 是错的。
    = 复刻 Scratch 语义的类型转换/比较工具集，沙箱与非沙箱皆可用。
  why_it_matters: |
    skill 写扩展类型转换须用 Scratch.Cast.* 且按 compare 的 0 比较语义判断，否则转换行为与 Scratch 不一致。
  tags: [term, extension-dev, api, pitfall]

- id: g35
  term: hideFromPalette / filter / isTerminal
  type: term
  source_chapter: extensions/assorted-apis
  author_definition: |
    hideFromPalette：从工具箱隐藏积木但保留已有副本继续工作（利于向后兼容）。filter：数组含 Scratch.TargetType.STAGE/SPRITE 限定可见目标，但拖放/背包仍可绕过，代码里仍要用 util.target.isStage 校验，"不要用 filter: []，改用 hideFromPalette"。isTerminal: true 阻止在 COMMAND 积木下继续连接积木，"仅影响外形连接，不会停止脚本/thread"。
  key_distinction: |
    ≠ "filter 能可靠限制可见目标" — 拖放/背包可绕过，须用 util.target.isStage 兜底；空数组无效。
    ≠ "isTerminal 会停止脚本" — 只影响外形连接，不停止脚本/线程。
    ≠ "hideFromPalette 是删除积木" — 只是从工具箱隐藏，已有副本继续工作，是兼容友好手段。
    = 三者是积木"外观/可见性/连接性"控制，不等于运行逻辑控制。
  why_it_matters: |
    skill 隐藏旧积木、限制目标、做末级积木时须用对这几个属性并加 util.target.isStage 兜底，否则既有项目被破坏或限制失效。
  tags: [term, extension-dev, api, compatibility, pitfall]

- id: g36
  term: startHats / util.startHats / BEFORE_EXECUTE
  type: term
  source_chapter: extensions/hats
  author_definition: |
    Event block 用 `Scratch.BlockType.EVENT`，不写对应函数，isEdgeActivated:false 为必需，用 startHats 触发。两个版本：块外 `Scratch.vm.runtime.startHats`，块内 `util.startHats`；"块内误用 runtime 版会破坏脚本执行"。BEFORE_EXECUTE 是 VM 事件，脚本执行前触发，用于谓词帽自启动。
  key_distinction: |
    ≠ "startHats 只有一个" — 块外/块内版本不同，块内必须用 util.startHats，误用 runtime 版会破坏脚本执行。
    ≠ "startHats 第一参数是短 opcode" — 须用完整 opcode（extensionid_opcode，如 eventexampleunsandboxed_whenSpacePressed）。
    ≠ "BEFORE_EXECUTE 是积木" — 它是 VM 事件，用于谓词帽在脚本执行前自启动。
    = startHats 是主动触发 event/hat 脚本的机制；按菜单 value（非 text）与角色过滤。
  why_it_matters: |
    skill 生成 event/hat 触发代码必须区分块内外 startHats 版本与完整 opcode，否则脚本执行被破坏或触发不到。
  tags: [term, extension-dev, hats, api, pitfall]

- id: g37
  term: predicate hat / edge-activated hat / isEdgeActivated / shouldRestartExistingThreads
  type: term
  source_chapter: extensions/hats
  author_definition: |
    谓词帽（Predicate-based hats）：仅非沙箱，BlockType.HAT，必须定义返回 true/false 的函数，条件"为真"时运行，需自己调用 startHats，isEdgeActivated:false 必需，每帧只运行一次。
    边沿触发帽（Edge-activated hats）：条件"变为真"时运行，任何扩展（含沙箱）可用，isEdgeActivated:true，Scratch 每帧开头自动 startHats、无需手动调用、也不要用 shouldRestartExistingThreads。
  key_distinction: |
    ≠ "谓词帽与边沿触发帽是一回事" — 谓词帽"为真即运行"、仅非沙箱、需手动 startHats、每帧一次；边沿帽"变为真才运行"、沙箱也能用、自动启动。
    ≠ "所有 HAT 都用 shouldRestartExistingThreads" — 边沿触发帽明确不要用该标记；只有 event/predicate 场景才涉及重启已有线程。
    ≠ "HAT 与 EVENT 相同" — EVENT 响应外部事件、自身从不执行、仅非沙箱；HAT 带条件判断。
    = 三类触发积木（EVENT/predicate HAT/edge-activated HAT）适用面与启动方式不同。
  why_it_matters: |
    skill 选帽积木类型时必须按"外部事件/条件为真/条件变真"与沙箱与否区分，并正确设置 isEdgeActivated、是否手动 startHats。
  tags: [term, extension-dev, hats, api]

- id: g38
  term: Backward compatibility（向后兼容 / 只增不改）
  type: term
  source_chapter: extensions/compatibility
  author_definition: |
    核心原则："破坏兼容等于损坏项目。" 绝不改动：扩展 ID、积木 opcode 与类型、参数 ID/类型；绝不删除积木（改建新积木、旧积木标 hideFromPalette:true）；绝不向已有积木添加参数；不改 isTerminal、不改 acceptReporters；不显著改变行为。必须破坏时"新建完全不同 ID 的扩展，旧版原样保留（如 fetch → fetch2）"。
  key_distinction: |
    ≠ "改个显示名/颜色也算破坏兼容" — name/docsURI/color/图标、text（参数不变时）、disableMonitor 等属于"可以改"清单。
    ≠ "修 bug 必须绝对禁止" — 小 bug 修复通常 OK，靠充分测试；删除菜单项危险，增加菜单项 OK。
    = 向后兼容的本质是保护"已存在项目"的序列化数据，ID/opcode/类型/参数是绝不碰的不变量。
  why_it_matters: |
    skill 迭代扩展时必须按"只增不改"清单操作，否则会损坏用户已保存的项目；必须破坏时新建 ID 而非改旧扩展。
  tags: [term, extension-dev, compatibility, core-concept]

- id: g39
  term: Addon / Active Editor / DOM Discovery（编辑器插件 / 活动编辑器 / DOM 发现）
  type: term
  source_chapter: development/addons
  author_definition: |
    Addon 跑在编辑器 UI 里、不在 Scratch 项目内部。Active Editor Rule：只有活动编辑器窗口有可写 Blockly workspace，非活动窗口是快照/只读。DOM Discovery：用 tab.waitForElement 等助手和可重跑 observer，别存永久 DOM 引用；UI 插入要幂等（先查存在、用稳定 class/data 属性、活动编辑器变化时清除过期元素）。
  key_distinction: |
    ≠ "addon 改的是 Scratch 项目" — 它改的是编辑器 UI（按钮/面板/主题/调试），跑在 GUI 页面而非项目内部。
    ≠ "DOM 查询一次永久有效" — New UI 会移动/重挂载 DOM，硬编码引用失效，必须元素发现 + 幂等注入。
    ≠ "所有窗口都能写" — 只有一个活动编辑器可写，非活动窗口是快照。
    = addon 开发的核心约束来自 New UI 的窗口化与 DOM 重挂载。
  why_it_matters: |
    skill 写/装 addon 必须遵循活动编辑器唯一可写与幂等 DOM 注入，并双 UI（Old/New）测试，否则 addon 在窗口化界面失效。
  tags: [term, addon, editor, pitfall]

- id: g40
  term: ScratchX / .sbx（旧扩展体系 / 项目格式）
  type: term
  source_chapter: development/scratchx
  author_definition: |
    "02Engine 对 ScratchX 扩展和 .sbx 项目有初级支持。" ScratchX 是 Scratch 团队基于 Scratch 2 的、允许非官方扩展的版本，因 Flash 移除而失效、2023 年初关站。ScratchX 扩展运行在扩展沙箱中，硬件相关功能不可用，很多依赖的旧 web 服务已失效。用法：找到扩展 JS 源码 URL，像普通自定义扩展一样加载。
  key_distinction: |
    ≠ "完全支持 ScratchX" — 只是"初级支持"，硬件功能不可用、依赖的旧 web 服务多已失效。
    ≠ "是一种新扩展标准" — 是已关站的旧体系（2023 初），属于生态背景而非推荐路径。
    = 加载方式类似普通自定义扩展（找 JS 源码 URL），但能力受限。
  why_it_matters: |
    skill 遇到 .sbx/ScratchX 时应说明其过时与功能受限，避免用户把它当主流扩展方案。
  tags: [term, legacy, scratchx, background]

- id: g41
  term: Bun / Redux action（仓库脚本运行器 / GUI 状态分发）
  type: term
  source_chapter: development/getting-started
  author_definition: |
    仓库开发用 Bun 作为常规包/脚本运行器："除非特定 README 说明，不要用 npm install/run/test"。Redux action：GUI 设置流为 React 组件 → Redux action → VM/renderer 应用，如 FPS→setFramerate、OPF→setOpsPerFrame、编译器→setCompilerOptions；"GUI 不应直接重实现 VM 行为，应调用 VM API 或 dispatch Redux action"。
  key_distinction: |
    ≠ "仓库用 npm" — 文档明确用 Bun，混用包管理器会出问题。
    ≠ "GUI 可直接改 VM 状态" — 应通过 dispatch Redux action 或调用 VM API，而非重实现 VM 行为。
    = Bun 是仓库统一运行器；Redux action 是 GUI 到 VM 的状态传递通道。
  why_it_matters: |
    skill 涉及仓库开发或 GUI 改设置时必须用 Bun、走 Redux action/VM API，否则构建失败或绕过 VM 状态管理。
  tags: [term, development, tooling, background]
```
