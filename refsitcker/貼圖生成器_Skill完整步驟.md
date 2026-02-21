# 🎨 貼圖生成器 Skill - 完整執行步驟

## 📌 Skill 基本資訊

**Skill 名稱：** 貼圖生成器  
**檔案位置：** `scripts/general/貼圖生成器.py`  
**執行方式：** `manage_scripts(action='run', script_name='貼圖生成器', input_data={...})`

---

## 📥 輸入參數

```python
{
  "theme": "主題名稱",           # 選填，不填則從 characters.txt 隨機選取並標記已使用
  "reference_image": "圖片路徑"  # 選填，不填則從 refimage 資料夾隨機選取
}
```

### **參數說明：**

| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `theme` | string | ❌ | 貼圖主題（如：「焦慮的PM-小明」）。不填時自動從 `characters.txt` 隨機選取未使用的角色 |
| `reference_image` | string | ❌ | 參考圖片路徑（如：`data/stickerline/initial/refimage/ref1.png`）。不填時從 `refimage/` 隨機選取 |

---

## 🔄 完整執行流程（7 步驟）

### **Step 0：腳本自動準備**（3-5秒）

**Skill 自動處理：**
1. **主題選擇**
   - 有 `theme` 參數 → 直接使用
   - 無 `theme` 參數 → 從 `data/stickerline/initial/characters.txt` 隨機選取
     - 格式：`【標籤】角色名稱 (描述)`
     - 範例：`【焦慮PM】小明 (每天處理需求變更的專案經理)`
     - 選取後自動標記 `[已使用]`

2. **參考圖片選擇**
   - 有 `reference_image` 參數 → 直接使用
   - 無 `reference_image` 參數 → 從 `data/stickerline/initial/refimage/` 隨機選取

3. **建立工作目錄**
   - 路徑：`data/stickerline/public/{safe_theme}/`
   - 準備檔案：
     - `sticker_list.txt` （文案儲存）
     - `generation_prompts.txt` （圖片 prompt 儲存）

**輸出結果：**
```json
{
  "status": "ready",
  "theme": "焦慮的PM-小明",
  "work_dir": "data/stickerline/public/焦慮的PM-小明",
  "reference_image": "data/stickerline/initial/refimage/ref1.png",
  "style_note": "參考圖片 ... 的視覺風格。可愛卡通角色風格，線條簡潔，表情豐富生動，色彩鮮明",
  "sticker_list_file": "data/stickerline/public/焦慮的PM-小明/sticker_list.txt",
  "prompts_file": "data/stickerline/public/焦慮的PM-小明/generation_prompts.txt",
  "workflow": {
    "step1": "生成 40 個「焦慮的PM-小明」貼圖創意文案",
    "step2": "將文案儲存到 ...",
    "step2.5": "輸出 sticker_list.txt 內容通知使用者",
    "step3": "根據風格生成 40 個圖片 prompt",
    "step4": "批次生成 40 張透明背景圖片",
    "step5": "打包成 ZIP"
  }
}
```

---

### **Step 1：生成 40 個貼圖創意文案**（10-15秒）

根據主題和角色設定，生成 40 個貼圖情境文案

**範例文案格式：**
```
1. 【開會遲到】汗流浹背地跑進會議室 - "抱歉我遲到了！"
2. 【需求又改】崩潰抱頭 - "又要改需求？？"
3. 【加班到深夜】疲憊趴桌 - "今天又要爆肝了..."
...
40. 【週末愉快】放鬆躺平 - "終於可以休息了！"
```

**AI 生成要點：**
- 符合角色人設
- 涵蓋日常溝通場景
- 每條包含：【情境標籤】動作描述 - "對話文字"

---

### **Step 2：儲存文案到檔案**（1秒）

將生成的文案儲存到：`data/stickerline/public/{theme}/sticker_list.txt`

---

### **✨ Step 2.5：輸出文案通知使用者**（5-10秒）⭐

根據使用者所在平台，自動輸出文案內容（**僅通知，自動繼續執行**）

#### **網頁版：**
```
📝 貼圖文案已生成！以下是 40 個文案：

1. 【開會遲到】汗流浹背地跑進會議室 - "抱歉我遲到了！"
2. 【需求又改】崩潰抱頭 - "又要改需求？？"
...
40. 【週末愉快】放鬆躺平 - "終於可以休息了！"

✅ 文案已儲存，接下來將自動生成圖片...
```

#### **Telegram 版：**
透過 `telegram-agent` 發送相同內容

