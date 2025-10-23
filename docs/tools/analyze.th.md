# Analyze Tool - วิเคราะห์โค้ดอย่างเป็นระบบ

**เครื่องมือสำหรับทำความเข้าใจโค้ดและสถาปัตยกรรมผ่านเวิร์กโฟลว์ที่มีขั้นตอนชัดเจน**

`analyze` ช่วยสำรวจโค้ดเบส เข้าใจโครงสร้าง สังเกตแพทเทิร์น และจับการตัดสินใจด้านสถาปัตยกรรม มันพา Claude เดินตามเวิร์กโฟลว์หลายขั้นที่จัดระเบียบไว้ เพื่อรวบรวมข้อมูลให้ครบก่อนสรุปผลเชิงผู้เชี่ยวชาญ

## Thinking Mode

ค่าเริ่มต้นคือ `medium` (8,192 โทเคน) เลือก `high` เมื่อต้องการวิเคราะห์สถาปัตยกรรมอย่างลึก (แลกมาด้วยโทเคน) หรือ `low` เมื่อต้องการภาพรวมเร็ว ๆ ประหยัดโทเคน ~6k

## ลำดับเวิร์กโฟลว์

เวิร์กโฟลว์ของ analyze แบ่งเป็นสองระยะ:

**Investigation (Claude นำ):**
1. **ขั้นแรก** – วางแผนวิเคราะห์และเริ่มตรวจโครงสร้าง
2. **ขั้นถัด ๆ** – สำรวจสถาปัตย์ แพทเทิร์น การพึ่งพา การตัดสินใจออกแบบ
3. **ตลอดเวิร์กโฟลว์** – เก็บ findings, ไฟล์ที่เกี่ยวข้อง, บันทึกความมั่นใจ
4. **จบบทวิเคราะห์** – เมื่อ Claude เห็นว่าครบถ้วน จะส่งสัญญาณปิดระยะนี้

**Expert Analysis:**
- รวบรวมสรุปทุกประเด็น
- ให้ insight ด้านสถาปัตยกรรม/แพทเทิร์น
- แนะนำแนวทางปรับปรุง
- ตัดสินผลเชิงผู้เชี่ยวชาญ (ข้ามได้ถ้าความมั่นใจ = certain)

## ตัวอย่างคำสั่ง
```
"ใช้ gemini วิเคราะห์ main.py เพื่อเข้าใจการทำงาน"
"ให้ gemini ดูโครงสร้างโฟลเดอร์ src/ ในเชิงสถาปัตย์"
```

## จุดเด่น

- วิเคราะห์ได้ทั้งไฟล์เดี่ยวและโฟลเดอร์ใหญ่ (คัดไฟล์อย่างชาญฉลาด)
- รองรับประเภทการวิเคราะห์หลายแบบ: architecture, performance, security, quality, general
- ดึงเนื้อหาไฟล์จริง แต่จะรายงานเป็น path เพื่อไม่รกหน้าจอ
- ชี้แพทเทิร์น, anti-pattern, โอกาสรีแฟกเตอร์
- ใช้โมเดลบริบทยาวได้ (Gemini Pro 1M token)
- ทำแผนภาพความสัมพันธ์ข้ามไฟล์ อธิบายสถาปัตยกรรมด้วยคำ
- วิเคราะห์ภาพ (diagram, UML, flowchart) ได้
- แนะนำให้ค้นเว็บเมื่อควรอ้างอิงข้อมูลล่าสุด

## พารามิเตอร์หลัก

**ค่าในระหว่างสืบสวน:**
- `step`, `step_number`, `total_steps`
- `next_step_required`
- `findings`
- `files_checked`, `relevant_files`
- `relevant_context`
- `issues_found`
- `confidence` (exploring/low/medium/high/certain)
- `backtrack_from_step`
- `images`

**ค่าเริ่มต้น (step 1):**
- `prompt` (จำเป็น)
- `model` (auto|pro|flash|flash-2.0|flashlite|o3|o3-mini|o4-mini|gpt4.1|gpt5|gpt5-mini|gpt5-nano)
- `analysis_type` (architecture|performance|security|quality|general)
- `output_format` (summary|detailed|actionable)
- `temperature`
- `thinking_mode` (เฉพาะ Gemini)
- `use_assistant_model` (เปิด/ปิด expert analysis)
- `continuation_id`

## ประเภทการวิเคราะห์ (สรุป)

- **General** – โครงสร้าง, การไหลข้อมูล, การออกแบบโดยรวม
- **Architecture** – ความสัมพันธ์โมดูล, layered design, maintainability
- **Performance** – คอขวด, complexity, I/O, DB efficiency
- **Security** – แนวทางรักษาความปลอดภัย, validation, authz/authn
- **Quality** – maintainability, coverage, documentation, best practices

## รูปแบบผลลัพธ์

- **Summary** – สรุปประเด็นสำคัญ
- **Detailed** – เจาะลึกพร้อมโค้ดอ้างอิง
- **Actionable** – รายการแนะนำพร้อมลำดับความสำคัญ

## ข้อแนะนำการใช้งาน

- ระบุสิ่งที่อยากรู้ให้ชัด
- เลือก analysis_type ให้ตรงเป้าหมาย
- วิเคราะห์ไฟล์/โมดูลที่เกี่ยวข้องร่วมกัน
- ใช้โมเดลบริบทยาวเมื่อโค้ดใหญ่
- แนบ diagram หรือเอกสารประกอบถ้ามี
- ใช้ continuation เพื่อต่อยอดการวิเคราะห์ครั้งก่อน

## ฟีเจอร์ขั้นสูง

- **Large Codebase** – โมเดลบริบทยาวรองรับ monorepo
- **Cross-File Mapping** – อธิบายการพึ่งพาหลายโมดูล
- **Pattern Recognition** – บอก design pattern/anti-pattern
- **Web Search** – แนะนำหัวข้อให้ Claude ไปค้นเพิ่มเติม

## เลือก `analyze` หรือเครื่องมืออื่น?

- `analyze` – ทำความเข้าใจโครงสร้าง/สถาปัตยกรรม
- `codereview` – หา bug/issue พร้อมแนวแก้
- `debug` – เจาะจงปัญหา runtime/perf
- `refactor` – แนะนำการปรับโค้ดเฉพาะจุด
- `chat` – คุยทั่วไปแบบไม่ต้องการเวิร์กโฟลว์เต็มรูปแบบ
