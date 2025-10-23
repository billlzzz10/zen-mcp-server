# การเพิ่มผู้ให้บริการใหม่

คู่มือนี้อธิบายวิธีเพิ่มผู้ให้บริการโมเดล AI รายใหม่ให้กับ Zen MCP Server สถาปัตยกรรมของผู้ให้บริการออกแบบให้ขยายได้ง่ายตามแพทเทิร์นคงที่

## ภาพรวม

ผู้ให้บริการแต่ละรายจะ:
- สืบทอด `ModelProvider` (พื้นฐาน) หรือ `OpenAICompatibleProvider` (สำหรับ API ที่เข้ากับรูปแบบ OpenAI)
- ประกาศโมเดลที่รองรับผ่านอ็อบเจ็กต์ `ModelCapabilities`
- ตอบสนองเมท็อด abstract ขั้นต่ำ (`get_provider_type()` และ `generate_content()`)
- ลงทะเบียนใน `configure_providers()` เพื่อให้เปิด/ปิดจากตัวแปรสภาพแวดล้อมได้
- ใช้คลาสช่วย เช่น `AzureOpenAIProvider` เมื่อต่างกันแค่การตั้งค่าคลายเอนต์

### สเกลคะแนนความฉลาด

ตั้ง `intelligence_score` (1–20) เพื่อควบคุมการจัดอันดับใน auto mode และผลลัพธ์ของ `listmodels` คะแนนรันไทม์จะเริ่มจากค่าที่กำหนด แล้วบวกคะแนนเพิ่มเล็กน้อยตามบริบท, การรองรับ thinking และฟีเจอร์อื่น ๆ ([ดูรายละเอียด](model_ranking.md))

## เลือกแนวทางพัฒนา

**ทางเลือก A: เขียนผู้ให้บริการเต็มรูปแบบ (`ModelProvider`)**
- เหมาะสำหรับ API ที่มีฟีเจอร์เฉพาะหรือการยืนยันตัวตนต่างไปจาก OpenAI
- ควบคุมคำขอและการจัดการผลลัพธ์ได้ทั้งหมด
- เติม `MODEL_CAPABILITIES`, override `generate_content()` / `get_provider_type()` และ override การดึงแคตตาล็อก (เช่น `get_all_model_capabilities()`) เมื่อโมเดลมาจากรีจิสทรีหรือปลายทางอื่น (override `count_tokens()` เฉพาะเมื่อมีตัวนับโทเคนของผู้ให้บริการนั้น)

**ทางเลือก B: ใช้คลาส `OpenAICompatibleProvider`**
- เหมาะกับ API ที่ใช้รูปแบบ Chat Completions ของ OpenAI
- ประกาศ `MODEL_CAPABILITIES` และ override `get_provider_type()` เป็นหลัก การเชื่อมต่อ HTTP, การตรวจ alias และการจำกัด model จะจัดการให้ทั้งหมด
- อย่าลืมเรียก `_resolve_model_name()` ใน `generate_content()` หาก override เอง เพื่อให้ alias แปลงเป็นชื่อจริงก่อนเรียก SDK

**ทางเลือก C: ใช้ `AzureOpenAIProvider`**
- สำหรับดีพลอยโมเดล OpenAI บน Azure
- ใช้โค้ด pipeline เดิมแต่เปลี่ยนเป็นไคลเอนต์ Azure และ map ชื่อโมเดล → deployment
- กำหนด deployment ใน [`conf/azure_models.json`](../conf/azure_models.json) (หรือไฟล์ที่ `AZURE_MODELS_CONFIG_PATH` ชี้)
- เอนทรีต้องมีฟิลด์ `deployment` ตาม schema `ModelCapabilities` ดูตัวอย่างที่ [Azure OpenAI Configuration](azure_openai.md)

## ขั้นตอนปฏิบัติ

### 1. เพิ่มชนิดผู้ให้บริการ

เพิ่มค่าใน enum `ProviderType` (`providers/shared/provider_type.py`)

```python
class ProviderType(Enum):
    GOOGLE = "google"
    OPENAI = "openai"
    EXAMPLE = "example"  # เพิ่มตรงนี้
```

### 2. สร้างคลาสผู้ให้บริการ

