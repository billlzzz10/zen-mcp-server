# เริ่มต้นใช้งาน Zen MCP Server

คู่มือนี้พาคุณติดตั้ง Zen MCP Server ตั้งแต่ศูนย์ ครอบคลุมการติดตั้ง การตั้งค่า และการใช้งานครั้งแรก

## สิ่งที่ต้องมี

- **Python 3.10+** (แนะนำ 3.12)
- **Git**
- **[ติดตั้ง uv](https://docs.astral.sh/uv/getting-started/installation/)** (สำหรับวิธี uvx)
- **ผู้ใช้ Windows**: ต้องมี WSL2 เพื่อใช้ Claude Code CLI

## ขั้นที่ 1: ขอ API Key

ต้องมี API key อย่างน้อยหนึ่งตัว เลือกตามความต้องการ:

### ทางเลือก A: OpenRouter (แนะนำสำหรับผู้เริ่มต้น)
**API เดียวใช้ได้หลายโมเดล**
- สมัครที่ [OpenRouter](https://openrouter.ai/)
- สร้าง API key
- กำหนดเพดานค่าใช้จ่ายในแดชบอร์ด
- ใช้ GPT-4, Claude, Gemini และอื่น ๆ ผ่าน API เดียว

### ทางเลือก B: ผู้ให้บริการโดยตรง

**Gemini (Google):**
- ไปที่ [Google AI Studio](https://makersuite.google.com/app/apikey)
- สร้าง API key
- **หมายเหตุ:** หากต้องใช้ Gemini 2.5 Pro ต้องใช้คีย์แบบชำระเงิน

**OpenAI:**
- ไปที่ [OpenAI Platform](https://platform.openai.com/api-keys)
- สร้าง API key เพื่อใช้ O3, GPT-5

**X.AI (Grok):**
- ไปที่ [X.AI Console](https://console.x.ai/)
- สร้าง API key สำหรับโมเดล Grok

**แพลตฟอร์ม DIAL:**
- ไปที่ [DIAL Platform](https://dialx.ai/)
- สร้าง API key เพื่อเข้าถึงโมเดลแบบไม่ผูกผู้ขาย

### ทางเลือก C: โมเดลโลคอล (ฟรี)

**Ollama:**
```bash
curl -fsSL https://ollama.ai/install.sh | sh
ollama serve
ollama pull llama3.2
```

**ทางเลือกโลคอลอื่น:** vLLM, LM Studio, Text Generation WebUI ฯลฯ

ℹ️ **[ดูคู่มือตั้งค่าโมเดลกำหนดเองเต็มรูปแบบ](custom_models.md)**

## ขั้นที่ 2: ติดตั้ง

เลือกวิธีที่สะดวก:

### วิธี A: ตั้งค่าแบบทันใจด้วย uvx (แนะนำ)

**ต้องติดตั้ง uv ก่อน:** [คู่มือการติดตั้ง](https://docs.astral.sh/uv/getting-started/installation/)

เลือกผู้ช่วยโค้ดที่ใช้ แล้วเพิ่มคอนฟิก:

**Claude Desktop:**
```json
{
  "mcpServers": {
    "zen": {
      "command": "sh",
      "args": [
        "-c",
        "for p in $(which uvx 2>/dev/null) $HOME/.local/bin/uvx /opt/homebrew/bin/uvx /usr/local/bin/uvx uvx; do [ -x \"$p\" ] && exec \"$p\" --from git+https://github.com/BeehiveInnovations/zen-mcp-server.git zen-mcp-server; done; echo 'uvx not found' >&2; exit 1"
      ],
      "env": {
        "PATH": "/usr/local/bin:/usr/bin:/bin:/opt/homebrew/bin:~/.local/bin",
        "GEMINI_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

**Claude Code CLI:** สร้าง `.mcp.json` ที่รากโปรเจ็กต์
```json
{
  "mcpServers": {
    "zen": {
      "command": "sh",
      "args": [
        "-c",
        "for p in $(which uvx 2>/dev/null) $HOME/.local/bin/uvx /opt/homebrew/bin/uvx /usr/local/bin/uvx uvx; do [ -x \"$p\" ] && exec \"$p\" --from git+https://github.com/BeehiveInnovations/zen-mcp-server.git zen-mcp-server; done; echo 'uvx not found' >&2; exit 1"
      ],
      "env": {
        "PATH": "/usr/local/bin:/usr/bin:/bin:/opt/homebrew/bin:~/.local/bin",
        "GEMINI_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

**Gemini CLI:** แก้ `~/.gemini/settings.json`
```json
{
  "mcpServers": {
    "zen": {
      "command": "sh",
      "args": [
        "-c",
        "for p in $(which uvx 2>/dev/null) $HOME/.local/bin/uvx /opt/homebrew/bin/uvx /usr/local/bin/uvx uvx; do [ -x \"$p\" ] && exec \"$p\" --from git+https://github.com/BeehiveInnovations/zen-mcp-server.git zen-mcp-server; done; echo 'uvx not found' >&2; exit 1"
      ],
      "env": {
        "PATH": "/usr/local/bin:/usr/bin:/bin:/opt/homebrew/bin:~/.local/bin",
        "GEMINI_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

**Codex CLI:** แก้ `~/.codex/config.toml`
```toml
[mcp_servers.zen]
command = "bash"
args = ["-c", "for p in $(which uvx 2>/dev/null) $HOME/.local/bin/uvx /opt/homebrew/bin/uvx /usr/local/bin/uvx uvx; do [ -x \"$p\" ] && exec \"$p\" --from git+https://github.com/BeehiveInnovations/zen-mcp-server.git zen-mcp-server; done; echo 'uvx not found' >&2; exit 1"]

[mcp_servers.zen.env]
PATH = "/usr/local/bin:/usr/bin:/bin:/opt/homebrew/bin:$HOME/.local/bin:$HOME/.cargo/bin:$HOME/bin"
GEMINI_API_KEY = "your_api_key_here"
```

> เปิด web-search tool เพื่อให้ `apilookup` ทำงานได้:
```toml
[tools]
web_search = true
```

**Qwen Code CLI:** แก้ `~/.qwen/settings.json`
```json
{
  "mcpServers": {
    "zen": {
      "command": "bash",
      "args": [
        "-c",
        "for p in $(which uvx 2>/dev/null) $HOME/.local/bin/uvx /opt/homebrew/bin/uvx /usr/local/bin/uvx uvx; do [ -x \"$p\" ] && exec \"$p\" --from git+https://github.com/BeehiveInnovations/zen-mcp-server.git zen-mcp-server; done; echo 'uvx not found' >&2; exit 1"
      ],
      "cwd": "/path/to/zen-mcp-server",
      "env": {
        "PATH": "/usr/local/bin:/usr/bin:/bin:/opt/homebrew/bin:~/.local/bin",
        "GEMINI_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

อย่าลืมแทน API key ให้ตรงกับผู้ให้บริการที่ใช้งานจริง

**OpenCode CLI:** แก้ `~/.config/opencode/opencode.json`
```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "zen": {
      "type": "local",
      "command": [
        "/path/to/zen-mcp-server/.zen_venv/bin/python",
        "/path/to/zen-mcp-server/server.py"
      ],
      "cwd": "/path/to/zen-mcp-server",
      "enabled": true,
      "environment": {
        "GEMINI_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

เพิ่มคีย์อื่น ๆ (`OPENAI_API_KEY`, `OPENROUTER_API_KEY` ฯลฯ) ตามที่ใช้

#### ลูกค้า IDE (Cursor & VS Code)

Zen ใช้กับ IDE ที่รองรับ MCP ได้โดยตั้งคำสั่งเหมือนตัวอย่าง CLI

**Cursor IDE**
1. Settings → Integrations → MCP
2. Add MCP Server ใส่ command/args ตามตัวอย่าง และกำหนด environment (เช่น `GEMINI_API_KEY`)
3. Cursor จะเปิดเซิร์ฟเวอร์ตามต้องการอัตโนมัติ

**VS Code (Claude Dev Extension)**
1. ติดตั้งเวอร์ชัน 0.6.0 ขึ้นไป
2. Command Palette → Claude: Configure MCP Servers → Add server
3. กรอก command/args เหมือนตัวอย่าง uvx และเพิ่ม environment ที่ต้องใช้
4. บันทึกแล้ว VS Code จะรีโหลดเซิร์ฟเวอร์เมื่อใช้งาน

> ถ้าอยากย่อคำสั่ง สามารถใช้ `uvx --from git+https://github.com/BeehiveInnovations/zen-mcp-server.git zen-mcp-server` ได้ แต่ต้องแน่ใจว่า `uvx` อยู่ใน PATH ของทุกไคลเอนต์

### วิธี B: โคลนและตั้งค่าในเครื่อง

```bash
git clone https://github.com/BeehiveInnovations/zen-mcp-server.git
cd zen-mcp-server
./run-server.sh        # ดูแลทุกขั้นตอน
./run-server.ps1       # สำหรับ PowerShell
./run-server.sh -c     # แสดงคอนฟิกสำหรับ Claude Desktop
./run-server.sh --help
```

สคริปต์จะ:
- สร้าง virtualenv
- ติดตั้ง dependencies
- สร้างไฟล์ .env
- ตั้งค่า Claude integrations
- แสดงคอนฟิกที่คัดลอกไปใช้ได้ทันที

> หลัง `git pull` ให้รัน `./run-server.sh` ซ้ำ

ผู้ใช้ Windows: ดู [คู่มือ WSL](wsl-setup.md)

## ขั้นที่ 3: ตั้งค่า API Key

### หากใช้ uvx
เพิ่ม API key ในคอนฟิก MCP ตามตัวอย่างก่อนหน้า

### หากโคลนรีโพ
แก้ไฟล์ `.env`
```bash
nano .env
```
เพิ่ม API key (ต้องมีอย่างน้อยหนึ่งตัว)
```env
GEMINI_API_KEY=your-gemini-api-key-here
OPENAI_API_KEY=your-openai-api-key-here
XAI_API_KEY=your-xai-api-key-here
OPENROUTER_API_KEY=your-openrouter-key

DIAL_API_KEY=your-dial-api-key-here
DIAL_API_HOST=https://core.dialx.ai
DIAL_API_VERSION=2024-12-01-preview
DIAL_ALLOWED_MODELS=o3,gemini-2.5-pro

CUSTOM_API_URL=http://localhost:11434/v1
CUSTOM_API_KEY=
CUSTOM_MODEL_NAME=llama3.2
```

## ป้องกันไคลเอนต์ timeout

ไคลเอนต์ MCP บางตัวมี timeout สั้น ให้ตั้งอย่างน้อย 5 นาที (300000 ms)

### Claude Code & Claude Desktop
เพิ่มค่าใน `~/.claude/settings.json` หรือใน `env` ของเซิร์ฟเวอร์
```json
{
  "env": {
    "MCP_TIMEOUT": "300000",
    "MCP_TOOL_TIMEOUT": "300000"
  }
}
```

### Codex CLI
เพิ่มค่าใน `~/.codex/config.toml`
```toml
[mcp_servers.zen]
command = "..."
args = ["..."]
startup_timeout_sec = 300
tool_timeout_sec = 300
```

### Gemini CLI
ตั้งค่าใน `~/.gemini/settings.json`
```json
{
  "mcpServers": {
    "zen": {
      "command": "uvx",
      "args": ["zen-mcp-server"],
      "timeout": 300000
    }
  }
}
```

> เวอร์ชัน 0.2.1+ อาจยังจำกัด ~60 วินาทีสำหรับบาง transport หากยัง timeout ให้แบ่งงาน หรือรออัปเดต

**ข้อควรรู้**
- ไม่ต้องรีสตาร์ทหลังแก้ค่า
- หากตั้งหลาย API key ระบบจะเลือก native API ก่อน OpenRouter
- ตั้ง alias โมเดลได้ที่ [`conf/custom_models.json`](../conf/custom_models.json)

## ขั้นที่ 4: ทดสอบการติดตั้ง

### Claude Desktop
1. รีสตาร์ทแอป
2. เริ่มการสนทนาใหม่
3. ลอง: `"Use zen to list available models"`

### Claude Code CLI
1. ปิดเซสชันเดิม
2. รัน `claude` ในโฟลเดอร์โปรเจ็กต์
3. ลอง: `"Use zen to chat about Python best practices"`

### Gemini CLI
> ปัจจุบัน Zen MCP เชื่อมกับ Gemini CLI ได้ แต่การเรียก tools ยังทำงานไม่สมบูรณ์ ดู [คู่มือ Gemini CLI](gemini-setup.md)

### Qwen Code CLI
1. รีสตาร์ท (`qwen exit`)
2. รัน `qwen mcp list --scope user` ตรวจว่า `zen` เป็น `CONNECTED`
3. ลอง: `"/mcp"` หรือ `"Use zen to analyze this repo"`

### OpenCode CLI
1. รีสตาร์ท OpenCode (หรือใช้คำสั่ง Reload Config)
2. ใน Settings → Tools → MCP ตรวจว่า `zen` เปิดอยู่
3. ลอง: `"Use zen to list available models"`

### Codex CLI
1. รีสตาร์ท CLI
2. เริ่มสนทนาใหม่
3. ลอง: `"Use zen to list available models"`

### คำสั่งตัวอย่าง
```
"Use zen to list available models"
"Chat with zen about the best approach for API design"
"Use zen thinkdeep with gemini pro about scaling strategies"
"Debug this error with o3: [paste error]"
```

> Codex CLI รองรับ MCP ได้ดีและตั้ง environment ให้อัตโนมัติเมื่อใช้สคริปต์ติดตั้ง

## ขั้นที่ 5: เริ่มใช้ Zen

### รูปแบบการใช้งานพื้นฐาน

**ให้ Claude เลือกโมเดล:**
```
"Use zen to analyze this code for security issues"
"Debug this race condition with zen"
"Plan the database migration with zen"
```

**ระบุโมเดลเอง:**
```
"Use zen with gemini pro to review this complex algorithm"
"Debug with o3 using zen for logical analysis"
"Get flash to quickly format this code via zen"
```

**เวิร์กโฟลว์หลายโมเดล:**
```
"Use zen to get consensus from pro and o3 on this architecture"
"Code review with gemini, then precommit validation with o3"
"Analyze with flash, then deep dive with pro if issues found"
```

### สารบัญเครื่องมือแบบย่อ

**🤝 การร่วมมือ:** `chat`, `thinkdeep`, `planner`, `consensus`

**🔍 วิเคราะห์โค้ด:** `analyze`, `codereview`, `debug`, `precommit`

**🛠️ การพัฒนา:** `refactor`, `testgen`, `secaudit`, `docgen`

**🧰 ยูทิลิตี้:** `challenge`, `tracer`, `listmodels`, `version`

ℹ️ **[ดูบัญชีเครื่องมือเต็ม](tools/)**

## ปัญหาพบบ่อย

### "zen not found" หรือ "command not found"

**วิธี uvx:** ตรวจว่า `uv` อยู่ใน PATH, รัน `which uvx`, ตรวจ PATH ให้มี `/usr/local/bin` และ `~/.local/bin`

**วิธีโคลน:** รัน `./run-server.sh` ตรวจ virtualenv (`which python` ควรชี้ไป `.zen_venv/bin/python`)

### ปัญหา API Key

**"Invalid API key":** ตรวจ `.env` หรือคอนฟิก MCP, ทดสอบคีย์กับ API ของผู้ให้บริการ, ตรวจว่ามีช่องว่างหรือเครื่องหมายเกินไหม

**"Model not available":** รัน `"Use zen to list available models"`, ตรวจตัวแปร environment ที่จำกัดโมเดล, ดูสิทธิ์คีย์

### ปัญหาประสิทธิภาพ

**ตอบช้า:** ใช้โมเดลเร็วขึ้น (เช่น flash), ลดโหมดคิด (`minimal`, `low`), จำกัดโมเดลแพง

**โทเคนล้น:** เลือกโมเดลบริบทยาว, แบ่งคำร้อง, ดู [Large prompt support](advanced-usage.md#working-with-large-prompts)

### แหล่งช่วยเหลือเพิ่มเติม

ℹ️ **[คู่มือแก้ปัญหาเต็ม](troubleshooting.md)** · **[คู่มือใช้งานขั้นสูง](advanced-usage.md)** · **[คู่มือการตั้งค่า](configuration.md)**

## ต่อจากนี้ทำอะไรดี?

- ลองเวิร์กโฟลว์ใน README หลัก
- สำรวจ [Tools Reference](tools/)
- อ่าน [Advanced Usage Guide](advanced-usage.md)
- ปรับ [ตัวเลือกการตั้งค่า](configuration.md)
- ร่วมพูดคุยผ่าน issue หรือ discussion

## เทมเพลตตั้งค่าด่วน

### ชุดพัฒนา (สมดุล)
```env
DEFAULT_MODEL=auto
GEMINI_API_KEY=your-key
OPENAI_API_KEY=your-key
GOOGLE_ALLOWED_MODELS=flash,pro
OPENAI_ALLOWED_MODELS=o4-mini,o3-mini
```

### เน้นประหยัด
```env
DEFAULT_MODEL=flash
GEMINI_API_KEY=your-key
GOOGLE_ALLOWED_MODELS=flash
```

### เน้นประสิทธิภาพสูง
```env
DEFAULT_MODEL=auto
GEMINI_API_KEY=your-key
OPENAI_API_KEY=your-key
GOOGLE_ALLOWED_MODELS=pro
OPENAI_ALLOWED_MODELS=o3
```

### เน้นโมเดลโลคอล
```env
DEFAULT_MODEL=auto
CUSTOM_API_URL=http://localhost:11434/v1
CUSTOM_MODEL_NAME=llama3.2
GEMINI_API_KEY=your-key
```

ขอให้สนุกกับการสร้างทีมพัฒนา AI ของคุณ! 🚀
