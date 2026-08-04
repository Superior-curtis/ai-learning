# AI Learning Hub — 一本 AI 產業的書

> AI moves fast. This is a **bilingual (中文 / English) book** that grows into a reference for the whole AI industry — from how models guess the next word, to building agents, to the economics of the field.

## 目錄 / Table of Contents

| Part | Title | 篇數 |
|---|---|---|
| **00** | **Foundations** / 導論 | 1 |
| **01** | **How Models Work** / 模型原理 | 20 |
| **02** | **Building Applications** / 建構應用 | 15 |
| **03** | **Tools & Platforms** / 工具與平台 | 55 |
| **04** | **Industry & Business** / 產業與商業 | 3 |
| **05** | **Safety & Society** / 安全與社會 | 4 |

## 00 · Foundations / 導論 (1 篇)

> What AI is, where it came from, why now.

### AI Foundations / AI 導論  (1 篇)

> From zero: what AI is, where it came from, why now.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [什麼是 AI：一份白話地圖](intro/intro-01-what-is-ai.zh.md) | [What Is AI: A Plain-English Map](intro/intro-01-what-is-ai.en.md) | 2026-08-04 |

## 01 · How Models Work / 模型原理 (20 篇)

> From next-token prediction to training and model families.

### LLM Core / LLM 基礎  (5 篇)

