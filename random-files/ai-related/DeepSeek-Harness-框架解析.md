> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0PctDmlipiy?projectId=f3d2b0c674fb4db2af4b208d7c43e49f

# you asked

message time: 2026-08-15 19:30:52

解释 deepseek harness

---

# yuanbao response

"DeepSeek Harness"（简称 DSH）是 DeepSeek 于 2026 年 8 月 13 日随 V4 Pro 正式版一同开源的**智能体（Agent）运行框架**，MIT 协议，当前版本 v0.1 开发者预览版[citation:5][citation:7]。要理解它，先得理解 "harness" 这个词在 AI 圈的含义，再看 DeepSeek 的具体实现。

## "Harness" 在 AI 语境里指什么

"Harness" 本意是马具——套在马上让它听话干活的那套束缚装置。在机器学习领域，这个词今年开始被广泛用来指代**包裹在模型外面的那层中间件/调度系统**：它负责处理喂给模型的输入、接收模型返回的输出，并管理提示词、上下文、工具编排、Agent 循环、状态管理、错误处理、安全与权限等一系列"让模型真正能干活"的琐事[citation:11]。

Claude Code 是 Anthropic 模型的 harness，Codex 是 OpenAI GPT 的 harness，类似的还有 Aider、Cline、Goose、OpenCode 等[citation:11]。同一个模型，套上不同的 harness，跑分和成本可以差出 30%[citation:12]——这就是为什么 2026 年下半年，行业竞争的焦点正从"模型能力"转向"harness 工程"[citation:16]。

DeepSeek 官方用一条公式把这个关系讲得很直白：

> **Agent = Model + Harness**
> 模型是大脑，负责推理；Harness 是身体和神经系统，负责把推理落到真实环境里[citation:1][citation:5][citation:8]。

没有 harness，模型只是个聊天机器人，碰不到你的文件、跑不了你的命令；套上 harness，它才变成能读代码、改文件、跑测试、提交 commit 的 Agent[citation:1][citation:3]。

## DeepSeek Harness 的具体定位

DSH 不是又一个绑定自家模型的 Coding 助手，而是一套**力求通用的 Agent 运行基础设施**。它的差异化在于架构哲学——**"一切皆插件"（Everything is a Plugin）**[citation:5][citation:7][citation:11]。

底层跑的是一个叫 **Cordis** 的插件元框架（北大 + DeepSeek 联合研发，已支撑 Koishi 聊天机器人生态四年），内核只干三件事：插件加载、卸载、依赖管理[citation:4][citation:14]。在这个内核上，Harness 把以下能力**全部实现为可替换的插件**[citation:11][citation:14]：

- 🧠 **模型适配器**：可接入 DeepSeek、OpenAI、Claude、Kimi 等 20+ 主流模型[citation:2]
- 🔧 **工具系统**：文件读写、Shell、浏览器、API 调用
- 💾 **会话与存储**：仅追加（append-only）日志，完整记录系统提示词、思维链、工具调用、子 Agent 调度
- 📦 **沙箱**：本地或远程隔离环境
- 🔄 **Agent Loop**：执行循环、重试策略、回滚逻辑
- 🖥️ **UI**：Web、CLI、Headless 等

默认部署下就有 **159 个插件**，且每个都能单独开关[citation:14]。开发者无需改框架源码，仅靠配置文件就能换模型、加工具、切沙箱、挂审计钩子——部署差异归到配置层，而不是代码分支[citation:3][citation:14]。

## 四种预制运行模式

针对不同场景，DSH 预置了四种 Profile[citation:2][citation:7]：

| 模式 | 适用场景 |
|---|---|
| **标准模式** | 通用首选，代码开发、数据分析，全套工具栈 |
| **PTC 程序化模式** | 模型生成 TypeScript 代码串联多轮工具链，适合复杂自动化 |
| **极简模式** | 只留 Shell + 文件编辑，专为公平基准测试设计 |
| **创造模式** | 运行时调试插件、自定义全新 Agent 预设 |

