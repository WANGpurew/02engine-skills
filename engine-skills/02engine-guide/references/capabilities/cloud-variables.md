<!-- capability_id: cap.e2.cloud-variables | revision: 1 | status: active -->

# 云变量多人数据与合规 Bot 开发

## R — 原文 (Reading)

> "The variable length limit has been increased to 100000 characters. Cloud variables can still only hold numbers. Cloud variables are reset whenever the server restarts or when no one is in the project for a short period of time, so things like leaderboards won't be saved very long. Cloud variables are disabled when the editor is opened."
>
> 译文：变量长度上限提高到 100000 字符，但云变量仍只能存数字；服务器重启或短时无人会重置，排行榜存不久；编辑器打开时云变量被禁用。
>
> — 02Engine 功能文档·cloud-variables / slug: cloud-variables（原文行250–256；源：candidates/framework.md f09、principle.md p08、case.md c06、counter-example.md ce11/ce12）

> "Bots must provide a valid User-Agent header in their connection. That includes contact information ... and the name and version of the cloud variable library being used ... Pretending to be a browser is not okay and is easy to detect. User-Agents that are lacking information will be blocked."
>
> 译文：bot 连接必须带有效 User-Agent 头，含联系方式与所用云变量库名/版本；伪装浏览器不允许且易检测，信息不全的 User-Agent 会被阻断。
>
> — 02Engine 功能文档·cloud-variables / slug: cloud-variables（原文行274/302；源：candidates/framework.md f10、principle.md p09、case.md c05、counter-example.md ce14）

> "Instead, libraries should offer event-driven APIs. Instead of constantly asking the server for the latest values, open exactly one WebSocket, and let the server send updates as they happen. ... The server will periodically send a WebSocket ping frame, and you must respond with a pong or else the connection will drop."
>
> 译文：库应提供事件驱动 API——不要反复向服务器要最新值，只开一个 WebSocket、让服务器有更新即推送；服务器周期性发 ping 帧，必须回 pong，否则连接断开。
>
> — 02Engine 功能文档·cloud-variables / slug: cloud-variables（原文行337/354；源：candidates/framework.md f10、principle.md p11/p13、case.md c07、counter-example.md ce15/ce16）

## I — 方法论骨架 (Interpretation)

把"用云变量做多人/排行榜"拆成数据设计 + bot 合规两层：
① **独立云服**：协议与 Scratch 云变量相同，服务器 `wss://clouddata.02engine.org`，官方 Node.js 参考库非必须；
② **数据硬约束**：仍只存数字、长度上限 10 万字符、服务器重启/短时无人即重置（排行榜不持久）、编辑器打开时禁用、无公开历史日志、勿做无审核聊天室；
③ **连带禁用**：用云变量或自定义扩展会禁用视频侦测的颜色侦测（修复 Scratch 隐私警告的同类问题）；
④ **User-Agent**：bot 连接必须带人类可读的有效 UA（联系方式 + 库名/版本），库应强制暴露设置 API；浏览器内 bot 例外（自动带 Origin），伪装浏览器不允许且易检测，缺信息会被封；
⑤ **Project ID 与用户名**：ID 不限数字、可为任意文本，非 Scratch 项目用 `example.com/my-project` 便于核验、库选项用字符串（无意义 ID 大量活动会被禁用）；用户名推荐 `player` + 2–7 位随机数（连接更快、不校验 Scratch API）；
⑥ **连接模型**：事件驱动、恰好一个 WebSocket、让服务器推更新，禁快速开关连接；更新被缓冲（不保序、可丢），每秒 >10 次完全冗余；收到 ping 必回 pong；
⑦ **排障**：记录 WebSocket close code（4XXX）与无效 JSON 原始文本。
核心洞察：云变量是"只存数字、会重置、编辑器内禁用"的共享状态通道，不是持久存储；bot 的合规核心是"可追责的身份（UA/ID）+ 单连接事件驱动 + 保活（pong）+ 限频"。

## A1 — 书中的应用 (Past Application)

