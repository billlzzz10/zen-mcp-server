# คู่มือการทดสอบ

โปรเจ็กต์นี้มีทั้งชุด Unit test และ Simulator test เพื่อครอบคลุมการทำงานแบบ end-to-end

## การรันทดสอบ

### ข้อกำหนดพื้นฐาน
- ตั้งค่าสภาพแวดล้อมด้วย `./run-server.sh`
  - ใช้ `./run-server.sh -f` หากต้องการให้ tail ล็อกอัตโนมัติหลังสตาร์ท

### Unit Test

รัน pytest เพื่อทดสอบทั้งหมดหรือเฉพาะไฟล์
```bash
python -m pytest -xvs                     # ทั้งชุดพร้อม verbose
python -m pytest tests/test_providers.py -xvs  # เจาะจงไฟล์
```

### Simulator Test

Simulator test จำลองการใช้งานจริงของ Claude CLI กับ MCP server:
- ใช้โปรโตคอล MCP บน stdio จริง
- มีบทสนทนาหลายรอบ ครอบคลุมหลายเครื่องมือ
- ตรวจสอบล็อก และการประมวลผลไฟล์

> **สำคัญ:** ตั้ง `LOG_LEVEL=DEBUG` ใน `.env` เพื่อให้เทสต์ตรวจสอบรายละเอียดจากล็อกได้ครบ

#### การติดตามล็อกขณะรันทดสอบ

โปรโตคอล MCP stdio จะกัก `stderr` ระหว่างเรียกเครื่องมือ (เพื่อไม่ให้รบกวน JSON-RPC) ล็อกจึงถูกเขียนลงไฟล์ในโฟลเดอร์ `logs/`

```bash
./run-server.sh -f                       # สตาร์ทและ tail ล็อกหลัก
# หรือ
tail -f -n 500 logs/mcp_server.log       # ล็อกหลัก มีรายละเอียดการรันเครื่องมือ
tail -f logs/mcp_activity.log            # บันทึก TOOL_CALL / TOOL_COMPLETED
ls -lh logs/mcp_*.log*                   # ตรวจขนาดไฟล์ (หมุนที่ 20MB)
```

ระบบจะหมุนล็อกอัตโนมัติที่ 20MB เก็บสำเนา 10 ไฟล์สำหรับ `mcp_server.log` และ 5 ไฟล์สำหรับ `mcp_activity.log`

#### รัน Simulator Test ทั้งหมด
```bash
python communication_simulator_test.py
python communication_simulator_test.py --verbose
python communication_simulator_test.py --keep-logs  # เก็บล็อกไว้ดูภายหลัง
```

#### รันทีละเคส
```bash
python communication_simulator_test.py --individual basic_conversation
python communication_simulator_test.py --individual content_validation
python communication_simulator_test.py --individual cross_tool_continuation
python communication_simulator_test.py --individual memory_validation
```

#### ตัวเลือกอื่น
```bash
python communication_simulator_test.py --list-tests
python communication_simulator_test.py --tests basic_conversation content_validation
```

### เช็กคุณภาพโค้ด

ก่อนคอมมิทควรรัน linting:
```bash
ruff check .
black --check .
isort --check-only .

# แก้ไขให้อัตโนมัติ
ruff check . --fix
black .
isort .
```

## แต่ละชุดเทสต์ครอบคลุมอะไรบ้าง

### Unit Test
- ฟังก์ชันของผู้ให้บริการโมเดล (การเริ่มต้น, ตรวจ capability)
- การทำงานของเครื่องมือ MCP
- ระบบความจำบทสนทนา (thread, continuation)
- การจัดการไฟล์ (ตรวจ path, โทเคน, การลบซ้ำ)
- กลไก Auto mode (เลือกโมเดล, fallback)

### HTTP Recording/Replay (VCR)
- เทสต์ API ที่มีต้นทุนสูง (เช่น o3-pro) ด้วยการบันทึก/เล่นซ้ำ
- ยืนยันความถูกต้องกับผลจริงครั้งแรก แล้วใช้การเล่นซ้ำเพื่อลดค่าใช้จ่าย
- ดูรายละเอียดใน [vcr-testing.md](./vcr-testing.md)

### Simulator Test
- บทสนทนาพื้นฐานหลายรอบ
- การสานต่อบริบทข้ามเครื่องมือ
- การลบไฟล์ซ้ำ (dedup)
- การเลือกโมเดลตามคอนฟิก
- การจัดสรรโทเคนในสถานการณ์จริง
- การเก็บ/เรียกบทสนทนาผ่าน Redis (ถ้าตั้งค่า)

## การมีส่วนร่วม

แนวทางการส่งงาน ทดสอบที่ต้องผ่าน และมาตรฐานโค้ดดูเพิ่มเติมที่ [Contributing Guide](./contributions.md)

### สรุปคำสั่งที่ใช้บ่อย
```bash
./code_quality_checks.sh
python -m pytest -xvs
python communication_simulator_test.py
```

อย่าลืมให้เทสต์ทั้งหมดผ่านก่อนเปิด PR!
