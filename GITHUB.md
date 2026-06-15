> 讀取本檔案後，回覆時必須在開頭標示 【GH】
>
> 修改任何 GitHub 設定前，必須先上網搜尋最新官方文件及常見問題。不得憑記憶直接修改。

# GitHub 設定全記錄

---

## 使用範圍

| 功能 | 使用專案 | 說明 |
|------|---------|------|
| Git push 觸發部署 | 全部 | Cloudflare Pages / Coolify 自動部署 |
| Decap CMS OAuth | hugo-site | CMS 後台登入驗證 |
| GitHub App (Coolify) | zhuoye-line, zhuoye-crm, monday-dashboard | Coolify 存取 private repo |
| Deploy Key | 視需要 | Private repo 免 GitHub App 的替代方案 |

---

## Coolify GitHub App 設定

### 注意事項（來自 Coolify 踩坑）

1. **Repo 一次設對，絕不事後改 Git URL** — 錯了就刪除應用重建（Coolify bug #8917）
2. **Private repo 需 GitHub App 已授權** — 確認 App 有 Repository access 權限
3. **同 commit 不重建** — 點 Force Deploy (without cache)

### 授權確認

Settings → GitHub Apps → 你的 App → Repository access → 確認目標 repo 在清單中

---

## Decap CMS OAuth（hugo-site）

### 用途

內容編輯者透過 GitHub OAuth 登入 Decap CMS 後台（`/admin/`）

### 關鍵設定

| 設定 | 說明 |
|------|------|
| OAuth App | GitHub Settings → Developer settings → OAuth Apps |
| Authorization callback URL | `https://zhuoye.com.tw/admin/` 或 Cloudflare Pages Functions 端點 |
| Client ID / Secret | 存在環境變數，不提交 repo |

---

## 踩坑紀錄

（尚無記錄，如有請補充）

---

## 常見問題速查

| 問題 | 可能原因 | 檢查方向 |
|------|---------|---------|
| Coolify 拉不到 private repo | GitHub App 未授權或權限不足 | GitHub App → Repository access |
| Decap CMS 無法登入 | OAuth App 設定錯誤或 callback URL 不符 | GitHub OAuth App 設定 |
| Git push 沒觸發部署 | Webhook 未設定或 repo 權限變更 | 平台部署設定頁面 |
