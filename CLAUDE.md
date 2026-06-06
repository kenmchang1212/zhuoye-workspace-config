# Project 工作區共用規則

## 🚦 程式開發強制閘門（Gate 1-5）

每個程式任務開始必評估 Gate 1-5；符合觸發條件才執行。自行讀取、自行最優判斷、直接輸出結論。先跑 Gate，Gate 可能改變任務等級，再判定最終等級與角色鏈。跳過任一 Gate 直接修改 → A0 強制介入。

| Gate | 觸發條件 | 動作 | 詳見 |
|------|----------|------|------|
| Gate 1 Context Pack | 業務規則/金額/統計/客戶/CPA/時數/DB/API/部署/Webhook/DNS/中修以上 | 讀專案文件，5行內輸出摘要，未完成禁止寫程式 | [Context Pack](#-context-pack) |
| Gate 2 Source Check | 涉及外部平台（Coolify/LINE/Cloudflare/Netlify/monday.com等） | 查官方文件，輸出查證來源，查不到禁改高風險設定 | [Source Check](#-source-check-gate) |
| Gate 3 Incident | 部署/Webhook/DNS/外部平台失敗≥1次 | 停止quick fix，A7全局診斷，輸出根因+一次性修復計畫 | [A7 Incident Mode](#-a7-incident-mode) |
| Gate 4 資安 | 登入/權限/token/Webhook驗證/個資/金鑰 | 啟動A6，Critical/High未解前禁止commit/push | [A6 角色表](#-a0-a9-角色與決策邊界) |
| Gate 5 A0 | 新功能/同功能≥2次/跨≥5檔案/前端+API+DB/A2重複警告/A6/A7/A9提Critical/High/需求不明 | 停止開發，交A0重設需求 | [A0 需求閘門](#-a0-需求閘門) |

---

## 🧠 Context Pack（Gate 1）

中修以上或涉及業務規則/資料庫/API/部署/Webhook/DNS/外部平台時觸發。自行讀取、自行判斷、直接輸出摘要，未完成不得修改檔案。

讀取順序：Project/CLAUDE.md → 專案 CLAUDE.md → DOMAIN_RULES.md → PROJECT_CONTEXT.md → CURRENT_TASK.md → KNOWN_ISSUES.md → 部署時加讀 COOLIFY.md

輸出（5 行內）：`專案 / 本次目標 / 必讀規則來源 / 不可破壞事項 / 已知風險`

claude-mem 只作輔助索引，不得為業務規則唯一來源。文件與官方文件或實際程式衝突 → 標記並更新，不得忽略。

---

## 🚨 A7 Incident Mode（Gate 3）

觸發：Coolify/Netlify 部署失敗、Webhook 失敗、DNS/SSL/Domain 失敗、外部平台 API 失敗、同一錯誤修過仍失敗。

1. 立即停止 quick fix，禁止直接改程式碼
2. A7 收集證據，輸出 Incident Report：

```
A7 Incident Report
1. 失敗類型：
2. 影響專案/服務：
3. 部署平台與 Domain：
4. 第一個真正錯誤：
5. 相關 log 證據：
6. 已檢查：build / env / db / domain:DNS / webhook:callback / Dockerfile:Nixpacks / COOLIFY.md / 官方文件
7. 根因判斷：
8. 不該做的 quick fix：
9. 一次性修復計畫：
10. 驗證方式：
```

限制：不得只根據最後一行錯誤修改、未讀 COOLIFY.md 不得改部署、未確認根因不得連續部署、第二次仍失敗重出完整報告不得繼續猜。完成 Incident Report 並確認根因後，才可依一次性修復計畫修改。

---

## 🔎 Source Check Gate（Gate 2）

涉及 Coolify/LINE/Cloudflare/Netlify/monday.com 等程式相關第三方平台，修改前必做：

```
Source Check：
- 平台：
- 查證日期：
- 文件來源：
- 相關規則：
- 與專案文件衝突：
- 結論：
```

優先查官方文件，其次專案文件。衝突→以最新官方文件為準並更新專案文件。無法查→標「⚠️ 無文件佐證」，不得改高風險設定。不得用「我記得」「通常是」當依據。

---

## 🎯 工作模式與風險分級

先跑 Gate（可能改變等級），再判定最終等級與角色鏈。使用者關鍵字只作為初始等級；強制升級規則永遠優先。未指定時依風險自行判定。

| 關鍵字 | 等級 | 角色鏈 |
|--------|------|--------|
| 小修（初級修改） | S | A1 → A2自檢 → commit |
| 中修（中級修改） | M | Context Pack → A1 → A2 → 必要時A3 → commit |
| 大修（高級修改） | L/XL | Context Pack → 依風險自動判定等級與角色鏈 |

強制升級（任一至少 L）：
- 登入/權限/token/Webhook驗證/個資 → 至少 L，啟動 A6
- Coolify/DNS/Webhook/外部平台設定 → 至少 L，啟動 Source Check；失敗啟動 A7
- 業務計算/客戶分類/CPA/時數/金額統計 → 至少 L，必讀 DOMAIN_RULES.md
- 同功能修改 ≥2 次 → 啟動 A0；≥3 次未收斂 → 自動 A9

角色鏈：
- S：A1 → A2自檢 → commit
- M：Context Pack → A1 → A2 → 必要時A3 → commit
- L：Context Pack → A1 → A2 → A4 → 必要時A5a/A6/A7/A8 → commit
- XL：A0 → Context Pack → A1 → A2 → A3 → A4 → A5a → A5b → A6 → A7 → A8 → 必要時A9 → commit

A2自檢範圍：syntax/type error、build error、console error、基本合理性。禁止全專案 review。

---

## 🤖 A0-A9 角色與決策邊界

同一問題只有一個決策 owner。其他角色只提風險或證據，不得越權。A1 不做產品/架構/安全/部署決策。

決策優先：A0 需求 > A6 資安 > A7 維運 > A4 架構 > A8 ROI > A2 程式正確性 > A3 UI > A5a/A5b 流程文件 > A1 實作
衝突：需求→A0、安全→A6、部署→A7、架構→A4、ROI→A8

| 角色 | Owner 範圍 | 可做 | 禁止 |
|---|---|---|---|
| A0 需求閘門 | 需求範圍、成功標準、不做事項 | 停止任務、縮小範圍、要求PRD、呼叫/grill-with-docs或/to-prd | 寫程式、指定低階技術細節 |
| A1 開發 | 依 owner 結論精準實作 | 修正已確認問題、修根因所需小範圍連帶修改（先說明原因） | 順手重構/加功能/改架構、未經A7改部署、未經Source Check改外部平台 |
| A2 程式審查 | bug、邏輯錯誤、會壞的重複程式 | 指出會造成錯誤的問題 | 審架構(A4)/安全(A6)/UI(A3)、建議重構或抽象化 |
| A3 UI 審查 | 美感、一致性、資訊密度 | 只報Critical/High。格式：`[元件] 在 [頁面] [問題描述]，建議 [修正]`（附檔案:行號、影響範圍） | 擴大功能、改業務邏輯 |
| A4 架構審查 | data flow、service boundary、DB結構、模組耦合 | 決定架構邊界。需求不清時呼叫A0 | 建議K8s/微服務/新語言/新DB/訊息佇列 |
| A5a 流程審查 | 流程合理性、使用者體驗 | L+觸發，A2/A4後執行：誰會用？太複雜？ | 決定部署修法、改業務決策 |
| A5b 文件記憶 | 文件維護、Context Pack導航 | 任務前載入、任務後更新 | 用claude-mem取代專案文件 |
| A6 資安 | 權限、token、個資、Webhook驗證、資料存取 | 阻擋Critical/High。四級：Critical（權限漏洞/API Key外洩/SQL injection→立即停止）> High（敏感資料未加密/缺輸入驗證）> Medium（已知漏洞/CSP不足）> Low（資訊揭露） | 為方便降低安全 |
| A7 維運部署 | Coolify、DNS、env、build、db部署、Webhook可用性 | Incident Mode、根因診斷。輕量原則：單一VPS、確保備份/硬碟/重啟，不建議K8s/微服務/多雲 | 未診斷就quick fix、未讀COOLIFY.md就改部署 |
| A8 老闆顧問 | ROI、效率、客戶滿意度 | 砍低ROI新增功能 | 否決bug/資安/資料完整性修復、否決使用者明確要求的必要功能 |
| A9 外部顧問 | 外部獨立審查、卡關校正 | 提Critical/High可暫停任務。三模式：`/a9檢查`（高性價比）、`/a9審查`（最高品質）、`/a9討論`（非程式多輪討論）。回覆強制標 `🤖 A9（OpenAI）：`。自動觸發：同功能≥3次未收斂。fallback：API不可用→DeepSeek替代，標`⚠️ A9 fallback`。DeepSeek收到回覆後須四級分類（🔴必要/🟡建議/🔵較好/⚪不改），暫停等使用者確認後才執行 | 取代A0/A4/A6/A7的owner決策 |

### A1 3-Strike Error Protocol
同一錯誤 → 第1次診斷修復 → 第2次換方法 → 第3次重新思考 → 仍失敗交使用者判斷。

---

## 🛑 A0 需求閘門（Gate 5）

觸發（任一即停）：新功能/同功能修改≥2次/跨≥5檔案/同時影響前端+API+DB/A2重複警告≥2次/A6/A7/A9提Critical/High/需求或資料源不明。

1. 停止A1開發
2. A0讀Context Pack，只確認業務決策/需求邊界/成功標準/資料來源，不問低階技術
3. 必要時呼叫/grill-with-docs或/to-prd
4. 輸出六選一：繼續/先釐清/先產PRD/縮小範圍/砍掉重做/暫停
5. 只有「繼續」後A1才能開發

---

## ✅ Definition of Done

1. 已完成必要Gate（Context Pack/Source Check/Incident Mode/Security Gate/A0）
2. 已驗證：syntax/type/build檢查、必要測試、UI確認、部署確認
3. 已更新文件：CHANGE_LOG_AI.md、必要時CURRENT_TASK.md/KNOWN_ISSUES.md/DOMAIN_RULES.md/COOLIFY.md
4. 已輸出完成摘要：改什麼/驗證結果/commit:push狀態/殘留風險

有blocking error不得宣稱完成。未驗證部署不得說部署完成。

---

## 部署平台

- VPS：阿里雲國際版香港 4C8G
- 部署工具：Coolify（dashboard.zhuoye.com.tw:8000）
- DNS：Cloudflare

1. 涉及部署/Domain/SSL/環境變數/Webhook/Dockerfile/Nixpacks 時，必須先讀：
   - 工作區 `COOLIFY.md`
   - 該專案 `<專案>/COOLIFY.md`
   - 該專案 `PROJECT_CONTEXT.md`
2. 部署失敗 → Gate 3 觸發 A7 Incident Mode
3. 修復後必須更新該專案 `COOLIFY.md` 或 `KNOWN_ISSUES.md`

---

## 共用技術棧偏好

- 前端：Next.js 14 + TypeScript + Tailwind CSS
- 資料庫：SQLite（輕量）或 PostgreSQL（多用戶/需權控）
- ORM：Prisma（預設 prisma db push；既有正式資料庫變更需先確認備份與風險）
- UI：shadcn/ui + Recharts
- 新專案預設從此技術棧開始，有理由才偏離

---

## 所有專案

| 專案 | 目錄 | Domain | 部署 | 說明 |
|---|---|---|---|---|
| Monday 儀表板 | `monday-dashboard/` | dashboard.zhuoye.com.tw | Coolify | 工作活動/案件/客戶貢獻三主體 |
| 卓燁 LINE OA | `zhuoye-line/` | line-admin.zhuoye.com.tw, api.zhuoye.com.tw | Coolify | LINE Bot 後台「小卓」 |
| 卓燁官網 | `ZHUOYE/hugo-site/` | zhuoye.com.tw | Netlify | Hugo 靜態官網 |

---

## 🧭 Gate 1 專案上下文入口

進入任一專案時，先依本表讀取對應文件。不得只靠記憶或 claude-mem 判斷業務規則。

| 專案 | 何時必讀 | 必讀文件 | 特別注意 |
|---|---|---|---|
| Monday 儀表板 `monday-dashboard/` | 涉及客戶貢獻、工作活動、案件、客戶分類、人員時數、統計報表、monday.com API | `monday-dashboard/CLAUDE.md`、`monday-dashboard/DOMAIN_RULES.md`、`monday-dashboard/PROJECT_CONTEXT.md`、`monday-dashboard/KNOWN_ISSUES.md` | 客戶貢獻監控不得憑記憶判斷；業務規則、時數規則、客戶分類邏輯、monday 欄位 mapping 一律以 `DOMAIN_RULES.md` 為準 |
| 卓燁 LINE OA `zhuoye-line/` | 涉及 LINE Bot、Webhook、訊息格式、Rich Menu、API、後台權限 | `zhuoye-line/CLAUDE.md`、`zhuoye-line/DOMAIN_RULES.md`、`zhuoye-line/PROJECT_CONTEXT.md`、`zhuoye-line/COOLIFY.md` | LINE API/Webhook 修改前必須做 Source Check |
| 卓燁官網 `ZHUOYE/hugo-site/` | 涉及官網內容、SEO、表單、Netlify 部署 | `ZHUOYE/hugo-site/CLAUDE.md`、`ZHUOYE/hugo-site/PROJECT_CONTEXT.md` | Netlify 設定修改前必須做 Source Check |

若必讀文件不存在：
1. 不得假設規則不存在
2. 先回報缺少哪份文件
3. 若任務會產生新規則，完成後建立或補齊文件

---

## 部署相關文件

- 通用 Coolify 知識：`COOLIFY.md`（Dockerfile 模板、Nixpacks vs Dockerfile、踩坑全記錄）
- 各專案 Coolify 步驟：`<專案>/COOLIFY.md`
- 備份/還原：`BACKUP.md`、`RESTORE.md`

---

## 📌 業務規則存放原則

- 全域 `~/.claude/CLAUDE.md` 只放工作流程與通用規則，不放單一專案業務規則
- `~/Project/CLAUDE.md` 放程式開發規則與專案入口，不放詳細計算邏輯
- 詳細業務規則一律放在各專案內：`DOMAIN_RULES.md`、`PROJECT_CONTEXT.md`、專案自己的 `CLAUDE.md`

Claude 不得因為全域或 Project 層文件沒寫到某業務規則，就假設該規則不存在。

---

## 🆕 新專案啟動

多檔案/需設計/涉及 UI → 禁止直接寫程式：brainstorming → writing-plans → 視覺用 impeccable/frontend-design → 確認後才 executing-plans。單一檔案/純資料處理不在此限。

1. 確認需求（全域 brainstorming → writing-plans）
2. 在 `~/Project/` 下建立目錄
3. 技術棧預設使用上方共用偏好
4. 若需部署 → 建立 `<專案>/COOLIFY.md`（參考 `COOLIFY.md` 模板）
5. 寫 `<專案>/CLAUDE.md`（只寫該專案獨有規格）

---

## 安全規則

- API Token/金鑰一律放 `.env`，已在 `.gitignore`
- 不提交任何 credentials 到 repo
- Node.js 專案部署前跑 `npm audit`

---

## 專案規格

- 卓燁 LINE OA 專案規格：見 `zhuoye-line/CLAUDE.md`
- Monday 儀表板專案規格：見 `monday-dashboard/CLAUDE.md`
