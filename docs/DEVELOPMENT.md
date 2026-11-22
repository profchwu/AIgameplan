# 開發過程與技術說明

這份文件記錄「AI 教育遊戲開發任務地圖」從需求到上線的完整過程、每次修改的原因，以及程式架構。目的是讓之後接手的人（或 ChatGPT 等 AI）不用重看對話，就能理解並繼續優化。

- 網站：https://profchwu.github.io/AIgameplan/
- 課程教材：https://profchwu.github.io/AIgameplan/course.html
- 原始碼：網站本體只有一個檔案 `index.html`（HTML＋CSS＋JavaScript，約 60 KB，不需要建置工具）
- 對應的提示詞原文：[PROMPTS.md](PROMPTS.md)
- 交給 ChatGPT 優化的提示詞：[CHATGPT-OPTIMIZE.md](CHATGPT-OPTIMIZE.md)

---

## 1. 開發時間軸

| # | 需求（使用者） | AI 的處理 | 產出 |
| --- | --- | --- | --- |
| 1 | 規劃 7 步驟的 AI 教育遊戲 checklist 學習單 | 每步驟設計「填寫表格＋檢核清單」 | 線上文件版學習單 |
| 2 | 找出不足；列出所有遊戲類型與風格；每步驟附可複製的 prompt | 補上「試玩修正」「成效分析」兩步；遊戲類型 52 種、風格 44 種；每步驟加 prompt 程式碼框 | 學習單擴充為 9 步驟 |
| 3 | 加入學術問卷（Likert、中英對照、引用來源），分專家 UI/UX 與學生認知／情意／技能 | 先上網查證文獻（EGameFlow、MEEGA+、GEQ、認知負荷量表等），再整理 9 組量表與 APA 參考文獻 | 學習單新增 4-3 問卷評量工具 |
| 4 | 做成互動網站 | 單一 HTML 檔：關卡地圖、可勾選、填寫內容自動帶入 prompt、一鍵複製、localStorage 自動儲存 | 互動網站 v1 |
| 5–6 | 部署到 GitHub | GitHub 連接器沒有寫入權限，改用使用者已登入的瀏覽器在 GitHub 網頁上傳檔案，再開啟 GitHub Pages | profchwu.github.io/AIgameplan |
| 7 | 遊戲類型太少（RPG、射擊、3D 島嶼…） | 新增 41 種主流電玩類型 | 遊戲類型共 92 種 |
| 8 | 問卷與發布平台分開；一關一頁 | 問卷獨立成第 5 關（共 10 關）；改成用網址 #s1–#s10 切換頁面，每頁有上一關／下一關 | 網站 v2 |
| 9 | 問卷要分大類、視覺化，每個量表附原始與近期論文 APA 引用，可複製 | 量表分 6 大類 12 個，彩色卡片、底部固定選取列；逐筆上網查證近期驗證／回顧論文 | 網站 v3 |
| 10 | finish it | README、學習單文件與網站內容同步 | — |
| 11 | banner 太高 | 標題與進度併成一行、地圖縮小、手機隱藏地圖；橫幅從約 440px 降到約 180px | 網站 v3.1 |
| 13 | 檢查可優化之處 | 用桌機與手機寬度逐頁檢測，列出 13 項建議 | 優化清單 |
| 14 | 列印另存、下載檔案、遊戲類型先分大類、上傳開發紀錄 | 新增 A4 列印版面、下載 .md／.json、匯入備份；遊戲類型改為「8 大類 → 細類 → 類型」三層；新增 docs/ 三份文件 | 網站 v4（本版） |
| 15–16 | 把開發過程做成上課用的教材，PPT 樣式、每頁圖文並呈、網頁形式 | 做成 16:9 網頁簡報：每頁一張自繪插圖（SVG）＋重點文字，支援方向鍵／滑動換頁、全螢幕、總覽、列印成 PDF；內容涵蓋四階段的提示詞原文、提示詞技巧、查證與倫理、課堂活動與評量規準 | course.html |

## 2. 重要設計決定

- **單一 HTML 檔**：老師不需要會 npm 或建置工具，GitHub 網頁上傳一個檔案就能更新。
- **資料只存在瀏覽器（localStorage，鍵名 `aiEduGame2`）**：不收集學生個資、不需後端。代價是換裝置會遺失，所以加了下載／匯入備份檔。
- **一關一頁用網址 hash**（`#s1`～`#s10`、`#finish`）：可以直接分享某一關的連結，瀏覽器上一頁也能用。
- **Prompt 自動帶入**：使用者在第 1 關填的年級、單元、目標等，會用 `v(key, 預設提示)` 帶進每一關的 prompt；沒填的部分以黃色【】標示，提醒自行替換。
- **問卷題目的著作權**：只有 SUS 使用原題（作者允許免費使用）；其他量表都是依構面改編的示例題，並在頁面註明「正式研究請取得原始量表」。
- **文獻查證**：近期文獻都經過網路查證；技能類量表沒有找到可靠的近期驗證論文，因此只列經典文獻。

