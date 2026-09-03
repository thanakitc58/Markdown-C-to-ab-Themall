# Dev Handoff — ของที่เหลือ (ไฟล์เดียว)

| | |
|--|--|
| **อัปเดต** | 2026-09-03 |
| **สำหรับ** | Dev แก้ PR เดียว · QA เทสตามนี้ |
| **Macro** | `Mer-C_Convert_To_STD.txt` |
| **เอกสารอื่น** | อ้างอิงเสริมเท่านั้น — **ยึดไฟล์นี้เป็นหลัก** |

---

## ข้อความสั้นส่ง Dev (copy ได้)

> อ่านไฟล์นี้ทั้งไฟล์ — สรุปงานค้าง + logic + JSON ตัวอย่างครบในที่เดียว
>
> **อย่าพัง:** group `(promo, mat-group)` · `bonusBuyNumber` · `promotionArea` P1/P4 · CONDITIONS ไม่ตัดก้อน · Compensate แถว vendor ไม่ซ้ำ
>
> **แก้รอบนี้ 3 งาน:**
> 1. Compensate vendor ซ้ำ → rate ต้องตามแถว (P0 ยืนยันบั๊ก)
> 2. D003 Coupon `(B)` merge เข้า `(A)` (P1)
> 3. FOC/MEK1 จาก `cost_foc` (P1)

---

## สถานะรวม

| # | หัวข้อ | สถานะ |
|---|--------|--------|
| — | Group + `bonusBuyNumber` + mat-group | ✅ ผ่าน |
| — | `promotionArea` = `P1`/`P4` | ✅ ผ่าน |
| — | CONDITIONS ไม่หาย (3→3) | ✅ ผ่าน |
| — | Compensate (vendor ไม่ซ้ำ) | ✅ ผ่าน |
| — | Multiple · `HEADER.rebateChargeback` ว่าง | ✅ โอเค (ไม่บังคับกรอก) |
| **1** | Compensate **vendor ซ้ำ** คนละ rate | ❌ ยืนยันบั๊ก |
| **2** | D003 Coupon merge | ❌ ยังไม่ทำ |
| **3** | FOC / MEK1 | ❌ ยังไม่เริ่ม |

**ลำดับแก้:** 1 → 2 → 3  

**ไม่ทำรอบนี้:** F003 merge · dedupe stores

---

# สิ่งที่ผ่านแล้ว (อย่าพัง) — logic สั้นๆ

## Group + BBY

**Group key:** `(noof_promotion, noof_material_grouping)`  
(+ profile/mechanic ถ้า mixed)

| สถานการณ์ | ผล |
|-----------|-----|
| promo ต่าง | คนละ BBY |
| promo เดียว · mat-group ต่าง | คนละ BBY |
| promo เดียว · mat-group เดียวกัน | **1 BBY** |
| ลูกค้าใส่ `noof_bonus_buy` = `1,1,1` แต่ group ต่าง | ยังได้เลข `"1"`,`"2"` หลัง group |

```json
// C MATERIALS (ย่อ)
[
  { "noof_promotion": 1, "noof_material_grouping": 1, "noof_bonus_buy": 1 },
  { "noof_promotion": 2, "noof_material_grouping": 2, "noof_bonus_buy": 1 },
  { "noof_promotion": 2, "noof_material_grouping": 2, "noof_bonus_buy": 1 }
]

// AB BONUSBUYS ที่ถูก — 2 ก้อน ไม่ใช่ 3
[
  { "bonusBuyNumber": "1" },
  { "bonusBuyNumber": "2" }
]
```

## promotionArea

- อยู่ที่ `BONUSBUYS[].bonusBuyHeader.promotionArea`
- ค่าถูก: `"P1"` หรือ `"P4"` (จาก plant / macro)
- **ห้าม** map จาก `disp_mprice_category` (เช่น `"Skincare"`)

## rebateChargeback (Multiple)

- `HEADER.rebateChargeback` **ว่างได้** — reason อยู่ที่ `CONDITIONS[].conditionHeader.cond_hdr_reason`
- ถ้า C ส่ง `"X"` มาใน HEADER → AB ห้ามทำลายเป็น `""`
- **ไม่บังคับ** set HEADER จาก CONDITIONS

---

# งาน 1 — Compensate vendor ซ้ำ → `condition_rate` (P0)

## คืออะไร / อยู่ที่ไหน

