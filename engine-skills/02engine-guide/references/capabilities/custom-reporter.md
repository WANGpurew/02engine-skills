<!-- capability_id: cap.e2.custom-reporter | revision: 1 | status: active -->

# 自定义报告积木、递归与纯函数式设计

## R — 原文 (Reading)

> "02Engine now supports custom reporters which lets you use any custom block in your project as a reporter, like the addition block built into Scratch. To make a custom reporter, just drag a 'return' block into the script -- it'll be located at the bottom of the custom block list."
>
> 译文：02Engine 现支持自定义报告积木——可把项目里任意自定义积木当报告积木用（像 Scratch 自带的加法积木）；做法只是拖入一个 "return" 积木（位于自定义积木列表底部），无需勾选任何选项。
>
> — 02Engine 功能文档·return / slug: return（源：candidates/framework.md f03）

> "If you can take a large problem and continuously break it down into smaller problems until you reach a known 'base case', many algorithms can be written very elegantly. Using recursion, you can rewrite the repeat block to use no variables at all:"
>
> 译文：若能把大问题不断拆成更小的问题直到一个已知的"基线条件"，很多算法能写得很优雅；用递归可以完全不用变量就重写 repeat 积木。
>
> — 02Engine 功能文档·return / slug: return（源：candidates/framework.md f04）

> "It is recommended to ensure that custom reporters only run some algorithm and output a final answer. To ensure your project runs the same in the compiler and interpreter, custom reporters should avoid things like moving sprites, saying text, waiting, etc."
>
> 译文：建议让自定义报告积木只运行某种算法并输出最终答案；为确保项目在编译器与解释器下行为一致，报告积木应避免移动角色、说文字、等待之类操作。
>
> — 02Engine 功能文档·return / slug: return（源：candidates/framework.md f05、principle.md p04、counter-example.md ce05）

## I — 方法论骨架 (Interpretation)

自定义报告积木技术上没增加新能力，只是让重复逻辑更易读写。
① **启用与建法**：左下角扩展库（与 "pen" 同处）启用 "Custom Reporters" → 像平常一样建自定义积木，只需在脚本里拖入一个 "return" 积木（在自定义积木列表底部）、无需勾任何选项；执行到 return 时效果类似 "stop this script"；
② **形状自动重塑**：02Engine 会猜积木形状，可能猜错，右键选 "Changed To Stacked Block" 或 "Change To Reporter" 手动改；若所有 return 都是布尔值（如 "1 > 2"）会呈布尔形状，但这只是视觉辅助——任何自定义报告积木都能放进任何输入；
③ **递归**：把大问题不断拆到"基线条件（base case）"，让积木调用自身逼近基线，可不用任何变量重写 repeat；循环与递归可互相改写；
④ **纯函数设计**：报告积木只算算法、输出最终值，避免移动/说/等待等副作用——编译器有短路优化（如 `<(1=2) and (3=3)>` 前半 false 就跳过后半），会跳过有副作用的积木，而解释器总是全算，深层嵌套的执行顺序两者也可能不同，副作用正是行为不一致的根源；
⑤ **栈溢出边界**：解释器支持无限递归，编译器受 JS 栈大小限制（约几千层，因系统/浏览器而异），递归过深会 stack overflow；
⑥ **兼容**：用此功能的项目无法上传 Scratch 官网。
核心洞察：把"要复用的一段计算"封装成带 return 的积木当报告积木嵌入别处，换来可读性；代价是必须保持纯计算——一旦掺入副作用，编译器/解释器就会分叉，递归再深一点还会栈溢出。

## A1 — 书中的应用 (Past Application)

### 案例一：用 return 把自定义积木变成可嵌入报告积木（worked_example，slug: return / f03）
- **问题**：反复拼接文本等场景里，想把一个自定义积木当报告积木（像加法积木）嵌进别的积木，提升可读性。
- **方法执行**：左下扩展库启用 "Custom Reporters" → 像平常一样建自定义积木，在脚本底部拖入 "return" 积木（无需勾选选项）→ 02Engine 自动猜形状（猜错则右键 "Changed To Stacked Block" / "Change To Reporter" 改）→ 把该积木嵌入其他积木的输入槽复用。
- **结论**：文档所述效果——技术上并未做到"以前做不到的事"，只是更易读写；执行到 return 积木类似 "stop this script"；用此功能的项目无法上传 Scratch 官网。

### 案例二：用递归免变量重写 repeat（worked_example，slug: return / f04）
- **问题**：想不用任何变量实现重复执行，让算法更优雅。
- **方法执行**：明确问题的基线条件 → 把大问题拆成更小的同类子问题 → 让积木调用自身逼近基线条件；控制递归深度（编译器约几千层栈溢出），跨平台测试。
- **结论**：文档所述效果——可用递归不用变量重写 repeat；解释器支持无限递归，编译器超几千层会栈溢出；报告积木须保持纯计算，否则编译器/解释器行为不一致。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 想把重复的自定义积木做成能嵌进别的积木（像加法积木那样）的报告积木。
2. 想用递归代替 repeat、不用变量写算法（如阶乘、求和、遍历）。
3. 自定义积木形状被猜错，或返回值在编译器与解释器下行为不一致（同段积木两处运行结果不同）。
4. 递归脚本报 stack overflow、或想知道报告积木/递归能不能上传 Scratch 官网。

