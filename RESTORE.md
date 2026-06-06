# 卓燁 LINE OA 後台 — 還原手冊

發生 VPS 故障或需要搬遷時，依照以下步驟在新 VPS 重建服務。

> 部署詳細說明與踩坑記錄請見 [COOLIFY.md](../COOLIFY.md)

## 前提

- 新 VPS 已安裝 Ubuntu 24.04
- 已設定 SSH key 登入
- 防火牆開放 80/443/SSH
- 網域 DNS 由 Cloudflare 管理

## 1. 安裝 Coolify

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

安裝完成後依指示開啟 Coolify Web UI，設定管理者帳號。

## 2. 連接 GitHub 並部署專案

1. Coolify → Sources → 連接 GitHub（用 GitHub App，不要用 PAT token）
2. Coolify → Projects → New Project → "zhuoye"
3. New Resource → Application → 選 `kenmchang1212/zhuoye-line`，branch `main`
4. **Build pack: Dockerfile**（不要選 Nixpacks）
5. Port: 3000

## 3. 設定環境變數

在 Coolify Application → Environment Variables → Production 中設定：

| Key | Value | Buildtime | Runtime |
|---|---|---|---|
| `DATABASE_URL` | 從 PostgreSQL 資源取得連線字串 | ✓ | ✓ |
| `LINE_CHANNEL_ACCESS_TOKEN` | 從 LINE Developer Console 取得 | ✓ | ✓ |
| `LINE_CHANNEL_SECRET` | 從 LINE Developer Console 取得 | ✓ | ✓ |
| `NODE_ENV` | `production` | ✗ | ✓ |

> **重要**：DATABASE_URL 必須 Buildtime + Runtime 都勾，Prisma generate 在建置階段需要連資料庫。

## 4. 設定 PostgreSQL 資料庫

若 PostgreSQL 尚未安裝，在 Coolify 中建立：

1. New Resource → Database → PostgreSQL
2. 資料庫名稱：`zhuoye_line`
3. 記下連線資訊，更新 `DATABASE_URL`

## 5. 還原資料庫

```bash
# 將備份檔上傳到 VPS 後
psql -U <user> -d zhuoye_line -f /path/to/backup.sql
```

或從物件儲存下載：

```bash
# Cloudflare R2
aws s3 cp s3://zhuoye-backups/backup-YYYY-MM-DD.sql . --endpoint-url https://<account>.r2.cloudflarestorage.com
psql -U <user> -d zhuoye_line -f backup-YYYY-MM-DD.sql
```

## 6. 執行 Prisma 遷移與種子資料

在 Coolify Application → Terminal：

```bash
npx prisma db push
npx tsx prisma/seed.ts
```

## 7. 設定 DNS

在 Cloudflare DNS 新增記錄：

| 類型 | 名稱 | 內容 |
|------|------|------|
| A | line-admin.zhuoye.com.tw | <VPS IP> |
| A | api.zhuoye.com.tw | <VPS IP> |

## 8. 設定 Coolify 網域與 SSL

1. Application → General → Domains 輸入框
2. **多個 domain 用逗號分隔**（不要換行，不要點 Generate Domain 按鈕）：
   ```
   https://line-admin.zhuoye.com.tw,https://api.zhuoye.com.tw
   ```
3. 點右上角 Save
4. Coolify 自動申請 Let's Encrypt SSL 憑證

## 9. 測試

### Webhook 測試

```bash
curl -s -w "\n%{http_code}" -X POST https://api.zhuoye.com.tw/line/webhook \
  -H "Content-Type: application/json" \
  -d '{"events":[]}'
# 預期：401 + {"error":"Missing signature"}
# 注意：401 是正常回應！表示 Webhook 正常運行，只是缺少 LINE 簽名
```

### 後台頁面測試

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://line-admin.zhuoye.com.tw/login
curl -s -o /dev/null -w "%{http_code}\n" https://line-admin.zhuoye.com.tw/
curl -s -o /dev/null -w "%{http_code}\n" https://line-admin.zhuoye.com.tw/settings
curl -s -o /dev/null -w "%{http_code}\n" https://line-admin.zhuoye.com.tw/users
curl -s -o /dev/null -w "%{http_code}\n" https://line-admin.zhuoye.com.tw/audit-log
# 預期：全部 200
```

### 登入測試

1. 開啟 https://line-admin.zhuoye.com.tw/login
2. 使用預設管理者帳號：`kenmchang1212@gmail.com` / `admin123`
3. 登入後建議立即修改密碼（設定 → 變更密碼）

## 10. 設定 LINE Webhook URL

1. 登入 LINE Developer Console
2. 選擇卓燁的 Provider → Channel
3. Messaging API → Webhook URL 設為 `https://api.zhuoye.com.tw/line/webhook`
4. 點擊 "Verify" 確認連線成功

## 11. 排程自動備份

在 VPS 上設定 cron job：

```bash
crontab -e
```

新增：

```
0 3 * * * /path/to/backup.sh
```

備份腳本位於專案 `scripts/backup.sh`，需先設定物件儲存連線資訊。
