# Project 工作區共用規則

> 一般任務直接推進；只有高風險情境才啟動安全網。

本檔是 ~/Project/ 下所有開發專案的共用規則，各專案獨有規格見該專案子目錄 CLAUDE.md。
（思考習慣、進專案讀文件、Memory 分流見全域 CLAUDE.md，此處不重複。）

## 技術棧偏好

新專案預設，既有專案以現有程式碼為準：
- 前端：Next.js 14 + TypeScript + Tailwind CSS + shadcn/ui
- 後端：Node.js / Next.js API routes
- 資料庫：SQLite（輕量）或 PostgreSQL（多用戶），ORM: Prisma
- 圖表：Recharts
- 部署：Coolify on 阿里雲國際版香港 4C8G，Cloudflare DNS

## 情境專家（高風險才觸發，做錯會出事）

你負責守的領域，碰到就從這角度把關：
- **資安專家**：涉及 token / 權限 / 個資 / 金流 / Webhook 驗證 / 客戶資料時觸發，Critical 未解不上線
- **部署專家**：改部署設定 / DNS / Coolify 時觸發，先讀 COOLIFY.md／PROJECT_CONTEXT.md（失敗診斷見故障關卡）

## 情境關卡（高風險才觸發，動手前必過的檢查）

- **查證關卡**：改外部平台設定（Dockerfile / Webhook URL / Cloudflare·Coolify·LINE·monday）或連線/金鑰類 env（DATABASE_URL / API key / token / secret）→ 先查官方文件，不憑記憶
- **讀文件關卡**：涉及 DB / API / 業務規則 / 金額（改 Prisma schema / 新 API / 改金額計算）→ 先讀專案文件（清單與按需讀見全域「進專案＋專案文件」）
- **故障關卡**：部署 / DNS / Webhook 失敗（build 失敗、502、Webhook 不回、DNS 解析失敗）→ 停 quick fix，找第一個真正錯誤、不猜，一次修到根因
- **資料關卡**：刪除或批次修改正式資料（批次刪客戶 / 批次改記帳費 / 無 WHERE 的 update）→ 先確認範圍＋備份，禁止無條件批次操作

## 品質把關（不是風險，是「做出來要好懂」）

- **老闆視角**：Dashboard / 管理面板需求時觸發，確保老闆 30 秒看懂

## Skills（核心名稱對應）

工具定位見全域 CLAUDE.md。未安裝同名 skill 時用對應替代，無替代則用工程判斷、不卡任務。

| 核心名稱 | 實際對應 | 定位 |
|---------|---------|------|
| `code-graph` | MCP codegraph | 程式怎麼組織—結構、依賴、影響範圍 |
| `dashboard-design` | `ui-designer` + 工程判斷 | 顯示什麼資訊—KPI、數據架構、老闆視角 |
| `workflow-design` | `prompt-optimizer` | 流程怎麼走—步驟、狀態、分支 |
| `design-interface` | `impeccable` + `frontend-design` | 畫面怎麼呈現—排版、顏色、互動、動畫 |
| `debug-root-cause` | `systematic-debugging` | 為什麼壞了—根因，非 quick fix |

## 開發慣例

- 改程式前先理解相關檔案結構；改完自查 syntax / type / build / 邏輯
- 不順手重構、不自己加未要求的功能
- 不確定業務規則時停下來問
- 同一問題修兩次仍失敗 → debug-root-cause，不繼續猜
- 影響正式環境 / 資料 / 安全 / 部署 → 立即升級故障關卡

## 部署

共用平台：阿里雲香港 Coolify + Cloudflare DNS；各專案細節見該專案 PROJECT_CONTEXT.md 及 COOLIFY.md。
Git：直接 commit main → push → Coolify Redeploy（監聽 main），不開 feature branch；僅大型重構/高風險才另開。
流程：①先讀該專案 PROJECT_CONTEXT.md＋COOLIFY.md ②查官方文件 ③改完記錄到 KNOWN_ISSUES.md 或 COOLIFY.md。

## 安全底線

- Token / 金鑰一律 .env、已在 .gitignore，不提交
- Node.js 專案部署前跑 npm audit
- 涉及金流 / 個資 / 權限 → 資安專家介入

## 文件原則

五種專案文件（CLAUDE / PROJECT_CONTEXT / DOMAIN_RULES / KNOWN_ISSUES / CHANGE_LOG_AI）用途與讀寫時機見全域「進專案＋專案文件」。
習慣累積：使用者穩定偏好才寫入，不記 debug log、不記一次性小修。

## 完成回報

格式：改了什麼 / 怎麼驗證 / 殘留風險 / 是否已 commit / 是否已 push
