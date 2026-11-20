---
name: inline-js
description: |
  当用户要在 02Engine 的积木项目里用注释直接写JavaScript、用JS调积木/读写变量，或#code脚本卡住/报错时调用。
  核心: 在顶层帽子积木注释的行首写#code，之后内容作为该脚本的JS body(逃生舱)； 可用变量target/runtime/stage/thread/p0..、block按opcode调积木、vars按名读写变量列表、wait延迟； 含yield按generator编译、yield* wait(ms)延迟；非常驻脚本末尾须retire();return;，长循环加yield防冻结； 只影响该脚本、依赖此功能不与Scratch兼容、只用理解且信任的代码。
  不适用于: 写独立扩展(用 extension-authoring)、自定义报告积木(用 custom-reporter)。
  Triggers: "#code"/内嵌JS/直接写JS/target/opcode/block/vars/yield/retire/generator/逃生舱
metadata:
  cangjie.generated-by: cangjie-tools v2.5.0
  cangjie.capability-id: cap.e2.direct-js
  cangjie.capability-revision: 1
  cangjie.bundle-id: bundle.02engine-guide
  cangjie.source-title: 02Engine 文档（积木搭建与扩展使用）
  cangjie.tags: javascript, escape-hatch, advanced
---
<!-- capability_id: cap.e2.direct-js | revision: 1 | status: active -->

# 注释内嵌 JS 逃生舱（#code）

## R — 原文 (Reading)

> "02Engine can compile a hat block from JavaScript written inside a comment. This is an advanced escape hatch for experiments, debugging, and high-control projects. Add a comment to a top-level hat block and start a line with #code. Everything after the #code line is used as the JavaScript body for that script."
>
> 译文：02Engine 能把写在注释里的 JavaScript 编译成帽子积木，这是用于实验、调试与高控制项目的高级"逃生舱"。给顶层帽子积木加注释，让某行以 `#code` 开头，`#code` 之后的所有内容都作为该脚本的 JS body。
>
> — 02Engine 功能文档·direct-javascript-coding / slug: direct-js（原文行95/100/106；源：candidates/framework.md f06、principle.md p05、case.md c01）

> "block is a proxy that resolves opcode names through the runtime. Use vars.stage(name, type) or vars.target(name, type). For normal variables, the second argument can be omitted. For lists, pass 'list'."
>
> 译文：`block` 是经 runtime 按 opcode 名解析的代理；用 `vars.stage(name, type)` 或 `vars.target(name, type)` 读写变量，普通变量省略第二参数，列表传 `'list'`。
>
> — 02Engine 功能文档·direct-javascript-coding / slug: direct-js（原文行140/162；源：candidates/framework.md f07、case.md c02/c03）

> "If the code contains yield, 02Engine compiles it as a generator script. Use yield* wait(ms) for a delay. For scripts that do not run forever, explicitly retire and return. Add yield in long loops so the editor does not freeze."
>
> 译文：代码含 `yield` 时按 generator 脚本编译；延迟用 `yield* wait(ms)`；非常驻脚本须显式 `retire(); return;` 结束；长循环要加 `yield`，否则编辑器会冻结。
>
> — 02Engine 功能文档·direct-javascript-coding / slug: direct-js（原文行179/204/214/224；源：candidates/framework.md f08、principle.md p06/p07、case.md c04）

## I — 方法论骨架 (Interpretation)

把"在积木项目里直接写 JS"当成受约束的逃生舱，而非写独立扩展：
① **入口**：给顶层帽子积木加注释，某行行首写 `#code`，其后全部内容即该脚本的 JS body；标记必须在行首，之前的注释行仍可作笔记或 TurboWarp 编译标志；
② **可用变量**：脚本工厂注入 `target`（运行角色）/`runtime`（VM 运行时）/`stage`（舞台）/`thread`（当前线程）/`p0,p1,...`（过程参数）/`block`/`vars`/`wait`；
③ **调积木**：`block` 是按 opcode 名经 runtime 解析的代理，`block.looks_sayforsecs({MESSAGE,SECS})`，高级用法手动传上下文 `{target}`；
④ **读写变量**：`vars.stage(name,type)` / `vars.target(name,type)`，普通变量省略 type、列表传 `'list'`，改值后置 `_monitorUpToDate = false`；
⑤ **让出与延迟**：含 `yield` 按 generator 编译，长循环加 `yield` 防冻结，`yield* wait(ms)` 延迟；
⑥ **结束契约**：非常驻脚本结尾显式 `retire(); return;`（对应编译器清理路径），长运行 generator 可一直 yield；
⑦ **作用域与错误**：只影响含 `#code` 的那一个脚本，其他脚本正常编译；绕过该脚本积木编译后，语法错误/不安全 VM 调用以 JS/编译器错误暴露，而非 Scratch 积木错误。
核心洞察：`#code` 是"单脚本、原生 JS、直触 VM 内部"的高控制通道——用注入的稳定助手（block/vars/wait）而非私有字段，用 yield/retire 补齐让出与清理，代价是不兼容 Scratch 且须自担信任风险。

