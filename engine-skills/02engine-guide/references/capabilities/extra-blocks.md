<!-- capability_id: cap.e2.extra-blocks | revision: 1 | status: active -->

# 02Engine 额外积木与 Scratch 兼容边界

## R — 原文 (Reading)

> "These blocks are 'compatible' with Scratch because they're actually just modified argument reporters. Every block beyond this warning is incompatible with Scratch. Projects that use them cannot be uploaded to the Scratch website."
>
> 译文：这些积木"兼容" Scratch，因为它们本质只是修改过的参数报告积木（modified argument reporters）；此警告之后的每一个积木都与 Scratch 不兼容，使用它们的项目无法上传到 Scratch 官网。
>
> — 02Engine 功能文档·blocks / slug: blocks（源：candidates/framework.md f01、principle.md p01、counter-example.md ce01）

> "It's like 'mouse down?' but lets you check each individual button. Keep in mind that due to how Scratch interprets mouse input, it's possible for a block like 'is primary mouse button down?' to report true while the standard 'mouse down?' reports false."
>
> 译文：它像 "mouse down?" 但能逐个按钮检测；注意，由于 Scratch 解释鼠标输入的方式，"is primary mouse button down?" 可能为 true，而标准 "mouse down?" 却为 false。
>
> — 02Engine 功能文档·blocks / slug: blocks（源：candidates/framework.md f02、principle.md p02、counter-example.md ce02）

> "- (0) primary is usually left click - (1) middle is usually scroll wheel - (2) secondary is usually right click (running this block once will disable right click on the stage)"
>
> 译文：参数 (0) primary 通常是左键、(1) middle 通常是滚轮、(2) secondary 通常是右键（运行这个积木一次会禁用舞台上的右键）。
>
> — 02Engine 功能文档·blocks / slug: blocks（源：principle.md p02、counter-example.md ce02）

## I — 方法论骨架 (Interpretation)

02Engine 内置一套继承自 TurboWarp 的"额外积木"类别，暴露类 Scratch 运行时通常无法访问的运行时功能；更多积木需到 02Engine 扩展库安装可信自定义扩展。
① **兼容分界线（warning）**：文档用一条警告线划界——线前的 `is compiled?`（是否已编译）/ `is 02Engine?`（是否为 02Engine）本质是"修改过的参数报告积木（modified argument reporters）"、兼容 Scratch；线后每一个积木都不兼容 Scratch，用了项目无法上传 scratch.mit.edu；
② **不是"能用即可"**：Scratch 兼容性是"能否上传官网"的硬边界；即便不用任何 02Engine 专属积木，兼容性仍取决于项目用到的其他扩展与设置；
③ **last key pressed（最后按下的键）**：报告最后一次被按下的键；
④ **mouse button down?（某鼠标按钮是否按下）**：类似 "mouse down?" 但可逐个按钮检测，参数 (0) primary 通常左键、(1) middle 通常滚轮、(2) secondary 通常右键；
⑤ **鼠标键陷阱**：因 Scratch 解释鼠标输入的方式，单个按钮积木可能与标准 "mouse down?" 状态矛盾（primary 为 true 而 mouse down? 为 false）；(2) secondary 运行一次会禁用舞台右键；
⑥ **更多积木靠扩展**：需要额外积木时到扩展库安装可信自定义扩展（外部 URL 经非沙箱 runner 加载，只装可信来源）。
核心洞察：兼容性用"警告线"一刀切——分界只在"改过的参数报告积木"这一档，其余专属积木一律牺牲上传资格换能力；检测输入时把"最后按键"与"逐按钮鼠标"当成两类独立信号，别与标准 mouse down? 混用做等价判断。

## A1 — 书中的应用 (Past Application)

### 案例一：额外积木兼容性自检（scenario，slug: blocks / f01）
- **问题**：想在项目里用 02Engine 的额外积木，但不确定用完后还能不能上传 Scratch 官网。
- **方法执行**：用积木前先定位文档警告分界线 → 若需保持兼容，只用线前的 `is compiled?` / `is 02Engine?` 等 modified argument reporters → 需要更多积木时到扩展库装可信自定义扩展 → 一旦用了线后积木，就放弃上传官网（接受兼容性割裂）。
- **结论**：文档所述效果——线前积木兼容 Scratch、线后积木使项目无法上传；即使不用专属积木，兼容性仍取决于项目的其它扩展与设置。

### 案例二：多鼠标按钮检测（scenario，slug: blocks / f02 + p02）
- **问题**：要分别检测左键、中键/滚轮、右键，而标准 "mouse down?" 只能检测单一的按下状态。
- **方法执行**：用 `mouse button down?` 按参数 0=primary（左）/ 1=middle（滚轮）/ 2=secondary（右）逐个检测；不与标准 "mouse down?" 混用做等价判断；警惕 secondary 积木运行一次会禁用舞台右键。
- **结论**：文档所述效果——可逐个按钮检测；单按钮状态可能与标准 "mouse down?" 不一致（primary 为 true 而标准为 false 的情况）；(2) secondary 积木运行一次会禁用舞台右键。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 想知道 02Engine 比 Scratch 多了哪些可用积木、某积木到底能不能上传 Scratch 官网。
2. 要检测最后按下的键，或分别检测左/中/右键（如做多键操作、FPS 风格控制）。
3. 发现 "primary mouse button down?" 为 true 但标准 "mouse down?" 为 false，或舞台右键突然不能用了。
4. 想用更多积木但不知要到扩展库装可信自定义扩展，或担心积木用了以后项目传不上去。