其中**极简模式**挺有意思：DeepSeek 专门做了一个"不讨好模型"的模式来跑分，相当于公开表态"我的模型经得起最严苛的 harness 考验"，跟某些把 harness 当滤镜用的厂商形成对比[citation:3]。

## 它解决的实际问题

光说不练没感觉，看一组 DeepSeek 官方跑分——同一个 V4-Pro 模型，**套上 Harness 前后**的对比[citation:3]：

- DeepSWE：12.8 → 62.7（**+49.9**）
- DSBench-Hard：31.1 → 67.2（**+36.1**）
- Cybergym：52.7 → 83.3（**+30.6**）
- Terminal Bench 2.1：72.1 → 87.9（**+15.8**）

模型权重没变，涨的都是 harness 的功劳。这印证了一个趋势：**2026 下半年，模型能力和参数的耦合正在解耦**，后训练、harness、解码策略、缓存任何一环的进步都能直接反映到跑分上[citation:3]。

## 快速上手

基于 Node.js / TypeScript，三条命令跑起来[citation:3][citation:10]：

```bash
npx @deepseek-ai/dsh web
```

浏览器打开 `127.0.0.1:3080` 即可使用，数据都在本地。

---

**所以一句话解释**：DeepSeek Harness 是 DeepSeek 开源的、把"大模型"变成"能动手干活的 Agent"的那套运行框架，核心思想是所有能力模块化、插件化，让开发者可以自由拼装属于自己的 Agent 运行时。它对标的是 Codex 和 Claude Code，但走的是完全开源、模型无关、插件至上的路线[citation:5][citation:8]。

> 💡 如果你只是想用 AI 帮改代码，把它当作开源版的 Claude Code 来用就行；如果你是框架开发者，它的"一切皆插件"架构和 append-only 日志系统才是真正值得深挖的部分——前者给灵活性，后者让 Agent 的每一步都可追溯、可回放、可调试[citation:12][citation:14]。

---

# you asked

message time: 2026-08-15 19:34:27

实例，python友好吗？

---

# yuanbao response

对 Python 开发者来说，DeepSeek Harness **友好，而且是有意为之的友好**——它专门提供了 Python SDK，让你不用碰 Node.js 那一整套也能把 Agent 能力嵌进自己的 Python 应用里。

## 为什么说它对 Python 友好

DeepSeek Harness 的主体虽然是 TypeScript/Node.js 实现（CLI 用 `npx @deepseek-ai/dsh` 启动），但它**针对 Python 开发者单独发布了 SDK 包** `deepseek-harness-sdk`，并且做了两件降低门槛的事：

- **SDK 内置运行时**，宿主机不需要单独装 Node.js
- **支持 Python 3.10+**，Linux x64/arm64 或 macOS 14+ arm64

也就是说，Python 开发者走 `pip install` 这条路就能直接用，不必为了用 Harness 而去配一套 Node 环境。

## 最小可运行实例

```bash
pip install deepseek-harness-sdk
export DEEPSEEK_API_KEY="sk-your-key"
```

```python
from pathlib import Path
from deepseek_harness import DeepSeekHarness

CONFIG = Path("examples/jsonrpc-agent/minimal.cordis.yml").resolve()
workspace = Path("/abs/path/to/workspace").resolve()
sessions = Path("/abs/path/to/sessions").resolve()

with DeepSeekHarness(
    provider="deepseek-official",
    model="deepseek-v4-flash",   # 或 deepseek-v4-pro
    max_tokens=49_152,
    cwd=str(workspace),
    session_root=str(sessions),
    cordis=str(CONFIG),
) as harness:
    result = harness.run(
        "检查代码仓库，并修复失败的测试。",
        session_id="example-001",
    )
    print(result.final_response)
```

`harness.run()` 返回的是 `RunResult` 对象，常用字段：

