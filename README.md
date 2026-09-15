# copilot-claude-setup-windows




# 使用 Windows 系統以 GitHub Copilot 驅動 Claude Code

## 1. 概念釐清：我們究竟在做什麼？

* **Claude Code 是什麼？**  
  它不同於一般的「聊天機器人」，它是一位「代理工程師（AI Agent）」。它直接住在你電腦的黑底白字視窗（終端機）裡，能自己閱讀檔案、幫你建立資料夾，甚至自動寫程式與修復錯誤。
* **我們為什麼需要本地網關（Copilot-API）？**  
  Claude Code 原本只能用官方付費的 API。我們透過一個開源的「中繼橋樑（網關）」，讓 Claude Code 可以借用你現有的 **GitHub Copilot 訂閱額度**來工作，無需額外購買 Anthropic API 點數。



## 2. 事前準備清單（Prerequisites）

在開始前，請確認你已經準備好以下條件：

1. **有效訂閱的 GitHub 帳號**：確認你的 GitHub 帳號已開通 **GitHub Copilot** 服務。
2. **安裝 Node.js**（提供執行 JavaScript 的環境）：
   * 前往 [Node.js 官方網站](https://nodejs.org/) 下載 **LTS 版本**（長期支援版）安裝檔，一路按「Next」完成安裝。
3. **安裝 Git**（版本控制工具，外掛安裝必備）：
   * 前往 [Git for Windows 官網](https://git-scm.com/download/win) 下載 64-bit 版本，一路按「Next」保持預設安裝。



## 3. 完整安裝與設定流程

### 第一步：解除 Windows 系統限制（一次性設定）

Windows 預設會阻擋終端機執行自動化腳本，這會導致後續指令跳出紅字。

1. 點擊 Windows 開始功能表，輸入 `PowerShell`。
2. 在搜尋結果的「Windows PowerShell」上**按滑鼠右鍵**，選擇 **「以系統管理員身分執行」**。
3. 複製並貼上以下指令後按下 Enter：
   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope LocalMachine

```

4. 當畫面詢問是否變更時，輸入字母 `Y` 並按下 Enter。
5. **關閉此管理員視窗**。

---

### 第二步：安裝 Claude Code 工具

1. 按下鍵盤上的 `Win + R`，輸入 `powershell` 並按 Enter，打開**一般的 PowerShell 視窗**。
2. 執行以下指令，將 Claude Code 下載並安裝到電腦中：
```powershell
npm install -g @anthropic-ai/claude-code

```


*安裝過程約需 1～2 分鐘，請耐心等待直到終端機回到可輸入游標。*

---

### 第三步：連結並授權 GitHub Copilot 帳號

1. 在同一個 PowerShell 視窗中，輸入以下指令發起登入授權：
```powershell
npx @jeffreycao/copilot-api@latest auth login --provider copilot

```


2. 終端機會顯示類似以下提示：
```text
i Please enter the code "XXXX-XXXX" in [https://github.com/login/device](https://github.com/login/device)

```


3. 打開瀏覽器前往 [https://github.com/login/device](https://github.com/login/device)。
4. 在網頁上填入終端機中顯示的 8 碼代碼，並點擊綠色的 **Authorize GitHub** 按鈕。
5. 回到終端機，你會看到授權成功的綠字提示。

---

### 第四步：啟動本地網關（中繼橋樑）

1. 在終端機中輸入以下指令啟動服務：
```powershell
npx @jeffreycao/copilot-api@latest start

```


2. 看到畫面顯示 `Listening on http://localhost:4141` 時，代表中繼站已啟動成功！
3. **⚠️ 重要注意事項**：
**這個視窗是網關的引擎，使用 Claude Code 期間請保持開啟，絕對不能關閉！** 你可以將它縮小到背景執行。

---

### 第五步：建立專案與連線設定檔

現在打開**第二個全新的 PowerShell 視窗**：

1. **建立專屬工作資料夾**：
```powershell
New-Item -ItemType Directory -Path "C:\copilot_test" -Force
Set-Location -Path "C:\copilot_test"
New-Item -ItemType Directory -Path ".claude" -Force

```


2. **寫入連線設定檔**（將目標導向本機的 4141 網關）：
```powershell
@'
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:4141",
    "ANTHROPIC_AUTH_TOKEN": "dummy",
    "ANTHROPIC_MODEL": "gpt-4o"
  }
}
'@ | Out-File -FilePath ".claude\settings.json" -Encoding utf8

```


*(註：`ANTHROPIC_MODEL` 可以填入你想使用的 Copilot 模型代號，例如 `gpt-4o`)*

---

### 第六步：啟動 Claude Code 與安裝優化外掛

1. 在專案視窗（`C:\copilot_test`）輸入指令進入 Claude：
```powershell
claude

```


2. **安裝官方擴充套件市場**：
進入 Claude 對話框後，輸入以下指令並送出：
```text
/plugin marketplace add [https://github.com/caozhiyuan/copilot-api.git](https://github.com/caozhiyuan/copilot-api.git)

```


3. **安裝外掛模組**：
依序輸入以下兩行指令（若跳出選單，直接按 Enter 選擇 `Install for you (user scope)` 即可）：
```text
/plugin install agent-inject@copilot-api-marketplace
/plugin install tool-search@copilot-api-marketplace

```



---

## 4. 如何驗證是否成功連線？

1. 在 Claude 對話框中隨意傳送一句話（例如輸入 `哈囉`）。
2. 打開瀏覽器，前往本地儀表板網址：
```text
http://localhost:4141/usage-viewer?endpoint=http://localhost:4141/usage

```


3. 點擊頁面上的 **Refresh**。若在下方 **Request Events** 表格中看到剛才發送的訊息與模型名稱，代表設定 100% 成功！

---

## 5. 日常使用的標準 SOP

未來每次開機要使用時，只需簡化為以下兩步驟：

1. **開啟終端機 1**（啟動網關放背景）：
```powershell
npx @jeffreycao/copilot-api@latest start

```


2. **開啟終端機 2**（進入專案並使用）：
```powershell
cd C:\copilot_test
claude

```



---

## 6. 參考資料與專案來源

* **本地網關專案原始碼（GitHub）**：[caozhiyuan/copilot-api](https://github.com/caozhiyuan/copilot-api)
* **Claude Code 官方文件**：[Anthropic Claude Code Overview](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)

> *本教學由個人實作整理，並借助 AI 輔助編寫潤飾。所提之品牌與商標皆歸屬原公司所有。*


```
