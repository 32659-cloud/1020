# CLAUDE.md

這是一支在我自己電腦上執行的 Discord bot（Python 3.12 + uv）：指定頻道有人傳食物照片，就用 `claude -p` 判斷、估熱量、記進 Excel，再回覆上傳者。

## 開始任何工作前

先讀 `docs/architecture.md`。所有設計決定（欄位、流程、回覆格式、檔名規則、安全做法）都以它為準；需求背景在 `docs/requirements.md`。如果程式需要和文件不同的做法，先改文件、並告訴我，再改程式。

## 四段防護

`docs/architecture.md` 第 12 節定義了四段防護。每一段都必須用清楚的註解標出開始和結束，格式固定如下：

```python
# ===== 防護 N 開始：<這一段在防什麼> =====
# 關掉之後的對照行為：<一句話>
...
# ===== 防護 N 結束 =====
```

| 編號 | 防什麼 | 位置 |
|---|---|---|
| 防護 1 | `claude -p` 讀到 DATA_DIR 裡別人的照片和紀錄 | `analyzer.py` |
| 防護 2 | 上傳者打的字被當成給 `claude -p` 的指令 | `analyzer.py` |
| 防護 3 | 食物名稱當檔名時，路徑跑到別處 | `storage.py` |
| 防護 4 | 寫進 Excel 的文字被當成公式 | `excel_log.py` |

- **一開始全部開啟。** 之後上課時會逐段關掉做對照，所以每一段要能整段拿掉或換掉，不可以把防護邏輯散落在別處。
- 不要為了「方便」擅自關掉、弱化或合併任何一段。

## 不可以 commit 的東西

- **`.env` 絕對不可以 commit**，裡面有 `DISCORD_TOKEN`。只能 commit `.env.example`（只有名稱，沒有真實的值）。
- **DATA_DIR 裡的東西不要 commit**：照片（`photos/`）和 Excel（`food_log.xlsx`）都是別人的飲食紀錄。DATA_DIR 應放在 repo 外面，萬一放在 repo 裡，也必須在 `.gitignore` 裡。
- 日誌、錯誤訊息和程式輸出都不可以印出 token。
- commit 之前用 `git status` 確認沒有上述檔案。