- `final_response`：模型最终输出
- `finish_reason`：`completed` / `max-tokens` / `error`
- `events`：完整事件流，思维链与工具调用全部有迹可循

## 几个 Python 场景下很好用的特性

**1. Headless 模式跑 Python 任务**

```bash
dsh run --profile headless "Create calc.py with add/sub/mul/div functions, run and test 2+3, output the result."
```

**2. 批量评测脚本**

```python
TASKS = [
    ("task-001", "Inspect the repository and fix the failing tests."),
    ("task-002", "Implement the missing function in src/main.py."),
]
with DeepSeekHarness(...) as harness:
    for sid, prompt in TASKS:
        result = harness.run(prompt, session_id=sid)
```

**3. Session 复用**：同一个 `session_id` 跨多次 `run()` 调用会保留 Bash 进程、工作目录、Shell 变量——这意味着你可以分步让 Agent 改 Python 项目，上下文不断档。

**4. Append-only 日志**：每次运行的完整事件流都会落盘，支持恢复、分叉、回放，做实验复现非常方便。

> ⚠️ 一个 Python 开发者容易踩的坑：不要在 Harness 上下文内使用 `print()` 或将日志写向 stdout。Harness 子进程的 stdout 被 SDK 协议占用，普通 print 会干扰 JSON-RPC 协议，日志请写文件。

## 客观的边界

- **Windows 原生不在官方支持矩阵内**（Linux x64/arm64 或 macOS 14+ arm64），Windows 用户可能需要 WSL2。
- Harness 的**插件生态本体是 TypeScript**（基于 Cordis 框架），如果你想深度定制插件，还是得写 TS；Python SDK 主要是"消费方"角色，调用已有 Agent 预设。
- 框架本身仍是 **v0.1 开发者预览版**，API 可能变动。

---

**所以结论很明确**：如果你是个 Python 开发者，想在自己的项目里嵌入一个能读文件、跑命令、修代码的 Agent，DeepSeek Harness 的 Python SDK 几乎零门槛——`pip install` + 几行代码就能跑起来，对 Python 技术栈是真正友好的。但如果你要做插件级深度定制，那层还是 TypeScript 的天下。

---

# you asked

message time: 2026-08-15 19:37:11

为什么typescript nodejs 为主体

---

# yuanbao response

一句话先给结论：**这是"插件架构需要 + 团队技术路径 + 行业共识"三重因素叠加的结果**，不是拍脑袋选的。

下面拆开说——

## 1. Cordis 本身就是 TypeScript 生态的（路径依赖）

DSH 的插件内核 Cordis，源自 Koishi（一个用 TypeScript 写的开源聊天机器人框架，北大团队维护，已经跑了 4 年）。Koishi 本身就是 TS + Node.js 的，Cordis 是从里面抽出来的插件元框架。

DeepSeek 团队选择 Cordis 做 Harness 的底座，等于**直接继承了一整套已经验证过的插件加载、依赖注入、生命周期管理机制**——这些能力如果用 Python 从零写，工作量巨大且不一定稳定。既然底座是 TS 的，上层建筑自然也是 TS。

## 2. Agent 运行时恰好是 Node.js 的"甜区"

Agent 框架干的核心活儿是什么？

| Agent 运行时的需求 | Node.js/TS 的匹配度 |
|---|---|
| 大量并发的异步 I/O（流式输出、工具调用、API 请求） | ✅ 事件循环 + async/await 原生支持，V8 处理高并发 I/O 非常成熟 |
| 长时间运行的进程（Agent Loop 可能跑几分钟） | ✅ Node 单进程长驻模型天然适合 |
| 需要调用各种外部工具/CLI/Shell | ✅ `child_process` 成熟，跨平台 |
| 插件要能动态加载/卸载/热更新 | ✅ CommonJS + ESM 动态 `import()` 很灵活 |
| 前端 UI（Web 界面） | ✅ 同一门语言前后端通吃 |

