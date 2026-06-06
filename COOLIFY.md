# Coolify 部署全記錄

> 最後更新：2026-06-03，基於 zhuoye-line 實際部署經驗

---

## Nixpacks vs Dockerfile：為什麼改用 Dockerfile

| | Nixpacks | Dockerfile |
|---|---|---|
| 運作方式 | 自動偵測框架、自動建置 | 手寫建置步驟 |
| 優點 | 零設定，推上去就能跑 | 完全控制，行為可預測 |
| 缺點 | 黑箱，出錯難排查；無法控制 Node/Prisma 版本 | 需手寫，需了解 Docker |
| zhuoye-line 舊部署 | 用 Nixpacks，正常運作 | — |
| zhuoye-line 新部署 | — | 改用 Dockerfile，可控性優先 |

**結論**：Next.js + Prisma 這種有原生相依套件的組合，用 Dockerfile 才能精確控制 Prisma engine 的路徑和 Alpine 相容性。Nixpacks 能跑是運氣好，出問題無從查起。

---

## 部署前必檢

1. **Proxy 必須 Running** — Servers → localhost → Proxy 綠燈
2. **Domains 加 `https://` 前綴** — 不加會導致 Traefik 生成 `Host(空)` → 全路由 404
3. **GitHub App 權限確認** — 確認 App 已授權目標 repo（Settings → GitHub Apps → 你的 App → Repository access）
4. **Repo 公開或已授權** — Private repo 需透過 GitHub App 或 Deploy Key，不能直接拉

---

## Dockerfile 模板（Next.js 14 + Prisma 6）

```dockerfile
FROM node:20-alpine
WORKDIR /app

# 系統相依：python3/make/g++ 給 bcryptjs 等 native module，tzdata 給時區
RUN apk add --no-cache python3 make g++ tzdata
ENV TZ=Asia/Taipei

# 第一階段：只複製 package.json，讓 npm ci 可被 Docker cache
COPY package.json package-lock.json* ./
RUN npm ci

# 第二階段：複製 Prisma schema，generate client
COPY prisma/schema.prisma ./prisma/
COPY prisma.config.ts ./
RUN npx prisma generate

# 第三階段：複製全部原始碼，執行 Next.js build
COPY . .
RUN npm run build

# 關鍵：手動複製 Prisma query engine 到 Next.js 執行期路徑
# Prisma 6.x + Next.js 14 需要 engine 在 .next/server/chunks/
RUN cp src/generated/prisma/libquery_engine-linux-musl-openssl-3.0.x.so.node .next/server/chunks/

ENV NODE_ENV=production
EXPOSE 3000

# 啟動：先 db push（確保 schema 同步），再 seed（初始資料），再 next start
CMD ["sh", "-c", "npx prisma db push --skip-generate && npx tsx prisma/seed.ts && exec npx next start -H 0.0.0.0 -p 3000"]
```

### Dockerfile 分層說明

| 層 | 目的 | 為什麼這樣排 |
|---|---|---|
| apk add | 安裝系統套件 | 給 bcryptjs 編譯 native addon、設定台北時區 |
| COPY package.json + npm ci | 安裝 npm 套件 | package.json 不常變，Docker cache 可跳過重裝 |
| COPY prisma + prisma generate | 產生 Prisma Client | schema 異動才重建這層 |
| COPY . + npm run build | Next.js 建置 | 原始碼異動才重建 |
| cp engine | 搬移 Prisma query engine | 建置後 .next 目錄已存在，複製 engine 進去 |
| CMD | 容器啟動 | db push 確保 schema → seed 初始資料 → next start 啟動 |

---

## 環境變數完整清單

### zhuoye-line 案例

| Key | Value | Buildtime | Runtime | 說明 |
|---|---|---|---|---|
| `DATABASE_URL` | `postgresql://...` | ✓ | ✓ | Prisma 建置時需連線（generate），執行期也需連線 |
| `LINE_CHANNEL_ACCESS_TOKEN` | 從 LINE Console 取得 | ✓ | ✓ | LINE Bot 發送訊息用 |
| `LINE_CHANNEL_SECRET` | 從 LINE Console 取得 | ✓ | ✓ | LINE Webhook 簽名驗證用 |
| `NODE_ENV` | `production` | ✗ | ✓ | Next.js 最佳化模式，建置時不需要 |

