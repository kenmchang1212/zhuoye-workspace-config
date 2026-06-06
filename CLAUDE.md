# Project 工作區共用規則

## 部署平台
- VPS：阿里雲國際版香港 4C8G
- 部署工具：Coolify（dashboard.zhuoye.com.tw:8000）
- DNS：Cloudflare
- **部署前必須讀取該專案的 COOLIFY.md**（完整步驟、環境變數、Domain、踩坑記錄）

## 共用技術棧偏好
- 前端：Next.js 14 + TypeScript + Tailwind CSS
- 資料庫：SQLite（輕量）或 PostgreSQL（多用戶/需權控）
- ORM：Prisma（prisma db push，不用 migrate）
- UI：shadcn/ui + Recharts
- 新專案預設從此技術棧開始，有理由才偏離

## 所有專案

| 專案 | 目錄 | Domain | 部署 | 說明 |
|---|---|---|---|---|
| Monday 儀表板 | `monday-dashboard/` | dashboard.zhuoye.com.tw | Coolify | 工作活動/案件/客戶貢獻三主體 |
| 卓燁 LINE OA | `zhuoye-line/` | line-admin.zhuoye.com.tw, api.zhuoye.com.tw | Coolify | LINE Bot 後台「小卓」 |
| 卓燁官網 | `ZHUOYE/hugo-site/` | zhuoye.com.tw | Netlify | Hugo 靜態官網 |

## 部署相關文件
- 通用 Coolify 知識：`COOLIFY.md`（Dockerfile 模板、Nixpacks vs Dockerfile、踩坑全記錄）
- 各專案 Coolify 步驟：`<專案>/COOLIFY.md`
- 備份/還原：`BACKUP.md`、`RESTORE.md`

## 新專案啟動
1. 確認需求（全域 brainstorming → writing-plans）
2. 在 `~/Project/` 下建立目錄
3. 技術棧預設使用上方共用偏好
4. 若需部署 → 建立 `<專案>/COOLIFY.md`（參考 `COOLIFY.md` 模板）
5. 寫 `<專案>/CLAUDE.md`（只寫該專案獨有規格）

## 安全規則
- API Token/金鑰一律放 `.env`，已在 `.gitignore`
- 不提交任何 credentials 到 repo
- 部署前跑 `npm audit`

## 專案規格
- 卓燁 LINE OA 專案規格：見 `zhuoye-line/CLAUDE.md`
- Monday 儀表板專案規格：見 `monday-dashboard/CLAUDE.md`
