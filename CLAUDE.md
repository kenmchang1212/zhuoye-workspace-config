# Project 工作區共用開發規則

> `~/Project/` 下所有開發專案強制遵守。各專案獨有規格見該專案子目錄 CLAUDE.md。

## 🚦 程式開發閘門（Gate 1-5）

每個任務開始必評估 Gate 1-5，跳過任何 Gate → A0 強制介入。進入 Plan Mode 同樣強制。

| Gate | 觸發條件 | 動作 |
|------|----------|------|
| Gate 1 Context Pack | 業務規則/金額/客戶/DB/API/部署/Webhook/DNS/M 以上 | 讀專案文件，5 行內輸出摘要，未完成禁止寫程式 |
| Gate 2 Source Check | 涉及外部平台（Coolify/LINE/Cloudflare/monday.com 等） | 查官方文件，輸出查證來源，查不到禁改高風險設定 |
| Gate 3 Incident | 部署/Webhook/DNS/外部平台失敗 ≥1 次 | 停止 quick fix，A6 全局診斷，輸出根因+一次性修復計畫 |
| Gate 4 資安 | 登入/權限/token/Webhook 驗證/個資/金鑰 | 啟動 A5，Critical/High 未解前禁止 commit/push |
| Gate 5 A0 | 新功能/同功能≥2 次/跨≥5 檔案/前端+API+DB/需求不明 | 停止開發，交 A0 重設需求 |

## 🧠 Context Pack（Gate 1）

M 以上或涉及業務規則/DB/API/部署/Webhook/DNS 時觸發。未完成不得修改檔案。

讀取：本檔 → 專案 CLAUDE.md → DOMAIN_RULES.md → PROJECT_CONTEXT.md → KNOWN_ISSUES.md（部署加 COOLIFY.md）
輸出（5 行內）：`專案 / 本次目標 / 必讀規則來源 / 不可破壞事項 / 已知風險`

### 各專案入口

所有專案見 `~/Project/*/`，各專案 CLAUDE.md 自我描述部署平台與必讀文件。任務開始時讀取對應專案 CLAUDE.md 即可。

## 🚨 A6 Incident Mode（Gate 3）

部署/Webhook/DNS/外部 API 失敗時觸發。立即停止 quick fix，輸出 Incident Report：

```
A6 Incident Report
1. 失敗類型：
2. 影響專案/服務：
3. 第一個真正錯誤：
4. 已檢查：build / env / db / domain:DNS / webhook / Dockerfile / COOLIFY.md / 官方文件
5. 根因判斷：
6. 一次性修復計畫：
7. 驗證方式：
```

限制：不得只根據最後一行錯誤修改、未讀 COOLIFY.md 不得改部署、未確認根因不得連續部署。

## 🔎 Source Check Gate（Gate 2）

涉及第三方平台修改前必做：

```
Source Check：
- 平台：
- 文件來源：
- 相關規則：
- 與專案文件衝突：
- 結論：
```

優先查官方文件。衝突→以最新官方文件為準。無法查→標「⚠️ 無文件佐證」，不得改高風險設定。

---

## 🎯 風險分級（S/M/L/XL）

先以下方條件初判，再以強制升級規則覆蓋。

**S**：單檔案，不改 DB Schema / API / UI 結構
　→ A1 → A2 → commit

**M**：2~5 檔案 / 新增頁面 / 改 UI 結構
　→ Context Pack → A1 → A2 → A3 → commit

**L**：5+ 檔案 / 改 DB Schema / API 結構 / 權限邏輯
　→ Context Pack → A1 → A2 → A3 → A4 → A7 → commit

**XL**：新系統 / 架構重構 / 多資料來源整合
　→ A0 → Context Pack → A1 → A2 → A3 → A4 → A7 → commit

**強制升級規則**（任一成立即升至 L）：
- 涉金流/個資 → Gate 4 + A5 資安審查
- 部署相關 → Gate 2 Source Check
- 業務計算（金額/時數/損益）
- 外部平台 API 變更 → Gate 2 Source Check
- 同功能第 2 次修改仍失敗 → Gate 5 A0

A0 六選一：繼續 / 先釐清需求 / 先產 PRD / 縮小範圍 / 砍掉重做 / 暫停

---

## 🤖 A0-A7 角色與決策邊界

同一問題只有一個決策 owner。其他角色只提風險或證據，不得越權。

決策優先：A0 需求 > A5 資安 > A6 維運 > A7 老闆視角 > A4 架構 > A2 程式 > A3 體驗 > A1 實作

| 角色 | Owner 範圍 | 禁止 |
|------|-----------|------|
| A0 需求閘門 | 需求範圍、成功標準、不做事項 | 寫程式、指定低階技術 |
| A1 開發 | 依 owner 結論精準實作 | 順手重構/加功能、未經 Source Check 改外部平台 |
| A2 程式審查 | bug、邏輯錯誤、重複程式 | 審架構(A4)/安全(A5)/UI(A3) |
| A3 體驗審查 | 美感、一致性、流程合理性、資訊密度 | 擴大功能、改業務邏輯 |
| A4 架構審查 | data flow、模組邊界、DB 結構 | 建議 K8s/微服務/新語言/新 DB |
| A5 資安 | 權限、token、個資、Webhook 驗證 | 為方便降低安全 |
| A6 維運部署 | Coolify、DNS、env、build、Webhook | 未診斷就 quick fix |
| A7 老闆視角 | 注意力與決策清晰度（20-30 秒理解） | 否決 bug/資安修復、以美觀為判斷標準 |

