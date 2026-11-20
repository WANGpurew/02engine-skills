<!-- capability_id: cap.e2.extension-library | revision: 1 | status: active -->

# 扩展库选型、安装与来源排查

## R — 原文 (Reading)

> "The source sidebar groups extensions by where their metadata comes from: ... Search by name, description, tags, source name, and author/credit text. Filters can be combined. If the result is empty, clear filters or switch sources."
>
> 译文：来源侧栏按扩展元数据的出处分组（Official Scratch / 02Engine / TurboWarp / PenguinMod / Mist / SharkPool / Other）；可按名称、描述、标签、来源名、作者/署名文本搜索，过滤器可叠加；结果为空时清空过滤或切换来源。
>
> — 02Engine 功能文档·extension-library / slug: extension-library（行576-588；源：candidates/framework.md f14/f15）

> "Extensions are processed one by one. A failure does not stop the rest of the queue. URL extensions can use normal import or text import. Native extensions always use normal import, even if text import is selected. A completion summary reports how many extensions succeeded or failed."
>
> 译文：批量导入逐个处理、单个失败不中断队列；URL 扩展可用 normal 或 text import；原生扩展始终走 normal import（即便选了 text import）；结束给成功/失败汇总。
>
> — 02Engine 功能文档·extension-library / slug: extension-library（源：candidates/case.md c12）

> "Native: a built-in extension loaded by ID. Batch: can be added to a batch import queue. Not Scratch-compatible: the extension or feature creates projects that the Scratch website may not support. ... 02Engine's VM currently loads custom extension URLs through the unsandboxed extension runner. Only install extensions from sources you trust."
>
> 译文：徽章含义——Native 是按 ID 加载的内置扩展、Batch 是可入批量队列、Not Scratch-compatible 是生成的项目官网可能不支持；当前 VM 通过非沙箱 runner 加载自定义扩展 URL，只装可信来源。
>
> — 02Engine 功能文档·extension-library / slug: extension-library（行576-588；源：principle.md p19/p20、counter-example.md ce21/ce22）

## I — 方法论骨架 (Interpretation)

扩展库是一个"多来源工作台"：保留 Scratch 原生扩展，叠加外部画廊与 02Engine 专属动作，统一可搜索。
① **七个来源分组**——Official Scratch（Pen/Music/Translate 及硬件）、02Engine、TurboWarp、PenguinMod、Mist、SharkPool、Other（不匹配上述分组的已加载项）；默认按原列表/画廊策展顺序，不按字母重排；
② **搜索 + 叠加过滤**——按名称/描述/标签/来源名/作者搜，叠加收藏/当前批量选中/Scratch 兼容/原生/自定义过滤；结果为空先清过滤或换来源；
③ **收藏规则**——有 URL 时按扩展 URL 存、否则按 ID 存；重复条目在收藏区与来源区共享收藏状态；
④ **单装走卡片常规行为**——原生扩展直接加载；URL 扩展按需弹导入方式对话框；Custom Extension 卡打开自定义加载器；Custom Reporters 项启用自定义过程返回值；原生扩展不支持文本导入；
⑤ **批量导入**——可排队卡片角落有复选框，勾选后点 "Batch import X extensions"；逐个处理、单个失败不中断、URL 扩展可 normal/text、原生始终 normal、特殊动作走各自路径、结束给汇总；`Custom Extension` 卡故意不可批量；
⑥ **徽章确认状态**——Native / Batch / Not Scratch-compatible；"Scratch 兼容"仅指"可安全用于类 Scratch 环境"，不保证能上传或运行于 scratch.mit.edu；
⑦ **信任边界**——外部 URL 经 unsandboxed runner 加载、可访问普通项目触不到的强浏览器/VM API，因此只装可信来源。
核心洞察：把"能不能找到 / 装对没有 / 敢不敢装"三件事分开——来源与过滤解决找，徽章与汇总解决装对，可信来源解决敢装。

