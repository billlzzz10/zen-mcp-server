# Zen MCP: หลายเวิร์กโฟลว์ บริบทเดียว

<div align="center">

  [Zen ขณะทำงาน](https://github.com/user-attachments/assets/0d26061e-5f21-4ab1-b7d0-f883ddc2c3da)

🌐 **[ชมตัวอย่างเพิ่มเติม](#-ชมเครื่องมือทำงานจริง)**

### CLI ของคุณ + โมเดลหลากหลาย = ทีมพัฒนา AI ของคุณ

**ใช้ CLI ที่คุณถูกใจ:**  
[Claude Code](https://www.anthropic.com/claude-code) · [Gemini CLI](https://github.com/google-gemini/gemini-cli) · [Codex CLI](https://github.com/openai/codex) · [Qwen Code CLI](https://qwenlm.github.io/qwen-code-docs/) · [Cursor](https://cursor.com) · และอื่น ๆ อีกมากมาย

**ผสานโมเดลหลายตัวในพรอมต์เดียว:**  
Gemini · OpenAI · Anthropic · Grok · Azure · Ollama · OpenRouter · DIAL · โมเดลบนอุปกรณ์

</div>

---

## 🔗 สะพาน CLI-to-CLI มาถึงแล้ว

เครื่องมือ **[`clink`](docs/tools/clink.md)** (CLI + Link) เชื่อม CLI ภายนอกเข้ากับเวิร์กโฟลว์ของคุณโดยตรง:

- **เชื่อมต่อ CLI ภายนอก** เช่น [Gemini CLI](https://github.com/google-gemini/gemini-cli), [Codex CLI](https://github.com/openai/codex), และ [Claude Code](https://www.anthropic.com/claude-code)
- **CLI ซับเอเจนต์** – สร้าง CLI แยกบริบทจากภายใน CLI ปัจจุบันได้เลย! Claude Code เรียก Codex เป็นซับเอเจนต์ได้, Codex เปิด Gemini CLI ได้ ฯลฯ ส่งงานหนัก (รีวิวโค้ด, หา Bug) ไปยังคอนเท็กซ์สดใหม่โดยไม่รบกวนหน้าต่างหลัก แต่ละซับเอเจนต์ส่งคืนเฉพาะผลลัพธ์สุดท้าย
- **แยกบริบท** – ทำการสืบค้นอีกเส้นทางโดยไม่ทำให้ workspace หลักรก
- **บทบาทเฉพาะทาง** – สร้าง `planner`, `codereviewer` หรือบทบาทที่ตั้งเองด้วย system prompt เฉพาะ
- **ความสามารถ CLI ครบชุด** – ค้นเว็บ ตรวจไฟล์ ใช้ MCP tools และค้นคู่มือล่าสุดได้ครบ
- **การสานต่อไร้รอยต่อ** – ซับ CLI ทำงานระดับเดียวกับเอเจนต์หลัก มีบริบทสนทนาร่วมกันเต็มรูปแบบ

```bash
# Codex สร้าง Codex ซับเอเจนต์เพื่อรีวิวโค้ดในคอนเท็กซ์แยก
clink with codex codereviewer to audit auth module for security issues
# ซับเอเจนต์รีวิวแยก ส่งรายงานกลับโดยไม่ทำให้คอนเท็กซ์หลักรก

# ฉันทามติจากหลายโมเดล -> ส่งต่องานพร้อมบริบทสมบูรณ์
Use consensus with gpt-5 and gemini-pro to decide: dark mode or offline support next
Continue with clink gemini - implement the recommended feature
# Gemini ได้รับบริบทรวมดีเบตและเริ่มลงมือได้ทันที
```

ℹ️ **[อ่านรายละเอียด clink เพิ่มเติม](docs/tools/clink.md)**

---

## ทำไมต้อง Zen MCP?

**ทำไมต้องพึ่งโมเดลเดียว ในเมื่อคุณจัดทีมโมเดลได้?**

Zen MCP เป็น Model Context Protocol server ที่ยกระดับเครื่องมืออย่าง [Claude Code](https://www.anthropic.com/claude-code), [Codex CLI](https://developers.openai.com/codex/cli) และ IDE เช่น [Cursor](https://cursor.com) หรือ [ส่วนขยาย Claude Dev VS Code](https://marketplace.visualstudio.com/items?itemName=Anthropic.claude-vscode) **Zen เชื่อมเครื่องมือ AI ที่คุณชอบเข้ากับโมเดลหลายค่ายพร้อมกัน** เพื่อการวิเคราะห์โค้ด แก้ปัญหา และร่วมมือได้เหนือกว่าเดิม

### ความร่วมมือของ AI พร้อมการสานต่อบริบท

Zen รองรับ **การสนทนาแบบมีเธรด** ทำให้ CLI ของคุณ **พูดคุยกับหลายโมเดล แลกเหตุผล ขอความเห็นที่สอง หรือจัดดีเบตระหว่างโมเดล** เพื่อขุดพบคำตอบที่รอบด้าน

คุณยังควบคุมการสนทนา แต่โมเดลที่เหมาะจะช่วยในแต่ละสับภารกิจ บริบทถูกส่งต่อข้ามเครื่องมือและโมเดลอย่างลื่นไหล ทำเวิร์กโฟลว์ซับซ้อน เช่น รีวิวโค้ดหลายโมเดล → วางแผนอัตโนมัติ → ลงมือทำ → ตรวจ pre-commit ได้ภายในการสนทนาเดียว

> **คุณยังเป็นคนคุมเกม** CLI ที่คุณเลือกคือผู้บัญชาการทีม AI แต่คุณออกแบบเวิร์กโฟลว์เอง สร้างพรอมต์ให้ Gemini Pro, GPT-5, Flash หรือโมเดลออฟไลน์เข้ามาประชุมเมื่อถึงเวลา

<details>
<summary><b>เหตุผลที่ควรใช้ Zen MCP</b></summary>

เวิร์กโฟลว์ตัวอย่างเมื่อใช้ Claude Code:

1. **ออเคสตราหลายโมเดล** – Claude ประสาน Gemini Pro, O3, GPT-5 และอีกกว่า 50 โมเดลให้ตรงงาน
2. **เวทีกู้บริบท** – แม้บริบท Claude รีเซ็ต ให้โมเดลอื่นช่วย “ย้ำความจำ” และเดินต่อได้เลย
3. **เวิร์กโฟลว์มีวินัย** – บังคับกระบวนการสืบสวนอย่างเป็นขั้นตอน ป้องกันการสรุปด่วน
4. **ขยายบริบทพ้นขีดจำกัด** – ส่งงานใหญ่ให้ Gemini (1 ล้านโทเคน) หรือ O3 (200K โทเคน)
5. **สานต่อบริบทจริง** – โมเดลแต่ละตัวจำได้ว่าอีกตัวพูดอะไรไว้ก่อนหน้า
6. **ใช้จุดแข็งเฉพาะโมเดล** – วิเคราะห์ลึกกับ Gemini Pro, เร็วทันใจด้วย Flash, เหตุผลเข้มกับ O3, ความเป็นส่วนตัวด้วย Ollama
7. **รีวิวระดับมืออาชีพ** – หลายรอบพร้อมระดับความรุนแรง คำแนะนำ actionable และฉันทามติ
8. **ผู้ช่วยดีบักมืออาชีพ** – สืบ root cause อย่างเป็นระบบ บันทึกสมมติฐานและระดับความมั่นใจ
9. **เลือกโมเดลให้อัตโนมัติ** – Claude เลือกโมเดลที่เหมาะแต่ละสับภารกิจ (หรือคุณระบุเองได้)
10. **รองรับงานภาพ** – วิเคราะห์สกรีนช็อต แผนภาพ รูปภาพผ่านโมเดลที่มีความสามารถด้านภาพ
11. **รองรับโมเดลโลคอล** – รัน Llama, Mistral และอื่น ๆ ในเครื่องเพื่อความเป็นส่วนตัว
12. **เลี่ยงเพดานโทเคน MCP** – จัดการพรอมต์/คำตอบใหญ่โดยไม่ชนขีดจำกัด 25K

**จุดเด่นที่สุด:** เมื่อบริบท Claude หาย เพียงสั่ง “continue with O3” – คำตอบโมเดลอื่นจะจุดประกายให้ Claude จำบทสนทนาได้ทันทีโดยไม่ต้องป้อนไฟล์ใหม่

#### ตัวอย่างเวิร์กโฟลว์รีวิวโค้ดหลายโมเดล

1. `Perform a codereview using gemini pro and o3 and use planner to generate a detailed plan, implement the fixes and do a final precommit check by continuing from the previous codereview`
2. เรียกเวิร์กโฟลว์ [`codereview`](docs/tools/codereview.md) ให้ Claude ตรวจโค้ดหา issue
3. ระหว่างการผ่านแต่ละครั้งจะรวบรวมโค้ดและบันทึกประเด็น
4. รักษาระดับความมั่นใจ (`exploring`, `low`, `medium`, `high`, `certain`)
5. สรุปรายการปัญหาตั้งแต่วิกฤตถึงความเสี่ยงต่ำ
6. ส่งข้อมูลให้ **Gemini Pro** วิเคราะห์ซ้ำ
7. ส่งต่อให้ O3 พร้อมบริบทเพิ่มเติมหากมีการค้นพบใหม่
8. Claude รวมฟีดแบ็กทั้งหมดเป็นรายงานเดียว พร้อมจุดดีในโค้ด และแก้ไขความเข้าใจผิดตามที่โมเดลอื่นช่วยชี้
9. ใช้ [`planner`](docs/tools/planner.md) แยกงานหากต้องรีแฟกเตอร์ใหญ่
10. Claude ลงมือแก้ไขจริง
11. ส่งให้ Gemini Pro ตรวจ [`precommit`](docs/tools/precommit.md) ปิดงาน

ทั้งหมดเกิดในเธรดเดียว! โมเดลแต่ละตัวรับรู้ปฏิสัมพันธ์ก่อนหน้า ทำงานร่วมกันแบบมีความต่อเนื่อง

**นึกถึง Zen MCP ว่าเป็น "กาวซูเปอร์" ของโลก AI**

> Claude ยังคุมเกมอยู่ – แต่ **คุณ** เป็นผู้ออกคำสั่ง
> Zen ถูกออกแบบให้ Claude เรียกโมเดลอื่นเฉพาะเมื่อจำเป็น พร้อมสนทนาตอบโต้มีความหมาย
> **คุณ** คือคนร่างพรอมต์ทรงพลังที่จะดึง Gemini, Flash, O3 หรือทำงานคนเดียว
> คุณคือผู้กำกับการสนทนา และคือ “ความฉลาดที่แท้จริง” ในที่นี้

</details>

#### สแต็ก AI ที่แนะนำ

<details>
<summary>สำหรับผู้ใช้ Claude Code</summary>

- **Sonnet 4.5** – งานเอเจนต์และออเคสตราทั้งหมด
- **Gemini 2.5 Pro** หรือ **GPT-5-Pro** – วิเคราะห์ลึก รีวิวเพิ่ม ดีบัก และตรวจ pre-commit

</details>

<details>
<summary>สำหรับผู้ใช้ Codex</summary>

- **GPT-5 Codex Medium** – งานเอเจนต์และออเคสตราทั้งหมด
- **Gemini 2.5 Pro** หรือ **GPT-5-Pro** – วิเคราะห์ลึก รีวิวเพิ่ม ดีบัก และตรวจ pre-commit

</details>

## เริ่มต้นอย่างรวดเร็ว (5 นาที)

**ต้องมี:** Python 3.10+, Git, [ติดตั้ง uv](https://docs.astral.sh/uv/getting-started/installation/)

**1. ขอ API Key** (เลือกอย่างน้อยหนึ่ง)
- [OpenRouter](https://openrouter.ai/) – API เดียวเข้าถึงหลายโมเดล
- [Gemini](https://makersuite.google.com/app/apikey) – โมเดลล่าสุด
- [OpenAI](https://platform.openai.com/api-keys) – O3, GPT-5
- [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/) – สำหรับองค์กร
- [X.AI](https://console.x.ai/) – โมเดล Grok
- [DIAL](https://dialx.ai/) – เข้าถึงโมเดลแบบไม่ผูกผู้ขาย
- [Ollama](https://ollama.ai/) – โมเดลโลคอลฟรี

**2. ติดตั้ง** (เลือกหนึ่ง)

### ทางเลือก A: โคลนแล้วตั้งค่าอัตโนมัติ (แนะนำ)
```bash
git clone https://github.com/BeehiveInnovations/zen-mcp-server.git
cd zen-mcp-server
./run-server.sh
```

### ทางเลือก B: ตั้งค่าด้วย [uvx](https://docs.astral.sh/uv/getting-started/installation/)
```json
{
  "mcpServers": {
    "zen": {
      "command": "bash",
      "args": ["-c", "for p in $(which uvx 2>/dev/null) $HOME/.local/bin/uvx /opt/homebrew/bin/uvx /usr/local/bin/uvx uvx; do [ -x \"$p\" ] && exec \"$p\" --from git+https://github.com/BeehiveInnovations/zen-mcp-server.git zen-mcp-server; done; echo 'uvx not found' >&2; exit 1"],
      "env": {
        "PATH": "/usr/local/bin:/usr/bin:/bin:/opt/homebrew/bin:~/.local/bin",
        "GEMINI_API_KEY": "your-key-here",
        "DISABLED_TOOLS": "analyze,refactor,testgen,secaudit,docgen,tracer",
        "DEFAULT_MODEL": "auto"
      }
    }
  }
}
```

**3. เริ่มใช้งานทันที**
```
"Use zen to analyze this code for security issues with gemini pro"
"Debug this error with o3 and then get flash to suggest optimizations"
"Plan the migration strategy with zen, get consensus from multiple models"
"clink with cli_name=\"gemini\" role=\"planner\" to draft a phased rollout plan"
```

ℹ️ **[คู่มือติดตั้งเต็ม](docs/getting-started.md)** · **[ตั้งค่า Cursor & VS Code](docs/getting-started.md#ide-clients)** · **[ชมเครื่องมือทำงานจริง](#-ชมเครื่องมือทำงานจริง)**

## การตั้งค่าผู้ให้บริการ

Zen เปิดใช้งาน provider ใด ๆ ที่มี credential ใน `.env` ดู `.env.example` เพื่อปรับแต่งเชิงลึก

## เครื่องมือหลัก

> **หมายเหตุ:** เครื่องมือแต่ละตัวประกอบด้วยเวิร์กโฟลว์และพารามิเตอร์หลายขั้น ซึ่งใช้พื้นที่บริบทแม้ไม่ใช้งาน เพื่อประสิทธิภาพจึงปิดบางตัวโดยค่าเริ่มต้น ดู [การตั้งค่าเครื่องมือ](#การตั้งค่าเครื่องมือ) เพื่อเปิดเพิ่มเติม

**การร่วมมือ & วางแผน** *(เปิดค่าเริ่มต้น)*
- **[`clink`](docs/tools/clink.md)** – สะพานไป CLI ภายนอก
- **[`chat`](docs/tools/chat.md)** – ระดมไอเดีย ขอความเห็น ตรวจแนวทาง
- **[`thinkdeep`](docs/tools/thinkdeep.md)** – วิเคราะห์เชิงลึกและมุมมองทางเลือก
- **[`planner`](docs/tools/planner.md)** – แยกงานใหญ่เป็นแผนปฏิบัติได้จริง
- **[`consensus`](docs/tools/consensus.md)** – ได้ข้อสรุปจากหลายโมเดลพร้อมกำหนดจุดยืน

**การวิเคราะห์โค้ด & คุณภาพ**
- **[`debug`](docs/tools/debug.md)** – สืบ root cause อย่างเป็นระบบ
- **[`precommit`](docs/tools/precommit.md)** – ตรวจความพร้อมก่อนคอมมิท ป้องกัน regression
- **[`codereview`](docs/tools/codereview.md)** – รีวิวมืออาชีพพร้อมน้ำหนักปัญหาและคำแนะนำ
- **[`analyze`](docs/tools/analyze.md)** *(ปิดเริ่มต้น)* – เข้าใจสถาปัตยกรรมและการเชื่อมโยงของโค้ดเบสใหญ่

**เครื่องมือพัฒนา** *(ปิดเริ่มต้น)*
- **[`refactor`](docs/tools/refactor.md)** – รีแฟกเตอร์ด้วยการย่อยงานอย่างชาญฉลาด
- **[`testgen`](docs/tools/testgen.md)** – สร้างเทสครอบคลุมพร้อมเคสปลายสุด
- **[`secaudit`](docs/tools/secaudit.md)** – ตรวจความปลอดภัย (OWASP Top 10)
- **[`docgen`](docs/tools/docgen.md)** – สร้างเอกสารพร้อมวิเคราะห์ความซับซ้อน

**ยูทิลิตี้**
- **[`apilookup`](docs/tools/apilookup.md)** – บังคับค้นคู่มือ API/SDK ปีปัจจุบัน
- **[`challenge`](docs/tools/challenge.md)** – กระตุ้นการคิดเชิงวิพากษ์
- **[`tracer`](docs/tools/tracer.md)** *(ปิดเริ่มต้น)* – วิเคราะห์ call-flow

<details>
<summary><b id="การตั้งค่าเครื่องมือ">⚙️ การตั้งค่าเครื่องมือ</b></summary>

### ค่าเริ่มต้น

**เปิดใช้งาน**: `chat`, `thinkdeep`, `planner`, `consensus`, `codereview`, `precommit`, `debug`, `apilookup`, `challenge`

**ปิดไว้**: `analyze`, `refactor`, `testgen`, `secaudit`, `docgen`, `tracer`

### เปิดเครื่องมือเพิ่ม

ลบชื่อออกจาก `DISABLED_TOOLS`

**ตัวอย่างแก้ `.env`:**
```bash
DISABLED_TOOLS=analyze,refactor,testgen,secaudit,docgen,tracer
DISABLED_TOOLS=refactor,testgen,secaudit,docgen,tracer  # เปิด analyze
DISABLED_TOOLS=                                         # เปิดทั้งหมด
```

**ตั้งค่าผ่าน MCP config:**
```json
{
  "mcpServers": {
    "zen": {
      "env": {
        "DISABLED_TOOLS": "refactor,testgen,secaudit,docgen,tracer",
        "DEFAULT_MODEL": "pro",
        "DEFAULT_THINKING_MODE_THINKDEEP": "high",
        "GEMINI_API_KEY": "your-gemini-key",
        "OPENAI_API_KEY": "your-openai-key",
        "OPENROUTER_API_KEY": "your-openrouter-key",
        "LOG_LEVEL": "INFO",
        "CONVERSATION_TIMEOUT_HOURS": "6",
        "MAX_CONVERSATION_TURNS": "50"
      }
    }
  }
}
```

**เปิดทั้งหมด:**
```json
{
  "mcpServers": {
    "zen": {
      "env": {
        "DISABLED_TOOLS": ""
      }
    }
  }
}
```

- เครื่องมือจำเป็น (`version`, `listmodels`) ปิดไม่ได้
- เปลี่ยนค่าต้องรีสตาร์ทเซสชัน Claude
- เครื่องมือแต่ละตัวใช้บริบทเพิ่ม เลือกเปิดเฉพาะที่จำเป็น

</details>

## 📺 ชมเครื่องมือทำงานจริง

<details>
<summary><b>Chat Tool</b> – ตัดสินใจร่วมกัน สนทนาหลายรอบ</summary>

**ตัวอย่างเลือก Redis หรือ Memcached**

[Chat Redis or Memcached_web.webm](https://github.com/user-attachments/assets/41076cfe-dd49-4dfc-82f5-d7461b34705d)

**สนทนาแบบหลายรอบพร้อมสานต่อ**

[Chat With Gemini_web.webm](https://github.com/user-attachments/assets/37bd57ca-e8a6-42f7-b5fb-11de271e95db)

</details>

<details>
<summary><b>Consensus Tool</b> – ดีเบตหลายโมเดลและหาข้อสรุป</summary>

[Zen Consensus Debate](https://github.com/user-attachments/assets/76a23dd5-887a-4382-9cf0-642f5cf6219e)

</details>

<details>
<summary><b>PreCommit Tool</b> – ตรวจการเปลี่ยนแปลงแบบครบวงจร</summary>

<div align="center">
  <img src="https://github.com/user-attachments/assets/584adfa6-d252-49b4-b5b0-0cd6e97fb2c6" width="950">
</div>

</details>

<details>
<summary><b>API Lookup Tool</b> – API ปัจจุบัน vs ล้าสมัย</summary>

[API without Zen](https://github.com/user-attachments/assets/01a79dc9-ad16-4264-9ce1-76a56c3580ee)

[API with Zen](https://github.com/user-attachments/assets/5c847326-4b66-41f7-8f30-f380453dce22)

</details>

<details>
<summary><b>Challenge Tool</b> – คิดเชิงวิพากษ์ vs ตอบรับอัตโนมัติ</summary>

![without_zen@2x](https://github.com/user-attachments/assets/64f3c9fb-7ca9-4876-b687-25e847edfd87)

![with_zen@2x](https://github.com/user-attachments/assets/9d72f444-ba53-4ab1-83e5-250062c6ee70)

</details>

## คุณสมบัติเด่น

**การออเคสตรา AI**
- เลือกโมเดลอัตโนมัติให้เหมาะกับงาน
- เชื่อมโมเดลหลายตัวในบทสนทนาเดียว
- สานต่อบริบทข้ามเครื่องมือและโมเดล
- [Context revival](docs/context-revival.md) เพื่อเดินต่อแม้บริบทรีเซ็ต

**การรองรับโมเดล**
- ผู้ให้บริการหลากหลาย: Gemini, OpenAI, Azure, X.AI, OpenRouter, DIAL, Ollama
- โมเดลล่าสุด: GPT-5, Gemini 2.5 Pro, O3, Grok-4, Llama ในเครื่อง
- [Thinking modes](docs/advanced-usage.md#thinking-modes) ปรับความลึก vs ค่าใช้จ่าย
- รองรับการวิเคราะห์ภาพ

**ประสบการณ์นักพัฒนา**
- เวิร์กโฟลว์มีกรอบ ช่วยเลี่ยงการรีบสรุป
- จัดการไฟล์อัตโนมัติ คุมโทเคน
- ค้นเว็บเพื่ออัปเดตความรู้
- [Large prompt support](docs/advanced-usage.md#working-with-large-prompts) เลี่ยงเพดาน 25K

## ตัวอย่างเวิร์กโฟลว์

**โค้ดรีวิวหลายโมเดล**
```
"Perform a codereview using gemini pro and o3, then use planner to create a fix strategy"
```

**ดีบักร่วมกัน**
```
"Debug this race condition with max thinking mode, then validate the fix with precommit"
```

**วางแผนสถาปัตยกรรม**
```
"Plan our microservices migration, get consensus from pro and o3 on the approach"
```

ℹ️ **[Advanced Usage Guide](docs/advanced-usage.md)** – เวิร์กโฟลว์ซับซ้อน การตั้งค่าโมเดล ฟีเจอร์ power user

## ลิงก์ด่วน

**📘 เอกสาร**
- [ภาพรวมเอกสาร](docs/index.md)
- [เริ่มต้นใช้งาน](docs/getting-started.md)
- [บัญชีเครื่องมือ](docs/tools/)
- [การใช้งานขั้นสูง](docs/advanced-usage.md)
- [การตั้งค่า](docs/configuration.md)
- [เพิ่มผู้ให้บริการ](docs/adding_providers.md)
- [จัดอันดับโมเดล](docs/model_ranking.md)

**⚙️ การติดตั้ง & สนับสนุน**
- [ตั้งค่า WSL](docs/wsl-setup.md)
- [แก้ปัญหา](docs/troubleshooting.md)
- [ร่วมพัฒนา](docs/contributions.md)

## ไลเซนส์

ใช้สัญญา Apache 2.0 – ดู [LICENSE](LICENSE)

## คำขอบคุณ

สร้างด้วยพลังความร่วมมือของหลายโมเดล AI 🤝
- [MCP (Model Context Protocol)](https://modelcontextprotocol.com)
- [Codex CLI](https://developers.openai.com/codex/cli)
- [Claude Code](https://claude.ai/code)
- [Gemini](https://ai.google.dev/)
- [OpenAI](https://openai.com/)
- [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/)

### ประวัติการกดดาว

[![Star History Chart](https://api.star-history.com/svg?repos=BeehiveInnovations/zen-mcp-server&type=Date)](https://www.star-history.com/#BeehiveInnovations/zen-mcp-server&Date)