### 语言信号
- "自定义积木 / 报告积木 / return / 返回值 / 可嵌入 / Custom Reporters"
- "递归 / 基线条件 / base case / 代替循环 / 不用变量写 repeat"
- "栈溢出 / stack overflow / 几千层 / 编译器解释器不一致 / 短路 / 副作用"
- "形状猜错 / Changed To Reporter / Changed To Stacked Block / reporter"

### 与相邻 skill 的区分
- 与 `inline-js`（also_read）：本卡是"自定义积木 + return + 递归"的积木层技巧；direct-js 是在积木项目注释里用 `#code` 写原生 JS。要在积木层做报告积木用本卡，要写原生 JS 用 direct-js。
- 与 `ai-agent-workflow`（also_read）：本卡讲报告积木本身的设计规则；agent 卡是让 AI 小步改项目的工作流。让 AI 帮忙改项目用 agent。
- 与 `extra-blocks`：extra-blocks 讲 02Engine 内置额外积木及 Scratch 兼容边界；本卡讲用户自建的报告积木。注意两者都不兼容 Scratch 官网，但机理不同。

## E — 可执行步骤 (Execution)

**输入契约**：一个要复用的计算逻辑（可拆到基线条件）+ 是否需嵌入别的积木作输入 + 目标运行环境（编译器/解释器，是否上传官网）。
1. **启用 Custom Reporters**：左下角扩展库启用 "Custom Reporters"。完成标准：自定义积木列表底部出现 "return" 积木；判停：项目要上传 Scratch 官网 → 不要启用/使用本功能。
2. **建带 return 的自定义积木**：像平常一样建自定义积木，把 "return" 拖入脚本；执行到 return 即返回（类似 stop this script）。完成标准：积木能返回最终值并可嵌入别的积木输入槽。
3. **校正形状**：02Engine 自动猜形状；猜错则右键 "Changed To Stacked Block" / "Change To Reporter"。完成标准：形状符合预期；记住形状只是视觉辅助、任何自定义报告积木都能放进任何输入。
4. **保持纯函数**：报告积木只算算法、return 最终值；去除移动角色/说文字/等待等副作用；不在 `and`/`or` 分支里依赖被短路侧的副作用。完成标准：同一积木在编译器与解释器下结果一致。
5. **递归（如需）**：明确基线条件 → 拆成更小的同类子问题 → 积木调用自身逼近基线；控制深度（编译器约几千层栈溢出）、在编辑器与打包运行时都测。完成标准：基线条件能及时收敛、无栈溢出；判停：深度不可控或只在解释器测过 → 改写为循环或降低深度。
6. **复用**：把报告积木嵌入其他积木的输入槽，替代重复逻辑。完成标准：逻辑只写一处、可读性提升。
**输出契约**：一份带 "return" 的自定义报告积木（含基线条件/递归设计）+ 纯函数性核对（无移动/说/等待副作用、不依赖短路侧）+ 编译器/解释器一致性验证 + 递归深度与"不上传 Scratch 官网"的兼容性说明。

## B — 边界 (Boundary) ★

### 不要用情况
- 要写独立 JS 扩展 → `extension-authoring`；要在积木项目注释里写原生 JS → `inline-js`。
- 想问某内置积木是否兼容 Scratch → `extra-blocks`；想让 AI 帮忙改项目 → `ai-agent-workflow`。

### 作者在文档中警告的失败模式
- **不兼容 Scratch 官网**：用自定义报告积木/递归的项目无法上传 scratch.mit.edu。
- **编译器栈溢出**：递归超约几千层通常抛 stack overflow；解释器支持无限递归但编译器受 JS 栈限制（上限因系统/浏览器而异）。只在解释器里测过递归会踩坑。
- **副作用致行为不一致**：报告积木里做移动/说/等待，编译器短路会跳过、解释器总全算、深层嵌套执行顺序也可能不同——同一项目两环境结果分叉。
- **短路跳过含副作用积木**：如 `<(1=2) and (3=3)>` 前半为 false 就跳过后半，别把计数/移动积木塞进 `and`/`or` 右侧。
- **形状可能被猜错**：需右键手动改；但布尔形状只是视觉辅助，不代表只能放布尔输入。

### 文档的盲点 / 局限
- 技术上未增加新能力，只是更易读写（文档自述）。
- "约几千层"是经验上限、非精确常量，随系统/浏览器变化。
- 形状判断基于 return 值类型，复杂分支可能猜错。
- 用此功能的项目不与 Scratch 兼容，发布前需在编辑器与打包运行时都验证一致性。

### 容易混淆的邻近方法论
- 报告积木（可嵌入、带 return）vs 普通堆叠自定义积木（不返回值）。
- 递归（拆到基线条件、调用自身）vs 循环（repeat 积木）——可互相改写。
- 编译器（短路优化、栈限制、可能重排执行顺序）vs 解释器（全算、无限递归、按序执行）。
- 积木"形状"（布尔/圆角，视觉辅助）vs 实际可放置的输入槽（任何自定义报告积木都能放任何输入）。

---

## 相关 skills
- composes-with: `inline-js`、`ai-agent-workflow`
- depends-on: 无

## 审计信息
- 来源: summary-blocks.md §2（return）；candidates/framework.md f03-f05、principle.md p03/p04、case.md（return 页演示见 f03-f05 source_quote）、counter-example.md ce03-ce06；slug return（原文行约47-86）
- 引用回查: R 段三处英文引用逐字核对候选池 f03/f04/f05（源自 `02engine-focused.txt` 对应 slug），均 ≤100 词；中文翻译忠实原意。
- 元数据: verified.yaml `cap.e2.custom-reporter`（status active / importance high / destination promoted）；also_read: inline-js, ai-agent-workflow。
