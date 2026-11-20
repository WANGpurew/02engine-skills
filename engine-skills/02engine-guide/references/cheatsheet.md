# 决策规则速查 — 02Engine 文档（积木搭建与扩展使用）

| 能力 | 一句话规则 |
|---|---|
| 从零编写 02Engine 扩展的完整流程 | 按 hello-world→inputs→async→沙箱决策→dev server→APIs→hats→兼容→share 递进，非沙箱用IIFE+权限化API，兼容守"只增不改"。 |
| 扩展库选型、安装与来源排查 | 按来源/关键词/兼容徽章过滤扩展库→单装或批量导入→看徽章确认加载；外部URL扩展只装可信来源。 |
| 02Agent 小步改项目与 Skill bridge | 概览→只读目标→搜积木/扩展→小步改→getDiagnostics确认；bridge只监听127.0.0.1、中文参数用stdin/base64。 |
| 自定义报告积木、递归与纯函数式设计 | 扩展库启用Custom Reporters→建自定义积木拖入return即成报告积木；只计算无副作用，递归注意栈溢出。 |
| 02Engine 额外积木与 Scratch 兼容边界 | 警告线前的参数报告积木兼容Scratch，线后专属积木会使项目无法上传Scratch官网。 |
| 注释内嵌 JS 逃生舱（#code） | 帽子积木注释行首写#code即可把后面当原生JS跑；用target/runtime/block/vars/wait，长循环yield、末尾retire()+return。 |
| 云变量多人数据与合规 Bot 开发 | 云变量独立服务器、10万字符上限但只存数字且会重置；bot带User-Agent、单WebSocket事件驱动、回pong、≤10次/秒。 |
| 编译器选择、故障回退与 OPF 性能调优 | 新编译器靠类型分析提速但编辑器内被warp timer抵消、可能破坏少数扩展、故障回退老编译器；性能先FPS后OPF。 |
| 扩展热重载桥、SB3 转 CLI 与 JS 导出路径 | VSCode Toolbox桥(1101)热重载扩展；Packager转Node.js CLI用cli.getArg/log/exit；要可读JS用leopardjs。 |
| Addon 体系与旧扩展/ScratchX 背景 | 加积木写扩展、改编辑器UI写addon；addon只写活动编辑器且幂等注入；globals/ScratchX是背景。 |

> 速查只给结论；需要原文依据、案例或反例时读对应能力卡。
