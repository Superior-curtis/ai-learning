# CLAUDE.md：讓 Claude Code 真正理解你的專案

> 📅 2026-07-27 · 使用教學
> 深入解析 CLAUDE.md 如何讓 Claude Code 讀懂你的專案規範與偏好。

---

## CLAUDE.md：讓 Claude Code 真正理解你的專案

### CLAUDE.md 是什麼

CLAUDE.md 是 Claude Code 的專案記憶檔案。當你在專案根目錄放一份 `CLAUDE.md`，Claude Code 每次啟動都會自動讀取它，理解這個專案的規範、技術棧、常用指令和邊界限制。

很多時候 AI「不聽話」的問題，不是模型不夠聰明，而是系統沒有拿到清楚的專案約束。

***

### 一個可直接套用的模板

```markdown
# 專案說明

- 這是一個 Next.js + TypeScript 專案
- 樣式使用 Tailwind CSS
- 頁面放在 app/ 目錄
- 共用元件放在 components/ 目錄

# 開發規範

- 優先複用已有元件
- 不要隨意新增依賴
- 變數命名使用 camelCase
- 修改後執行 build 檢查

# 常用指令

- 安裝依賴：npm install
- 本機開發：npm run dev
- 正式建置：npm run build

# 注意事項

- 不要修改 legacy/ 目錄
- 涉及金流邏輯時先給方案，不要直接改
```

***

### 寫 CLAUDE.md 的一個原則

**不要寫廢話，要寫真正會影響行為的內容。**

| 該寫的 | 不該寫的 |
|--------|----------|
| 修改後必須執行 `npm run build` | 我們注重程式碼品質 |
| 不允許新增依賴，除非先說明理由 | 請保持程式碼乾淨 |
| 表單元件統一複用 `components/forms` | 這是個 Next.js 專案 |

CLAUDE.md 寫的不是介紹詞，而是 Claude Code 在這個專案裡的長期工作說明書。寫得越具體、越貼近真實約束，Claude Code 的表現就越穩定。

***

### 進階技巧

**分層結構：**

* 全域 CLAUDE.md（`~/.claude/CLAUDE.md`）— 你個人的通用規範，所有專案都適用
* 專案 CLAUDE.md（`專案根目錄/CLAUDE.md`）— 只針對當前專案

**常見用途：**

* 定義技術棧和框架版本
* 記錄特殊依賴和環境變數
* 指定程式碼風格偏好
* 標記不該修改的目錄或檔案
* 記下常見問題的解決方式
