# 嵌入與向量搜尋：把語意放進座標空間

> 📅 2026-08-01 · 核心機制
> 什麼是 embedding、餘弦相似度怎麼算、向量資料庫在做什麼，以及檢索什麼時候會成功、什麼時候會失敗。

---

上一篇文章我們看到，模型內部的每個詞都是一個向量。這篇文章回答：**這些向量從哪裡來、有什麼用**。答案是「嵌入（Embedding）」——把文字轉成座標，讓「語意相近」變成「空間上相近」，而向量搜尋就是在這個座標空間裡找最近的鄰居。

## 什麼是嵌入

嵌入是把一個 token、句子或文件，映射成一個固定維度的實數向量。例如用 `sentence-transformers` 套件，一句話會被轉成一個 384 或 1024 維的向量。

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("BAAI/bge-small-en-v1.5")

vec_a = model.encode("The cat sat on the mat")
vec_b = model.encode("A dog lay on the floor")
vec_c = model.encode("Interest rates rose this quarter")

print(vec_a.shape)  # (384,)
```

重點不是維度大小，而是**位置**：這三個向量在空間裡，`vec_a` 和 `vec_b` 比較靠近（都是動物與地板），`vec_c` 離它們很遠（是財經新聞）。

### 從「辭典」到「語意地圖」

早期的方法（one-hot、TF-IDF）把詞當成獨立的代號：「貓」和「狗」沒有任何共通點，因為它們在字典裡是不一樣的欄位。

嵌入的關鍵突破是：**把語意變成連續座標**。學到的向量空間裡，相似的詞聚在一起，向量之間的差還有方向意義——經典例子：

```
向量("國王") - 向量("男人") + 向量("女人") ≈ 向量("女王")
```

這說明了嵌入捕捉的不只是「字典定義」，而是詞之間的關係結構。現代句子嵌入沿用了同一套哲學，只是把對象從「詞」升級成「整個句子」。

## 怎麼比對「相近」

有了向量，剩下的問題是：如何測量「相近」？最常用的是**餘弦相似度（Cosine Similarity）**，它只看兩個向量的方向，不看長度——這對文字特別合適，因為句子的長短不該影響語意判斷。

```
cos_sim(A, B) = (A · B) / (|A| · |B|) = Σᵢ AᵢBᵢ / (√Σᵢ Aᵢ² · √Σᵢ Bᵢ²)
```

範圍是 -1 到 1：1 表示同方向（非常相近），0 表示正交（無關），-1 表示相反。

```python
import numpy as np

def cosine_similarity(a, b):
    a = np.asarray(a, dtype=np.float32)
    b = np.asarray(b, dtype=np.float32)
    return float((a @ b) / (np.linalg.norm(a) * np.linalg.norm(b)))

print(cosine_similarity(vec_a, vec_b))  # 高，例如 0.62
print(cosine_similarity(vec_a, vec_c))  # 低，例如 0.05
```

實務上通常先把向量正規化（`a / ||a||`），讓餘弦相似度退化成普通內積，加快計算。

## 向量資料庫在做什麼

「找出最相近的 K 個向量」聽起來簡單，但當你有 1000 萬份文件時，逐一計算餘弦相似度就太慢了。向量資料庫（Vector Database）專門解決這個問題：

* **索引結構**：用近似最近鄰（ANN）演算法，例如 HNSW（分層可導航小世界圖）或 IVF（倒排檔案），把搜尋壓到毫秒級。
* **支援增刪改**：可以動態加入新向量、刪除舊向量。
* **混合過濾**：在向量搜尋前先用 metadata（標籤、日期、權限）過濾。

常見工具：FAISS（Facebook 的函式庫）、Chroma（輕量、適合小專案）、Qdrant、Pinecone、Weaviate、Milvus。

### 最小可用範例：FAISS

```python
import faiss
import numpy as np

# 假設所有向量已正規化（長度 = 1）
embeddings = np.random.rand(10_000, 384).astype("float32")
faiss.normalize_L2(embeddings)

index = faiss.IndexFlatIP(384)   # 內積 == 餘弦相似度（正規化後）
index.add(embeddings)            # 建立索引

query = np.random.rand(1, 384).astype("float32")
faiss.normalize_L2(query)

scores, ids = index.search(query, k=5)  # 回傳前 5 近鄰
print(ids, scores)
```

對更大的規模，可以換成 `IndexHNSWFlat`，搜尋仍是毫秒級。

## 檢索什麼時候會成功

* **語意近義**：問「如何讓狗別亂叫」，能召回「吠叫的行為矯正」——即使兩者沒有共同的關鍵字。
* **多語言對照**：好的多語嵌入能把不同語言裡意思相同的句子放在相近位置。
* **模糊查詢**：使用者記不清精確措辭，向量搜尋仍然能找到相關內容。

## 檢索什麼時候會失敗

向量搜尋不是萬靈丹。常見的失效模式：

| 問題 | 原因 | 應對 |
|------|------|------|
| 精確比對失效 | 搜尋「API v2.3」卻召回「API v3.1」 | 加上關鍵字 / 精確匹配的混合搜尋 |
| 需要多個條件同時成立 | 餘弦相似度是軟性排序，沒有布林邏輯 | 先用 metadata 過濾再排序 |
| 嵌入不理解稀有詞 | 專有名詞、程式碼片段在訓練語料中少見 | 用更大的模型、或加上同義詞展開 |
| 語意太細微 | 標題與內文的語意無法用向量區分 | 嘗試多欄位 / 多向量檢索 |
| 資料本身就很像 | 幾千份保險條款彼此相近，top-k 全是近似複製品 | 用重新排序（rerank）打破平手 |

還有一個常被忽略的陷阱：**嵌入模型和檢索任務不匹配**。把「用於分類」的嵌入拿去做檢索，效果通常很差；應該選用專門訓練的檢索嵌入（如 BGE、E5、GTE），並最好在自家資料上微調。

## 評估檢索品質

用三個簡單指標衡量檢索好壞：

* **召回率（Recall@k）**：正確文件有沒有出現在前 k 名？
* **精確率（Precision@k）**：前 k 名裡有多少是正確的？
* **MRR（Mean Reciprocal Rank）**：第一份正確文件的排名有多前面？

有了基準資料集（query → 期望文件），跑一次評估就能知道：換嵌入模型、改 chunk 大小、加 metadata 過濾，到底是變好還是變壞。

## 總結

| 概念 | 一句話 |
|------|--------|
| 嵌入 | 把文字映射成向量，讓語意相近 = 空間相近 |
| 餘弦相似度 | 用方向比對向量，適合文字 |
| 向量資料庫 | 用 ANN 索引在百萬級向量中做毫秒級搜尋 |
| 成功關鍵 | 檢索嵌入 + 好的 chunk + metadata 過濾 |
| 常見失敗 | 精確比對、複合條件、稀有詞、資料太相似 |

嵌入把「理解語意」變成「算距離」——而這正是 RAG 的第一塊積木。下一篇我們會把嵌入、檢索和生成拼成完整的 RAG 管線。
