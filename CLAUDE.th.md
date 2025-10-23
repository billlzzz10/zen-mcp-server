# คู่มือพัฒนา Claude สำหรับ Zen MCP Server

ไฟล์นี้รวบรวมคำสั่งและเวิร์กโฟลว์ที่จำเป็นสำหรับการพัฒนาและดูแล Zen MCP Server เมื่อทำงานร่วมกับ Claude ใช้คำแนะนำนี้เพื่อรันเช็กคุณภาพ จัดการเซิร์ฟเวอร์ ตรวจสอบล็อก และรันเทสต์ได้อย่างมีประสิทธิภาพ

## คำสั่งอ้างอิงด่วน

### การตรวจคุณภาพโค้ด

ก่อนปรับแก้หรือส่ง PR ให้รันเช็กคุณภาพแบบครบถ้วนเสมอ:

```bash
# เปิด virtual environment ก่อน
source venv/bin/activate

# รันเช็กทั้งหมด (lint, format, tests)
./code_quality_checks.sh
```

สคริปต์นี้จะทำให้:
- Ruff linting พร้อม auto-fix
- จัดรูปแบบด้วย Black
- เรียง import ผ่าน isort
- รันชุด unit test ทั้งหมด (ไม่รวม integration tests)
- ยืนยันว่าการตรวจทุกอย่างผ่านครบ 100%

**รัน integration tests (ต้องมี API key):**
```bash
./run_integration_tests.sh
./run_integration_tests.sh --with-simulator
```

### การจัดการเซิร์ฟเวอร์

#### ติดตั้งหรืออัปเดตเซิร์ฟเวอร์
```bash
./run-server.sh
```

สคริปต์จะ:
- สร้าง virtual environment
- ติดตั้ง dependencies
- สร้างหรืออัปเดตไฟล์ .env
- ตั้งค่า MCP ให้ทำงานร่วมกับ Claude
- ตรวจสอบ API key

#### ดูล็อก
```bash
./run-server.sh -f
# หรือ
tail -f logs/mcp_server.log
```

### การจัดการล็อก

#### ดูล็อกเซิร์ฟเวอร์
```bash
tail -n 500 logs/mcp_server.log
tail -f logs/mcp_server.log
tail -n 100 logs/mcp_server.log
grep "ERROR" logs/mcp_server.log
grep "tool_name" logs/mcp_activity.log
```

#### ติดตามการทำงานของเครื่องมือ
```bash
tail -n 100 logs/mcp_activity.log
tail -f logs/mcp_activity.log
tail -f logs/mcp_activity.log | grep -E "(TOOL_CALL|TOOL_COMPLETED|ERROR|WARNING)"
```

#### ไฟล์ล็อกที่มีให้

```bash
# ล็อกหลัก (สูงสุด 20MB เก็บย้อนหลัง 10 ไฟล์)
tail -f logs/mcp_server.log

# ล็อกกิจกรรมเครื่องมือ (สูงสุด 20MB เก็บย้อนหลัง 5 ไฟล์)
tail -f logs/mcp_activity.log
```

**สำหรับการวิเคราะห์ด้วยโค้ด (ใช้ในเทสต์):**
```python
from simulator_tests.log_utils import LogUtils

recent_logs = LogUtils.get_recent_server_logs(lines=500)
errors = LogUtils.check_server_logs_for_errors()
matches = LogUtils.search_logs_for_pattern("TOOL_CALL.*debug")
```

### การทดสอบ

Simulation tests ใช้ตรวจสอบว่าเซิร์ฟเวอร์ MCP ทำงานครบกระบวนการจริงด้วย API key ที่คุณตั้งค่าไว้

> **สำคัญ:** หลังแก้โค้ดทุกครั้ง รีสตาร์ทเซสชัน Claude เพื่อให้เห็นการเปลี่ยนแปลง

#### รัน simulation tests ทั้งชุด
```bash
python communication_simulator_test.py
python communication_simulator_test.py --verbose
```

#### โหมดทดสอบรวดเร็ว (แนะนำเมื่อเวลาจำกัด)
```bash
python communication_simulator_test.py --quick
python communication_simulator_test.py --quick --verbose
```

**Quick mode** ครอบคลุม 6 เทสต์สำคัญ:
- `cross_tool_continuation`
- `conversation_chain_validation`
- `consensus_workflow_accurate`
- `codereview_validation`
- `planner_validation`
- `token_allocation_validation`

เทสต์เหล่านี้ยืนยันฟังก์ชันหลัก: ความจำบทสนทนา (`utils/conversation_memory.py`), การทำงานของ chat tool, การจัดการไฟล์และการลบซ้ำ, การเลือกโมเดล (flash/flashlite/o3) และเวิร์กโฟลว์ข้ามเครื่องมือ

> เครื่องมือบางตัว (analyze, codereview, planner, consensus ฯลฯ) ต้องการพารามิเตอร์เฉพาะ ให้ทดสอบทีละตัวเมื่อจำเป็น

