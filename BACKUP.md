# 備份系統總覽

> 最後更新：2026-06-03

## 架構圖

```
VPS PostgreSQL ──→ Cloudflare R2 (daily 3am CST)
              └──→ Dropbox (daily 3am CST)

本機 Mac ──→ Dropbox (daily 2am) — git bundle 全專案
```

## 涵蓋範圍

| 備份對象 | 方式 | 目標 | 頻率 | 保留 |
|----------|------|------|------|------|
| VPS zhuoye_line 資料庫 | pg_dump + gzip | R2 + Dropbox | 每日 03:00 | 14 天 |
| VPS coolify 資料庫 | pg_dump + gzip | R2 + Dropbox | 每日 03:00 | 14 天 |
| zhuoye-website | git bundle --all | Dropbox | 每日 02:00 | 14 天 |
| zhuoye-line | git bundle --all | Dropbox | 每日 02:00 | 14 天 |
| monday-dashboard | git bundle --all | Dropbox | 每日 02:00 | 14 天 |
| investment-monitor | git bundle --all | Dropbox | 每日 02:00 | 14 天 |

## 儲存目標

### Cloudflare R2
- Bucket: `zhuoye-postgres-backup`
- 端點: `e83df7239e68f814ebb958001bfdd973.r2.cloudflarestorage.com`
- 帳號: Kenmchang1212@gmail.com
- 方案: Free（10GB）

### Dropbox
- App: `zhuoye-backup`（App folder 類型，隔離於 `Apps/zhuoye-backup/`）
- 路徑: `backup/`（資料庫）、`projects/`（git bundle）

## VPS 備份

- 腳本: `/opt/pg-backup.sh`
- 排程: `/etc/cron.d/pg-backup`
- 日誌: `/var/log/pg-backup.log`
- 工具: rclone v1.74.2 + pg_dump + gzip
- 設定: `/root/.config/rclone/rclone.conf`

## 本機備份

- 腳本: `/Users/zhangminkai/opt/project-backup.sh`
- 排程: `~/Library/LaunchAgents/com.zhuoye.project-backup.plist`（每日 02:00）
- 日誌: `/Users/zhangminkai/opt/project-backup.log`
- 工具: rclone + git bundle
- 設定: `~/.config/rclone/rclone.conf`

## GitHub 備份

所有專案原始碼同時存在 GitHub：
- `kenmchang1212/zhuoye-website`
- `kenmchang1212/zhuoye-line`（含備份腳本於 `scripts/`）
- `kenmchang1212/monday-dashboard`
- `kenmchang1212/investment-monitor`

## 還原

### 資料庫還原
從 R2 或 Dropbox 下載備份檔，解壓後用 `psql` 匯入。

### 專案還原（異地重建）
從 Dropbox 下載 `projects/日期/` 目錄，內含各專案 `.bundle` 檔：
```bash
git clone 專案名.bundle 專案名
```
另含 `rclone.conf` 可重建備份管道。

## 驗證

- 手動執行 `/Users/zhangminkai/opt/project-backup.sh` 確認無報錯
- VPS: `tail /var/log/pg-backup.log` 查看最近備份紀錄
- Dropbox: `rclone ls dropbox:backup/` 查看資料庫備份
- Dropbox: `rclone ls dropbox:projects/` 查看專案備份
