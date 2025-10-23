# การตั้งค่า Gemini CLI

> **หมายเหตุ:** Zen MCP Server เชื่อมต่อกับ Gemini CLI ได้แล้ว แต่ยังเรียกใช้เครื่องมือ (tools) ไม่สมบูรณ์ ภายหลังจะอัปเดตเอกสารนี้เมื่อทำงานได้เต็มรูปแบบ

ขั้นตอนนี้สรุปวิธีตั้งค่า Zen MCP Server ให้ทำงานร่วมกับ [Gemini CLI](https://github.com/google-gemini/gemini-cli)

## สิ่งที่ต้องมี

- ติดตั้งและตั้งค่า Zen MCP Server เรียบร้อย
- ติดตั้ง Gemini CLI
- มี API key อย่างน้อยหนึ่งตัวในไฟล์ `.env`

## ขั้นตอนการตั้งค่า

1. แก้ไฟล์ `~/.gemini/settings.json` แล้วเพิ่ม:

```json
{
  "mcpServers": {
    "zen": {
      "command": "/path/to/zen-mcp-server/zen-mcp-server"
    }
  }
}
```

2. เปลี่ยน `/path/to/zen-mcp-server` ให้ตรงกับตำแหน่งที่ติดตั้งจริง

3. หากยังไม่มีสคริปต์ `zen-mcp-server` ให้สร้างไฟล์ดังนี้:

```bash
#!/bin/bash
DIR="$(cd "$(dirname "$0")" && pwd)"
cd "$DIR"
exec .zen_venv/bin/python server.py "$@"
```

แล้วตั้งสิทธิ์รันได้ด้วย `chmod +x zen-mcp-server`

4. รีสตาร์ท Gemini CLI

เมื่อเสร็จแล้ว เครื่องมือ Zen ทั้ง 15 ตัวจะปรากฏในเซสชันของ Gemini CLI