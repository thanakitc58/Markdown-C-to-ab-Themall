# C → AB — Index ตาม Profile (9 ตัว)

สเปกราย profile แยกไฟล์ละตัว — ใช้เป็นแนวแปลง JSON (`LAYOUT: "AB"`)

| Profile | กลุ่ม | ไฟล์ |
|---------|------|------|
| **P001** | A — Get-only | [`profile/MerC_to_AB_P001.md`](profile/MerC_to_AB_P001.md) |
| **P010** | A — Get-only | [`profile/MerC_to_AB_P010.md`](profile/MerC_to_AB_P010.md) |
| **P011** | A — Get-only | [`profile/MerC_to_AB_P011.md`](profile/MerC_to_AB_P011.md) |
| **P015** | A — Get-only | [`profile/MerC_to_AB_P015.md`](profile/MerC_to_AB_P015.md) |
| **D001** | B — Buy+Get | [`profile/MerC_to_AB_D001.md`](profile/MerC_to_AB_D001.md) |
| **F001** | B — Buy+Get | [`profile/MerC_to_AB_F001.md`](profile/MerC_to_AB_F001.md) |
| **D002** | C (โครง = B) | [`profile/MerC_to_AB_D002.md`](profile/MerC_to_AB_D002.md) |
| **D003** | C — แยก element | [`profile/MerC_to_AB_D003.md`](profile/MerC_to_AB_D003.md) |
| **F003** | C — แยก index | [`profile/MerC_to_AB_F003.md`](profile/MerC_to_AB_F003.md) |

**เอกสารร่วม**

| ไฟล์ | ใช้เมื่อ |
|------|---------|
| `payload_AB_Reference.md` | ชื่อ field ปลายทาง AB (JSON contract) |
| `MerC_to_AB_Phase1_Shared.md` | ของร่วมทุก profile (header / skip / lookup) |
| `MerC_to_AB_Function_Split.md` | โครงโค้ด shared → router กลุ่ม → if ราย profile |

**อ่านยังไง:** ของร่วมก่อน → router กลุ่ม A/B/C → เปิดไฟล์ profile ที่รับผิดชอบ

---

## ตารางเปรียบเทียบเร็ว

| Profile | กลุ่ม | เขียนฝั่ง | `get.field8` | % normalize | `validTime*` | `referenceCode` | Online EN/TH (ถ้า P4) |
|---------|------|-----------|:------------:|:-----------:|:------------:|:---------------:|:---------------------:|
| **P001** | A | `get[]` เท่านั้น | ✅ | เต็ม | ✅ | ❌ | ❌ |
| **P010** | A | `get[]` เท่านั้น | ✅ | เต็ม | ❌ | ✅ | ✅ |
| **P011** | A | `get[]` เท่านั้น | ✅ | เต็ม | ✅ | ❌ (รับได้แต่ไม่เขียน) | ✅ |
| **P015** | A | `get[]` เท่านั้น | ✅ | เต็ม | ✅ | ❌ | ❌ |
| **D001** | B | `buy[]` + `get[]` | ❌ | เต็ม | ✅ | ❌ | ✅ |
| **F001** | B | `buy[]` + `get[]` | ❌ | **ง่าย** | ✅ | ❌ | ✅ |
| **D002** | C | `buy[]` + `get[]` (เหมือน B) | ❌ | เต็ม | ✅ | ❌ | ✅ |
| **D003** | C | buy **หรือ** get คนละ element | ❌ | เต็ม | ✅ | ❌ | ✅ |
| **F003** | C | buy **หรือ** get แยก index | ❌ | **ง่าย** | ✅ | ❌ | ✅ |

**Normalize เต็ม:** ถ้า `%/100 < 1` ใช้ค่าเดิม; ถ้า `>100` หาร 100; ถ้า `<1` คูณ 100  
**Normalize ง่าย:** `field22 = % / 100` ตรงๆ

**ลำดับส่วนลด (ทุก profile):** `promoPrice` → `fieldP` · `discountAmt` → `fieldR` · `discountPct` → `field22` (อันแรกที่มี)

**Out of scope:** P100, P103, F002

---

## ของร่วม (อย่า copy ซ้ำในแต่ละ profile)

ทำครั้งเดียวใน shared แล้วทุก profile ใช้:

- Header: วันที่ `DD.MM.YYYY`, vendor ว่าง→`NOBP`, rebate ตัดอักษร, contractType เฉพาะ `Z2…`
- แถว: `field2` = MAT / MGPNew, `field4` = material หรือ group name
- Mechanic → `buyQty` / `getQty`
- ส่วนลด `pickDiscount` ตามลำดับด้านบน
- `stores[]`, `card/tender/…` = เว้น `[]` ได้

รายละเอียด: `MerC_to_AB_Phase1_Shared.md` + หมวด 2 ใน `MerC_to_AB_Function_Split.md`

ชื่อ field ปลายทางล็อกกับ `payload_AB_Reference.md` — อย่าตั้งชื่อใหม่
