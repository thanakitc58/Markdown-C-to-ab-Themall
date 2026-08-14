# Markdown-C-to-ab-Themall

เอกสารและไฟล์อ้างอิงสำหรับแปลงโปรโมชัน **Mer C → Mer AB** (เว็บ / payload)

---

## ต้องดูไฟล์ไหนบ้าง

### หลัก (ทำงานเว็บ / convert) — เปิดตามลำดับ

| ลำดับ | ไฟล์ | ดูเมื่อไหร่ |
|------|------|------------|
| 1 | `Copy of The To-Be Promotion Template Structure.xlsx` | **ชื่อจริงบนเว็บ** — ชีต `MerAB - Header` / `MerAB - item Details` |
| 2 | `payload_AB_Reference.md` | **สัญญา JSON ฝั่ง AB** — key อะไร ↔ ชื่อใน To-Be แถวไหน |
| 3 | `MerC_to_AB_Phase1_Shared.md` | **ของร่วมทุก profile** — ส่งตรง / เขียนซ้ำ / lookup / skip |
| 4 | `MerC_to_AB_ByProfile.md` | **ต่างกันทีละ profile** (9 ตัว) — Buy/Get, %, time, reference |
| 5 | `MerC_to_AB_Function_Split.md` | **โครงโค้ด** — shared → กลุ่ม A/B/C → if ราย profile |

ความสัมพันธ์:

```
ชื่อบนเว็บ (To-Be xlsx)  ↔  payload_AB_Reference.md  ↔  logic convert (Phase1 + ByProfile)
```


### ตัวอย่าง / ต้นทาง

| ไฟล์ | ใช้ทำอะไร |
|------|-----------|
| `payload.md` | ตัวอย่าง payload ฝั่ง **C** (ต้นทาง) |
| `MerC_to_AB_ExistsInAB_Mapping.md` | แมป cell เก่า Mer C → คอลัมน์ Excel AB |
| `Mer-C_Field_Mapping_Guide.md` | คู่มือภาพรวม mapping / ไฟล์ต้นทาง |

### เมื่อสงสัย logic ใน macro

| ไฟล์ | ใช้ทำอะไร |
|------|-----------|
| `Mer-C_Convert_To_STD.txt` | โค้ด VBA ต้นฉบับ (`Gen_Pxxx` ฯลฯ) |
| `Mer C_Promotion Template_V3.7.xlsb` | template กรอกฝั่ง C จริง |
| `Convert_C_to_AB_Template_V2.xlsb` | ไฟล์แปลง C→AB ฝั่ง Excel |

### ไม่ต้องเปิดบ่อย

- `~$...xlsx` — ไฟล์ล็อก Excel
- ชีต `(BAK)` / `(Bak)` ใน To-Be — ของเก่าสำรอง

---

## เลือกเปิดตามงาน

| งาน | เปิดไฟล์ |
|-----|----------|
| เทียบชื่อบนจอ | To-Be xlsx + `payload_AB_Reference.md` |
| เขียน convert รอบแรก | Phase1 → Function_Split → ByProfile |
| ไล่ bug ราย profile | ByProfile → ถ้ายังงงค่อยเปิด `Mer-C_Convert_To_STD.txt` |

**Scope ลูกค้า:** 9 profile — P001, P010, P011, P015, D001, F001, D002, D003, F003  
**Out of scope:** P100, P103, F002
