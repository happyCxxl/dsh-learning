# DSH 源码与思想学习路线

**根本目的**：深刻掌握 DSH 的源码、harness 思想与其中的前沿技术。
**性质**：**活文档**。骨架会随学习进度增删——每学完一编，回来修订下一编的步骤。
**约定**：全程**不修改 DSH 源码**；所有产物（笔记、图、脚本、插件）放在本目录 `.learning\` 下。

---

# 一、学习路线

## 1.1 定位

| 项 | 说明 |
|---|---|
| 主教材 | DSH 的 TypeScript / Node 源码本身 |
| 主线 | 结构装配 → 插件内核 → 运行内核 → 思想与前沿 |
| 语言线 | **缺什么补什么**，不前置成课程。每步标注该步涉及的语言点，附录 A 是可检索的索引 |
| Java 的用法 | 仅在概念确实同构时作为**辅助类比**出现，不承担讲解职责 |
| SDK 应用 | 后期目标，**不参与倒推**；路线由"源码与思想"驱动 |
| 深度 | 到实现层（状态机、投影、并发、冻结）。**每一步的具体深度在学到该步时再定** |

## 1.2 怎么学（方法约定）

1. **源码为准。** 任何结论都要能定位到文件与行；文档与源码冲突时以源码为准。
2. **先现象、后机制、再取舍。** 每步都问三层：它做了什么 → 怎么做到的 → 为什么这么设计。
3. **允许暂时不懂。** 每步留"待解答"，很多问题会在后续步骤自动消解。
4. **动手即学习。** 动手 = 写脚本验证 / 写测试 / 画图 / 写插件，不是改 DSH 源码。
5. **每一步都要有可复述的结论。** 讲不出来就不算学完。

## 1.3 颗粒度

| 级别 | 数量 | 粒度 |
|---|---|---|
| L1 编 | 4 | 一个能力域 |
| L2 步骤 | 38 | 每个 0.5–1 天，只聚焦 1–2 个文件，可独立验收 |
| L3 每步内容 | 固定四项 | **读什么** · **做什么** · **关键问题（附答案）** · **产出物** |

完成标准只有一条：**该步的讲解与关键问题的答案已落入 `notes\`**，且产出物已落盘。所有问题一律附带答案，不需要自行作答。

## 1.4 已知将来要增补的编（现在不做，避免摊大）

- **插件 UI 线**：客户端插件的槽位、渲染、状态同步（等第二编学完后决定是否单列）
- **SDK / 协议线**：用 SDK 做应用、JSON-RPC 协议、ACP 集成（等第三编学完后决定）
- **持久化与存储线**：存储实现、SQLite、远程存储（等第三编 3.1–3.3 学完后评估）

## 1.5 步骤总览

| 编 | 步骤 | 主题 |
|---|---|---|
| **一 结构与装配** | 1.1–1.6 | 术语分层 · 装配顺序 · id 覆盖 · patch 语法 · 依赖方向 · 形态对比 |
| **二 插件内核 cordis** | 2.1–2.9 | 核心对象 · 依赖注入 · 可逆副作用 · 事件模型 · 作用域 · 配置组合 · 热重载 · 扩展点 · 写插件 |
| **三 运行内核** | 3.1–3.16 | 会话格式 · 事件全景 · 投影 · Agent 句柄 · 状态机 · 请求冻结 · LLM 层 · 流式重试 · 工具注册 · 工具流水线 · 并发调度 · 压缩 · spill 恢复 · 子代理 · 目标计划 · 调度可观测 |
| **四 思想与前沿** | 4.1–4.7 | 无特权内核 · 事件溯源取舍 · 可见即已记录 · 能力接缝 · 契约治理 · 横向对比 · 前沿技术 |

---

# 二、学习内容

## 2.1 素材索引（随用随查，不通读）

| 素材 | 路径 |
|---|---|
| 术语表 | `docs\glossary.zh.md` |
| 总架构（含「目标 → 机制」对照表） | `docs\architecture.zh.md` |
| 模块图（搜索用） | `docs\module-graph.zh.md` |
| 工具目录 / 配置目录 / 持久化目录 | `docs\tool-catalog.zh.md`、`docs\config-catalog.zh.md`、`docs\persistence-catalog.zh.md` |
| 事件全景 | `docs\event-producer-consumer.zh.md` |
| 工具流水线 | `docs\tool-execution-pipeline.zh.md` |
| agent 生命周期 | `docs\agent-lifecycle.zh.md` |
| cordis 教程（8 篇） | `docs\cordis-tutorial\` |
| cordis API | `docs\cordis-api\context.zh.md` |
| 扩展点手册 | `docs\cookbook\extension-cookbook.zh.md` |
| 能力接缝 | `docs\capability-seams.zh.md` |
| 测试说明 | `docs\testing.zh.md` |
| 升级指南（6 份） | `docs\upgrade-guide\` |
| 设计笔记（"为什么"） | `.agents\notes\` |
| 子系统文档（64 篇） | `docs\subsystems\` |
| 一个真实插件（对照） | `~\.dsh\profiles\desktop\node_modules\deepseek-idesign` |
| 第三方插件规范 | `D:\application\dsh\dsh-plugins\docs\PLUGIN_SPEC.md` |
| 本机产品 CLI | `D:\application\dsh-desktop\resources\runtime\cli\bin\dsh.cmd` |
| 仓库源码 CLI | 仓库根执行 `pnpm dsh --profile <name> ...` |

---

## 第一编 · 结构与装配

### 1.1 术语与分层地图
- **读**：`docs\glossary.zh.md`；`docs\architecture.zh.md`（开头 + 第 147 行起对照表）；`packages\core\README.zh.md`
- **做**：抄一张名词对照表，不懂的标问号
- **验收**：`profile` / `bundle` / `patch` 各是什么？`ctx` / `service` / `plugin` 什么关系？
- **产出**：`notes\1-结构与装配\1.1-名词与分层.md`

### 1.2 profile 与 patch 的组合顺序
- **读**：`packages\boot\app-boot\src\profile.ts`（开头注释与 `PROFILE_TEMPLATES`）
- **做**：dump 本机 web profile 的插件树，标出全部层边界
  ```powershell
  D:\application\dsh-desktop\resources\runtime\cli\bin\dsh.cmd --profile web --dump-config > $env:TEMP\web-tree.yml
  ```
- **验收**：三层叠加的准确顺序？`--dump-config` 与 `--dump-default-config` 差哪一层？
- **产出**：`notes\1-结构与装配\1.2-装配顺序.md`

### 1.3 bundle 层与 id 覆盖规则
- **读**：`packages\boot\app-boot\src\profile-plugins.ts`；任一 bundle 的 `cordis.patch.yml`
- **做**：找出同一个 `id` 出现在多层的实例，验证谁生效
- **验收**：`id` 与 `name` 的语义差别？为什么 `id` 必须稳定？`disabled` 为什么允许写成表达式？
- **产出**：`notes\1-结构与装配\1.3-id与覆盖规则.md`
- **语言点**：配置里的表达式在运行时求值

### 1.4 cordis.yml / cordis.patch.yml 语法
- **读**：`~\.dsh\profiles\desktop\cordis.yml` 与 `cordis.patch.yml`；`docs\cordis-tutorial\05-config.zh.md`
- **做**：在**你自己的 profile** 里加一条 `insert`，观察是否生效（改的是 profile，不是源码）
- **验收**：`insert` 等操作的语义是什么？patch 与配置的区别？
- **产出**：`notes\1-结构与装配\1.4-patch语法.md`

### 1.5 依赖方向与分层门禁
- **读**：`pnpm-workspace.yaml`；`packages\` 目录结构；`scripts\` 下的依赖类校验脚本
- **做**：挑一条链路画出依赖方向（如 Web 请求 → API → 会话 → 存储）
- **验收**：`packages/core` 与 `packages/bundle` 职责差别？为什么 `apps/` 在最上层？门禁脚本在防什么？
- **产出**：`notes\1-结构与装配\1.5-分层与依赖方向.md`

### 1.6 多形态装配对比
- **读**：`packages\bundle\` 下 `base` / `web-app` / `headless` / `sdk-app` / `sdk-minimal` / `acp-app` 的 `cordis.patch.yml`
- **做**：做对比表（各自装了什么、少了什么）
- **验收**：从 `base` 到 Web 产品中间加了什么？`sdk-minimal` 为什么刻意不带 `base`？
- **产出**：`notes\1-结构与装配\1.6-形态对比.md`

---

## 第二编 · 插件内核 cordis

### 2.1 四个核心对象
- **读**：`vendor\cordis\src\context.ts`、`registry.ts`（先看类型与导出）
- **做**：画出 `Context` / `Registry` / `Fiber` / `Service` 的关系图
- **验收**：`ctx` 是什么？插件登记在哪？`Fiber` 代表什么？
- **产出**：`notes\2-插件内核\2.1-四个核心对象.md`
- **语言点**：`Proxy` / `Reflect` —— `ctx` 的实现基础

### 2.2 依赖注入与 inject
- **读**：`vendor\cordis\src\registry.ts`、`service.ts`；本机某插件的 `inject` 声明
- **做**：写插件 A/B，B 依赖 A 的服务，观察加载顺序；再删掉 `inject` 看会怎样
- **验收**：`inject` 解决什么问题？服务未就绪时行为如何？可选依赖怎么写？
- **产出**：`experiments\inject-demo.mjs` + `notes\2-插件内核\2.2-依赖注入.md`
- **语言点**：`Symbol` 作为服务标识

### 2.3 生命周期与可逆副作用
- **读**：`vendor\cordis\src\fiber.ts`（只看 `effect`、`dispose`、状态机三处）
- **做**：注册嵌套 `ctx.effect`，打印 disposer 的执行顺序
- **验收**：`effect` 回调立即执行还是延迟？disposer 逆序如何保证依赖先销毁？三态迁移条件？
- **产出**：`experiments\effect-order.mjs` + `notes\2-插件内核\2.3-可逆副作用.md`
- **语言点**：闭包；`Symbol.asyncDispose`（若实现中用到）

### 2.4 五种事件分发
- **读**：`docs\cordis-primer.zh.md`（分发模式一节）；`vendor\cordis\src\` 中事件实现
- **做**：对同一事件分别用 `emit` / `waterfall` / `serial` / `parallel` / `bail` 注册监听器，观察调用与返回值差异
- **验收**：五种模式差别？哪个不调 `next()` 会短路？返回值如何聚合？
- **产出**：`experiments\events-demo.mjs` + `notes\2-插件内核\2.4-事件模型.md`

### 2.5 作用域：extend / isolate / intercept
- **读**：`docs\cordis-api\context.zh.md`；`vendor\cordis\src\context.ts`
- **做**：构造父子作用域，验证"近者优先 / 遮蔽"
- **验收**：三者各改变什么？为什么需要"按会话隔离注册"？
- **产出**：`notes\2-插件内核\2.5-作用域.md`

### 2.6 配置与组合
- **读**：`docs\cordis-tutorial\05-config.zh.md`、`06-composition-and-hmr.zh.md`
- **做**：给自定义插件加 `config`，再用 patch 覆盖它
- **验收**：插件如何声明与读取配置？配置与 patch 如何配合？
- **产出**：`notes\2-插件内核\2.6-配置与组合.md`
- **语言点**：Schema 校验库（DSH 用 schemastery）

### 2.7 热重载边界
- **读**：`packages\boot\hmr\README.zh.md`、`packages\boot\hmr\src\index.ts`
- **做**：改动 profile patch，观察哪些改动热生效、哪些必须重启
- **验收**：哪些改动热生效、哪些必须重启？HMR 在监听什么变化？（2.7 已核实：「bundle 层只在启动时读取」这一说法不准确，真正的判据见 `notes\2-插件内核\2.7-热重载.md`）
- **产出**：`notes\2-插件内核\2.7-热重载.md`

### 2.8 DSH 扩展点全景
- **读**：`docs\cookbook\extension-cookbook.zh.md`；`docs\capability-seams.zh.md`
- **做**：列表化：扩展点 / 注册入口源码 / 对应 cookbook
- **验收**：至少说出 5 类扩展点及入口？能力接缝的三角色是什么？
- **产出**：`notes\2-插件内核\2.8-扩展点地图.md`

### 2.9 写插件全流程
- **读**：`docs\cookbook\adding-a-tool.zh.md`；`packages\boot\app-boot\src\compatibility-preflight.ts`；`PLUGIN_SPEC.md`
- **做**：写工具插件 → 打包 → 安装到自己的 profile → 验证 → 卸载；再故意造一个不兼容版本，观察拒绝与豁免
- **验收**：`dsh.bundle` 与 `dsh.client` 的区别？为什么装完要重启？兼容性判定依据？豁免为什么必须显式？
- **产出**：`plugins\<你的插件>\` + `notes\2-插件内核\2.9-插件全流程.md`
- **语言点**：`package.json` 的 `exports` 子路径与条件导出（插件契约的根）

---

## 第三编 · 运行内核

### 3.1 会话格式与仅追加日志
- **读**：`packages\session\session-format\`；`docs\session-format-status.zh.md`
- **做**：找到本机会话文件，确认它是**分帧压缩的追加日志**
- **验收**：为什么只能追加？格式版本如何演进（迁移链）？
- **产出**：`notes\3-运行内核\3.1-会话格式.md`
- **语言点**：JSONL；zstd 帧结构

### 3.2 事件类型全景
- **读**：`docs\event-producer-consumer.zh.md`
- **做**：把事件按"谁发出 / 谁消费 / 是否持久"归类成表
- **验收**：持久事件与实时事件如何区分？waterfall 与 serial 的调用约定差别？
- **产出**：`notes\3-运行内核\3.2-事件全景.md`

### 3.3 投影与消息推导
- **读**：`packages\session\session-projection\`；`packages\core\session\src\index.ts`
- **做**：说明"模型看到的消息"是如何从事件算出来的
- **验收**：为什么不直接存消息？投影需要显式注册吗？缺失会怎样？
- **产出**：`notes\3-运行内核\3.3-投影与消息.md`

### 3.4 Agent 句柄与生命周期
- **读**：`packages\core\agent\README.zh.md` 与 `packages\core\agent\src\`
- **做**：列出 Agent 的全部公开方法与语义
- **验收**：为什么外部只能通过句柄交互？`create` 与 `resume` 的差别？
- **产出**：`notes\3-运行内核\3.4-Agent句柄.md`

### 3.5 turn / step 状态机与 inbox
- **读**：`packages\core\agent-loop\src\agent.ts`、`inbox.ts`
- **做**：画状态迁移图，标出 `followup` / `steer` / `inject` 的插入点
- **验收**：turn 与 step 什么关系？inbox 解决什么？取消如何生效？
- **产出**：`notes\3-运行内核\3.5-状态机与inbox.md`

### 3.6 请求构造与请求冻结
- **读**：`packages\core\agent-loop\src\runtime-context.ts`；`packages\core\system-prompt\src\index.ts`
- **做**：按顺序列出一次请求被拼装的全部输入
- **验收**："先落日志再发出"的意义？冻结发生在哪一步？系统提示词如何分段与遮蔽？
- **产出**：`notes\3-运行内核\3.6-请求构造.md`

### 3.7 LLM 层：适配器与路由
- **读**：`packages\llm\llm\src\index.ts`；`packages\llm\llm-deepseek\`；`docs\cookbook\adding-an-llm-adapter.zh.md`
- **做**：说明"换一个模型供应商"要动哪些地方
- **验收**：适配器接口边界在哪？默认值与路由怎么定？
- **产出**：`notes\3-运行内核\3.7-LLM适配层.md`

### 3.8 流式、重试与计量
- **读**：`packages\llm\llm-retry\`；`packages\llm\token-meter\`；`packages\core\agent-loop\src\assistant-stream.ts`
- **做**：对比"流式帧"与"持久消息"的关系
- **验收**：失败与取消如何被记录？token 如何计量与上报？
- **产出**：`notes\3-运行内核\3.8-流式与重试.md`
- **语言点**：`AsyncIterable` / 异步生成器；Node Stream 与背压

### 3.9 工具注册表与 schema DSL
- **读**：`packages\core\tools\src\index.ts`；`docs\cookbook\adding-a-tool.zh.md`
- **做**：摘一个真实工具的声明，逐字段解释
- **验收**：schema 从哪来？工具模式（native / ptc / both）差别？参数如何校验？
- **产出**：`notes\3-运行内核\3.9-工具注册表.md`

### 3.10 工具执行流水线与把关
- **读**：`docs\tool-execution-pipeline.zh.md`；`packages\guard\`；`packages\interaction\user-approval\`
- **做**：跑一次需要审批的调用，记录每个把关点的介入条件
- **验收**：一次工具调用经过哪些阶段？守卫与审批各防什么？
- **产出**：`notes\3-运行内核\3.10-工具流水线.md`

### 3.11 工具并发调度
- **读**：`packages\core\agent-loop\src\tool-calls.ts`
- **做**：说明"独占屏障 + 有界并行池"为什么必要，什么工具必须独占
- **验收**：并行度如何限制？乱序结果如何回填？
- **产出**：`notes\3-运行内核\3.11-并发调度.md`

### 3.12 上下文压缩
- **读**：`packages\compaction\compaction-basic\`；`packages\context\`
- **做**：说明触发条件与"遮蔽而非删除"的落地方式
- **验收**：何时触发压缩？原始事件还在吗？模型看到的内容如何变化？
- **产出**：`notes\3-运行内核\3.12-上下文压缩.md`

### 3.13 spill 与恢复
- **读**：`packages\spill\`；`packages\session\session-persistence-jsonl\`
- **做**：说明中断/重启后如何恢复，哪些内容会落到 spill
- **验收**：spill 存什么？恢复依据是什么？
- **产出**：`notes\3-运行内核\3.13-spill与恢复.md`

### 3.14 子代理与委派
- **读**：`packages\subagent\`
- **做**：画出主 agent 与子 agent 的交互与上下文隔离
- **验收**：子代理为什么需要独立上下文？结果如何回传？
- **产出**：`notes\3-运行内核\3.14-子代理.md`

### 3.15 目标、计划与待办
- **读**：`packages\goal\`；`packages\plan\`；`packages\todo\`
- **做**：说明"长期目标如何驱动续跑"
- **验收**：目标与计划的边界？待办如何影响下一步输入？
- **产出**：`notes\3-运行内核\3.15-目标与计划.md`

### 3.16 调度与可观测性
- **读**：`packages\schedule\`；`packages\session\session-stats\`、`session-telemetry\`（按需）
- **做**：说明定时/延迟任务如何触发 agent，统计与遥测记录了什么
- **验收**：调度任务的触发路径？可观测性数据从哪来、给谁用？
- **产出**：`notes\3-运行内核\3.16-调度与可观测.md`

---

## 第四编 · 思想、取舍与前沿

### 4.1 为什么"无特权内核"
- **读**：`docs\architecture.zh.md`；`packages\bundle\sdk-minimal\`
- **做**：列出至少 3 个"可以被替换掉"的实现，并说明替换的代价
- **验收**：无特权内核的收益与代价各是什么？
- **产出**：`notes\4-思想与前沿\4.1-无特权内核.md`

### 4.2 为什么事件溯源 + 只追加
- **读**：`.agents\notes\` 中相关篇目
- **做**：写出该选择的收益与代价各 3 条
- **验收**：如果允许改写历史，会失去什么？
- **产出**：`notes\4-思想与前沿\4.2-事件溯源取舍.md`

### 4.3 为什么"模型可见即已记录"
- **读**：`packages\core\agent-loop\`（请求相关）与相关设计笔记
- **做**：用一个具体故障场景说明这条不变量的价值
- **验收**：它保护了什么？付出的开销在哪？
- **产出**：`notes\4-思想与前沿\4.3-可见即已记录.md`

### 4.4 能力接缝与可替换性
- **读**：`docs\capability-seams.zh.md`
- **做**：挑一条接缝（如文件系统或沙箱），说明换提供者的波及面
- **验收**：三角色定义？替换会影响哪些层？
- **产出**：`notes\4-思想与前沿\4.4-能力接缝.md`

### 4.5 契约治理与兼容性设计
- **读**：`docs\upgrade-guide\`（6 份）；`packages\boot\app-boot\src\compatibility-preflight.ts`
- **做**：总结一次破坏性变更的完整交付物清单
- **验收**：为什么宁可拒绝加载也不冒险？豁免为什么必须显式？
- **产出**：`notes\4-思想与前沿\4.5-契约治理.md`

### 4.6 横向对比
- **读**：自行检索对照体系（Spring / LangChain / VS Code 插件 / Claude Code）
- **做**：做一张"同一个问题在不同体系的解法"对比表
- **验收**：DSH 的选择在哪些点上有别于它们、为什么？
- **产出**：`notes\4-思想与前沿\4.6-横向对比.md`

### 4.7 前沿技术观察
- **读**：`packages\mcp\`、`packages\acp\`、`packages\skill\`、`packages\workflow\`、`packages\ptc-runtime\`
- **做**：逐个说清解决什么问题、在 DSH 里的位置与边界
- **验收**：MCP / ACP / skills / workflow / PTC 各是什么、彼此边界在哪？
- **产出**：`notes\4-思想与前沿\4.7-前沿技术观察.md`

---

## 附录 A · 语言知识索引（缺什么补什么）

用法：读到某一步卡在语言机制上，来这张表定位，再去 DSH 对应文件里看它**实际怎么用**。

| 语言 / 运行时知识 | 首次需要于 | DSH 中的落点 |
|---|---|---|
| `package.json` 的 `exports` 子路径与条件导出 | 2.9 | 各插件与本仓库各包的 `package.json` |
| 配置表达式的运行时求值 | 1.3 | `--dump-config` 输出中的 `disabled` 字段 |
| `Proxy` / `Reflect` | 2.1 | `vendor\cordis\src\context.ts` |
| `Symbol`（服务标识、迭代协议） | 2.2 | `vendor\cordis\src\service.ts` |
| 闭包与依赖逆序释放 | 2.3 | `vendor\cordis\src\fiber.ts` |
| 配置 Schema 校验 | 2.6 | 使用 schemastery 的各插件 |
| `AsyncIterable` / 异步生成器 | 3.8 | `packages\core\agent-loop\src\assistant-stream.ts` |
| Node Stream 与背压 | 3.8 | `packages\llm\` 下的流式实现 |
| JSONL 与 zstd 分帧 | 3.1 | `packages\session\session-persistence-jsonl\` |
| 泛型与类型推导在 DSL 中的用法 | 3.9 | `packages\core\tools\src\index.ts` |
| `.d.ts` 与类型导出边界 | 3.9 | 各包的 `lib\types\` |
| 子进程与 Worker | 3.11、3.16 | `packages\subprocess\` |
| 测试框架与快照测试 | 4.1 | `vitest.config.ts`；`packages\core\agent-loop\tests\mock-adapter.ts` |

---

# 三、学习成果

## 3.1 编级验收

| 编 | 通过标准 |
|---|---|
| 一 结构与装配 | 能说清三层叠加与覆盖规则；能自己裁一个 profile 并用 dump 验证 |
| 二 插件内核 | 能解释依赖注入、可逆副作用、事件分发、作用域；能写出并装卸一个插件 |
| 三 运行内核 | 能完整讲出一次请求的链路与状态迁移；能解释压缩、恢复、并发调度 |
| 四 思想与前沿 | 能论证每处设计的取舍；能做横向对比；能说清 MCP/ACP/skills/workflow 的定位 |

## 3.2 产出物清单

| 类型 | 内容 | 位置 |
|---|---|---|
| 笔记 | 38 篇（每步一篇） | `.learning\notes\` |
| 实验脚本 | 依赖注入、effect 顺序、事件分发等 | `.learning\experiments\` |
| 插件 | 一个可装卸的 DSH 插件（含 README） | `.learning\plugins\` |
| 图 | 分层依赖图、状态迁移图、请求链路图 | 并入对应笔记 |

## 3.3 毕业检查清单

- [ ] 我能解释"没有特权内核"的确切含义与代价
- [ ] 我能说清 profile / bundle / patch 的组合与覆盖规则
- [ ] 我能解释依赖注入、可逆副作用、五种事件分发的差别
- [ ] 我能解释作用域隔离与"按会话定制能力"
- [ ] 我能写出 DSH 插件并完成打包 → 安装 → 验证 → 卸载
- [ ] 我能完整讲出 turn / step 状态机与 inbox 的作用
- [ ] 我能解释"先落日志再发出"与请求冻结
- [ ] 我能解释一次工具调用经过的所有把关环节与并发调度
- [ ] 我能解释压缩为什么是遮蔽、恢复依靠什么
- [ ] 我能论证事件溯源、能力接缝、契约治理三处设计的取舍
- [ ] 我能说清 MCP / ACP / skills / workflow / PTC 各自解决什么问题

## 3.4 成果去向：怎么用于求职

| 要证明的能力 | 用什么证明 |
|---|---|
| 理解 AI Agent 工程 | 第三编笔记：状态机、请求冻结、工具流水线、并发调度、压缩与恢复 |
| 能设计可扩展架构 | 第二编笔记 + 自己的插件 + 与其他插件体系的机制对比 |
| 懂 LLM 应用全栈 | 第三编 3.7/3.8 的模型层 + 第四编的能力接缝与多形态装配 |
| 会动手验证而非空谈 | 实验脚本 + 自写测试，不是只有笔记 |
| 工程素养与前沿视野 | 第四编的契约治理与前沿技术观察 |

## 3.5 进度记录

| 步骤 | 状态 | 日期 | 产出物 |
|---|---|---|---|
| 1.1–1.6 结构与装配 | ☐ | — | notes\1-结构与装配\1.x |
| 2.1–2.9 插件内核 | ☐ | — | experiments\ + notes\2-插件内核\2.x |
| 3.1–3.16 运行内核 | ☐ | — | notes\3-运行内核\3.x |
| 4.1–4.7 思想与前沿 | ☐ | — | notes\4-思想与前沿\4.x |

---

*本文档为活文档：每完成一编回来修订下一编的步骤与深度。*
