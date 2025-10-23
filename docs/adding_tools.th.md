# การเพิ่มเครื่องมือใหม่ให้ Zen MCP Server

เครื่องมือของ Zen MCP เป็นคลาส Python ที่สืบทอดจากโครงสร้างร่วมใน `tools/shared/base_tool.py` ทุกเครื่องมือจะต้องมีโมเดลรับคำขอ (Pydantic), system prompt และเมท็อดที่คลาสฐานกำหนดไว้เป็น abstract วิธีที่เร็วที่สุดคือคัดลอกเครื่องมือที่ใกล้เคียงที่สุดกับสิ่งที่ต้องการ (`tools/chat.py` สำหรับงาน request/response ธรรมดา, `tools/consensus.py` หรือ `tools/codereview.py` สำหรับเวิร์กโฟลว์หลายขั้น) เพื่อไม่ให้โค้ดหลุดคอนเวนชัน

## 1. เลือกสถาปัตยกรรมของเครื่องมือ

Zen รองรับ 2 รูปแบบ (อยู่ใน `tools/simple/base.py` และ `tools/workflow/base.py`)

- **SimpleTool**: เรียกโมเดลครั้งเดียว รับคำขอ → ประกอบพรอมต์ → ส่งไปที่โมเดล → คืนคำตอบ คลาสฐานดูแลการสร้างสคีมา การต่อบทสนทนา การโหลดไฟล์ การคุมอุณหภูมิ การ retry และ hook สำหรับจัดรูปคำตอบ
- **WorkflowTool**: เวิร์กโฟลว์หลายขั้นตอนผ่าน `BaseWorkflowMixin` เครื่องมือจะสะสมผลลัพธ์ บังคับให้ Claude หยุดคิดก่อนข้ามขั้น และเรียกโมเดลผู้เชี่ยวชาญได้ตามเงื่อนไข ใช้เมื่อการทำงานต้องมีหลายช่วง เช่น debug, codereview, consensus

ไม่แน่ใจให้เทียบตัวอย่าง `tools/chat.py` (Simple) กับ `tools/consensus.py` (Workflow)

## 2. หน้าที่ร่วมของเครื่องมือ

ไม่ว่าจะใช้รูปแบบใด คลาสที่สืบทอด `BaseTool` ต้อง override เมท็อดเหล่านี้:

- `get_name()` – ชื่อเครื่องมือที่ใช้ใน MCP registry
- `get_description()` – คำอธิบายสั้น ๆ แบบ action-oriented
- `get_system_prompt()` – นำเข้า prompt จาก `systemprompts/`
- `get_input_schema()` – ใช้ตัวช่วย `SchemaBuilder` หรือ `WorkflowSchemaBuilder` (override เมื่อจำเป็นต้องคุม schema เอง)
- `get_request_model()` – คืนคลาส Pydantic ที่ตรวจสอบพารามิเตอร์
- `async prepare_prompt(...)` – ประกอบเนื้อหาที่จะส่งให้โมเดล (มี helper เช่น `prepare_chat_style_prompt`)

ส่วนอื่น ๆ เช่น การเลือกโมเดล (`ToolModelCategory`), การจำบทสนทนา, token budgeting, การจัดการ error/ retry, serialization ครอบคลุมโดยคลาสฐานแล้ว Override พิเศษเมื่อจำเป็นเท่านั้น (เช่น `get_default_temperature`, `format_response`)

## 3. สร้าง Simple Tool

1. **นิยาม request model** ที่สืบทอด `ToolRequest` เพื่อกำหนดฟิลด์และ validation
2. **สร้างคลาสเครื่องมือ** สืบทอด `SimpleTool` แล้ว override เมท็อดที่จำเป็น โดยใช้ `SchemaBuilder` และฟิลด์คงที่ที่เตรียมให้ใน `SimpleTool`

