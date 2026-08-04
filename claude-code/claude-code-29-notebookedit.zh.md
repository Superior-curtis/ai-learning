# NotebookEditTool：編輯 Notebook

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · 工具組
> 了解 Claude Code 如何以 cell 為粒度編輯 Jupyter Notebook。

---

## NotebookEditTool：編輯 Notebook

.ipynb 檔案雖然本質上是 JSON，但語義上它是一組有順序的 cell，混合了 code / markdown，帶輸出、元資料和語言資訊。Claude Code 給它單獨做了 NotebookEditTool。

***

### 輸入 Schema

**操作粒度是 cell 級，不是整檔案字串級。**

***

### 編輯鏈路

模型需要修改 .ipynb
⬇
校驗副檔名與權限
⬇
讀取並解析 notebook JSON
⬇
按 cell\_id 做 replace/insert/delete
⬇
保持 notebook 結構 → 寫回

***

### 一句話總結

> Claude Code 沒把 Notebook 降級成普通文字，而是給這種「程式碼 + 文件」混合格式**單獨做了一條受控編輯鏈路**，以 cell 為操作單位，保留結構完整性。