## A1 — 书中的应用 (Past Application)

### 案例一：扩展库批量导入操作流程（scenario，slug: extension-library / c12）
- **问题**：需要一次安装多个扩展——卡片如何排队、哪些能进批量、单个失败怎么办。
- **方法执行**：勾选可排队卡片角落的复选框 → 点 "Batch import X extensions" → 逐个处理、单个失败不中断队列；URL 扩展可 normal/text import、原生扩展始终 normal import、特殊动作走各自路径；`Custom Extension` 卡故意不可批量选（走独立手动加载）。
- **结论**：文档所述效果——批量导入逐个处理且失败不中断，结束给成功/失败汇总；原生扩展即便选了 text import 仍走 normal import。

### 案例二：扩展库兼容性徽章识别（scenario，slug: extension-library / c13）
- **问题**：装扩展前如何判断它是内置、能否批量、是否兼容 Scratch。
- **方法执行**：看卡片徽章——Native（按 ID 加载的内置）、Batch（可入批量队列）、Not Scratch-compatible（生成的项目 Scratch 官网可能不支持）；理解 "Scratch compatible" 仅指"可安全用于类 Scratch 环境"。
- **结论**：文档所述效果——用徽章可确认加载状态与兼容边界；`Not Scratch-compatible` 不保证能上传或运行在官网，只表示类 Scratch 环境可用。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 想找并装某个扩展（Pen/Translate/Translate 或第三方扩展），不知在扩展库哪里、有哪些来源。
2. 要一次装多个扩展，或批量装到一半失败，想确认哪些成功、哪些失败。
3. 某扩展在扩展库找不到，想按来源分组、清空过滤来定位。
4. 想确认某扩展是否兼容 Scratch、能否上传官网，或判断一个外部 URL 扩展是否可信、敢不敢装。

### 语言信号
- "扩展库 / 找扩展 / 装扩展 / 安装扩展 / 批量导入 / Batch import"
- "来源 / Sources / Official Scratch / TurboWarp / PenguinMod / 收藏 / Favorites"
- "徽章 / Native / Batch / Not Scratch-compatible / 兼容 Scratch / 能上传官网吗"
- "找不到扩展 / 过滤 / extension library / 可信来源 / unsandboxed runner"

### 与相邻 skill 的区分
- 与 `extension-authoring`（also_read）：本卡是"选型、安装现成扩展"；build 卡是"从零写扩展"。想装现成的用本卡，想自己写 JS 用 build。
- 与 `ai-agent-workflow`（also_read）：本卡是用户手动选型安装；agent 卡是"让 AI 搜装扩展"（searchExtensions→installExtension）+ 整项目改动。让 AI 代劳或改项目用 agent。

## E — 可执行步骤 (Execution)

