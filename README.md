# Markdown-C-to-ab-Themall

เอกสารอ้างอิงแปลงโปรโมชัน **Mer C → Mer AB** (เว็บ / payload)

**ให้เพื่อนดูเป็นหลัก:** `payload_AB_Reference.md` + `MerC_to_AB_ByProfile.md` + `MerC_to_AB_Function_Split.md`  
(+ ชื่อบนจอจากไฟล์ To-Be xlsx)

---

## ไฟล์ล่าสุด (ใช้ทำงานเว็บ)

| ลำดับ | ไฟล์ | ใช้ทำอะไร |
|------|------|-----------|
| 1 | `Copy of The To-Be Promotion Template Structure.xlsx` | **ชื่อจริงบนเว็บ** — ชีต `MerAB - Header` / `MerAB - item Details` |
| 2 | `payload_AB_Reference.md` | **สัญญา JSON ฝั่ง AB** — key ↔ ชื่อใน To-Be (แถวชีต) |
| 3 | `MerC_to_AB_ByProfile.md` | ต่างกันทีละ profile (9 ตัว) — Buy/Get, %, time, reference |
| 4 | `MerC_to_AB_Function_Split.md` | โครงโค้ด — shared → กลุ่ม A/B/C → if ราย profile |

```
ชื่อบนเว็บ (To-Be xlsx)  ↔  payload_AB_Reference.md  ↔  ByProfile + Function_Split
```

> ไฟล์ขึ้นต้น `~$` = lock ของ Excel ไม่ใช่ตัวจริง

---

## ไฟล์อื่น (ยังไม่ sync ชื่อ To-Be / ของเก่า)

| ไฟล์ | สถานะ | ใช้เมื่อ |
|------|--------|---------|
| `MerC_to_AB_Phase1_Shared.md` | ยังไม่อัปชื่อ To-Be | อ่านกฎของร่วม (ส่งตรง / lookup / skip) — ชื่อฟิลด์ให้ยึด `payload_AB_Reference` |
| `MerC_to_AB_ExistsInAB_Mapping.md` | ของเก่า (As-Is → Excel cell) | ตอนต้อง export กลับไฟล์ AB / ไล่ cell |
| `Mer-C_Field_Mapping_Guide.md` | ของเก่า (VBA / Promotion BBY) | แกะ macro / template ต้นทาง |
| `payload.md` | ตัวอย่างฝั่ง C | ดู shape payload ต้นทาง |

อย่าเริ่มจาก ExistsInAB / Field Mapping Guide ถ้ายังไม่ต้อง export Excel

---

## เมื่อสงสัย logic ใน macro

| ไฟล์ | ใช้ทำอะไร |
|------|-----------|
| `Mer-C_Convert_To_STD.txt` | โค้ด VBA ต้นฉบับ (`Gen_Pxxx`) |
| `Mer C_Promotion Template_V3.7.xlsb` | template กรอกฝั่ง C |
| `Convert_C_to_AB_Template_V2.xlsb` | แปลง C→AB ฝั่ง Excel |

---

## เลือกเปิดตามงาน

| งาน | เปิดไฟล์ |
|-----|----------|
| เทียบชื่อบนจอ | To-Be xlsx + `payload_AB_Reference.md` |
| เขียน convert | `payload_AB_Reference` → Function_Split → ByProfile |
| ไล่ bug ราย profile | ByProfile → ถ้ายังงงค่อยเปิด `Mer-C_Convert_To_STD.txt` |
| export กลับ Excel | ExistsInAB + ภาคผนวก A ใน `payload_AB_Reference` |

**Scope ลูกค้า:** 9 profile — P001, P010, P011, P015, D001, F001, D002, D003, F003  
**Out of scope:** P100, P103, F002