#### รัน simulation test รายกรณี
```bash
python communication_simulator_test.py --list-tests
python communication_simulator_test.py --individual basic_conversation
python communication_simulator_test.py --individual content_validation
python communication_simulator_test.py --individual cross_tool_continuation
python communication_simulator_test.py --individual memory_validation
python communication_simulator_test.py --tests basic_conversation content_validation
python communication_simulator_test.py --individual memory_validation --verbose
```

ตัวอย่างเทสต์เพิ่มเติม: `per_tool_deduplication`, `cross_tool_comprehensive`, `line_number_validation`, `model_thinking_config`, `o3_model_selection`, `ollama_custom_url`, `openrouter_fallback`, `openrouter_models`, `token_allocation_validation`, `testgen_validation`, `refactor_validation`, `consensus_stance`

#### รันเฉพาะ unit tests
```bash
python -m pytest tests/ -v -m "not integration"
python -m pytest tests/test_refactor.py -v
python -m pytest tests/test_refactor.py::TestRefactorTool::test_format_response -v
python -m pytest tests/ --cov=. --cov-report=html -m "not integration"
```

#### รัน integration tests (ใช้โมเดลโลคอลฟรี)

**เตรียมสภาพแวดล้อม:**
```bash
curl -fsSL https://ollama.ai/install.sh | sh
ollama serve
ollama pull llama3.2
export CUSTOM_API_URL="http://localhost:11434"
```

**รันเทสต์:**
```bash
python -m pytest tests/ -v -m "integration"
python -m pytest tests/test_prompt_regression.py::TestPromptIntegration::test_chat_normal_prompt -v
python -m pytest tests/ -v
```

> Integration tests ใช้โมเดล local-llama ผ่าน Ollama ฟรีไม่จำกัด แต่ต้องตั้ง `CUSTOM_API_URL` ให้ชี้ไปยัง Ollama และถูกตัดออกจาก `code_quality_checks.sh` เพื่อประหยัดเวลา

### เวิร์กโฟลว์การพัฒนา

#### ก่อนเริ่มแก้ไข
1. เปิด virtualenv: `source .zen_venv/bin/activate`
2. รัน `./code_quality_checks.sh`
3. ตรวจสุขภาพเซิร์ฟเวอร์: `tail -n 50 logs/mcp_server.log`

#### หลังแก้ไข
1. รัน `./code_quality_checks.sh` อีกรอบ
2. รัน `./run_integration_tests.sh`
3. รัน `python communication_simulator_test.py --quick`
4. หากจำเป็น รันเทสต์รายกรณี: `python communication_simulator_test.py --individual <test_name>`
5. ตรวจล็อก: `tail -n 100 logs/mcp_server.log`
6. รีสตาร์ท Claude เพื่อโหลดโค้ดใหม่

#### ก่อนคอมมิทหรือเปิด PR
1. เช็กคุณภาพรอบสุดท้าย
2. รัน integration tests
3. รัน quick mode
4. (ทางเลือก) รัน `./run_integration_tests.sh --with-simulator`
5. ยืนยันว่าเทสต์ผ่านทั้งหมด

### การแก้ปัญหาพบบ่อย

#### เซิร์ฟเวอร์มีปัญหา
```bash
./run-server.sh
grep "ERROR" logs/mcp_server.log | tail -20
which python   # ควรชี้ไป .../.zen_venv/bin/python
```

#### เทสต์ล้มเหลว
```bash
python communication_simulator_test.py --quick --verbose
python communication_simulator_test.py --individual <test_name> --verbose
tail -f logs/mcp_server.log
LOG_LEVEL=DEBUG python communication_simulator_test.py --individual <test_name>
```

#### ปัญหา lint
```bash
ruff check . --fix
black .
isort .
ruff check .
black --check .
isort --check-only .
```

### บริบทโครงสร้างไฟล์

- `./code_quality_checks.sh` – สคริปต์ตรวจคุณภาพครบวงจร
- `./run-server.sh` – ติดตั้งและจัดการเซิร์ฟเวอร์
- `communication_simulator_test.py` – เฟรมเวิร์กทดสอบ end-to-end
- `simulator_tests/` – โมดูลเทสต์จำลอง
- `tests/` – ชุด unit test
- `tools/` – การติดตั้ง MCP tools
- `providers/` – การเชื่อมต่อผู้ให้บริการโมเดล
- `systemprompts/` – แหล่ง system prompt
- `logs/` – ไฟล์ล็อก

### ข้อกำหนดสภาพแวดล้อม

- Python 3.9+ พร้อม virtualenv
- ติดตั้ง dependencies จาก `requirements.txt`
- ตั้งค่า API key ใน `.env`

คู่มือนี้ครอบคลุมสิ่งที่ต้องใช้ในการทำงานกับ Zen MCP Server ผ่าน Claude อย่างมีประสิทธิภาพ รัน `./code_quality_checks.sh` ทั้งก่อนและหลังแก้ไขเสมอเพื่อรักษาคุณภาพโค้ด