> How models predict the next word: tokens, context, sampling, hallucination.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [預測下一個詞：LLM 的核心把戲](llm-core/llmcore-01-next-token.zh.md) | [Predicting the Next Word: The Core Trick](llm-core/llmcore-01-next-token.en.md) | 2026-08-04 |
| 2 | [Token：語言不是以「字」為單位](llm-core/llmcore-02-tokens.zh.md) | [Tokens: Language Isn't Processed as Words](llm-core/llmcore-02-tokens.en.md) | 2026-08-04 |
| 3 | [上下文窗口：模型能看到多遠](llm-core/llmcore-03-context.zh.md) | [The Context Window: How Far the Model Can See](llm-core/llmcore-03-context.en.md) | 2026-08-04 |
| 4 | [抽樣與溫度：怎麼「猜」是可以調的](llm-core/llmcore-04-sampling.zh.md) | [Sampling & Temperature: How the “Guessing” Can Be Tuned](llm-core/llmcore-04-sampling.en.md) | 2026-08-04 |
| 5 | [幻覺：當預測失誤時](llm-core/llmcore-05-hallucination.zh.md) | [Hallucination: When the Prediction Goes Wrong](llm-core/llmcore-05-hallucination.en.md) | 2026-08-04 |

### LLM Internals / LLM 原理  (4 篇)

> Attention, RAG, embeddings, quantization — how models work.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [注意力機制：模型如何決定「誰該聽誰」](llm-internals/llm-01-attention.zh.md) | [Attention: How Models Decide Who Listens to Whom](llm-internals/llm-01-attention.en.md) | 2026-08-01 |
| 2 | [嵌入與向量搜尋：把語意放進座標空間](llm-internals/llm-02-embeddings.zh.md) | [Embeddings and Vector Search: Putting Semantics into Coordinates](llm-internals/llm-02-embeddings.en.md) | 2026-08-01 |
| 3 | [RAG 從第一性原理講起：檢索增強生成](llm-internals/llm-03-rag.zh.md) | [RAG from First Principles: Retrieval-Augmented Generation](llm-internals/llm-03-rag.en.md) | 2026-08-01 |
| 4 | [量化：用更少的記憶體跑模型](llm-internals/llm-04-quantization.zh.md) | [Quantization: Running Models on Less Memory](llm-internals/llm-04-quantization.en.md) | 2026-08-01 |

### Training / 模型訓練  (5 篇)

> Pre-training, fine-tuning, RLHF — how models are taught.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [預訓練：讀完整個網際網路](training/train-01-pretraining.zh.md) | [Pre-training: Reading the Entire Internet](training/train-01-pretraining.en.md) | 2026-08-04 |
| 2 | [微調：教它一門專長](training/train-02-finetuning.zh.md) | [Fine-tuning: Teaching It a Specialty](training/train-02-finetuning.en.md) | 2026-08-04 |
| 3 | [RLHF：讓模型符合人類偏好](training/train-03-rlhf.zh.md) | [RLHF: Making Models Match Human Preferences](training/train-03-rlhf.en.md) | 2026-08-04 |
| 4 | [訓練資料：模型吃了什麼](training/train-04-corpus.zh.md) | [The Training Corpus: What the Model Ate](training/train-04-corpus.en.md) | 2026-08-04 |
| 5 | [規模法則：為什麼越大越不同](training/train-05-scaling-laws.zh.md) | [Scaling Laws: Why Bigger Is Different, Not Just Bigger](training/train-05-scaling-laws.en.md) | 2026-08-04 |

### Model Families / 模型家族  (6 篇)

> GPT, Claude, Llama… open vs closed, multimodal, reasoning.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [開放 vs 封閉模型](model-families/models-01-open-vs-closed.zh.md) | [Open vs. Closed Models](model-families/models-01-open-vs-closed.en.md) | 2026-08-04 |
| 2 | [主要模型家族導覽：GPT、Claude、Llama…](model-families/models-02-families-tour.zh.md) | [A Tour of the Major Model Families](model-families/models-02-families-tour.en.md) | 2026-08-04 |
| 3 | [多模態：當模型張開了眼睛](model-families/models-03-multimodal.zh.md) | [Multimodal Models: When Models Open Their Eyes](model-families/models-03-multimodal.en.md) | 2026-08-04 |
| 4 | [推理模型：答案之前的沉思](model-families/models-04-reasoning.zh.md) | [Reasoning Models: The Pause Before the Answer](model-families/models-04-reasoning.en.md) | 2026-08-04 |
| 5 | [蒸餾與小型模型：把大象裝進手機](model-families/models-05-small-models.zh.md) | [Distillation and Small Models: Fitting an Elephant into a Phone](model-families/models-05-small-models.en.md) | 2026-08-04 |
| 6 | [評估與基準：怎麼量『模型多強』](model-families/models-06-evaluation.zh.md) | [Evaluation and Benchmarks: Measuring How Strong a Model Is](model-families/models-06-evaluation.en.md) | 2026-08-04 |

## 02 · Building Applications / 建構應用 (15 篇)

> Prompting, RAG, fine-tuning, agents, and MCP.

### Prompting / 提示工程  (4 篇)

> Practical techniques for talking to models.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [提示詞的基礎：把話講清楚](prompting/prompt-01-basics.zh.md) | [Prompting Basics: Say What You Mean](prompting/prompt-01-basics.en.md) | 2026-08-04 |
| 2 | [角色、例子與輸出格式：把結構交給模型](prompting/prompt-02-structure.zh.md) | [Roles, Examples, and Output Formats: Structuring the Prompt](prompting/prompt-02-structure.en.md) | 2026-08-04 |
| 3 | [鏈式思考：讓模型一步一步想](prompting/prompt-03-chain-of-thought.zh.md) | [Chain-of-Thought: Reason Step by Step](prompting/prompt-03-chain-of-thought.en.md) | 2026-08-04 |
| 4 | [系統提示與強韌性：讓提示經得起考驗](prompting/prompt-04-robustness.zh.md) | [System Prompts and Robustness: Prompts That Hold Up](prompting/prompt-04-robustness.en.md) | 2026-08-04 |

### RAG / RAG 檢索  (4 篇)

> Retrieval, chunking, hybrid search, reranking.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [為什麼要 RAG：別讓模型靠記憶瞎掰](rag/rag-01-what-is-rag.zh.md) | [Why RAG: Stop Making the Model Answer from Memory](rag/rag-01-what-is-rag.en.md) | 2026-08-04 |
| 2 | [切塊與索引：把文件拆成能撿的單位](rag/rag-02-chunking.zh.md) | [Chunking & Indexing: Turning Documents into Grab-able Units](rag/rag-02-chunking.en.md) | 2026-08-04 |
| 3 | [混合檢索與重排：向量不是唯一的排序標準](rag/rag-03-hybrid.zh.md) | [Hybrid Retrieval & Reranking: Vectors Are Not the Only Ranking Signal](rag/rag-03-hybrid.en.md) | 2026-08-04 |
| 4 | [評估 RAG 系統：把「感覺還行」變成數字](rag/rag-04-evaluating.zh.md) | [Evaluating RAG: Turning 'Seems Fine' into Numbers](rag/rag-04-evaluating.en.md) | 2026-08-04 |

### Fine-tuning / 微調實務  (3 篇)

> When to fine-tune and how LoRA keeps it cheap.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [微調 vs RAG：怎麼選](fine-tuning/finetune-01-finetune-vs-rag.zh.md) | [Fine-tuning vs RAG: How to Choose](fine-tuning/finetune-01-finetune-vs-rag.en.md) | 2026-08-04 |
| 2 | [LoRA 實務：把微調變便宜](fine-tuning/finetune-02-lora.zh.md) | [LoRA in Practice: Making Fine-tuning Cheap](fine-tuning/finetune-02-lora.en.md) | 2026-08-04 |
| 3 | [資料準備與評估：微調的勝負手](fine-tuning/finetune-03-data-and-eval.zh.md) | [Data & Evaluation: What Decides a Fine-tune](fine-tuning/finetune-03-data-and-eval.en.md) | 2026-08-04 |

### AI Agents / AI 代理  (4 篇)

> LangGraph, CrewAI, sub-agents, and orchestration.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [什麼是 AI Agent？從定義到代理迴圈](agents/agents-01-what-is-agent.zh.md) | [What Is an AI Agent? From Definition to the Agent Loop](agents/agents-01-what-is-agent.en.md) | 2026-08-01 |
| 2 | [Agent 編排：單一代理 vs 多代理](agents/agents-02-orchestration.zh.md) | [Agent Orchestration: Single vs Multi-Agent](agents/agents-02-orchestration.en.md) | 2026-08-01 |
| 3 | [MCP 與 Agent 生態系：工具標準化的關鍵](agents/agents-03-mcp.zh.md) | [MCP and the Agent Ecosystem: The Key to Standardizing Tools](agents/agents-03-mcp.en.md) | 2026-08-01 |
| 4 | [動手打造你的第一個 Agent：逐步教學](agents/agents-04-first-agent.zh.md) | [Build Your First Agent, Step by Step](agents/agents-04-first-agent.en.md) | 2026-08-01 |

## 03 · Tools & Platforms / 工具與平台 (55 篇)

> Claude Code, local AI, model providers, and vector DBs.

### Claude Code / Claude Code  (51 篇)

> Anthropic's terminal AI agent: architecture, tools, and practice.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [從這裡開始：Claude Code 學習路徑](claude-code/claude-code-00-start-here.zh.md) | [Start Here: The Claude Code Learning Path](claude-code/claude-code-00-start-here.en.md) | 2026-07-28 |
| 2 | [Claude Code 快速上手](claude-code/claude-code-01-quickstart.zh.md) | [Claude Code Quickstart](claude-code/claude-code-01-quickstart.en.md) | 2026-07-27 |
| 3 | [CLAUDE.md：讓 Claude Code 真正理解你的專案](claude-code/claude-code-02-claudemd.zh.md) | [CLAUDE.md: Make Claude Code Truly Understand Your Project](claude-code/claude-code-02-claudemd.en.md) | 2026-07-27 |
| 4 | [Claude Code 核心概念一覽](claude-code/claude-code-03-core-concepts.zh.md) | [Claude Code Core Concepts at a Glance](claude-code/claude-code-03-core-concepts.en.md) | 2026-07-27 |
| 5 | [Claude Code 原始碼架構總覽](claude-code/claude-code-04-architecture.zh.md) | [Claude Code Source Architecture Overview](claude-code/claude-code-04-architecture.en.md) | 2026-07-27 |
| 6 | [Tool 工具系統：Claude Code 的執行力來自工具，不只來自模型](claude-code/claude-code-05-tools.zh.md) | [The Tool System: Claude Code's Execution Power Comes from Tools, Not Just the Model](claude-code/claude-code-05-tools.en.md) | 2026-07-27 |
| 7 | [Slash Commands 命令系統：使用者如何直接控制 Claude Code](claude-code/claude-code-06-commands.zh.md) | [Slash Commands: How Users Directly Control Claude Code](claude-code/claude-code-06-commands.en.md) | 2026-07-27 |
| 8 | [上下文系統：Git、CLAUDE.md 與系統提示詞注入](claude-code/claude-code-07-context.zh.md) | [The Context System: Git, CLAUDE.md, and System Prompt Injection](claude-code/claude-code-07-context.en.md) | 2026-07-27 |
| 9 | [上下文壓縮管理：Claude Code 的六層分級壓縮](claude-code/claude-code-08-compression.zh.md) | [Context Compression Management: Claude Code's Six-Tier Compression](claude-code/claude-code-08-compression.en.md) | 2026-07-27 |
| 10 | [核心循環解析：QueryEngine 如何驅動一次任務](claude-code/claude-code-09-engine.zh.md) | [Inside the Core Loop: How QueryEngine Drives a Task](claude-code/claude-code-09-engine.en.md) | 2026-07-27 |
| 11 | [Claude Code 的提示詞工程：六層分級提示系統](claude-code/claude-code-10-prompt.zh.md) | [Claude Code's Prompt Engineering: A Six-Tier Prompt System](claude-code/claude-code-10-prompt.en.md) | 2026-07-27 |
| 12 | [檔案讀寫與編輯鏈路：Claude Code 的基本盤](claude-code/claude-code-11-files.zh.md) | [The File Read/Write and Edit Pipeline: Claude Code's Foundation](claude-code/claude-code-11-files.en.md) | 2026-07-27 |
| 13 | [Bash 工具為什麼這麼關鍵：開發環境操作總線](claude-code/claude-code-12-bash.zh.md) | [Why the Bash Tool Matters: The Development Environment Operation Bus](claude-code/claude-code-12-bash.en.md) | 2026-07-27 |
| 14 | [狀態管理解析：AppStateStore 裡到底保存了什麼？](claude-code/claude-code-13-state.zh.md) | [State Management Decoded: What Does AppStateStore Actually Hold?](claude-code/claude-code-13-state.en.md) | 2026-07-27 |
| 15 | [權限與安全機制：為什麼 Claude Code 不會無腦亂跑](claude-code/claude-code-14-security.zh.md) | [Permissions and Security: Why Claude Code Doesn't Run Wild](claude-code/claude-code-14-security.en.md) | 2026-07-27 |
| 16 | [Plan Mode 在架構裡的位置：不只是模式切換](claude-code/claude-code-15-planmode.zh.md) | [Plan Mode's Place in the Architecture: More Than a Mode Toggle](claude-code/claude-code-15-planmode.en.md) | 2026-07-27 |
| 17 | [從原始碼看 Claude Code 的產品邊界與局限](claude-code/claude-code-16-boundaries.zh.md) | [Claude Code's Product Boundaries and Limitations, Seen from the Source Code](claude-code/claude-code-16-boundaries.en.md) | 2026-07-27 |
| 18 | [MCP 與 LSP 整合：把外部世界接進 Claude Code](claude-code/claude-code-17-mcp.zh.md) | [MCP and LSP Integration: Wiring the Outside World Into Claude Code](claude-code/claude-code-17-mcp.en.md) | 2026-07-27 |
| 19 | [插件、Skills 與 Agent：Claude Code 如何走向平台化](claude-code/claude-code-18-platform.zh.md) | [Plugins, Skills, and Agents: How Claude Code Is Moving Toward a Platform](claude-code/claude-code-18-platform.en.md) | 2026-07-27 |
| 20 | [Claude Code 的 Skills 系統：不只是 Prompt](claude-code/claude-code-19-skills.zh.md) | [Claude Code's Skills System: More Than Just a Prompt](claude-code/claude-code-19-skills.en.md) | 2026-07-27 |
| 21 | [多 Agent 與子任務機制：從單執行緒到協作系統](claude-code/claude-code-20-multiagent.zh.md) | [Multi-Agent and Sub-Task Mechanisms: From a Single Thread to a Collaborative System](claude-code/claude-code-20-multiagent.en.md) | 2026-07-27 |
| 22 | [遠端會話與橋接能力：終端在本地，不代表執行一定在本地](claude-code/claude-code-21-remote.zh.md) | [Remote Sessions and Bridge Capabilities: A Local Terminal Doesn't Mean Local Execution](claude-code/claude-code-21-remote.en.md) | 2026-07-27 |
| 23 | [AgentTool：子 Agent 調度器](claude-code/claude-code-22-agenttool.zh.md) | [AgentTool: The Sub-Agent Dispatcher](claude-code/claude-code-22-agenttool.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 24 | [BashTool：Shell 執行器深入解析](claude-code/claude-code-23-bashtool.zh.md) | [BashTool: A Deep Dive into the Shell Executor](claude-code/claude-code-23-bashtool.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 25 | [FileReadTool：讀取檔案深入解析](claude-code/claude-code-24-fileread.zh.md) | [FileReadTool: A Deep Dive into Reading Files](claude-code/claude-code-24-fileread.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 26 | [FileEditTool：編輯檔案深入解析](claude-code/claude-code-25-fileedit.zh.md) | [FileEditTool: Editing Files in Depth](claude-code/claude-code-25-fileedit.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 27 | [FileWriteTool：寫入檔案深入解析](claude-code/claude-code-26-filewrite.zh.md) | [FileWriteTool: Writing Files in Depth](claude-code/claude-code-26-filewrite.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 28 | [GlobTool：查找檔案](claude-code/claude-code-27-globtool.zh.md) | [GlobTool: Finding Files](claude-code/claude-code-27-globtool.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 29 | [GrepTool：搜尋內容](claude-code/claude-code-28-greptool.zh.md) | [GrepTool: Searching Content](claude-code/claude-code-28-greptool.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 30 | [NotebookEditTool：編輯 Notebook](claude-code/claude-code-29-notebookedit.zh.md) | [NotebookEditTool: Editing Notebooks](claude-code/claude-code-29-notebookedit.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 31 | [WebFetchTool：抓取網頁](claude-code/claude-code-30-webfetch.zh.md) | [WebFetchTool: Fetching Web Pages](claude-code/claude-code-30-webfetch.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 32 | [WebSearchTool：聯網搜尋](claude-code/claude-code-31-websearch.zh.md) | [WebSearchTool: Web Search](claude-code/claude-code-31-websearch.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 33 | [TodoWriteTool：待辦清單](claude-code/claude-code-32-todowrite.zh.md) | [TodoWriteTool: Todo List](claude-code/claude-code-32-todowrite.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 34 | [AskUserQuestionTool：向使用者提問](claude-code/claude-code-33-askuser.zh.md) | [AskUserQuestionTool: Asking the User](claude-code/claude-code-33-askuser.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 35 | [SkillTool：執行 Skills](claude-code/claude-code-34-skilltool.zh.md) | [SkillTool: Running Skills](claude-code/claude-code-34-skilltool.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 36 | [EnterPlanModeTool：進入 Plan Mode](claude-code/claude-code-35-enterplan.zh.md) | [EnterPlanModeTool: Entering Plan Mode](claude-code/claude-code-35-enterplan.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 37 | [ExitPlanModeTool：退出 Plan Mode](claude-code/claude-code-36-exitplan.zh.md) | [ExitPlanModeTool: Exiting Plan Mode](claude-code/claude-code-36-exitplan.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 38 | [TaskCreateTool：建立任務](claude-code/claude-code-37-taskcreate.zh.md) | [TaskCreateTool: Creating Tasks](claude-code/claude-code-37-taskcreate.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 39 | [TaskGetTool：讀取任務](claude-code/claude-code-38-taskget.zh.md) | [TaskGetTool: Reading Tasks](claude-code/claude-code-38-taskget.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 40 | [TaskUpdateTool：更新任務](claude-code/claude-code-39-taskupdate.zh.md) | [TaskUpdateTool: Updating Tasks](claude-code/claude-code-39-taskupdate.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 41 | [TaskListTool：列出任務](claude-code/claude-code-40-tasklist.zh.md) | [TaskListTool: Listing Tasks](claude-code/claude-code-40-tasklist.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 42 | [TaskStopTool：停止任務](claude-code/claude-code-41-taskstop.zh.md) | [TaskStopTool: Stopping Tasks](claude-code/claude-code-41-taskstop.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 43 | [TaskOutputTool：讀取任務輸出](claude-code/claude-code-42-taskoutput.zh.md) | [TaskOutputTool: Reading Task Output](claude-code/claude-code-42-taskoutput.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 44 | [SendMessageTool：Agent 通訊](claude-code/claude-code-43-sendmessage.zh.md) | [SendMessageTool: Agent Communication](claude-code/claude-code-43-sendmessage.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 45 | [ListMcpResourcesTool：列出 MCP 資源](claude-code/claude-code-44-listmcp.zh.md) | [ListMcpResourcesTool: Listing MCP Resources](claude-code/claude-code-44-listmcp.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 46 | [ReadMcpResourceTool：讀取 MCP 資源](claude-code/claude-code-45-readmcp.zh.md) | [ReadMcpResourceTool: Reading MCP Resources](claude-code/claude-code-45-readmcp.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 47 | [LSPTool：語言服務接入](claude-code/claude-code-46-lsptool.zh.md) | [LSPTool: Language Service Integration](claude-code/claude-code-46-lsptool.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 48 | [自己做一個 Claude Code 需要哪些模組](claude-code/claude-code-47-buildyourown.zh.md) | [What Modules You Need to Build Your Own Claude Code](claude-code/claude-code-47-buildyourown.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 49 | [Claude Code 的 Skills 系統：不只是 Prompt](claude-code/claude-code-48-skills-system.zh.md) | [Claude Code's Skills System: More Than Just a Prompt](claude-code/claude-code-48-skills-system.en.md) | Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) |
| 50 | [一次任務的生命週期：Claude Code 代理迴圈逐步拆解](claude-code/claude-code-49-agent-loop.zh.md) | [Life of a Task: The Claude Code Agent Loop, Step by Step](claude-code/claude-code-49-agent-loop.en.md) | 2026-07-28 |
| 51 | [Claude Code 完全入門指南](claude-code/claude-code-guide.zh.md) | [Claude Code Complete Beginner's Guide](claude-code/claude-code-guide.en.md) | 2026-07-27 |

### Local AI / 本地 AI  (4 篇)

> Ollama, llama.cpp, self-hosted models, offline inference.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [用 Ollama 跑起你的第一個本地模型](local-ai/local-01-ollama.zh.md) | [Run Your First Local Model with Ollama](local-ai/local-01-ollama.en.md) | 2026-08-01 |
| 2 | [llama.cpp 與 GGUF 解析](local-ai/local-02-llamacpp.zh.md) | [llama.cpp and GGUF, Explained](local-ai/local-02-llamacpp.en.md) | 2026-08-01 |
| 3 | [Apple Silicon 與 NVIDIA：在你的硬體上跑模型](local-ai/local-03-mac-vs-gpu.zh.md) | [Apple Silicon vs NVIDIA: Running Models on Your Hardware](local-ai/local-03-mac-vs-gpu.en.md) | 2026-08-01 |
| 4 | [打造一套完全私密的本地 RAG 流程](local-ai/local-04-local-rag.zh.md) | [Build a Private Local RAG Stack](local-ai/local-04-local-rag.en.md) | 2026-08-01 |

## 04 · Industry & Business / 產業與商業 (3 篇)

> Landscape, economics, startups, and AI & work.

### Industry / 產業地圖  (3 篇)

> The AI value chain, players, and business models.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [AI 產業價值鏈](industry/industry-01-value-chain.zh.md) | [The AI Industry Value Chain](industry/industry-01-value-chain.en.md) | 2026-08-04 |
| 2 | [主要玩家地圖](industry/industry-02-players.zh.md) | [The Player Map](industry/industry-02-players.en.md) | 2026-08-04 |
| 3 | [商業模式](industry/industry-03-business-models.zh.md) | [Business Models](industry/industry-03-business-models.en.md) | 2026-08-04 |

## 05 · Safety & Society / 安全與社會 (4 篇)

> Security, alignment, policy, and ethics.

### AI Security / AI 安全  (4 篇)

> Prompt injection, jailbreaks, evals, and the model attack surface.

| # | 文章 (中文) | English | 日期 |
|---|---|---|---|
| 1 | [提示注入（Prompt Injection）深入解析](ai-security/security-01-prompt-injection.zh.md) | [Prompt Injection, Explained](ai-security/security-01-prompt-injection.en.md) | 2026-08-01 |
| 2 | [越獄（Jailbreak）與它是如何運作的](ai-security/security-02-jailbreaks.zh.md) | [Jailbreaks and How They Work](ai-security/security-02-jailbreaks.en.md) | 2026-08-01 |
| 3 | [LLM 應用程式的紅隊與評測（Evals）](ai-security/security-03-evals.zh.md) | [Red-Teaming and Evals for LLM Apps](ai-security/security-03-evals.en.md) | 2026-08-01 |
| 4 | [強化你的 LLM 生產應用程式](ai-security/security-04-secure-apps.zh.md) | [Hardening a Production LLM Application](ai-security/security-04-secure-apps.en.md) | 2026-08-01 |

## 📬 Contribute

Articles are in Markdown (converted from the interactive MDX originals on the website). Feel free to open issues or PRs for corrections or new topics.

---

_Built with ❤️ — a bilingual book on the AI industry, from the [myself](https://blog-916bd.web.app) website._
