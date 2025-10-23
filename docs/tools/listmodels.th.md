# ListModels Tool - แสดงโมเดลที่พร้อมใช้งาน

`listmodels` แสดงผู้ให้บริการที่ตั้งค่าไว้ รายชื่อโมเดล, alias, ขนาด context และความสามารถ ช่วยให้เลือกโมเดลได้เหมาะกับงาน

## การใช้งาน
```
"Use zen to list available models"
```

## สิ่งที่จะแสดง
- สถานะผู้ให้บริการและลำดับความสำคัญ
- รายละเอียดโมเดล + alias
- ขนาด context window
- ความสามารถเสริม (extended thinking, vision ฯลฯ)
- แจ้งว่า API key ใดตั้งไว้/ยังขาด
- โมเดลที่ถูกจำกัดด้วย environment variable (หากตั้งไว้)

## ตัวอย่างเอาต์พุต (โดยสรุป)
```
?? Google (Gemini) - ✅ Configured
  • pro (gemini-2.5-pro) – 1M context, thinking mode
  • flash …

?? OpenAI - ✅ Configured
  • o3 …

?? Custom/Local - ✅ Configured
  • local-llama …

?? OpenRouter - ❌ Not configured (แนะนำตั้ง OPENROUTER_API_KEY)
```

## เมื่อไรควรใช้
- ก่อนวางแผนงานเพื่อดูว่ามีโมเดลใดให้เลือก
- ตรวจ capability เช่น vision, extended thinking
- ยืนยันว่า API key ตั้งถูกต้อง
- เลือกโมเดลตามขนาด context/ความเร็ว/ราคา

## ขึ้นอยู่กับการคอนฟิก
- `GEMINI_API_KEY`, `OPENAI_API_KEY`, `OPENROUTER_API_KEY`, `CUSTOM_API_URL`
- ตัวแปรจำกัดโมเดล เช่น `OPENAI_ALLOWED_MODELS`
- หากตั้ง restriction ไว้ เครื่องมือจะแสดงว่ามีโมเดลไหนถูกจำกัดบ้าง

## ไม่มีพารามิเตอร์เพิ่มเติม
เพียงเรียกใช้งานเพื่อดูข้อมูลทั้งหมด

## เทียบกับเครื่องมืออื่น
- `listmodels` → ดูตัวเลือกที่มี
- `chat` → ถามคำถามทั่วไปเกี่ยวกับการเลือกโมเดล
- `version` → ตรวจเวอร์ชันเซิร์ฟเวอร์และคอนฟิกอื่น