### 注意事項

- **DATABASE_URL 必須 Buildtime + Runtime 都勾**：Prisma generate 在建置階段需要連資料庫讀 schema 資訊，執行期當然也需要
- **LINE 金鑰建議 Buildtime + Runtime 都勾**：Next.js build 可能預渲染部分頁面時用到
- **NODE_ENV 只勾 Runtime**：建置階段 Next.js 自行處理，不需要手動設
- **刪除應用前務必先備份所有環境變數**：Coolify 沒有匯出功能，只能一個一個點眼睛複製
- **環境變數值在 Coolify log 中會被遮蔽**（顯示為 REDACTED），不用擔心外洩

---

## Domain 設定

### 多個 Domain 用逗號分隔

Coolify Domains 輸入框**不接受換行**，多個 domain 用逗號分隔：

```
https://line-admin.zhuoye.com.tw,https://api.zhuoye.com.tw
```

### Generate Domain 按鈕的陷阱

**`Generate Domain` 按鈕會產生隨機網域（如 `xxx.47.239.84.14.sslip.io`），不是「新增一個自訂網域」的意思。**

- 要加自訂網域 → 直接在輸入框內打完整 URL，逗號分隔
- Generate Domain 只用於快速測試，正式環境不要用

### Direction 設定

保持 `Allow www & non-www.` 即可。

---

## 踩坑全記錄

### 坑 1：更換 Git Repository URL 導致 Coolify 內部狀態損毀（Critical）

**時間**：2026-06-03，耗時約 2 小時，嘗試 ≥ 5 次均失敗

**現象**：
- 將舊應用（指向 `zhuoye-claude-config` repo）的 Git URL 改成 `zhuoye-line` 後
- 每次部署都回 `GitHub API call failed: Error: Not Found`
- 即使 repo 設為 public、加上 GitHub App 授權、Force Deploy 都無效

**根因**：
- Coolify v4.1.1 內部 bug（GitHub Issues #8917, #5641）
- 更換 Git URL 時，內部 project ID 和 repo 的 mapping 不同步
- `Not Found` 並非真的 404，而是 Coolify 內部 auth/project ID 對不上

**正確做法**：
- **建立 Coolify 應用時就設對 repo，之後絕不更動 Git URL**
- 如果 repo 錯了 → **直接刪除整個應用重建**，不要嘗試改 URL
- 刪除前先備份環境變數

### 坑 2：同一個 Git commit 不會觸發重建（Medium）

**現象**：push 修正後點 Deploy，Log 顯示 `No build configuration changed & image found...Build step skipped`

**原因**：Coolify 以 commit SHA 判斷是否需要重建，同一個 commit 的 image 已存在就跳過

**正確做法**：點 Deploy 旁的下拉選單 → **Force Deploy (without cache)**

### 坑 3：Prisma query engine 在建置階段找不到（High）

**現象**：
```
PrismaClientInitializationError: Prisma Client could not locate the Query Engine
for runtime "linux-musl-openssl-3.0.x".
```

**原因**：
- Next.js build 時會預渲染（prerender）API 路由
- 預渲染時執行到 `prisma.user.findUnique()` 等呼叫
- Prisma engine 必須在 `.next/server/chunks/` 才能被找到
- Dockerfile 中 `cp engine` 在建置之後才執行，建置階段沒有 engine

**解法**：
1. 在 Dockerfile 的 `npm run build` 之前先 `mkdir -p .next/server/chunks && cp engine .next/server/chunks/`
2. 或是：所有用到 Prisma 的 API 路由都加上 `export const dynamic = "force-dynamic"` 跳過預渲染

**目前採用解法 2**，因為解法 1 的 `.next` 目錄在 build 過程中可能被清空。

### 坑 4：Docker 不支援 BuildKit（Low，僅警告）

**現象**：`Docker 29.1.3 on deployment server (localhost) does not support BuildKit`

**影響**：無，建置仍正常完成。只是 build output progress 顯示受限。

### 坑 5：LINE_CHANNEL 金鑰的 Docker secrets 警告（Low，僅警告）

**現象**：
```
SecretsUsedInArgOrEnv: Do not use ARG or ENV instructions for sensitive data
(ARG "LINE_CHANNEL_ACCESS_TOKEN")
```

**原因**：Coolify 將環境變數透過 Docker build args 傳入，觸發 Docker 的安全警告

