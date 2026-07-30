# AI 教學資源總入口 · AI Teaching Resources Hub

駱世民 教授（國立暨南國際大學國際企業學系）數位教材的單一入口。
雙語（中／EN／並列）、RWD、深淺色主題、可安裝為 PWA、支援離線瀏覽。

**線上網址｜Live:** https://lolopodcast.github.io/AI-Workshop-Hub/
**內容管理｜Manager:** https://lolopodcast.github.io/AI-Workshop-Hub/admin.html

---

## 檔案結構

| 檔案 | 用途 | 您平常會不會動到 |
|---|---|---|
| `catalog.json` | **所有教材清單與分類設定**。分類、項目、標籤、顯示開關、密碼雜湊都在這裡。 | 會（透過 `admin.html`） |
| `index.html` | 總入口頁面。程式碼與內容分離，不含任何教材清單。 | 通常不會 |
| `admin.html` | 圖形化編輯器。改完匯出新的 `catalog.json`。 | 會 |
| `manifest.json` | PWA 設定（名稱、圖示、捷徑）。 | 很少 |
| `sw.js` | Service worker，負責離線快取。 | 改 `index.html` 時要改版號 |
| `icons/` | PWA 圖示（192／512／maskable／apple-touch／favicon／SVG）。 | 很少 |

---

## 日常維護：增減或修改教材

1. 開啟 `admin.html`（線上或本機皆可）。
2. 展開分類 → 直接編修欄位；用 `↑ ↓` 調整順序；勾／取消「顯示」控制收放；
   `＋ 新增項目`／`刪除` 增減內容。
3. 按右上 **`⤓ 下載 catalog.json`**。
4. 把下載到的 `catalog.json` **覆蓋**本資料夾內的同名檔案。
5. 推上線：

```bash
git add catalog.json && git commit -m "Update catalog" && git push
```

> `admin.html` 只在瀏覽器裡運作，**不會自動寫回硬碟**。沒有執行第 4 步，改動不會生效。

### 只改 catalog.json 不必動 sw.js
`catalog.json` 在 service worker 裡走 network-first，內容更新會立刻反映。
但如果您修改了 `index.html`、`admin.html` 或 CSS，請同時把 `sw.js` 最上方的
`CACHE = "ai-hub-v1.0.0"` 版號往上加（例如 `v1.0.1`），否則已安裝過的使用者會繼續看到舊版。

---

## 顯示／隱藏 vs. 密碼保護

兩種收放機制，用途不同：

**`published: false`（顯示開關）** — 該分類或項目根本不會輸出到頁面，原始碼裡也看不到。
想暫時下架某份教材時用這個，這是唯一真正「藏得住」的方式。

**`lock`（分類密碼）** — 未解鎖時顯示密碼輸入框；密碼以 SHA-256 雜湊存在 `catalog.json`，
解鎖狀態記在瀏覽器 `localStorage`。

> ⚠️ **重要安全說明**
> 靜態網站的密碼屬於**前端遮蔽**，不是存取控制。教材清單與網址就在 `catalog.json` 裡，
> 任何人都能直接讀取；懂技術的人可以繞過密碼框。
> 它足以擋掉一般訪客隨手點開，但**請勿在受密碼保護的分類放真正機密資料**
> （個資、成績、未公開文件）。真要限定特定人才能看，請改用 `published: false` 完全下架，
> 或改採伺服器端驗證（Cloudflare Access、Netlify 密碼保護等）。

### 目前設定
- 「課程營運與專題評量」已設密碼，因為含班級側寫分析。
- 預設密碼：**`NCNU2026`**
- 換密碼：`admin.html` → 展開該分類 → 輸入新密碼 → 按「產生雜湊」→ 下載並覆蓋 `catalog.json`。

雜湊公式為 `SHA-256(分類ID + ":" + 密碼)`。也可用指令自行計算：

```bash
printf '%s' 'course-ops:您的新密碼' | openssl dgst -sha256
```

---

## 本機預覽

必須用伺服器開啟（`file://` 會讓瀏覽器擋掉 `fetch("catalog.json")` 與 service worker）：

```bash
python -m http.server 8765
```

然後開 http://localhost:8765

---

## 使用者可用的操作

- **語言**：右上按鈕循環切換 中／EN → 中文 → EN。選擇會記住。
- **主題**：深淺色切換，預設跟隨系統。選擇會記住。
- **搜尋**：中英文皆可，多個關鍵字為 AND；按 `/` 聚焦搜尋框，`Esc` 清除。
- **標籤篩選**：類型（教材／工具／測驗／課程頁／研究／影片／成果）
  ＋ 對象（教師／學生／主管研究者），多選為 AND。
- **安裝**：支援的瀏覽器會出現「安裝」按鈕；安裝後可離線開啟。

---

## 內容分類（8 類 / 49 項）

| # | 分類 | 項目數 | 密碼 |
|---|---|---|---|
| 1 | AI 基礎與工具生態 | 9 | — |
| 2 | 高教轉型與組織變革 | 5 | — |
| 3 | 學習、認知與動機 | 5 | — |
| 4 | 生涯發展與創業 | 4 | — |
| 5 | 社會、科技與風險 | 3 | — |
| 6 | EMI/EML 工作坊與教學工具 | 9 | — |
| 7 | 課程營運與專題評量 | 7 | 🔒 |
| 8 | 產品與專案實作 | 7 | — |

---

## 設計說明

- **配色**與既有 48 份教材一致：深綠 `#004030`、沙色 `#D8C3A5`；
  字體 Merriweather（標題）＋ Inter／Noto Sans TC（正文）。
- **不使用任何 CDN 執行程式庫**（無 React／Tailwind／Babel）。純原生 JS 與手寫 CSS，
  因此不會因 CDN 版本變動而白畫面，離線也能完整運作。
  唯一外部資源是 Google Fonts，載入失敗會自動退回系統字體。
- **教材一律開新視窗連出**（`target="_blank" rel="noopener noreferrer"`），
  不使用 iframe 內嵌——既有教材頁內建網域與框架防護，內嵌會顯示 `Frame Blocked`。
- **無障礙**：略過導覽連結、`:focus-visible` 外框、`aria-pressed` 篩選狀態、
  `prefers-reduced-motion` 支援、鍵盤可完整操作。

---

## 新增一個分類

`admin.html` → `＋ 新增分類`，或直接在 `catalog.json` 的 `categories` 陣列加一筆：

```json
{
  "id": "my-category",
  "order": 9,
  "published": true,
  "icon": "◆",
  "name": { "zh": "分類名稱", "en": "Category Name" },
  "desc": { "zh": "一句話說明。", "en": "One-line description." },
  "lock": null,
  "items": [
    {
      "id": "my-item",
      "published": true,
      "title": { "zh": "教材名稱", "en": "Material Title" },
      "desc": { "zh": "說明。", "en": "Description." },
      "url": "https://example.github.io/my-material/",
      "tags": ["material", "teacher"]
    }
  ]
}
```

可用標籤：`material` `tool` `quiz` `course` `research` `video` `showcase`
（類型）／ `teacher` `student` `exec`（對象）。新標籤請同時在 `tagLabels` 加中英文對照。
