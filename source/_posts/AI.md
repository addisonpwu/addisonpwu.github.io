---
title: DeepSeek-R1為什麼會成功
date: 2024/01/27
comments: true
categories: AI
tags: [DeepSeek]

---
# 在本地部署 DeepSeek R1：使用 Ollama 的完整指南  

## 引言  
DeepSeek R1 的發佈標誌著開源大語言模型領域的一次重大突破。其性能不僅媲美 OpenAI 的付費模型 o1，還能通過本地部署實現數據隱私與算力自主控制。本文將結合最新實踐，詳解如何通過 Ollama 部署 DeepSeek R1，並深度解析其技術特性與衍生模型（如 R1 Zero）的核心差異。

---

## 什麼是 DeepSeek R1？  
DeepSeek R1 是由中國 AI 初創公司深度求索（DeepSeek）開發的推理優化模型，基於自研的 DeepSeek-V3 架構，通過**兩階段強化學習（RL）**實現突破性性能：  
1. **純 RL 預訓練**：無需監督數據，僅通過基於規則的獎勵系統（如答案準確性和邏輯鏈完整性）驅動模型自我進化。  
2. **監督微調（SFT）+ RLHF**：引入長思維鏈（CoT）數據穩定模型輸出，並通過人類反饋強化學習提升安全性與實用性。  

其核心優勢包括：  
- **開源免費**：完整代碼與權重公開，支持商業用途。  
- **參數靈活性**：提供 1.5B 至 671B 的蒸餾版本，適配從個人筆記本到企業級服務器的硬件。  
- **推理專精**：在數學證明（如拉格朗日中值定理推導）、代碼生成等任務中表現卓越，AIME 2024 數學基準得分超越 OpenAI-o1。  

---

## 使用 Ollama 部署 DeepSeek R1 的實戰步驟  

### 1. 環境準備與 Ollama 安裝  
- **硬件要求**：  
  | 模型版本 | 最低顯存 | 推薦 GPU |  
  |----------|----------|----------|  
  | 1.5B     | 4GB      | RTX 3050 |  
  | 7B/8B    | 6-10GB   | RTX 3060 |  
  | 14B      | 12GB+    | RTX 3090 |  
  | 32B+     | 24GB+    | A100/H100|   

- **安裝 Ollama**：  
  - **Windows/macOS**：訪問 [Ollama 官網](https://ollama.com) 下載安裝包，默認路徑需預留 ≥12GB 磁盤空間（建議 C 盤）。  
  - **Linux**：命令行一鍵安裝：  
    ```bash  
    curl -fsSL https://ollama.com/install.sh | sh  
    ```  
  - 驗證安裝：  
    ```bash  
    ollama --version  # 輸出版本號即成功  
    ```  

### 2. 模型下載與版本選擇  
可在這https://ollama.com/search捜尋更多模型，如deepseek-r1:1.5b、deepseek-r1:7b...

通過 Ollama 拉取模型時需**顯式指定版本**（默認下載 7B）：  
```bash  
# 示例：下載 8B 版本（適合 10GB 顯存）  
ollama run deepseek-r1:8b  
```  
若下載中斷，可重複執行命令觸發斷點續傳。  

### 3. 部署驗證與基礎交互  
啟動模型後，可通過命令行直接測試：  
```bash  
ollama run deepseek-r1:8b  
>>> 使用羅爾定理證明拉格朗日中值定理  

# Windows的环境变量监听
OLLAMA_HOST  0.0.0.0

# 启动命令
ollama serve
```  

### 4. 進階應用：可視化與 API 集成  
- **Open Web UI 部署**：  
  ```bash  
  docker run -d -p 3000:8080 --restart always ghcr.io/open-webui/open-webui:main  
  ```  
  訪問 `http://localhost:3000` 即可使用類 ChatGPT 的交互界面。  

- **Python API 調用**：  
  ```python  
  import openai  
  client = openai.Client(base_url="http://localhost:11434/v1", api_key="ollama")  
  response = client.chat.completions.create(  
    model="deepseek-r1:8b",  
    messages=[{"role": "user", "content": "寫一個Python貪吃蛇遊戲"}]  
  )  
  ```  
  需注意：複雜任務（如遊戲開發）可能耗時數分鐘，建議調整 `temperature` 參數平衡創造力與效率。  

---

## DeepSeek R1 Zero 與 R1 的技術差異  
| 特性                | DeepSeek R1 Zero         | DeepSeek R1              |  
|---------------------|--------------------------|--------------------------|  
| **訓練方法**         | 純強化學習（無監督數據） | RL + 監督微調 + RLHF     |  
| **可讀性**           | 輸出邏輯鏈較混亂         | 步驟清晰，附帶 `<think>` 標籤解析 |  
| **多語言支持**       | 僅英文優化               | 中英文混合處理增強       |  
| **安全過濾**         | 無                       | 通過 RLHF 抑制有害輸出   |  
| **適用場景**         | 研究用途                 | 生產環境                 |  

**關鍵結論**：R1 在 R1 Zero 基礎上，通過引入 CoT 數據微調和第二輪 RL 訓練，解決了邏輯斷裂與安全性問題，更適合實際應用。  

---

## 部署問題排查與優化技巧  
1. **顯存不足**：  
   - 啟用 CPU-GPU 混合推理：`OLLAMA_NUM_GPU=1`（僅限 Linux）。  
   - 使用低精度模式：啟動時添加 `--precision fp16` 參數。  

2. **下載超時**：  
   - 更換國內鏡像源：`OLLAMA_HOST=鏡像地址 ollama run...` 。  

3. **生成速度慢**：  
   - 限制輸出長度：`ollama run --max-tokens 300` 。  

