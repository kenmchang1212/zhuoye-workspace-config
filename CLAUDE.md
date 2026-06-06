# Project 工作區共用規則

## 部署平台

- VPS：阿里雲國際版香港 4C8G
- 部署工具：Coolify（dashboard.zhuoye.com.tw:8000）
- DNS：Cloudflare

部署相關強制規則見全域 Gate 2（Source Check）、Gate 3（Incident Mode）及 A7 Incident Mode。本工作區特有資訊：

1. 涉及部署/Domain/SSL/環境變數/Webhook/Dockerfile/Nixpacks 時，必須先讀：
   - 工作區 `COOLIFY.md`
   - 該專案 `<專案>/COOLIFY.md`
   - 該專案 `PROJECT_CONTEXT.md`
2. 部署失敗 → 全域 Gate 3 觸發 A7 Incident Mode
3. 修復後必須更新該專案 `COOLIFY.md` 或 `KNOWN_ISSUES.md`

---

## 共用技術棧偏好

- 前端：Next.js 14 + TypeScript + Tailwind CSS
- 資料庫：SQLite（輕量）或 PostgreSQL（多用戶/需權控）
- ORM：Prisma（prisma db push，不用 migrate）
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

## 🧭 專案上下文入口（任務前必讀）

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

- 全域 `~/.claude/CLAUDE.md` 只放工作流程與 Gate，不放單一專案業務規則
- `~/Project/CLAUDE.md` 只放專案入口與共用規則，不放詳細計算邏輯
- 詳細業務規則一律放在各專案內：`DOMAIN_RULES.md`、`PROJECT_CONTEXT.md`、專案自己的 `CLAUDE.md`

Claude 不得因為全域或 Project 層文件沒寫到某業務規則，就假設該規則不存在。

---

## 新專案啟動

1. 確認需求（全域 brainstorming → writing-plans）
2. 在 `~/Project/` 下建立目錄
3. 技術棧預設使用上方共用偏好
4. 若需部署 → 建立 `<專案>/COOLIFY.md`（參考 `COOLIFY.md` 模板）
5. 寫 `<專案>/CLAUDE.md`（只寫該專案獨有規格）

---

## 安全規則

- API Token/金鑰一律放 `.env`，已在 `.gitignore`
- 不提交任何 credentials 到 repo
- 部署前跑 `npm audit`

---

## 專案規格

- 卓燁 LINE OA 專案規格：見 `zhuoye-line/CLAUDE.md`
- Monday 儀表板專案規格：見 `monday-dashboard/CLAUDE.md`
