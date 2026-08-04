# 強化你的 LLM 生產應用程式

> 📅 2026-08-01 · 架構總覽
> 從威脅模型到護欄、工具權限、密鑰管理、稽核日誌，再到一份務實的安全檢查清單——把 LLM 應用從 demo 變成可上線的系統。

---

## 強化你的 LLM 生產應用程式

「demo 能用」和「生產能跑」之間，隔著一整條安全工程。

前幾篇講了注入、越獄與評測。這一篇把這些收斂成一套**生產架構**：從威脅模型開始，逐層加上護欄，最後用一份檢查清單收尾。

***

### 先畫出你的威脅模型

安全設計的第一步不是加功能，而是回答四個問題：

1. **資產**：系統裡最值錢、最不能外洩的是什麼？（客戶資料、系統提示詞、工具權限、API 金鑰）
2. **攻擊者**：誰會攻擊？動機是什麼？（好奇的測試者、想要資料的對手、想騙客服的帳號盜用者）
3. **攻擊面**：資料從哪裡進來、往哪裡去？（使用者輸入、RAG 文件、模型輸出、工具呼叫、後端 API）
4. **後果**：最壞情況是什麼？（資料外洩、未授權動作、聲譽傷害、法律責任）

```
攻擊面示意：

使用者輸入 ─┐
RAG 文件 ──┼→ [護欄:輸入] → [LLM] → [護欄:輸出] → [工具層:權限] → 外部世界
模型輸出 ──┘                          ↓
                                   [稽核日誌 / 監控]
```

威脅模型不是一次性的。**每次新增工具、資料來源或使用者角色，都要重新跑一遍。**

***

### 護欄：輸入與輸出兩端

#### 輸入端

* **輸入長度限制**：避免超長上下文（也是多樣本越獄的養分）
* **分類器掃描**：用 Moderation API 或自建分類器，偵測明顯有害的輸入
* **角色分離**：把「使用者輸入」「RAG 內容」「系統指令」放進不同信任層級與不同訊息角色

```python
def sanitize_input(user_text: str, max_chars: int = 4000) -> str:
    if len(user_text) > max_chars:
        raise ValueError(f"輸入過長：{len(user_text)} 字元")
    decision = moderation.classify(user_text)
    if decision.harmful:
        raise PermissionError("輸入被內容審查攔截")
    return user_text
```

#### 輸出端

模型可能被攻破，所以輸出端是最後的檢查點：

* **結構化輸出**：要求模型輸出 JSON 並驗證 schema，而不是自由文字
* **敏感模式攔截**：API 金鑰、信箱、URL 的 regex 掃描
* **工具呼叫驗證**：白名單 + 參數 schema 檢查

```python
def guard_output(raw: str) -> str:
    if re.search(r"sk-[A-Za-z0-9]{20,}", raw):
        raise PermissionError("輸出包含疑似 API 金鑰，已攔截")
    return raw
```

***

### 工具權限：最小權限 + 人為審批

LLM 應用最危險的擴張是**工具**。權限設計的三條鐵律：

1. **最小工具集合**：只暴露目前任務需要的工具。信箱助理不需要「刪除帳號」。
2. **綁定使用者**：所有動作以**目前使用者**的身份執行，而不是以服務帳號的身份。
3. **破壞性動作要人為確認**：寄信、改資料、轉帳這類動作，先暫停，等使用者確認。

```python
TOOL_POLICY = {
    "send_reply":      {"confirm": False, "destructive": False},
    "escalate_ticket": {"confirm": False, "destructive": False},
    "delete_message":  {"confirm": True,  "destructive": True},
    "modify_order":    {"confirm": True,  "destructive": True},
}

def authorize_tool(user: User, func: str, args: dict) -> Action:
    policy = TOOL_POLICY[func]
    # 1. 工具存在嗎？
    if func not in TOOL_POLICY:
        raise PermissionError(f"工具不存在：{func}")
    # 2. 使用者有權限嗎？
    if func not in user.permitted_tools:
        raise PermissionError(f"使用者沒有 {func} 的權限")
    # 3. 破壞性動作需要人為確認
    if policy["destructive"]:
        return Action.PENDING_CONFIRMATION
    return Action.EXECUTE
```

**資料庫權限也要跟著縮**：代理連線用唯讀或限縮角色，而不是管理員帳號。

***

### 密鑰與機密管理

LLM 應用會接觸大量密鑰：API 金鑰、資料庫密碼、第三方服務 token。規則很簡單：

* **永遠不要**把密鑰寫進前端、程式碼、提示詞或日誌
* 用環境變數或密鑰管理服務（Vault、雲端 secret manager）
* 密鑰要有**最小範圍**與**輪換**：每個服務一把，分開管理
* 把「密鑰外洩」當成必然事件：偵測、輪換、事後檢討都要有

```
錯誤：OPENAI_API_KEY=sk-xxx 寫進 .env 然後 commit
正確：從 secret manager 注入，本機開發用未 commit 的 .env.local
      （而且已加入 .gitignore）
```

**別忘了系統提示詞本身也是機密**——提示注入攻擊常以竊取提示詞為目標。

***

### 稽核與監控

沒有日誌，就等於沒有真相。生產環境必須回答：「這個請求發生了什麼？」

每個 LLM 請求至少記錄：

* 請求 ID、時間、使用者身份
* 輸入摘要（小心敏感資料，可先脫敏）
* 模型輸出摘要
* 呼叫了哪些工具、參數是什麼
* 評測／審查結果（有沒有被護欄攔截）

```python
def audit(request_id: str, user_id: str, func: str, args: dict, result: str):
    log_event(
        request_id=request_id,
        user_id=user_id,
        tool=func,
        args_redacted=redact(args),   # 脫敏後才記錄
        result_summary=summarize(result),
        blocked=result == "BLOCKED",
    )
```

監控兩件事：

* **攔截率**：護欄攔截比例突然下降 → 護欄可能失效
* **異常模式**：單一使用者短時間大量請求、特定注入模式重複出現

***

### 事件應變

就算做到上述一切，還是要預設「會出事」。最小應變流程：

1. **偵測**：攔截率、錯誤率、可疑請求的告警
2. **止血**：關閉工具權限、回滾模型版本、停用有問題的系統提示詞
3. **取證**：從稽核日誌還原請求鏈
4. **修正**：加測試案例（這次的攻擊手法進入評測集）、修復弱點
5. **復盤**：威脅模型哪裡漏了？

關鍵：**每次事件都是評測集的補丁**。出事的攻擊手法應該變成永久的自動化測試案例。

***

### 實務安全檢查清單

| 類別 | 檢查項 |
|------|--------|
| 護欄 | 有輸入/輸出過濾？有長度限制？有分類器掃描？ |
| 工具 | 只暴露最小工具集？權限綁定目前使用者？破壞性動作需人為確認？ |
| 密鑰 | 沒有密鑰在程式碼/前端/日誌？密鑰有最小範圍與輪換？ |
| 資料 | 資料庫用最小權限角色？RAG 內容有標示不可信？ |
| 稽核 | 每個請求有日誌？敏感資料有脫敏？有攔截率監控？ |
| 評測 | 評測集有安全層？跑在 CI/nightly？事件手法有進評測集？ |

用這份清單在每次發布前過一遍，比任何「保證」都可靠。

***

### 一句話總結

> 生產安全不是靠「一個強壯的系統提示詞」，而是靠**一套分層的架構**：威脅模型定義風險，護欄攔截兩端，工具權限限制破壞力，密鑰管理與稽核日誌提供可追蹤性，而評測確保這一切在每次改動後仍然成立。