### 案例一：bot 的 User-Agent 与 wss 连接（worked_example，slug: cloud-variables / c05）
- **问题**：用 Node.js 写 02Engine 云变量 bot，须带有效 User-Agent（联系方式 + 库名/版本）才能连接。
- **方法执行**：文档给出好 UA 示例（如 `multiplayer leaderboard bot by https://scratch.mit.edu/users/TestMuffin`），库应暴露强制设置 UA 的 API；用 Node.js ws 客户端连接到 `wss://clouddata.02engine.org`，在 headers 写 `user-agent`：`const ws = new WebSocket("wss://clouddata.02engine.org", { headers: { "user-agent": userAgentGoesHere } });`。浏览器内 bot 例外（自动带 Origin），伪装浏览器不允许且易检测。
- **结论**：文档所述效果——有效 User-Agent 是连接必需，缺信息者被阻断、伪装浏览器易被检测；库应强制用户填联系方式，缺失则给明确错误。

### 案例二：事件驱动单 WebSocket 取代轮询开关（worked_example，slug: cloud-variables / c07）
- **问题**：反例用 `while True: cloudlibrary.get_var(...)` 每次开关连接轮询变量，bot 慢且耗网络/CPU。
- **方法执行**：改为注册回调 `def on_set(name, value): ...` 并 `connection = cloudlibrary.connect(project_id, username, user_agent, on_set)`，只开一个 WebSocket、由服务器推送更新；`get_var` 若要用，内部也须单连接、事件驱动、只返回最近值。
- **结论**：文档所述效果——单 WebSocket 在变量无变化时保持空闲、有变化即时到达；反复开关连接被文档明确判为不允许。

### 案例三：字符串 Project ID 与 player+随机用户名（scenario，slug: cloud-variables / c06）
- **问题**：bot 如何取合法 Project ID、用户名怎样取连接更快。
- **方法执行**：非 Scratch 项目用 `example.com/my-project` 形式便于核验，库选项设为字符串而非整数；用户名推荐 `player` + 2~7 位随机数（服务器不向 Scratch API 校验，连接启动更快）。
- **结论**：文档所述效果——随机 player 用户名跳过 Scratch API 校验、连接更快；无意义 ID 的大量活动会被禁用，故库 ID 选项须用字符串。

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?
1. 要用云变量做排行榜、多人状态/同步或自定义云变量客户端。
2. 要写一个云变量 bot、自定义库，或排查 bot 连不上、被拒、被封、频繁掉线。
3. 发现云变量在编辑器里不生效、排行榜数据存不久、或颜色侦测突然被禁用。
4. 想知道变量名长度上限、能存什么类型、更新频率多少合适。

### 语言信号
- "云变量 / 排行榜 / 多人 / cloud variables / clouddata / 10万字符"
- "bot / 机器人 / User-Agent / 联系方式 / 伪装浏览器"
- "WebSocket / 单连接 / 事件驱动 / 轮询 / get_var / pong / ping / close code / 4XXX"
- "Project ID / example.com/my-project / player 用户名 / 更新太频繁 / 视频侦测禁用"

### 与相邻 skill 的区分
- 与 `extra-blocks`（also_read）：本卡管云变量数据与 bot；extra-blocks 管内置额外积木与 Scratch 兼容边界。视频颜色侦测被禁用这一点与"用自定义扩展"相关，但根因判定在本卡。
- 与 `extension-authoring`：本卡是云变量数据/bot 协议；build 卡是从零写扩展。若 bot 逻辑要封装成扩展积木，跨到 build。
- 与 `extension-library`：自定义扩展与云变量都会禁用视频颜色侦测；扩展选型/来源可信度归 extension-library。

## E — 可执行步骤 (Execution)

