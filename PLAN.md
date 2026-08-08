# PMR Tool 互動式教學 — 規劃書

日期：2026-07-26（2026-07-30 依 app 現況修訂）　　狀態：開工中

## 目標

取代舊的螢幕錄影教學（`daniel0128-TW/PMR_Tool_Tutorial`，速度太快、字太小、且頁面已大改版），改成一份**單檔 HTML 互動教學**，參考 `projects/ntuh-helper-docs/tutorial.html` 的作法。

## 已定案的三個決定

| 項目 | 決定 |
|---|---|
| 部署 | 新開獨立 repo `pmr-tool-docs`（與 ntuh-helper-docs 平行，GitHub Pages），與 app 主 repo 解耦 |
| 呈現 | 純 CSS 手刻假 UI，無截圖。頁面改版只需改 HTML |
| 範圍 | 完整 8 章 |

## app 現況基準（2026-08-08，commit `01774d2`）

刻假 UI 一律以此為準，**不要照舊影片或舊記憶**。開工前先跑一次
`git log 01774d2..origin/main`——這個 repo 幾乎每天都在動。

- **`components/ui/` (shadcn) 全數刪除** → 純 Tailwind class，可直接照抄到教學的 CSS
- **三種狀態**（教學第 4 章的主軸）：
  - `null`＝**沒評**（預設，不進 note）
  - **確認正常**——`AllNormalButton`「全部正常 / 已全部標記正常」，**再按一次＝取消**，清回未評
  - **無法配合**——`QuestionRenderer` 的 `UserX` 鈕，只有 `COOPERATION_QUESTIONS` 那 13 題有（CN／DTR／MAS／FOIS／Sphincter／Barthel／mRS 沒有）。標記後表單換成虛線框「此項無法評估／病歷會印出 Unable to assess」
  - **GCS 建議**：`suggestUnableFromGCS`，門檻 E≤2／M≤5／V≤3，跳琥珀色卡片列出項目 ＋「一併標記為無法配合」一鍵套用。**只建議不自動寫入**
- **`ExpandableRow`**：每個量表項目收成一列（名稱＋現值格，未評顯示 `–`），點一下才在下方展開評分表；同時只開一列。MMT 的分頁鈕已移除 → **一題一頁**
- **PHI 政策已定**：只收病歷號（MRN）＋床號，前端無姓名欄位（`patients.name` 是歷史欄名，存的是 MRN）
- **閒置規則**：≥7 天出現「久未評估」警示、≥14 天卡片上出現紅 chip、**第 28 天清除**
- **訪客模式入口**：Dashboard 上的虛線鈕「訪客模式 · 不綁病人 · 自選模組 · 只產 note」，登入狀態可直接切過去不登出；登入者也可不建病人直接評估
- **Dashboard**：病房分組（4E1 / 4W1 / 其他）＋床號排序；header 兩個多選模式 `Email`（今日）與 `Weekly`
- **ASIA**：tap 扣分、長按 **550ms** 設 NT、格子 **44×44**、**10px 移動門檻**（把「按著捲動」判為無效，不觸發 tap 也不觸發長按）、`↓0` 批次歸零、本地 draft（重載不會清空 76 格）。**沒有 undo**（刻意 revert）→ 教學要寫「誤觸就再點回去」，不可寫「可復原」
- **ResidentGate**：14 個 step 的 reducer（`components/residentGateFlow.ts`，`STEP_BACK` 定義每步返回目標）

## 技術規格

- **單檔** `tutorial.html`：內嵌 CSS + 原生 JS，無外部相依、無建置步驟
- **視覺基底**沿用 ntuh-helper-docs：深色底 `#0f172a`、卡片 `#1e293b`、邊框 `#334155`、主色 `#2563eb`、step-rail 頂部導覽 + 底部 dots + 上/下一步
- **差異點**：NTUH 版用瀏覽器外框；PMR 是 mobile-first → 改做 **phone shell**（圓角 + 固定 375px 寬），內容用真 app 的淺色配色（白底 / `from-blue-50 to-white` 漸層）
- **假 UI 元件庫**（一次刻好、各章複用）：
  - `.phone` 手機外框、`.appbar` 頂部列
  - `.pcard` 病人卡片、`.qbtn` 選項鈕（照 `choiceClass`：`rounded-xl border-2 font-bold`，選中＝實色填滿白字）
  - `.erow` ExpandableRow（含右側現值格 `–`）、`.allnormal` 全部正常鈕
  - `.modbar` 模組色條（0 gray / 1 blue / 2 emerald / 3 violet / 4 amber，對齊 `lib/moduleTheme.ts`）
  - `.grid-asia` ASIA 三欄（左藍 / 中央節段＋解剖地標 / 右紅），格子 44×44
  - `.modal` 報表彈窗
