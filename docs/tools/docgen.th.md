# DocGen Tool - สร้างเอกสารอย่างครอบคลุม

`docgen` วิเคราะห์โครงสร้างโค้ด ประเมินความซับซ้อน และบันทึกข้อควรระวัง เพื่อสร้างเอกสารครบถ้วนตามมาตรฐานโปรเจ็กต์ ผ่านเวิร์กโฟลว์ที่บังคับตรวจทุกไฟล์

## เวิร์กโฟลว์
**ขั้นค้นหา (Step 1):** Claude ต้องรวบรวมรายชื่อไฟล์ทั้งหมดที่ต้องเขียนเอกสาร พร้อมนับจำนวน

**ขั้นจัดทำเอกสาร (Step 2+):**
- จัดทำเอกสารทีละไฟล์ พร้อมอัปเดตตัวนับ `num_files_documented`
- ทำจน `num_files_documented = total_files_to_document`

**ขั้นสรุป:**
- รวมกลยุทธ์เอกสารให้สอดคล้องกัน
- ใส่ docstring+complexity, call flow, gotchas, behavior แปลก, standard ของโปรเจ็กต์

## โมเดลแนะนำ
Gemini Pro หรือ O3 ที่เข้าใจความสัมพันธ์โค้ดเชิงลึกและมี context ใหญ่ เหมาะกับการสร้างเอกสารพร้อมวิเคราะห์ประเด็นซ่อนเร้น

## ตัวอย่างพรอมต์
```
“Use zen to generate documentation for the UserManager class”
“Document the authentication module with complexity analysis using gemini pro”
```

## จุดเด่น
- ตรวจครบทุกไฟล์ด้วยตัวนับ
- รองรับสไตล์เอกสารสมัยใหม่ต่อภาษา (Python docstring, JSDoc, Javadoc, Swift/Objective-C /// ฯลฯ)
- วิเคราะห์ความซับซ้อน (Big O)
- บันทึก call flow และ dependency
- ตรวจใหญ่ ๆ ทีละส่วนสำหรับไฟล์ยาว
- มีขั้นตรวจสอบสุดท้ายว่าทำครบทุกเมธอด
- สามารถ update เอกสารเดิมที่ผิด/ไม่ครบ
- เพิ่มคอมเมนต์ inline ให้ logic ซับซ้อน

## พารามิเตอร์สำคัญ
- `step`, `step_number`, `total_steps`, `next_step_required`
- `findings`, `relevant_files`
- `num_files_documented`, `total_files_to_document`
- ตัวเลือกหลัก: `document_complexity`, `document_flow`, `update_existing`, `comments_on_complex_logic`

## สิ่งที่เอกสารครอบคลุม
- คำอธิบายฟังก์ชัน/เมธอด + ประเภทพารามิเตอร์
- ค่าที่คืน + exception ที่โยน
- Big O (เวลา/พื้นที่)
- ใครเรียก/ถูกเรียกโดยใคร (call flow)
- gotchas, edge case, dependency เฉพาะ, สถานะที่ต้องระวัง
- คอมเมนต์ inline สำหรับ logic/อัลกอริทึมซับซ้อน
- ตัวอย่าง usage หรือ best practice (ถ้าควรมี)

## ภาษา/สไตล์ที่รองรับ
- Python docstring
- Objective-C/Swift `///`
- JavaScript/TypeScript `/** */`
- Java `/** */`
- C#, C/C++, Go, Rust (ตามคอนเวนชัน)

## ใช้อย่างมีประสิทธิภาพ
- ปล่อยให้เครื่องมือควบคุมตัวนับจนจบทุกไฟล์
- ใช้ค่า default เพื่อให้มี complexity + call flow + ปรับเอกสารเดิม
- หากไฟล์ใหญ่ ระบบจะแบ่งย่อยอัตโนมัติ
- สำรวจผลลัพธ์หลังจบ เพราะเครื่องมืออาจพบ bug/issue ระหว่างทาง

## เทียบกับเครื่องมืออื่น
- `docgen` → ใช้เขียนเอกสาร
- `analyze` → เข้าใจโครงสร้างโดยไม่เขียนเอกสาร
- `codereview` → ตรวจความครบถ้วนของเอกสารเป็นส่วนหนึ่งของรีวิว
- `refactor` → ปรับโครงสร้างก่อนเอกสาร เพื่อให้การอธิบายง่ายขึ้น
