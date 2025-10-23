# HTTP Transport Recorder สำหรับทดสอบ

ยูทิลิตี้นี้บันทึกและเล่นซ้ำคำขอ HTTP ระดับ transport เพื่อทดสอบ API ที่มีค่าใช้จ่ายสูง (เช่น o3-pro) ด้วยข้อมูลจริง

## ภาพรวม

HTTP Transport Recorder ช่วย:
- ลดค่าใช้จ่าย (บันทึกครั้งเดียวแล้วเล่นซ้ำได้ไม่จำกัด)
- ทำให้เทสต์มีผลลัพธ์คงที่โดยอาศัยคำตอบจริง
- ใช้งานร่วมกับ httpx และ OpenAI SDK ได้อย่างราบรื่น
- กำจัดข้อมูลอ่อนไหว (PII) อัตโนมัติ

## เริ่มต้นอย่างรวดเร็ว

```python
from tests.transport_helpers import inject_transport

def test_expensive_api_call(monkeypatch):
    inject_transport(monkeypatch, "tests/openai_cassettes/my_test.json")
    result = await chat_tool.execute({"prompt": "2+2?", "model": "o3-pro"})
```

## วิธีทำงาน

1. **รันครั้งแรก** – หากยังไม่มี cassette จะบันทึกคำตอบจริง
2. **รันครั้งถัดไป** – ใช้ข้อมูลที่บันทึกไว้เล่นซ้ำ
3. **ต้องการบันทึกใหม่** – ลบไฟล์ cassette แล้วรันทดสอบอีกครั้ง

## การใช้งานในเทสต์

```python
from tests.transport_helpers import inject_transport

async def test_with_recording(monkeypatch):
    inject_transport(monkeypatch, "tests/openai_cassettes/my_test.json")
    result = await chat_tool.execute({"prompt": "2+2?", "model": "o3-pro"})
```

หากต้องการตั้งค่าด้วยตนเองดูตัวอย่างใน `test_o3_pro_output_text_fix.py`

## การลบข้อมูลอ่อนไหวอัตโนมัติ

ฟีเจอร์ sanitization จะลบข้อมูลต่อไปนี้ก่อนบันทึก:
- API key, bearer token และ header ที่เกี่ยวข้อง
- ข้อมูลส่วนบุคคล เช่น อีเมล IP เบอร์โทร
- URL ที่มีพารามิเตอร์สำคัญ
- รูปแบบพิเศษอื่น ๆ ที่กำหนดเองได้

ปกติเปิดใช้งานโดย `RecordingTransport` หากต้องปิดให้ตั้ง `sanitize=False`

```python
transport = TransportFactory.create_transport(cassette_path, sanitize=False)
```

## โครงสร้างไฟล์ที่เกี่ยวข้อง

```
tests/
├─ openai_cassettes/         # เก็บไฟล์บันทึก
│   └─ *.json                # เทสแต่ละรายการ
├─ http_transport_recorder.py
├─ pii_sanitizer.py
├─ transport_helpers.py
├─ sanitize_cassettes.py
└─ test_o3_pro_output_text_fix.py
```

## การ sanitize cassette ที่มีอยู่

```bash
python tests/sanitize_cassettes.py
python tests/sanitize_cassettes.py tests/openai_cassettes/my_test.json
python tests/sanitize_cassettes.py --no-backup
```

สคริปต์จะสร้างไฟล์แบ็กอัปพร้อม timestamp แล้วทำความสะอาดข้อมูลอ่อนไหว

## การจัดการต้นทุน

- เสียค่าใช้จ่ายเฉพาะรันแรกที่บันทึก
- เล่นซ้ำได้ฟรีหลังจากนั้น
- เหมาะสำหรับ CI เพราะไม่ต้องใช้ API key ในโหมด replay

## การบันทึกใหม่

```bash
rm tests/openai_cassettes/my_test.json
python -m pytest tests/test_o3_pro_output_text_fix.py
```

## รายละเอียดการทำงานภายใน

- `RecordingTransport` – บันทึกคำขอจริงพร้อม sanitize
- `ReplayTransport` – ตอบจาก cassette ที่บันทึกไว้
- `TransportFactory` – เลือกโหมดอัตโนมัติจากการมีอยู่ของไฟล์
- `PIISanitizer` – กฎ sanitize สำหรับข้อมูลอ่อนไหว

> แม้ว่าระบบจะล้างข้อมูลให้อัตโนมัติ แต่ควรตรวจเทปใหม่ก่อน commit เผื่อมีข้อมูลเฉพาะองค์กรที่ต้องลบเอง

ดูรายละเอียดเพิ่มได้ใน:
- `tests/http_transport_recorder.py`
- `tests/pii_sanitizer.py`
- `tests/transport_helpers.py`