# Dev Handoff — สิ่งที่ยังผิด (พร้อมตัวอย่าง JSON)

|                    |                                                                                                          |
| ------------------ | -------------------------------------------------------------------------------------------------------- |
| **อัปเดต**         | 2026-09-03 (รอบ 2 — หลังแก้ 3b)                                                                          |
| **สำหรับ**         | Dev แก้ต่อจาก A+B                                                                                        |
| **ยึดหลัก**        | Macro `Mer-C_Convert_To_STD.txt` เป็นหลัก · To-Be แยก Manual/Auto                                        |
| **โค้ด**           | `constants/promotion-payload.js` → `convertLayoutCToAbc`                                                 |
| **แผนที่ฟังก์ชัน** | [`convertLayoutCToAbc-functions.md`](convertLayoutCToAbc-functions.md)                                   |

---

## ข้อความสั้นส่ง Dev

> **อัปเดต 2026-09-03 รอบ 2**
>
> **ผ่านแล้ว — อย่าพัง:** A+B · CONDITIONS N→N · `rebateChargeback` · `promotionArea` P1/P4 · Compensate (รวม **3b vendor ซ้ำ**)
>
> **ค้าง:**
>
> 1. **D003 Coupon merge** — โค้ดมีแล้ว (`mergeGetIntoPreviousBonusBuy`) · **ยังไม่ QA จริง**
> 2. **FOC / MEK1** — **ข้ามก่อน** · รอ backend spec จากพี่โป้ง (ดูหัวข้อ 6)
>
> **รู้แล้ว:** Multiple ไม่บังคับ `HEADER.rebateChargeback` · CONDITIONS หาย / Skincare promotionArea = แก้แล้ว

---

## สถานะรวม (หลังแก้ 2026-09-03 รอบ 2)

| #   | หัวข้อ                               | สถานะ                                      |
| --- | ------------------------------------ | ------------------------------------------ |
| A+B | Group + `bonusBuyNumber` + mat-group | ✅ ผ่าน                                    |
| 1   | CONDITIONS หาย 3→1                   | ✅ ผ่าน                                    |
| 2   | `HEADER.rebateChargeback`            | ✅ ว่างได้ (Multiple) · เทส `"X"` optional |
| 3a  | Compensate → rate (vendor ไม่ซ้ำ)    | ✅ ผ่าน                                    |
| 3b  | Compensate vendor ซ้ำ คนละ rate      | ✅ แก้แล้ว (`vendorIndex`)                 |
| 4   | `promotionArea`                      | ✅ `P1` (ไม่ใช่ Skincare)                  |
| 5   | D003 Coupon merge                    | ⚠️ **โค้ดมี · ค้าง QA**                    |
| 6   | FOC / MEK1                           | ⏸️ **ข้ามก่อน · รอ spec backend**          |

**เหลือทำ:** 5 (QA) → 6 (หลังได้ spec)

---

# 0) A+B — ผ่านแล้ว (อย่าพัง)

## กติกา

- Group: `(noof_promotion, noof_material_grouping)` · ไม่มี mat-group → แยกตาม promo
- `bonusBuyNumber` = `"1"`, `"2"`, … หลัง group

## ตัวอย่าง C (ย่อ) — INPUT ที่ใช้เทส

```json
{
  "LAYOUT": "C",
  "HEADER": {
    "theme": "C011",
    "promotionName": "M Price 08/2027",
    "bonusBuyProfile": "",
    "rebateChargeback": "",
    "contractType": "",
    "singleMultiple": "Multiple",
    "vendorCode": "",
    "periodFrom": "2027-08-10",
    "periodTo": "2026-08-28",
    "timeFrom": "00:00:00",
    "timeTo": "00:00:00",
    "days": ["Sun"]
  },
  "MATERIALS": [
    {
      "lineNumber": 0,
      "noof_promotion": 1,
      "material": "1000728050",
      "vendor": "BTG07",
      "sales_price_normal": 490,
      "sales_price_promo": 500,
      "bonus_buy_profile": "D001",
      "mechanic": "2For",
      "disp_mprice_category": "Skincare",
      "stores": ["13KA", "14KA"]
    },
    {
      "lineNumber": 1,
      "noof_promotion": 2,
      "material": "1000728050",
      "vendor": "MSH02",
      "sales_price_normal": 490,
      "sales_price_promo": 500,
      "bonus_buy_profile": "D001",
      "mechanic": "2For",
      "comp_qty_in_sap": 0.5,
      "comp_set": 1,
      "disp_mprice_category": "Skincare",
      "stores": ["13KA", "14KA"]
    },
    {
      "lineNumber": 2,
      "noof_promotion": 2,
      "material": "1000728050",
      "vendor": "MSH02",
      "sales_price_normal": 490,
      "sales_price_promo": 500,
      "bonus_buy_profile": "D001",
      "mechanic": "2For",
      "comp_qty_in_sap": 0.5,
      "comp_set": 1,
      "disp_mprice_category": "Skincare",
      "stores": ["13KA", "14KA"]
    }
  ]
}
```

