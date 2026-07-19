# Ubuntu 26.04 LTS 下 Aider AI 程式碼助手安裝與完整設定指南

本文件提供在 Ubuntu 26.04 LTS 環境下，從零開始安裝、設定並優化 Aider AI 程式碼助手的完整步驟。內容涵蓋 Python 版本相容性處理、GitHub 免費模型串接、自動化快速啟動設定、預設提示詞配置以及多模型切換腳本。

---

## 一、Python 相容性環境與 Aider 安裝

Ubuntu 26.04 LTS 預設內建較新版本的 Python，可能與 Aider 部分底層套件（如 `tree-sitter`）產生相容性問題。為確保系統穩定性，建議透過 `deadsnakes` PPA 安裝成熟度較高的 Python 3.12，並使用 `pipx` 進行隔離安裝。

請開啟終端機並依序執行以下指令：

```bash
sudo apt update
sudo apt install python3-pip pipx software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.12 python3.12-venv -y
pipx ensurepath
source ~/.bashrc
pipx install --python python3.12 aider-chat
```
行號說明：
1：更新系統本機套件清單。
2：安裝 Python 套件管理工具 `pipx` 與 PPA 軟體源管理工具。
3：將 `deadsnakes` PPA 來源加入系統中。
4：再次更新套件清單以包含新加入的 PPA 來源。
5：安裝指定的 Python 3.12 版本及其虛擬環境建置模組。
6：將 `pipx` 的執行路徑寫入目前使用者的環境變數中。
7：重新載入 Shell 設定檔以套用路徑變更。
8：強制指定 Python 3.12 直譯器，透過 `pipx` 於隔離環境中下載並安裝 Aider。

---

## 二、GitHub Models API 金鑰設定與專案初始化

Aider 支援透過 GitHub Models 提供的免費額度進行開發。使用前必須先完成 GitHub Personal Access Token (PAT) 的申請，並在具備 Git 版本的專案目錄下執行。

### 1. 申請 GitHub Token

1. 登入 GitHub 帳號，前往 **Settings > Developer settings > Personal access tokens > Tokens (classic)**。
2. 點擊 **Generate new token (classic)**。
3. 填寫 Token 名稱，到期日可依需求調整，權限勾選部分保持預設即可（無須額外勾選即可呼叫 GitHub Models API）。
4. 點擊生成並複製該字串（格式以 `ghp_` 開頭）。**請妥善保管，勿外洩此金鑰。**

### 2. 初始化專案與首次啟動

Aider 強烈依賴 Git 進行代碼變更追蹤，請勿直接在家目錄（`~`）啟動。

```bash
export GITHUB_API_KEY="ghp_你的實際金鑰字串"
mkdir my-aider-project
cd my-aider-project
git init
aider --model github/gpt-4o

```

行號：說明
1：設定 LiteLLM 與 Aider 要求的 `GITHUB_API_KEY` 環境變數。
2：建立一個專門用於測試或開發的專案資料夾。
3：切換至該專案資料夾。
4：將該目錄初始化為 Git 儲存庫，供 Aider 追蹤程式碼變更。
5：啟動 Aider 並指定呼叫 GitHub 平台的 `gpt-4o` 模型。若出現上下文視窗未知的警告，直接按 Enter 忽略即可。

---

## 三、環境自動化與快速啟動設定

為了避免每次關閉終端機後都需要重新輸入金鑰與冗長的模型參數，建議將金鑰與指令別名常駐於系統的 `~/.bashrc` 設定檔中。

請執行以下指令進行自動化配置：

```bash
echo 'export GITHUB_API_KEY="ghp_你的實際金鑰字串"' >> ~/.bashrc
echo 'alias aider="aider --model github/gpt-4o"' >> ~/.bashrc
source ~/.bashrc

```

行號：說明
1：將 GitHub API 金鑰以環境變數形式永久追加至變數設定檔尾端。
2：為 Aider 建立簡短別名，未來在任何 Git 專案目錄下只需輸入 `aider` 即會自動帶入模型參數。
3：重新載入設定檔，使所有配置在當前視窗立即生效。

---

## 四、預設提示詞與開發規範配置

若希望 Aider 在每一次對話中都遵循特定的程式碼風格、架構規範或全域提示詞，可以使用 `.aider.conf.yml` 設定檔搭配 Markdown 規範檔來達成。

### 1. 建立全域或專案設定檔

在專案根目錄或家目錄（`~/.aider.conf.yml`）下建立設定檔：

```yaml
model: github/gpt-4o
read:
  - CONVENTIONS.md

```

