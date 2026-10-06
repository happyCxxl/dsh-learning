# DSH 源码学习路线

面向 AI 全栈求职，重点掌握 **插件思想** 与 **harness 思想**。
主教材：`D:\application\dsh\deepseek-harness`（TypeScript monorepo）。

**学习方式约定**：全程**不修改 dsh 源码**。所有练习代码、测试与笔记都放在本目录（`deepseek-harness\.learning\`）下，已跟踪的源码保持零改动。

---

# 一、学习路线

## 1.1 总览

| 阶段 | 主题 | 目标 | 预计 |
|---|---|---|---|
| 1 | 全景与共同语言 | 能说清 dsh 是什么、一次请求经过哪些环节 | 0.5–1 天 |
| 2 | cordis 插件内核 | 掌握插件框架的可逆注册模型 | 1–2 天 |
| 3 | 在 harness 里写插件 | **写出一个能跑的真插件** | 2–4 天 |
| 4 | core 核心循环精读 | 掌握事件溯源与工具执行流水线 | 3–5 天 |
| 5 | 全栈产品化视角 | 理解同一内核如何长出多种产品形态 | 3–5 天 |

## 1.2 三条原则

1. **读 + 跑 + 自己写验证，缺一不可。** 只读源码答不出"如果让你改，你改哪"。每个阶段都配动手实验——但动手的意思是**写测试与写插件**，不是改 dsh 源码。
2. **每阶段必须有可展示产出物。** 求职看的是做过什么，不是读过什么。
3. **验收标准是"能讲清楚"。** 检查点都是自测问题——不看资料能讲明白才算过。

## 1.3 目录约定

```
deepseek-harness\.learning\
├── LEARNING-ROADMAP.md     ← 本文件
├── experiments\            ← 阶段 1–2 的探索性脚本（如最小 cordis 插件）
├── plugins\                ← 阶段 3 自己写的 dsh 插件（独立包，不进 workspace）
└── notes\                  ← 读书笔记、时序图、长文产出
```

`.learning\` 不在 `pnpm-workspace.yaml` 的 glob 范围内（只匹配 `packages/*/*`、`vendor/*`、`apps/*` 等），因此放东西进来**不会**影响仓库构建与 workspace。

## 1.4 为什么这样排

先建全景（阶段 1），因为"插件扩展的是什么"必须先有落点；再攻插件内核（阶段 2–3），这是你的第一诉求，且 `docs\cordis-tutorial\` 只有 36KB，投入产出比最高；然后深入 core 循环（阶段 4），这是 harness 思想的骨头；最后抬升到产品层（阶段 5），这是全栈岗位最关心的部分。

---

# 二、学习内容

## 2.1 素材总览

| 素材 | 路径 | 用途 |
|---|---|---|
| 设计笔记库 | `.agents\notes\` | **「为什么这么设计」的一手决策记录**（1197 篇中文笔记；`implemented` 227 篇，其中 `architecture` 141 篇）。按主题 grep，不通读 |
| 分版本升级指南 | `docs\upgrade-guide\` | 各子系统的契约变更 + 迁移步骤 + 验证方法（中英双语，6 份中文） |
| 源码 monorepo | `D:\application\dsh\deepseek-harness` | 主教材：`.ts` 源码 + 测试 + `docs\` 下 182 篇中文文档 |
| 仓库自用 agent 技能 | `.agents\skills\` | AI 原生仓库如何被 AI 开发（`dsh-code-review`、`dsh-doc`、`dsh-pre-push-checks`、`dsh-find-simplifications` 等 15 个） |
| 插件集合仓库 | `D:\application\dsh\dsh-plugins` | 5 个真实插件 + `docs\PLUGIN_SPEC.md`（第三方视角的插件规范） |
| 已安装的第三方插件 | `~\.dsh\profiles\desktop\node_modules\deepseek-idesign` | 一个正在本机运行的完整插件标本 |
| 安装目录内的打包产物 | `resources\app.asar` 内 `dsh\` | 每个包自带的 `README.zh.md` + 编译后 `lib/*.js`（无 `src/`，无测试） |

> **设计笔记库的用法**：按主题 grep（如 `cordis`、`fiber`、`effect`、`tool call`、`compaction`），读 `implemented/architecture` 里最新日期的十几篇。你会看到一套真实系统在权衡中成型的过程——这种"为什么"的材料比源码本身更难得。

## 2.2 阶段 1 · 全景与共同语言

**精读**（全中文，按顺序）
1. `docs\glossary.zh.md` — 术语表，先统一词汇
2. `docs\architecture.zh.md` — 总架构；第 145 行起有「目标 → 机制」对照表，值得反复查
3. `docs\agent-lifecycle.zh.md` — agent 生命周期时序
4. `docs\tool-execution-pipeline.zh.md` — 工具执行流水线
5. `docs\event-producer-consumer.zh.md` — 事件的生产者/消费者全景
6. `packages\core\README.zh.md` — core 分组地图
7. `docs\module-graph.zh.md`、`docs\graph-atlas.zh.md` — **当字典查，不要通读**
8. 抽读 `.agents\notes\implemented\architecture\` 最新 5–10 篇

**动手**（全部只读，不产生任何改动）
- 看真实插件树：`pnpm dsh --profile web --dump-config`
- 端到端跑一句：`pnpm dsh --profile headless "say hi"`
- 读一份升级指南体会"契约"长什么样：`docs\upgrade-guide\v0.2.0-rc.2\subpath-plugin-display-manifest\guide.zh.md`（27 行，"变更 / 迁移 / 确认"三段式）

**笔记**：在 `.learning\notes\` 下画你的第一版"一次请求全链路"图。

## 2.3 阶段 2 · cordis 插件内核

**精读**（先教程，再源码）
1. `docs\cordis-primer.zh.md` — 入门；五种事件分发模式是重点
2. `docs\cordis-tutorial\` **全 8 篇**（每篇 3–5KB，共约 36KB，可一口气读完）：
   `index.zh.md` → `01-first-plugin.zh.md` → `02-lifecycle-and-effects.zh.md` → `03-services.zh.md` → `04-events.zh.md` → `05-config.zh.md` → `06-composition-and-hmr.zh.md` → `07-into-the-harness.zh.md`
3. `docs\cordis-api\context.zh.md` — `ctx.extend` / `ctx.isolate` / `ctx.intercept`
4. 源码（**从下往上读**）：`vendor\cordis\src\` 下的 `service.ts` → `registry.ts` → `context.ts` → `fiber.ts`
5. `.agents\notes\` 里 grep `cordis`、`fiber`、`effect`，看设计者自己的解释

**核心要点**：cordis 里的一切注册都是**可逆副作用**——`ctx.effect(execute, label)` 返回 disposer，在 fiber 销毁时**逆序**执行。这是"全插件化、无特权内核"能成立的根本原因。

**动手**：在 `.learning\experiments\` 下写一个最小 cordis 插件（函数 + `apply(ctx)` + 一个 `ctx.effect`），照 `01-first-plugin.zh.md` 跑通。
`vendor\cordis` 已经构建好（`@deepseek-ai/cordis`，入口 `vendor\cordis\lib\index.js`），实验脚本直接引用它即可，**不需要改动仓库任何文件**。

## 2.4 阶段 3 · 在 harness 里写插件

**精读**
1. `docs\cookbook\extension-cookbook.zh.md` — 扩展点总览（本阶段的路线图）
2. `docs\capability-seams.zh.md` — 能力接缝：Definition / Provider / Consumer 三角色
3. `docs\cookbook\adding-a-tool.zh.md` → `packages\core\tools\src\index.ts`
4. 按兴趣再挑 2–3 个扩展点：
   - 命令 → `packages\interaction\commands\src\index.ts`
   - 设置卡片 → `packages\settings\settings\src\index.ts` + `docs\cookbook\adding-a-settings-card.zh.md`
   - LLM 适配器 → `docs\cookbook\adding-an-llm-adapter.zh.md`
   - 客户端 UI 槽位 → `packages\client\ui-slots\src\index.ts` + `docs\subsystems\slots.zh.md`
   - Host↔Client 通道 → `packages\host\webserver\src\index.ts`
5. `D:\application\dsh\dsh-plugins\docs\PLUGIN_SPEC.md` — 依赖策略与兼容性红线
6. **真实插件解剖**：
   - `dsh-plugins\plugins\dsh-peek\`：`package.json`(1.3KB) + `cordis.patch.yml`(540B) + `lib\index.js`(4.5KB) + `lib\client.js`(24KB)
   - `~\.dsh\profiles\desktop\node_modules\deepseek-idesign\` — 正在运行的插件

**插件最小结构**（从 dsh-peek 提取）
- `package.json`：`"type":"module"` + `main:"lib/index.js"`；`exports` 必含 `.` / `./client` / `./cordis.patch.yml` / `./package.json`；`dsh.bundle.patch:"./cordis.patch.yml"`；`dsh.client:{platform:"web"}`；`files` 白名单
- `cordis.patch.yml`：顶层数组，形如 `- insert: [{ id: <唯一id>, name: '<包名>' }]`
- `lib/index.js`：`export const name` + `export const inject = ['webServer']` + `export function apply(ctx)`，内部用 `ctx.effect(...)` 注册并在 disposer 里注销

**进阶机制**（读到能看懂即可）
- **子路径插件**：包可导出 `my-plugins/search` 这样的子路径作为一个"插件行"，标题/描述来自子路径 locale 文件，图标来自导出的 `./search/icon`。见 `docs\upgrade-guide\v0.2.0-rc.2\subpath-plugin-display-manifest\guide.zh.md`
- **插件兼容性拒绝**：装插件前看 `peerDependencies`；相关实现见 `packages\boot\app-boot\src\compatibility-preflight.ts`

**动手（本阶段核心，产物放 `.learning\plugins\<你的插件名>\`）**
1. 按上述结构建一个最小插件包——它是**独立包，不是 dsh 源码的一部分**
2. 打包并安装到 profile —— **注意**：`PLUGIN_SPEC.md` 明确禁止直接 add 本地目录，先读它给的原因
3. 重启 dsh，验证工具/命令/路由生效
4. 卸载，确认注册被干净撤销

## 2.5 阶段 4 · core 核心循环精读

**精读**
1. `packages\core\README.zh.md`
2. `packages\core\agent\README.zh.md` — Agent 句柄：`create`/`resume`/`followup`/`steer`/`inject`/`cancel`/`whenIdle`
3. `packages\core\session\README.zh.md` + `packages\core\session\src\index.ts` — 仅追加日志 + `deriveMessages()` 投影
4. `packages\core\system-prompt\README.zh.md` + `src\index.ts` — 有序段 + 变量 + 工具 schema 的权威组装
5. `packages\core\tools\README.zh.md` + `src\index.ts`
6. `packages\core\scope\README.zh.md` — 作用域隔离与事件路由
7. `packages\core\agent-loop\README.zh.md` — **含源码地图与设计理念，先读它再读代码**
8. 源码 `packages\core\agent-loop\src\`：
   - `agent.ts` — 驱动主体（轮次/步骤状态机）
   - `inbox.ts` — 收件箱与唤醒
   - `runtime-context.ts` — 运行时上下文投影
   - `tool-calls.ts` — 工具调度与并发（独占屏障 + 有界并行池）
   - `assistant-stream.ts`、`constants.ts`、`index.ts`
9. `docs\event-producer-consumer.zh.md`

**三个要彻底搞懂的设计**
1. **"模型可见即已记录"**：为什么请求必须先落日志才能发出。
2. **双轨事件**：持久 `session/*`（可回放）vs 实时 `agent/*`（含进程本地流式帧）vs 能力事件；waterfall 必须调 `next()`，serial 不调。
3. **工具三阶段**：`tool/call` → `tools/pre-execute` → 守卫/approval → `tools/execute` → `post-execute` → `tools/result`。设计笔记：`.agents\notes\implemented\architecture\2026-09-22-tool-call-three-phases.zh.md`、`2026-09-24-preparing-tool-arguments.zh.md`

**可选案例研究**：`.agents\notes\` 中关于运行时不变式机制**存与废**的笔记（`2026-07-19-package-owned-invariant-service.zh.md` 等），配合 `docs\upgrade-guide\v0.2.0-rc.2\remove-runtime-invariants\guide.zh.md`——一套机制为什么加、又为什么删，两边理由齐全，是练架构判断力的好材料。

**动手（靠测试观察，不改任何源码）**
- 直接跑仓库自带测试（零改动）：
  - `pnpm vitest run packages/core/agent-loop/tests/loop.spec.ts`
  - `pnpm vitest run packages/core/tools/tests/tools.spec.ts`
  - 配套 `packages\core\agent-loop\tests\mock-adapter.ts` 用脚本化模型响应驱动循环，**不需要真实 API**
- 想自己写断言：仿照 `packages\core\agent-loop\tests\loop.spec.ts` 的写法与位置（仓库测试运行器的约定所在地），断言事件顺序与状态迁移。这样写出的测试文件是**未跟踪新增文件**，已跟踪的源码仍与原版逐字节相同
- 前置条件：仓库根已有 `node_modules`；部分测试需先 `pnpm run build:native-system`

## 2.6 阶段 5 · 全栈产品化视角

**精读**
1. `packages\boot\app-boot\src\profile.ts` — profile 组合顺序与 `PROFILE_TEMPLATES` 定义（注意路径是 `packages\boot\`）
2. `packages\boot\hmr\README.zh.md` — 热重载边界：什么能热更、什么必须重启
3. `packages\boot\plugin-manager\src\index.ts` 与 `src\tools.ts` — 插件管理器；后者本身就是一份能力说明
4. `packages\bundle\` — 产品装配：`base` / `web-app` / `headless` / `sdk-app` / `sdk-minimal` / `acp-app`
5. `apps\` — 应用层：`dsh`（CLI 元包）、`dsh-desktop`、`dsh-desktop-host`（asar 里 `cli.js` 的宿主）、`dsh-web-frontend`
6. `docs\api-gateway.zh.md` + `packages\api\` — 后端 API 层
7. `packages\client\`（59 个客户端包，其中 `ui-*` 51 个）+ `docs\subsystems\`（64 篇中文子系统文档，按需查）
8. `packages\sdk\`（`protocol`/`server`/`client`）+ `packages\acp\`
9. `docs\cookbook\adding-a-remote-api.zh.md`、`docs\cookbook\adding-a-vendored-package.zh.md`
10. `docs\development.zh.md` + `docs\testing.zh.md`

**动手**（都是只读自省）
- 用仓库自带的自省脚本理解全貌（它们**从代码生成文档**，是极好的学习工具）：
  `pnpm run verify-module-graph`、`pnpm run verify-tool-catalog`、`pnpm run verify-config-catalog`、`pnpm run verify-persistence-catalog`
  对照生成物：`docs\module-graph.zh.md`、`docs\tool-catalog.zh.md`、`docs\config-catalog.zh.md`、`docs\persistence-catalog.zh.md`
- `pnpm run verify-application-entrypoints` — 看"所有 Node 应用入口必须经 dsh launcher"如何被强制
- 翻完 `docs\upgrade-guide\` 的 6 份指南，体会契约变更如何被治理

---

# 三、学习成果

## 3.1 各阶段验收检查点

**阶段 1**
- dsh 的"内核"是什么？为什么说它没有需要打补丁的特权内核？
- profile / bundle / patch 三层的组合顺序？
- 一次请求从 CLI 到模型流式返回，中间有哪些持久化事件？
- 工具执行前后各有哪些"把关"环节？

**阶段 2**
- 一个 cordis 插件有哪三种形态？`inject` 声明了什么、解决什么问题？
- `emit` / `waterfall` / `parallel` / `serial` / `bail` 的区别？哪个不调 `next()` 会短路？
- `ctx.effect` 的 disposer 执行顺序？这如何对应"卸载插件即撤销其注册"？
- Service 与它的 fiber 是什么关系？为什么随 fiber 自动注销很重要？

**阶段 3**
- dsh 暴露了哪 4 类以上扩展点？各自注册入口是什么？
- 插件为什么要声明 `inject`？不声明会怎样？
- 为什么 `bundle.patch` 层只在启动时读取、热重载只覆盖用户 patch 层？
- 子路径插件与包根插件的区别？

**阶段 4**
- 为什么 session 是"仅追加"的？压缩为什么是"遮蔽"而不是删除？
- `agent/request` 与 `llm/stream` 各负责什么？
- 一个工具结果从产生到写入 session 经过哪些 hook？在哪一步被守卫拦截？
- 工具并发的独占屏障解决什么问题？

**阶段 5**
- 从 `dsh-base` 到 `dsh-web-app`，一个产品怎么"装配"出来？`sdk-minimal` 为什么刻意不走 `dsh-base`？
- 换掉一个能力 provider（如 fs/sandbox）会怎样改变整个产品？
- 客户端插件（`dsh.client`）与宿主插件（`dsh.bundle`）的边界在哪？
- 新增一个远程 API 要动哪些包？

## 3.2 各阶段产出物

| 阶段 | 产出物 | 放哪 |
|---|---|---|
| 1 | 一张手绘的"一次请求全链路"图 | `.learning\notes\` |
| 2 | 最小 cordis 插件 demo + 笔记《cordis 的可逆注册为什么重要》 | `.learning\experiments\` + `notes\` |
| 3 | **一个自己写的、能跑起来的 dsh 插件**（含 README：用途/权限/副作用/兼容性） | `.learning\plugins\` |
| 4 | 一篇《dsh 一次请求的完整时序》技术长文（带真实事件名与源码位置） | `.learning\notes\` |
| 5 | 一篇《dsh 的多产品装配：profile / bundle / patch》+ 一份"新增一个前端形态要动哪些包"的方案设计 | `.learning\notes\` |

## 3.3 毕业检查清单

- [ ] 我能画出 dsh 分层图，并解释"没有特权内核"的含义
- [ ] 我能写出最小 cordis 插件，并解释 `ctx.effect` 的可逆性
- [ ] 我能写出 dsh 插件（工具或命令），走完打包→安装→验证→卸载
- [ ] 我能完整讲出一次请求的事件序列（含事件名）
- [ ] 我能解释"模型可见即已记录"，以及它带来的可回放性
- [ ] 我能解释能力接缝如何让同一内核长出多种产品
- [ ] 我能说清 profile/bundle/patch 三层组合顺序与热重载边界
- [ ] 我有 2 个以上可展示产出物（插件 + 技术长文）

## 3.4 成果去向：怎么用于求职

| 要证明的能力 | 用 DSH 的什么证明 |
|---|---|
| 能设计可扩展插件架构 | cordis 可逆注册 + 你的插件 demo + 对比分析（vs VS Code / Chromium extension） |
| 懂 AI Agent 工程 | 阶段 4 时序长文：事件溯源、工具把关流水线、并发调度、上下文压缩 |
| 懂 LLM 应用全栈 | 阶段 5 装配方案：同一内核 → CLI / Web / 桌面 / SDK |
| 会写可验证的测试 | 自己写的 agent-loop 断言测试（用 mock adapter 脚本化模型响应） |
| 工程素养与 AI 原生开发 | 60+ 个 `verify-*` 门禁脚本（用生成物替代手工文档）；`.agents\skills\` 里 15 个团队自用 agent 技能 |
| 架构判断力 | 不变式机制存废的完整论证（笔记 + 升级指南 + 迁移步骤）——"该删什么"比"会写什么"更稀缺 |

**面试可讲的四个反常识点**
1. 会话压缩是**遮蔽**而非删除——因为日志不可变。
2. 请求**先落日志再发送**，所以能从日志重建它做断言。
3. 插件卸载能撤销全部注册，因为每个注册都是带回滚的副作用，**连 agent loop 本身都能被配置替换**。
4. 他们**删掉了**运行时不变式机制，改成"隔离并记录监听器失败"——体现了"机制 vs 兜底"的成本权衡。

## 3.5 进度记录

| 阶段 | 状态 | 完成日期 | 产出物 |
|---|---|---|---|
| 1. 全景与共同语言 | ☐ 未开始 | — | 全链路手绘图 |
| 2. cordis 插件内核 | ☐ 未开始 | — | 最小 cordis demo + 笔记 |
| 3. 在 harness 里写插件 | ☐ 未开始 | — | 一个可运行插件 |
| 4. core 循环精读 | ☐ 未开始 | — | 时序长文 + 自写测试 |
| 5. 全栈产品化 | ☐ 未开始 | — | 装配方案设计 |
