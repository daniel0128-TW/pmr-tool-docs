# PMR Tool Docs

[PM&R Bedside Assessment Pro](https://github.com/daniel0128-TW/PMR_Tool_by_PYT) 的互動式使用教學。

- `tutorial.html` — 單檔互動教學（8 章），無外部相依、無建置步驟，直接用瀏覽器打開即可
- `PLAN.md` — 規劃書與 app 現況基準（改教學前先讀）

取代舊的螢幕錄影教學（`PMR_Tool_Tutorial`）：影片速度太快、字太小，且 app 已大幅改版。

## 維護

教學裡的手機畫面是**純 CSS 手刻**的，不是截圖——app 改版時只要改 `tutorial.html` 的 HTML，不用重錄或重截。

刻假 UI 時對照 app 的這幾個檔案：

| 教學章節 | 對照來源 |
|---|---|
| 0 模式選擇 | `App.tsx`、`AccessKeyGate.tsx` |
| 1 登入 | `ResidentGate.tsx`、`residentGateFlow.ts` |
| 2 訪客模式 | `pages/GuestHome.tsx` |
| 3 病人管理 | `pages/Dashboard.tsx`、`PatientFormModal.tsx` |
| 4 評估 | `Assessment.tsx`、`ExpandableRow.tsx`、`AllNormalButton.tsx` |
| 5 ASIA | `pages/ASIAAssessment.tsx`、`WorksheetRefModal.tsx` |
| 6 報表 | `PatientHistory.tsx`、`ReportModal.tsx` |

配色以 `lib/moduleTheme.ts`（模組 0–4：gray / blue / emerald / violet / amber）與 `lib/questionStyles.ts` 的 `choiceClass` 為準。
