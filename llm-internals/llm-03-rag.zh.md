# RAG 從第一性原理講起：檢索增強生成

> 📅 2026-08-01 · 核心機制
> 從模型知識是「凍結」的這個事實出發，一步步拆解 chunking、索引、檢索、生成四段管線，以及最常見的坑。

---

RAG（Retrieval-Augmented Generation，檢索增強生成）是目前讓 LLM 回答「自己知識範圍外」問題的主流做法。這篇文章不從 API 講起，而是先回到第一性原理：模型到底為什麼會「不知道」，再把整個管線從零拼起來。

## 第一性原理：模型的知識是凍結的

大型語言模型的能力，來自訓練時看過的數十億筆資料。訓練一結束，模型對世界的了解就「凍結」了：

* **它不知道訓練截止日之後發生的事**（「去年六月的新聞？」不知道）。
* **它沒有看過你的私有文件**（你的公司的產品手冊、內部政策）。
* **它會「自信地編造」**——當它不知道答案時，沒有機制說「我不知道」，它傾向於給出聽起來合理的內容，也就是幻覺（hallucination）。

RAG 的應對非常直接：**不要讓模型憑記憶回答，先從外部資料庫把相關內容「撿」回來，塞進提示詞裡，讓模型「照著參考資料回答」。**

```
沒有 RAG：  問題 → LLM（憑記憶）→ 可能編造
有 RAG：    問題 → 檢索器（從你的資料庫找答案）
              ↓
        問題 + 相關段落 → LLM → 引用來源的回答
```

這等於給模型一本「可以翻閱的書」，而不是要求它背下整本書。

## 管線全貌：四段

RAG 系統有四段，前兩段是**離線準備**，後兩段是**線上服務**：

```
離線：   文件 → 切塊 → 嵌入 → 建索引
線上：   問題 → 嵌入 → 檢索 top-k → 拼接提示 → LLM 生成
```

### 離線：切塊（Chunking）

文件太長，無法整份嵌入成一個有用的向量，必須切成小塊：

* **切塊大小**：常見 300–800 tokens。太小則上下文不足，太大則語意被稀釋、且容易塞進不相關內容。
* **重疊（overlap）**：相鄰區塊重疊 10–20%，避免剛好把關鍵句切在邊界上。
* **切割策略**：先按結構切（標題、章節、段落），再按固定長度切，保留標題資訊會明顯提升檢索品質。
* **小技巧**：把 chunk 加上前後文（例如章節標題）再嵌入，稱為「contextual chunk」。

### 離線：嵌入與建索引

每個 chunk 用嵌入模型轉成向量，存進向量資料庫（見上一篇）。同時把每個 chunk 的原文和 metadata 存起來，檢索到向量後才能取出文字。

### 線上：檢索

使用者的問題也轉成向量，在索引中找最相近的 K 個 chunk（常見 K = 3–10）。好的檢索會回傳「問題需要的關鍵證據」；差的檢索回傳一堆相關但無用的段落。

### 線上：生成

把問題和檢索到的段落拼進提示詞，要求模型「只依據提供的資料回答，並標註來源」。

```python
system_prompt = (
    "You are a helpful assistant. Answer using ONLY the context below. "
    "If the context does not contain the answer, say you don't know. "
    "Cite the source of each claim.\n\n"
    "### CONTEXT ###\n{context}"
)

user_query = "Our refund policy: how many days, and any exceptions?"

# context = "\n\n".join(f"[doc {i}] {chunk.text}" for i, chunk in enumerate(top_k))
final_prompt = system_prompt.format(context=context) + f"\n\n### QUESTION ###\n{user_query}"
```

生成階段的三個要點：

* **角色設定要「強制」引用**：不寫「只依據提供的資料」，模型就可能摻入自己的記憶。
* **讓模型有說「不知道」的空間**：這大幅降低幻覺。
* **追蹤來源**：要求模型標註用了哪個文件，方便使用者查證。

## 完整管線（概念式）

以下是一個端到端的範例，採用概念式程式碼（實際 API 以你選的函式庫版本為準）：

```python
# pip install sentence-transformers chromadb
from sentence_transformers import SentenceTransformer
import chromadb

embedder = SentenceTransformer("BAAI/bge-small-en-v1.5")
client = chromadb.PersistentClient(path="./chroma")
collection = client.get_or_create_collection("docs")

# --- 離線：切塊 → 嵌入 → 建索引 ---
def chunk_document(text, size=400, overlap=80):
    chunks = []
    start = 0
    while start < len(text):
        chunks.append(text[start : start + size])
        start += size - overlap
    return chunks

for doc_id, text in documents.items():
    chunks = chunk_document(text)
    vectors = embedder.encode(chunks).tolist()
    collection.add(
        ids=[f"{doc_id}:{i}" for i in range(len(chunks))],
        embeddings=vectors,
        documents=chunks,
        metadatas=[{"doc_id": doc_id} for _ in chunks],
    )

# --- 線上：檢索 ---
query = "What is the refund window?"
q_vec = embedder.encode([query]).tolist()
top_k = collection.query(query_embeddings=q_vec, n_results=5)

# --- 線上：生成 ---
context = "\n\n".join(top_k["documents"][0])
final_prompt = build_prompt(query, context)   # 見上方範例
answer = call_llm(final_prompt)               # 呼叫你的模型 API
```

## 最常見的坑

RAG 系統「能跑」容易，「跑得好」很難。九成問題出在檢索，不在生成：

| 坑 | 症狀 | 修法 |
|----|------|------|
| 切塊把答案切成兩半 | 明明有答案卻召不回 | 加 overlap、按結構切、contextual chunk |
| 嵌入模型不適合檢索 | 召回結果文不對題 | 換 BGE / E5 / GTE 等檢索專用嵌入 |
| 撿回的段落不相關 | 上下文塞滿雜訊，答案被帶偏 | 調低 K、加 metadata 過濾、加 reranker |
| 資料太舊或缺失 | 答不出來但沒人知道是「資料缺」 | 建立資料更新機制與品質檢查 |
| 提示詞沒強制引用 | 模型還是用自己記憶 | 明示「只依據上下文」+ 允許說不知道 |
| 沒有評估 | 改了不知道變好變壞 | 建立 query→答案 的測試集，量 Recall / 正確率 |

還有一個觀念上的坑：**把 RAG 當成「什麼都能救」的銀彈**。如果問題需要跨多份文件推理，或需要最新即時資料，單靠一次檢索 + 一次生成往往不夠，需要多輪檢索、或結合工具呼叫。

## 總結

| 階段 | 做的事 | 常見工具 |
|------|--------|----------|
| 切塊 | 把文件切成語意完整的區塊 | 自訂程式、LangChain 分割器 |
| 索引 | chunk → 向量 → 存入向量庫 | sentence-transformers + FAISS / Chroma |
| 檢索 | 問題向量化，取 top-k 相近 chunk | FAISS、Chroma、Qdrant |
| 生成 | 問題 + 上下文 → 模型回答 | 各家 LLM API |

RAG 的第一性原理只有一句話：**模型不知道你的資料，那就把資料拿給它看。** 把這四段管線做扎實——尤其是檢索品質——你就能讓模型針對你的私有資料，給出可引用、可信的回答。

下一篇我們談一個相反的優化方向：不是讓模型看到更多資料，而是讓模型本身佔用更少的記憶體——量化。
