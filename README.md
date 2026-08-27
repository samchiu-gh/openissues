# 個人工作紀錄 — GitHub Pages + Firebase Firestore 版

## 這個版本跟你原本用的 artifact 差在哪

- 資料存在 Firestore，**任何裝置登入同一個帳號都會即時同步**，不再綁死在單一瀏覽器分頁。
- 加了**登入畫面**，未登入完全看不到、動不了任何資料。這不是可有可無的裝飾——沒有這一層，你的資料在網路上是公開的。
- 目前**沒有主動推播提醒**（到期不會自動 email 或跳通知），只有你打開網頁時才會看到顏色分級。這是刻意先不做，等你確認基本功能能跑再談。

---

## 部署步驟（照順序做，跳步驟會卡關）

### 1. 建立 Firebase 專案
1. 前往 https://console.firebase.google.com ，建立新專案（免費 Spark 方案就夠，不用升級）。
2. 左側選單「建構」→「Firestore Database」→建立資料庫，**選「正式版模式（production mode）」**，地區選離你近的（asia-east1 台灣/東亞可）。
3. 左側選單「建構」→「Authentication」→開始使用→啟用「電子郵件/密碼」登入方式。
4. 在「Authentication」→「Users」分頁，**手動新增你自己一組帳號**（email + 密碼）。這個網站不開放自行註冊，只有你手動建立的帳號能登入——這是刻意的，別人猜不到帳密就進不去。

### 2. 取得 Firebase 設定金鑰
1. 專案總覽頁 → 齒輪圖示「專案設定」→ 拉到最下面「你的應用程式」→ 點「網頁」圖示（`</>`）建立一個網頁應用。
2. 會看到一段 `firebaseConfig = {...}`，複製起來。
3. 打開 `index.html`，找到這段：
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     ...
   };
   ```
   整段換成你剛剛複製的內容。

### 3. 套用安全規則
1. Firestore Database → 「規則」分頁。
2. 把內容整個換成本資料夾裡 `firestore.rules` 的內容，按「發布」。
3. **這一步不能省略。** 沒套這個規則，你的資料庫預設是「正式模式」會拒絕所有讀寫（等於網站打不開資料），或如果你當初選了「測試模式」，則是「任何人都能讀寫」（等於公開資料庫）。兩種預設都不是你要的。

### 4. 部署到 GitHub Pages
1. 在 GitHub 建一個新的 repository（可以設為 Public，因為前端程式碼公開沒關係，真正的防線在 Firestore 規則）。
2. 把 `index.html` 上傳到這個 repo 的根目錄（或用 git push）。
3. repo 的「Settings」→「Pages」→ Source 選「Deploy from a branch」→ branch 選 `main`／資料夾選 `/ (root)`→ Save。
4. 等 1-2 分鐘，GitHub 會給你一個網址，格式類似 `https://你的帳號.github.io/repo名稱/`，打開就能看到登入畫面。

---

## 之後要加「到期自動提醒」的話

現在的免費 Firebase 方案（Spark）**不能跑排程函式**（Cloud Functions 的定時觸發需要升級到 Blaze 付費方案，即使你用量在免費額度內也要綁信用卡）。如果你不想開通 Blaze，另一條免費路是：

- 用 **GitHub Actions 的排程功能**（對 public repo 完全免費），寫一支小程式每天固定時間呼叫 Firestore REST API 檢查有沒有到期/逾期項目，再用免費的發信服務（如 Resend 每月有免費額度）寄信給你。

這是下一步，確認這版能正常登入、新增、同步之後我們再接。
