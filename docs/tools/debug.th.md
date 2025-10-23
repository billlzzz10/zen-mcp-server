# Debug Tool - สืบสวนปัญหาอย่างเป็นระบบ

`debug` นำ Claude ผ่านเวิร์กโฟลว์สืบสวนหลายขั้น เมื่อเก็บหลักฐานครบแล้วจึงเรียกโมเดลผู้เชี่ยวชาญ (หรือให้ Claude ทำเองทั้งหมดได้)

## ตัวอย่างพรอมต์
```
Get gemini to debug why my API returns 400 errors randomly with this stack trace…
Use debug tool to find out why the app is crashing…
```

## เวิร์กโฟลว์
**Investigation Phase:**
1. ระบุอาการและตั้งสมมติฐานเบื้องต้น
2. ตรวจโค้ด/ล็อก ทดสอบสมมติฐาน
3. บันทึกไฟล์ที่ดู, พบอะไร, ความเชื่อมั่น
4. ใช้ backtrack ได้เมื่อมีข้อมูลใหม่
5. แจ้งจบเมื่อเห็น root cause

**Expert Analysis:**
- ส่งสรุปขั้นตอน, หลักฐาน, สมมติฐาน, confidence ให้โมเดลที่เลือก (ยกเว้นเมื่อมั่นใจ `certain`)

> ระบุ “don’t use any other model” ถ้าต้องการให้ Claude จัดการทุกขั้นเอง

## จุดเด่น
- บังคับให้ทำ investigation ทีละขั้นพร้อมบันทึกผล
- ติดตามไฟล์/เมธอดที่ตรวจ
- ปรับสมมติฐานและความมั่นใจระหว่างทาง
- รวมกับ expert analysis ได้เมื่อเสร็จ
- รองรับ stack trace, ล็อก, สกรีนช็อต
- ทำงานต่อเนื่องข้ามเซสชันด้วย continuation
- ใช้โมเดลบริบทยาวเพื่อตรวจล๊อกขนาดใหญ่
- แนะนำให้ค้นเว็บเมื่อจำเป็น

## พารามิเตอร์
- `step`, `step_number`, `total_steps`, `next_step_required`
- `findings`, `files_checked`, `relevant_files`, `relevant_methods`
- `hypothesis`, `confidence`
- `backtrack_from_step`, `images`, `continuation_id`
- `model`, `thinking_mode`, `use_assistant_model`

## ตัวอย่างใช้งาน
```
Debug this TypeError: 'NoneType' object has no attribute 'split'
Use gemini to debug API 500 error พร้อม stack trace
```

## หมวดปัญหาที่ครอบคลุม
- Runtime errors (exception, crash, memory leak)
- Logic bugs (algorithm, state, concurrency)
- Integration issues (API, DB, third-party)
- Performance problems (ช้า, ใช้ CPU/IO สูง)

## แนวทางที่ดี
- รายละเอียดขั้นสืบสวน: บอกว่าทำอะไรและทำไม
- บันทึกไฟล์ที่เปิด แม้ไม่เจอปัญหา (เห็นเส้นทางการสืบสวน)
- ปรับสมมติฐานตามข้อมูลใหม่
- แนบหลักฐาน: ล็อก, สกรีนช็อต, error dialog
- อธิบายอาการและสิ่งที่คาดหวัง vs สิ่งที่เกิดขึ้นจริง
- ระบุ environment, เวอร์ชัน, ขั้นตอนที่ลองไปแล้ว

## ฟีเจอร์ขั้นสูง
- วิเคราะห์ล็อกขนาดใหญ่ด้วยโมเดล 1M context
- ตรวจหลายไฟล์ที่เกี่ยวข้องพร้อมกัน
- แนะนำคำค้นสำหรับ error message/ known issue

## ใช้ debug หรือเครื่องมืออื่น?
- `debug` → มีอาการหรือ error ที่ต้องสืบสวน
- `codereview` → หา issue ทั่วไปโดยไม่มี error เฉพาะ
- `analyze` → อยากเข้าใจโครงสร้างระบบ
- `precommit` → ตรวจความพร้อมก่อน commit
