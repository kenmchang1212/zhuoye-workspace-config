> 讀取本檔案後，回覆時必須在開頭標示 【CF】
>
> 修改任何 Cloudflare 設定前，必須先上網搜尋最新官方文件及常見部署失敗問題。不得憑記憶直接修改。

# Cloudflare 設定全記錄

---

## 使用範圍

| 專案 | DNS | Pages |
|------|-----|-------|
| hugo-site（官網） | ✓ | ✓ |
| zhuoye-line | ✓ | — |
| zhuoye-crm | ✓ | — |
| monday-dashboard | ✓ | — |

---

## DNS 管理

所有 zhuoye.com.tw 子網域統一由 Cloudflare DNS 管理。

| Domain | 指向 | 用途 |
|--------|------|------|
| zhuoye.com.tw | Cloudflare Pages | 官網 |
| www.zhuoye.com.tw | zhuoye.com.tw (CNAME) | 官網 redirect |
| line-admin.zhuoye.com.tw | 阿里雲 VPS IP | LINE 後台 |
| api.zhuoye.com.tw | 阿里雲 VPS IP | LINE Webhook |
| crm.zhuoye.com.tw | 阿里雲 VPS IP | 客戶管理中心 |
| dashboard.zhuoye.com.tw | 阿里雲 VPS IP | 工作進度儀表板 |

### DNS 變更注意

- DNS 生效通常 1-5 分鐘（Cloudflare 全球邊緣網路快）
- 修改前確認目標 IP 正確
- 新增子網域需同時在 Coolify 設定對應 Domain

---

## Cloudflare Pages（hugo-site）

### 建置設定

| 設定 | 值 |
|------|-----|
| Framework preset | Hugo |
| 建置指令 | `hugo --gc` |
| 輸出目錄 | `public/` |
| 環境變數 | `HUGO_VERSION`（設定目標 Hugo 版本） |
| 自訂網域 | zhuoye.com.tw, www.zhuoye.com.tw |

### 部署流程

Git push → Cloudflare Pages 自動偵測 → `hugo --gc` 建置 → 部署到 `public/`

### CSP 設定

CSP 透過 `static/_headers` 檔案設定（Cloudflare Pages 原生支援 `_headers`）。修改 CSP 後需確認不影響：GA4、Leaflet 地圖、Google Fonts、Decap CMS。

---

## 踩坑紀錄

（尚無記錄，如有請補充）

---

## 常見問題速查

| 問題 | 可能原因 | 檢查方向 |
|------|---------|---------|
| 部署後網站打不開 | DNS 記錄未指向 Pages | Cloudflare DNS 設定 |
| 自訂網域失效 | Pages 自訂網域設定遺失 | Dashboard → Pages → 專案 → Custom domains |
| SSL 憑證錯誤 | SSL/TLS 模式不正確 | 設為 Full (strict) |
| CSP 報錯 | 新增資源未放行 | 檢查 `_headers` 檔案 |
