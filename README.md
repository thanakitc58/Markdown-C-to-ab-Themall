# Markdown-C-to-ab-Themall

เอกสารอ้างอิงแปลงโปรโมชัน **Mer C → Mer AB** (เว็บ / payload)

**ดูเป็นหลัก:** `payload_AB_Reference.md` + ไฟล์ราย profile (`MerC_to_AB_P001.md` …) + `MerC_to_AB_Function_Split.md`  
(+ ชื่อบนจอจากไฟล์ To-Be xlsx)

---

## ไฟล์ล่าสุด (ใช้ทำงานเว็บ)

| ลำดับ | ไฟล์ | ใช้ทำอะไร |
|------|------|-----------|
| 1 | `Copy of The To-Be Promotion Template Structure.xlsx` | **ชื่อจริงบนเว็บ** — ชีต `MerAB - Header` / `MerAB - item Details` |
| 2 | `payload_AB_Reference.md` | **สัญญา JSON ฝั่ง AB** — key ↔ ชื่อใน To-Be (แถวชีต) |
| 3 | `MerC_to_AB_ByProfile.md` | **index 9 profile** + ตารางเปรียบเทียบ |
| 4 | `MerC_to_AB_{P001\|P010\|…\|F003}.md` | สเปก C→AB แยกราย profile (ตาราง + สูตร + skeleton JSON) |
| 5 | `MerC_to_AB_Function_Split.md` | โครงโค้ด — shared → กลุ่ม A/B/C → if ราย profile |

```
ชื่อบนเว็บ (To-Be xlsx)  ↔  payload_AB_Reference.md  ↔  ไฟล์ราย profile + Function_Split
```

### ราย profile (9 ไฟล์)

| กลุ่ม | ไฟล์ |
|------|------|
| A | `MerC_to_AB_P001.md` · `MerC_to_AB_P010.md` · `MerC_to_AB_P011.md` · `MerC_to_AB_P015.md` |
| B | `MerC_to_AB_D001.md` · `MerC_to_AB_F001.md` |
| C | `MerC_to_AB_D002.md` · `MerC_to_AB_D003.md` · `MerC_to_AB_F003.md` |

---

## วิธีดูไฟล์

### ถ้าจะเริ่มเขียน convert

เปิดตามลำดับนี้:

1. `payload_AB_Reference.md` — contract ปลายทางของ JSON ฝั่ง AB
2. `MerC_to_AB_ByProfile.md` — ตารางเปรียบเทียบ 9 profile
3. `profile/MerC_to_AB_*.md` — กฎเฉพาะ profile ที่กำลังทำ
4. `MerC_to_AB_Function_Split.md` — โครง helper / router ที่ควรแยกในโค้ด
5. `MerC_to_AB_Header_Shared.md` — สูตร shared ฝั่ง header
6. `MerC_to_AB_Phase1_Shared.md` — กฎร่วมอื่นๆ ที่ไม่ควรเขียนซ้ำ

### ไฟล์ไหนคืออะไร

| ไฟล์ | ใช้ดูอะไร |
|------|-----------|
| `payload_AB_Reference.md` | key ฝั่ง AB, required/optional/auto, และชื่อ field บน To-Be |
| `MerC_to_AB_ByProfile.md` | สรุปว่าทั้ง 9 profile ต่างกันยังไง เช่น `get.field8`, `% normalize`, `validTime*`, `referenceCode`, online EN/TH |
| `profile/MerC_to_AB_*.md` | spec ราย profile: ตาราง C -> AB, สูตร, skeleton JSON, checklist |
| `MerC_to_AB_Function_Split.md` | โครง implementation ที่แนะนำ เช่น `pickDiscount`, `lookupBuyGetQty`, `materialType`, `mapHeaderShared` |
| `MerC_to_AB_Header_Shared.md` | logic shared ของ header เช่น date format, `NOBP`, normalize rebate, normalize contract type, lookup theme / purchasing group |
| `MerC_to_AB_Phase1_Shared.md` | กฎร่วมของทุก profile เช่น MAT vs MGP, omit key ว่าง, lookup / skip / normalize พื้นฐาน |

### ถ้ายังงงว่า field ต้นทางมาจากไหน

ค่อยเปิดไฟล์เสริมพวกนี้:

| ไฟล์ | ใช้เมื่อ |
|------|---------|
| `Mer-C_Field_Mapping_Guide.md` | อยากรู้ว่า field ฝั่ง Mer C อยู่คอลัมน์ไหน และพึ่ง master / lookup อะไรบ้าง |
| `MerC_to_AB_ExistsInAB_Mapping.md` | ต้องไล่ mapping แบบอิง Excel cell หรือ export กลับไฟล์ AB |
| `payload.md` | อยากดูตัวอย่าง shape ของ payload ฝั่ง C |
| `Mer-C_Convert_To_STD.txt` | ต้องแกะ VBA ต้นฉบับเพื่อเช็ก logic จริงของ macro |

### ถ้าจะดู logic lookup / สูตรโดยเฉพาะ

ดู 4 ไฟล์นี้ก่อน:

- `MerC_to_AB_Header_Shared.md` — lookup / normalize ฝั่ง header
- `MerC_to_AB_Function_Split.md` — helper กลาง เช่น qty / discount / material type
- `Mer-C_Field_Mapping_Guide.md` — master ที่ใช้ lookup จาก Mer C
- `MerC_to_AB_ExistsInAB_Mapping.md` — พฤติกรรมสูตร / lookup แบบอิง Excel macro

ตัวอย่าง logic สำคัญที่ต้องรู้:

- `Theme` / `Purchasing Group` อาจต้อง lookup ชื่อเต็ม
- `Mechanic` ใช้ lookup `buyQty` / `getQty`
- `Vendor` ว่าง -> `NOBP`
- `Rebate Chargeback` และ `Contract Type` มี normalize rule
- ส่วนลดเลือกตามลำดับ `promo -> amt -> %`
- `%` บาง profile ใช้ normalize แบบเต็ม บาง profile ใช้แบบง่าย

### ถ้าจะเทส D001

เปิดชุดนี้:

- `profile/MerC_to_AB_D001.md`
- `tungconvert/D001/README.md`
- `tungconvert/D001/*.json`
- `payload_AB_Reference.md`

> หมายเหตุ: fixture ตอนนี้มีครบสุดที่ D001; profile อื่นยังพึ่ง spec เป็นหลัก

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
| เขียน convert | `payload_AB_Reference` → Function_Split → ไฟล์ราย profile |
| ไล่ bug ราย profile | `MerC_to_AB_Pxxx.md` → ถ้ายังงงค่อยเปิด `Mer-C_Convert_To_STD.txt` |
| export กลับ Excel | ExistsInAB + ภาคผนวก A ใน `payload_AB_Reference` |

**Scope ลูกค้า:** 9 profile — P001, P010, P011, P015, D001, F001, D002, D003, F003  
**Out of scope:** P100, P103, F002
