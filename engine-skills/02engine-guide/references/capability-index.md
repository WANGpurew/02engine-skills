# 能力索引（完整版）

| capability_id | 标题 | 重要度 | 意图 | 关键词 | 能力卡 |
|---|---|---|---|---|---|
| cap.e2.build-extension | 从零编写 02Engine 扩展的完整流程 | critical | 写一个 02Engine 扩展；自定义积木怎么加参数/菜单；异步积木怎么写；沙箱还是非沙箱怎么选；事件/帽积木怎么做；扩展怎么保持兼容和发布 | 扩展、getInfo、register、blockType、沙箱、非沙箱、IIFE、hats、向后兼容、extension | capabilities/extension-authoring.md |
| cap.e2.extension-library | 扩展库选型、安装与来源排查 | high | 怎么找并装一个扩展；扩展库有哪些来源；怎么批量导入扩展；某扩展找不到怎么排查；扩展是否兼容Scratch | 扩展库、安装扩展、批量导入、来源、徽章、兼容、收藏、extension library | capabilities/extension-library.md |
| cap.e2.agent-workflow | 02Agent 小步改项目与 Skill bridge | high | 让AI改02Engine项目；02Agent怎么用；让AI搜索安装扩展；Skill bridge怎么连；AI改脚本没同步怎么办 | 02Agent、AI改项目、DSL、虚拟文件、getDiagnostics、Skill bridge、applyPatch、小步改 | capabilities/ai-agent-workflow.md |
| cap.e2.custom-reporter | 自定义报告积木、递归与纯函数式设计 | high | 怎么做一个可嵌入的自定义积木；自定义积木怎么返回值；怎么用递归替代循环；自定义积木在编译器下行为不对 | 自定义积木、报告积木、return、递归、栈溢出、副作用、reporter | capabilities/custom-reporter.md |
| cap.e2.extra-blocks | 02Engine 额外积木与 Scratch 兼容边界 | medium | 02Engine多哪些积木；某积木能不能上传Scratch；怎么检测最后按下的键/各个鼠标键 | 额外积木、is compiled、is 02Engine、last key pressed、mouse button、兼容、Scratch | capabilities/extra-blocks.md |
| cap.e2.direct-js | 注释内嵌 JS 逃生舱（#code） | medium | 在积木项目里写JS；#code怎么用；用JS调积木或读变量；JS脚本卡住怎么办 | #code、内嵌JS、逃生舱、target、opcode、yield、retire、generator | capabilities/inline-js.md |
| cap.e2.cloud-variables | 云变量多人数据与合规 Bot 开发 | medium | 用云变量做排行榜/多人；写云变量bot；bot连不上或被禁；云变量为什么被禁用 | 云变量、排行榜、bot、User-Agent、WebSocket、pong、clouddata | capabilities/cloud-variables.md |
| cap.e2.compiler-and-perf | 编译器选择、故障回退与 OPF 性能调优 | medium | 项目在新编译器下出错；怎么让项目跑更快；OPF/FPS怎么调；怎么临时关编译器调试 | 新编译器、warp timer、老编译器、OPF、FPS、禁用编译器、性能 | capabilities/compiler-and-perf.md |
| cap.e2.extension-debug-convert | 扩展热重载桥、SB3 转 CLI 与 JS 导出路径 | medium | 怎么热重载调试扩展；把项目转成CLI工具；怎么拿到项目JS；调试桥连不上 | 热重载、VSCode Toolbox、CLI、cli.getArg、LeopardJS、JS导出 | capabilities/extension-debug-convert.md |
| cap.e2.addon-and-legacy | Addon 体系与旧扩展/ScratchX 背景 | low | 该写扩展还是addon；addon怎么兼容New UI；ScratchX旧扩展还能用吗 | addon、活动编辑器、DOM发现、globals、ScratchX、sbx | capabilities/addon-and-legacy.md |