## 3. 程式架構（index.html）

```
<head>
  <style> 精簡橫幅覆寫樣式
  <style> 主樣式（CSS 變數、深淺色主題、各元件、@media print 列印版面）
<body>
  header      標題、完成度、關卡地圖（SVG）
  nav.tabs    10 關＋匯出／列印的分頁列
  main#steps  由 JavaScript 依 STEPS 產生 10 個 section
  #finish     匯出、列印、下載、匯入、清除
  #printView  列印時才顯示的學習單版面
  <script>    全部程式
```

### 主要資料陣列（要改內容，改這些就好）

| 名稱 | 內容 | 格式 |
| --- | --- | --- |
| `GAMES` | 92 種遊戲類型 | `[細類, 名稱, 適合的學習目標, 難度1–3]` |
| `SUPER` | 8 個遊戲大類 | `[大類名稱, 說明, 顏色, [包含的細類…]]` |
| `STYLES` | 44 種視覺風格 | `[大類, 名稱, 英文關鍵字, 適合對象]` |
| `STORE` | 後台儲存方式 | `[id, 名稱, 說明]` |
| `PLAT` | 發布平台 | `[名稱, 說明]` |
| `CATS` | 問卷 6 大類 | `[id, 名稱, 顏色]` |
| `INST` | 12 個量表 | `{id, cat, name, purpose, who, when, scale, orig:[APA], recent:[APA], items:[[構面, 中文題, 英文題]]}` |
| `STEPS` | 10 個關卡 | `{id, short, title, lead, fields, checks, prompts, custom}` |

- `fields`：`[鍵名, 標籤, 提示文字, 'w'(多行)]`
- `prompts`：`[標題, () => 樣板字串]`，樣板內用 `v('鍵名','預設提示')` 帶入填寫內容
- `custom`：該關的特殊區塊（`games`、`store`、`survey`、`styles`、`plat`、`kirk`）

### 主要函式

| 函式 | 作用 |
| --- | --- |
| `save()` | 存到 localStorage |
| `renderGames()` / `renderStyles()` / `renderInst()` | 重繪遊戲類型、風格、問卷區 |
| `renderPrompts()` | 依填寫內容更新所有 prompt |
| `drawMap()` / `progress()` | 關卡地圖與完成度 |
| `show()` | 依網址 hash 切換頁面 |
| `allText()` | 產生 Markdown 版完整規劃（複製與下載共用） |
| `printHTML()` / `doPrint()` | 產生列印版面並呼叫列印 |
| `dl()` | 下載檔案 |

## 4. 如何更新網站

1. 修改本機的 `index.html`（或請 AI 修改）。
2. 到 GitHub 的 repository → 點 `index.html` → 右上角鉛筆圖示（Edit）→ 全選貼上新內容 → Commit changes。
   或在 repository 首頁按 Add file → Upload files，拖入新檔覆蓋。
3. 等 1–2 分鐘，GitHub Pages 會自動更新。手機若看到舊版，重新整理或在網址後加 `?v=新數字`。

## 5. 已知限制與待辦

- 技能類量表缺近期驗證文獻。
- 下列文獻沒有 DOI（書籍、早期會議或報告）：Bloom (1956)、Brooke (1996)、Dave (1970)、Simpson (1972)、Harrow (1972)、IJsselsteijn et al. (2013)、Kirkpatrick & Kirkpatrick (2006)、Likert (1932)；Lynn (1986) 的 DOI 尚未查證，暫不列出。其餘文獻均已附 DOI。
- 量表點數在 prompt 中固定寫 5 點，但部分量表原本為 7 點或 0–10 分。
- 遊戲類型與視覺風格目前只能單選（尚無「主類型＋輔助類型」）。
- 頁面缺少社群分享預覽（og:image、description）與網站小圖示。
- 深色模式下問卷分類的藍、紫色對比偏低。
- 部分按鈕（分類篩選、文獻複製）在手機上高度低於 44px。
- 在 Claude 內嵌預覽中無法列印與下載（沙盒限制）；在 GitHub Pages 網站上正常。
