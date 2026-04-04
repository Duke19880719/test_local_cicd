# 本地 CI/CD 配置與 GitHub Actions 自動化部署 (Vue 專案)

本文檔說明了如何使用 **GitHub Actions** 和 **本地部署** 配置 **Vue.js** 專案的 **CI/CD 管道**。

## 前置條件

1. **Node.js**（版本 18.x 或更新版本）。
2. **Vue CLI**（或者使用 **Vite**，取決於你的選擇）。
3. 擁有 **GitHub 帳號** 和一個 **GitHub 儲存庫** 來存放你的程式碼。
4. **GitHub Actions Runner**（用於本地部署）。

---

## 步驟 1：初始化你的 Vue 專案

你可以選擇使用 **Vue CLI** 或 **Vite** 來建立你的 Vue.js 專案。

### 使用 Vue CLI 創建專案

```bash
npm install -g @vue/cli
vue create my-project


步驟 2：設置本地 GitHub Actions Runner

為了讓 GitHub Actions 可以在本地運行（在你的伺服器或機器上），你需要配置一個自託管的 Runner。

創建 Runner 資料夾：
進入 C: 磁碟（或者你選擇的其他磁碟）。
創建一個名為 actions-runner 的資料夾（例如：C:\actions-runner）。
下載並配置 GitHub Actions Runner：
前往你的儲存庫的 Settings > Actions > Runners > Add Runner。
按照 GitHub 提供的步驟，下載相應版本的 runner 並生成註冊令牌。
安裝並配置 Runner：
打開命令提示符（或 PowerShell），進入 C:\actions-runner 資料夾。
按照 GitHub 提供的說明註冊 runner，命令類似這樣：

./config.cmd --url https://github.com/username/repository --token YOUR_TOKEN

啟動 Runner：

運行以下命令啟動 runner：

./run.cmd

此時，runner 會開始等待來自 GitHub Actions 的工作。

步驟 3：創建 .github 資料夾並設置工作流程文件

創建 .github 資料夾：

在你的專案根目錄下創建一個名為 .github 的資料夾，然後在該資料夾內創建一個名為 workflows 的子資料夾。
mkdir -p .github/workflows

添加 GitHub Actions 工作流程文件：

在 .github/workflows 資料夾內創建一個名為 ci-cd.yml 的文件，內容如下：
name: Vue CI/CD (Local Deploy)

on:
  push:
    branches: [ "master" ]

jobs:
  build-and-deploy:
    runs-on: self-hosted  # 使用自託管的 runner

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Deploy to local folder
        shell: cmd
        run: |
          if not exist dist (
            echo dist not found
            exit 1
          )
          xcopy dist C:\deploy\vueapp /E /I /Y

      - name: Start server
        shell: cmd
        run: |
          start /B npx serve -s C:\deploy\vueapp -l 5000

步驟 4：將變更推送到 GitHub

創建完 GitHub Actions 工作流程後，將你的程式碼推送到 GitHub 儲存庫：

git add .github/
git commit -m "Add GitHub Actions CI/CD workflow"
git push origin master

當你推送更改到 master 分支時，GitHub Actions 將自動觸發，並在你的自託管 Runner 上運行管道。

步驟 5：本地部署應用程式

當 CI/CD 管道成功完成後，將自動把構建後的 Vue 應用部署到本地伺服器。

構建好的文件會被複製到 C:\deploy\vueapp 資料夾（如工作流程所定義）。
本地伺服器將使用 npx serve 啟動，並且綁定在端口 5000。

你現在可以在瀏覽器中訪問你的 Vue 應用，網址為 http://localhost:5000。

排錯與解決方法
問題：npm install 遇到 Node.js 版本錯誤
問題：

有時候你可能會遇到與 Node.js 版本相容性的錯誤，特別是使用 Vue CLI 時。例如：

Error: Could not find a part of the path 'C:\nvm4w\nodejs'.
解決方法：
確保你已安裝 Node.js v18.x 並將其設為當前活動版本。
若需要，使用 nvm (Node 版本管理器) 切換 Node.js 版本。
如果是在自託管的 runner 中運行，請確保 runner 環境變數和路徑設置正確。
問題：Runner 配置錯誤
問題：

如果你在設置 GitHub Actions runner 時遇到問題，例如 "Could not find a part of the path" 或令牌註冊錯誤。

解決方法：
確保 runner 正確配置並已註冊到你的 GitHub 儲存庫中。再次檢查令牌和儲存庫 URL。
使用 config.cmd 和 run.cmd 腳本來手動配置和啟動 runner。

心得: vue cli 和 nodejs 的版本問題多多，感覺用vite 會好一點。