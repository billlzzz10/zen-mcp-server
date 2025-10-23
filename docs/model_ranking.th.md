# การจัดอันดับความสามารถของโมเดล

โหมด auto ต้องการรายชื่อโมเดลสั้น ๆ แต่เชื่อถือได้ เซิร์ฟเวอร์จึงคำนวณคะแนนความสามารถ (capability rank) ให้ทุกโมเดลระหว่างรันด้วยสูตรง่าย ๆ:

1. เริ่มจาก `intelligence_score` ที่มนุษย์กำหนด (1–20) แล้วคูณ 5 เพื่อแปลงเป็นสเกล 0–100
2. เติมโบนัสเล็กน้อยสำหรับความสามารถเชิงเทคนิค:
   - **บริบท**: เพิ่มสูงสุด +5 (ใช้ log-scale เมื่อบริบทใหญ่กว่า ~1K โทเคน)
   - **ขนาดคำตอบ**: +2 ถ้า output ≥ 65K โทเคน, +1 ถ้า ≥ 32K
   - **คิดลึก (extended thinking)**: +3 หากรองรับ
   - **Function calling / JSON / ภาพ**: อย่างละ +1 หากรองรับ
   - **ปลายทาง custom**: −1 เพื่อให้บริการบนคลาวด์มีอันดับนำ เว้นแต่ปรับเอง
3. จำกัดคะแนนสุดท้ายให้อยู่ระหว่าง 0–100 เพื่อใช้งานต่อได้ง่าย

ตัวอย่างโค้ด:

```python
base = clamp(intelligence_score, 1, 20) * 5
ctx_bonus = min(5, max(0, log10(context_window) - 3))
output_bonus = 2 if max_output_tokens >= 65_000 else 1 if >= 32_000 else 0
feature_bonus = (
    (3 if supports_extended_thinking else 0)
    + (1 if supports_function_calling else 0)
    + (1 if supports_json_mode else 0)
    + (1 if supports_images else 0)
)
penalty = 1 if provider == CUSTOM else 0

effective_rank = clamp(base + ctx_bonus + output_bonus + feature_bonus - penalty, 0, 100)
```

โบนัสตั้งใจให้มีน้ำหนักน้อย เพราะ `intelligence_score` จากมนุษย์มีอิทธิพลหลัก ทำให้องค์กรควบคุมลำดับได้ง่าย

## เลือกค่า intelligence_score อย่างไร

ตารางแนะนำที่สอดคล้องกับลำดับชั้นของผู้ให้บริการทั่วไป:

| ค่า | แนวทาง |
|-----|---------|
| 18–19 | โมเดลเหตุผลระดับแนวหน้า (Gemini 2.5 Pro, GPT-5) |
| 15–17 | โมเดลทั่วไปที่แข็งแกร่ง บริบทใหญ่ (O3 Pro, DeepSeek R1) |
| 12–14 | ผู้ช่วยสมดุล (Claude Opus/Sonnet, Mistral Large) |
| 9–11  | โมเดลที่เร็ว (Gemini Flash, GPT-5 Mini, Mistral Medium) |
| 6–8   | โมเดลโลคอล/ประหยัด (Llama 3 70B, Claude Haiku) |
| ≤5    | โมเดลทดลองหรือเบา |

จดเหตุผลที่ให้คะแนนไว้ เพื่อให้การอัปเดตภายหลังคงความสอดคล้อง

## คะแนนนี้ถูกใช้ตอนไหนบ้าง

- คำอธิบายพารามิเตอร์ `model` ใน schema ของเครื่องมือ เมื่อเปิด auto mode
- ส่วน “top models” ของเครื่องมือ `listmodels`
- ข้อความ fallback เมื่อโมเดลที่ขอไม่พร้อมใช้งาน

ระบบจะจัดอันดับเฉพาะโมเดลที่ไม่ถูกจำกัดจากคอนฟิก ดังนั้นเฉพาะโมเดลที่อนุญาตเท่านั้นจะถูกแสดง

## ปรับแต่งเพิ่มเติมได้อย่างไร

- ปรับ `intelligence_score` ในคอนฟิกของผู้ให้บริการหรือไฟล์โมเดล custom
- สืบทอดคลาสผู้ให้บริการแล้ว override `get_effective_capability_rank()`
- ปรับคะแนนก่อนนำไปใช้ผ่าน `get_capabilities_by_rank()`

ส่วนใหญ่แค่ตั้ง `intelligence_score` ให้เหมาะก็พอที่จะควบคุมพฤติกรรม auto mode แล้วโดยไม่ต้องแก้โค้ด