| ฝั่ง | Field |
|------|--------|
| C กรอก | `MATERIALS[].comp_qty_in_sap` / `comp_set` / `comp_f3` |
| AB ใช้ | `CONDITIONS[].conditionType[].cond_type_condition_rate` |
| หน้าที่ | อัตรา chargeback ต่อสัญญา — **ไม่ใช่** ช่องใน BONUSBUYS |

## Logic (macro `Fill_Contract` ~4903+)

1. อ่าน Compensate **ทีละแถว** MATERIALS  
2. เลือก rate ตามลำดับ:
   1. `comp_qty_in_sap` (col 59) ถ้ามีและใช้ได้
   2. ไม่งั้น `comp_set` (col 60)
   3. ไม่งั้น `comp_f3` (% · col 61)
3. จับคู่ CONDITION ↔ MATERIAL ด้วย **ลำดับ occurrence ของ vendor** หรือ `lineNumber`  
   **ห้าม** `findFirst(vendor)` แล้วใช้ rate แถวแรกทุกครั้ง

JSON keys: `comp_qty_in_sap` · `comp_set` · `comp_f3` (ไม่ใช่ `comp_%`)

> อย่าเทียบ rate กับ `CONDITIONS` ใน C ที่ลูกค้าอาจใส่คนละค่า — source คือ MATERIALS Compensate

## INPUT C (ย่อ — เคสที่ยืนยันบั๊ก)

```json
{
  "LAYOUT": "C",
  "HEADER": {
    "singleMultiple": "Multiple",
    "rebateChargeback": "",
    "bonusBuyProfile": ""
  },
  "MATERIALS": [
    {
      "lineNumber": 0,
      "noof_promotion": 1,
      "vendor": "BTG07",
      "bonus_buy_profile": "D001",
      "mechanic": "2For",
      "comp_qty_in_sap": 1,
      "comp_set": 2
    },
    {
      "lineNumber": 1,
      "noof_promotion": 2,
      "noof_material_grouping": 2,
      "vendor": "MSH02",
      "bonus_buy_profile": "D001",
      "mechanic": "2For",
      "comp_qty_in_sap": 0.5,
      "comp_set": 1
    },
    {
      "lineNumber": 2,
      "noof_promotion": 2,
      "noof_material_grouping": 2,
      "vendor": "MSH02",
      "bonus_buy_profile": "D001",
      "mechanic": "2For",
      "comp_qty_in_sap": 1,
      "comp_set": 2
    }
  ],
  "CONDITIONS": [
    { "lineNumber": 0, "conditionHeader": { "cond_hdr_vendor": "BTG07" } },
    { "lineNumber": 1, "conditionHeader": { "cond_hdr_vendor": "MSH02" } },
    { "lineNumber": 2, "conditionHeader": { "cond_hdr_vendor": "MSH02" } }
  ]
}
```

## OUTPUT ที่ผิด ❌ (build ปัจจุบัน)

```json
{
  "LAYOUT": "AB",
  "CONDITIONS": [
    {
      "lineNumber": 0,
      "conditionHeader": { "cond_hdr_vendor": "BTG07" },
      "conditionType": [{ "cond_type_condition_rate": 1 }]
    },
    {
      "lineNumber": 1,
      "conditionHeader": { "cond_hdr_vendor": "MSH02" },
      "conditionType": [{ "cond_type_condition_rate": 0.5 }]
    },
    {
      "lineNumber": 2,
      "conditionHeader": { "cond_hdr_vendor": "MSH02" },
      "conditionType": [{ "cond_type_condition_rate": 0.5 }]
    }
  ]
}
```

แถว 2 ควรเป็น `1` แต่ได้ `0.5` (ดึงค่า occurrence แรกของ MSH02)

## OUTPUT ที่ถูก ✅

```json
{
  "LAYOUT": "AB",
  "CONDITIONS": [
    {
      "lineNumber": 0,
      "conditionHeader": { "cond_hdr_vendor": "BTG07" },
      "conditionType": [{ "cond_type_condition_rate": 1 }]
    },
    {
      "lineNumber": 1,
      "conditionHeader": { "cond_hdr_vendor": "MSH02" },
      "conditionType": [{ "cond_type_condition_rate": 0.5 }]
    },
    {
      "lineNumber": 2,
      "conditionHeader": { "cond_hdr_vendor": "MSH02" },
      "conditionType": [{ "cond_type_condition_rate": 1 }]
    }
  ]
}
```

| CONDITIONS | vendor | rate |
|------------|--------|------|
| 0 | BTG07 | **1** |
| 1 | MSH02 | **0.5** |
| 2 | MSH02 | **1** |

