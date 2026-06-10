# Project 工作區共用規則

> 本檔是 `~/Project/` 下所有開發專案的共用開發規則，不代表某個名為 Project 的專案。各專案獨有規格請見該專案子目錄下的 CLAUDE.md。

## 🚦 程式開發強制閘門（Gate 1-5）

每個程式任務開始必評估 Gate 1-5（符合觸發條件才執行，自行判斷、直接輸出結論）。Gate 可能改變任務等級，先跑 Gate 再判定最終等級與角色鏈。跳過任一 Gate → A0 強制介入。

進入 Plan Mode 同樣強制，規劃文件須包含：Gate 1-5 判定表（逐一標註觸發與否+說明）+ 最終等級（S/M/L/XL）+ 角色鏈（逐一標註觸發原因）。未完成不得實作。適用所有專案，不得被專案層級文件覆蓋。

Gate 非固定順序，一般流程：Gate 0（提問）→ Gate 5（需求/A0）→ Gate 1（Context）→ Gate 2/3（視觸發）→ Gate 4/DoD（收尾）。

| Gate | 觸發條件 | 動作 | 詳見 |
|------|----------|------|------|
| Gate 1 Context Pack | 業務規則/金額/統計/客戶/CPA/時數/DB/API/部署/Webhook/DNS/中修以上 | 讀專案文件，5行內輸出摘要，未完成禁止寫程式 | [Context Pack](#-context-pack) |
| Gate 2 Source Check | 涉及外部平台（Coolify/LINE/Cloudflare/Netlify/monday.com等） | 查官方文件，輸出查證來源，查不到禁改高風險設定 | [Source Check](#-source-check-gate) |
| Gate 3 Incident | 部署/Webhook/DNS/外部平台失敗≥1次 | 停止quick fix，A7全局診斷，輸出根因+一次性修復計畫 | [A7 Incident Mode](#-a7-incident-mode) |
| Gate 4 資安 | 登入/權限/token/Webhook驗證/個資/金鑰 | 啟動A6，Critical/High未解前禁止commit/push | [A6 角色表](#-a0-a9-角色與決策邊界) |
| Gate 5 A0 | 新功能/同功能≥2次/跨≥5檔案/前端+API+DB/A2重複警告/A6/A7/A9提Critical/High/需求不明 | 停止開發，交A0重設需求 | [A0 需求閘門](#-a0-需求閘門) |

> Gate 5 是觸發 A0 的條件，不是決策。A0 是執行需求釐清與範圍控制的角色。Gate 5 問「是否需要 A0 介入」，A0 問「需求邊界/成功標準/資料來源」，兩者不重複。

---

## 🧠 Context Pack（Gate 1）

中修以上或涉及業務規則/資料庫/API/部署/Webhook/DNS/外部平台時觸發。未完成不得修改檔案。

讀取：本檔 → 專案 CLAUDE.md → DOMAIN_RULES.md → PROJECT_CONTEXT.md → CURRENT_TASK.md → KNOWN_ISSUES.md（部署加 COOLIFY.md）
輸出（5 行內）：`專案 / 本次目標 / 必讀規則來源 / 不可破壞事項 / 已知風險`
claude-mem 只作輔助索引，不為業務規則唯一來源。文件與官方文件/實際程式衝突 → 標記並更新。

### 各專案入口

| 專案 | 目錄 | Domain | 部署 | 必讀文件 | 特別注意 |
|------|------|--------|------|----------|---------|
| Monday 儀表板 | `monday-dashboard/` | dashboard.zhuoye.com.tw | Coolify | `CLAUDE.md` `DOMAIN_RULES.md` `PROJECT_CONTEXT.md` `KNOWN_ISSUES.md` | CPA/客戶貢獻/時數/欄位 mapping 以 DOMAIN_RULES.md 為準 |
| 卓燁 LINE OA | `zhuoye-line/` | line-admin.zhuoye.com.tw, api.zhuoye.com.tw | Coolify | `CLAUDE.md` `DOMAIN_RULES.md` `PROJECT_CONTEXT.md` `COOLIFY.md` | LINE API/Webhook 修改前必做 Source Check |
| 卓燁官網 | `ZHUOYE/hugo-site/` | zhuoye.com.tw | Cloudflare Pages | `CLAUDE.md` | Cloudflare Pages 設定修改前必做 Source Check |

必讀文件不存在時：不得假設規則不存在 → 回報缺少文件 → 任務若產生新規則，完成後補齊。

### CURRENT_TASK.md 永久待辦清單

不再做完清空，改為永久累積。規則：
- 列出建議清單時立即寫入，完成標 `[x]`、未完成留 `[ ]`
- 下次開啟只列未完成項；保留最近 10 次 session 完整記錄，超過自動清除
- 中修以上必更新

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

🎯 工作模式與風險分級（自動判定）

判定順序：先依下方條件初判 S/M/L/XL，再以強制升級規則檢查是否升等。強制升級規則觸發即覆蓋初判結果。

S：單檔案且全部符合
  - 不改 DB Schema / API 結構 / UI 結構
  - 角色鏈：A1 → A2 → commit

M：任一條件
  - 2~5 個檔案
  - 新增頁面
  - 修改 UI 結構（component/shared layout）
  - 角色鏈：Context Pack → A1 → A2 → A3 → commit

L：任一條件
  - 超過 5 個檔案
  - 修改 DB Schema（Prisma schema/table 結構）
  - 修改 API 結構（route/endpoint/Webhook）
  - 權限邏輯變更
  - 角色鏈：Context Pack → A1 → A2 → A4（獨立審查）→ A5a（獨立審查）→ A8（獨立審查）→ commit

XL：任一條件
  - 新系統
  - 架構重構
  - 多資料來源整合（monday + LINE + DB 等）
  - 角色鏈：A0 → Context Pack → A1 → A2 → A3（獨立審查）→ A4（獨立審查）→ A5a（獨立審查）→ A5b → A8（獨立審查）→ commit

強制升級規則（以下任一成立即升至 L，即使只有 1 個檔案）：
  - 涉金流/個資 → 升至 L，自動觸發 Gate 4 + A6 資安審查
  - 資安相關 → 升至 L，自動觸發 Gate 4 + A6 資安審查
  - 部署相關 → 升至 L，自動觸發 Gate 2 Source Check（必須查官方文件，禁止憑記憶）
  - 業務計算（金額/時數/損益）→ 升至 L
  - 涉及外部平台 API 變更（LINE/monday/FinMind/Netlify/Cloudflare 等）→ 升至 L，自動觸發 Gate 2 Source Check
  - 同功能第 2 次修改仍失敗 → 觸發 A0
    A0 六選一：繼續 / 先釐清需求 / 先產PRD / 縮小範圍 / 砍掉重做 / 暫停
    A0 需輸出目標快照：本次目標 / 上層目標 / 不處理 / 成功標準 / 偏離警訊

使用者關鍵字對照：中修（中級修改）→ M、大修（高級修改）→ L/XL。關鍵字僅作初始參考，最終等級以條件判定為準。

A2自檢範圍：S級：syntax/type/build/console/基本合理性 | M級以上：上述 + bug/邏輯錯誤/潛在故障風險/明顯違反既有模式。禁止全專案 review。

S/M/L/XL 決定風險等級與角色鏈，Gate 決定必要檢查關卡，TaskCreate 追蹤多步驟工作（不取代 Gate）。

---

## 🤖 A0-A9 角色與決策邊界

同一問題只有一個決策 owner。其他角色只提風險或證據，不得越權。A1 不做產品/架構/安全/部署決策。

決策優先：A0 需求 > A6 資安 > A7 維運 > A8 老闆視角 > A4 架構 > A2 程式正確性 > A3 UI > A5a/A5b 流程文件 > A1 實作
衝突：需求→A0、安全→A6、部署→A7、老闆視角→A8、架構→A4

🎯 每個角色要回答的核心問題
- A2：有沒有寫對？
- A3：好不好用？
- A4：架構對不對？
- A5a：流程順不順？
- A8：老闆 30 秒看懂嗎？

| 角色 | Owner 範圍 | 可做 | 禁止 |
|---|---|---|---|
| A0 需求閘門 | 需求範圍、成功標準、不做事項 | 停止任務、縮小範圍、要求PRD、呼叫/grill-with-docs或/to-prd。任務啟動時生成《目標快照》（本次目標/關聯上層目標/偏離驗證方式），收斂時以此為基準檢查是否偏離 | 寫程式、指定低階技術細節 |
| A1 開發 | 依 owner 結論精準實作 | 修正已確認問題、修根因所需小範圍連帶修改（先說明原因） | 順手重構/加功能/改架構、未經A7改部署、未經Source Check改外部平台 |

🔗 角色鏈強制規則
- M 以上任務：必須執行完整角色鏈，不可跳過
- A1 不得自行宣告其他角色「已 implicitly 完成」
- 標註「獨立審查」的角色：必須以獨立審查區塊輸出，執行方式見「獨立審查執行規範」
- 每個角色必須輸出獨立審查區塊，格式見「審查輸出格式」

| A2 程式審查 | bug、邏輯錯誤、會壞的重複程式 | 指出會造成錯誤的問題 | 審架構(A4)/安全(A6)/UI(A3)、建議重構或抽象化 |
| A3 UI 審查 | 美感、一致性、資訊密度 | A3 審查層級：<br>- 對外系統（官網）：三階段<br>　審查：impeccable critique + audit<br>　修復：frontend-design 或 impeccable craft<br>　打磨：make-interfaces-feel-better（16 條微互動檢查）<br>- 核心後台（監控/LINE 後台/客戶中心）：二階段<br>　審查：impeccable critique<br>　修復：直接修改，不強制使用設計工具<br>- CLI 工具（投資系統）：跳過 A3<br>新專案依系統類型自動套用：對外系統 = 三階段，核心後台 = 二階段，CLI = 跳過。不需每次重新判定。 | 擴大功能、改業務邏輯 |
| A4 架構審查 | data flow、service boundary、DB結構、模組耦合 | 建議架構邊界，無上層衝突時可自行決定。需求不清時呼叫A0 | 建議K8s/微服務/新語言/新DB/訊息佇列 |
| A5a 流程審查 | 流程合理性、使用者體驗 | A5a 流程審查：模擬目標使用者（老闆/同事/客戶）完成主要任務流程，檢查多餘步驟、迷路點、重複輸入、操作中斷點、不合理等待。以獨立審查區塊執行。 | 決定部署修法、改業務決策 |
| A5b 文件記憶 | 文件維護、Context Pack導航 | 任務前載入、任務後更新 | 用claude-mem取代專案文件 |
| A6 資安 | 權限、token、個資、Webhook驗證、資料存取 | 阻擋Critical/High。四級：Critical（權限漏洞/API Key外洩/SQL injection→立即停止）> High（敏感資料未加密/缺輸入驗證）> Medium（已知漏洞/CSP不足）> Low（資訊揭露） | 為方便降低安全 |
| A7 維運部署 | Coolify、DNS、env、build、db部署、Webhook可用性 | Incident Mode、根因診斷。輕量原則：單一VPS、確保備份/硬碟/重啟，不建議K8s/微服務/多雲 | 未診斷就quick fix、未讀COOLIFY.md就改部署 |
| A8 老闆視角審查 | 注意力與決策清晰度層：Cognition（20-30秒理解）、Focus（注意力集中度）、Risk visibility（風險可見性）。「發散」定義：畫面資訊密度過高或結構不清，導致關鍵決策資訊無法在第一時間被識別。輸出：20-30秒測驗/聚焦問題（含風險可見性）/遺漏/可隱藏資訊/建議。純建議，無否決權 | 指出資訊過載或關鍵資訊被埋沒、指出老闆視角看不到價值的新功能 | 評論技術架構/後端設計/實作方式、否決bug修復/資安修復/資料完整性修復、以美觀與否作為判斷標準、不得要求停止開發、不得否決需求、不得否決修復、最終決策權永遠屬於使用者 |
| A9 外部顧問 | 提供外部技術意見與替代方案 | 僅 A0 建議+使用者批准後呼叫。回覆強制標 `🤖 A9（OpenAI）：`。fallback：API不可用→DeepSeek替代，標`⚠️ A9 fallback`。輸出為外部建議，A0 收斂後由使用者最終決策 | 取代A0/A4/A6/A7/A8的owner決策、主動暫停任務 |

### A0 呼叫 A9 流程
1. A0 判斷當前問題超出系統知識邊界 / A2-A7 無法收斂 / 使用者無法決策
2. A0 向使用者提出建議，說明：卡關點、為何需要外部意見
3. 使用者批准後，A0 格式化問題（附：任務目標 / 使用者方向偏好 / 已嘗試方案 / 瓶頸）
4. A9 輸出視為外部建議，A0 收斂後由使用者最終決策
5. 自動觸發條件：同功能修改≥3次未收斂 → A0 判斷是否建議呼叫 A9（非自動呼叫）

---

📋 審查輸出格式（M 以上強制執行）

A3 UI Review（獨立審查）
  - 核心問題：好不好用？
  - 發現問題：
  - 修正建議：
  - 判定：PASS / PASS-WITH-NOTES / FAIL
  （PASS → 可進入下一階段）
  （PASS-WITH-NOTES → 可繼續，但需記錄改善建議）
  （FAIL → A1 修正後重新提交 A3）

A4 Architecture Review（獨立審查）
  - 核心問題：架構對不對？
  - 資料流：
  - 模組邊界：
  - 技術債：
  - 技術風險（效能/安全性/可維護性/耦合度）：
  - 判定：PASS / PASS-WITH-NOTES / FAIL
  （PASS → 可進入下一階段）
  （PASS-WITH-NOTES → 可繼續，但需記錄改善建議）
  （FAIL → A1 修正後重新提交 A4）

A5a 流程審查（獨立審查）
  - 核心問題：流程順不順？
  - 模擬目標使用者：老闆 / 同事 / 客戶
  - 模擬使用者操作步驟：
  - 檢查項目：
    - 多餘步驟
    - 容易迷路
    - 重複輸入
    - 操作中斷點
    - 不合理等待
  - 流程風險：
  - 改善建議：
  - 判定：PASS / FAIL
  （FAIL → A1 修正後重新提交 A5a）

A8 Boss Review（獨立審查）
  - 核心問題：老闆 30 秒看懂嗎？
  - 30秒重點：能否在 30 秒內看懂核心資訊
  - 最大決策風險（誤判方向/錯過重要訊號/看不懂重點）：
  - 最優先處理事項：
  - 判定：PASS / FAIL
  （FAIL → 回到 A1，簡化或重構資訊呈現）

### A1 3-Strike Error Protocol
同一錯誤 → 第1次診斷修復 → 第2次換方法 → 第3次重新思考 → 仍失敗交使用者判斷。若影響正式環境/資料/安全/部署，立即升級 A7 Incident Mode，不等待 3 次。

---

🤖 獨立審查執行規範

需以獨立審查區塊執行的角色：A3 / A4 / A5a / A8

執行方式（依環境選擇）：
- 優先：使用 Task 工具啟動獨立 Agent 執行審查
- 若環境不支援 Task（例如 DeepSeek 外掛模式）：由當前模型以「模擬獨立 Agent」方式執行，必須輸出完整的獨立審查區塊，不得與 A1 開發內容混合
- 若無法確認環境是否支援 Task → 預設使用模擬獨立審查，確保審查不被跳過

執行原則：
- 每個獨立審查只專注自己的領域，不兼做其他角色
- 審查結果必須由主模型彙整後決定下一步（繼續/修正/重構）
- 同一次審查中，A3 和 A4 必須是不同審查實例

不可兼任的組合：
- A1 不得兼任 A3 / A4 / A5a / A8
- 審查區塊不可內嵌於開發輸出中，必須獨立呈現

---

## 🛑 A0 需求閘門（Gate 5）

觸發（任一即停）：新功能/同功能修改≥2次/跨≥5檔案/同時影響前端+API+DB/A2重複警告≥2次/A6/A7/A9提Critical/High/需求或資料源不明。

1. 停止A1開發
2. A0讀Context Pack，只確認業務決策/需求邊界/成功標準/資料來源，不問低階技術
3. 必要時呼叫/grill-with-docs或/to-prd
4. 輸出六選一：繼續/先釐清/先產PRD/縮小範圍/砍掉重做/暫停
5. 只有「繼續」後A1才能開發

### A0 目標快照格式（每次任務啟動必輸出）
- 本次目標：
- 上層目標：
- 不處理：
- 成功標準：
- 偏離警訊：（什麼狀況代表偏離目標）

---

## ✅ Definition of Done

1. 已完成必要Gate（Context Pack/Source Check/Incident Mode/Security Gate/A0）
1a. M 以上：各角色審查區塊必須已輸出，缺少任何指定角色輸出 = 任務未完成，禁止 commit。獨立審查角色必須有明確區塊標記。
2. 已驗證：syntax/type/build | lint | 功能驗證（UI變動附前後對比）| 部署後確認正常回應
3. 已更新文件：CHANGE_LOG_AI.md、CURRENT_TASK.md（標記完成）、必要時 KNOWN_ISSUES.md/DOMAIN_RULES.md/COOLIFY.md
4. 已輸出完成摘要：改什麼/驗證結果/commit:push狀態/殘留風險

有blocking error不得宣稱完成。未驗證部署不得說部署完成。

---

## 部署原則與文件入口

共用平台：阿里雲國際版香港 4C8G / Coolify（dashboard.zhuoye.com.tw:8000）/ Cloudflare DNS。實際部署步驟、網域、環境變數以各專案 `PROJECT_CONTEXT.md` 及 `<專案>/COOLIFY.md` 為準。

1. 涉及部署/Domain/SSL/環境變數/Webhook/Dockerfile/Nixpacks 時，必先讀：工作區 `COOLIFY.md`、該專案 `<專案>/COOLIFY.md`、該專案 `PROJECT_CONTEXT.md`
2. 部署失敗 → Gate 3 觸發 A7 Incident Mode
3. 修復後更新該專案 `COOLIFY.md` 或 `KNOWN_ISSUES.md`
4. 備份/還原：見 `BACKUP.md`、`RESTORE.md`

---

## 共用技術棧偏好

以下為新專案或未明確指定時的預設；既有專案以專案內文件與現有程式碼為準。

- 前端：Next.js 14 + TypeScript + Tailwind CSS
- 資料庫：SQLite（輕量）或 PostgreSQL（多用戶/需權控）
- ORM：Prisma（預設 prisma db push；既有正式資料庫變更需先確認備份與風險）
- UI：shadcn/ui + Recharts
---

## 📌 業務規則存放原則

全域 `~/.claude/CLAUDE.md` → 工作流程與通用規則；`~/Project/CLAUDE.md` → 開發規則與專案入口。詳細業務規則/計算邏輯一律放各專案 `DOMAIN_RULES.md`、`PROJECT_CONTEXT.md`、專案 `CLAUDE.md`。不得因上層文件未記載就假設規則不存在。

---

## 🆕 新專案啟動

多檔案/需設計/涉及 UI → brainstorming → writing-plans → 視覺用 impeccable/frontend-design → executing-plans。單一檔案/純資料處理不在此限。

1. 確認需求 → 在 `~/Project/` 下建立目錄 → 技術棧預設使用共用偏好（有理由才偏離）
2. A5b 建立文件（空模板，後續依規則補）：
   - `PROJECT_CONTEXT.md`（技術棧/部署/DB/外部服務）
   - `DOMAIN_RULES.md`（業務規則，記住或被問≥2次時補）
   - `KNOWN_ISSUES.md`（錯誤特徵/根因/解法）
   - `COOLIFY.md`（僅 Coolify 部署；Cloudflare Pages/Netlify 不適用）
   - `CHANGE_LOG_AI.md`（每次修改後）
3. 寫 `<專案>/CLAUDE.md`（只寫該專案獨有規格）

---

## 安全規則

- API Token/金鑰一律放 `.env`，已在 `.gitignore`；不提交 credentials 到 repo
- Node.js 專案部署前跑 `npm audit`

---

## 專案規格

各專案規格見上方 [各專案入口](#各專案入口) 所列必讀文件。
