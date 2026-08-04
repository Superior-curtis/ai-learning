# Agent 編排：單一代理 vs 多代理

> 📅 2026-08-01 · 核心機制
> 從序列、階層到群集，認識三種主流的 Agent 編排模式，以及何時該把工作拆給子代理。

---

## 為什麼需要編排

單一 Agent 能完成很多事，但有些任務天生需要分工：研究要查資料、撰稿要把資料寫成文章、審查要把文章抓漏。與其把所有能力塞進一個 Agent，不如**用多個專精的 Agent 協作**——而「如何把他們組織起來」就是編排（orchestration）。

編排要回答三個問題：

* 誰先做、誰後做？（流程順序）
* 誰負責指揮、誰負責執行？（權責）
* 結果怎麼傳遞給下一步？（交接）

***

## 三種基本架構

### 1. 序列式（Sequential）

Agent 一個接一個執行，前一個的輸出是後一個的輸入。

```
研究員 → 撰稿人 → 審查員
```

適合流程固定、步驟清楚的任務，例如「先查資料 → 再寫稿 → 最後校對」。優點是簡單可預測；缺點是沒有彈性，任何一步失敗就卡住。

### 2. 階層式（Hierarchical）

有一個「協調者」（coordinator）負責拆解任務、分派給子代理，再彙整結果。

```
              協調者
              /     \
        研究員        撰稿人
             \     /
              彙整
```

適合任務會變化、需要動態判斷「誰來做」的場景。協調者決定分工，子代理專注執行。

### 3. 群集式（Swarm）

一群對等的 Agent，透過「交接」（handoff）把工作輪流丟給彼此，沒有固定的指揮者。適合像是客服這種角色會動態切換的場景。

```
  A ──handoff──▶ B ──handoff──▶ C
  ▲                             │
  └─────────handoff─────────────┘
```

> 群集式最有彈性，但也最難除錯：工作流向是動態的，不容易預測與重現。

***

## 用框架實作：三個例子

### LangGraph：把流程當成圖

LangGraph 把編排建模成一個**狀態圖**：節點是處理步驟，邊是流程，狀態在節點之間流動。

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    query: str
    draft: str
    review: str

def draft(state: State) -> dict:
    return {"draft": f"草稿：{state['query']}"}

def review(state: State) -> dict:
    return {"review": f"審查通過：{state['draft']}"}

graph = StateGraph(State)
graph.add_node("draft", draft)
graph.add_node("review", review)
graph.add_edge(START, "draft")
graph.add_edge("draft", "review")
graph.add_edge("review", END)

app = graph.compile()
result = app.invoke({"query": "寫一份產品新聞稿"})
```

### CrewAI：把任務當成團隊

CrewAI 用「角色 + 任務 + 流程」的模型：每個 Agent 有角色，每個 Task 指定由誰負責，Crew 決定流程。

```python
from crewai import Agent, Task, Crew, Process

researcher = Agent(
    role="研究員",
    goal="蒐集準確的市場資料",
    backstory="專注於數據與事實",
)

writer = Agent(
    role="撰稿人",
    goal="把研究轉成可讀的文章",
    backstory="擅長清晰表達",
)

research_task = Task(
    description="整理 2026 年 AI Agent 市場規模",
    expected_output="一份含資料來源的摘要",
    agent=researcher,
)

write_task = Task(
    description="把研究摘要寫成 800 字文章",
    expected_output="一篇完整的文章",
    agent=writer,
)

crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential,  # 或 Process.hierarchical
)

result = crew.kickoff()
```

### AutoGen：用團隊對話協作

AutoGen 的現代版 `autogen-agentchat` 把多 Agent 變成一個「團隊」，讓成員輪流發言，直到任務完成。

```python
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(
    model="gpt-4o",
    api_key="your-key",
)

writer = AssistantAgent(
    name="writer",
    system_message="你負責起草文章。",
    model_client=model_client,
)
reviewer = AssistantAgent(
    name="reviewer",
    system_message="你負責檢查文章並提出修改。",
    model_client=model_client,
)

team = RoundRobinGroupChat([writer, reviewer], max_turns=6)
await team.run(task="寫一篇關於 AI Agent 的短文")
```

***

## 什麼時候該用子代理？

子代理有成本：它要重新建立上下文、重新探索、再回報，最後主代理還要讀它的報告。只在以下情況使用：

* 工作真的可以**平行化**：例如一次檢查 20 個檔案，每個檔案獨立。
* 任務需要**不同的專業角色**：研究、撰寫、審查各自獨立。
* 主代理的上下文已接近上限：把部分工作外包給子代理，可以節省主代理的 token。

相反地，如果一件事主代理用幾個工具呼叫就能做完，就不要拆——多一層代理只會更慢、更貴。

***

## 常見陷阱

* **過度分工**：簡單任務也拆子代理，成本和延遲白白增加。
* **忽略交接**：子代理的結果沒有良好格式，主代理看不懂。
* **沒有上限**：子代理可以無限開下去，token 帳單也無限長。
* **共享狀態**：子代理之間沒有溝通管道，各自做出矛盾的決定。

***

## 重點回顧

* 編排回答三個問題：順序、權責、交接。
* 三種基本模式：序列式、階層式、群集式。
* LangGraph 用圖、CrewAI 用角色任務、AutoGen 用團隊對話。
* 子代理有成本，只在平行化、專業分工、或省上下文時使用。
* 先從單一 Agent 開始，需要時再拆。
