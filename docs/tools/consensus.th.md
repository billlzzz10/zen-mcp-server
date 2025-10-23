# Consensus Tool - รวมมุมมองจากหลายโมเดล

`consensus` ใช้หลายโมเดลมาวิเคราะห์ข้อเสนอของคุณด้วยบทบาทต่างกัน (สนับสนุน/คัดค้าน/เป็นกลาง) จากนั้น Claude สังเคราะห์เป็นข้อสรุปสมดุล

## Thinking Mode
ค่าเริ่มต้น `medium` ปรับเป็น `high` หรือ `max` เมื่อเป็นการตัดสินใจใหญ่ที่ต้องการเหตุผลลึก

## ทำงานอย่างไร
1. กำหนดจุดยืนแต่ละโมเดล (`for`, `against`, `neutral` หรือคำพ้อง)
2. แต่ละโมเดลให้ความเห็นตามบทบาทพร้อม guardrail ป้องกันแนวคิดไม่เหมาะสม
3. Claude รวมผลและสรุปคำแนะนำ
4. รองรับ natural language เช่น “supportive”, “critical” ได้อัตโนมัติ

## ตัวอย่างพรอมต์
```
Use consensus with flash supportive และ pro critical เพื่อประเมิน migration REST → GraphQL
Get consensus จาก o3, flash, pro: o3 เน้น security, flash เน้นความเร็ว, pro เป็นกลาง
```

## จุดเด่น
- กำหนดบทบาท/คำสั่งเฉพาะแต่ละโมเดลได้ (`stance_prompt`)
- ป้องกันแนวคิดผิดจริยธรรม (ไม่สนับสนุนสิ่งอันตราย)
- หากกำหนด stance ผิดจะ fallback เป็น neutral พร้อมเตือน
- รองรับไฟล์/ภาพประกอบ
- ต่อเนื่องได้ด้วย continuation
- แนะนำให้ค้นเว็บเมื่อจำเป็น

## พารามิเตอร์
- `prompt`
- `models`: รายการ `{model, stance, stance_prompt}`
- `files`, `images`, `focus_areas`
- `temperature`, `thinking_mode`, `continuation_id`

## ตัวอย่าง config
```json
[
  {"model": "flash", "stance": "for"},
  {"model": "pro", "stance": "against"}
]
```

## คำแนะนำการใช้งาน
- ใส่บริบทและข้อจำกัดของโปรเจ็กต์
- สร้างสมดุลระหว่างฝ่ายที่สนับสนุน/คัดค้าน
- ระบุหัวข้อที่อยากให้เน้น (security, UX, performance)
- แนบไฟล์หรือ mockup เพื่อประกอบการตัดสินใจ
- ใช้ continuation เพื่อฟันธงรอบถัดไปหรือเจาะลึกมุมที่น่าสนใจ

## เปรียบเทียบเครื่องมืออื่น
- `consensus` → ต้องการมุมมองหลายด้านเพื่อชั่งน้ำหนัก
- `chat` → สนทนา/brainstorm จุดเดียว
- `thinkdeep` → ให้โมเดลขยายเหตุผลของแนวคิดเดียว
- `analyze` → ทำความเข้าใจระบบโดยไม่มีการถกเถียง