**输入契约**：一个扩展获取意图（扩展名/功能/来源线索）+ 是否批量 + 对外部 URL 来源的可信性判断 + 是否计划上传 Scratch 官网（决定兼容要求）。
1. **定位来源**：切到目标扩展可能所属的 Sources 分组（七类之一）。完成标准：该分组列表中出现目标扩展；判停：所有分组都没有 → 判断扩展未加载或来源不在列表内。
2. **搜索过滤**：按名称/描述/标签/作者搜索，按需叠加收藏/批量选中/Scratch 兼容/原生/自定义过滤。完成标准：结果集含目标扩展；判停：结果为空 → 先清过滤或换来源，别急着判定"没有"。
3. **看徽章**：确认 Native（内置按 ID）/ Batch（可批量）/ Not Scratch-compatible（官网可能不支持）。完成标准：明确加载状态与兼容边界；注意"兼容"≠可上传官网。
4. **单装**：点卡片走常规行为——原生直接加载、URL 扩展按需弹导入对话框、Custom Extension 开自定义加载器、Custom Reporters 启用自定义返回值。完成标准：扩展出现在工具箱可拖用；判停：原生扩展不支持文本导入（始终走 VM 常规路径）。
5. **批量（如需）**：勾选有 Batch 徽章的可排队卡片 → "Batch import X extensions" → 逐个处理、单个失败不中断、URL 可 normal/text、原生始终 normal、Custom Extension 卡不可批量。完成标准：结束给出成功/失败汇总，逐条核对失败项。
6. **收藏**：有 URL 按 URL 存、否则按 ID 存；重复条目共享收藏状态。完成标准：收藏跨来源一致、换来源后仍可见。
7. **信任核对（URL 扩展）**：URL 经 unsandboxed runner 加载、权限高 → 只装可信来源；陌生 URL 默认拒绝、先审阅源码。完成标准：来源可信确认后才装；判停：来源不明/不可信 → 不装。
8. **兼容核对（要上传官网时）**：徽章只做初筛，实际到 scratch.mit.edu 验证能否上传/运行；不用任何 02Engine 专属积木时兼容性还取决于其它扩展与设置。完成标准：官网上传成功或明确放弃上传。
**输出契约**：已安装扩展清单（名称/来源/徽章/加载方式 yes）+ 批量导入成功/失败汇总（含失败项原因）+ 兼容性判断结论（能否上传官网的核对结果）+ 外部 URL 来源可信性核对记录。

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill
- 自己写一个扩展 → `extension-authoring`；让 AI 代为搜装扩展或改项目 → `ai-agent-workflow`。
- 给编辑器 UI 加按钮/面板/主题 → 写 addon，不在本 skill（聚焦积木与扩展）范围。

### 作者在文档中警告的失败模式
- **徽章≠上传保证**：`Not Scratch-compatible` 仅表示"可安全用于类 Scratch 环境"，不保证能上传或运行于 scratch.mit.edu；只看徽章会误判上传资格。
- **原生扩展不支持文本导入**：批量导入时即便选了 text import，原生扩展仍走 normal import。
- **非沙箱信任风险**：当前 VM 通过 unsandboxed extension runner 加载 URL 扩展，可访问普通项目无法触及的强浏览器/VM API；从任意来源安装 = 把强权限交给未知代码，只装可信来源。
- **结果为空**：多为过滤条件叠加过多或来源分组选错，先清过滤/换来源，别误判"扩展不存在"。

### 文档的盲点 / 局限
- 七类来源分组保持各来源策展顺序，不按字母重排。
- `Custom Extension` 卡故意不可批量选（走独立手动加载流程）。
- "Scratch 兼容"不等于官网可用；即便不用专属积木，兼容性仍取决于项目的其他扩展与设置。
- 收藏有 URL 按 URL、无 URL 按 ID，换设备/换 URL 后按 URL 收藏的条目可能失联。

### 容易混淆的邻近方法论
- Native 徽章（按 ID 内置）vs Batch 徽章（可入批量队列）vs Not Scratch-compatible（官网风险提示）——三者维度不同。
- 单装 vs 批量导入的路径差异：URL 扩展的 text import 只在批量可选，单装走卡片常规行为。
- 收藏按 URL 存 vs 按 ID 存。
- 扩展库选型安装（本卡）vs 从零写扩展（build）vs 让 AI 搜装扩展（agent）。

---

## 相关 skills
- composes-with: `extension-authoring`、`ai-agent-workflow`
- depends-on: 无

## 审计信息
- 来源: summary-extensions.md §1（extension-library）；candidates/framework.md f14/f15、principle.md p19/p20、case.md c12/c13、counter-example.md ce21/ce22；slug extension-library（原文行576-588）
- 引用回查: R 段英文引用逐字核对 `02engine-focused.txt` 行576-588 与候选池 f14/f15/c12/p19/p20，均 ≤100 词；中文翻译忠实原意。
- 元数据: verified.yaml `cap.e2.extension-library`（status active / importance high / destination promoted）；also_read: extension-authoring, ai-agent-workflow。
