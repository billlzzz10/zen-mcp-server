# CodeReview Tool - รีวิวโค้ดเชิงมืออาชีพ

`codereview` ช่วยรีวิวโค้ดอย่างเป็นระบบ มีการจัดลำดับความสำคัญตามระดับความรุนแรง และรองรับหลายรูปแบบการรีวิว ตั้งแต่งานตรวจสไตล์รวดเร็วไปจนถึงการวิเคราะห์ด้านความปลอดภัยเชิงลึก เวิร์กโฟลว์จะบังคับให้ Claude เดินผ่านขั้นตอนอย่างเป็นลำดับ มีการหยุดระหว่างขั้น เพื่อให้ตรวจสอบโค้ดได้รอบด้านก่อนสรุปผลผู้เชี่ยวชาญ

## Thinking Mode

ค่าเริ่มต้น `medium` (8,192 โทเคน) เลือก `high` สำหรับโค้ดวิกฤตด้านความปลอดภัย หรือ `low` เมื่ออยากตรวจสไตล์แบบรวดเร็ว

## โครงเวิร์กโฟลว์

**ระยะสืบสวน (Claude นำ):**
1. ระบุแผนรีวิวและเริ่มตรวจโครงสร้าง
2. ตรวจคุณภาพ, ความปลอดภัย, ประสิทธิภาพ, แพทเทิร์นออกแบบ
3. บันทึก findings, ไฟล์สำคัญ, ประเด็นปัญหา, ระดับความเชื่อมั่น
4. แจ้งจบเมื่อรีวิวครอบคลุม

**ระยะวิเคราะห์โดยผู้เชี่ยวชาญ:**
- สรุปผลพร้อมหลักฐาน
- รายการไฟล์/แพทเทิร์นที่เกี่ยวข้อง
- ปัญหาแบ่งตามความรุนแรง
- คำแนะนำสุดท้าย (ข้ามได้หากความเชื่อมั่น = certain)

> หากอยากให้ Claude รีวิวเองทั้งหมด (ไม่เรียกโมเดลอื่น) ให้ระบุในพรอมต์ว่า “อย่าใช้โมเดลอื่น”

## โมเดลแนะนำ
Gemini Pro หรือ Flash เหมาะมากเพราะ context 1M สามารถมองภาพรวมโปรเจ็กต์ใหญ่ได้ดี

## ตัวอย่างพรอมต์
```
Perform a codereview with gemini pro and review auth.py for security issues…
```
หรือจะรันหลายรีวิวคู่ขนานก็ได้ เช่น ให้ o3 หา issue ร้ายแรง ส่วน flash หา quick win แล้วรวมรายงาน

## จุดเด่น
- จัด issue ตาม severity (CRITICAL/HIGH/MEDIUM/LOW)
- รองรับ review แบบ security, performance, quick review
- ตรวจตามมาตรฐาน coding style
- ตั้ง severity filter ได้
- รองรับภาพประกอบ/สกรีนช็อต
- วิเคราะห์หลายไฟล์/ทั้งไดเรกทอรี
- ให้คำแนะนำที่ปฏิบัติได้จริงพร้อมบรรทัดอ้างอิง
- รองรับภาษาหลากหลายและปัญหาด้านสถาปัตย์

## พารามิเตอร์สำคัญ
- `step`, `step_number`, `total_steps`, `next_step_required`
- `findings`, `files_checked`, `relevant_files`, `relevant_context`
- `issues_found`
- `confidence`
- `backtrack_from_step`, `images`

**การตั้งค่าเริ่มต้น (step 1):**
- `prompt`, `model`, `review_type`, `focus_on`, `standards`, `severity_filter`
- `temperature`, `thinking_mode`, `use_assistant_model`, `continuation_id`

## ประเภทรีวิว
- Full (ค่าเริ่มต้น): ครอบคลุม bug, security, performance, maintainability
- Security: เน้นความปลอดภัย/ช่องโหว่
- Performance: มองหาคอขวด/โอกาส optimize
- Quick: ตรวจเร็วเรื่องสไตล์/ปัญหาพื้นฐาน ใช้โทเคนน้อย

## ระดับความรุนแรง
- CRITICAL – ช่องโหว่, crash, data loss
- HIGH – ลอจิกผิด, ปัญหาประสิทธิภาพ, ความน่าเชื่อถือ
- MEDIUM – code smell, maintainability
- LOW – สไตล์, เอกสาร, ปรับปรุงเล็กน้อย

## ตัวอย่างการใช้งาน
```
“Review the authentication module in auth/ for security vulnerabilities with gemini pro”
“Use o3 to review backend/api.py for performance issues…”
“Quick review utils.py with flash รายงานเฉพาะ critical/high”
```

## วิธีใช้ให้ได้ผล
- ให้บริบท เช่น โค้ดทำอะไร ต้องการตรวจอะไรเป็นพิเศษ
- เลือก review_type ให้ตรงงาน
- ตั้ง severity filter เพื่อลด noise
- แนบไฟล์ที่เกี่ยวข้องทั้งหมด
- ใช้งานรีวิวคู่ขนานสำหรับมุมมองหลายแบบ
- ใช้ continuation เพื่อลงรายละเอียดต่อใน issue สำคัญ

## ผลลัพธ์ที่ได้
- Executive summary
- รายการ issue พร้อม severity/บรรทัด/คำแนะนำ
- Quick wins และแผนปรับปรุงระยะยาว
- หมายเหตุด้านความปลอดภัย (ถ้าเกี่ยวข้อง)

## เปรียบเทียบกับเครื่องมืออื่น
- ใช้ `codereview` เมื่ออยากหา bug/issue จริงจัง
- ใช้ `analyze` เพื่อเข้าใจโครงสร้างโดยไม่เน้นหาปัญหา
- ใช้ `debug` เมื่อมี error ชัดเจนต้องสืบสวน
- ใช้ `refactor` หากอยากได้คำแนะนำโครงสร้าง/modernize
