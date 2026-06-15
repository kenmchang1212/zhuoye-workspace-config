> 讀取本檔案後，回覆時必須在開頭標示 【LINE】
>
> 修改任何 LINE API/Webhook 設定前，必須先上網搜尋最新官方文件及常見問題。不得憑記憶直接修改。

# LINE Messaging API 設定全記錄

---

## 使用範圍

| 專案 | 用途 |
|------|------|
| zhuoye-line | LINE Bot 後台「小卓」，收發訊息、Webhook |
| zhuoye-crm | 客戶管理中心，與 zhuoye-line 共用 LINE OA |

---

## 基本設定

### LINE OA（官方帳號）

| 設定 | 說明 |
|------|------|
| Provider | 卓燁聯合會計師事務所 |
| Channel | Messaging API |
| Plan | 需確認目前方案（免費/輕量/標準） |

### Webhook

| 設定 | 值 |
|------|-----|
| Webhook URL | `https://api.zhuoye.com.tw/line/webhook` |
| 對應 Next.js route | `/api/line/webhook` |
| Webhook 驗證 | Channel Secret 簽名驗證 |
| 注意 | Coolify 需設定 `api.zhuoye.com.tw` Domain 並 rewrite |

### Channel Token

| 金鑰 | 用途 | 安全 |
|------|------|------|
| Channel Access Token | Bot 發送訊息、呼叫 LINE API | 只在後端，不暴露前端 |
| Channel Secret | Webhook 簽名驗證 | 只在後端 |

---

## 圖文選單

六格：關於卓燁、聯絡資訊、公司/行號設立、工商變更、記帳報稅、預約諮詢

---

## 踩坑紀錄

（尚無記錄，如有請補充）

---

## 常見問題速查

| 問題 | 可能原因 | 檢查方向 |
|------|---------|---------|
| Webhook 無回應 | Webhook URL 未設定或 rewrite 失效 | LINE Console → Webhook settings |
| 401 Missing signature | Channel Secret 驗證失敗 | 確認環境變數正確 |
| Bot 無法發送訊息 | Channel Access Token 過期或錯誤 | LINE Console 重新發行 |
| 訊息已讀不回 | 關鍵字規則未命中或 Webhook 未處理 | 檢查未命中紀錄 |
