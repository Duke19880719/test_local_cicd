# 📌 專案名稱：test_local_cicd

## 📌 專案說明

這是一個使用 **Vue CLI 建置的 Vue 3 專案**，並搭配 **GitHub Actions + Self‑Hosted Runner** 實作 CI/CD（自動建置 & 本地部署）。

目標是：

✅ 在 commit 推到 master 時自動執行建置流程  
✅ 自動編譯 production bundle  
✅ 部署到本地資料夾  
✅ 启动靜態伺服器

主要技術：

- Vue 3 + Vue CLI  
- GitHub Actions  
- Self‑Hosted Runner（Windows）  
- Node.js 18  
- VS Code 開發

---

## 🗂️ 專案結構
├─ .github
│ └─ workflows
│ └─ deploy.yml # GitHub Actions CI/CD 設定
├─ src # Vue 專案原始碼
├─ public
├─ package.json
├─ README.md
├─ dist # build 輸出（GitHub Actions 會自動產生）
└─ ...


---

## 💡 開發環境需求

| 工具/套件 | 版本 |
|-----------|------|
| Node.js   | 18.x（LTS） |
| npm       | 10.x（配合 Node 18） |
| Vue CLI   | 5.x |
| GitHub Actions | 自動化流程 |
| Self‑Hosted Runner | Windows runner |

---

# 🚀 本地開發流程

1. **安裝 Node 18**
   - 建議安裝 Node 18 LTS 版本（不要太新如 Node 20）  
   - Vue CLI 5 對新 Node 版本尚未完全支援（舊依賴可能限制版本）([github.com](https://github.com/vuejs/vue-cli/issues/7424?utm_source=chatgpt.com))

2. **檢查 Node / npm 版本**

```bash
node -v      # 應為 18.x
npm -v       # 應為 10.x

3. 安裝專案依賴
npm install

4. 本地建置
npm run build

5. 本地啟動
npm run serve

🧪 Self‑Hosted Runner（本地 CI/CD Runner）設定
📍 Windows runner 安裝
在 GitHub Repo → Settings → Actions → Runners
新增 runner，選 Windows
下載 runner，解壓到資料夾（例：C:\actions-runner）
📍 重新 config runner

進入解壓後資料夾：

cd C:\actions-runner
.\config.cmd --url https://github.com/<user>/<repo> --token <TOKEN>
📍 啟動 runner
前台模式（測試）
.\run.cmd
背景服務模式
# 安裝為 Windows 服務
.\svc install

# 啟動
.\svc start
⭐ GitHub Actions Workflow

這是完整的 CI/CD YAML 設定（已修正 Node/部署/Debug）：

name: Vue CI/CD (Local Deploy)

on:
  push:
    branches:
      - master

jobs:
  build-and-deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Use Node.js 18
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Debug Node environment
        run: |
          where node
          node -v
          npm -v
        shell: cmd

      - name: Install dependencies
        run: npm ci
        shell: cmd

      - name: Build
        run: npm run build
        shell: cmd

      - name: Deploy to local folder
        shell: cmd
        run: |
          if not exist dist (
            echo dist not found
            exit 1
          )
          if not exist "C:\deploy\vueapp" (
            mkdir "C:\deploy\vueapp"
          )
          xcopy dist "C:\deploy\vueapp" /E /I /Y

      - name: Start server
        shell: cmd
        run: |
          start /B npx serve -s C:\deploy\vueapp -l 5000
🧠 版本相容性 & Node 版本問題

Vue CLI 與 Node 版本密切相關：

📌 Vue CLI（@vue/cli 5）在某些模組上不完全支援最新 Node 版本（如 Node 20）
部分舊 package 會報出 Engine 不相容錯誤，因此建議使用 Node 18 LTS。(github.com
)

Vue CLI 官方文件也提醒需管理 Node 版本（可用 nvm / nvm‑windows）以確保相容性。(cli.vuejs.org
)

🧩 實作問題紀錄 & 解決方法
❗ 問題 1 — CI npm install 找不到舊 NVM 路徑

錯誤Log

Error: Could not find a part of the path 'C:\nvm4w\nodejs'

原因

runner 當時的 PATH 仍然指向舊 NVM 安裝的 Node 路徑。CI 跑 npm install 時找不到 node.exe。

解法

確認 runner.exe 所在環境安裝正確 Node 18
移除過期的 C:\nvm4w\nodejs 環境變數 / PATH
刪掉 .path 快照讓 runner 重新載入系統 PATH
確保 workflow 使用 actions/setup-node@v4 指定 Node18
❗ 問題 2 — 無法啟動 runner 服務

錯誤Log

'.\svc' 不是 Cmdlet、函數、指令檔或可執行程式

原因 & 解法

這表示 runner 尚未安裝成 Windows 服務。需要先用：

.\svc install
.\svc start
❗ 問題 3 — 本地 Node 版本過新導致依賴不相容

症狀

npm install / build 時會報錯 "The engine "node" is incompatible with this module"。

分析

Vue CLI 內部部分依賴對 Node 版本 engine 有限制，Node 20 可能造成安裝失敗 / 相依問題。

解法

使用 nvm 或 nvm‑windows 降到 Node 18：

nvm install 18
nvm use 18
🧾 總結

這個 CI/CD 實作使用：

✔ Vue CLI + GitHub Action
✔ Self‑Hosted Runner
✔ Node 18 LTS
✔ 本地部署到 Windows 目錄
✔ 自動啟動靜態伺服器