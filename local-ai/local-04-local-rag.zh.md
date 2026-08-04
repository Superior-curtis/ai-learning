# 打造一套完全私密的本地 RAG 流程

> 📅 2026-08-01 · 使用教學
> 用本地嵌入模型 + 向量資料庫 + 本地 LLM，組合出隱私、成本都自己掌握的 RAG 流程。

---

前三篇我們學會了跑模型，也搞懂了硬體。這一系列的最後一篇，把這些技能串成一個真正的應用：**本地的 RAG**。讓 LLM 讀你自己的文件，而且資料永遠不離開你的電腦。

***

## 為什麼要本地 RAG

RAG（Retrieval-Augmented Generation，檢索增強生成）讓模型在回答前，先從你的文件庫中檢索相關片段，再據此生成答案。這是讓 LLM「懂你的文件」的標準做法。

用雲端 API 做 RAG，你的文件會先傳給第三方。有些文件（醫療、財務、公司內部資料）根本不適合離開自己的機器。

**本地 RAG 的優點：**

* 隱私：文件與查詢都不出機器
* 成本：除了硬體與電費，沒有每次呼叫的費用
* 掌控：模型、嵌入、資料庫全部自管
* 離線：沒有網路也能用

**代價：** 效果通常比頂級雲端模型略遜，且你需要自己維護流程。

***

## 三個零件

一個 RAG 流程由三塊組成：**嵌入模型**把文字轉成向量、**向量資料庫**做相似度搜尋、**LLM** 根據檢索到的內容生成回答。三塊我們都用本地方案：

| 零件 | 本地方案 |
|------|---------|
| 嵌入模型 | `nomic-embed-text`（透過 Ollama） |
| 向量資料庫 | Chroma（本機 PersistentClient） |
| LLM | `qwen2.5:7b`（透過 Ollama） |

***

## 安裝與拉取模型

```bash
pip install chromadb ollama
```

```bash
ollama pull nomic-embed-text
ollama pull qwen2.5:7b
```

`nomic-embed-text` 是 Ollama 提供的嵌入模型，約 274MB；`qwen2.5:7b` 是生成模型。如果你的機器記憶體較小，可以改用 `qwen2.5:3b`。

***

## 步驟一：切分文件

RAG 的第一步是把文件切成「可檢索的片段」。片段太小會失去上下文，太大則檢索精準度下降，所以一般會設定**片段大小與重疊**。

```python
def chunk_text(text: str, chunk_size: int = 500, overlap: int = 100) -> list[str]:
    words = text.split()
    chunks = []
    step = chunk_size - overlap
    for i in range(0, len(words), step):
        chunk = " ".join(words[i : i + chunk_size])
        chunks.append(chunk)
    return chunks
```

### 切分的原則

* 一個片段約 300-800 字（或 400-1000 token）
* 片段之間留 10-20% 的重疊，避免切斷關鍵句子

***

## 步驟二：嵌入與索引

把每個片段用嵌入模型轉成向量，再存入向量資料庫。

```python
import ollama
import chromadb

client = chromadb.PersistentClient(path="./notes_db")
collection = client.get_or_create_collection(name="notes")

docs = [
    "公司的定價方案分為免費版與 Pro 版，Pro 版每月 9 美元。",
    "退款政策：付費訂閱可在 14 天內無條件退款。",
    # ... 你的文件片段
]

ids = [f"note-{i}" for i in range(len(docs))]
embeddings = [
    ollama.embed(model="nomic-embed-text", input=d)["embeddings"][0]
    for d in docs
]

collection.add(ids=ids, documents=docs, embeddings=embeddings)
```

`PersistentClient` 會把資料存到本機的 `./notes_db` 目錄，關掉程式資料還在。

***

## 步驟三：查詢與回答

查詢時，把問題也嵌入成向量，在資料庫裡找最接近的片段，再把這些片段連同問題一起丟給 LLM。

```python
def ask(question: str) -> str:
    q_emb = ollama.embed(model="nomic-embed-text", input=question)["embeddings"][0]

    hits = collection.query(query_embeddings=[q_emb], n_results=3)
    context = "\n\n".join(hits["documents"][0])

    prompt = f"""你是一個私密的個人助理。請只用下面提供的資料回答問題。

資料：
{context}

問題：{question}
答案："""

    response = ollama.chat(
        model="qwen2.5:7b",
        messages=[{"role": "user", "content": prompt}],
    )
    return response["message"]["content"]
```

### 為什麼要「只用資料回答」

這一段提示詞非常重要。它要求模型不要憑記憶亂答，把輸出限制在檢索到的內容上，這是 RAG 減少幻覺的關鍵。

***

## 完整可執行的流程

把前面幾段組起來，就是一個完整的本地 RAG：

```python
import ollama
import chromadb

def chunk_text(text: str, chunk_size: int = 500, overlap: int = 100):
    words = text.split()
    step = chunk_size - overlap
    return [" ".join(words[i : i + chunk_size]) for i in range(0, len(words), step)]

def build_index(file_path: str, db_path: str = "./notes_db", name: str = "notes"):
    client = chromadb.PersistentClient(path=db_path)
    collection = client.get_or_create_collection(name=name)

    raw = open(file_path, encoding="utf-8").read()
    chunks = chunk_text(raw)

    ids = [f"{file_path}-{i}" for i in range(len(chunks))]
    embeddings = [
        ollama.embed(model="nomic-embed-text", input=c)["embeddings"][0]
        for c in chunks
    ]
    collection.add(ids=ids, documents=chunks, embeddings=embeddings)
    return collection

def ask(question: str, collection) -> str:
    q_emb = ollama.embed(model="nomic-embed-text", input=question)["embeddings"][0]
    hits = collection.query(query_embeddings=[q_emb], n_results=3)
    context = "\n\n".join(hits["documents"][0])

    prompt = f"請只用下面的資料回答問題。\n\n資料：\n{context}\n\n問題：{question}"
    response = ollama.chat(
        model="qwen2.5:7b",
        messages=[{"role": "user", "content": prompt}],
    )
    return response["message"]["content"]

# 第一次建立索引
collection = build_index("my_notes.txt")

# 之後反覆查詢
print(ask("我們的退款政策是什麼？", collection))
```

執行前記得先啟動 Ollama（或先跑一次 `ollama run` 讓它在背景啟動）。

***

## 隱私與成本：算給你看

| 項目 | 雲端 RAG | 本地 RAG |
|------|---------|---------|
| 資料離開你的機器 | 是 | 否 |
| 每次查詢費用 | 依 token 計費 | 0 |
| 硬體成本 | 0 | 一次性的機器支出 |
| 電費 | 0 | 少量 |
| 隱私風險 | 依供應商 | 由你掌控 |

本地 RAG 的邊際成本接近零：一旦模型下載好，跑一萬次查詢也不會多花一分錢。

***

## 延伸方向

這套流程可以往幾個方向升級：

* **更好的嵌入模型**：`bge-m3` 支援多語言，中文效果更好
* **更大的向量庫**：切換到 Qdrant 或 LanceDB，支援百萬級向量
* **多文件格式**：加入 PDF / DOCX 的解析（如 `pypdf`）

***

## 總結

* RAG = 嵌入模型 + 向量資料庫 + LLM，三塊都能本地跑
* 切分 → 嵌入 → 檢索 → 生成，是四個固定步驟
* 本地 RAG 的關鍵賣點是隱私與零邊際成本
* 先跑通這條最小流程，再按需求逐步升級