## Expected BBY (ผ่านแล้ว)

|           | Expected                  |
| --------- | ------------------------- |
| จำนวนก้อน | **2**                     |
| เลข       | **`"1"`**, **`"2"`**      |
| ก้อน 1    | promo 1 · 1 แถว           |
| ก้อน 2    | promo 2 · **merge 2 แถว** |

## AB ที่ถูกฝั่ง BBY (ย่อ) ✅

```json
{
  "LAYOUT": "AB",
  "BONUSBUYS": [
    {
      "lineNumber": 0,
      "bonusBuyNumber": "1",
      "bonusBuyHeader": {
        "bonusBuyNumber": "1",
        "bonusBuyProfile": "D001",
        "mechanic": "2For",
        "validTimeFrom": "00:00:00",
        "validTimeTo": "00:00:00",
        "wbsNumber": "AP.27.8883.10.CP.01"
      },
      "buy": [
        {
          "bonusBuyNumber": "1",
          "field2": "Material",
          "field4": "1000728050",
          "field9": 2,
          "field8": "490",
          "salesUnit": "EA"
        }
      ],
      "get": [
        {
          "bonusBuyNumber": "1",
          "field2": "Material",
          "field4": "1000728050",
          "getQuantity": 2,
          "fieldP": "500",
          "unit": "EA"
        }
      ]
    },
    {
      "lineNumber": 1,
      "bonusBuyNumber": "2",
      "bonusBuyHeader": {
        "bonusBuyNumber": "2",
        "bonusBuyProfile": "D001",
        "mechanic": "2For",
        "validTimeFrom": "00:00:00",
        "validTimeTo": "00:00:00",
        "wbsNumber": "AP.27.8883.10.CP.01"
      },
      "buy": [
        {
          "bonusBuyNumber": "2",
          "field2": "Material",
          "field4": "1000728050",
          "field9": 2,
          "field8": "490",
          "salesUnit": "EA"
        }
      ],
      "get": [
        {
          "bonusBuyNumber": "2",
          "field2": "Material",
          "field4": "1000728050",
          "getQuantity": 2,
          "fieldP": "500",
          "unit": "EA"
        }
      ]
    }
  ],
  "MATERIALS": []
}
```

---

# 1) CONDITIONS หาย 3 → 1 ✅ (ผ่านแล้ว)

## อาการ

|           | C                                        | AB ที่ได้ (ผิด) |
| --------- | ---------------------------------------- | --------------- |
| จำนวนก้อน | **3**                                    | **1**           |
| ที่หาย    | ก้อน reason `X` + Z200 (vendor MSH02) ×2 | หายหมด          |

## C CONDITIONS (ย่อ) — มีครบ 3

```json
{
  "CONDITIONS": [
    {
      "lineNumber": 0,
      "conditionHeader": {
        "cond_hdr_reason": "1",
        "cond_hdr_vendor": "BTG07",
        "cond_hdr_vendor_name": "บริษัท เบทาโกรเกษตรอุตสาหกรรม",
        "cond_hdr_payment_method": "A",
        "cond_hdr_sales_organization": "2013"
      },
      "conditionType": [
        { "cond_type_condition_rate": 1, "cond_type_unit_2": "EA" }
      ],
      "contractNumber": "1"
    },
    {
      "lineNumber": 1,
      "conditionHeader": {
        "cond_hdr_reason": "X",
        "cond_hdr_contract_type": "Z200",
        "cond_hdr_vendor": "MSH02",
        "cond_hdr_payment_method": "M",
        "cond_hdr_sales_organization": "2013",
        "cond_hdr_settlement_option": "2"
      },
      "conditionType": [
        { "cond_type_condition_rate": 2, "cond_type_unit_2": "EA" }
      ],
      "contractNumber": "1"
    },
    {
      "lineNumber": 2,
      "conditionHeader": {
        "cond_hdr_reason": "X",
        "cond_hdr_contract_type": "Z200",
        "cond_hdr_vendor": "MSH02",
        "cond_hdr_payment_method": "M",
        "cond_hdr_sales_organization": "2009",
        "cond_hdr_settlement_option": "2"
      },
      "conditionType": [
        { "cond_type_condition_rate": 1, "cond_type_unit_2": "EA" }
      ],
      "contractNumber": "1"
    }
  ]
}
```

