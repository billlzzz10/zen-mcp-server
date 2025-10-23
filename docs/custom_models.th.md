# การตั้งค่าโมเดลกำหนดเองและ API

คู่มือนี้อธิบายการตั้งค่าโมเดลจากหลายผู้ให้บริการ เช่น OpenRouter, ปลายทาง API กำหนดเอง และเซิร์ฟเวอร์โมเดลในเครื่อง Zen MCP Server ใช้รีจิสทรีโมเดลแบบรวมเพื่อจัดการทั้งหมดผ่านคอนฟิกเดียว

## ผู้ให้บริการที่รองรับ

- **OpenRouter** – เข้าถึงโมเดลเชิงพาณิชย์ (GPT-4, Claude, Mistral ฯลฯ) จาก API เดียว
- **ปลายทางกำหนดเอง** – โมเดลในเครื่อง (Ollama, vLLM, LM Studio, text-generation-webui)
- **API ที่เข้ากับ OpenAI** – ปรับใช้เองหรือ API เอกชนที่ใช้รูปแบบเดียวกัน

## เลือกใช้อะไรดี

**ใช้ OpenRouter เมื่อ**
- ต้องการโมเดลที่ API ต้นทางไม่มี
- อยากรวมบิลหลายผู้ให้บริการไว้ที่เดียว
- อยากทดลองหลายโมเดลโดยไม่ต้องสมัครคีย์หลายชุด

**ใช้ Custom URL เมื่อ**
- ต้องการโมเดลในเครื่อง (Ollama, vLLM, LM Studio)
- มีเซิร์ฟเวอร์ inference ที่ตั้งเอง
- มี API ส่วนตัวที่ใช้รูปแบบ OpenAI
- ต้องการควบคุมค่าใช้จ่ายบนฮาร์ดแวร์ของตัวเอง

**ใช้ API ต้นทาง (Gemini/OpenAI ฯลฯ) เมื่อ**
- ต้องการเข้าถึงฟีเจอร์ล่าสุดโดยตรง
- อาจได้ latency และราคาที่ดีกว่า
- ต้องการการสนับสนุนจากผู้ให้บริการโดยตรง

**ผสมได้**: จะใช้ OpenRouter + โมเดลในเครื่อง + API ต้นทางพร้อมกันก็ได้ (ระบบจะให้ความสำคัญ API ต้นทางก่อน หากชื่อโมเดลซ้ำ)

## alias ของโมเดล

Zen มีไฟล์รีจิสทรีหลายชุด:
- `conf/openai_models.json`
- `conf/gemini_models.json`
- `conf/xai_models.json`
- `conf/openrouter_models.json`
- `conf/dial_models.json`
- `conf/custom_models.json`

คัดลอกไฟล์ที่ต้องใช้ แล้วตั้งตัวแปร `*_MODELS_CONFIG_PATH` ให้ชี้ไฟล์ที่แก้ไข เพื่อประกาศโมเดล/alias ที่ต้องการ

### โมเดล OpenRouter (คลาวด์)

ตัวอย่าง alias ที่มีอยู่ใน `conf/openrouter_models.json`:

| Alias | โมเดลจริง | จุดเด่น |
|-------|-----------|---------|
| `opus`, `claude-opus` | `anthropic/claude-opus-4.1` | Claude รุ่นเรือธงพร้อม vision |
| `sonnet`, `sonnet4.5` | `anthropic/claude-sonnet-4.5` | Claude สมดุล บริบทสูง |
| `haiku` | `anthropic/claude-3.5-haiku` | Claude รุ่นเร็วพร้อม vision |
| `pro`, `gemini` | `google/gemini-2.5-pro` | Gemini frontier พร้อม thinking mode |
| `flash` | `google/gemini-2.5-flash` | Gemini ตอบไวพร้อม vision |
| `mistral` | `mistralai/mistral-large-2411` | Mistral frontier (ข้อความ) |
| `llama3` | `meta-llama/llama-3-70b` | โมเดลเปิดขนาดใหญ่ |
| `deepseek-r1` | `deepseek/deepseek-r1-0528` | โมเดล reasoning จาก DeepSeek |
| `perplexity` | `perplexity/llama-3-sonar-large-32k-online` | โมเดลที่เสริมการค้นหา |

ตรวจไฟล์ JSON เพื่อดู alias และฟีเจอร์ทั้งหมด ถ้ามีโมเดลใหม่จาก OpenRouter ให้เพิ่มเองได้

### โมเดล Custom/Local

| Alias | ชื่อโมเดลในเครื่อง | หมายเหตุ |
|-------|--------------------|-----------|
| `local-llama`, `local` | `llama3.2` | ต้องตั้ง `CUSTOM_API_URL` |

โมเดลในเครื่องประกาศผ่าน [`conf/custom_models.json`](conf/custom_models.json) ส่วน OpenRouter ประกาศใน [`conf/openrouter_models.json`](conf/openrouter_models.json)

ไฟล์รีจิสทรีของผู้ให้บริการอื่นใช้ schema เดียวกัน การแก้ไขไฟล์เหล่านี้ช่วยให้คุณ
- เพิ่ม alias (เช่น `enterprise-pro` → `gpt-5-pro`)
- ประกาศฟีเจอร์ใหม่เมื่อผู้ให้บริการเพิ่มความสามารถ
- ปรับขนาดบริบทเมื่อมีการอัปเดต

> อย่าลืมรีสตาร์ทเซิร์ฟเวอร์หลังแก้ไฟล์ JSON เพื่อโหลดค่าใหม่

คะแนน `intelligence_score` ในแต่ละ entry มีผลกับลำดับที่แนะนำใน auto mode (ดู [Model Ranking](model_ranking.md))