#### ทางเลือก A: ผู้ให้บริการแบบกำหนดเองเต็มรูปแบบ

สร้างไฟล์ `providers/example.py`

```python
"""ตัวอย่างผู้ให้บริการโมเดล."""

import logging
from typing import Optional

from .base import ModelProvider
from .shared import (
    ModelCapabilities,
    ModelResponse,
    ProviderType,
    RangeTemperatureConstraint,
)

logger = logging.getLogger(__name__)


class ExampleModelProvider(ModelProvider):
    """ตัวอย่างผู้ให้บริการโมเดล."""

    MODEL_CAPABILITIES = {
        "example-large": ModelCapabilities(
            provider=ProviderType.EXAMPLE,
            model_name="example-large",
            friendly_name="Example Large",
            intelligence_score=18,
            context_window=100_000,
            max_output_tokens=50_000,
            supports_extended_thinking=False,
            temperature_constraint=RangeTemperatureConstraint(0.0, 2.0, 0.7),
            description="Large model for complex tasks",
            aliases=["large", "big"],
        ),
        "example-small": ModelCapabilities(
            provider=ProviderType.EXAMPLE,
            model_name="example-small",
            friendly_name="Example Small",
            intelligence_score=14,
            context_window=32_000,
            max_output_tokens=16_000,
            temperature_constraint=RangeTemperatureConstraint(0.0, 2.0, 0.7),
            description="Fast model for simple tasks",
            aliases=["small", "fast"],
        ),
    }

    def __init__(self, api_key: str, **kwargs):
        super().__init__(api_key, **kwargs)
        # สร้างไคลเอนต์ API ที่นี่

    def get_all_model_capabilities(self) -> dict[str, ModelCapabilities]:
        return dict(self.MODEL_CAPABILITIES)

    def get_provider_type(self) -> ProviderType:
        return ProviderType.EXAMPLE

    def generate_content(
        self,
        prompt: str,
        model_name: str,
        system_prompt: Optional[str] = None,
        temperature: float = 0.7,
        max_output_tokens: Optional[int] = None,
        **kwargs,
    ) -> ModelResponse:
        # เรียก API จริงและสร้าง ModelResponse
        ...
```

#### ทางเลือก B: ผู้ให้บริการที่เข้ากับ OpenAI

```python
from .openai_compatible import OpenAICompatibleProvider
from .shared import ModelCapabilities, ProviderType


class ExampleOpenAICompatibleProvider(OpenAICompatibleProvider):
    MODEL_CAPABILITIES = {
        "example-model": ModelCapabilities(
            provider=ProviderType.EXAMPLE,
            model_name="example-model",
            friendly_name="Example Model",
            intelligence_score=15,
            context_window=64_000,
            max_output_tokens=16_000,
            description="OpenAI-compatible example model",
            aliases=["example", "example-model"],
        ),
    }

    def __init__(self, api_key: str, base_url: str | None = None, **kwargs):
        super().__init__(api_key=api_key, base_url=base_url, **kwargs)

    def get_provider_type(self) -> ProviderType:
        return ProviderType.EXAMPLE
```

คลาส `OpenAICompatibleProvider` จะดูแล `MODEL_CAPABILITIES`, การแก้ alias, การตรวจโมเดลและข้อจำกัดให้อัตโนมัติ ส่วนใหญ่จึงต้องระบุตามตัวอย่างข้างต้นเท่านั้น

### 3. ลงทะเบียนผู้ให้บริการ

ใน `providers/registry.py` เพิ่มตัวแปรสภาพแวดล้อมที่ต้องตรวจ

```python
# ในฟังก์ชัน _get_api_key_for_provider
    ProviderType.EXAMPLE: "EXAMPLE_API_KEY",
```

ใน `server.py`:
1. **import คลาส**
```python
from providers.example import ExampleModelProvider
```
2. **เพิ่มใน `configure_providers()`**
```python
example_key = os.getenv("EXAMPLE_API_KEY")
if example_key:
    ModelProviderRegistry.register_provider(ProviderType.EXAMPLE, ExampleModelProvider)
    logger.info("Example API key found - Example models available")
```
3. **เพิ่มลำดับความสำคัญ** – แทรกใน `ModelProviderRegistry.PROVIDER_PRIORITY_ORDER` ให้เหมาะสม (ปกติลำดับคือ provider native → custom → openrouter)

