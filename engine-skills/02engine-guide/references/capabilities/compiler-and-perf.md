<!-- capability_id: cap.e2.compiler-and-perf | revision: 1 | status: active -->

# 编译器选择、故障回退与 OPF 性能调优

## R — 原文 (Reading)

> "The broad idea is that the compiler analyzes scripts to determine what kinds of values each variable may have at each point in the script. This allows it to remove unnecessary type conversions and generate more specialized code. The old compiler tried to do this too, but was unable to reason about loops or conditionals."
>
> 译文：新编译器分析各变量在脚本每点可能的值类型，去掉冗余类型转换、生成更专门的代码；老编译器也想这么做，但无法推断循环与条件。
>
> — 02Engine 功能文档·new-compiler / slug: compiler-and-perf（原文行416；源：candidates/framework.md f11、principle.md p16）

> "The warp timer forces blocks marked as 'run without screen refresh' to briefly pause after running for 500 ms to prevent an infinite loop from causing you to lose unsaved work. ... the warp timer breaks many of the assumptions that the new compiler uses to optimize projects, so do not expect major performance improvements while you are in the editor or otherwise have the warp timer enabled."
>
> 译文：warp timer 强制"不刷新屏幕运行"积木运行 500ms 后短暂暂停，防无限循环丢失未保存工作；它破坏新编译器的优化假设，故编辑器内/启用它时别期待显著提升。
>
> — 02Engine 功能文档·new-compiler / slug: compiler-and-perf（原文行402/404；源：candidates/framework.md f11、principle.md p15、counter-example.md ce19）

> "If your project is broken, you can use https://experiments.02engine.org/old-compiler/ instead until we fix the bug in the new compiler. If you also need to package your project, you can use https://packager-legacy.02engine.org/old-compiler/."
>
> 译文：项目被新编译器弄坏时，改用 https://experiments.02engine.org/old-compiler/ 直到 bug 修复；需打包时用 https://packager-legacy.02engine.org/old-compiler/。
>
> — 02Engine 功能文档·new-compiler / slug: compiler-and-perf（原文行371/373；源：candidates/framework.md f11、principle.md p17、counter-example.md ce18）

## I — 方法论骨架 (Interpretation)

把"编译器选择/性能"当成横切排错与调优层，按三层处理：
① **新编译器机制**：靠类型分析去冗余转换、生成更专门代码（如 1~100 求和，老编译器有冗余 `(+sum.value||0)`、边界 `>=0.5`，新编译器为 `sum.value + i.value`、边界 `>0`）；提速因项目而异（部分约两倍、部分不变、不会变慢），在网站/打包器/桌面 app（v1.15.0+）体现；
② **warp timer 抵消**：强制"不刷新屏幕运行"积木每 500ms 暂停防死循环丢工作，编辑器打开时自动启用，破坏优化假设 → 编辑器内/启用时无显著提升（但不会更慢）；启用 warp timer 或积木未标"不刷新屏幕运行"时，新编译器生成与老编译器相同代码；
③ **故障回退**：官方扩展列表全部正常、绝大多数自定义扩展正常；少数用 `i_will_not_ask_for_help_when_these_break` API 的自定义扩展会被新编译器破坏 → 项目用 old-compiler URL 过渡、打包用 packager-legacy URL；
④ **禁用编译器**：关闭编译器使脚本立即更新（同 Scratch），仅用于排查编译器 bug 或编辑器创建项目；可装 "Disable compiler in editor" 插件默认关闭；文档警告"除非清楚在做什么否则不要改"；
⑤ **OPF 调优**：OPF（每帧操作数，Advanced Settings 中作 Custom OPF）控制每帧 VM 工作量，CPU 密集/重计算/CLI 打包可调高、需流畅动画/编辑大项目变卡时保持低；FPS 控产帧频率、OPF 控每帧工作量，二者同时拉高会迅速拉满 CPU → 先调 FPS、不够再调 OPF。
核心洞察：新编译器的提速只在"无 warp timer 干扰"的发布环境成立；出事优先回退老编译器而非禁用编译器（后者仅调试用）；性能调优先 FPS 后 OPF、别同时拉高。

> 路由说明：本能力经 02engine-guide 路由入口访问（未独立成 Skill），user 问 02Engine 编译器/性能/老编译器回退/OPF/FPS 时从 02engine-guide 命中本卡。