**หมายเหตุ:** หากเรียกโมเดลด้วยชื่อเต็มที่ไม่มีในไฟล์ ระบบจะใช้ค่ามาตรฐาน (บริบท 32K ไม่มี extended thinking ฯลฯ) ซึ่งอาจไม่ตรงกับจริง แนะนำให้เพิ่มโมเดลใหม่ลงรีจิสทรีเพื่อบอกความสามารถที่ถูกต้อง

## วิธีตั้งค่าอย่างรวดเร็ว

### ทางเลือก 1: ใช้ OpenRouter

1. สมัครและสร้าง API key ที่ [openrouter.ai](https://openrouter.ai)
2. ตั้งค่า `.env`:
   ```env
   OPENROUTER_API_KEY=your-key
   DEFAULT_MODEL=auto
   # เลือกโมเดลที่อนุญาต (เมื่อจำเป็น)
   OPENROUTER_ALLOWED_MODELS=opus,sonnet,mistral
   ```
3. (เลือกได้) ปรับ alias/ฟีเจอร์ใน `conf/openrouter_models.json`

### ทางเลือก 2: ใช้โมเดลในเครื่อง (เช่น Ollama)

```env
CUSTOM_API_URL=http://localhost:11434/v1
CUSTOM_API_KEY=
CUSTOM_MODEL_NAME=llama3.2
DEFAULT_MODEL=auto
```

พร้อมเพิ่มรายการใน `conf/custom_models.json` สำหรับโมเดลที่ใช้ เช่น `llama3.2`

## ตัวอย่างการตั้งค่าปลายทาง OpenAI-compatible

**Ollama:**
```bash
CUSTOM_API_URL=http://localhost:11434/v1
CUSTOM_API_KEY=
CUSTOM_MODEL_NAME=llama3.2
```

**vLLM:**
```bash
CUSTOM_API_URL=http://localhost:8000/v1
CUSTOM_API_KEY=
CUSTOM_MODEL_NAME=llama3.2
```

**LM Studio:**
```bash
CUSTOM_API_URL=http://localhost:1234/v1
CUSTOM_API_KEY=lm-studio
CUSTOM_MODEL_NAME=local-model
```

**text-generation-webui:**
```bash
CUSTOM_API_URL=http://localhost:5001/v1
CUSTOM_API_KEY=
CUSTOM_MODEL_NAME=your-loaded-model
```

## การเรียกใช้งานโมเดล

**ด้วย alias:**
```
"Use opus for deep analysis"         → anthropic/claude-opus-4
"Use sonnet to review this code"     → anthropic/claude-sonnet-4
"Use pro via zen"                    → google/gemini-2.5-pro
"Use mistral via zen"                → mistral/mistral-large
"Use local-llama to analyze"         → llama3.2 (local)
```

**ด้วยชื่อเต็ม:**
```
"Use anthropic/claude-opus-4 via zen"
"Use openai/gpt-4o via zen"
"Use deepseek/deepseek-coder via zen"
"Use llama3.2 via zen"
```

## ลอจิกเลือกผู้ให้บริการ

ระบบจะเลือกผู้ให้บริการตามลำดับนี้:
1. โมเดลที่ประกาศใน `conf/custom_models.json` → ส่งไปปลายทาง `CUSTOM_API_URL`
2. โมเดลใน `conf/openrouter_models.json` → ใช้ OpenRouter
3. โมเดลที่เหลือ → เลือกจากรูปแบบชื่อ/ผู้ให้บริการอื่นตามคีย์ที่ตั้งไว้

ลำดับความสำคัญโดยรวม:
1. API ต้นทาง (Google, OpenAI, X.AI ฯลฯ)
2. ปลายทาง custom/local
3. OpenRouter

## การตั้งค่าไฟล์ JSON

แต่ละ entry ในไฟล์รีจิสทรีใช้ schema เดียวกับ [`ModelCapabilities`](../providers/shared/model_capabilities.py)

**ตัวอย่าง OpenRouter:**
```json
{
  "model_name": "vendor/model-name",
  "aliases": ["short-name"],
  "context_window": 128000,
  "supports_extended_thinking": false,
  "supports_json_mode": true,
  "supports_function_calling": true,
  "description": "Model description"
}
```

**ตัวอย่างโมเดล local:**
```json
{
  "model_name": "my-local-model",
  "aliases": ["local-model", "custom"],
  "context_window": 128000,
  "supports_extended_thinking": false,
  "supports_json_mode": false,
  "supports_function_calling": false,
  "description": "My custom Ollama/vLLM model"
}
```

คำอธิบายฟิลด์:
- `model_name`: ชื่อโมเดลจริง (เช่น `vendor/model` หรือ `llama3.2`)
- `aliases`: ชื่อสั้นที่ผู้ใช้เรียกได้
- `context_window`: โทเคนรวม input+output
- `supports_extended_thinking`: รองรับ reasoning mode หรือไม่
- `supports_json_mode`: ให้ JSON ได้เสมอหรือไม่
- `supports_function_calling`: รองรับ function/tool calling หรือไม่
- `description`: คำอธิบายสั้น ๆ

## โมเดลยอดนิยม

- GPT-4 (OpenAI)
- Claude (Opus/Sonnet/Haiku)
- Mistral Large
- Llama 3
- ดูเพิ่มเติมที่ [openrouter.ai/models](https://openrouter.ai/models)

## การแก้ปัญหา

- **"Model not found"**: ตรวจชื่อที่ openrouter.ai/models
- **"Insufficient credits"**: เติมเครดิตในบัญชี OpenRouter
- **"Model not available"**: ตรวจสิทธิ์ในแดชบอร์ด OpenRouter