**输入契约**：一个要用云变量的多人/排行榜意图（状态能编码成数字）+ bot 的连接参数（Project ID、用户名、User-Agent）+（可选）所用 WebSocket 库与事件回调。
1. **核对数据可行性**：确认多人状态可编码成数字、单变量 ≤100000 字符；接受会重置、编辑器打开时禁用、只存数字。完成标准：状态能用数字表达且不做长期存储假设；判停：需存文本/长期存档 → 放弃云变量或自行数字编码、另找持久化。
2. **核对连带影响**：确认项目是否同时用视频侦测颜色侦测/自定义扩展（会被禁用颜色侦测）。完成标准：已确认是否接受颜色侦测不可用。
3. **设置身份**：bot 带有效 User-Agent（联系方式 + 库名/版本，库强制暴露 API），Project ID 用字符串（非 Scratch 项目用 `example.com/my-project`），用户名用 `player` + 2~7 位随机数。完成标准：身份信息齐全且人类可读；判停：缺 UA 或伪装浏览器 → 会被阻断/检测，补全后再连。
4. **建单连接事件驱动**：恰好开一个 WebSocket 到 `wss://clouddata.02engine.org`，注册 on_set 回调接收推送，不轮询 get_var、不快速开关连接。完成标准：连接空闲时不耗资源、变量变化时即时回调。
5. **保活与限频**：启用 ping/pong 支持，收到 ping 必回 pong；把更新频率控制在 ≤10 次/秒（>10 次完全冗余）。完成标准：连接稳定不掉线、无冗余写入。
6. **记录排障信息**：记录 close code（4XXX）与无效 JSON 原始文本。完成标准：连接异常时有可诊断日志。

**输出契约**：云变量数据设计（数字编码方案 + 上限/重置/禁用约束说明）+ bot 合规配置（User-Agent / 字符串 Project ID / player 随机用户名）+ 单 WebSocket 事件驱动连接实现（含 on_set 回调、pong 保活、≤10 次/秒限频）+ close code/无效 JSON 排障日志 + 视频颜色侦测连带影响确认。

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill
- Scratch 官网云变量（本卡仅指 02Engine 独立云服）。
- 纯本地状态/变量 → 用普通积木，不必上云。
- 需要持久存档或存文本/对象 → 云变量只存数字且会重置，另找方案。

### 作者在文档中警告的失败模式
- **编辑器打开时禁用**：编辑器开着时云变量不生效，联机调试排行榜会失败。
- **只存数字、会重置**：重启或短时无人即清空，排行榜存不久；无公开历史日志，勿做无审核聊天室。
- **视频颜色侦测被禁用**：用云变量或自定义扩展会禁用视频侦测的颜色侦测。
- **bot 无 UA/伪装浏览器被封**：有效 User-Agent 必需，信息不全会被 block、伪装浏览器易被检测；浏览器内 bot 例外（带 Origin）。
- **快速开关连接违规**：get_var 式反复开关连接慢且耗资源，被明确判为不允许，须单 WebSocket 事件驱动。
- **>10 次/秒冗余**：服务器缓冲更新、不保序、可丢，超频写入无收益。
- **不回 pong 掉线**：忽视服务器 ping 会被断连。

### 文档的盲点 / 局限
- 服务器地址固定 `wss://clouddata.02engine.org`，协议与 Scratch 云变量一致但服务器独立。
- 官方提供 Node.js 参考库但非必须，可用任意 WebSocket 库，库作者须强制设置 UA 的 API。
- 用户名不能冒充 Scratch 团队成员（含 ScratchCat）；服务器拒绝非 Scratch 现有账户的用户名（bot 用 player+随机数规避校验）。
- 排障只能靠 close code 与无效 JSON 原文，服务器不提供公开历史日志。

### 容易混淆的邻近方法论
- 02Engine 独立云服 vs Scratch 官网云变量：地址/规则不同，本卡只讲前者。
- 普通变量（10 万字符上限）vs 云变量"仍只存数字"：长度放宽不等于类型放宽。
- 事件驱动单 WebSocket（推送）vs get_var 轮询（反复开关）：后者被明确禁止。
- 浏览器内 bot 的 UA 例外 vs 非浏览器 bot 必须设 UA：别把例外当通用许可。

---

## 相关 skills
- composes-with: `extra-blocks`
- depends-on: 无

## 审计信息
- 来源: summary-blocks.md §5（cloud-variables）；candidates/framework.md f09/f10、principle.md p08–p14、case.md c05–c07、counter-example.md ce11–ce17；slug: cloud-variables（原文行250/252/254/256/262/274/302/306/316/337/350/354）
- 引用回查: R 段三处英文引用逐字核对 `02engine-focused.txt` cloud-variables 页，均 ≤100 词；中文翻译忠实原意。
- 元数据: verified.yaml `cap.e2.cloud-variables`（status active / importance medium / destination promoted）；also_read: extra-blocks。