**影響**：無，這是 Docker 的 generic warning。Coolify 的環境變數管理是安全的。

### 坑 6：DATABASE_URL 為無效值，不是 PostgreSQL 連線格式（Critical）

**時間**：2026-06-03，耗時約 30 分鐘

**現象**：
```
PrismaClientInitializationError:
error: Error validating datasource `db`: the URL must start with the protocol `postgresql://` or `postgres://`.
```

**原因**：
- DATABASE_URL 的值為 `QdA1GN6BOHzn...` 之類的亂碼，不是 `postgresql://` 開頭的連線字串
- 從 Preview Deployments 複製時可能複製到錯誤的值，或被 Coolify 的加密/遮蔽功能干擾

**正確做法**：
- 從 PostgreSQL 資源頁面取得 Connection String（不是從 Preview Deployments）
- 確認值以 `postgresql://` 開頭
- 不要倚賴眼睛圖示顯示的值（有時顯示不完全）

### 坑 7：Dockerfile CMD 缺少 prisma db push + seed，資料庫為空（Critical）

**時間**：2026-06-03，耗時約 20 分鐘

**現象**：
```
PrismaClientKnownRequestError (code: P2021):
The table `public.User` does not exist in the current database.
```

**原因**：
- Dockerfile CMD 只包含 `next start`，沒有 `prisma db push` 和 `seed`
- 容器啟動後 Next.js 直接開始服務，但資料庫沒有任何資料表
- Nixpacks 舊應用之所以正常，是因為 Nixpacks 可能在建置階段自動執行了 migration

**正確做法**：
- Dockerfile CMD 必須包含三個步驟：`prisma db push --skip-generate && npx tsx prisma/seed.ts && npx next start`
- 使用 `exec` 啟動 next start 確保 signal 正確傳遞
- seed.ts 全部使用 `upsert` / `findFirst + create` 模式，重複執行安全

---

## 建置失敗模式速查

| 錯誤關鍵字 | 原因 | 解法 |
|---|---|---|
| `PrismaClientInitializationError` + `linux-musl-openssl` | Prisma engine 沒在 `.next/server/chunks/` | 確認 Dockerfile 有 `cp engine` 那行 |
| `Error occurred prerendering page` | API 路由被 Next.js 預渲染，執行時 DB 連不到 | 該路由加 `export const dynamic = "force-dynamic"` |
| `Module not found: Can't resolve` | npm 套件缺漏或 import 路徑錯誤 | 檢查 package.json 和 import 路徑 |
| `GitHub API call failed: Error: Not Found` | Coolify 內部 Git 設定損毀 | 刪除應用重建，不要嘗試修 |
| `Build step skipped` | 同 commit 已有 image cache | Force Deploy (without cache) |
| `the URL must start with the protocol postgresql://` | DATABASE_URL 值不是 PostgreSQL 連線格式 | 從 PostgreSQL 資源頁面取得正確 Connection String |
| `The table public.X does not exist` (P2021) | Dockerfile CMD 缺 prisma db push + seed | CMD 加入 prisma db push + seed，重新部署 |

---

---

## 部署前檢查清單

推送程式碼前逐項確認：

| # | 檢查項 | 說明 |
|---|---|---|
| 1 | Dockerfile CMD | 確認有 `prisma db push --skip-generate && npx tsx prisma/seed.ts && npx next start` |
| 2 | Prisma engine copy | 確認有 `cp libquery_engine-*.so.node .next/server/chunks/` |
| 3 | 新 API 路由 | 有用到 DB 的 Route Handler 加 `export const dynamic = "force-dynamic"` |
| 4 | 本地 build 通過 | `npm run build` 無錯誤 |
| 5 | DATABASE_URL | 以 `postgresql://` 開頭，Buildtime + Runtime 都勾 |
| 6 | Domains | 逗號分隔，https:// 前綴 |
| 7 | GitHub repo | Private 需 GitHub App 已授權 |
| 8 | 環境變數 | 四個都要設（DATABASE_URL, LINE_CHANNEL_ACCESS_TOKEN, LINE_CHANNEL_SECRET, NODE_ENV） |

---

## 部署後驗證

### 必要檢查