## AB ที่ผิด ❌

```json
{
  "CONDITIONS": [
    {
      "lineNumber": 0,
      "conditionHeader": {
        "cond_hdr_reason": "1",
        "cond_hdr_vendor": "BTG07",
        "cond_hdr_payment_method": "A",
        "cond_hdr_sales_organization": "2013"
      },
      "conditionType": [{ "cond_type_condition_rate": 1 }],
      "contractNumber": "1"
    }
  ]
}
```

## AB ที่ต้องการ (ช่วงนี้ — passthrough) ✅

จนกว่า Fill_Contract จะเสร็จสมบูรณ์: **copy ครบ 3 ก้อน** จาก C อย่าตัด

```json
{
  "CONDITIONS": [
    {
      "lineNumber": 0,
      "conditionHeader": { "cond_hdr_reason": "1", "cond_hdr_vendor": "BTG07" }
    },
    {
      "lineNumber": 1,
      "conditionHeader": {
        "cond_hdr_reason": "X",
        "cond_hdr_contract_type": "Z200",
        "cond_hdr_vendor": "MSH02"
      }
    },
    {
      "lineNumber": 2,
      "conditionHeader": {
        "cond_hdr_reason": "X",
        "cond_hdr_contract_type": "Z200",
        "cond_hdr_vendor": "MSH02"
      }
    }
  ]
}
```

**Checklist**

- [x] C มี N ก้อน → AB ได้ N ก้อน (passthrough) — `mapConditionsFromMerC`
- [x] ห้ามเหลือแค่ก้อนแรก

---

# 2) `HEADER.rebateChargeback` ✅ (ผ่านแล้ว)

## คืออะไร

- **ช่องเดียวทั้งใบ** ใน `HEADER` — ไม่แยกต่อ promo / ต่อ vendor
- ค่า = รหัส Reason ฝั่ง chargeback เช่น `"X"`, `"A"`, `"1"`
- Reason ละเอียดต่อสัญญาอยู่ที่ `CONDITIONS[].conditionHeader.cond_hdr_reason`

## กติกาที่ต้องทำ

| สถานการณ์                                         | พฤติกรรม                                                                |
| ------------------------------------------------- | ----------------------------------------------------------------------- |
| C `HEADER.rebateChargeback` = `"X"`               | AB ต้องเป็น **`"X"`** ห้ามเป็น `""`                                     |
| C HEADER ว่าง · CONDITIONS มีทั้ง `"1"` และ `"X"` | **ไม่บังคับ** ให้ HEADER = `"X"` อัตโนมัติ · เก็บ reason ที่ CONDITIONS |
| Contract `Z2…` + Reason `X`                       | ตาม macro: `rebateChargeback = "X"` เมื่อมีค่าจากแหล่งที่ถูกต้อง        |

## ตัวอย่างที่ผิด ❌

```json
{
  "HEADER": {
    "rebateChargeback": "",
    "contractType": ""
  }
}
```

(เมื่อ C ส่ง `"X"` มาแล้วถูก normalize หาย — เคสงาน 1 เดิม)

## ตัวอย่างที่ถูก ✅ (เมื่อ C กรอก X + Z200)

```json
{
  "HEADER": {
    "rebateChargeback": "X",
    "contractType": "Z200"
  }
}
```

## เคสเทสล่าสุด (HEADER ว่างใน C)

C:

```json
"HEADER": { "rebateChargeback": "", "contractType": "" }
```

→ AB HEADER ว่างได้  
แต่ **CONDITIONS ที่มี reason X ต้องไม่หาย** (ดูหัวข้อ 1)

**Checklist**