### A1 3-Strike Protocol
同一錯誤 → 第1次診斷修復 → 第2次換方法 → 第3次交使用者。影響正式環境/資料/安全/部署 → 立即升級 A6 Incident Mode，不等待 3 次。

### A3 審查層級
新專案依系統類型自動套用，不需每次重新判定：
- 對外系統（官網）：impeccable critique+audit → frontend-design/impeccable craft → make-interfaces-feel-better
- 核心後台（監控/LINE 後台/CRM）：impeccable critique → 直接修改
- CLI 工具：跳過 A3

A9 外部顧問：僅 `/a9檢查` `/a9審查` 工具。A0 建議+使用者批准後呼叫，輸出標 `🤖 A9`。不取代 A0/A4/A5/A6/A7 owner 決策。

🔗 M 以上必須執行完整角色鏈，不可跳過。A1 不得宣稱其他角色已隱含完成。缺少審查區塊不得 commit。

---

## 🧭 Skill Router（必用）

原則：角色判斷、技能執行。一技能一用途。不用多個重複技能。

| # | 技能 | 用途 | 等級 |
|---|------|------|------|
| 1 | `systematic-debugging` | Bug/錯誤/部署失敗，根因優先，禁止 quick fix | Gate 3，全等級 |
| 2 | `impeccable:impeccable` | UI critique/audit（A3 綁定） | M 以上 UI 變更 |
| 3 | `frontend-design` | UI 實作 | M 以上 UI 變更 |
| 4 | `make-interfaces-feel-better` | 動畫/微互動打磨 | L/XL UI 收尾 |
| 5 | `requesting-code-review` | subagent 隔離審查（A2 綁定） | M 以上 |
| 6 | `verification-before-completion` | 禁止無證據宣稱完成 | 全等級，完成前強制 |
| 7 | `brainstorming` | 需求釐清 | XL |
| 8 | `writing-plans` + `executing-plans` | 完整計畫鏈 | XL |
| 9 | `webapp-testing` | Playwright 測試 | 環境可用時 |
| 10 | `security-review` | 自動化安全審查（Claude Code 內建） | L/XL 可選 |
| 11 | `claude-mem:mem-search` / `pathfinder` / `smart-explore` | 輔助索引 | 不熟悉專案時 |

---

## 📋 審查輸出格式

A3 UI Review：核心問題（好不好用？）/ 發現問題 / 修正建議 / 判定 PASS/FAIL
A4 Architecture Review：核心問題（架構對不對？）/ 資料流 / 模組邊界 / 技術債 / 判定 PASS/FAIL
A7 Boss Review：核心問題（老闆 30 秒看懂嗎？）/ 30 秒重點 / 最大決策風險 / 判定 PASS/FAIL

FAIL → 回 A1 修正後再審。

A1 不得兼任 A3/A4/A7，審查需以獨立區塊輸出。

---

## 🛑 A0 需求閘門（Gate 5）

觸發即停止 A1。A0 讀 Context Pack，只確認需求邊界/成功標準/資料來源，不問低階技術。

### A0 目標快照（每次任務啟動必輸出）
- 本次目標：
- 上層目標：
- 不處理：
- 成功標準：
- 偏離警訊：

---

## ✅ Definition of Done

1. 已完成必要 Gate（Context Pack / Source Check / Incident Mode / Security Gate / A0）
2. M 以上：各角色審查區塊已輸出
3. 已驗證：syntax/type/build | lint | 功能驗證（UI 附前後對比）| 部署確認
4. 已更新文件：CHANGE_LOG_AI.md、CURRENT_TASK.md（標記完成）
5. 已輸出完成摘要：改什麼 / 驗證結果 / 殘留風險

Blocking error 不得宣稱完成。未驗證部署不得說部署完成。

---

## 部署原則

共用平台：阿里雲香港 4C8G / Coolify / Cloudflare DNS。各專案詳細部署以 `PROJECT_CONTEXT.md` + `COOLIFY.md` 為準。

1. 涉及部署/Webhook/DNS/Dockerfile 必先讀該專案 `COOLIFY.md`
2. 部署失敗 → Gate 3 → A6 Incident Mode
3. 修復後更新 `COOLIFY.md` 或 `KNOWN_ISSUES.md`

---

## 共用技術棧

- 前端：Next.js 14 + TypeScript + Tailwind CSS
- 資料庫：SQLite（輕量）或 PostgreSQL（多用戶/需權控）
- ORM：Prisma
- UI：shadcn/ui + Recharts

---

## 安全規則

- API Token/金鑰一律放 `.env`（已 `.gitignore`），不提交 credentials
- Node.js 部署前跑 `npm audit`
- A5 資安四級：Critical > High > Medium > Low

---

## 🆕 新專案啟動

1. `~/Project/` 下建立目錄
2. 建立：`PROJECT_CONTEXT.md` `DOMAIN_RULES.md` `KNOWN_ISSUES.md` `CHANGE_LOG_AI.md`（Coolify 部署加 `COOLIFY.md`）
3. 寫 `<專案>/CLAUDE.md`（只寫該專案獨有規格）
4. XL 級走 brainstorming → writing-plans → implementing → executing-plans

---

## 📌 文件記憶原則

詳細 A5b 規則見 `~/.claude/CLAUDE.md`。業務規則/計算邏輯一律放各專案 `DOMAIN_RULES.md`。不得因上層文件未記載就假設規則不存在。