## A1 — 书中的应用 (Past Application)

### 案例一：#code 最小骨架 + 调积木与读写变量（worked_example，slug: direct-js / c01+c02+c03）
- **问题**：想在积木项目注释里直接写原生 JS 做高控制逻辑，同时要能调用普通 Scratch 积木、读写项目中的变量与列表，而不受积木语法约束。
- **方法执行**：给顶层帽子积木加注释、行首写 `#code`，其后为 JS body——文档示例用 `console.log('Hello from 02Engine')` 与 `target.setXY(100, 50)` 起步；用 `block.looks_sayforsecs({MESSAGE:'Hello', SECS:1})` 调积木（高级用法 `block.motion_gotoxy({X:0,Y:0},{target})`）；用 `const score = vars.stage('score'); score.value += 1; score._monitorUpToDate = false;` 读写普通变量，用 `vars.target('items','list').value.push('new item')` 读写列表；结尾 `retire(); return;`。
- **结论**：文档所述效果——02Engine 把该注释编译为该脚本的 JS body，可访问 target/runtime/stage 等内部并通过 block/vars 助手调积木、读写变量列表；非常驻脚本须 `retire()+return()` 结束，改值后须置 `_monitorUpToDate = false` 刷新监视器。

### 案例二：yield 长循环、wait 延迟与正确结束（worked_example，slug: direct-js / c04）
- **问题**：#code 脚本要持续行动（每帧右移 4 像素），或先说一句话、延迟一秒后再说一句，且不冻结编辑器、能正确结束。
- **方法执行**：代码含 `yield` 即按 generator 编译——持续动作用 `while(true){ target.setXY(target.x+4, target.y); yield; }`；延迟用 `block.looks_say({MESSAGE:'Wait...'}); yield* wait(1000); block.looks_say({MESSAGE:'Done'});`，最后 `retire(); return;`。
- **结论**：文档所述效果——长循环用 `yield` 让出防编辑器冻结，`yield* wait(ms)` 实现毫秒延迟；非常驻脚本须显式 `retire()+return`，长运行 generator 可一直 yield。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 要在积木项目里用注释直接写 JavaScript，做实验、调试或高控制逻辑，不想受积木语法约束。
2. 想知道在 #code 里怎么调用积木（按 opcode）、读写舞台/角色变量与列表、做毫秒级延迟。
3. #code 脚本让编辑器卡住冻结，或执行一次后疑似没正确结束，想找 yield/retire 的用法。
4. 遇到 #code 报错/行为异常，或想知道它是否影响别的脚本、能否上传 Scratch。

### 语言信号
- "#code / 内嵌 JS / 直接写 JS / 注释里写 JS / 逃生舱 / escape hatch"
- "target / runtime / stage / thread / opcode / block / vars / wait / generator"
- "yield / yield* wait / 延迟 / 长循环卡住 / 编辑器冻结 / retire / return"
- "读写变量 / 列表 / _monitorUpToDate / 不兼容 Scratch / 脚本没结束"

### 与相邻 skill 的区分
- 与 `custom-reporter`（also_read）：本卡是"在注释里写原生 JS、直触 VM 内部"；custom-reporter 是"把自定义积木拖入 return 变报告积木/递归"。要可嵌入的报告积木用 custom-reporter。
- 与 `extension-authoring`（also_read）：本卡是单项目、单脚本的 JS 逃生舱；build 卡是"从零写一个可分发扩展"（getInfo/register/沙箱决策/发布）。要做可复用扩展用 build。
- 与 `compiler-and-perf`（router）：#code 绕过该脚本积木编译，错误以 JS 错误暴露；若怀疑是编译器 bug 本身则用 compiler-and-perf 排查。

## E — 可执行步骤 (Execution)

