# LEX ARCHIVUM — 部署說明

## 架構
```
GitHub Pages  ──→  使用者瀏覽器
                        ↓
                   Firebase Auth（Google 登入）
                        ↓
                   Firestore（案件資料即時同步）
                        ↓
                   Anthropic API + Google Drive MCP
```

---

## 一、Firebase 設定（5 分鐘）

### 1. 啟用 Google 登入
Firebase Console → Authentication → Sign-in method → Google → 啟用

### 2. 建立 Firestore 資料庫
Firebase Console → Firestore Database → 建立資料庫 → Production mode → 選擇區域（建議 asia-east1）

### 3. 設定 Firestore 安全規則
Firebase Console → Firestore → 規則 → 貼入以下內容後發布：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 每個使用者只能讀寫自己的資料
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

### 4. 取得 Firebase Config
Firebase Console → 專案設定（齒輪）→ 你的應用程式 → SDK 設定 → 複製 firebaseConfig

### 5. 加入授權網域
Firebase Console → Authentication → Settings → 授權網域 → 新增網域：
- `你的帳號.github.io`
- `localhost`（開發用）

---

## 二、GitHub 部署（3 分鐘）

```bash
# 1. 建立新的 GitHub repo（建議設為 private，保護案件資料）
# 2. 將本資料夾的所有檔案推送至 main branch
git init
git add .
git commit -m "Initial deployment"
git remote add origin https://github.com/你的帳號/你的repo.git
git push -u origin main

# 3. 啟用 GitHub Pages
# GitHub repo → Settings → Pages → Source 選 GitHub Actions → 儲存
```

部署完成後，每次 push 到 main 就會自動更新。

---

## 三、第一次開啟（手機 & 電腦均適用）

1. 開啟 `https://你的帳號.github.io/你的repo`
2. 貼入 Firebase Config JSON + Anthropic API Key
3. 用 Google 帳號登入
4. 開始建立案件

**手機安裝為 App：**
- iOS Safari：分享 → 加入主畫面
- Android Chrome：選單 → 加入主畫面

---

## 四、Anthropic API Key 說明

前往 https://console.anthropic.com → API Keys → Create Key

此 Key 存於瀏覽器 localStorage，僅用於呼叫 Anthropic API 分析書狀。
建議在 Anthropic Console 設定 Usage Limits 控制用量。

---

## 五、Google Drive MCP

分析書狀時，系統會透過 Anthropic 的 MCP 功能讀取 Google Drive 檔案。
需要在 Claude.ai 的設定頁面連接 Google Drive 才能使用此功能。

---

## 六、資料安全

- 案件資料存於**您自己**的 Firebase Firestore
- Anthropic API Key 存於各裝置的 localStorage
- Firebase Auth 使用 Google OAuth，無需管理密碼
- Firestore 安全規則確保只有登入的帳號能讀取自己的資料
- 建議將 GitHub repo 設為 **private**

---

## Firestore 資料結構

```
users/
  {uid}/
    cases/
      {caseId}/
        - id: string
        - name: string          # 案件名稱
        - folderId: string      # Google Drive 資料夾 ID
        - folderUrl: string     # 原始連結
        - createdAt: ISO string
        - updatedAt: ISO string
        - files: Array
          - id, name, mimeType  # Drive 檔案資訊
          - status: pending | done | error
          - extracted: Object   # AI 擷取結果
        - caseData:
          - 訴之聲明: string[]
          - 答辯聲明: string[]
          - 請求權基礎: string[]
          - 實務見解: Object[]
          - 判決結果: Object | null
          - 法官心證: string[]
```
