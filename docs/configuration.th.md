# คู่มือการตั้งค่า

คู่มือนี้ครอบคลุมตัวเลือกการตั้งค่าทั้งหมดของ Zen MCP Server เซิร์ฟเวอร์อ่านค่าผ่านตัวแปรสภาพแวดล้อมที่กำหนดไว้ในไฟล์ `.env`

## การตั้งค่าแบบรวดเร็ว

**Auto Mode (แนะนำ):** ตั้ง `DEFAULT_MODEL=auto` แล้วให้ Claude เลือกโมเดลที่เหมาะในแต่ละงานโดยอัตโนมัติ

```env
DEFAULT_MODEL=auto
GEMINI_API_KEY=your-gemini-key
OPENAI_API_KEY=your-openai-key
```

## สารบัญการตั้งค่าทั้งหมด

### ค่าที่จำเป็น

**โฟลเดอร์โปรเจ็กต์:**
> ใช้ `pwd` ปัจจุบันเป็นฐาน ไม่ต้องตั้งเพิ่ม

### API Key (ต้องมีอย่างน้อยหนึ่งตัว)

> **สำคัญ:** เลือกใช้ OpenRouter หรือ API ผู้ให้บริการโดยตรงอย่างใดอย่างหนึ่ง หากตั้งทั้งคู่ระบบจะสับสนว่าควรเรียกที่ไหน

**ตัวเลือก 1: API โดยตรง (แนะนำหากต้องการเข้าถึงเต็มรูปแบบ)**
```env
GEMINI_API_KEY=your_gemini_api_key_here        # https://makersuite.google.com/app/apikey
OPENAI_API_KEY=your_openai_api_key_here        # https://platform.openai.com/api-keys
XAI_API_KEY=your_xai_api_key_here              # https://console.x.ai/
```

**ตัวเลือก 2: OpenRouter (API เดียวเข้าถึงหลายโมเดล)**
```env
OPENROUTER_API_KEY=your_openrouter_api_key_here  # https://openrouter.ai/
# หากใช้ OpenRouter ให้คอมเมนต์คีย์ของผู้ให้บริการโดยตรงด้านบน
```

**ตัวเลือก 3: ปลายทางที่รองรับ OpenAI API (โมเดลโลคอล)**
```env
CUSTOM_API_URL=http://localhost:11434/v1   # ตัวอย่าง Ollama
CUSTOM_API_KEY=                            # Ollama ไม่ต้องใช้คีย์
CUSTOM_MODEL_NAME=llama3.2                 # โมเดลเริ่มต้น
```

**การเชื่อมต่อโมเดลโลคอล:**
- ใช้ URL `http://localhost:...` ได้เลย เพราะเซิร์ฟเวอร์รันในเครื่อง

### การตั้งค่าโมเดล

**โมเดลเริ่มต้น:**
```env
DEFAULT_MODEL=auto  # ค่าอื่น: pro, flash, o3, o3-mini, o4-mini, ฯลฯ
```

**ข้อมูลโมเดลมาตรฐาน:** อยู่ในไฟล์ JSON ภายใต้ `conf/`
- `conf/openai_models.json`
- `conf/gemini_models.json`
- `conf/xai_models.json`
- `conf/openrouter_models.json`
- `conf/dial_models.json`
- `conf/custom_models.json`

แต่ละไฟล์ระบุฟิลด์ที่รองรับไว้ใน `_README` ของตัวเอง และกำหนด alias, ขีดจำกัด, ฟีเจอร์ เช่น `allow_code_generation` หากต้องปรับแก้ให้คัดลอกไฟล์แล้วอ้างผ่านตัวแปร `*_MODELS_CONFIG_PATH`

| ผู้ให้บริการ | โมเดลหลัก | alias ที่พบได้บ่อย |
|---------------|------------|---------------------|
| OpenAI | `gpt-5`, `gpt-5-pro`, `gpt-5-mini`, `gpt-5-nano`, `gpt-5-codex`, `gpt-4.1`, `o3`, `o3-mini`, `o3-pro`, `o4-mini` | `gpt5`, `gpt5pro`, `mini`, `nano`, `codex`, `o3mini`, `o3pro`, `o4mini` |
| Gemini | `gemini-2.5-pro`, `gemini-2.5-flash`, `gemini-2.0-flash`, `gemini-2.0-flash-lite` | `pro`, `gemini-pro`, `flash`, `flash-2.0`, `flashlite` |
| X.AI | `grok-4`, `grok-3`, `grok-3-fast` | `grok`, `grok4`, `grok3`, `grok3fast`, `grokfast` |
| OpenRouter | ดูใน `conf/openrouter_models.json` (อัปเดตตลอดเวลา) | เช่น `opus`, `sonnet`, `flash`, `pro`, `mistral` |
| Custom | กำหนดเอง เช่น `llama3.2` | ตั้ง alias เองได้ต่อรายการ |

> คำแนะนำ: คัดลอกไฟล์ JSON ที่ต้องแก้ → ปรับค่าตามต้องการ → ตั้ง `*_MODELS_CONFIG_PATH` ให้ชี้ที่ไฟล์ใหม่ เพื่อเปิด/ปิดฟีเจอร์ เช่น JSON mode, function calling, การสร้างโค้ด

### การอนุญาตสร้างโค้ด (`allow_code_generation`)

ตั้งในไฟล์ manifest ของโมเดล เพื่อเปิดคำสั่งสร้างโค้ดขนาดใหญ่ผ่านเครื่องมือ `chat`

```json
{
  "model_name": "gpt-5",
  "allow_code_generation": true,
  ...
}
```