## Acceptance

- [ ] เคสด้านบนได้ `1` · `0.5` · `1`
- [ ] vendor คนละตัวยังถูก
- [ ] CONDITIONS จำนวนก้อนไม่หด
- [ ] Group / BBY เดิมไม่พัง

---

# งาน 2 — D003 Coupon merge (P1)

## คืออะไร

D003 แยกก้อนตามท้าย mechanic `(A)` / `(B)`  
เคส **Coupon** ตาม macro: แถว `(B)` **ไม่สร้างก้อนใหม่** — เอา `get` ไปรวมก้อน `(A)` ก่อนหน้า

Mechanic จริงจากตาราง Master:

| Mechanic | บทบาท |
|----------|--------|
| `A (Coupon)+B (A)` | buy |
| `A (Coupon)+B (B)` | get → **merge** |

## Logic

```
แถว (A) → สร้าง BONUSBUY[n] มี buy[] · get: []
แถว (B) Coupon → เอา get[] ใส่ BONUSBUY[n] เดิม · ไม่ push ก้อนใหม่
ผล: 2 แถว C → 1 ก้อน AB มีทั้ง buy + get
```

**ผิด:** 2 ก้อน (buy-only + get-only)  
**ถูก:** 1 ก้อน (buy + get)

## INPUT C (ย่อ)

```json
{
  "LAYOUT": "C",
  "HEADER": {
    "bonusBuyProfile": "D003",
    "singleMultiple": "Multiple"
  },
  "MATERIALS": [
    {
      "lineNumber": 0,
      "noof_promotion": 1,
      "material": "1000728050",
      "bonus_buy_profile": "D003",
      "mechanic": "A (Coupon)+B (A)",
      "sales_price_normal": 490,
      "sales_price_promo": 555,
      "stores": ["13KA", "14KA"]
    },
    {
      "lineNumber": 1,
      "noof_promotion": 1,
      "material": "1000728050",
      "bonus_buy_profile": "D003",
      "mechanic": "A (Coupon)+B (B)",
      "sales_price_promo": 555,
      "stores": ["13KA", "14KA"]
    }
  ]
}
```

## OUTPUT ที่ผิด ❌

```json
{
  "LAYOUT": "AB",
  "BONUSBUYS": [
    {
      "bonusBuyNumber": "1",
      "bonusBuyHeader": {
        "bonusBuyProfile": "D003",
        "mechanic": "A (Coupon)+B (A)"
      },
      "buy": [{ "field4": "1000728050", "field9": 1 }],
      "get": []
    },
    {
      "bonusBuyNumber": "2",
      "bonusBuyHeader": {
        "bonusBuyProfile": "D003",
        "mechanic": "A (Coupon)+B (B)"
      },
      "buy": [],
      "get": [{ "field4": "1000728050", "getQuantity": 1, "fieldP": "555" }]
    }
  ]
}
```

## OUTPUT ที่ถูก ✅

```json
{
  "LAYOUT": "AB",
  "BONUSBUYS": [
    {
      "bonusBuyNumber": "1",
      "bonusBuyHeader": {
        "bonusBuyNumber": "1",
        "bonusBuyProfile": "D003",
        "mechanic": "A (Coupon)+B (A)",
        "promotionArea": "P1"
      },
      "buy": [
        {
          "bonusBuyNumber": "1",
          "field2": "Material",
          "field4": "1000728050",
          "field9": 1
        }
      ],
      "get": [
        {
          "bonusBuyNumber": "1",
          "field2": "Material",
          "field4": "1000728050",
          "getQuantity": 1,
          "fieldP": "555"
        }
      ],
      "stores": ["13KA", "14KA"]
    }
  ]
}
```

`BONUSBUYS.length === 1` · มีทั้ง `buy` และ `get`

## Acceptance

- [ ] 2 แถว Coupon A+B → **1** ก้อน
- [ ] ไม่เหลือก้อน get-only แยก
- [ ] group / เลข BBY โปรไฟล์อื่นไม่พัง

อ้างอิงเสริม: `profile/MerC_to_AB_D003.md` · macro `Gen_D003`

---

# งาน 3 — FOC / MEK1 (P1)

## คืออะไร

| | |
|--|--|
| ความหมาย | Free of Charge (ต้นทุน FOC) |
| กรอกที่ | MATERIALS: `cost_foc` (col 46) + `cost_date_start` / `cost_date_end` |
| Output | **แยกจาก** `BONUSBUYS` (section / ไฟล์ MEK1 ตาม backend) |
| ว่าง | **ไม่สร้าง** FOC |