## A1 — 书中的应用 (Past Application)

### 案例一：新编译器性能评估与 warp timer 的影响（scenario，slug: new-compiler / f11+p15+ce19）
- **问题**：想知道新编译器到底快多少、为什么在编辑器里看不到提升。
- **方法执行**：文档给出基准（Chromium 140 / Arch Linux / i7-4790k、warp timer 关闭）：Linux in Scratch 21s→12s、SHA-256 2711→3010 次/秒、Quicksort 0.0515s→0.0451s；解释 warp timer 每 500ms 暂停破坏优化假设、编辑器打开时自动启用；给出 1~100 求和的类型分析对比。
- **结论**：文档所述效果——提速因项目而异（部分约两倍、部分不变、不会更慢），但编辑器内/启用 warp timer 时无显著提升（也不更慢）；要评估提速须到网站/打包/桌面环境。

### 案例二：编译器故障回退与禁用（scenario，slug: new-compiler/disable-compiler / f11+f12+ce18）
- **问题**：新编译器下项目出错，或含用 `i_will_not_ask_for_help_when_these_break` 的自定义扩展被破坏，如何过渡？
- **方法执行**：项目改用 `https://experiments.02engine.org/old-compiler/`，需打包用 `https://packager-legacy.02engine.org/old-compiler/`；若要排查编译器 bug 则临时关闭编译器（或装 "Disable compiler in editor" 插件）让脚本立即更新对比。
- **结论**：文档所述效果——老编译器 URL 可供过渡直到扩展兼容/bug 修复；禁用编译器只用于调试或编辑器建项目，文档警告非必要不要改。

### 案例三：先 FPS 后 OPF 的性能调优（scenario，slug: opf / f13+p34+ce42）
- **问题**：重计算项目想跑更快、或动画项目想更流畅，FPS 与 OPF 怎么调。
- **方法执行**：判断项目是 CPU 密集（计算/模拟/CLI 打包、Turbo mode）还是重视觉（动画/大量特效/编辑大项目变卡）；先调 FPS，仍不足再调 Custom OPF；避免二者同时拉高。
- **结论**：文档所述效果——FPS 控产帧频率、OPF 控每帧工作量，同时拉高会迅速高 CPU；重视觉项目调高 OPF 反降流畅度与响应。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 项目在新编译器下出错、变卡，或含某自定义扩展后行为异常，想知道是否编译器问题、如何回退。
2. 想提升项目运行性能，问 OPF/FPS 怎么调、Turbo mode 不够快怎么办。
3. 在编辑器里测速发现"新编译器没变快"，想知道原因。
4. 想临时关编译器让脚本实时更新来排查，或问某 addon（Disable compiler in editor）。

### 语言信号
- "新编译器 / 老编译器 / 编译出错 / old-compiler / 回退 / 编译器 bug"
- "warp timer / 不刷新屏幕运行 / 500ms / 编辑器里没提速"
- "OPF / 每帧操作数 / Custom OPF / FPS / 性能调优 / 项目变慢 / 高 CPU"
- "禁用编译器 / Disable compiler in editor / i_will_not_ask_for_help_when_these_break"

### 与相邻 skill 的区分
- 与 `extension-authoring`（also_read）：本卡处理"编译器/性能/扩展被编译器破坏后的回退"；build 卡处理"怎么写/发布扩展"。扩展本身加载/兼容问题归 build，编译器层故障归本卡。
- 与 `inline-js`：#code 绕过单脚本积木编译、错误以 JS 错误暴露；若怀疑是编译器 bug 本身用本卡。
- 路由说明：本卡经 02engine-guide 路由入口访问（未独立成 Skill），非独立入口。

## E — 可执行步骤 (Execution)