### 语言信号
- "额外积木 / Extra Blocks / is compiled / is 02Engine / 多哪些积木"
- "能不能上传 Scratch / 兼容 Scratch / 警告线 / 上传官网 / Scratch-compatible"
- "最后按下的键 / last key pressed / 鼠标按钮 / mouse button down / 左键 中键 右键"
- "右键不能用 / 状态不一致 / 舞台右键 / primary / secondary"

### 与相邻 skill 的区分
- 与 `custom-reporter`（also_read）：本卡讲 02Engine 内置额外积木及 Scratch 兼容边界；custom-reporter 讲把自定义积木变报告积木/递归。要写报告积木用 custom-reporter。
- 与 `extension-authoring`：本卡是"用内置额外积木"；build 卡是"从零写扩展"。想给项目造新积木用 build；只是问内置积木能否上传用本卡。
- 与 `extension-library`：`Not Scratch-compatible` 徽章用它判断扩展兼容；本卡判断的是文档警告线前后的内置积木。二者都强调"兼容≠能上传官网"。

## E — 可执行步骤 (Execution)

**输入契约**：一个积木使用意图 + 是否计划上传 Scratch 官网（决定兼容要求）+（输入检测时）要检测的按键/鼠标按钮。
1. **定位警告线**：确认目标积木位于文档警告线之前还是之后。完成标准：明确兼容性归属；判停：要上传官网却用了线后积木 → 停用该积木或放弃上传。
2. **保持兼容（需上传官网时）**：只用线前的 `is compiled?` / `is 02Engine?` 等 modified argument reporters。完成标准：项目可上传官网（仍须核对其它扩展与设置，别只看积木）。
3. **用线后积木（仅 02Engine 内/打包）**：接受无法上传官网；需要更多积木时到扩展库装可信自定义扩展（外部 URL 经非沙箱 runner、只装可信来源）。完成标准：明确兼容性割裂并已确认来源可信。
4. **检测最后按键**：用 `last key pressed` 报告最后一次按下的键。完成标准：读到预期的键名/数值。
5. **检测鼠标按钮**：用 `mouse button down?` 按 0 primary（左）/ 1 middle（滚轮）/ 2 secondary（右）逐个检测；不与标准 `mouse down?` 混用做等价判断。完成标准：各按钮状态符合预期；判停：依赖舞台右键菜单/交互的工作流避免使用 secondary（运行一次会禁用舞台右键）。
6. **官网核对（要上传时）**：徽章/线前判断只做初筛，实际到 scratch.mit.edu 验证能否上传/运行。完成标准：上传成功或明确放弃上传。
**输出契约**：额外积木使用清单（每块标注线前/线后、是否兼容 Scratch）+ 按键/鼠标检测实现（含 0/1/2 口径说明）+ 上传官网可行性结论（含实际验证）+ 扩展库补充积木的可信来源核对记录。

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill
- 自定义积木/递归 → `custom-reporter`；写独立 JS 扩展 → `extension-authoring`；某扩展是否兼容 → `extension-library` 的徽章判断。
- 做编辑器界面/主题修改 → 不在本 skill 聚焦范围内。

### 作者在文档中警告的失败模式
- **线后积木破坏上传资格**：警告线之后的每个积木都不兼容 Scratch，用了项目无法上传 scratch.mit.edu；别看到线前"兼容"的参数报告积木就以为整段额外积木都兼容。
- **secondary 右键副作用**：mouse button down? (2) secondary 运行一次会禁用舞台上的右键，影响舞台上正常操作。
- **鼠标状态矛盾**：因 Scratch 解释鼠标输入的方式，"is primary mouse button down?" 可能为 true 而标准 "mouse down?" 为 false——不要把二者当等价信号混用。
- **兼容不等于能用**：不必专属积木时兼容性还取决于其他扩展与设置；`is compiled?` / `is 02Engine?` 只是"改过的参数报告积木"，并非保证上传的通行证之外的全面兼容。

### 文档的盲点 / 局限
- `turbowarp-blocks` 页 404 无正文，额外积木以 `blocks` 页为准，不臆造缺失页内容。
- "Scratch 兼容"是硬边界（能否上传），不是"功能能否在类 Scratch 环境跑起来"的模糊判断。
- 更多积木需到扩展库装可信自定义扩展；外部 URL 经非沙箱 runner 加载，只装可信来源（见 extension-library）。
- 鼠标按钮 0/1/2 的"通常"映射（左/滚轮/右）依赖设备与系统，特殊鼠标可能不同。

### 容易混淆的邻近方法论
- 警告线前（参数报告积木、兼容）vs 线后（专属积木、不兼容）。
- `is compiled?`（是否已编译）vs `is 02Engine?`（是否为 02Engine）——两者都是 modified argument reporters、都兼容。
- `last key pressed`（最后按键，单一报告）vs `mouse button down?`（鼠标按钮，按 0/1/2 逐个检测）。
- `mouse button down?` 的 primary vs 标准 `mouse down?`：状态可能不一致，不能互推。

---

## 相关 skills
- composes-with: `custom-reporter`
- depends-on: 无

## 审计信息
- 来源: summary-blocks.md §1（blocks）；candidates/framework.md f01/f02、principle.md p01/p02、case.md（blocks 页演示见 f01/f02 source_quote）、counter-example.md ce01/ce02；slug blocks（原文行23-38）
- 引用回查: R 段三处英文引用逐字核对候选池 f01/f02/p02（源自 `02engine-focused.txt` blocks 页，原文行23-38），均 ≤100 词；中文翻译忠实原意。
- 元数据: verified.yaml `cap.e2.extra-blocks`（status active / importance medium / destination promoted）；also_read: custom-reporter。