### 2. 建立規範檔案 `CONVENTIONS.md`

在相同目錄下建立 `CONVENTIONS.md`，並將你的預設提示詞寫入其中。例如：

```markdown
# 開發規範提示詞
- 所有程式碼變更必須優先考慮簡潔性。
- 函式與變數命名必須統一採用 camelCase 格式。
- 數學公式請嚴格使用 LaTeX 語法（行內使用 $...$，獨立區塊使用 $$...$$）。
- 嚴禁在程式碼中加入無意義的註解。

```

---

## 五、多模型切換自動化腳本

若有切換不同平台免費模型（如 Google Gemini、Groq 或 GitHub 的其他開源模型）的需求，可建立自動化互動式腳本。

請建立一個名為 `switchAider.sh` 的檔案：

```bash
#!/bin/bash

declare -A modelMap
modelMap[1]="github/gpt-4o"
modelMap[2]="github/Llama-3.2-90B-Vision-Instruct"
modelMap[3]="github/Mistral-large"
modelMap[4]="gemini/gemini-1.5-pro-latest"
modelMap[5]="groq/llama3-70b-8192"

echo "可用免費/額度模型清單："
echo "1. GitHub: GPT-4o"
echo "2. GitHub: Llama-3.2-90B"
echo "3. GitHub: Mistral-large"
echo "4. Google: Gemini-1.5-Pro (需設定 GEMINI_API_KEY)"
echo "5. Groq: Llama-3-70B (需設定 GROQ_API_KEY)"

read -p "請輸入對應數字 [1-5]: " userChoice

selectedModel=${modelMap[$userChoice]}

if [ -z "$selectedModel" ]; then
    echo "輸入無效，離開程式。"
    exit 1
fi

echo "正在切換至 $selectedModel 並啟動 Aider..."
aider --model "$selectedModel"

```

行號：說明
1：指定直譯器為 bash。
3：宣告一個名為 `modelMap` 的關聯陣列。
4-8：定義數字與各模型完整字串的映射關係。
10-16：於終端機印出人性化的選單介面與前置環境變數提醒。
18：接收使用者的鍵盤輸入。
20：透過關聯陣列將數字轉換為實際的模型字串。
22-25：檢查轉換結果是否為空，若無效則提示並終止執行。
27-28：輸出提示訊息並正式透過 Aider 載入目標模型。

> **執行前準備：** 建立檔案後，請執行 `chmod +x switchAider.sh` 賦予執行權限。若選用 Google 或 Groq 模型，請確保已在系統中設定 `GEMINI_API_KEY` 或 `GROQ_API_KEY` 環境變數。

---

## 六、Aider 內部常用指令對照表

進入 Aider 對話互動介面（`>` 提示符號）後，可透過以 `/` 開頭的控制指令管理上下文與版本控制：

| 指令 | 說明與最佳實務 |
| --- | --- |
| `/add <檔案名稱>` | 將指定檔案加入對話上下文，供模型讀取、分析並進行程式碼改寫。 |
| `/drop <檔案名稱>` | 將檔案移出對話上下文，用於縮減 Token 消耗並避免無關程式碼干擾模型判斷。 |
| `/clear` | 清除當前對話歷史紀錄，在開啟全新功能開發時非常實用。 |
| `/undo` | 撤銷模型上一次產生的程式碼變更，並自動還原該次 Git 提交（Commit）。 |
| `/diff` | 在不離開終端機的情況下，直接檢視目前模型對程式碼所做的具體變更差異。 |
| `/help` | 呼叫系統內建的輔助說明選單。 |
| `/exit` | 安全儲存目前狀態並退出 Aider 互動介面。 |

該說明文檔已經整合了從 Ubuntu 26.04 LTS 基礎環境準備、解決 Python 版本相容性問題、GitHub 模型 API 串接、環境變數持久化設定，以及多模型切換自動化腳本與常用內部指令的所有步驟。您可以直接下載並作為完整的安裝參考手冊使用。

---

## 參考資料

- Python Software Foundation. (n.d.). *Python 3.12 documentation*. Retrieved from https://docs.python.org/3.12/
- GitHub. (n.d.). *Personal access tokens (classic)*. Retrieved from https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token
- Ubuntu. (n.d.). *Ubuntu 26.04 LTS release notes*. Retrieved from https://releases.ubuntu.com/26.04/
- pipx. (n.d.). *Install and Run Python Applications in Isolated Environments*.
- Aider. (n.d.). *Aider Documentation*. Retrieved from https://aider.chat/docs/