- [x] อย่า `slice`/`substring` จน `"X"` กลายเป็น `""` — `mapRebateChargeback`
- [x] อย่าบังคับ HEADER = `"X"` แค่เพราะมี CONDITIONS บางก้อนเป็น X
- [x] รักษา CONDITIONS reason ให้ครบ

---

# 3) Compensate → `condition_rate` ✅ (ผ่านแล้ว · รวม 3b)

## อาการ

MATERIALS promo 2 มี:

```json
"comp_qty_in_sap": 0.5,
"comp_set": 1
```

แต่ CONDITIONS ใน AB (ที่เหลือ) ยังเป็น rate จากมือ / ไม่สะท้อน compensate

## กติกา (macro)

ลำดับอ่าน rate:

1. `comp_qty_in_sap` (col 59) ถ้ามีและใช้ได้ → rate
2. ไม่งั้น `comp_set` (col 60)
3. ไม่งั้น `comp_f3` (% · col 61)

JSON keys: `comp_qty_in_sap` · `comp_set` · `comp_f3` (ไม่ใช่ `comp_%`)

## ตัวอย่างที่ผิด ❌

```json
{
  "conditionType": [{ "cond_type_condition_rate": 1 }]
}
```

(ทั้งที่แถวมี `comp_qty_in_sap: 0.5`)

## ตัวอย่างที่ต้องการ ✅ (หลัง Fill_Contract + map compensate)

เมื่อ gen จากแถวที่มี `comp_qty_in_sap: 0.5`:

```json
{
  "conditionType": [{ "cond_type_condition_rate": 0.5 }]
}
```

(เคสเทสเก่า handoff: qty 8 / set 16 → rate **8**)

## 3b) Vendor ซ้ำ คนละ rate ✅ (แก้ 2026-09-03)

**อาการเดิม:** MSH02 2 ก้อน CONDITIONS — แถว material `comp_qty_in_sap: 0.5` แล้ว `1` → AB ได้ rate `0.5`, `0.5` ทั้งคู่

**สาเหตุ:** `applyCompensateRateToNode` ใช้ `.find()` → เอาแถวแรกของ vendor เสมอ

**แก้:** ส่ง `vendorIndex` (ลำดับ occurrence ต่อ vendor) · ใช้ `.filter()` แล้วเลือก `[vendorIndex]`

| ก้อน CONDITIONS | Vendor | vendorIndex | rate ที่ได้ |
| --------------- | ------ | ----------- | ----------- |
| 0 (reason=1)    | BTG07  | 0           | จากแถว BTG07 |
| 1 (reason=X)    | MSH02  | 0           | **0.5**     |
| 2 (reason=X)    | MSH02  | 1           | **1**       |

**Checklist**

- [x] เรียก `mapConditionsFromMerC()` / Fill_Contract ตามแผน
- [x] อ่าน compensate จาก MATERIALS
- [x] อย่าทำให้ CONDITIONS หาย (หัวข้อ 1)
- [x] vendor ซ้ำ → แยก rate ตามลำดับก้อน (3b)

---

# 4) `promotionArea` ได้ `"Skincare"` ✅ (ผ่านแล้ว)

## ที่อยู่

```
BONUSBUYS[].bonusBuyHeader.promotionArea
```

## คืออะไร / ควรเป็นอะไร

|                   |                                                             |
| ----------------- | ----------------------------------------------------------- |
| ความหมาย          | **พื้นที่โปร** (Promotion Area)                             |
| ค่าที่ถูก (macro) | **`P1`** (ร้านปกติ) หรือ **`P4`** (ออนไลน์ · มี plant 22KA) |
| ค่าที่ผิด         | `"Skincare"` จาก `disp_mprice_category`                     |
| Mer C กรอก?       | **ไม่บังคับ** (To-Be AB = Formula/Auto ได้)                 |
| Macro             | คำนวณเอง เขียน cell AA — **ไม่อ่านหมวดสินค้า**              |

> Handoff เดิมที่บอก map จาก `disp_mprice_category` = **ผิด — ยกเลิก**

## ตัวอย่างที่ผิด ❌

```json
{
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "D001",
    "mechanic": "2For",
    "promotionArea": "Skincare"
  }
}
```

## ตัวอย่างที่ต้องการ ✅

**ทางเลือก A — ตาม macro (แนะนำ):**