```python
from pydantic import Field
from systemprompts import CHAT_PROMPT
from tools.shared.base_models import ToolRequest
from tools.simple.base import SimpleTool

class ChatRequest(ToolRequest):
    prompt: str = Field(..., description="Your question or idea.")
    files: list[str] | None = Field(default_factory=list)
    working_directory: str = Field(
        ..., description="Absolute full directory path where the assistant AI can save generated code for implementation."
    )

class ChatTool(SimpleTool):
    def get_name(self) -> str:
        return "chat"

    def get_description(self) -> str:
        return "General chat and collaborative thinking partner."

    def get_system_prompt(self) -> str:
        return CHAT_PROMPT

    def get_request_model(self):
        return ChatRequest

    def get_tool_fields(self) -> dict[str, dict[str, object]]:
        return {
            "prompt": {"type": "string", "description": "Your question."},
            "files": SimpleTool.FILES_FIELD,
            "working_directory": {
                "type": "string",
                "description": "Absolute full directory path where the assistant AI can save generated code for implementation.",
            },
        }

    def get_required_fields(self) -> list[str]:
        return ["prompt", "working_directory"]

    async def prepare_prompt(self, request: ChatRequest) -> str:
        return self.prepare_chat_style_prompt(request)
```

หากต้องการคุม schema เองให้ override `get_input_schema()` (เช่นใน `tools/chat.py`), แต่ส่วนใหญ่ให้ `SimpleTool` จัดการรวมฟิลด์มาตรฐานให้อัตโนมัติ

## 4. สร้าง Workflow Tool

Workflow tool สืบทอด `WorkflowTool` และใช้ `BaseWorkflowMixin`

1. **สร้าง request model** ที่สืบทอด `WorkflowRequest` (หรือคลาสย่อย) เพิ่มฟิลด์ที่จำเป็น เช่น `CodeReviewRequest`
2. **Override hook** ที่กำกับขั้นตอน เช่น `get_required_actions(...)`, `should_call_expert_analysis(...)`, `prepare_expert_analysis_context(...)`
3. **สคีมา** ใช้ค่าเริ่มต้นของ `WorkflowTool` (ผ่าน `WorkflowSchemaBuilder`) หรือ override ถ้าต้องตั้งเอง

```python
from pydantic import Field
from systemprompts import CONSENSUS_PROMPT
from tools.shared.base_models import WorkflowRequest
from tools.workflow.base import WorkflowTool

class ConsensusRequest(WorkflowRequest):
    models: list[dict] = Field(..., description="Models to consult (with optional stance).")

class ConsensusTool(WorkflowTool):
    def get_name(self) -> str:
        return "consensus"

    def get_description(self) -> str:
        return "Multi-model consensus workflow with expert synthesis."

    def get_system_prompt(self) -> str:
        return CONSENSUS_PROMPT

    def get_workflow_request_model(self):
        return ConsensusRequest

    def get_required_actions(self, step_number: int, confidence: str, findings: str, total_steps: int, request=None) -> list[str]:
        if step_number == 1:
            return ["Write the shared proposal all models will evaluate."]
        return ["Summarize the latest model response before moving on."]

    def should_call_expert_analysis(self, consolidated_findings, request=None) -> bool:
        return not (request and request.next_step_required)

    def prepare_expert_analysis_context(self, consolidated_findings) -> str:
        return "\n".join(consolidated_findings.findings)
```

WorkflowTool เก็บประวัติขั้นตอนรวมให้และจัดการ continuation ID ถ้าต้องการค่า default ให้ใช้ helper เช่น `get_standard_required_actions`

## 5. ลงทะเบียนเครื่องมือ

1. สร้างหรือใช้ system prompt ใน `systemprompts/` แล้ว export ใน `systemprompts/__init__.py`
2. เพิ่มคลาสเครื่องมือใน `tools/__init__.py`
3. เพิ่มอินสแตนซ์ในพจนานุกรม `TOOLS` ของ `server.py`
4. (ถ้าต้องการ) เพิ่ม template ใน `PROMPT_TEMPLATES` เพื่อให้ไคลเอนต์เรียกง่าย
5. ตรวจว่า `DISABLED_TOOLS` รองรับการเปิด/ปิดเครื่องมือใหม่

## 6. ตรวจสอบเครื่องมือ

- รัน unit test ที่เกี่ยวข้อง: `python -m pytest tests/ -v -m "not integration"`
- เพิ่ม scenario ใน `simulator_tests/communication_simulator_test.py` แล้วรันแบบเฉพาะเคสหรือ `--quick`
- หากเครื่องมือเกี่ยวข้องกับผู้ให้บริการภายนอก ให้พิจารณาเทสต์แบบ integration (`./run_integration_tests.sh --with-simulator`)

ทำตามขั้นตอนเหล่านี้จะช่วยให้เครื่องมือใหม่สอดคล้องกับโครงสร้างเดิมและลดความต่างระหว่างเอกสารกับโค้ดจริง