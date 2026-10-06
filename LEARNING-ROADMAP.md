# DSH 从零学习路线

**适用对象**：完全没接触过 DSH，也没接触过 agent harness、插件框架、依赖注入的人。
**最终目标**：从"不知道 DSH 是什么"，到"能读懂它的内核、能自己写插件、能把这段经历讲给别人听"。
**学习方式约定**：全程**不修改 DSH 源码**；所有练习代码、脚本、笔记都放在本目录（`.learning\`）下。

---

# 一、学习路线

## 1.1 前提与不假设

**假设你会**：写 TypeScript / JavaScript；用命令行；看懂 JSON。
**不假设你会**：agent harness、插件框架、依赖注入、事件溯源、DSH 的任何概念。

所以这条路线的第一步**不是读代码**，而是先看到现象、再解释现象、最后才看实现。

## 1.2 三篇七阶段

| 篇 | 阶段 | 主题 | 你会得到什么 | 预计 |
|---|---|---|---|---|
| 基础 | 1 | 先跑起来 | 亲眼看到"AI 调工具"是什么样 | 0.5 天 |
| 基础 | 2 | 概念地基 | 搞懂 harness 到底在解决什么问题 | 1–2 天 |
| 基础 | 3 | 一次请求的完整链路 | 能画出 DSH 的运行全景图 | 1–2 天 |
| 核心 | 4 | 插件思想与 cordis | 理解"能装能卸能替换"是怎么做到的 | 2–3 天 |
| 核心 | 5 | 写你的第一个插件 | **一个能跑的 DSH 插件** | 3–5 天 |
| 进阶 | 6 | 内核精读 | 读懂 agent 循环与事件溯源 | 4–6 天 |
| 进阶 | 7 | 产品全景 | 看懂同一内核如何变成多种产品 | 3–5 天 |

## 1.3 四个原则

1. **先会用，再懂原理。** 第一件事是跑起来看现象，不是打开架构文档。看不懂的术语先记下，第二轮再回来。
2. **允许"暂时不懂"。** 每阶段维护一份「问题清单」，读不懂就往下走——很多问题会在后面的阶段自动消解。
3. **每个阶段必须有能讲给别人听的东西。** 讲不出来 = 没学会。
4. **一切结论自己验证一次。** "读着通顺"不算懂；能跑个脚本、翻出日志、看到现象，才算。

## 1.4 卡住时先查哪里（速查表）

| 你的困境 | 去哪查 |
|---|---|
| 术语看不懂 | `docs\glossary.zh.md` |
| 某个功能在哪个包里 | `docs\module-graph.zh.md`（很大，用搜索，别通读） |
| 某个配置项什么意思 | `docs\config-catalog.zh.md` |
| 系统一共提供了哪些工具 | `docs\tool-catalog.zh.md` |
| **为什么这么设计** | `.agents\notes\`（中文设计笔记，按主题搜） |
| 某功能某个版本改了什么 | `docs\upgrade-guide\` |

---

# 二、学习内容

## 2.1 素材总览

| 素材 | 路径 | 怎么用 |
|---|---|---|
| 术语表 | `docs\glossary.zh.md` | 当字典，随时查 |
| 总架构 | `docs\architecture.zh.md` | 阶段 2 读开头，阶段 3 精读并回查 |
| 生命周期与流水线 | `docs\agent-lifecycle.zh.md`、`docs\tool-execution-pipeline.zh.md` | 阶段 3 精读 |
| 事件清单 | `docs\event-producer-consumer.zh.md` | 阶段 3、6 精读 |
| 插件教程 | `docs\cordis-tutorial\`（8 篇，共约 36KB） | **阶段 4 的主线，从头读** |
| 插件框架源码 | `vendor\cordis\src\` | 阶段 4 精读 |
| 扩展点手册 | `docs\cookbook\extension-cookbook.zh.md` | 阶段 5 的路线图 |
| 设计笔记 | `.agents\notes\`（1197 篇中文） | 任何时候想知道"为什么"就搜 |
| 升级指南 | `docs\upgrade-guide\`（6 份中文） | 想理解"契约/兼容性"时读 |
| 内核源码 | `packages\core\` | 阶段 6 精读 |
| 一个真实插件样本 | `~\.dsh\profiles\desktop\node_modules\deepseek-idesign` | 阶段 5 对照读（第三方插件，已装在你机器上） |
| 安装目录内打包产物 | `resources\app.asar` 内 `dsh\` | 每个包自带 `README.zh.md`，可当补充材料 |

## 阶段 1 · 先跑起来（0.5 天）

**学什么**：DSH 是什么、能做什么。这个阶段**不读任何源码**。

**动手**
1. 打开桌面版，给它一个需要"动手"的任务，例如：
   > 列出 `D:\application\dsh` 下所有文件夹，并统计每个文件夹里的文件数
   注意观察三件事：它什么时候停下来**问你批准**、它**调用了什么工具**、侧栏/任务面板显示了什么。
2. 再给它一个纯问答任务（例如"解释一下什么是事件溯源"），对比两者的区别——**这就是"用不用工具"的分水岭**。

**动手（第一个脚本）**：把它刚产生的对话日志解出来，看它到底记了什么。
DSH 把会话存成追加式日志：`C:\Users\cl152\.dsh\sessions\<工作区名>\<会话id>\session.v4.jsonl.zstd`。
注意它是 **zstd 压缩、而且不是一个整块**——每追加一批数据就压成**一个独立的帧**，所以要用下面的方式逐帧解（这不是我猜的，是实测出来的：一个 456KB 的文件里有 331 个帧）：

```js
// dump-session-events.mjs —— 统计你自己某个会话里的事件类型分布
import { readdirSync, readFileSync, statSync } from 'node:fs'
import { join } from 'node:path'
import { zstdDecompressSync } from 'node:zlib'

