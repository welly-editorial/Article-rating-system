# Welly 審稿評分系統 · GitHub Pages 版

完整的兩工具系統，部署在 GitHub Pages，公開任意可看。

## 目錄結構

```
github-pages-version/
├── index.html        ← 導覽首頁（兩個工具入口）
├── panel.html        ← 評分計算面板（給審稿人）
├── dashboard.html    ← 個人儀表板（給撰文編輯）
└── README.md         ← 本檔（部署 SOP）
```

## 兩個工具的差別

| 工具 | 路徑 | 給誰 | 資料來源 |
|---|---|---|---|
| 評分計算面板 | `/panel.html` | 6 位審稿人 | 純前端計算，不讀任何外部資料 |
| 個人儀表板 | `/dashboard.html` | 12 位撰文編輯 | 從 Google Sheet「發佈到網路」CSV 抓取 |

**關鍵差別**：評分面板完全靜態、不需要任何設定就能用。儀表板需要設定 Sheet publish URL 才能讀資料。

---

## 部署 SOP（首次設定）

### Step 1：建立 GitHub Repository

到 [github.com/new](https://github.com/new) 建一個新 repo（建議名稱 `welly-review-system`）。可以放在 `wellyeditor` organization 下（跟 client-notifier 同位置）。

GitHub Pages 免費版要求 repo 必須是 Public，所以選 Public。

### Step 2：上傳檔案

兩種方式擇一：

**方式 A：拖檔上傳（不用裝 git）**
1. repo 建好後 → **Add file → Upload files**
2. 把 `index.html`、`panel.html`、`dashboard.html`、`README.md` 四個檔案拖進去
3. 按下方 **Commit changes**

**方式 B：git command line**
```bash
cd "/Users/chunmei/Desktop/Claude Cowork/Skills/review-score-calculator/github-pages-version"
git init
git add .
git commit -m "init review system"
git branch -M main
git remote add origin https://github.com/wellyeditor/welly-review-system.git
git push -u origin main
```

### Step 3：啟用 GitHub Pages

1. 進 repo 頁面 → **Settings**
2. 左邊選單 → **Pages**
3. **Source** 選 `Deploy from a branch`
4. **Branch** 選 `main`、資料夾 `/ (root)`
5. 按 Save
6. 等 1-2 分鐘部署
7. 出現網址：`https://wellyeditor.github.io/welly-review-system/`

### Step 4：設定 Google Sheet「發佈到網路」（給儀表板用）

只有「個人儀表板」需要這步，評分面板不用。

1. 打開 [審稿評分系統_資料庫_v3 Sheet](https://docs.google.com/spreadsheets/d/1sWt0CUDSBWzLmDNK1H2pwWNI0zhpvTnOByjZMI3P5gA/edit)
2. 上方選單 **檔案 → 共用 → 發佈到網路**
3. **發佈內容** 選「整份文件」
4. **格式** 選「逗號分隔值（.csv）」
5. 按「發佈」→ 確認
6. 複製產生的連結（會像 `https://docs.google.com/spreadsheets/d/.../pub?output=csv`）

> 警告：發佈後**任何拿到連結的人都能看到 Sheet 內容**。確認你接受這個隱私邊界再繼續。

### Step 5：把 CSV URL 填進 dashboard.html

打開 `dashboard.html`，找到這一段（約第 113 行）：

```javascript
const SHEET_CSV_URL = 'PASTE_YOUR_PUBLISH_TO_WEB_CSV_URL_HERE';
```

把 placeholder 換成你 Step 4 拿到的 URL。

存檔後 push 到 GitHub（重複 Step 2 的 commit 流程，或在網頁上直接編輯 → commit changes）。

### Step 6：訪問網址測試

打開 `https://wellyeditor.github.io/welly-review-system/`：

- 首頁應該看到兩個工具卡片
- 點「評分計算面板」 → 可填表單點按鈕（純前端，立刻能用）
- 點「個人儀表板」 → 選編輯姓名應該看到 OKR 與評分歷史

如果儀表板顯示「尚未設定 Google Sheet publish-to-web URL」，代表 Step 5 沒做或 URL 填錯，回去修。

---

## 日常使用

### 審稿人

打開 `https://wellyeditor.github.io/welly-review-system/panel.html` →
1. 填文章名、Doc 連結、撰文編輯、審稿人
2. 切「文章」或「架構」模式
3. 點扣分項目的 `＋` / `−` 按鈕（總扣分即時更新）
4. 特殊情況用「其他」自由填寫
5. 按「產出 TSV」
6. 複製 TSV 貼進 Sheet 第一個空白列
7. 複製 Trello 摘要貼進該篇 Trello 卡片

### 撰文編輯

打開 `https://wellyeditor.github.io/welly-review-system/dashboard.html` →
1. 選自己姓名
2. 看 OKR、達標率、評分歷史
3. 要存檔按「下載 Markdown」

### 主管

跟撰文編輯一樣流程，但下拉選「全部編輯」可看全團隊統計。

---

## 隱私邊界提醒

| 項目 | 可見性 |
|---|---|
| GitHub Pages 網站 | 任何拿到網址的人都能看 |
| Google Sheet 公開 CSV URL | 任何拿到連結的人都能讀 |
| 評分資料、編輯姓名、審稿人姓名、扣分細節 | **全部公開** |
| 評分面板的點選計次（localStorage） | 只在使用者自己的瀏覽器 |

如果要保留隱私要走「Google Apps Script API + GitHub Pages 加 auth」更複雜的方案。

---

## 維護與更新

- **改 UI 樣式／功能**：改對應的 .html 檔 → push 到 GitHub → Pages 自動更新（1-2 分鐘）
- **改 OKR 結構**：到 Google Sheet 改第一個區塊 → publish URL 不變 → 儀表板自動讀新版
- **加新編輯**：改 `panel.html` 跟 `dashboard.html` 內的 `<select id="editor"...>` 加 `<option>`，順便更新 `NAME_MAP`
- **新增扣分項目**：改 `panel.html` 內的 `ITEMS` 物件，加進對應的 tier 群組
- **撤銷發佈**：到 Sheet → 檔案 → 共用 → 發佈到網路 → 停止發佈 → 儀表板會讀不到資料

---

## URL 分享技巧

儀表板支援 URL 參數，直接分享特定編輯/月份/模式的 view：

```
https://wellyeditor.github.io/welly-review-system/dashboard.html#editor=黃睿慈
https://wellyeditor.github.io/welly-review-system/dashboard.html#editor=黃睿慈&month=2026-05
https://wellyeditor.github.io/welly-review-system/dashboard.html#editor=黃睿慈&mode=文章
```

直接貼這種連結給編輯，他們開就會看到自己對應的 view。

---

## 評分面板的 localStorage 行為

評分面板在使用者瀏覽器存以下資料（重新整理不會掉）：

- `rsp_article_name` / `rsp_doc_url` / `rsp_editor` / `rsp_reviewer`：表單值
- `rsp_mode`：上次選的模式（文章/架構）
- `rsp_counts_文章` / `rsp_counts_架構`：兩個模式各自的計次
- `rsp_other_文章` / `rsp_other_架構`：兩個模式各自的「其他」扣分項

評完一篇要記得按「清空計次」，否則下一篇會帶入上一篇的計次。

---

## 相關連結

- [審稿評分系統_資料庫_v3 Sheet](https://docs.google.com/spreadsheets/d/1sWt0CUDSBWzLmDNK1H2pwWNI0zhpvTnOByjZMI3P5gA/edit)
- [Welly Editor 同仁 GitHub 範例（client-notifier）](https://wellyeditor.github.io/welly-client-notifier/)
- 完整使用手冊：`../USAGE.md`
- SKILL 規格與對照表：`../SKILL.md`