反过来看 Python——它在**模型训练和数据处理**上是绝对王者，但在**长时间运行的后台服务 + 插件热加载 + 前后端统一**这块，Node.js 的体验确实更顺滑。

## 3. "一切皆插件"需要强类型系统

DSH 有 159 个插件，插件之间还有依赖关系。这种规模的插件系统如果写在 Python 里：

- 接口约定靠文档和 `Protocol`/`abc`，运行时才发现类型不对
- 插件 A 依赖插件 B 的某个方法，B 改了签名，A 要到运行时才炸
- 159 个插件的自动补全和重构？Python 的 LSP 体验远不如 TS

TypeScript 的**编译时类型检查 + 接口约束 + 装饰器 + 依赖注入**，对管理这种规模的插件生态几乎是刚需。你可以把它理解为：插件系统越大，越需要 TS 这种"把错误拦在编译期"的语言。

## 4. 行业对标：主流 Coding Agent 几乎都选了 TS

这不是 DeepSeek 独有的选择，看看 2026 年的格局：

| Agent 框架 | 主体语言 |
|---|---|
| Claude Code (Anthropic) | TypeScript |
| Codex CLI (OpenAI) | TypeScript（部分 Go） |
| Cline (VS Code 插件) | TypeScript |
| Aider | Python（少数派） |
| OpenCode | TypeScript |
| Goose (Block) | Go + TS 混合 |

**为什么？** 因为 Coding Agent 本质上是一个**编排层（orchestration layer）**——它不跑模型推理（那是 GPU 的事），它做的是调度、I/O、工具链管理、UI 渲染。这个层用 TS/Node 写，开发效率高、生态好、前后端统一，是行业共识。

Python 在 AI 领域的绝对优势在**模型侧**（PyTorch、transformers、训练脚本），但 Agent harness 是**应用侧**——这两侧的"最适语言"在 2026 年已经明显分化了。

## 5. Python SDK 是"务实的桥接"，不是"重写"

DeepSeek 的做法很聪明：

```
┌─────────────────────────────────┐
│   TypeScript/Node.js 主体        │  ← 159 个插件、Cordis 内核、Web UI
│   （完整能力，插件开发者的世界）   │
├─────────────────────────────────┤
│   JSON-RPC 协议层                │  ← 进程间通信
├─────────────────────────────────┤
│   Python SDK（薄封装）           │  ← pip install，内置 Node runtime
│   （消费方：调 Agent，不写插件）  │
└─────────────────────────────────┘
```

Python SDK 不需要用户装 Node.js，因为它**把 Node runtime 打包进去了**，通过 JSON-RPC 跟 TS 主体通信。这就像 Playwright 的 Python 包——底层是 Node.js 的浏览器自动化引擎，但 Python 开发者只需要 `pip install playwright` 就能用。

这个策略的好处是：**一份核心代码维护，多语言都能用**，不用给每种语言重写一遍 harness 逻辑。

## 总结

| 维度 | 为什么选 TS/Node |
|---|---|
| 历史路径 | Cordis 底座就是 TS 的，改不了 |
| 架构匹配 | 异步 I/O + 长进程 + 动态插件加载 = Node 甜区 |
| 规模管理 | 159 个插件需要强类型，TS 碾压 Python |
| 行业趋势 | 所有主流 Coding Agent 都选了 TS |
| Python 策略 | 不重写，用 SDK 桥接——你用 Python 调，底层还是 TS 跑 |

所以本质上是：**DeepSeek 把"最难写的编排层"用最合适的工具写了，然后给 Python 开发者留了一扇好开的门。** 这不是对 Python 不友好，而是工程上最合理的分工——模型推理在云端（语言无关），编排调度用 TS（最顺手），你用 Python 调它（最方便）。

---

# you asked

message time: 2026-08-15 19:39:50

做为一个data analyst data engineer data scientist 学习typescript的成本，有，多高，

---

# yuanbao response

