# llama.cpp 與 GGUF 解析

> 📅 2026-08-01 · 核心機制
> 拆解 llama.cpp 與 GGUF：量化等級、單檔格式，以及如何真正把推理跑起來。

---

Ollama 很好用，但它只是「包裝」。真正在背後做苦力的是 llama.cpp 這類推理引擎。這一篇我們拆開來講：llama.cpp 是什麼、GGUF 檔案長什麼樣、量化是怎麼回事，最後實際把推理跑起來。

***

## llama.cpp 是什麼

llama.cpp 是一個用 C/C++ 寫的開源專案，目標是讓大型語言模型能在普通消費級硬體上運作——包括沒有獨立顯卡的 CPU、Apple Silicon，以及 NVIDIA GPU。

**它的幾大特色：**

* 純 C/C++，無需 Python 依賴，啟動快、記憶體開銷低
* 支援 CPU 推理，Mac 上走 Metal，NVIDIA 走 CUDA
* 讀取 GGUF 格式的量化模型
* 提供 CLI、Server 與各種語言的 binding

### 為什麼它這麼重要

在 llama.cpp 出現之前，跑一個 7B 模型通常需要一張高階 GPU。llama.cpp 證明了只要用對格式與量化，普通筆電也能跑出可以接受的品質與速度。它是「本地 AI」浪潮的重要基石。

***

## GGUF：單一檔案的模型格式

GGUF 是 llama.cpp 使用的模型檔案格式，也是目前本地開源模型事實上的標準。

### 為什麼需要 GGUF

一個 Hugging Face 上的原始模型通常包含：

* 好幾個 `safetensors` 權重檔案（可能幾十 GB）
* `tokenizer.json`、`config.json` 等設定檔
* 缺少其中任何一個就無法載入

GGUF 把這一切打包成**單一檔案**，載入時不需要查閱任何外部檔案。這個設計讓「下載一個檔案 → 開始推理」變成可能。

### GGUF 檔案裡有什麼

* 模型的張量（權重）資料
* 模型架構與超參數（層數、維度、激活函數）
* 完整的分詞器（tokenizer）資料
* 量化資訊（用哪種量化、每種張量多少 bit）

因為所有資訊都在檔內，GGUF 自帶描述性，推理引擎不需要事先知道模型結構。

***

## 量化：用一點點品質換大量記憶體

原始權重通常用 16-bit 浮點數（FP16）儲存。一個 7B 模型光是權重就約 14GB，對多數人來說太大。

量化的想法很簡單：把權重從 16 bit 壓到 4 bit、5 bit 或 8 bit。代價是輸出品質略降，但記憶體用量大幅下降。

**記憶體公式：**

```
所需記憶體 ≈ 參數量 × 每權重 bit 數 / 8
```

7B 模型、Q4（約 4.5 bit/權重）→ 約 4GB。

***

## 常見量化等級

| 等級 | 約略 bit/權重 | 約略大小（7B 模型） | 用途 |
|------|--------------|------------------|------|
| Q4_0 | ~4.1 | ~3.9GB | 最小的合理選項，速度優先 |
| Q4_K_M | ~4.8 | ~4.4GB | 最推薦的預設，品質/大小平衡最佳 |
| Q5_K_M | ~5.5 | ~5.0GB | 想要更好品質、記憶體夠時 |
| Q6_K | ~6.6 | ~5.9GB | 接近原始品質 |
| Q8_0 | ~8.5 | ~7.6GB | 幾乎無損，需要較多記憶體 |
| F16 | 16 | ~14GB | 原始權重，訓練/精調用 |

### 怎麼選

* 記憶體有限或不確定 → Q4_K_M
* 品質優先、記憶體足夠 → Q5_K_M 或 Q6_K
* 只是想測試工具流程 → Q4_0

***

## 從原始碼編譯 llama.cpp

先安裝 CMake 與編譯工具，然後：

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release
```

編譯完成後，主要執行檔在 `build/bin/`：

* `llama-cli`：命令列推理
* `llama-server`：HTTP 伺服器
* `llama-quantize`：量化工具

### 用 Homebrew（macOS）

```bash
brew install llama.cpp
```

這樣可以直接使用 `llama-cli`，不需要自己編譯。

***

## 下載一個 GGUF 模型

GGUF 檔案通常放在 Hugging Face。以下載 Llama-2-7B-Chat 的 Q4_K_M 為例：

```bash
mkdir -p models
wget https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGUF/resolve/main/llama-2-7b-chat.Q4_K_M.gguf -O models/llama-2-7b-chat.Q4_K_M.gguf
```

現在很多官方模型也會直接提供 GGUF 檔案，直接在 Hugging Face 頁面搜尋「GGUF」即可。

***

## 用 llama-cli 跑推理

```bash
./build/bin/llama-cli \
  -m ./models/llama-2-7b-chat.Q4_K_M.gguf \
  -p "The sky is blue because" \
  -n 128
```

* `-m`：指定模型檔案
* `-p`：提示詞
* `-n`：最多生成多少個 token

### 常用參數

| 參數 | 作用 |
|------|------|
| `-t` | CPU 執行緒數 |
| `-ngl N` | 把前 N 層放到 GPU（Mac 上填 `99` 表示全下放） |
| `-c N` | 上下文長度（如 `4096`） |
| `--temp X` | 溫度 |
| `-i` | 互動模式 |

### Mac 上全 GPU 加速

```bash
./build/bin/llama-cli -m ./models/model.gguf -p "Hello" -n 64 -ngl 99
```

`-ngl 99` 代表盡可能把層數全部卸載到 Metal。

***

## 啟動 llama-server

llama.cpp 也附帶一個 OpenAI 相容的伺服器：

```bash
./build/bin/llama-server -m ./models/llama-2-7b-chat.Q4_K_M.gguf --port 8080
```

然後用 curl 呼叫：

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "local", "messages": [{"role": "user", "content": "Hello!"}]}'
```

`/v1/chat/completions` 是 OpenAI 的相容端點，很多既有工具可以直接切過來用。

***

## 自己轉檔與量化

如果你手上有一個 Hugging Face 格式的模型，可以自己轉成 GGUF 並量化：

```bash
# 1. 轉成 GGUF（F16）
python3 convert_hf_to_gguf.py ./my-hf-model --outfile mymodel-f16.gguf

# 2. 量化成 Q4_K_M
./build/bin/llama-quantize mymodel-f16.gguf mymodel-Q4_K_M.gguf Q4_K_M
```

`convert_hf_to_gguf.py` 在 llama.cpp 的倉庫根目錄。

***

## 總結

* llama.cpp 是讓本地推理可行的底層引擎
* GGUF 把模型打包成單一自描述檔案
* 量化用少量品質換取大幅的記憶體下降
* `Q4_K_M` 是絕大多數情況的安全預設
* 動手跑一次 llama-cli，就掌握了本地推理的核心