**重點：**
- ✅ 僅通知，不等待確認
- ✅ 自動繼續後續步驟
- ✅ 使用者可隨時喊停修改

---

### **Step 3：生成 40 個圖片 Prompt**（15-20秒）

讀取 `sticker_list.txt`，為每個文案生成對應的圖片生成 prompt

**Prompt 範例：**
```
1. A cute cartoon PM character running into meeting room, sweating and panicking, 
   with speech bubble "Sorry I'm late!", simple line art, transparent background

2. A cute cartoon PM character holding head in frustration, stressed expression,
   with speech bubble "Requirements changed AGAIN??", simple line art, transparent background
```

**儲存位置：** `data/stickerline/public/{theme}/generation_prompts.txt`

---

### **Step 4：批次生成 40 張圖片**（60-90秒）

批次生成圖片，規格：
- **尺寸**：2K (2048x2048)
- **格式**：PNG (透明背景)
- **命名**：`1.png` ~ `40.png`
- **位置**：`data/stickerline/public/{theme}/`

---

### **Step 5：打包成 ZIP 檔案**（2-3秒）

打包內容：
```
{theme}_Stickers.zip
├── 1.png ~ 40.png
└── sticker_list.txt
```

**檔案位置：** `data/stickerline/public/{theme}/{theme}_Stickers.zip`

---

### **Step 6：完成通知**（1秒）

```
✅ 貼圖生成完成！

📦 ZIP 檔案：焦慮的PM-小明_Stickers.zip
📊 包含：40 張貼圖 + 文案清單
📁 工作目錄：data/stickerline/public/焦慮的PM-小明/

🎉 可以直接上傳到 LINE Creators Market！
```

---

## ⏱️ 總執行時間

```
Step 0: 準備          3-5秒
Step 1: 生成文案     10-15秒
Step 2: 儲存文案      1秒
Step 2.5: 通知       5-10秒  ⭐ 僅通知
Step 3: 生成Prompts  15-20秒
Step 4: 生成圖片     60-90秒
Step 5: 打包ZIP       2-3秒
Step 6: 完成通知      1秒
────────────────────────────
總計：            約 2-3 分鐘（全自動）
```

---

## 🎯 使用範例

### **完全自動（推薦）：**
```python
manage_scripts(action='run', script_name='貼圖生成器', input_data={})
```

### **指定主題：**
```python
manage_scripts(
    action='run',
    script_name='貼圖生成器',
    input_data={"theme": "焦慮的PM-小明"}
)
```

### **指定主題 + 參考圖：**
```python
manage_scripts(
    action='run',
    script_name='貼圖生成器',
    input_data={
        "theme": "社畜工程師-阿強",
        "reference_image": "data/stickerline/initial/refimage/ref2.jpeg"
    }
)
```

---

## 📂 檔案結構

```
data/stickerline/
├── initial/
│   ├── characters.txt           # 100 個角色清單
│   └── refimage/                # 參考圖片
│       ├── ref1.png
│       └── ref2.jpeg
└── public/
    └── {theme}/                 # 每個主題一個資料夾
        ├── 1.png ~ 40.png
        ├── sticker_list.txt
        ├── generation_prompts.txt
        └── {theme}_Stickers.zip
```

---

## 🎨 Step 2.5 的重要性

### **為什麼需要 Step 2.5？**

1. **✅ 透明度** - 讓使用者知道生成了什麼文案
2. **✅ 追蹤進度** - 清楚看到執行到哪個階段
3. **✅ 保留彈性** - 使用者可立即喊停修改
4. **✅ 自動化流程** - 不需確認，自動繼續

### **使用者體驗：**
```
使用者：「生成一套貼圖」

AI：✓ Step 0~2 完成

AI：📝 貼圖文案已生成！以下是 40 個文案：
     1. 【開會遲到】...
     2. 【需求又改】...
     ...
     ✅ 接下來將自動生成圖片...

AI：（自動繼續 Step 3~6）

使用者：（可隨時喊停）「等一下！我想修改第 5 條」
```

---

## 📝 相關文件

- **Skill 檔案**：`scripts/general/貼圖生成器.py`
- **完整流程文件**：`docs/貼圖生成器_完整執行流程.md`

---

## 🚀 快速開始

```python
# 執行 Skill（全自動）
manage_scripts(action='run', script_name='貼圖生成器', input_data={})

# 等待 2-3 分鐘

# 取得結果：data/stickerline/public/{theme}/{theme}_Stickers.zip
```

**✨ 現在你可以開始批量生成 LINE 貼圖了！**