**ควรเปิดเมื่อ:**
- โมเดลนั้นทรงพลังมากกว่า CLI หลัก (เช่น GPT-5 เมื่อใช้ Claude Code Sonnet)
- ต้องการให้โมเดลช่วยสร้างโค้ดเต็มชุด แล้ว CLI หลักค่อยรีวิว/ปรับใช้
- ใช้กับงานรีแฟกเตอร์ใหญ่หรือสร้างโมดูลใหม่ทั้งก้อน

**แนวทาง:**
1. เปิดเฉพาะโมเดลที่เชื่อถือได้ ให้ผลลัพธ์สมบูรณ์
2. ตรวจให้แน่ใจว่าค่าอุณหภูมิตั้งเหมาะสม (0.2–0.4)
3. ใช้ร่วมกับ `precommit` เพื่อตรวจโค้ดก่อนคอมมิท

### การจำกัดโมเดล (Model Restrictions)

ใช้ตัวแปร `*_ALLOWED_MODELS` เพื่อจำกัดรายชื่อโมเดลต่อผู้ให้บริการ Auto mode ก็จะเลือกเฉพาะในรายการนี้

```env
GOOGLE_ALLOWED_MODELS=flash,pro
OPENAI_ALLOWED_MODELS=o4-mini,o3
XAI_ALLOWED_MODELS=grok-3,grok-3-fast,grok-4
OPENROUTER_ALLOWED_MODELS=opus,sonnet,mistral
```

**หมายเหตุ:**
- alias ไม่แยกตัวพิมพ์เล็กใหญ่ อ่านจากไฟล์ manifest
- หาก override manifest alias ใหม่จะถูกนำมาใช้ทันที
- หากไม่พบโมเดลใน manifest ระบบจะใช้การตรวจทั่วไป ซึ่งอาจขาดข้อมูลฟีเจอร์บางอย่าง

**ตัวอย่าง:**
```env
# ประหยัดค่าใช้จ่าย
OPENAI_ALLOWED_MODELS=o4-mini
GOOGLE_ALLOWED_MODELS=flash

# ใช้โมเดลเดียวเป็นมาตรฐาน
OPENAI_ALLOWED_MODELS=o4-mini
GOOGLE_ALLOWED_MODELS=pro

# สมดุลความเร็วและคุณภาพ
GOOGLE_ALLOWED_MODELS=flash,pro
XAI_ALLOWED_MODELS=grok,grok-3-fast
```

### การตั้งค่าขั้นสูง

**เปลี่ยนตำแหน่งไฟล์โมเดล:**
```env
OPENAI_MODELS_CONFIG_PATH=/path/to/openai_models.json
GEMINI_MODELS_CONFIG_PATH=/path/to/gemini_models.json
XAI_MODELS_CONFIG_PATH=/path/to/xai_models.json
OPENROUTER_MODELS_CONFIG_PATH=/path/to/openrouter_models.json
DIAL_MODELS_CONFIG_PATH=/path/to/dial_models.json
CUSTOM_MODELS_CONFIG_PATH=/path/to/custom_models.json
```

**การตั้งค่าบทสนทนา:**
```env
CONVERSATION_TIMEOUT_HOURS=5  # อายุความจำ AI-to-AI (ชั่วโมง)
MAX_CONVERSATION_TURNS=20     # จำนวนรอบสูงสุด (1 รอบ = ผู้ใช้ + โมเดล)
```

**การตั้งค่า Logging:**
```env
LOG_LEVEL=DEBUG  # ค่าอื่น: INFO, WARNING, ERROR
```

## ตัวอย่างไฟล์ .env

### โหมดพัฒนา
```env
DEFAULT_MODEL=auto
GEMINI_API_KEY=your-gemini-key
OPENAI_API_KEY=your-openai-key
XAI_API_KEY=your-xai-key
LOG_LEVEL=DEBUG
CONVERSATION_TIMEOUT_HOURS=1
```

### โหมดโปรดักชัน
```env
DEFAULT_MODEL=auto
GEMINI_API_KEY=your-gemini-key
OPENAI_API_KEY=your-openai-key
GOOGLE_ALLOWED_MODELS=flash
OPENAI_ALLOWED_MODELS=o4-mini
LOG_LEVEL=INFO
CONVERSATION_TIMEOUT_HOURS=3
```

### โหมดพัฒนาในเครื่อง
```env
DEFAULT_MODEL=llama3.2
CUSTOM_API_URL=http://localhost:11434/v1
CUSTOM_API_KEY=
CUSTOM_MODEL_NAME=llama3.2
LOG_LEVEL=DEBUG
```

### ใช้ OpenRouter เดียว
```env
DEFAULT_MODEL=auto
OPENROUTER_API_KEY=your-openrouter-key
OPENROUTER_ALLOWED_MODELS=opus,sonnet,gpt-4
LOG_LEVEL=INFO
```

## หมายเหตุสำคัญ

- **การเชื่อมต่อโลคอล**: ใช้ URL localhost ได้โดยตรง
- **ลำดับความสำคัญ API key**: หากตั้งทั้ง native และ OpenRouter ระบบจะเลือก native ก่อน
- **ข้อจำกัดโมเดล**: มีผลกับทุกโหมด (รวม auto) หากปล่อยว่าง = อนุญาตทุกโมเดล
- **ปรับค่าคอนฟิก**: หลังแก้ `.env` ให้รัน `./run-server.sh` เพื่อโหลดค่าล่าสุด

## เอกสารที่เกี่ยวข้อง

- [Advanced Usage Guide](advanced-usage.md) – วิธีใช้โมเดลขั้นสูง, thinking mode, เวิร์กโฟลว์ power user
- [Context Revival Guide](context-revival.md) – การเก็บประวัติและรื้อฟื้นบริบท
- [AI-to-AI Collaboration Guide](ai-collaboration.md) – การประสานงานหลายโมเดล
