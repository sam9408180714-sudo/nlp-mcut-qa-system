# AI 協作反思 (Week 04：資料擷取與拓撲感知分塊)

## 1. 使用的 AI 工具 (Tools Used)
- Claude（Anthropic，claude.ai）

## 2. 主要協作情境 (Use Cases)
- 任務 1-A：請 AI 寫出從檔名萃取法規名稱的 Regex，用來動態清除溢出到內文的頁首頁尾。
- 任務 1-B：請 AI 寫出用捕捉群組合併「中文標題 + 英文標題」的 Regex。
- 任務 1-C：請 AI 寫出合併「中文條文 + 英文條文」清單項目的 Regex，並避免中文條文互相誤併。
- 請 AI 說明 VS Code 的操作步驟，以及 `.gitignore` 的 `**` 萬用字元寫法。

## 3. 關鍵提問紀錄 (Key Prompts)
**Prompt（任務 1-C）：**
> 請用 Python re 寫一個正則表示式，有兩個捕捉群組。群組 1 是某個非空白行，群組 2 是下一個以「- 」開頭的清單項目，中間可能夾著空白行。
> 限制：第二個清單項目在「- 」之後必須以英文字母開頭，這樣「第一條（中文）」和「第二條（中文）」才不會被誤併。
> 替換字串固定為 `\1 / \2`，請讓輸出和下面的 expected_topology_1c 完全相同：（貼上 raw 與 expected 文字）

**為什麼有效：** 這個 Prompt 同時給了「輸入範例、期望輸出、替換字串、防禦限制（英文字母開頭）」，AI 不用猜需求，產出的 Regex 可以直接拿 assert 驗證。

## 4. 人類審查與修正 (Human Oversight & Corrections)
- AI 給的 3 個 Regex 我沒有直接相信，而是先執行 1-A、1-B、1-C 的 assert 斷言，三個都顯示 `[Success]` 才放進批次處理流程。
- 範本中 `metrics.json` 的路徑是 `../../metrics.json`，但我的 Notebook 放在 `notebooks/` 底下，往上一層就是專案根目錄，所以改成 `../metrics.json`。
- 在 CPU 筆電上執行時，docling 預設會跑 OCR 與表格模型，批次處理卡了 40 分鐘以上；因為這些規章 PDF 本身就有文字層，我請 AI 協助關閉 `do_ocr` 與 `do_table_structure`，速度才恢復正常。
- 批次處理只執行一次 `re.sub`，遇到三行以上連續標題時合併不完整，所以改成和 1-B 一樣使用 `while` 迴圈重複合併。

## 5. 總結反思 (Reflection)
- 先寫好 expected 輸出和 assert，再請 AI 產生程式碼，可以馬上判斷 AI 的答案對不對，不會盲目相信。
- Regex 很容易「太貪婪」而吃到不該合併的段落，所以必須加入特徵限制（例如英文字母開頭），並用真實資料檢查清洗後的 md 檔。
- 清洗後的 Cleaned 資料，長度中位數和資訊熵都比 Raw 資料高，代表資料清洗會直接影響後續 NLP 任務的品質。
