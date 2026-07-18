# Aider 安裝與配置指南 (Termux 環境)

## 目錄
1. [概述](#概述)
2. [風險提示](#風險提示)
3. [安裝步驟](#安裝步驟)
   - [步驟一：安裝基礎套件](#步驟一安裝基礎套件)
   - [步驟二：建立相容性替身模組（Mocking）](#步驟二建立相容性替身模組mocking)
   - [步驟三：修改 Aider 原始碼](#步驟三修改-aider-原始碼)
   - [步驟四：配置 API 端點與快捷腳本](#步驟四配置-api-端點與快捷腳本)
4. [日常使用流程](#日常使用流程)

---

## 概述

本指南介紹如何在全新的 Termux 環境（Python 3.14+）中安裝並配置 Aider，並通過修改其原始碼解決與新版套件的相容性問題，最終實現與 GitHub Models 免費推論端點的集成。

**適用場景**：適用於全新安裝的 Termux 環境，並需要重現已驗證的修復流程。

---

## 風險提示

- **原始碼修改**：步驟二和步驟三涉及對第三方套件原始碼的直接修改（monkey patch）。版本更新後需重新套用，且可能因套件內部結構變動而失效。
- **模型名稱不一致**：步驟三第 5 項與步驟 4.2 使本地顯示的模型與實際請求的模型不一致。若目標端點的服務條款禁止此類用法，需自行評估合規性。
- **API 權杖安全性**：權杖（token）屬敏感資訊，請勿外流或提交至公開版本庫。

---

## 安裝步驟

### 步驟一：安裝基礎套件

安裝 Aider 和相容的舊版 OpenAI 用戶端函式庫。

```bash
pip install aider-chat==0.16.0
pip install "openai<1.0.0"
```

**說明**：
1. `aider-chat==0.16.0`：安裝與後續修改步驟相容的 Aider 版本。
2. `openai<1.0.0`：降級安裝舊版 OpenAI 套件，避免新版 API 結構與 Aider 內部呼叫方式不相容。

---

### 步驟二：建立相容性替身模組（Mocking）

部分套件在 Termux 上無法正常編譯或安裝，需手動建立替身模組以繞過檢查。

```bash
echo "def declare_namespace(name): pass" > /data/data/com.termux/files/usr/lib/python3.14/site-packages/pkg_resources.py

mkdir -p /data/data/com.termux/files/usr/lib/python3.14/site-packages/tree_sitter
echo "class Language: pass" > /data/data/com.termux/files/usr/lib/python3.14/site-packages/tree_sitter/__init__.py

cat << 'EOF' > /data/data/com.termux/files/usr/lib/python3.14/site-packages/tree_sitter_languages.py
def get_language(lang):
    return None
def get_parser(lang):
    raise ImportError("Termux 環境跳過 tree-sitter 解析")
EOF
```

**說明**：
1. 替代 `pkg_resources.declare_namespace` 為空函式，滿足匯入需求。
2. 建立空殼 `tree_sitter` 模組，解決原生二進位元件無法編譯的問題。
3. 建立 `tree_sitter_languages` 替身，主動拋出例外以略過語法解析。

---

### 步驟三：修改 Aider 原始碼

修改 Aider 的內部邏輯，允許載入未知模型名稱，並跳過上下文長度與白名單限制。

```bash
sed -i 's/raise ValueError(f"Unknown context window size for model: {name}")/self.max_context_tokens = 4096/g' /data/data/com.termux/files/usr/lib/python3.14/site-packages/aider/models/openai.py
sed -i 's/self.max_context_tokens = tokens \* 1024/self.max_context_tokens = 4096 * 1024/g' /data/data/com.termux/files/usr/lib/python3.14/site-packages/aider/models/openai.py
sed -i 's/self.tokenizer = tiktoken.encoding_for_model(name)/self.tokenizer = tiktoken.get_encoding("cl100k_base")/g' /data/data/com.termux/files/usr/lib/python3.14/site-packages/aider/models/openai.py
sed -i 's/raise ValueError(f"Unsupported model: {name}")/pass/g' /data/data/com.termux/files/usr/lib/python3.14/site-packages/aider/models/openai.py
sed -i 's/res = openai.ChatCompletion.create(\*\*kwargs)/if "model" in kwargs: kwargs["model"] = "gpt-4o-mini"\n    res = openai.ChatCompletion.create(**kwargs)/g' /data/data/com.termux/files/usr/lib/python3.14/site-packages/aider/sendchat.py
```

**說明**：
1. 允許未知模型名稱，並設置固定的上下文長度。
2. 替代 `tiktoken.encoding_for_model`，避免未知模型名稱導致的例外。
3. 移除模型白名單限制。
4. 覆寫 `kwargs["model"]`，將模型名稱改為 `gpt-4o-mini`。

---

### 步驟四：配置 API 端點與快捷腳本

#### 4.1 設定環境變數

```bash
echo 'export OPENAI_API_BASE="https://models.inference.ai.azure.com"' >> ~/.bashrc
echo 'export OPENAI_API_KEY="您的GitHub_Token"' >> ~/.bashrc
source ~/.bashrc
```

**說明**：
1. 設定 API 請求端點為 GitHub Models 推論服務。
2. 替換 `您的GitHub_Token` 為實際的 `ghp_` 開頭權杖字串。

#### 4.2 建立模型切換腳本

```bash
cat << 'EOF' > ~/switch_model.sh
#!/bin/bash
# 模型切換工具
GREEN='\033[0;32m'
CYAN='\033[0;36m'
YELLOW='\033[1;33m'
NC='\033[0m'
echo -e "${CYAN}==================================================${NC}"
echo -e "${YELLOW}      Aider 模型一鍵切換工具 (GitHub Models)      ${NC}"
echo -e "${CYAN}==================================================${NC}"
echo -e " [1] GPT-4o (完整版)"
echo -e " [2] GPT-4o-mini (輕量版)"
echo -e " [3] Phi-3.5-MoE (微軟專家開源模型)"
echo -e " [4] Phi-3.5-mini (微軟輕量模型)"
echo -e " [5] Llama-3-70b (Meta 強大模型)"
echo -e " [6] Llama-3-8b (Meta 輕量模型)"
echo -e " [7] Mistral-Large (Mistral 旗艦模型)"
echo -e " [8] Codestral (程式專用模型)"
echo -e "${CYAN}==================================================${NC}"
read -p "請輸入數字選擇模型: " choice
case $choice in
    1) target="gpt-4o";;
    2) target="gpt-4o-mini";;
    3) target="Phi-3.5-MoE-instruct";;
    4) target="Phi-3.5-mini-instruct";;
    5) target="meta-llama-3-70b-instruct";;
    6) target="meta-llama-3-8b-instruct";;
    7) target="mistral-large";;
    8) target="codestral";;
    *) echo "取消"; exit 0;;
esac
sed -i "s/kwargs\[\"model\"\] = \"[^\"]*\"/kwargs[\"model\"] = \"$target\"/g" "/data/data/com.termux/files/usr/lib/python3.14/site-packages/aider/sendchat.py"
echo -e "\n${GREEN}[成功] 已將遠端模型切換為: $target${NC}"
EOF
chmod +x ~/switch_model.sh
```

#### 4.3 建立啟動腳本

```bash
cat << 'EOF' > ~/a
#!/bin/bash
aider --skip-model-availability-check true --model gpt-4 --edit-format diff "$@"
EOF
chmod +x ~/a
```

---

## 日常使用流程

1. 執行 `~/switch_model.sh`，選擇欲使用的模型。
2. 進入 Git 專案資料夾，執行 `~/a` 啟動 Aider。

---

## 風險提示

- 步驟二、三涉及對第三方套件原始碼的直接修改（monkey patch），版本更新後需重新套用，且可能因套件內部結構變動而失效。
- 步驟三第 5 項與步驟 4.2 使本地顯示模型與實際請求模型不一致，若目的端點的服務條款禁止此類用法，需自行評估合規性。
- 權杖（token）屬敏感資訊，請勿外流或提交至公開版本庫。