- **互動**：模組多選、ExpandableRow 展開、ASIA 點擊扣分 / 長按 NT 做成**真的可點的假元件**
- **響應式**：手機時 step-rail 改橫向捲動

## 章節骨架（8 章）

| # | 章節 | 內容重點 | 比對來源 |
|---|---|---|---|
| 0 | 這是什麼 / 兩種模式怎麼選 | 訪客 vs 登入差別表（localStorage vs Postgres、有無病人管理與跨裝置）；登入後仍可一鍵切訪客 | `App.tsx`、`AccessKeyGate.tsx`、`Dashboard.tsx` |
| 1 | 第一次進入 | Access Key → 選人/註冊（Y1/Y2/R1–R4/VS + 姓氏）→ email → 4 位數 PIN；忘記 PIN 的 6 位數 OTP | `AccessKeyGate.tsx`、`ResidentGate.tsx`、`residentGateFlow.ts` |
| 2 | 訪客模式快速上手 | 不需登入、不上傳；病歷號選填、評估日期、今日已評估清單、刪除紀錄 | `pages/GuestHome.tsx` |
| 3 | 開一位病人（Dashboard） | 新增病人（Ward → 床號 → 病歷號，即時預覽 Bed ID）、病房分組、卡片資訊、7/14/28 天規則、Email/Weekly 多選模式 | `Dashboard.tsx`、`PatientFormModal.tsx`、`lib/bedId.ts` |
| 4 | 跑一次評估 | 入院完整評估 vs 自選模組；**ExpandableRow 操作法**、**null＝沒評 vs 全部正常**、進度條點開跳題、暫存、備註、Generate Note | `ModulePicker.tsx`、`Assessment.tsx`、`ExpandableRow.tsx`、`AllNormalButton.tsx`、`ProgressBar.tsx` |
| 5 | ASIA 專章 | C/T/L–S 分頁 + 上肢/下肢、從正常往下扣、tap 扣分 / 長按 550ms 設 NT、防誤觸門檻、`↓0` 批次歸零、5* 意義、worksheet 參考圖 | `ASIAAssessment.tsx`、`WorksheetRefModal.tsx`、`asiaData.ts`、`asiaAlgorithm.ts` |
| 6 | 產出報表 | Progress Note（兩次差異）、Compare 並排、Weekly Summary（6 日）、Today's Results；Copy / 下載 .txt / Email / 存 DB | `PatientHistory.tsx`、`ReportModal.tsx`、`generateNote.ts`、`compareScores.ts` |
| 7 | 常見問題 | 資料存多久、換裝置、離線、401 逾時、PHI（只填病歷號/床號）、ASIA 誤觸怎麼辦、Email 寄給誰 | `lib/api.ts`、schema 註解 |

## 執行階段

1. **Phase 0** — repo 骨架：`tutorial.html`（step-rail、panel 切換、鍵盤左右鍵）+ 8 個空章節
2. **Phase 1** — 假 UI 元件庫（phone shell、qbtn、erow、modbar、grid-asia、modal）
3. **Phase 2** — 章 0–3
4. **Phase 3** — 章 4–5（含可互動練習）
5. **Phase 4** — 章 6–7
6. **Phase 5** — 手機版檢查、README、`git init` + 推 GitHub + 開 Pages

## 待釐清

- 舊影片 repo 要 archive 還是加一行導向新教學
- 教學網址要不要回填到 app（需另開一輪改主 repo）

## 不做的事

- 不錄影片、不放截圖
- 不做多語系（純繁中）
- 不碰 PMR app 主 repo 的程式碼
