# การตั้งค่า Azure OpenAI

Azure OpenAI ช่วยให้ Zen MCP ติดต่อกับดีพลอย GPT-4o, GPT-4.1, GPT-5 และตระกูล o-series ที่คุณสร้างไว้บน Azure บทความนี้อธิบายการตั้งค่าที่เซิร์ฟเวอร์คาดหวัง: ตัวแปรสภาพแวดล้อมไม่กี่ตัว และไฟล์ JSON ที่ระบุรายชื่อดีพลอยทั้งหมดที่ต้องการเปิดให้ใช้งาน

## 1. ตัวแปรสภาพแวดล้อมที่จำเป็น

ตั้งค่าในไฟล์ `.env` (หรือบล็อก `env` ของ MCP):

```bash
AZURE_OPENAI_API_KEY=your_azure_openai_key_here
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
# AZURE_OPENAI_API_VERSION=2024-02-15-preview
```

หากไม่กำหนด API key และ endpoint ผู้ให้บริการ Azure จะไม่ถูกเปิดใช้งานเลย ควรปล่อยคีย์ว่างก็ต่อเมื่อปลายทางอนุญาตให้เข้าถึงแบบไม่ต้องยืนยันตัวตนจริง ๆ (พบได้ยาก)

## 2. สร้างรายการดีพลอยใน `conf/azure_models.json`

โมเดล Azure ถูกประกาศใน `conf/azure_models.json` (หรือไฟล์ที่ระบุผ่าน `AZURE_MODELS_CONFIG_PATH`) แต่ละเอนทรีใช้ schema เดียวกับ [`ModelCapabilities`](../providers/shared/model_capabilities.py) โดยเพิ่มคีย์บังคับอีกตัวคือ `deployment` ต้องตรงกับชื่อ deployment ใน Azure Portal (เช่น `prod-gpt4o`) เซิร์ฟเวอร์ใช้ค่านี้เพื่อส่งคำขอ หากเว้นว่างหรือสะกดผิด โมเดลนั้นจะถูกข้าม นอกจากนี้ยังเปิดฟีเจอร์เพิ่มเติมต่อโมเดลได้ เช่น ตั้ง `use_openai_response_api` เป็น `true` สำหรับดีพลอยที่ต้องเรียก endpoint `/responses` (เช่นโมเดลตระกูล O-series) หรือปล่อยว่างสำหรับ chat completions มาตรฐาน

```json
{
  "models": [
    {
      "model_name": "gpt-4o",
      "deployment": "prod-gpt4o",
      "friendly_name": "Azure GPT-4o EU",
      "intelligence_score": 18,
      "context_window": 600000,
      "max_output_tokens": 128000,
      "supports_temperature": false,
      "temperature_constraint": "fixed",
      "aliases": ["gpt4o-eu"],
      "use_openai_response_api": false
    }
  ]
}
```

เคล็ดลับ:
- คัดลอก `conf/azure_models.json` ไปไว้ในรีโพของคุณแล้ว commit หรือกำหนด `AZURE_MODELS_CONFIG_PATH` ให้ชี้ไปที่ไฟล์คัสตอม
- เพิ่มหนึ่งอ็อบเจ็กต์ต่อหนึ่งดีพลอย ตั้ง alias เพื่อเรียกสั้น ๆ เช่น `gpt4o-eu` ได้
- ฟิลด์ความสามารถอื่น ๆ เป็นตัวเลือก ยกเว้น `model_name`, `deployment`, `friendly_name` ถ้าไม่กำหนดจะใช้ค่าปลอดภัยแบบ conservative
- ตั้ง `use_openai_response_api=true` สำหรับโมเดลที่ต้องเรียก endpoint `/responses` (เช่น O3) ปล่อยว่างสำหรับ chat ปกติ

## 3. กำหนดข้อจำกัดเพิ่มเติม (ทางเลือก)

ใช้ `AZURE_OPENAI_ALLOWED_MODELS` เพื่อจำกัดว่ามีโมเดลใดบ้างที่ Claude สามารถเข้าถึงได้:

```bash
AZURE_OPENAI_ALLOWED_MODELS=gpt-4o,gpt-4o-mini
```

ระบบจะจับคู่ alias โดยไม่สนใจตัวพิมพ์ใหญ่-เล็ก

## 4. เช็กลิสต์อย่างรวดเร็ว

- [ ] ตั้ง `AZURE_OPENAI_API_KEY` และ `AZURE_OPENAI_ENDPOINT`
- [ ] ไฟล์ `conf/azure_models.json` (หรือไฟล์ที่ `AZURE_MODELS_CONFIG_PATH` ชี้) มีรายการดีพลอยครบพร้อมเมทาดาตาที่ต้องการ
- [ ] (ทางเลือก) ตั้ง `AZURE_OPENAI_ALLOWED_MODELS` เพื่อจำกัดการใช้งาน
- [ ] รัน `./run-server.sh` ใหม่ แล้วใช้เครื่องมือ `listmodels` เพื่อตรวจว่าโมเดล Azure ปรากฏพร้อมข้อมูลถูกต้อง

อ่านเพิ่มเติม: [`docs/adding_providers.md`](adding_providers.md) สำหรับโครงสร้างผู้ให้บริการ และ [README (Provider Configuration)](../README.md#provider-configuration) สำหรับตัวอย่างคอนฟิกแบบรวดเร็ว