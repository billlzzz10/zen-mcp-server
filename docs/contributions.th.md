# แนวทางการมีส่วนร่วมกับ Zen MCP Server

ขอบคุณที่สนใจร่วมพัฒนา! เอกสารนี้สรุปขั้นตอนการทำงาน มาตรฐานโค้ด และวิธีส่งผลงานคุณภาพ

## เริ่มต้น

1. fork รีโพบน GitHub
2. clone มายังเครื่องของคุณ
3. ตั้งค่าสภาพแวดล้อมด้วย:
   ```bash
   ./run-server.sh
   ```
4. สร้าง branch ใหม่จาก `main`:
   ```bash
   git checkout -b feat/your-feature-name
   ```

## กระบวนการพัฒนา

### 1. มาตรฐานคุณภาพโค้ด

ต้องผ่านการตรวจคุณภาพทั้งหมดก่อนส่ง

**วิธีแนะนำ (อัตโนมัติ):**
```bash
pre-commit install        # รันครั้งแรก
# หลังจากนี้ lint จะทำงานก่อน commit ทุกครั้ง (ruff, black, isort)
```

**วิธี manual:**
```bash
./code_quality_checks.sh
```
สคริปต์นี้จะรัน Ruff (พร้อม autofix), Black, isort, pytest ทั้งชุด และยืนยันว่าผ่าน 100%

**คำสั่งแยก:**
```bash
ruff check .
black --check .
isort --check-only .
ruff check . --fix
black .
isort .
python -m pytest -xvs
python communication_simulator_test.py
```

> **สำคัญ:**
> - เทสต์ทุกตัวต้องผ่าน 100%
> - lint ต้องไม่เหลือข้อผิดพลาด
> - หากเทสต์ล้มใน CI จะไม่รับ PR

### 2. ข้อกำหนดด้านเทสต์

- ฟีเจอร์ใหม่ต้องมี unit test ครอบคลุมกรณีสำเร็จและข้อผิดพลาด
- การเปลี่ยนแปลงเครื่องมือ (tools) ต้องมี simulator test เพิ่ม/ปรับใน `simulator_tests/`
- แก้บั๊กต้องเพิ่ม regression test ที่จับปัญหาเดิมได้

ตั้งชื่อไฟล์ทดสอบตามรูปแบบ `test_<feature>_<scenario>.py`

### 3. การเปิด PR

**รูปแบบชื่อ PR (Conventional Commits):**
- `feat: ...` เพิ่มฟีเจอร์ (เพิ่มเวอร์ชันย่อย)
- `fix: ...` แก้บั๊ก (เพิ่มเวอร์ชัน patch)
- `breaking: ...` หรือ `BREAKING CHANGE: ...` (เพิ่มเวอร์ชันหลัก)
- `perf: ...`, `refactor: ...` (patch)
- `docs: ...`, `chore: ...`, `test: ...`, `ci: ...`, `style: ...` (ไม่กระทบเวอร์ชัน)

**ใช้ PR template** และตรวจเช็กให้ครบ:
- ชื่อ PR ถูกต้อง
- รัน `./code_quality_checks.sh` และผ่านทั้งหมด
- ตรวจเองเรียบร้อย
- เพิ่มเทสต์ครบตามการเปลี่ยนแปลง
- อัปเดตเอกสารที่เกี่ยวข้อง
- เทสต์หน่วยและ simulator ที่เกี่ยวข้องผ่านทั้งหมด

### 4. สไตล์การเขียนโค้ด

- ใช้ Black (PEP 8) และใส่ type hints
- มี docstring สำหรับฟังก์ชันและคลาสที่เปิดเผย (public)
- ฟังก์ชันควรกระชับ (<50 บรรทัดถ้าเป็นไปได้)
- ใช้ชื่อสื่อความหมาย
- จัด import ด้วย isort (มาตรฐาน/third-party/โลคอล)

ตัวอย่าง docstring:
```python
def process_model_response(
    response: ModelResponse,
    max_tokens: Optional[int] = None
) -> ProcessedResult:
    """ประมวลผลและตรวจสอบคำตอบจากโมเดล"""
    ...
```

### 5. ประเภทการมีส่วนร่วมเฉพาะทาง

- เพิ่มผู้ให้บริการใหม่: ดู [Adding a New Provider](./adding_providers.md)
- เพิ่มเครื่องมือ: ดู [Adding a New Tool](./adding_tools.md)
- แก้ไขเครื่องมือเดิม: รักษาความเข้ากันได้ย้อนหลัง, อัปเดตเทสต์และเอกสาร

### 6. มาตรฐานเอกสาร

- อัปเดต README หากมีผลต่อผู้ใช้
- เพิ่ม docstring และปรับไฟล์ใน `docs/` ที่เกี่ยวข้อง
- ใส่ตัวอย่างใช้งานเมื่อมีฟีเจอร์ใหม่

### 7. ข้อความคอมมิต

- บรรทัดแรกสรุป (≤50 ตัวอักษร)
- เว้นบรรทัดว่างแล้วอธิบายรายละเอียด (ถ้าต้องการ)
- อ้าง issue ด้วย `Fixes #123`

ตัวอย่าง:
```
feat: add retry logic to Gemini provider

Implements exponential backoff...

Fixes #45
```

## ปัญหาพบบ่อยและวิธีแก้

- **Lint ไม่ผ่าน:** `ruff check . --fix`, `black .`, `isort .`
- **เทสต์ล้ม:** ตรวจเอาต์พุต, รันเฉพาะเทสต์ที่พัง, ตรวจสภาพแวดล้อม
- **Import error:** ตรวจว่าเปิด virtualenv แล้ว (`source .zen_venv/bin/activate`) และติดตั้ง dependencies ครบ

## การขอความช่วยเหลือ

- คำถามทั่วไป: เปิด issue พร้อม label `question`
- รายงานบั๊ก: ใช้ template bug report
- ขอฟีเจอร์: ใช้ template feature request
- พูดคุยทั่วไป: GitHub Discussions

## จรรยาบรรณ

- เคารพและเปิดรับทุกคน
- ให้ข้อเสนอแนะอย่างสร้างสรรค์
- ตั้งสมมติฐานถึงเจตนาดีของผู้อื่น

## การยกย่อง

- แสดงในหน้าผู้ร่วมพัฒนาบน GitHub
- กล่าวถึงใน release notes หากเป็นการเปลี่ยนแปลงสำคัญ
- ให้เครดิตพิเศษกับผลงานโดดเด่น

ขอบคุณที่ร่วมพัฒนา Zen MCP Server! ผลงานของคุณช่วยให้เครื่องมือนี้ดียิ่งขึ้นสำหรับทุกคน