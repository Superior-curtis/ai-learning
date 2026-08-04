# Token：語言不是以「字」為單位

> 📅 2026-08-04 · 核心概念
> 模型看的不是字、不是詞，而是 token——一種把文字切成小塊的單位。中文一句話，換成 token 可能會多花三倍的「錢」。

---

上一篇文章說，LLM 的任務是「猜下一個詞」。但這裡有個隱藏問題：**模型看的到底是什麼「詞」？**

你以為模型看到的是「字」或「詞」，其實它看到的是一種叫做 **token** 的單位。token 是 AI 世界最基礎、也最容易被忽略的貨幣——因為**它同時決定了模型的記憶上限、計費方式和生成速度**。

## token 是什麼

Token 是「把文字切成的小塊」。模型不認字、不認詞，它只認 token——由一個叫做 **tokenizer** 的程式，把輸入的文字切成一串 token。

切法因語言而異，很「不規則」：

```英文：常用詞幾乎一個&#x20;token&#x20;一個詞
Hello, world!    →  [Hello] [,] [world] [!]        (4 tokens)
The quick brown fox →  [The] [quick] [brown] [fox]   (4 tokens)
unbelievable       →  [un] [believ] [able]           (3 tokens，拆成詞根)
```

* 常用字，一個詞一個 token（`the`、`is`、`hello`）。
* 罕見字或組合詞，會被拆成幾個 token（`unbeliev-able`）。

```中文：每個字通常就是&#x20;1&#x20;個&#x20;token&#x20;以上
你好，世界！   →  [你] [好] [，] [世] [界] [！]     (6 tokens)
人工智慧         →  [人工] [智慧]                     (2 tokens，常用詞組)
颶風（颱風）     →  每個字 1-2 個 token，罕見字更貴
```

關鍵結論：**中文往往比英文「貴」**。同一句話，中文的 token 數可能是英文的 2-3 倍。這不是歧視——只是因為英文訓練資料多，tokenizer 對英文字詞切得更「省」。

## 為什麼要切 token

直接存整句不是更簡單嗎？問題在於字典的大小：

* 如果模型以「字」為單位，英文 26 個字母就夠——但語言不是由字母組成的，`un`、`believ`、`able` 這些「有語意的零件」才是。
* 如果以「詞」為單位，英文大概有 50 萬個常用詞，字典巨大；而且永遠會遇到沒看過的詞（人名、新創詞）。
* **Token 是折衷**：字典幾萬個常見「子詞」，既能覆蓋絕大多數文字，又比逐字處理更有語意。

就像把英文切成「可重複使用的樂高積木」：`un` + `believ` + `able` 三塊積木，可以組出 `unbelievable`、`believable`、`unbelievably`……不用為每個組合準備一塊新積木。

## BPE：tokenizer 怎麼決定切法

最常用的演算法叫 **BPE（Byte Pair Encoding）**，精神很簡單：**從「字」開始，把最常一起出現的兩個符號，合併成一個新符號，重複直到預算用光。**

1. 把所有文字先切成單字元。
2. 統計哪兩個字元最常「相鄰出現」。
3. 把它們合併成一個新 token。
4. 重複，直到 token 數量達到目標（例如 5 萬個）。

```text
"low low low low"  →  [l] [o] [w] [l] [o] [w] …
最常相鄰的是 l+o →  [lo] [w] [lo] [w] …
再來是 lo+w     →  [low] [low] …
→ 最終 low 變成 1 個 token
```

所以 tokenizer 的切法不是「語法正確的切法」，而是**「這樣切最省」的統計結果**。這就是為什麼同一個詞在不同語境可能被切成不同 token，也為什麼新詞或人名往往被拆得七零八落。

## token 為什麼重要：三個理由

**1. 它決定記憶上限（上下文）**

模型一次能看的內容上限，是用 token 數量的。8K 上下文 = 8K token，不是 8K 字。中文使用者特別有感：8K token 的上下文，裝不下的中文可能只有 3-4 千字。見 `llmcore-03-context`。

**2. 它決定計費**

API 幾乎都以 token 計價：輸入多少 token、輸出多少 token，明碼標價。同樣內容，中文的 token 數是英文的 2-3 倍，**成本也跟著乘以 2-3**。

> 一句話記住 token 的商業意義：token 就是 AI 的計費單位，而中文比英文更耗 token——所以同一句中文，成本更高。

**3. 它決定速度**

生成時，模型「抽一個 token」才輸出一個 token。輸出越少 token，回覆越快。這也是為什麼「簡短回答」會被當成一個工程優化點。

## 一個實用的估算

自己心算 token 數：

* **英文**：約 1 token ≈ 0.75 個英文單詞（100 字 ≈ 133 token）。
* **中文**：約 1 token ≈ 0.5-1 個中文字（100 字 ≈ 100-200 token）。

```用&#x20;API&#x20;工具確認（概念）
# 各家 SDK 都提供 token 計數或回傳用量
# Anthropic: 回應的 usage.input_tokens / output_tokens
# OpenAI:    response.usage.prompt_tokens / completion_tokens

response = client.messages.create(
  model="claude-sonnet-4-5",
  max_tokens=1000,
  messages=[{"role": "user", "content": "用繁體中文解釋 token"},
            {"role": "user", "content": "再用英文解釋一次"}],
)
print(response.usage.input_tokens)   # 兩個語言的 token 數一目了然
```

## 下次你看到「token」時

你現在知道：token 是文字的樂高積木、是模型計費的貨幣、是上下文與速度的天花板。它是 AI 世界少數幾個**真正決定你帳單數字**的概念。

下一步，看看這些 token 能裝進多大的盒子：上下文窗口（`llmcore-03-context`）。

#### Q: 同樣內容，為什麼中文通常比英文花更多 token？

* 中文是 Unicode，比較占儲存空間

* tokenizer 以統計方式切 token，中文常用詞組較少，單位文字切出的 token 較多

* 模型偏好英文，會故意少算英文

* 中文字集比較大，所以每個字都比較貴

> 💡 BPE 式 tokenizer 是統計出來的「最省切法」。英文訓練資料極多，詞根積木很豐富，所以英文切得省；中文較少被切成多字詞，導致同樣內容 token 數變多、成本變高。
