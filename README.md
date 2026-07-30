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
`CACHE = "ai-hub-v1.1.0"` 版號往上加（例如 `v1.1.1`），否則已安裝過的使用者會繼續看到舊版。

---

## 🔒 全站登入頁

`index.html` 一開啟就是一個全螢幕登入畫面（雙語、深淺色皆可切換），要輸入密碼才會顯示任何內容——
包含尚未輸入密碼時，頁面 DOM 裡也不會渲染教材清單，不是單純用 CSS 蓋住。

- **預設密碼：`proflolothebest`**
- 解鎖後這個瀏覽器會記住（存在 `localStorage`），下次造訪免輸入。
- 輸入錯誤會有漸進延遲（第 2 次錯 2 秒、第 3 次 4 秒…最高鎖 15 秒），友善勸退亂猜或機器人，
  但**不會**永久鎖死、也不會通報任何人。
- 想登出讓瀏覽器忘記密碼：右上角 🔒 按鈕（`index.html`／`admin.html` 都有）。
- `index.html` 和 `admin.html` 共用同一組密碼與同一個 `localStorage` 解鎖狀態，其中一頁解鎖過，
  另一頁也會自動略過登入畫面。
- 「課程營運與專題評量」分類**仍保留自己的第二道密碼**（見下方），兩道密碼互相獨立、疊加保護。
- 換密碼：`admin.html` 最上方「🔒 站台登入密碼」區塊 → 輸入新密碼 → 產生雜湊 → 下載並覆蓋
  `catalog.json`。清空後按「移除密碼」可讓整站免登入。
- 密碼設定存在 `catalog.json` 的 `meta.siteLock`，與分類密碼同一套 SHA-256 機制，
  同樣是**前端遮蔽而非真正的存取控制**（見下方安全說明）。

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
- **分類捲動列**：頁首下方一條可左右滑動的分類標籤列（仿「駱駱精選晨報」的作法），
  點擊會捲動到該分類，往下捲動時目前所在的分類會自動反白並置中——手機上左右滑、
  桌機上滾輪或拖曳皆可橫向捲動。
- **搜尋**：中英文皆可，多個關鍵字為 AND；按 `/` 聚焦搜尋框，`Esc` 清除。
- **篩選**：搜尋框旁的「篩選」按鈕展開類型（教材／工具／測驗／課程頁／研究／影片／成果）
  ＋ 對象（教師／學生／主管研究者）標籤，多選為 AND；有作用中的篩選時按鈕會顯示數字徽章。
- **安裝**：支援的瀏覽器會出現「安裝」按鈕；安裝後可離線開啟。
- **駱世民教授**：頁尾姓名列點擊後彈出視窗，內嵌 NCNU 國際企業學系系網的教師簡介頁
  （首次點擊才載入，右上角可另開新分頁或關閉）。

---

## 內容分類（8 類 / 51 項）

| # | 分類 | 項目數 | 密碼 |
|---|---|---|---|
| 1 | AI 基礎與工具生態 | 11 | — |
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
  例外是頁尾的 NCNU 系網教師簡介，該頁未設防嵌入標頭，故以彈出視窗內嵌。
- **無障礙**：略過導覽連結、`:focus-visible` 外框、`aria-pressed` 篩選狀態、
  `prefers-reduced-motion` 支援、鍵盤可完整操作（`Esc` 可關閉教授簡介彈窗）。

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
  "short": { "zh": "捲動列短標籤", "en": "Rail Label" },
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