**输入契约**：一个性能/编译问题（项目在新编译器下出错、想提速、或要排查编译器 bug）+ 运行环境（编辑器内 vs 网站/打包/桌面）+ 项目是否含自定义扩展（尤其特殊 API）。
1. **定位环境**：确认问题发生在编辑器内（warp timer 自动启用）还是发布环境。完成标准：明确 warp timer 是否干扰提速判断；判停：在编辑器内测速 → 不以提速与否下结论。
2. **评估提速**：在网站/打包/桌面（无 warp timer）环境评估性能；预期部分项目约两倍、部分不变、不会更慢。完成标准：得到可信的发布环境性能结论。
3. **故障回退**：若项目出错或含 `i_will_not_ask_for_help_when_these_break` 扩展 → 在线用 old-compiler URL、打包用 packager-legacy URL，并向官方报告。完成标准：项目在老编译器下恢复正常；判停：官方列表扩展与绝大多数自定义扩展本应正常，异常先确认是否特殊 API 扩展。
4. **（仅调试）禁用编译器**：先确认问题是编译器 bug 而非脚本逻辑 → 临时关闭编译器或装 "Disable compiler in editor" 插件让脚本实时更新对比 → 排查后恢复正常。完成标准：复现/定位到编译器 bug；判停：文档警告非必要不要改，排查结束即恢复。
5. **调优性能**：判断项目类型；先调 FPS，仍不足再调 Custom OPF；避免 FPS 与 OPF 同时拉高。完成标准：吞吐提升且编辑响应/动画流畅可接受；判停：重视觉/大项目编辑变卡时保持低 OPF。

**输出契约**：环境问题定位结论（warp timer 是否干扰）+ 发布环境性能评估 + 故障回退方案（old-compiler / packager-legacy URL，含是否需报告）+（如需）编译器 bug 调试记录 + FPS/OPF 调优参数与"先 FPS 后 OPF、不同时拉高"的说明。

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill
- 具体积木逻辑错误、脚本写错 → 用调试工具或各创作卡，本卡不修业务逻辑。
- 想写/发布扩展本体 → `extension-authoring`。
- 编辑器界面美化/布局 → 不在本 skill 范围。

### 作者在文档中警告的失败模式
- **warp timer 抵消提速**：编辑器内自动启用 warp timer，破坏优化假设 → 别在编辑器里据提速与否判断新编译器有效性（但也不会比老编译器慢）。
- **少数扩展被新编译器破坏**：用 `i_will_not_ask_for_help_when_these_break` 的自定义扩展故意深耦合编译器内部，新编译器改动后失效 → 回退老编译器。
- **别随意禁用编译器**：该选项只在调试编译器 bug / 编辑器建项目时有用，文档警告"除非清楚在做什么否则不要改"，长期关闭会偏离正式运行环境。
- **FPS 与 OPF 同升拉高 CPU**：二者相乘决定总工作量，同时拉满 CPU 飙升；重视觉项目调高 OPF 反降流畅度与响应。

### 文档的盲点 / 局限
- 提速是"史上最大单项改动"、可能有 bug，官方建议在固定位置报告损坏项目。
- 启用 warp timer 或积木未标"不刷新屏幕运行"时，新编译器生成与老编译器相同代码——这是防变量被其它脚本改类型的安全退化，不是 bug。
- 提速因项目而异，无统一倍率；基准仅在特定硬件/浏览器（Chromium 140 / Arch / i7-4790k、warp timer 关闭）测得。
- OPF/FPS 的具体取值随项目与环境变化，文档只给调优顺序与权衡，无通用最优值。

### 容易混淆的邻近方法论
- 新编译器（类型分析提速）vs 老编译器（故障回退用 URL）：回退是过渡，不是长期方案。
- 故障回退（换编译器 URL）vs 禁用编译器（关闭编译走即时更新）：前者用于让项目正常运行，后者仅调试。
- FPS（产帧频率）vs OPF（每帧工作量）：先 FPS 后 OPF，别同时拉高。
- "编辑器内无提速"是 warp timer 所致，不等于"新编译器无效"。

---

## 相关 skills
- composes-with: `extension-authoring`
- depends-on: 无（经 02engine-guide 路由入口访问，未独立成 Skill）

## 审计信息
- 来源: summary-blocks.md §6/§7（new-compiler/disable-compiler）+ summary-extensions.md §5（opf）；candidates/framework.md f11–f13、principle.md p15–p18/p34、case.md（基准与调优演示见 f11/f13 source_quote）、counter-example.md ce18–ce20/ce42；slug: compiler-and-perf（原文行371/373/402/404/416/477/1090–1101）
- 引用回查: R 段三处英文引用逐字核对 `02engine-focused.txt` new-compiler 页，均 ≤100 词；中文翻译忠实原意。
- 元数据: verified.yaml `cap.e2.compiler-and-perf`（status active / importance medium / destination **router**）；also_read: extension-authoring；A2 已注明经 02engine-guide 路由访问。
