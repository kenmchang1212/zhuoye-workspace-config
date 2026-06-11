# Project 工作區共用規則

> 一般任務直接推進；只有高風險情境才啟動安全網。

本檔是 ~/Project/ 下所有開發專案的共用開發規則。各專案獨有規格見該專案子目錄 CLAUDE.md。

## 技術棧偏好

新專案預設，既有專案以現有程式碼為準：
- 前端：Next.js 14 + TypeScript + Tailwind CSS + shadcn/ui
- 後端：Node.js / Next.js API routes
- 資料庫：SQLite（輕量）或 PostgreSQL（多用戶），ORM: Prisma
- 圖表：Recharts
- 部署：Coolify on 阿里雲國際版香港 4C8G，Cloudflare DNS

## 三步執行核心

不是角色、不是流程鐵律、不強制輸出。是思考習慣。
簡單任務直接執行，不必顯性輸出三步。

- **Understand**：理解意圖、判斷風險、選擇技能
- **Design**：最小可行解、不過度設計、不順手改其他東西
- **Execute + Validate**：實作、驗證、回報

## 情境專家（Situational Experts）

> 情境專家不是常規流程的一部分。只有符合觸發條件時才啟動。

- **Security Expert**：涉及 token / 權限 / 金流 / 個資 / 登入時觸發
- **Deployment Expert**：部署失敗時觸發。先讀 COOLIFY.md，找第一個真正錯誤，不猜
- **Boss View Expert**：Dashboard / 管理面板需求時觸發，確保老闆 30 秒能看懂

## 情境 Gate（高風險才觸發）

### Source Check Gate
涉及外部平台設定時觸發。先查官方文件，不憑記憶。

觸發範例：改 Dockerfile、改 env 變數、改 Webhook URL、改 Cloudflare/Coolify/LINE/monday.com 設定

### Context Pack Gate
涉及 DB / API / 業務規則時觸發。先讀專案文件。

觸發範例：改 Prisma schema、新增 API endpoint、改金額計算邏輯

### Incident Mode Gate
部署 / DNS / Webhook 失敗時觸發。停止 quick fix，診斷根因，一次性修復。

觸發範例：Coolify build 失敗、網站 502、Webhook 不回電、DNS 解析失敗

## Skills（工具箱，自由選用）

Skills 是工具，不是流程。看到任務特徵符合就直接用，不需要判斷「是否該用」。
沒有合適 skill 時，直接用工程判斷。

| 技能 | 定位 |
|------|------|
| `code-graph` | 處理「程式怎麼組織」— 結構、依賴、影響範圍 |
| `dashboard-design` | 處理「顯示什麼資訊」— KPI 選擇、數據架構、老闆視角 |
| `workflow-design` | 處理「流程怎麼走」— 步驟順序、狀態轉換、條件分支 |
| `design-interface` | 處理「畫面怎麼呈現」— 排版、顏色、互動、動畫效果 |
| `debug-root-cause` | 處理「為什麼壞了」— 錯誤根因，非 quick fix |

五個技能處理不同層次的問題，非互斥選項。

## 開發慣例

- 程式碼修改前先理解相關檔案結構
- 改完自己檢查：syntax / type / build / 基本邏輯
- 不要順手重構、不要自己新增未要求功能
- 不確定業務規則時停下來問
- 同一問題修兩次仍失敗 → 改用 debug-root-cause，不繼續猜
- 如果影響正式環境 / 資料 / 安全 / 部署，立即升級 Incident Mode

## 部署

共用平台：阿里雲香港 Coolify + Cloudflare DNS。
各專案部署細節見該專案 PROJECT_CONTEXT.md 及 COOLIFY.md。

涉及部署時：
1. 先讀該專案 PROJECT_CONTEXT.md 和 COOLIFY.md
2. 查官方文件（Source Check Gate）
3. 找第一個真正錯誤，不要只看最後一行 log
4. 修復後記錄到 KNOWN_ISSUES.md 或 COOLIFY.md

## 安全底線

- Token / 金鑰一律 .env，已在 .gitignore，不提交
- Node.js 專案部署前跑 npm audit
- 涉及金流 / 個資 / 權限 → Security Expert 介入

## 文件原則

每個專案最多保留：
- CLAUDE.md：專案獨有規格
- PROJECT_CONTEXT.md：技術環境、部署資訊
- DOMAIN_RULES.md：業務規則、計算邏輯（被問 ≥2 次才寫入）
- KNOWN_ISSUES.md：重大錯誤根因與解法
- CHANGE_LOG_AI.md：重要修改紀錄

習慣累積：使用者穩定偏好才寫入，不記 debug log、不記一次性小修。

## 完成回報

改了什麼 / 怎麼驗證 / 殘留風險 / 是否 commit+push
