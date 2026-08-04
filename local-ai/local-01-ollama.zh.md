# 用 Ollama 跑起你的第一個本地模型

> 📅 2026-08-01 · 使用教學
> 從安裝、拉取模型到第一次對話，帶你零基礎跑起第一個本地 LLM。

---

這一篇是「本地 AI」系列的第一篇。目標很簡單：讓你在自己的電腦上跑起一個真正能對話的語言模型，全程離線、不花一毛錢的 API 費用。

***

## Ollama 是什麼

Ollama 是一個把「本地模型」變成一行指令的工具。你不需要懂 CUDA、不需要自己編譯 C++、也不用處理 GGUF 檔案格式——它把下載模型、啟動推理伺服器、提供對話介面這三件事全部包裝好了。

**Ollama 幫你解決的事：**

* 一行指令下載模型
* 自動選擇 CPU / GPU 後端
* 內建對話介面與 REST API
* 支援 macOS、Linux、Windows

### 為什麼從 Ollama 開始

因為它門檻最低。其他工具（如 llama.cpp）更強大也更彈性，但需要多懂一點底層。先用 Ollama 建立「本地模型真的跑得起來」的體感，再深入細節，學習曲線會平滑很多。

***

## 安裝 Ollama

### macOS（Homebrew）

```bash
brew install ollama
```

### Linux 與其他 macOS

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Windows

到 ollama.com/download 下載安裝程式，執行後 Ollama 會自動在背景啟動。

### 確認安裝成功

```bash
ollama --version
```

第一次執行 `ollama run` 時，如果伺服器還沒啟動，Ollama 會自動在背景幫你啟動。

***

## 拉取你的第一個模型

Ollama 的模型庫在 ollama.com/library。挑選標準是「你的記憶體夠不夠大」。先拉一個 3B 等級的入門模型：

```bash
ollama pull llama3.2
```

這個模型約 2GB，記憶體只有 8GB 的舊機器也能跑。

如果你的機器記憶體更低，可以改用更小的 1.5B 模型：

```bash
ollama pull qwen2.5:1.5b
```

### 查看已下載的模型

```bash
ollama list
```

`ollama list` 會顯示模型名稱、標籤、大小與最後修改時間。

***

## 開始第一次對話

```bash
ollama run llama3.2
```

看到 `>>>` 提示符號，代表模型已載入，可以直接打字對話：

```
>>> 用一句話解釋「快取」是什麼。
快取是把重複使用的資料暫時存放在更快的地方，以加速存取速度。

>>> /bye
```

輸入 `/bye` 即可結束對話。

### 直接把 prompt 當參數傳入

不想進入互動模式，也可以直接帶 prompt：

```bash
ollama run llama3.2 "為什麼天空是藍色的？"
```

這個寫法適合快速測試，不會進入互動介面。

### 在對話中調整參數

Ollama 的互動模式支援 `/set` 指令，可以即時調整生成參數：

```
>>> /set temperature 0.7
>>> /set num_ctx 8192
```

* `temperature`：數值越高輸出越隨機，越低越確定。
* `num_ctx`：上下文長度，越大能記住的內容越多，但越耗記憶體。

***

## 管理正在執行的模型

### 查看目前載入的模型

```bash
ollama ps
```

### 停止某個模型

```bash
ollama stop llama3.2
```

模型預設在空閒 5 分鐘後自動卸載，所以通常你不需要手動停。

***

## 用 API 呼叫本地模型

Ollama 在背景跑的其實是一個本地伺服器，預設監聽 `http://localhost:11434`。任何支援 OpenAI 相容介面的程式，都可以直接打這個端點。

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [{"role": "user", "content": "Hello!"}],
  "stream": false
}'
```

回傳的 JSON 中的 `message.content` 就是模型的回答。`stream: false` 代表等完整回答一次回傳，方便讀取。

***

## 用 Modelfile 打造你自己的模型

Ollama 提供一個叫 Modelfile 的檔案格式，讓你定義 system prompt 與預設參數。先建立一個檔案：

```
FROM llama3.2

SYSTEM 你是一個資深的 TypeScript 工程師，回答要簡短、具體、給範例。
PARAMETER temperature 0.2
```

然後建立成新模型：

```bash
ollama create mycoder -f Modelfile
```

之後就可以直接使用：

```bash
ollama run mycoder
```

***

## 下一步

你已經跑起第一個本地模型了。接下來可以往三個方向前進：

* 想知道模型底層怎麼運作？看系列下一篇：llama.cpp 與 GGUF
* 想知道你的 Mac 適合跑多大的模型？看硬體比較篇
* 想讓模型讀你自己的文件？看系列最後一篇：本地 RAG

先把 `ollama run` 玩熟，之後每一篇都會用到它。
