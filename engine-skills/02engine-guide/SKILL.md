---
name: 02engine-guide
description: |
  当用户询问 02Engine 文档本身、编译器与性能、扩展调试与项目转换、addon 体系或术语时调用(这些未独立成 Skill):
  新编译器/老编译器回退/禁用编译器、warp timer、OPF与FPS调优、扩展热重载桥(VSCode Toolbox 1101)、 SB3转Node.js CLI、JS导出路径(leopardjs)、addon与ScratchX、globals、扩展相关术语。
  不适用于: 编辑器界面修改(窗口拖拽/主题/背景图/布局等，本skill不涉及)、原版Scratch/TurboWarp本体。
  以下意图优先用独立 Skill: 写扩展→extension-authoring；选型安装扩展→extension-library； 02Agent改项目→ai-agent-workflow；自定义积木/递归→custom-reporter；额外积木兼容→extra-blocks； #code内嵌JS→inline-js；云变量bot→cloud-variables。
  Triggers: 02Engine文档/编译器/性能/OPF/FPS/热重载/CLI转换/JS导出/addon/ScratchX/术语
metadata:
  cangjie.generated-by: cangjie-tools v2.5.0
  cangjie.variant: router
  cangjie.bundle-id: bundle.02engine-guide
  cangjie.capability-count: 10
  cangjie.entrypoint-count: 8
---
# 02Engine 文档（积木搭建与扩展使用） — 来源路由入口（compact pack）

## 触发与不触发

**适用**：与本书能力域相关的咨询与任务（见下方路由表的意图列）。
**不适用**：
- 编辑器界面修改类（New UI、编辑器窗口、背景设置、自定义默认项目、主题）
- 原版 Scratch 或 TurboWarp 本体的使用问题
- 02Engine 服务端部署、协作运维、仓库内部实现细节
- 打包器完整文档与 Web 兼容杂项（CORS/账号/翻译/捐赠）

## 核心原则（常驻速览，概览类问题读到这里即可回答）

1. 02Engine 兼容 Scratch 但更强大：额外积木以"警告线"划分兼容性，线后积木使项目无法上传Scratch官网。
2. 自定义报告积木(拖return)可嵌入别的积木，支持递归；报告积木应只计算无副作用，否则编译器/解释器行为不一致、递归过深栈溢出。
3. #code 注释是逃生舱：行首标记后写原生JS，可用target/runtime/block/vars/wait，长循环要yield、非常驻脚本末尾retire()+return。
4. 云变量独立服务器、上限10万字符但只存数字、会重置；bot须带User-Agent、单WebSocket事件驱动、回pong、更新≤10次/秒。
5. 扩展库按七类来源/关键词/兼容徽章选型，可单装或批量导入；非沙箱加载外部URL→只装可信来源。
6. 02Agent 用概览→只读→搜积木/扩展→小步改→getDiagnostics确认的流程改项目，禁止一次性重写大项目。
7. 写扩展按 hello-world→inputs→async→沙箱决策→dev server→APIs→hats→兼容→share 顺序；沙箱有1帧延迟且不能访问internals。
8. 非沙箱扩展用IIFE+'use strict'隔离、权限化API(Scratch.fetch/canFetch等)、register只调一次、不得死循环。
9. 向后兼容"只增不改"：绝不改ID/opcode/类型/参数/isTerminal/acceptReporters，必须破坏时新建ID。
10. 性能先调FPS再调OPF；新编译器靠类型分析提速但编辑器内被warp timer抵消、可能破坏少数扩展、故障回退老编译器。

## 能力路由（先读本表，按意图加载 1 张能力卡）

| 用户意图 | 先读 | 补读/备注 |
|---|---|---|
| 写一个 02Engine 扩展；自定义积木怎么加参数/菜单；异步积木怎么写；沙箱还是非沙箱怎么选；事件/帽积木怎么做；扩展怎么保持兼容和发布 | references/capabilities/extension-authoring.md | 已晋级为独立 Skill `extension-authoring`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 怎么找并装一个扩展；扩展库有哪些来源；怎么批量导入扩展；某扩展找不到怎么排查；扩展是否兼容Scratch | references/capabilities/extension-library.md | 已晋级为独立 Skill `extension-library`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 让AI改02Engine项目；02Agent怎么用；让AI搜索安装扩展；Skill bridge怎么连；AI改脚本没同步怎么办 | references/capabilities/ai-agent-workflow.md | 已晋级为独立 Skill `ai-agent-workflow`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 怎么做一个可嵌入的自定义积木；自定义积木怎么返回值；怎么用递归替代循环；自定义积木在编译器下行为不对 | references/capabilities/custom-reporter.md | 已晋级为独立 Skill `custom-reporter`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 02Engine多哪些积木；某积木能不能上传Scratch；怎么检测最后按下的键/各个鼠标键 | references/capabilities/extra-blocks.md | 已晋级为独立 Skill `extra-blocks`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 在积木项目里写JS；#code怎么用；用JS调积木或读变量；JS脚本卡住怎么办 | references/capabilities/inline-js.md | 已晋级为独立 Skill `inline-js`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 用云变量做排行榜/多人；写云变量bot；bot连不上或被禁；云变量为什么被禁用 | references/capabilities/cloud-variables.md | 已晋级为独立 Skill `cloud-variables`（已安装时优先直接使用；本卡仅作原文与背景补充） |
| 项目在新编译器下出错；怎么让项目跑更快；OPF/FPS怎么调；怎么临时关编译器调试 | references/capabilities/compiler-and-perf.md | references/capabilities/extension-authoring.md |
| 怎么热重载调试扩展；把项目转成CLI工具；怎么拿到项目JS；调试桥连不上 | references/capabilities/extension-debug-convert.md | references/capabilities/extension-authoring.md |
| 该写扩展还是addon；addon怎么兼容New UI；ScratchX旧扩展还能用吗 | references/capabilities/addon-and-legacy.md | references/capabilities/extension-authoring.md |

**非能力类查询**：
- 书名/作者/章节/整书概览 → references/overview.md
- 术语解释 → references/glossary.md
- 决策规则速查（不需要原文依据时） → references/cheatsheet.md
- 完整意图与关键词索引（本表未覆盖的意图先查这里） → references/capability-index.md

## 加载规则

- 每次任务先读本文件，再按路由表加载 **1** 张能力卡；任务明确跨域时最多加载 2 张。
- 概览/书名类问题不加载能力卡，用「核心原则」与 overview.md 回答。
- 路由表与 capability-index.md 都无法命中的意图，明确告知超出本书范围，不要硬套。

## 边界与判停

- 意图在路由表与能力索引均无法命中 → 说明超出"积木搭建与扩展使用"范围，不硬套。
- 用户问编辑器界面美化/布局 → 说明本skill聚焦积木与扩展，不覆盖界面修改。
- 涉及非沙箱外部URL/bridge等信任边界 → 如实强调只连可信来源、bridge只监听本机。