## Logic

```
ถ้า MATERIALS[i].cost_foc มีค่า (เช่น "FOC"):
  → สร้าง FOC/MEK1 record จากแถวนั้น + วันเริ่ม/จบ
ไม่งั้น:
  → ข้าม ไม่สร้าง

ห้าม: ทำให้ BONUSBUYS หรือ CONDITIONS หาย/พัง
```

> Schema ชื่อ key ฝั่ง AB ของ FOC ให้ยึด backend / macro module ที่มีอยู่ — ถ้ายังไม่มี module ให้สร้างแยก ไม่ยัดเข้า `BONUSBUYS[]`

## INPUT C — มี FOC

```json
{
  "LAYOUT": "C",
  "HEADER": {
    "singleMultiple": "Multiple",
    "bonusBuyProfile": ""
  },
  "MATERIALS": [
    {
      "lineNumber": 0,
      "noof_promotion": 1,
      "material": "1000728050",
      "vendor": "BTG07",
      "bonus_buy_profile": "D001",
      "mechanic": "2For",
      "sales_price_promo": 555,
      "cost_foc": "FOC",
      "cost_date_start": "2027-08-10",
      "cost_date_end": "2027-08-28",
      "stores": ["13KA", "14KA"]
    }
  ]
}
```

## INPUT C — ไม่มี FOC (ไม่ต้องสร้าง)

```json
{
  "material": "1000728050",
  "bonus_buy_profile": "D001",
  "mechanic": "2For"
  // ไม่มี cost_foc หรือว่าง
}
```

## OUTPUT ที่ถูก ✅ (แนวทาง)

```json
{
  "LAYOUT": "AB",
  "BONUSBUYS": [
    {
      "bonusBuyNumber": "1",
      "bonusBuyHeader": {
        "bonusBuyProfile": "D001",
        "mechanic": "2For",
        "promotionArea": "P1"
      },
      "buy": [{ "field4": "1000728050" }],
      "get": [{ "field4": "1000728050", "fieldP": "555" }]
    }
  ],
  "CONDITIONS": [],
  "FOC": [
    {
      "material": "1000728050",
      "vendor": "BTG07",
      "cost_foc": "FOC",
      "validFrom": "2027-08-10",
      "validTo": "2027-08-28"
    }
  ]
}
```

หมายเหตุ:

- ชื่อ key `FOC` / โครงภายในให้ **ตรง schema backend จริง** (อาจเป็น `MEK1` หรือชื่ออื่น) — สำคัญคือ **แยกจาก BONUSBUYS** และ trigger จาก `cost_foc`
- ถ้า `cost_foc` ว่าง → **ไม่มี** array/section FOC (หรือเป็น `[]`)

## OUTPUT ที่ผิด ❌

- มี `cost_foc` แต่ไม่มี FOC output เลย  
- หรือยัด FOC เข้า `BONUSBUYS` จน mechanic/profile เพี้ยน  
- หรือ CONDITIONS/BBY หายหลังมี FOC

## Acceptance

- [ ] มี `cost_foc` → มี FOC/MEK1 output แยก
- [ ] ไม่มี `cost_foc` → ไม่สร้าง
- [ ] BONUSBUYS / CONDITIONS ยังครบและถูก

---

# Checklist รวมก่อน merge

### งาน 1 Compensate
- [ ] MSH02 `0.5` แล้ว `1` → AB rate `0.5` แล้ว `1`
- [ ] CONDITIONS ยัง 3 ก้อน

### งาน 2 D003
- [ ] Coupon A+B → `BONUSBUYS.length === 1` มี buy+get

### งาน 3 FOC
- [ ] มี/ไม่มี `cost_foc` ตามกติกา
- [ ] ไม่พัง BBY

### Regression (ต้องยังผ่าน)
- [ ] promo+mat-group → จำนวนก้อนถูก · เลข `"1"`,`"2"`
- [ ] `promotionArea` เป็น `P1`/`P4` ไม่ใช่ Skincare
- [ ] D001 buy/get / `fieldP` / stores / vendor `NOBP` ปกติ

---

## QA เทสหลัง Dev เสร็จ

1. ใบ Multiple เดิม · MSH02 qty คนละค่า → ดู `cond_type_condition_rate`  
2. ใบ D003 2 แถว Coupon → นับ `BONUSBUYS` = 1  
3. ใบ D001 + กรอก FOC → มี output แยก · BBY ยังอยู่