**输入契约**：一个 02Engine 积木项目 + 明确要用原生 JS 实现的高控制意图（如"每帧移动角色/读变量算完写回"）+ 已确认该 JS 是自己理解且信任的代码 + 接受项目不上传 Scratch 官网。
1. **写入口**：给顶层帽子积木加注释，行首写 `#code`，其后写 JS body（也可首行 `#code console.log(...)`）。完成标准：该脚本被识别为 JS 脚本；判停：标记不在行首 → 不被识别，改写至行首。
2. **用注入变量写逻辑**：用 `target`/`runtime`/`stage`/`thread`/`p0..` 做高控制逻辑，代码小而集中。完成标准：能访问所需 VM 内部对象；判停：需要私有字段时先评估版本维护成本（见 B 段）。
3. **调积木与读写变量**：用 `block.<opcode>({参数})` 调积木、`vars.stage/target(name[,type])` 读写变量列表，列表传 `'list'`，改值后置 `_monitorUpToDate = false`。完成标准：积木被调用、变量/列表值正确更新且监视器刷新。
4. **加让出与延迟**：长循环内插入 `yield` 防冻结，延迟用 `yield* wait(ms)`。完成标准：编辑器不冻结、延迟按毫秒生效；判停：无 yield 的 while/for 长循环导致冻结 → 立即补 yield。
5. **正确结束脚本**：非常驻脚本结尾写 `retire(); return;`；长运行 generator 保持 yield。完成标准：脚本执行一次后正确结束、无残留；判停：缺 retire+return → 脚本不走到清理路径、行为异常。
6. **双环境验证**：在编辑器与打包运行时都测一遍，确认无语法/不安全调用错误（这类错误以 JS/编译器错误暴露）。完成标准：两处运行一致、无报错，再分发。

**输出契约**：一段含 `#code` 标记的顶层帽子积木注释（JS body 用注入变量/block/vars/wait 实现）+ 明确的结束处理（retire+return 或持续 yield）+ 长循环 yield 与延迟说明 + 编辑器/打包双环境验证记录 + "项目不兼容 Scratch、仅用可信代码"的确认。

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill
- 只想给项目造可复用积木/报告积木 → `extension-authoring` 或 `custom-reporter`。
- 计划上传 Scratch 官网的项目 → 不要用 `#code`（依赖此功能不兼容 Scratch）。
- 具体积木逻辑错误、性能定位 → 用调试工具或 `compiler-and-perf`。

### 作者在文档中警告的失败模式
- **信任边界**：直接 JS 运行在编译后的脚本体、可访问 VM internals，只用自己理解且信任的代码；来源不明的片段会破坏 VM 状态。
- **非常驻脚本不 retire+return**：缺结束语句不走编译器清理路径，脚本无法正确结束、行为异常；长驻 generator 才用持续 yield 代替。
- **长循环不加 yield 会冻结编辑器**：`yield` 是 generator 脚本让出执行权的点，无让出则主线程被独占、界面卡死。
- **别依赖私有 VM 字段**：私有字段无兼容保证，内部实现变更后项目失效；优先用稳定的 block/vars/wait 助手。
- **错误以 JS 错误暴露**：绕过积木编译后，语法错误与不安全 VM 调用报 JS/编译器错误，而非 Scratch 积木错误，排错方式不同。

### 文档的盲点 / 局限
- `#code` 只影响注释含该标记的那一个脚本，其他脚本照常编译；不是全局开关。
- 标记必须在行首，行首之前的内容不被识别；`#code` 之前的注释行仍可作普通笔记或 TurboWarp 编译标志。
- 改值后须手动置 `_monitorUpToDate = false`，否则监视器不刷新。
- 依赖 02Engine 编译器，须在编辑器与打包运行时双测；`turbowarp-blocks` 页 404 不影响本功能。

### 容易混淆的邻近方法论
- `#code`（注释里写原生 JS、直触 VM 内部）vs `extension-authoring`（写独立扩展、getInfo/register/发布）。
- `block` 代理（按 opcode 名调积木）vs 直接调 VM 内部方法：前者是文档推荐的稳定助手，后者是易碎的私有耦合。
- `yield`（让出执行权、防冻结）vs `yield* wait(ms)`（延迟）：前者是循环让出点，后者是毫秒级等待助手。
- `retire(); return;`（非常驻脚本结束）vs 长运行 generator 持续 yield（不结束）——按脚本是否常驻二选一。

---

## 相关 skills
- composes-with: `custom-reporter`、`extension-authoring`
- depends-on: 无

## 审计信息
- 来源: summary-blocks.md §3（direct-javascript-coding）；candidates/framework.md f06–f08、principle.md p05–p07、case.md c01–c04、counter-example.md ce07–ce10；slug: direct-js（原文行95/98/100/140/162/179/204/214/224/226）
- 引用回查: R 段三处英文引用逐字核对 `02engine-focused.txt` direct-javascript-coding 页，均 ≤100 词；中文翻译忠实原意。
- 元数据: verified.yaml `cap.e2.direct-js`（status active / importance medium / destination promoted）；also_read: custom-reporter / extension-authoring。