const root = join(process.env.USERPROFILE, '.dsh', 'sessions')
const walk = (d) => readdirSync(d, { withFileTypes: true }).flatMap((e) => {
  const p = join(d, e.name)
  return e.isDirectory() ? walk(p) : (e.name.endsWith('.zstd') ? [p] : [])
})

const file = process.argv[2] ?? walk(root).sort((a, b) => statSync(b).size - statSync(a).size)[0]
const buf = readFileSync(file)

// zstd 帧魔数 28 B5 2F FD：按它切分，逐帧解压
const MAGIC = Buffer.from([0x28, 0xb5, 0x2f, 0xfd])
const offsets = []
for (let i = 0; (i = buf.indexOf(MAGIC, i)) >= 0; i += 4) offsets.push(i)

let text = ''
for (let n = 0; n < offsets.length; n++) {
  const end = n + 1 < offsets.length ? offsets[n + 1] : buf.length
  text += zstdDecompressSync(buf.subarray(offsets[n], end)).toString('utf8')
}

const lines = text.split('\n').filter(Boolean)
const types = {}
for (const line of lines) {
  const t = JSON.parse(line).type ?? '(none)'
  types[t] = (types[t] ?? 0) + 1
}
console.log('文件:', file)
console.log('帧数:', offsets.length, '事件数:', lines.length)
console.table(Object.entries(types).sort((a, b) => b[1] - a[1]).map(([type, count]) => ({ type, count })))
```

运行（用 DSH 自带的 Node，不需要额外装东西——`zstdDecompressSync` 是 Node 内置能力）：
```powershell
& "$env:USERPROFILE\.dsh\dsh-runtimes\dsh-primary-runtime\dependencies\node\bin\node.exe" dump-session-events.mjs
```

**你会看到的现象**（这是我在你机器上一个真实会话上跑出来的，你可以对照）：
- `tool/call` 233 次、`tool/result` 233 次 —— 工具调用与结果**成对出现**
- `step/start` 51 次、`step/end` 51 次 —— 一个"回合"里跑了 **51 个步骤**
- `assistant/message` 51 次 —— 每步一次模型回复
- `turn/start` 只有 1 次、`turn/end` 只有 1 次 —— 把整个任务包在中间
- `system/message`、`request/header`、`request/context` 各 1 次 —— 请求发出前先落好这些
- `approval/policy`、`sandbox/mode`、`permission/preset` 各 1 次 —— 环境与权限状态

**先别急着懂这些名字**。你现在只需要记住两个直觉：**（1）一切都是"事件"，按发生顺序记下来；（2）工具调用是成对记账的。**

**检查点**
- 用自己的话说：DSH 和直接用 ChatGPT 网页版，区别是什么？
- 你刚才那个任务，它一共调了几次工具？为什么这么多？

**产出物**：`notes\01-第一次使用与日志观察.md`（记下你看到的现象和 3 个疑问）

## 阶段 2 · 概念地基（1–2 天）

**学什么**：不碰 DSH 特有的名词，先把"为什么需要 harness"这件事想通。按下面五步走，每一步都是一个因果链：

1. **模型只会"文本进、文本出"。** 它没有记忆（每次都是你把它需要的上下文重新喂给它）、不能上网、不能读你的文件。
2. **于是有了"工具调用"。** 模型不直接查天气，而是输出一个结构化请求"请帮我调用 get_weather(城市=杭州)"；**由程序去执行**，再把结果作为新消息喂回去。
3. **于是有了"循环"。** 请求 → 模型说要调工具 → 程序执行 → 结果回填 → 再请求……直到模型说"我答完了"。这个循环就是 **agent loop**。
4. **于是有了"上下文窗口"问题。** 对话越长，喂回去的内容越多、越贵、越容易超出上限 → 所以要**压缩/摘要**。DSH 的做法是"遮蔽"而不是删除（阶段 6 会讲为什么）。
5. **于是有了 harness。** 把上面这些串起来，并补上真实产品必需的东西：权限与审批、日志与恢复、沙箱、可替换的模型、可扩展的插件机制。**harness = 让模型能安全地做成事的那层软件。**

**读什么**
- `docs\glossary.zh.md`：混个脸熟，不求记住
- `docs\architecture.zh.md`：**只读开头几段**，看它如何描述自己
- `docs\agent-lifecycle.zh.md`：读一遍，允许一半看不懂

**检查点**
- 为什么模型必须靠"工具调用"才能读文件？
- harness 相比"直接调模型 API"，多干了哪几件事？（至少说出 4 件）
- 上下文窗口满了会发生什么？为什么不能简单删掉旧消息？

**产出物**：一张手绘的「最小 agent 循环」图——不超过 6 个方框，**用你自己的词**，不许用 DSH 的术语。

## 阶段 3 · 一次请求的完整链路（1–2 天）

**学什么**：把阶段 2 的通用循环，换成 DSH 的真实实现。

**读什么**（按顺序）
1. `docs\tool-execution-pipeline.zh.md` —— 一次工具调用从发起到落地，中间经过哪些把关
2. `docs\event-producer-consumer.zh.md` —— 事件的生产者与消费者全景
3. `packages\core\README.zh.md` —— core 这一层由哪几个包组成、各自管什么
4. 回读 `docs\architecture.zh.md`，这次读第 145 行起的「目标 → 机制」对照表：**"我想要什么"对应"它用哪个机制实现"**

**动手**
- 回到阶段 1 的脚本，这次用 `--type` 过滤（改一行：只打印某个类型的事件），看看 `turn/start` → `step/start` → `assistant/message` → `tool/call` → `tool/result` → `step/end` → `turn/end` 的真实先后顺序
- 跑一次带审批的动作，观察 `approval/policy` 这类事件什么时候出现

**检查点**
- 一次请求从你用键盘敲下去，到看见流式回答，中间依次发生了什么？
- 工具执行的"把关"环节有哪几处？分别在防什么？
- `turn` 和 `step` 是什么关系？（提示：阶段 1 你看到 1 个 turn 里有 51 个 step）

**产出物**：把阶段 2 的图升级成「DSH 版全链路图」，标上真实事件名

## 阶段 4 · 插件思想与 cordis（2–3 天）

**学什么**：先想通"为什么需要插件系统"，再学 DSH 用的那套（cordis）。

**第一步：想清楚通用问题**（不看代码，先想）
一个软件如果想被第三方扩展，会依次撞上四个需求：
1. **能装能卸**：卸载后不能留残渣（注册的路由、挂的事件、占的资源都要撤干净）
2. **能替换**：同一个接口可以换成不同实现（本地文件 vs 远程存储）
3. **有先后**：A 依赖 B，就得保证 B 先就绪
4. **能互相通知**：一个模块做完了，其他模块想插一脚（拦截、改写、追加）

对应到技术手段就是：**生命周期（装/卸）→ 依赖注入（先后）→ 事件（通知）**。cordis 把这三件事做成了一个统一模型。

**第二步：读教程（这是本阶段主线）**
`docs\cordis-tutorial\` 全 8 篇，**从头按顺序读**，一共只要 36KB：
`index.zh.md` → `01-first-plugin.zh.md` → `02-lifecycle-and-effects.zh.md` → `03-services.zh.md` → `04-events.zh.md` → `05-config.zh.md` → `06-composition-and-hmr.zh.md` → `07-into-the-harness.zh.md`
搭配 `docs\cordis-primer.zh.md`（重点：五种事件分发模式）。

**第三步：读源码**（从下往上，`vendor\cordis\src\`）
`service.ts`（一个服务是什么）→ `registry.ts`（插件怎么被登记）→ `context.ts`（`ctx` 从哪来）→ `fiber.ts`（生命周期与「可逆副作用」）

**要真正握在手里的一个概念**：`ctx.effect(fn)` 里 `fn` **立即执行**，并且**返回一个"撤销函数"**；插件被卸载时，这些撤销函数**逆序**执行。这就是"卸载后不留残渣"的实现方式——**每个注册都是一笔可回滚的账**。

**动手**
- 在 `.learning\experiments\` 写一个最小 cordis 插件（20 行左右）：一个函数 + `apply(ctx)` + 一个 `ctx.effect`
- `vendor\cordis` 已经是构建好的（入口 `vendor\cordis\lib\index.js`），脚本直接引用它即可，**不用装依赖、不用改仓库**
- 故意做一次对比实验：一次把注册写在 `apply` 里直接执行，一次包在 `ctx.effect` 里然后主动卸载插件，观察两者的差别

**检查点**
- 插件系统要解决的四个需求分别是什么？各自靠什么机制实现？
- `inject` 声明了什么？去掉会怎样？
- 为什么"每个注册都必须返回一个撤销函数"这么重要？
- `emit` / `waterfall` / `parallel` / `serial` / `bail` 有什么区别？（哪个不调 `next()` 会短路？）

**产出物**：`.learning\experiments\` 下的最小 cordis 插件 + `notes\02-cordis的可逆注册.md`

## 阶段 5 · 写你的第一个插件（3–5 天）

**学什么**：把 cordis 的知识用到 DSH 上。DSH 本身就是一个 cordis 应用，它把"可扩展点"开放出来给你插。

**读什么**
1. `docs\cookbook\extension-cookbook.zh.md` —— **扩展点总览，本阶段的路线图**（工具、命令、设置卡片、UI 槽位、模型适配器……）
2. `docs\cookbook\adding-a-tool.zh.md` —— 跟着它做一个"工具"插件（最容易上手的一类）
3. `docs\capability-seams.zh.md` —— 能力接缝：定义 / 提供者 / 使用者三角色（理解"可替换"是怎么落地的）
4. **对照一个真实插件**：`~\.dsh\profiles\desktop\node_modules\deepseek-idesign`（第三方插件，已装在你机器上）——看它的 `package.json` 有哪些 `dsh` 字段、`cordis.patch.yml` 写了什么、入口文件怎么注册东西

**动手（本阶段核心产出）**
1. 在 `.learning\plugins\` 下建一个最小插件：`package.json` + `cordis.patch.yml` + 入口文件
2. 按官方 cookbook 的流程打包并安装到你的 profile
   **注意**：规范里明确禁止"直接安装本地目录"，先读它给的原因（这与依赖解析和软链接有关）
3. 重启 DSH，验证你的工具能被调用
4. 卸载，然后**去日志里确认没有残渣**（这正好复用阶段 1 的脚本）

**检查点**
- `package.json` 里 `dsh.bundle` 与 `dsh.client` 分别是干什么的？
- `cordis.patch.yml` 的 `insert` 是什么意思？为什么装完必须重启？
- 你的插件卸载后，怎么证明"什么都没留下"？
- 客户端插件（跑在浏览器里）和宿主插件（跑在 Node 里）为什么必须分开？两者怎么通信？

**产出物**：一个能跑的插件（含 README：它做什么、需要什么权限、有什么副作用）

## 阶段 6 · 内核精读（4–6 天）

**学什么**：harness 思想的骨头——**事件溯源**与**工具执行流水线**。

**读什么**（`packages\core\`）
1. `README.zh.md` —— 这一层的分组地图
2. `agent\README.zh.md` —— Agent 句柄：外部只能通过它跟 agent 交互
3. `session\README.zh.md` + `session\src\index.ts` —— **仅追加的事件日志**，以及消息是怎么"推导"出来的
4. `system-prompt\README.zh.md` —— 提示词如何被分段拼装
5. `tools\README.zh.md` —— 工具注册表与把关流水线
6. `scope\README.zh.md` —— 作用域：为什么不同会话可以有不同的能力集
7. `agent-loop\README.zh.md` —— **含源码地图与设计理念，先读它再读代码**
8. `agent-loop\src\` 的源码：`agent.ts`（轮次/步骤状态机）、`inbox.ts`（消息收件箱）、`runtime-context.ts`、`tool-calls.ts`（工具并发调度）、`assistant-stream.ts`

**三个要彻底搞懂的设计**
1. **"模型可见即已记录"**：请求必须先写进日志才能发出去。为什么要多这一步？（提示：可复现、可审计、可恢复）
2. **两套事件**：持久的 `session/*`（可回放）与实时的 `agent/*`（含流式片段）。为什么要分两套？
3. **工具三阶段**：`tool/call` → 前置把关（守卫、审批）→ 真正执行 → 后置处理 → `tool/result`。中间每一处都是"可插拔的拦截点"。

**动手（不碰源码）**
- 跑仓库自带测试（零改动）：
  `pnpm vitest run packages/core/agent-loop/tests/loop.spec.ts`
  `pnpm vitest run packages/core/tools/tests/tools.spec.ts`
  配套的 `packages\core\agent-loop\tests\mock-adapter.ts` 用**脚本化的假模型**驱动循环，所以不需要真实 API
- 自己写一个小测试，断言"一个 turn 里事件的先后顺序"。文件放在仓库测试目录下（跟既有测试同样位置），它是**未跟踪新增文件**，已跟踪的源码仍与原版逐字节相同
- 回到阶段 1 的脚本，这次带着问题看：为什么 `tool/call` 和 `tool/result` 一定成对？如果只看到 `tool/call` 没有 `tool/result`，说明发生了什么？

**检查点**
- 为什么会话日志是"只能追加"的？
- 什么时候会发生"压缩"？为什么它只做"遮蔽"而不真的删除？
- `agent/request` 与 `llm/stream` 各负责什么？
- 工具的并发是怎么调度的？为什么要"独占屏障"？

**产出物**：`notes\03-一次请求的完整时序.md`（带真实事件名与源码位置）

## 阶段 7 · 产品全景（3–5 天）

**学什么**：同一个内核，怎么长出 CLI、Web、桌面、SDK 四种产品形态。这是"全栈"视角的关键一环。

**读什么**
1. `packages\boot\app-boot\src\profile.ts` —— **产品是被"装配"出来的**：profile 里列出要哪些 bundle，bundle 里带 patch，用户可以再叠 patch
2. `packages\boot\hmr\README.zh.md` —— 什么能热更新、什么必须重启
3. `packages\bundle\` —— 现成的组装方案：`base` / `web-app` / `headless` / `sdk-app` / `sdk-minimal` / `acp-app`（注意 `sdk-minimal` 故意不带 `base`，是理解"可裁剪"的好例子）
4. `apps\` —— 真正面向用户的应用层（CLI、桌面、Web 前端）
5. `packages\api\` + `docs\api-gateway.zh.md` —— 后端接口层
6. `packages\client\`（59 个客户端包）+ `docs\subsystems\`（64 篇中文子系统文档，按需查）
7. `packages\sdk\` —— 用代码驱动 DSH 的官方方式

**动手**
- 用仓库自带的"自省脚本"看全景（它们**从代码生成文档**，是极好的学习工具）：
  `pnpm run verify-module-graph`、`pnpm run verify-tool-catalog`、`pnpm run verify-config-catalog`
  然后对照生成物 `docs\module-graph.zh.md`、`docs\tool-catalog.zh.md`、`docs\config-catalog.zh.md`
- 翻完 `docs\upgrade-guide\` 的 6 份指南：体会一个真实项目怎么管理"契约变更"

**检查点**
- 从 `dsh-base` 到一个完整的 Web 产品，中间加了什么？
- 换掉一个能力提供者（比如把本地文件换成远程存储），会影响哪些部分？
- 客户端插件与宿主插件的边界在哪？为什么要划这条线？

**产出物**：`notes\04-产品装配与插件边界.md`

---

# 三、学习成果

## 3.1 各阶段检查点（汇总，能全部讲清才算过）

| 阶段 | 必答问题 |
|---|---|
| 1 | DSH 与直接用网页版聊天机器人的区别？工具调用成对出现的意义？ |
| 2 | 模型为什么必须靠工具调用才能读文件？harness 比裸调 API 多做了什么？ |
| 3 | 一次请求的完整事件序列？"把关"在哪几处、各防什么？ |
| 4 | 插件系统的四个需求与对应机制？`ctx.effect` 的"可逆"为什么关键？ |
| 5 | 插件怎么被装上和卸下？怎么证明卸载没留残渣？ |
| 6 | 为什么日志只能追加？压缩为什么是遮蔽？工具并发怎么调度？ |
| 7 | 产品怎么被"装配"出来？能力替换会波及哪些层？ |

## 3.2 各阶段产出物

| 阶段 | 产出物 | 放哪 |
|---|---|---|
| 1 | 第一次使用与日志观察笔记 + 日志解析脚本 | `notes\01-*.md`、`experiments\dump-session-events.mjs` |
| 2 | 手绘「最小 agent 循环」图（用自己的词） | `notes\` |
| 3 | DSH 版全链路图（带真实事件名） | `notes\` |
| 4 | 最小 cordis 插件 + 可逆注册笔记 | `experiments\`、`notes\02-*.md` |
| 5 | **一个能跑的 DSH 插件**（含 README） | `plugins\` |
| 6 | 一次请求完整时序长文 + 自写测试 | `notes\03-*.md` |
| 7 | 产品装配与插件边界笔记 | `notes\04-*.md` |

## 3.3 毕业检查清单

- [ ] 我能解释 harness 存在的原因，以及它比裸调 API 多做了什么
- [ ] 我能画出 DSH 的运行全景，并说出关键事件名
- [ ] 我能解释插件系统要解决的四个需求，以及 cordis 如何实现
- [ ] 我能写出最小 cordis 插件，并解释"可逆注册"
- [ ] 我能写出 DSH 插件并完成打包→安装→验证→卸载
- [ ] 我能解释事件溯源：为什么日志只追加、压缩只遮蔽
- [ ] 我能说清一次工具调用经过的所有把关环节
- [ ] 我有 4 份以上笔记 + 1 个插件 + 1 个脚本作为可展示产出

## 3.4 成果去向：怎么用于求职

| 要证明的能力 | 用什么证明 |
|---|---|
| 理解 AI Agent 工程 | 阶段 3/6 的时序长文：事件溯源、工具流水线、并发调度、上下文压缩 |
| 能设计可扩展架构 | cordis 可逆注册的分析 + 自己的插件 + 与 VS Code / 浏览器扩展的机制对比 |
| 懂 LLM 应用全栈 | 阶段 7 的产品装配分析：同一内核 → CLI / Web / 桌面 / SDK |
| 会动手验证而非空谈 | 自己写的日志解析脚本 + agent loop 断言测试（都是真实可跑的代码） |
| 工程素养 | 仓库里 60+ 个 `verify-*` 门禁脚本的思路（用生成物替代手工文档） |

**面试可讲的四个反常识点**（这类内容最能体现真的读过）
1. 请求**先落日志、再发出去**——所以能重放、能审计。
2. 会话压缩是**遮蔽**而非删除——因为日志不可变。
3. 插件卸载能撤销全部注册，因为**每个注册都是带回滚的账**，连 agent loop 本身都能被配置替换。
4. 会话日志是**追加式 + 逐帧压缩**的（一个 456KB 文件里有 331 个 zstd 帧），所以它天生适合 append-only 写入。

## 3.5 进度记录

| 阶段 | 状态 | 完成日期 | 产出物 |
|---|---|---|---|
| 1. 先跑起来 | ☐ 未开始 | — | 观察笔记 + 解析脚本 |
| 2. 概念地基 | ☐ 未开始 | — | 最小循环图 |
| 3. 完整链路 | ☐ 未开始 | — | DSH 全链路图 |
| 4. 插件思想与 cordis | ☐ 未开始 | — | 最小 cordis 插件 + 笔记 |
| 5. 第一个插件 | ☐ 未开始 | — | 可运行的 DSH 插件 |
| 6. 内核精读 | ☐ 未开始 | — | 时序长文 + 测试 |
| 7. 产品全景 | ☐ 未开始 | — | 装配分析笔记 |