```json
{
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "D001",
    "mechanic": "2For",
    "promotionArea": "P1"
  }
}
```

ถ้าเป็นออนไลน์ (plant มี 22KA):

```json
"promotionArea": "P4"
```

(เมื่อ `P4` ถึงจะมี `onlineDescriptionEnglish` / `Thai`)

**ทางเลือก B — ยังไม่พร้อมคำนวณ plant:**

- **อย่าใส่** `promotionArea` ไปก่อน หรือ
- ใส่เฉพาะเมื่อ C มี field `promotionArea` จริงๆ (`P1`/`P4`)
- **ห้าม** copy `disp_mprice_category`

**Checklist**

- [x] ลบ/เลิก map จาก `disp_mprice_category` — `resolvePromotionArea`
- [x] ใส่ `P1`/`P4` ตาม macro หรือ omit จนกว่าทำถูก

---

# 5) D003 Coupon merge ⚠️ (โค้ดมี · ค้าง QA)

## อาการที่คาด

| แถว mechanic       | ผิด               | ถูก                            |
| ------------------ | ----------------- | ------------------------------ |
| `A (Coupon)+B (A)` | 1 ก้อน            | 1 ก้อน buy                     |
| `A (Coupon)+B (B)` | ก้อนใหม่ get-only | **merge get เข้าก้อนก่อนหน้า** |

## Expected

```json
{
  "BONUSBUYS": [
    {
      "bonusBuyNumber": "1",
      "buy": [{ "...": "จากแถว (A)" }],
      "get": [{ "...": "จากแถว (B) — merge เข้ามา" }]
    }
  ]
}
```

ไม่ใช่ 2 ก้อนแยก

## สถานะโค้ด

| ส่วน | ฟังก์ชัน | สถานะ |
| ---- | -------- | ----- |
| อ่าน side (A)/(B) | `parseMechanicSideD003` | ✅ |
| ตรวจแถว merge | `isD003CouponMergeGetRow` | ✅ |
| merge get เข้าก้อนก่อน | `convertMaterialsToBonusBuys` → `mergedGetLines` | ✅ |
| build get จาก merged | `buildGroupBonusBuyNode` case `D003` | ✅ |

**เทรซ logic (กรณีปกติ):** แถว `(A)` สร้าง group → แถว `(B)` push เข้า `mergedGetLines` → 1 ก้อน buy+get ✅

**Edge case ที่ยังไม่เทส:** แถว `(B)` มาก่อน `(A)` → `groups.length === 0` → แถว B อาจหาย (Mer C ปกติกรอก A ก่อน B)

**Checklist**

- [x] merge `(B)` → ก้อน `(A)` ก่อนหน้า — ในโค้ดแล้ว
- [ ] **QA จริง** ด้วย fixture / Submit Verification
- [ ] ยืนยันกรณี `(A)` อย่างเดียว (get ว่าง) ยังถูก

---

# 6) FOC / MEK1 ⏸️ (ข้ามก่อน · รอ spec backend)

## คืออะไร

- **FOC** = Free of Charge
- ช่องกรอก Mer C: **Cost Price … FOC** (`cost_foc` · col 46) + วัน `cost_date_start` / `cost_date_end`
- ว่าง = **ไม่สร้าง**
- Output **แยกจาก** `BONUSBUYS`

## ตัวอย่าง C ที่ควร trigger

```json
{
  "MATERIALS": [
    {
      "cost_foc": "FOC",
      "cost_date_start": "2027-08-10",
      "cost_date_end": "2027-08-28",
      "bonus_buy_profile": "D001",
      "mechanic": "2For"
    }
  ]
}
```

## Expected (ที่รู้แล้ว)

- Trigger: `cost_foc` มีค่า (เช่น `"FOC"`) + `cost_date_start` / `cost_date_end`
- `cost_foc` ว่าง → **ไม่สร้าง**
- Output **แยกจาก** `BONUSBUYS` · `BONUSBUYS` / `CONDITIONS` ต้องไม่พัง
- ทุก profile ใช้ logic เดียว — ขึ้นกับแถว MATERIALS ไม่ใช่ `bonusBuyProfile`

## ขาดข้อมูล — รอพี่โป้ง (ยังเขียนโค้ดไม่ได้)