直接给你一个判断：**作为数据分析师/数据工程师/数据科学家，学习 TypeScript 的成本不算高，但 ROI（投资回报率）取决于你想用它干什么**。如果你只是想"能读懂、能改、能写简单的 Agent 工具调用"，成本很低（几十小时）；如果想用 TS 替代 Python 做数据分析，那不仅成本高，而且方向错了。

## 你的基础决定了起点并不低

很多人学 TS 卡在 JavaScript 基础上——因为 TS 本质是"带了类型系统的 JavaScript"。但你已经有了编程经验（Python/SQL），这意味着：

- **类型心智模型可迁移**：你在 Python 里用 `mypy` 或 type hints 的经验，直接对应 TS 的静态类型系统
- **异步编程可迁移**：Python 的 `async/await` 心智模型在 TS 里完全对应
- **工具链意识可迁移**：`venv/pip` 的隔离概念对应 `pnpm/node_modules`
- **真正的障碍只有两个**：① JS 的"怪癖"（原型链、`this` 绑定、事件循环细节）；② 接受"写完了还要编译一步"的工程化思维——Python 的 `python script.py` 直接跑的心智要改

业界对学时的估算：系统培训课程大约 **54 小时**（9-12 周业余时间）；入门指南估算 **20-40 小时**能从入门到熟悉；如果已经懂 JS，大约 **2-4 周** 能上手。对你而言，因为有编程底子，**实际耗时会落在这些估算的下限**——重点不是"学语言"，而是"学生态和工程惯例"。

## 成本分三档，对号入座

### 🟢 第一档：能读、能改、能写简单脚本（成本：低）

**目标**：看懂 DeepSeek Harness 的配置文件、写个调用 LLM API 的脚本、改改别人的 TS 代码

**需要掌握**：
- 基本类型（`string`/`number`/`boolean`/`array`/`object`）
- `interface` 和 `type`（定义数据结构——对应 Python 的 TypedDict/Pydantic model）
- 函数签名带类型
- `async/await`（你 Python 里已经熟了）
- 读懂 `tsconfig.json`

**时间投入**：**20-40 小时**，分散到 2-3 周，每天 1 小时

**对你的价值**：⭐⭐⭐⭐
- 能直接玩 DeepSeek Harness 的插件和配置
- 能写 LLM 应用层的胶水代码
- 看懂 Vercel AI SDK、LangChain.js 等主流 AI 应用框架

### 🟡 第二档：能写 Node.js 服务端 / 数据 API / Agent 工具（成本：中）

**目标**：用 TS 写生产级的数据服务、REST API、给 Agent 写自定义工具插件

**额外需要掌握**：
- 泛型（Generics）
- Union 类型、Utility Types（`Partial`/`Pick`/`Omit`）
- npm/pnpm 包管理、模块系统
- Node.js 运行时细节、Express/Fastify/NestJS 中的一个
- 用 Zod 做运行时校验（因为 TS 的类型在编译后会被擦除，这点跟 Pydantic 完全不同，是最大的陷阱）
- 测试（Vitest）

**时间投入**：在第一档基础上再加 **60-100 小时**实战

**对你的价值**：⭐⭐⭐
- 能在 Python 数据管线之上，用 TS 写高性能的 API 层服务前端
- 能为 DeepSeek Harness 写生产级插件
- 数据科学岗位中要求 TypeScript 的比例在 2026 年显著上升，特别是 AI 产品公司

### 🔴 第三档：全栈 TypeScript / 前端开发（成本：高）

**目标**：React/Vue 前端 + Node 后端全栈

**额外需要掌握**：
- React 或 Vue 框架
- 前端构建工具（Vite/Webpack）
- CSS、DOM、浏览器 API
- 高级类型体操（条件类型、映射类型、模板字面量类型）

**时间投入**：**200+ 小时**，相当于一次职业转型

**对你的价值**：⭐⭐（对纯数据角色而言）
除非你想转型做 AI 应用工程师或全栈，否则不建议数据人投入这一档。