### 4. ตั้งค่าสภาพแวดล้อม

เพิ่มใน `.env`
```bash
EXAMPLE_API_KEY=your_api_key_here
# ปิดเครื่องมือบางตัว (ถ้าต้องการ)
DISABLED_TOOLS=debug,tracer
# จำกัดโมเดลที่ใช้งาน (สำหรับ OpenAI-compatible)
EXAMPLE_ALLOWED_MODELS=example-model-large,example-model-small
```

ตัวอย่างสำหรับ Azure OpenAI:
```bash
AZURE_OPENAI_API_KEY=your_azure_openai_key_here
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
# กำหนดโมเดลใน conf/azure_models.json หรือไฟล์ที่ AZURE_MODELS_CONFIG_PATH ชี้
# AZURE_OPENAI_API_VERSION=2024-02-15-preview
# AZURE_OPENAI_ALLOWED_MODELS=gpt-4o,gpt-4o-mini
# AZURE_MODELS_CONFIG_PATH=/absolute/path/to/custom_azure_models.json
```

`ModelCapabilities.description` ช่วยให้ auto mode เลือกโมเดลเหมาะสมขึ้น

### 5. ทดสอบผู้ให้บริการ

สร้างเทสต์ง่าย ๆ เพื่อยืนยันการตั้งค่า

```python
provider = ExampleModelProvider("test-key")
capabilities = provider.get_capabilities("large")
assert capabilities.context_window > 0
assert capabilities.provider == ProviderType.EXAMPLE
```

## แนวคิดสำคัญ

### ลำดับความสำคัญของผู้ให้บริการ
1. ผู้ให้บริการ native (เช่น Gemini, OpenAI, Example)
2. ผู้ให้บริการ custom (โมเดลโลคอล)
3. OpenRouter (สำรองสำหรับโมเดลที่เหลือ)

### การตรวจโมเดล
`ModelProvider.validate_model_name()` เรียก `get_capabilities()` ให้โดยอัตโนมัติ จึงมักไม่ต้อง override เว้นแต่ต้องการยกเว้นบางกรณี (เช่น `CustomProvider` ปฏิเสธโมเดล OpenRouter เพื่อให้ไปผ่านผู้ให้บริการเฉพาะ)

### Alias ของโมเดล
Alias ที่ประกาศใน `ModelCapabilities` จะถูกใช้อัตโนมัติผ่าน `_resolve_model_name()` ทั้งในการตรวจและการเรียก API หาก override `generate_content()` อย่าลืมเรียกเมท็อดนี้ก่อนติดต่อ SDK

## ข้อควรปฏิบัติที่ดี
- ตรวจสอบชื่อโมเดลให้ตรงกับที่รองรับจริง
- ใช้ `ModelCapabilities` อย่างสม่ำเสมอ
- ระบุ alias ที่เข้าใจง่ายเพื่อประสบการณ์ใช้งานที่ดี
- ใส่ error handling และ logging เพื่อดีบักง่าย
- ทดสอบด้วย API จริงเพื่อให้มั่นใจว่าทำงานได้ครบ
- ยึดตามแพทเทิร์นใน `providers/gemini.py` และ `providers/custom.py`

## เช็กลิสต์สั้น ๆ
- [ ] เพิ่มค่าใน enum `ProviderType`
- [ ] สร้างคลาสผู้ให้บริการและเมท็อดที่ต้องใช้ครบ
- [ ] เพิ่ม mapping API key ใน `providers/registry.py`
- [ ] ปรับ `PROVIDER_PRIORITY_ORDER`
- [ ] import และลงทะเบียนใน `server.py`
- [ ] มีเทสต์พื้นฐานตรวจการ validate และ capability
- [ ] ทดลองเรียก API จริง

## ตัวอย่าง
- ผู้ให้บริการเต็มรูปแบบ: `providers/gemini.py`
- ผู้ให้บริการที่เข้ากับ OpenAI: `providers/custom.py`
- คลาสฐาน: `providers/base.py`
