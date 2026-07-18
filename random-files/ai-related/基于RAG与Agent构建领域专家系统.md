> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0OTbRiCUnyi

# you asked

message time: 2026-07-09 22:08:34

我希望请教您一个问题，看看如何可以使用AI实现。是搭建知识库还是别的agent。

我的需求是：

我希望了解某一个领域，下载了一些文献，存储了一些文字资料（比如电子书等）。可以通过什么技术搭建一个类似于“老师”，“专家”的角色，通过问答或者自定义的课程设计来帮助我“从入门到精通”。

请问类似的技术，有开源课程可以吗？

---

# yuanbao response

你的需求拆开来其实是两件事：**"让 AI 读懂你的文献资料并回答相关问题"** 和 **"让 AI 扮演老师/专家，给你设计从入门到精通的课程路径"**。前者用 **RAG（检索增强生成）知识库** 就够，后者要在 RAG 基础上加 **Agent 工作流 + Prompt 人设工程**，两者是递进关系，不是二选一。

下面分四块讲：技术定性 → 三条落地路径（按技术胃口选）→ "老师角色"怎么塑造 → 开源课程推荐。

---

## 一、先定性：你的场景到底用 RAG 还是 Agent？

- **纯问答**（"这篇文献里作者怎么说 X 的？""帮我总结第二章"）→ **RAG 知识库** 搞定。流程是：文献 → 解析切片 → Embedding 向量化 → 存向量库 → 用户提问时检索相关片段 → 拼进 Prompt 给大模型生成。数据不离本地也能跑[citation:4]。
- **"老师/专家 + 课程设计"**（诊断你水平 → 出学习路径 → 分阶段讲授 → 出题测验 → 纠偏）→ 这是 **Agent 场景**。需要多步推理：模型自己决定"下一步该讲啥、要不要出道题、要不要回查知识库"，得用 LangGraph / Dify 工作流 / FastGPT 这类支持 Agent 编排的框架[citation:1][citation:9]。
- 现实做法：**RAG 打底，Agent 在上层编排教学流程**。先让 AI 能基于文献答对问题，再加"老师"的行为逻辑。

---

## 二、三条落地路径，按你想投入的代码量选

### 🟢 路径 A：零代码 / 桌面端，10 分钟跑通（新手首选）

适合"我想先看到效果，不想碰代码"。

- **AnythingLLM**（桌面端，Win/Mac/Linux）
  - 内置 Chroma 向量库 + 本地嵌入模型，装好 Ollama 拉个 `qwen:7b` 就能跑
  - 支持 PDF/MD/TXT/Notion 批量上传，文件夹级知识库
  - 本地运行，文献不出机[citation:4]

- **ChatWiki**（开源，Docker 部署） / **MaxKB**（1Panel 出品，GitHub 15k+ star）
  - 都是 Web 界面，支持 PDF/DOCX/MD 上传 → 自动切片向量化 → 关联机器人
  - MaxKB 多了工作流引擎，能往后接 Agent 场景留口子[citation:6][citation:10]

> 💡 这条路先别管"老师课程设计"，先把文献喂进去，看问答质量能不能接受。能接受再往 B/C 升。

### 🟡 路径 B：低代码工作流编排（推荐折中方案）

如果想做"老师"行为（出大纲、分阶段讲、出题），光 RAG 不够，需要工作流把多个步骤串起来。

- **Dify**：可视化工作流（Chatflow 引擎），条件分支/循环/并行都支持，RAG 引擎对 20+ 文档格式友好，能接 Ollama 本地模型也能接云端[citation:9]
- **FastGPT**：开源，QA 结构优化（对"老师答疑"场景比纯 chunk 召回准），Flow 模块可拖拽编排复杂问答，API 对齐 OpenAI[citation:1]
- **RAGFlow**：偏企业合同/FAQ 场景，但可视化拖拽 + 多模态（PDF/图）是亮点

在 Dify/FastGPT 里你能做的"老师"雏形：
```
用户水平诊断 → 查知识库生成课程大纲 → 分段讲授（每次检索对应章节） → 出题节点 → 判卷 → 下一章
```
每个节点是工作流里的一个 step，不用手写 Agent 逻辑。

### 🔵 路径 C：自研代码栈（自由度最高，技术向）

技术栈基本是这套组合[citation:2][citation:4]：

