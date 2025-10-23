# Clink Tool - สะพาน CLI-to-CLI

**เปิดซับเอเจนต์จาก CLI เดียว เชื่อมต่อ CLI ภายนอก และคุมบริบทแยกโดยไม่ต้องออกจากเซสชัน**

`clink` เปลี่ยน CLI หลักของคุณให้เป็นตัว orchestrate หลายเอเจนต์ เปิด Codex ซ้อนใน Codex, เรียก Gemini ที่มี context 1M, หรือเรียก Claude ที่ตั้งบทบาทไว้ พร้อมรักษาบริบทสนทนา

> **คำเตือน**: การตั้งค่ามาตรฐานเปิด CLI ลูกด้วย flag ที่ผ่อนคลายความปลอดภัย (เช่น `--yolo`, `--dangerously-bypass-approvals-and-sandbox`, `--permission-mode acceptEdits`) เพื่อให้แก้ไฟล์/รันเครื่องมือได้อัตโนมัติ หากไม่ต้องการ ให้ลบ flag เหล่านี้หรือปรับบทบาท prompt ตามความเสี่ยง

## ทำไมต้องใช้ Clink

### ปัญหา
กำลังดีบักใน Codex แต่ต้องการรีวิวความปลอดภัยขนาดใหญ่ ซึ่งจะกิน context มหาศาล

### วิธีแก้
```
clink with codex codereviewer to audit auth/
```
- ซับเอเจนต์ได้ context สด
- วิเคราะห์ลึกด้วยเครื่องมือของตัวเอง
- ส่งคืนเพียงรายงานสุดท้าย
- เซสชันหลักยังคงโฟกัสปัญหาเดิม

### Cross-CLI Orchestration
- Codex เรียก Gemini เพื่อวิเคราะห์โค้ด 500 ไฟล์ โดยไม่ต้องสลับเทอร์มินัล
- ใช้ `consensus` ถกประเด็น → ใช้ `clink` ส่งต่อให้ Gemini ลงมือ implement โดยมีบริบทครบ

## คุณสมบัติหลัก
- อยู่ใน CLI เดียวไม่ต้องสลับหน้าต่าง
- บริบทสนทนาต่อเนื่องข้าม CLI
- บทบาทสำเร็จรูป เช่น planner, codereviewer
- ใช้จุดเด่นของแต่ละ CLI ได้ (web search, context ขนาดใหญ่)
- ประหยัดโทเคน (ส่งเฉพาะ path ให้ CLI ลูกไปเปิดเอง)
- ทำงานร่วมกับ `planner`, `codereview`, `debug` ได้
- Gemini มีฟรี tier (1,000 requests/วัน)

## บทบาทที่มี
- `default` – ถามตอบทั่วไป
- `planner` – วางแผนหลายเฟส
- `codereviewer` – ให้รายงานโค้ดพร้อม severity

เพิ่มบทบาทใหม่ได้ที่ `conf/cli_clients/`

## พารามิเตอร์
- `prompt`
- `cli_name` (gemini|claude|codex)
- `role` (default|planner|codereviewer)
- `files`, `images`
- `continuation_id`

## ตัวอย่าง
```
clink with gemini planner ออกแบบ rollout 3 เฟสสำหรับ feature flags
clink to gemini codereviewer: ตรวจ payment_service.py หา race condition
clink codex: ทำ code review เต็มรูปแบบ
ถาม gemini breaking changes TypeScript 5.5
planner → clink → codereview เป็นเวิร์กโฟลว์เชื่อมต่อกัน
```

## วิธีทำงาน
1. คุณสั่ง `clink`
2. Zen เปิด CLI ที่กำหนด
3. ส่งพรอมต์ + บริบทไปให้
4. CLI ลูกใช้เครื่องมือของตัวเองตามปกติ
5. ผลลัพธ์ถูกส่งกลับในสนทนาเดิม
6. ใช้ `continuation_id` เพื่ออ้างอิงผลในเครื่องมืออื่นต่อ

## แนวทางปฏิบัติ
- ติดตั้งและเข้าสู่ระบบ CLI ภายนอกก่อนใช้
- เลือกบทบาทให้เหมาะกับงาน
- ใช้จุดเด่นแต่ละ CLI (Gemini context ใหญ่, Claude planner ฯลฯ)
- ส่ง path ให้ CLI เปิดเอง ลดโทเคน

## การตั้งค่า
คอนฟิกอยู่ใน `conf/cli_clients/` (เช่น `gemini.json`, `claude.json`, `codex.json`) ซึ่งชี้ไปยัง prompt ใน `systemprompts/clink/`

## เมื่อไหร่ควรใช้
- `clink` – ต้องเรียกความสามารถของ CLI ภายนอก
- `chat` – สนทนาในบริบทเดียวกันโดยไม่เรียก CLI ภายนอก
- `planner` – เวิร์กโฟลว์วางแผนใน Zen
- `codereview` – รีวิวโค้ด native ของ Zen

อย่าลืมติดตั้ง CLI ที่อยากใช้ เช่น Claude Code, Gemini CLI, Codex CLI
