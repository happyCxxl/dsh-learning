# DSH 学习路线

**面向**：有开发经验、但对 DSH 一无所知的人。
**目标**：能读懂 DSH 的结构与内核，能自己写插件，能把这套系统的设计讲给别人听。
**约定**：全程**不修改 DSH 源码**；所有产物（笔记、图、脚本、插件）放在本目录 `.learning\` 下。

---

# 一、学习路线

## 1.1 学什么（范围与边界）

**三条主线**

| 主线 | 能力目标 | 对应阶段 |
|---|---|---|
| A 结构与装配 | 能读懂"一个 DSH 产品是怎么被叠出来的"，能自己裁一个 profile | 阶段 1 |
| B 插件机制 | 能解释插件内核的三件套（生命周期 / 依赖注入 / 事件），能自己写并装卸插件 | 阶段 2 |
| C 运行内核 | 能完整讲出一次请求的链路，能解释会话事件溯源与工具流水线 | 阶段 3 |
| （支撑）工程与生态 | 能看懂这套系统的测试、契约治理与多形态装配方式 | 阶段 4 |

**明确不学**（避免摊大，需要时再单独开线）
- 原生模块实现层（C++ / Rust）
- Electron 桌面打包、自动更新、安装器
- Python 运行时、Office 文档引擎
- 性能基准与压力测试

## 1.2 怎么学（方法与颗粒度）

**四条方法约定**

1. **问题驱动，不以文件为单位。** 完成标准是"能回答验收问题"，不是"读完了某个文件"。
2. **证据驱动。** 每个结论都要能指出源码位置，或者跑出可观察结果——不接受"读着通顺"。
3. **顺序不可跳。** 结构 → 机制 → 内核 → 工程。前一阶段给后一阶段提供坐标系；跳过去会看得懂单词、读不懂句子。
4. **不动源码。** 动手 = 画图、写笔记、写测试、写插件。

**颗粒度（三级，每个步骤固定四件套）**

| 级别 | 数量 | 粒度 |
|---|---|---|
| L1 阶段 | 4 | 一个能力目标 |
| L2 步骤 | 16 | 每个 0.5–1 天，可独立验收 |
| L3 每步骤内容 | 固定四项 | **读什么**（≤4 个文件）· **做什么**（1 个动作）· **验收问题**（2–3 条）· **产出物**（1 个） |

判定"这一步做完"的唯一标准：**验收问题能不看资料答出来**，且产出物已落盘。

## 1.3 学习步骤总览

| 阶段 | 步骤 | 主题 | 预计 |
|---|---|---|---|
| **1 结构与装配** | 1.1 | 名词地图：先建立坐标系 | 0.5 天 |
| | 1.2 | 装配机制：profile / bundle / patch | 1 天 |
| | 1.3 | 包分层与依赖方向 | 0.5 天 |
| | 1.4 | 产品形态差异：同一内核的多种装配 | 0.5 天 |
| **2 插件机制** | 2.1 | cordis 三件套：生命周期 / 依赖注入 / 事件 | 1 天 |
| | 2.2 | 组合与热重载：patch 层与重载边界 | 0.5 天 |
| | 2.3 | 扩展点地图：能往哪些地方插 | 0.5 天 |
| | 2.4 | 写出第一个插件（含兼容性治理） | 1–2 天 |
| **3 运行内核** | 3.1 | 会话与事件溯源 | 1 天 |
| | 3.2 | Agent 循环与请求构造 | 1 天 |
| | 3.3 | 工具流水线：注册、把关、并发 | 1 天 |
| | 3.4 | 上下文压缩与恢复 | 0.5 天 |
| **4 工程与生态** | 4.1 | 测试与验收体系 | 0.5 天 |
| | 4.2 | 契约治理与设计决策 | 0.5 天 |
| | 4.3 | 客户端 / 宿主边界与 API 层 | 1 天 |
| | 4.4 | 多形态装配与 SDK | 0.5 天 |

合计约 **11–13 个工作日**。

---

# 二、学习内容

## 2.1 素材索引（随用随查，不要通读）

| 素材 | 路径 | 用途 |
|---|---|---|
| 术语表 | `docs\glossary.zh.md` | 查名词 |
| 总架构 | `docs\architecture.zh.md` | 阶段 1；第 145 行起是「目标 → 机制」对照表 |
| 模块图 | `docs\module-graph.zh.md` | **搜索用**：某功能在哪个包 |
| 工具目录 | `docs\tool-catalog.zh.md` | 系统提供了哪些工具 |
| 配置目录 | `docs\config-catalog.zh.md` | 某配置项什么意思 |
| 持久化目录 | `docs\persistence-catalog.zh.md` | 会话里存了什么 |
| 事件全景 | `docs\event-producer-consumer.zh.md` | 阶段 3 |
| 工具流水线 | `docs\tool-execution-pipeline.zh.md` | 阶段 3 |
| 生命周期 | `docs\agent-lifecycle.zh.md` | 阶段 3 |
| cordis 教程 | `docs\cordis-tutorial\`（8 篇，约 36KB） | 阶段 2 主线，按序读 |
| 扩展点手册 | `docs\cookbook\extension-cookbook.zh.md` | 阶段 2.3 |
| 能力接缝 | `docs\capability-seams.zh.md` | 阶段 2.3、4.4 |
| 测试说明 | `docs\testing.zh.md` | 阶段 4.1 |
| 升级指南 | `docs\upgrade-guide\`（6 份中文） | 阶段 4.2 精读 |
| 设计笔记 | `.agents\notes\` | 想知道"为什么这么设计"时按主题搜 |
| 子系统文档 | `docs\subsystems\`（64 篇中文） | 按需查 |
| 真实插件样本 | `~\.dsh\profiles\desktop\node_modules\deepseek-idesign` | 阶段 2.4 对照 |
| 第三方插件规范 | `D:\application\dsh\dsh-plugins\docs\PLUGIN_SPEC.md` | 阶段 2.4 |

**运行环境的两个入口**
- 本机产品 CLI：`D:\application\dsh-desktop\resources\runtime\cli\bin\dsh.cmd`
- 仓库源码 CLI：在仓库根跑 `pnpm dsh --profile <name> ...`

---

## 阶段 1 · 结构与装配

**阶段目标**：能读懂"一个 DSH 产品是怎么被叠出来的"，并自己裁一个 profile。

### 步骤 1.1 名词地图
- **读**：`docs\glossary.zh.md`；`docs\architecture.zh.md` 开头部分；`packages\core\README.zh.md`
- **做**：把架构文档里出现的名词抄成一张自己的对照表（名词 / 一句话解释 / 还不知道的先标问号）
- **验收**：① `profile` / `bundle` / `patch` 三者各自是什么？② `ctx`、`service`、`plugin` 是什么关系？
- **产出**：`notes\01-名词对照表.md`（允许留"待解答"）

### 步骤 1.2 装配机制
- **读**：`packages\boot\app-boot\src\profile.ts`（文件开头 21 行注释就写了组合顺序）
- **做**：dump 出你本机 web profile 的完整插件树，找出所有层边界：
  ```powershell
  D:\application\dsh-desktop\resources\runtime\cli\bin\dsh.cmd --profile web --dump-config > $env:TEMP\web-tree.yml
  ```
  对比 `--dump-config` 与 `--dump-default-config` 的差异
- **验收**：① 三层叠加的准确顺序？② 同一个 `id` 在多层出现，谁生效？③ `disabled` 为什么允许写成表达式？
- **产出**：`notes\02-装配机制.md`（贴出层边界片段 + 你的结论）

### 步骤 1.3 包分层与依赖方向
- **读**：`packages\` 目录结构（各 tier 各管什么）；`docs\module-graph.zh.md`（搜索用）；`pnpm-workspace.yaml`
- **做**：挑一条链路，画出它的依赖方向，例如"Web 请求 → 后端 API → 会话 → 存储"
- **验收**：① `packages/core` 与 `packages/bundle` 的职责差别？② 为什么 `apps/` 在 `packages/` 之上？
- **产出**：`notes\03-分层与依赖方向.md`（一张依赖箭头图）

### 步骤 1.4 产品形态差异
- **读**：`packages\bundle\` 下各 bundle 的 `cordis.patch.yml`；`apps\` 目录
- **做**：对比 `base` / `web-app` / `headless` / `sdk-minimal` 各自装了什么、少了什么
- **验收**：① 从 `base` 到 Web 产品，中间加了什么？② `sdk-minimal` 为什么刻意不带 `base`？
- **产出**：`notes\04-产品形态对比.md`（一张对比表）

---

## 阶段 2 · 插件机制

**阶段目标**：能解释插件内核的三件套，并写出一个能装能卸的插件。

### 步骤 2.1 cordis 三件套
- **读**：`docs\cordis-primer.zh.md`；`docs\cordis-tutorial\` 的 `01`–`04`；`vendor\cordis\src\` 的 `service.ts` → `registry.ts` → `context.ts` → `fiber.ts`
- **做**：在 `.learning\experiments\` 写一个约 20 行的最小 cordis 插件（函数 + `apply(ctx)` + 一个 `ctx.effect`），用 `vendor\cordis\lib\index.js` 跑通
- **验收**：① 插件系统要满足的四个需求是什么？② `inject` 解决什么问题？③ `ctx.effect` 返回的"撤销函数"为什么必须存在、按什么顺序执行？
- **产出**：`experiments\minimal-cordis-plugin.mjs` + `notes\05-cordis三件套.md`

### 步骤 2.2 组合与热重载
- **读**：`docs\cordis-tutorial\` 的 `05`–`07`；`packages\boot\hmr\README.zh.md`
- **做**：改一次你自己 profile 的 `cordis.patch.yml`，观察哪些改动能热生效、哪些必须重启
- **验收**：① `cordis.patch.yml` 在组合顺序里处于哪一层？② 为什么 bundle 层只在启动时读取？
- **产出**：`notes\06-热重载边界.md`

### 步骤 2.3 扩展点地图
- **读**：`docs\cookbook\extension-cookbook.zh.md`；`docs\capability-seams.zh.md`
- **做**：列出 DSH 暴露的扩展点，每个标注注册入口（源码路径）与最小示例位置
- **验收**：① 至少说出 5 类扩展点及其注册入口；② "能力接缝"的定义 / 提供者 / 使用者三角色分别是什么？
- **产出**：`notes\07-扩展点地图.md`

### 步骤 2.4 写出第一个插件
- **读**：`docs\cookbook\adding-a-tool.zh.md`；一个真实插件的 `package.json` 与入口（`deepseek-idesign`）；`PLUGIN_SPEC.md`
- **做**：在 `.learning\plugins\` 写一个工具插件 → 打包 → 安装到你自己的 profile → 验证能被调用 → 卸载
- **验收**：① `dsh.bundle` 与 `dsh.client` 的区别？② 为什么装完要重启？③ 怎么证明卸载后没有残留？④ 兼容性预检凭什么拒绝一个插件、豁免为什么必须显式？
- **产出**：`plugins\<你的插件>\`（含 README：用途 / 权限 / 副作用 / 兼容性）

---

## 阶段 3 · 运行内核

**阶段目标**：能完整讲出一次请求的链路，并解释为什么这么设计。

### 步骤 3.1 会话与事件溯源
- **读**：`packages\core\session\README.zh.md` 与 `src\index.ts`；`docs\persistence-catalog.zh.md`（按需查）
- **做**：把"一次会话里发生了什么"用事件序列写出来（事件名 + 含义）
- **验收**：① 为什么日志只能追加、不能改写？② 消息是怎么从事件"推导"出来的？③ 压缩为什么是遮蔽而不是删除？
- **产出**：`notes\08-会话事件溯源.md`

### 步骤 3.2 Agent 循环与请求构造
- **读**：`packages\core\agent\README.zh.md`；`packages\core\agent-loop\README.zh.md` 与 `src\agent.ts`、`src\inbox.ts`；`packages\core\system-prompt\README.zh.md`
- **做**：把"一个 turn 如何拆成多个 step"讲清楚，标出每一步状态迁移
- **验收**：① turn 与 step 的关系？② 请求为什么要"先落日志再发出"？③ 系统提示词是怎么被拼装的、不同会话为什么可以不同？
- **产出**：`notes\09-循环与请求.md`

### 步骤 3.3 工具流水线
- **读**：`docs\tool-execution-pipeline.zh.md`；`packages\core\tools\README.zh.md` 与 `src\index.ts`；`packages\core\agent-loop\src\tool-calls.ts`
- **做**：跑一次会触发审批的工具调用，记录"把关点"分别在什么条件下介入
- **验收**：① 一次工具调用经过哪些阶段？② 守卫与审批各防什么？③ 并发调度里"独占屏障"解决什么问题？
- **产出**：`notes\10-工具流水线.md`

### 步骤 3.4 上下文压缩与恢复
- **读**：`packages\compaction\`（各包 README）；`docs\subsystems\` 中相关篇目
- **做**：解释"长任务为什么不会爆上下文"，并说明压缩后原始信息还在不在
- **验收**：① 触发压缩的条件是什么？② 压缩后原始事件是否还在？③ 会话如何从中断中恢复？
- **产出**：`notes\11-压缩与恢复.md`

---

## 阶段 4 · 工程与生态

**阶段目标**：看懂这套系统是怎么被开发、约束与扩展的。

### 步骤 4.1 测试与验收体系
- **读**：`docs\testing.zh.md`；`vitest.config.ts`；`packages\core\agent-loop\tests\mock-adapter.ts`
- **做**：跑通一个内核测试，再自己写一个小测试断言事件顺序（放在仓库既有测试目录下，属未跟踪新增文件）
- **验收**：① 没有真实 API 时怎么测 agent？② 快照测试在测什么？
- **产出**：`notes\12-测试体系.md` + 一个自写测试

### 步骤 4.2 契约治理与设计决策
- **读**：`docs\upgrade-guide\`（6 份中文，逐份读完）；`.agents\notes\implemented\architecture\` 里挑 5 篇
- **做**：选一次真实的"机制存废"，把"为什么加、为什么删"整理成一段论证
- **验收**：① 一次破坏性变更要交付哪些东西？② 为什么"删掉一个机制"也需要成体系的说明？
- **产出**：`notes\13-契约治理.md`

### 步骤 4.3 客户端 / 宿主边界与 API 层
- **读**：`packages\client\`（59 个包）；`packages\host\webserver\src\index.ts`；`packages\api\` + `docs\api-gateway.zh.md`
- **做**：说清"浏览器里跑什么、Node 里跑什么、两者怎么通信"
- **验收**：① 客户端插件与宿主插件的边界在哪、为什么划这条线？② 后端接口层在整体里处于什么位置？
- **产出**：`notes\14-客户端与宿主边界.md`

### 步骤 4.4 多形态装配与 SDK
- **读**：`packages\sdk\`；`packages\bundle\sdk-app\`；`docs\cookbook\adding-a-remote-api.zh.md`
- **做**：用仓库自省脚本看待全景（`pnpm run verify-module-graph`、`verify-tool-catalog`、`verify-config-catalog`），对照生成的 `docs\module-graph.zh.md` 等
- **验收**：① 同一内核如何变成 CLI / Web / 桌面 / SDK？② "从代码生成文档"这种自省手段解决了什么问题？
- **产出**：`notes\15-多形态与SDK.md`

---

# 三、学习成果

## 3.1 阶段验收

| 阶段 | 通过标准（能不看资料讲清） |
|---|---|
| 1 结构与装配 | 三层叠加顺序；同 `id` 冲突规则；能自己裁一个 profile 并 dump 验证 |
| 2 插件机制 | 插件四需求 ↔ 三机制的对应；`ctx.effect` 可逆性；能装能卸一个自己写的插件 |
| 3 运行内核 | 一次请求的完整事件序列；"先落日志再发出"的理由；工具把关点；压缩语义 |
| 4 工程与生态 | 测试怎么脱离真实模型；破坏性变更的交付物；客户端/宿主边界；多形态装配方式 |

## 3.2 产出物清单

| 类型 | 内容 | 位置 |
|---|---|---|
| 笔记 | 15 篇（`01`–`15`，每步一篇） | `.learning\notes\` |
| 实验 | 最小 cordis 插件 | `.learning\experiments\` |
| 插件 | 一个可装卸的 DSH 插件（含 README） | `.learning\plugins\` |
| 测试 | 一个自写的事件顺序断言测试 | 仓库测试目录（未跟踪文件） |
| 图 | 分层依赖图、请求链路图 | 并入对应笔记 |

## 3.3 毕业检查清单

- [ ] 我能解释"没有特权内核"的确切含义，并举例说明什么可以被替换
- [ ] 我能说清 profile / bundle / patch 的组合顺序与覆盖规则
- [ ] 我能解释插件系统的四个需求与对应机制
- [ ] 我能写出最小 cordis 插件，并解释每个注册为何可回滚
- [ ] 我能写出 DSH 插件并完成 打包 → 安装 → 验证 → 卸载
- [ ] 我能解释事件溯源：为什么只追加、为什么压缩是遮蔽
- [ ] 我能说清一次工具调用经过的所有把关环节
- [ ] 我能解释同一内核如何装配出多种产品形态
- [ ] 我有 15 篇笔记 + 1 个插件 + 1 个脚本 + 1 个测试作为可展示产出

## 3.4 成果去向：怎么用于求职

| 要证明的能力 | 用什么证明 |
|---|---|
| 理解 AI Agent 工程 | 阶段 3 的四篇笔记：事件溯源、循环与请求、工具流水线、压缩与恢复 |
| 能设计可扩展架构 | 插件四需求 ↔ 三机制的分析 + 自己的插件 + 与其他插件体系（VS Code / 浏览器扩展）的机制对比 |
| 懂 LLM 应用全栈 | 阶段 4.3 / 4.4：客户端与宿主边界、多形态装配 |
| 会动手验证而非空谈 | 自写测试 + 可运行脚本，不是只有读书笔记 |
| 工程素养 | 步骤 4.1 / 4.2：测试体系与契约治理的理解 |

## 3.5 进度记录

| 步骤 | 状态 | 完成日期 | 产出物 |
|---|---|---|---|
| 1.1 名词地图 | ☐ | — | notes\01 |
| 1.2 装配机制 | ☐ | — | notes\02 |
| 1.3 包分层与依赖方向 | ☐ | — | notes\03 |
| 1.4 产品形态差异 | ☐ | — | notes\04 |
| 2.1 cordis 三件套 | ☐ | — | experiments + notes\05 |
| 2.2 组合与热重载 | ☐ | — | notes\06 |
| 2.3 扩展点地图 | ☐ | — | notes\07 |
| 2.4 写出第一个插件 | ☐ | — | plugins\ + notes |
| 3.1 会话与事件溯源 | ☐ | — | notes\08 |
| 3.2 Agent 循环与请求 | ☐ | — | notes\09 |
| 3.3 工具流水线 | ☐ | — | notes\10 |
| 3.4 压缩与恢复 | ☐ | — | notes\11 |
| 4.1 测试与验收体系 | ☐ | — | notes\12 + 测试 |
| 4.2 契约治理 | ☐ | — | notes\13 |
| 4.3 客户端/宿主边界 | ☐ | — | notes\14 |
| 4.4 多形态与 SDK | ☐ | — | notes\15 |