| # | คำถาม |
| - | ----- |
| 1 | Output อยู่ key ไหนใน payload? (`FOC` / `MEK1` / อื่น?) |
| 2 | Field ใน output มีอะไรบ้าง? — ขอตัวอย่าง JSON 1 ก้อน expected |
| 3 | แถว MATERIALS ที่มี `cost_foc` → 1 output ต่อแถว หรือ merge? |
| 4 | แยกไฟล์ หรืออยู่ payload เดียวกับ `BONUSBUYS` / `CONDITIONS`? |

**ข้อความส่งพี่โป้ง (copy ได้เลย):**

> พี่โป้งครับ ตอนนี้ทำ convert Layout C → AB ถึงข้อ 6 (FOC/MEK1) แล้ว  
> รู้แล้วว่า trigger จาก `cost_foc` มีค่า + `cost_date_start` / `cost_date_end` และว่าง = ไม่สร้าง  
> แต่ยังขาด **format JSON output ฝั่ง backend** ครับ ขอถามว่า:  
> 1) Output อยู่ key ไหน (`FOC` / `MEK1` / อื่น?)  
> 2) Field ใน output มีอะไรบ้าง — ช่วยตัวอย่าง JSON 1 ก้อนได้เลย  
> 3) แต่ละแถว MATERIALS ที่มี `cost_foc` สร้าง 1 ก้อน หรือ merge?  
> 4) แยกไฟล์ หรืออยู่ payload เดียวกับ BONUSBUYS/CONDITIONS?  
> ขอบคุณครับ 🙏

**Checklist**

- [x] รู้ trigger + input fields
- [ ] ได้ backend spec / ตัวอย่าง JSON
- [ ] เขียน `createFocMek1Output()` ใน `convertLayoutCToAbc`
- [ ] เทส: มี `cost_foc` → มี output · ว่าง → ไม่สร้าง · BBY/CONDITIONS ไม่พัง

---

# Macro vs To-Be (สั้นๆ)

|                 | คนกรอก Mer C  | Convert/macro ใส่ AB                   |
| --------------- | ------------- | -------------------------------------- |
| Manual ใน To-Be | กรอกถ้าใช้    | copy/calc                              |
| Formula / Auto  | **ไม่บังคับ** | ระบบใส่ (เช่น `promotionArea` = P1/P4) |
| Optional        | ไม่บังคับ     | มีก็ได้                                |

**ยึด macro เป็นหลัก** สำหรับครบ/ไม่ครบของ JSON convert

### ที่ macro เขียน → JSON ควรมี (D001)

`bonusBuyNumber` · profile · mechanic · validTime\* · **promotionArea P1/P4** · wbs · buy field2/4/9 · get field2/4/qty/unit · PหรือRหรือ% · stores · HEADER · CONDITIONS

### ที่ macro ไม่เขียน แต่มีใน JSON ได้

`buy.field8` · `buy.description` · `buy.ean` · `buy.salesUnit`

### ที่ไม่ต้องมีใน convert

`description` / `department` / `priceTag` / flags ใน header ก้อน (Auto บนจอ AB)

---

# Checklist รวมส่งเพื่อน

- [x] **0** รักษา A+B ที่ผ่านแล้ว
- [x] **1** CONDITIONS: C N ก้อน → AB N ก้อน
- [x] **2** `rebateChargeback`: อย่าทำลาย `"X"` · ไม่บังคับ set จากทุก CONDITIONS
- [x] **3** Compensate → `cond_type_condition_rate` (รวม 3b vendor ซ้ำ)
- [x] **4** `promotionArea` = `P1`/`P4` · ห้าม `Skincare`
- [ ] **5** D003 merge — โค้ดมี · **QA จริง**
- [ ] **6** FOC/MEK1 — **รอ spec backend** แล้วค่อยทำ

---

## ไฟล์อ้างอิง

| ไฟล์                                 | ใช้                                                           |
| ------------------------------------ | ------------------------------------------------------------- |
| `Mer-C_Convert_To_STD.txt`           | macro As-Is                                                   |
| `profile/MerC_to_AB_D001.md` §5, §15 | To-Be + audit keys                                            |
| `Add-rebateChargeback-normalize.md`  | งาน 1–3, 6                                                    |
| `Dev-handoff-Remaining-Work.md`      | checklist เดิม (ข้อ promotionArea จาก disp_mprice **ยกเลิก**) |