| 模块 | 选型 |
|---|---|
| 文档解析 | LangChain / LlamaIndex（`DirectoryLoader` 吃 PDF/MD/DOCX） |
| 文本切片 | `RecursiveCharacterTextSplitter`，按领域调 `chunk_size`（法律 300-500，技术可到 800） |
| Embedding | BGE-small-zh（中文轻量）/ all-MiniLM（英文） |
| 向量库 | Chroma（内嵌零运维）/ FAISS / Qdrant |
| 本地 LLM | Ollama + Qwen2.5:7b 或 DeepSeek-R1:7b |
| 编排框架 | LangGraph（做"老师 Agent"多步逻辑比纯 LangChain 顺） |
| 前端 | Streamlit / Gradio 半小时出界面 |

最小可运行参考 51CTO 那篇《从零搭建离线私有知识库》[citation:2]，或掘金那篇《从零搭建通用 RAG 智能 AI 助教》——后者已经做了"三种问答模式 + 飞书机器人"，离你的"助教"诉求最近[citation:8]。

---

## 三、"老师/专家"角色怎么塑造（这部分比选框架更关键）

RAG 解决的是"答对"，解决不了"会教"。要让 AI 从"搜索引擎"变成"老师"，核心在三件事：

1. **System Prompt 人设定调**
   不要只写"你是一个助手"，要写："你是一位 XX 领域的资深导师，面对从入门到精通的学习者。回答时先判断对方水平，用苏格拉底式反问引导思考，而不是直接给结论。每讲完一个概念出一道小题。"

2. **课程大纲生成节点**
   让模型先基于知识库文献目录/章节，生成一份 `入门→进阶→精通` 的三级大纲（可存在 JSON 里当状态），后面每轮对话先判断"当前学到哪"。

3. **Agent 记忆 + 进度**
   - 短期：对话 BufferMemory
   - 长期：把"已学章节 / 测验分数"存 VectorStoreMemory 或 Neo4j 图库，下次对话能接着教
   - 进阶还可以加 **Yuxi-Know** 那种知识图谱层，把领域概念之间"前置/后置/并列"关系建模出来，老师就知道"这个概念不能现在讲，得先讲 A"[citation:3]

> ⚠️ 一个容易踩的坑：文献直接切片喂 RAG，模型答细节还行，**但"从入门到精通的路径规划"往往文献里没有现成答案**（每本书的目录是作者的路径，不一定是你的路径）。这部分要靠模型自身推理 + 你给的 System Prompt 约束，不要指望 RAG 召回能解决。

---

## 四、开源课程推荐（你自己想学这套技术的话）

按"从听懂到能造"排：

1. **《LangChain 从零开始学习课程（重构版）》**（Gitee）- 4 阶段 16 课 + 4 个实战项目，RAG 系统占第 8-10 课，Agent 占 11-14 课，结构最完整，中文[citation:5]
2. **`ai-agents-from-zero`**（GitHub /didilili）- 2026 最新版，覆盖 LangChain/LangGraph/Dify/Coze/RAG/Agent/微调，对标"大模型应用开发工程师"，体系化最强（含面试题库）[citation:12]
3. **`langchain-course`**（GitHub /Bhavik-Jikadara）- 轻量 Crash Course，5 模块（Chat/Prompt/Chains/RAG/Agents），每课带 `.py` 可跑，适合周末速通
4. **DeepLearning.AI** 的 "Building Agentic RAG" / "LangChain for LLM App Dev"（英文，但质量稳）

---

## 给你的落地建议（最小可行顺序）

> 第一步别想太远，**先用 AnythingLLM + Ollama(qwen:7b) 把文献喂进去跑通问答**，确认切片召回质量——这一步 30 分钟。
>
> 第二步，如果问答 OK 但"老师感"不够，迁到 **Dify**，在工作流里加"诊断→大纲→讲授→出题"几个节点，System Prompt 把人设写好。
>
> 第三步，想要"学习进度记忆 + 多概念依赖图"，再上 **LangGraph + 自研**，或者试 Yuxi-Know 那种带知识图谱的架子[citation:3]。

你现在的文献大概是什么量级（几十篇 / 上百篇？PDF 居多还是 MD？）和领域（偏理工还是社科）？这两个会影响切片策略和大模型选型，告诉我可以再细化。