```bash
# 1. 後台登入頁
curl -s -o /dev/null -w "%{http_code}" https://line-admin.zhuoye.com.tw/login
# 預期：200

# 2. Webhook（注意：401 是正常，表示簽名驗證有在運作）
curl -s -w "\n%{http_code}" -X POST https://api.zhuoye.com.tw/line/webhook \
  -H "Content-Type: application/json" -d '{"events":[]}'
# 預期：401 + {"error":"Missing signature"}

# 3. 主要頁面可訪問（需要登入的會 redirect 到 login，也回 200）
curl -s -o /dev/null -w "%{http_code}" https://line-admin.zhuoye.com.tw/
curl -s -o /dev/null -w "%{http_code}" https://line-admin.zhuoye.com.tw/settings
curl -s -o /dev/null -w "%{http_code}" https://line-admin.zhuoye.com.tw/users
curl -s -o /dev/null -w "%{http_code}" https://line-admin.zhuoye.com.tw/audit-log
# 預期：全部 200
```

### 容器內診斷

如果外部回應異常，先進容器內確認：

```bash
# Coolify Terminal 或 docker exec
docker exec <容器名> curl -s localhost:3000
```

| 外部回應 | 容器內 curl localhost | 問題方向 |
|---------|---------------------|------|
| 純文字 404 (19B) | 正常 | Traefik/Proxy/Domains 設定問題 |
| HTML 404 (8KB+) | 正常 | Next.js 路由問題 |
| 502/503 | — | 容器未啟動或 crash |
| 連線拒絕 | — | 容器內服務未 listen 或 port 錯誤 |

---

## 鐵則

1. **Repo 一次設對，絕不事後改 Git URL** — 錯了就刪除重建
2. **部署失敗 ≥ 2 次 → 停，搜 GitHub Issues，不盲試**
3. **刪除應用前先備份環境變數** — Coolify 沒有匯出功能
4. **Domains 逗號分隔，不是換行**
5. **同 commit 不重建 → Force Deploy (without cache)**
6. **API 路由有用 DB → 加 `export const dynamic = "force-dynamic"`**
7. **Prisma engine 必須 cp 到 `.next/server/chunks/`**
8. **環境變數 DATABASE_URL 需 Buildtime + Runtime 都勾**
9. **DATABASE_URL 必須從 PostgreSQL 資源頁面取得**，確認以 `postgresql://` 開頭
10. **Dockerfile CMD 必須包含 `prisma db push` + `seed`**，否則資料庫是空的

---

## 2026-06-03 部署時間線（zhuoye-line）

| 時間 | 事件 | 結果 |
|---|---|---|
| 前期 | 開發 Phase 1+2 全部功能，build 通過，commit push | ✓ |
| 第 1 次 | Deploy 舊應用（指向錯誤 repo） | ✗ Git API call failed |
| 第 2 次 | 改 Git URL 為 zhuoye-line | ✗ Not Found |
| 第 3-5 次 | 改 repo public、重設 GitHub App、Force Deploy | ✗ 全部 Not Found |
| 調查 | 搜 GitHub Issues，確認 Coolify bug #8917 | 找到根因 |
| 重建 | 刪除舊應用 → 新建 → Dockerfile → 環境變數 → Domain | 建置成功 |
| 第 1 次 deploy | migrate-admin 路由預渲染失敗 | ✗ Prisma engine 找不到 |
| 修正 | 加 `force-dynamic` + Force Deploy | ✓ 成功 |
| 補 domain | 補上 api.zhuoye.com.tw domain（逗號分隔） | ✓ 頁面正常 |
| 後續測試 | 無法登入，login API 回 500 | ✗ DATABASE_URL 格式錯誤 |
| 修正 DATABASE_URL | 從 PostgreSQL 資源取得正確連線字串 | ✗ table not found |
| 排查 | Log 顯示 `table public.User does not exist` | ✗ CMD 缺 prisma db push + seed |
| 修正 Dockerfile | CMD 加入 db push + seed + Force Deploy | ✓ 全部正常 |

**總耗時**：約 4.5 小時

**根因分析**：
- Coolify Git URL bug：2 小時（平台問題，無可避免）
- DATABASE_URL 格式錯誤：30 分鐘（人為疏失）
- Dockerfile CMD 缺 migration：20 分鐘（設計疏漏，Nixpacks 自動處理了）
- Prisma engine + force-dynamic：30 分鐘（新知識）
- 其他（Domain 格式、環境變數設定等）：1 小時
