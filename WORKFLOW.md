# Vocabulary Architect Workflow

本文件定義 Vocabulary Architect 專案新增教材資料、更新網站、QA 審核與部署的標準工作流程。

所有 Agent 在執行本專案任務時，應同時遵守：

- `AGENTS.md`
- `WORKFLOW.md`

## 1. 專案分支原則

### main

- 保留穩定版本。
- 不直接修改。
- 不作為日常開發分支。
- 除非教師明確同意，不得 Merge 新功能或新教材資料。

### jesamine-cpu-patch-1

- 作為目前主要開發與測試分支。
- 建置 Agent 的更新應先進入此分支。
- QA Agent 只審核此分支最新版本。
- Vercel Production 目前追蹤此分支。
- QA 未通過前不得視為正式完成。

## 2. 新增教材資料的標準流程

例如新增 B2 七下、B4 八下、B6 九下，必須依以下順序進行。

### Step 1｜提供原始教材

教師提供教材來源，例如 PDF、Excel、CSV、Word、JSON 或其他原始教材資料。

建置 Agent 不可直接匯入網站。

### Step 2｜匯入前整理

建置 Agent 必須先將教材整理成與 `moe_vocab_data.json` 相同的資料格式。

至少檢查：

- `book`
- `book_code`
- `unit`
- `section`
- `number`
- `word`
- `part_of_speech`
- `syllables`
- `phonetic`
- `meaning`
- `source_page`

不得自行捏造教材沒有提供的內容。無法確認的項目標記「需教師確認」。

### Step 3｜匯入前 QA

在修改正式資料之前，建置 Agent 必須先產生資料整理報告。

報告至少包含：

- 各冊筆數與各 Unit 筆數
- 重複資料、空白欄位、拼字異常、中文異常、詞性缺漏
- 音標異常、音節異常、片語、句子、縮寫
- 含括號、連字號、撇號的項目
- 需教師確認項目

教師確認後，才進入正式合併。

## 3. 音節規則

`syllables` 欄位以實際英文發音為準，`/` 代表實際發音的音節邊界。

不得平均切字母、只依拼字規則推測或任意依母音數量切割。

已確認範例：

- `very` → `ver/y` → 2 syllables
- `really` → `real/ly` → 2 syllables
- `cousin` → `cous/in` → 2 syllables
- `favorite` → `fa/vor/ite` → 3 syllables
- `camera` → `cam/er/a` → 3 syllables
- `restroom` → `rest/room` → 2 syllables
- `T-shirt` → `T/shirt` → 2 syllables
- `drawer` → `drawer` → 1 syllable

若權威字典存在合理差異，標記「需教師確認」，不得自行選定。

## 4. 正式合併資料

教師確認資料後，建置 Agent 才能更新 `moe_vocab_data.json`。此檔案是唯一正式單字資料來源。

更新時必須：

1. 保留既有冊別資料。
2. 只新增或修改本次教師確認的資料。
3. 不得意外改動其他冊別。
4. 執行 JSON.parse 驗證。
5. 核對總筆數、各冊筆數與各 Unit 筆數。
6. 檢查重複與缺漏資料。

## 5. index.html 修改規則

如新增冊別需要修改網站：

- 採最小幅度修改，不得重寫整個網站。
- 不得改變既有 UI 與操作流程，除非教師明確要求。
- 不得移除原有功能。
- 必須保留既有 49 個函式、T1–T7、教師編輯、PDF 匯入、語音辨識、語音播放、學生登入、localStorage、報告與 PDF 輸出。
- 若 `index.html` 刪除大量程式碼，Agent 必須停止並回報原因。

## 6. 建置 Agent 交付格式

每次建置完成後，建置 Agent 必須建立乾淨上傳資料夾，例如 `github-patch-1-upload`。

資料夾內只能放本次真正需要上傳 GitHub 的正式檔案，例如：

- `index.html`
- `moe_vocab_data.json`
- `AGENTS.md`
- `WORKFLOW.md`

若某檔案本次沒有修改，就不需重複上傳。

不得放入 QA 報告、CSV 檢查表、Markdown 修正清單、ZIP 備份、臨時測試檔、舊版本或重複資料來源。

## 7. GitHub 更新流程

所有更新先進入 `jesamine-cpu-patch-1`：

建置 Agent
→ 產生乾淨上傳資料夾
→ 教師上傳 GitHub patch-1
→ Commit
→ QA Agent 審核
→ 建置 Agent 修正
→ QA Agent 複驗
→ 通過
→ Vercel 自動部署

不得直接修改 `main`。

## 8. QA Agent 工作範圍

QA Agent 是獨立審核者，不得修改檔案、Commit、建立 Pull Request、Merge 或自動修正資料。

QA Agent 可以讀取、分析、執行測試、比較版本、找出錯誤、提出建議及產生 QA 報告。

## 9. QA 檢查內容

每次新增教材至少檢查：

### 教材

- 英文拼字、中文、詞性、音標、音節數、音節切分位置
- 冊別、Unit、重複、缺漏、標點、大小寫
- 片語、句子、縮寫、複合詞、所有格、連字號、括號

### 程式

- JSON.parse、JavaScript 語法、資料是否能載入
- 冊別切換、Unit 切換、T1–T7、原有函式
- 教師編輯、PDF 匯入、語音功能、登入、localStorage、報告與 PDF 輸出

## 10. QA 結論

QA Agent 最後必須明確選擇：

### A. 通過

可以進入目前 Production 流程。

### B. 修正後可以

仍有明確必須修正項目。

### C. 暫緩

存在資料或程式風險，不建議部署。

若不是 A，QA Agent 必須列出必須修正與建議修正，且不得自行修改。

## 11. Vercel 部署

目前 Vercel Production Branch 是 `jesamine-cpu-patch-1`。

因此只要此分支出現新的 Commit，將由 Vercel 自動部署：

GitHub patch-1 → Vercel 自動部署 → Building → Ready

不需要每次重新建立 Vercel Project、Import Repository、設定 GitHub 權限或設定 Production Branch。

## 12. 上線後驗收

每次 Vercel 顯示 Ready 後，教師至少抽查：

- 新增冊別是否出現
- Unit 是否正確
- 隨機單字與已修正音節
- T1–T7 至少一項
- 登入、localStorage、語音功能

若發現問題，不得直接修改 Production，必須回到：

建置 Agent → patch-1 → QA → Vercel

## 13. 建議的新教材指令

未來教師可以對建置 Agent 說：

> 請依 AGENTS.md 與 WORKFLOW.md，匯入 B2、B4、B6 教材。先做匯入前 QA，不要直接修改網站。

建置 Agent 應自動依本流程執行。

## 14. 核心原則

專案優先順序：

1. 教材正確
2. 原功能不被破壞
3. 修改可追蹤
4. QA 獨立
5. 教師最後確認
6. 再部署
