# API Lookup Tool

`apilookup` บังคับให้โมเดลไปค้นข้อมูล API/SDK ล่าสุดจริง ๆ แทนที่จะพึ่งข้อมูลเก่าจาก training cut-off โดยเฉพาะสำหรับแพลตฟอร์มที่ผูกกับเวอร์ชัน OS (iOS, macOS, Android ฯลฯ) ที่เปลี่ยนบ่อย นอกจากนี้ยังรันอยู่ในซับโปรเซสเพื่อลดการใช้โทเคนในคอนเท็กซ์หลักของคุณ

## ทำไมต้องใช้

### เมื่อไม่มี Zen
```
ผู้ใช้: "How do I add glass look to a button in Swift?"
AI: (ค้นจากความรู้เก่า)
    "SwiftUI glass morphism ... iOS 18 ..."
```
ผลลัพธ์: ได้ API ของ iOS 18 ไม่ใช่ iOS 26 ปัจจุบัน

### เมื่อใช้ apilookup
```
ผู้ใช้: "use apilookup how do I add glass look to a button in swift?"
AI: 
  1) ค้น "what is the latest iOS version 2025" -> พบว่า iOS 26 ล่าสุด
  2) ค้น "iOS 26 SwiftUI glass effect button 2025" -> ได้ API ที่ถูกต้อง
```

## จุดเด่น

1. **ตรวจเวอร์ชัน OS ก่อนเสมอ**
   - ค้นหาว่าเวอร์ชันปัจจุบันคืออะไร
   - ห้ามอ้างความรู้เดิมโดยไม่ตรวจสอบ
2. **เน้นแหล่งข้อมูลทางการ** (เอกสาร official, repo, registry, blog ทางการ)
3. **ให้ผลลัพธ์ที่นำไปใช้ได้จริง** (หมายเลขเวอร์ชัน, breaking changes, ตัวอย่างโค้ด, คำเตือนการ deprecate)

## ใช้เมื่อใด
- ต้องการเอกสาร API/SDK ล่าสุด
- ทำงานกับเฟรมเวิร์กที่อิง OS
- ตรวจว่าคุณสมบัติรองรับบนเวอร์ชันใด
- หา migration guide หรือ breaking change
- ตรวจคำเตือนด้านความปลอดภัย/การยกเลิกใช้

## ตัวอย่างคำสั่ง
```
use apilookup how do I add glass look to a button in swift?
use apilookup what's the latest Stripe Python SDK version since v7?
use apilookup check the latest React release and new hooks in 2025
```

## การทำงาน
1. รับคำถามเกี่ยวกับ API/SDK
2. ฉีดคำสั่งบังคับให้ค้นข้อมูลปีปัจจุบัน
3. หากเกี่ยวกับ OS ต้องค้นเวอร์ชันก่อนแล้วค่อยหา API
4. ส่งคืนชุดคำสั่งพร้อมคำอธิบายการค้น
5. โมเดลดำเนินการค้นจริงและสรุปผลล่าสุดให้นำไปใช้

ผลลัพธ์เป็น JSON ที่มี `status: "web_lookup_needed"`, `instructions` และ `user_prompt`

## หมายเหตุสำหรับ Codex CLI

ถ้าใช้ผ่าน Codex CLI ต้องเปิดเครื่องมือค้นเว็บของ Codex ใน `~/.codex/config.toml`:
```toml
[tools]
web_search = true
```
หากไม่เปิด Zen จะขอให้ค้นเว็บซ้ำ ๆ แต่ Codex จะทำไม่ได้
