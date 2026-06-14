# 備份系統總覽（卓燁所有專案）

> 最後更新：2026-06-15（VPS 月付轉年付遷移後重建，舊 R2 unified 架構已淘汰）
> **本檔是所有專案備份的唯一計劃，各專案不另存備份文件。**

## 兩條備份線

```
【資料庫】新機 VPS ─加密AES256─→ R2        每日 03:00（留 7 天，主線）
                   └加密AES256─→ Dropbox   每 3 天 04:00（留 30 天，異地冗餘）

【程式碼】本機 Mac ─git bundle─→ Dropbox   每日 02:00 ＋ GitHub（即時）
```

- **資料庫**＝有狀態真資料，加密完整快照；R2 為主、Dropbox 異地冗餘，每份獨立可還原
- **程式碼**＝無狀態，靠 git bundle ＋ GitHub，不需加密

## VPS 資料庫備份（新機 8.217.40.74，`ssh newvps`）

- 統一腳本：`/opt/backups/vps-unified-backup.sh`（一支、兩模式）
  - `… r2`：每日 03:00 → R2，`rclone delete --min-age 7d`（留 7 天）
  - `… dropbox`：每 3 天 04:00 → Dropbox，`--min-age 30d`（留 30 天）
- 加密密碼：`/opt/backups/.backup-pass`（AES-256、root 600、使用者持有）— **務必異地保管一份副本；VPS 掛掉還原要用**
- 排程：root `crontab -l`；日誌 `/var/log/unified-backup.log`
- 備份內容（4 份，各獨立可還原）：

  | 名稱 | 來源 | 容器 |
  |---|---|---|
  | monday-dashboard | SQLite `/data/dev.db` | monday app（service `vegojpuupqnunsa99qn8h8y3-*`）|
  | zhuoye_line | PostgreSQL | `postgres-main`（alias k2rx9…）|
  | zhuoye_crm | PostgreSQL（正式 628 客戶庫）| `postgres-main` |
  | coolify | PostgreSQL（平台/部署設定，重建用）| `coolify-db` |

- 工具：rclone（remote `r2:` ＋ `dropbox:`）＋ `docker exec pg_dump`/gzip ＋ openssl
- 目的地：R2 `r2:zhuoye-postgres-backup/daily/`；Dropbox `dropbox:VPS/snapshots/`

## 本機程式碼備份（Mac）

- 腳本：`~/opt/project-backup.sh`｜排程：LaunchAgent `~/Library/LaunchAgents/com.zhuoye.project-backup.plist`（每日 02:00）｜日誌：`~/opt/project-backup.log`
- 內容：各專案 `git bundle` ＋ `.env` → `dropbox:projects/<日期>/`
- 已驗證涵蓋：zhuoye-crm、zhuoye-line、zhuoye-website（GitHub 為即時鏡像，見下）

## GitHub（程式碼即時備份）

`kenmchang1212/`：zhuoye-website、zhuoye-line、zhuoye-crm、monday-dashboard、zhuoye-workspace-config（~/Project 工作區設定）

## 還原

### 資料庫（加密）
```bash
# 1) 下載（R2 或 Dropbox 擇一）
ssh newvps 'rclone copy r2:zhuoye-postgres-backup/daily/<檔名>.enc /tmp/'
# 2) 解密 + 解壓
openssl enc -d -aes-256-cbc -pbkdf2 -pass file:/opt/backups/.backup-pass -in <檔名>.enc | gunzip > restored.sql
# 3) 匯入
#    PostgreSQL：docker exec -i postgres-main psql -U postgres <dbname> < restored.sql
#    SQLite：    restored 就是 dev.db，停容器 → 替換 /data/dev.db → 啟容器
```
### 程式碼
```bash
rclone copy dropbox:projects/<日期>/<專案>.bundle ./ && git clone <專案>.bundle <專案>
```

## 驗證

- VPS 資料庫：`ssh newvps 'tail /var/log/unified-backup.log'`｜`rclone lsl r2:zhuoye-postgres-backup/daily/`｜`rclone lsl dropbox:VPS/snapshots/`
- 本機程式碼：`launchctl list | grep project-backup`｜`rclone lsl dropbox:projects/`

## 遷移備註（2026-06-15）

- 舊機 47.239.84.14 的備份已**停用**（crontab 清空、`/etc/cron.d/unified-backup` 改名停用）；舊機留退路至 6/17，確認新機無誤後去阿里雲「釋放」（同時停付費）。
- **舊架構已淘汰**（勿再參考）：R2 每日 14 天＋Dropbox 每週 8 週、密碼檔 `/root/.backup-key`、`/opt/vps-unified-backup.sh` 在舊機、含備 `zhuoye-crm` 連字號空殼庫（已刪）。
- 2026-06-15 已實測：R2＋Dropbox 兩模式各 4 庫成功上傳、R2 還原測試通過（解密解壓得正常 SQL）。