## 关键认知：Python 和 TS 不是替代关系，是分工

这是最重要的部分。**2026 年的主流技术栈已经是"混合栈"**：

| 层 | 主力语言 | 你的角色 |
|---|---|---|
| 数据探索、特征工程、模型训练 | **Python** | 数据分析/科学家的主场 |
| 数据管线、批处理 | **Python** | 数据工程师的主场 |
| AI 应用层、Agent 编排 | **TypeScript** | 新兴交叉地带 |
| Web 前端、内部 Dashboard | **TypeScript** | 协作层 |
| 后端 API 服务 | TS 和 Python 都能做 | FastAPI(Py) vs NestJS(TS) |

数据很清楚：Python 仍是 AI/数据科学仓库的首选（GitHub 上 AI-tagged 仓库 58.2 万个，年增 50.7%），但 TypeScript 是 GitHub 总使用量第一的语言（263 万月活贡献者）。两者不是你死我活——**Python 写模型和管线，TypeScript 写应用和 API，是现代 AI 产品的标配组合**。

> 💡 一个真实的工程现象：很多数据团队尝试用 TS 做数据分析原型，结果发现——每改一行要等 `tsc` 编译、嵌套数据结构要写一堆 interface、pandas 一行能搞定的事在 TS 里要写几十行。所以**原型探索阶段 Python 完胜，生产应用层 TS 占优**。

## 给你的具体建议

**如果你主要是数据分析师/科学家**：
- 学到 **第一档** 就够了（20-40 小时）
- 目标：能读 TS 代码、能写调用 LLM API 的脚本、能配置 DeepSeek Harness
- 不需要深入前端、不需要学 React
- Python 仍然是你的主武器，TS 是"辅助理解 AI 应用层"的工具

**如果你主要是数据工程师**：
- 建议学到 **第二档**（总计 80-140 小时）
- 目标：能用 TS 写数据 API、给 Agent 系统写工具插件、做 Python↔TS 的服务间契约
- 这在 2026 年的 AI 工程岗位里是**明确的加分项**——能横跨 Python 数据层和 TS 应用层的工程师特别值钱

**时间怎么安排**（针对第一档）：
1. **Week 1**：花 5-8 小时学 TS 基础类型 + interface + 函数类型（推荐 TypeScript 官方 Handbook）
2. **Week 2**：花 5-8 小时学 async/await + 用 `fetch` 调 API + 读 2 个开源 TS 项目的代码
3. **Week 3-4**：花 10-24 小时做个小项目，比如"用 TS 调用 DeepSeek API 批量处理 CSV 数据"

## 一个反直觉的结论

**你作为数据人学 TS，最大的成本不是语法，而是"思维切换"**：
- Python 里 `dict` 随便传 → TS 里得先定义 `interface`
- Python 里运行时类型还在 → TS 里类型编译后被擦除，校验得用 Zod
- Python 里写完就跑 → TS 里多了一步编译

这三点每个 Python 程序员都会绊一下，但**一旦适应，你会反过来觉得 Python 太随意**。很多学了 TS 的数据工程师反馈："给复杂的数据管道加了类型后，重构信心完全不一样。"

> ⚠️ **别踩的坑**：不要用 TS 重写你的数据分析代码。pandas、numpy、scikit-learn、PyTorch 这些在 TS 里要么没有、要么性能差一个数量级。TS 的定位是"AI 应用层 + 工程化"，不是"数据分析替代者"。

---

**所以回到你的问题：成本高不高？**

不高，但要有边界——**花 20-40 小时学到"能读能改能写简单 TS 脚本"这一档，对任何数据岗位都是高 ROI 的投资**；再往上追全栈 TS，除非职业转型，否则边际效益急剧下降。Python 是你的主航道，TS 是让你在主航道之外能"跟应用工程师对话、能扩展 Agent 能力"的辅助技能。两者叠加，才是 2026 年数据人的完整竞争力。