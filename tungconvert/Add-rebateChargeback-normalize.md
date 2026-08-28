# Handoff — C→AB convert ครบ (shared + D003/F003)

| | |
|--|--|
| **อัปเดต** | 2026-08-28 |
| **สถานะ BBY** | 9 profile หลักทำแล้ว (D001/P011 เทสผ่าน) |
| **งานค้าง** | งาน 1–6 ด้านล่าง — ทำครบ = ส่งงาน C→AB ได้ (9 profile) |

---

## Checklist งานที่ต้อง implement

### Shared (ทุก profile — เขียนครั้งเดียว)

- [ ] **1. แก้ `normalizeRebate`** — Reason `X` + Z200 → `HEADER.rebateChargeback` = `"X"` ไม่ใช่ `""`
- [ ] **2. Implement `Fill_Contract`** — gen `CONDITIONS[]` จาก HEADER + แถว `MATERIALS` (ไม่พึ่ง copy Condition block อย่างเดียว)
- [ ] **3. Map Compensate** — `comp_qty_in_sap` / `comp_set` / `comp_%` → `cond_type_condition_rate` (+ condition type ตาม macro)

### Profile-specific / flow แยก

- [ ] **4. D003 Coupon merge** — mechanic `A (Coupon)+B (B)` → ใส่ `get` เข้า **ก้อน BONUSBUYS ก่อนหน้า** (จาก `A (Coupon)+B (A)`) ไม่สร้างก้อนใหม่
- [ ] **5. F003 (B) merge (ถ้าต้อง parity macro)** — mechanic ลงท้าย `(B)` → merge `get` เข้าก้อน `(A)` ก่อนหน้า (logic คล้าย D003 แต่ mechanic คนละชุด)
- [ ] **6. FOC / MEK1** — เมื่อ `cost_foc` + `cost_date_*` มีค่าใน MATERIALS → สร้าง output FOC แยก (ไม่ใช่ BONUSBUYS · ทุก profile ใช้ module เดียว)

### เทสรวมหลังทำครบ

- [ ] **7. เทสซ้ำ** — D001 + Reason X + Compensate (งาน 1–3) · D003 Coupon (งาน 4) · FOC (งาน 6)

**ไม่ต้องทำ (macro ไม่ส่ง AB):** `pack_size`, `disc_deal`, `forecast_*`, `gp_*`, `disp_mprice_*` → `MATERIALS: []` ใน AB ถูกแล้ว

**Out of scope ลูกค้า:** P100, P103, F002

---

## แยก profile ไหม?

| งาน | แยก 9 profile? | ทำที่ไหน |
|-----|----------------|----------|
| `normalizeRebate` | ❌ shared | `mapHeaderShared()` |
| `Fill_Contract` / Compensate | ❌ shared | `mapConditionsFromMerC()` |
| Coupon merge | ⚠️ **D003** (+ F003 ถ้างาน 5) | helper `mergeGetIntoPreviousBonusBuy()` เรียกจาก `buildD003` / `buildF003` |
| FOC / MEK1 | ❌ shared (trigger จาก MATERIALS) | module แยกจาก BBY JSON |
| BONUSBUYS | ✅ แยก profile | ทำแล้ว |

---

## โครง AB JSON เมื่อครบทุกงาน

```json
{
  "LAYOUT": "AB",
  "status": "...",
  "HEADER": {
    "rebateChargeback": "X",
    "contractType": "Z200",
    "...": "..."
  },
  "BONUSBUYS": [ "... ตาม profile ..." ],
  "CONDITIONS": [ "... gen จาก Mer C — งาน 2–3 ..." ],
  "MATERIALS": []
}
```

FOC/MEK1 (งาน 6) = **output แยก** ไม่ใส่ในก้อน `BONUSBUYS` ข้างบน

---

## Done = อะไร?

| หลังทำครบ checklist | ได้ไหม |
|----------------------|--------|
| C→AB 9 profile (BBY + Contract) ส่งลูกค้า | ✅ |
| macro clone 100% ทุก flow | ⚠️ ยกเว้น P100/P103/F002 |
| ทุกช่อง Mer C ไป AB | ❌ forecast/pack/gp ไม่ไป (macro ก็ไม่ไป) |

---

# งาน 1 — `rebateChargeback` หายหลัง convert

## อาการ

ตอนเลือก **Reason = X** + **Contract Type = Z200** แล้ว convert:

| ฟิลด์ AB | Expected | Actual (เว็บตอนนี้) |
|----------|----------|---------------------|
| `HEADER.contractType` | `"Z200"` | `"Z200"` ✅ |
| `HEADER.rebateChargeback` | `"X"` | `""` ❌ |
| `CONDITIONS[0].conditionHeader.cond_hdr_reason` | `"X"` | `"X"` ✅ |

`BONUSBUYS` ไม่พัง — ปัญหาอยู่ shared header เท่านั้น

### C (ก่อนแปลง)

```json
"HEADER": {
  "contractType": "Z200",
  "rebateChargeback": "X"
}
```

### AB (หลังแปลง — ผิด)

```json
"HEADER": {
  "rebateChargeback": "",
  "contractType": "Z200"
}
```

---

## Dropdown Reason ครบใน Mer C (คอลัมน์ BQ)

| รหัสนำ | ข้อความเต็มใน dropdown |
|--------|-------------------------|
| **A** | `A - สร้าง Contract หลังจบรายการ Promotion/Combine BBY/Fix Amount` |
| **X** | `X - สร้าง Contract พร้อม BBY` |
| **1** | `1 - Merchandise/Sectional Marketing/Outright (รับผิดชอบค่าใช้จ่าย)` |
| **2** | `2 - ราคาทุนพิเศษ/ส่วนลดทุน/ส่วนลด DEMO` |
| **3** | `3 - GWP/Premium/PO FOC/Free Item/BBY DC For Planning ไม่มีการเรียกเก็บ` |

สูตร Mer C (BQ) มัก auto:
- Contract Type = `Z301…` → Reason **A**
- Contract Type ขึ้นต้น `Z2` → Reason **X**

---

## กติกาที่ถูก (macro `set_RebateReason_n_ContractType`)

อ่านจาก `Mer-C_Convert_To_STD.txt` (~บรรทัด 400) — ใช้ **`Left(1)`** ของ Reason:

```
ถ้า Contract Type ขึ้นต้น "Z2":
  contractType     = ค่าเต็ม (เช่น Z200)
  rebateChargeback = ตัวอักษรแรกของ Reason → "A" | "X" | "1" | "2" | "3"

ถ้า Contract Type ไม่ใช่ Z2:
  contractType = ""
  ถ้าตัวแรกของ Reason ≠ "A":
    rebateChargeback = "0" + ตัวอักษรแรก → "0X" | "01" | "02" | "03"
  ไม่งั้น:
    rebateChargeback = "A"
```

**สาเหตุบั๊ก:** โค้ด `normalizeRebate` น่าจะ `slice(1)` / ตัดตัวแรกทิ้ง → `"X"` เหลือ `""`

**Fix แนวทาง:**

- ว่าง → ว่าง
- `code = Left(1)` ของ reason (รองรับข้อความยาวและรหัสสั้น)
- Z2 → `rebateChargeback = code`
- ไม่ใช่ Z2 และ `code !== "A"` → `"0" + code`
- ไม่ใช่ Z2 และ `code === "A"` → `"A"`
- **ห้าม** `substring(1)` บนสตริงที่เหลือรหัสตัวเดียว

### ตารางเทส Reason

| Reason (รหัสนำ) | Contract Type | `rebateChargeback` ที่ควรได้ |
|-----------------|---------------|------------------------------|
| `X…` | `Z200` / `Z2…` | `"X"` |
| `A…` | `Z2…` | `"A"` |
| `1…` | `Z2…` | `"1"` |
| `2…` | `Z2…` | `"2"` |
| `3…` | `Z2…` | `"3"` |
| `A…` | ว่าง / ไม่ใช่ Z2 | `"A"` |
| `X…` | ว่าง / ไม่ใช่ Z2 | `"0X"` |
| `1…` | ว่าง / ไม่ใช่ Z2 | `"01"` |
| `2…` | ว่าง / ไม่ใช่ Z2 | `"02"` |
| `3…` | ว่าง / ไม่ใช่ Z2 | `"03"` |

---

# งาน 2 — `Fill_Contract` gen `CONDITIONS[]`

## สถานะตอนนี้ vs ที่ต้องเป็น

| | ตอนนี้ (เว็บ) | ต้องเป็น (ตาม macro) |
|--|---------------|----------------------|
| `CONDITIONS` | **copy ตรง** จาก Condition block ที่กรอกบนฟอร์ม | **gen** จาก HEADER + แถว `MATERIALS` |
| Compensate ใน MATERIALS | **ไม่อ่an** | ไป `conditionType[].cond_type_condition_rate` |
| Charge Back col 69–77 | ไม่อ่an (ถ้ากรอกแค่ในแถวสินค้า) | ไป `conditionHeader`, `businessVolume*`, `conditionType` |
| ลูกค้ากรอก | 2 ที่ (แถวสินค้า + Condition block) | แถว Mer C (+ HEADER) พอ |

Macro: `Fill_Contract` ใน `Mer-C_Convert_To_STD.txt` (~บรรทัด 4625)

---

## แหล่งข้อมูล C ที่ต้องอ่an

### จาก HEADER

| C field | → AB |
|---------|-----|
| `rebateChargeback` / Reason | `conditionHeader.cond_hdr_reason` |
| `contractType` | `conditionHeader.cond_hdr_contract_type` |
| `periodFrom` / `periodTo` | `cond_hdr_start` / `cond_hdr_end` (format `DD.MM.YYYY`) |
| `promotionName`, `purchasingGroup` | `cond_hdr_header_text` (ตาม macro) |
| `vendorCode`, `vendorName` | `cond_hdr_vendor` / `cond_hdr_vendor_name` |

### จาก MATERIALS[] (แถวสินค้า)

| Mer C col | C JSON key | → AB |
|-----------|------------|-----|
| 59 | `compensate_quantity_in_sap` / `comp_qty_in_sap` | `cond_type_condition_rate` (ลำดับ 1) |
| 60 | `compensate_baht_per_set` / `comp_set` | `cond_type_condition_rate` (ลำดับ 2) |
| 61 | `compensate_percent` / `comp_f3` | rate แบบ % + condition type `ZR01` |
| 69 | Charge Back / Reason | reason (ซ้ำ HEADER ได้) |
| 70 | Contract Type | contract type |
| 76 | Condition Table | `cond_type_condition_table` |
| 77 | Field Combination | `bv_*_field_combination` |
| 72–75 | Settlement, Payment… | `conditionHeader` ที่เกี่ยว |
| `material`, `sales_unit` | | `conditionType` material / unit |
| `vendor` | | `bv_sales_vendor` ฯลฯ |

---

## กติกา Compensate → `condition_rate` (จาก macro)

ลำดับเลือก rate (~บรรทัด 4908–4931):

```
1. มี ฿/Qty In SAP (col 59) และ Contract Type ≠ Z222
   → Condition_Rate = ค่า Qty
   → condition type: "ZR05 - Charge Amount(PeQty)" (หรือ "1" ใน payload สั้น)

2. ไม่มี Qty แต่มี ฿/Set (col 60)
   → Condition_Rate = ค่า Set

3. มี Compensate % (col 61)
   → condition type: "ZR01 - Charge Back %"
   → rate = % (macro มี normalize % เพิ่ม)
```

**หมายเหตุ:** ถ้ามีทั้ง Qty และ Set — macro ใช้ **Qty ก่อน**

---

## เคสเทสจริงที่พบ (D001 + Contract X + Compensate)

### C — MATERIALS

```json
"comp_qty_in_sap": 8,
"comp_set": 16,
"bonus_buy_profile": "D001",
"mechanic": "2For"
```

### C — CONDITIONS (กรอก Condition block เอง — workaround)

```json
"conditionType": [{
  "cond_type_condition_table": "V 163",
  "cond_type_condition_type": "1",
  "cond_type_condition_rate": 1,
  "cond_type_unit_2": "EA",
  "cond_type_contract_no": "1"
}]
```

### AB ตอนนี้

- `CONDITIONS` = copy ตรงจาก Condition block → `condition_rate: 1` ✅ (copy)
- **`comp_qty_in_sap: 8` / `comp_set: 16` ไม่ถูก map** ❌

### AB หลัง implement Fill_Contract (expected)

```json
"conditionType": [{
  "cond_type_condition_table": "V 163",
  "cond_type_condition_type": "1",
  "cond_type_condition_rate": 8,
  "cond_type_unit_2": "EA",
  "cond_type_contract_no": "1"
}]
```

(ใช้ **8** จาก Qty เพราะ macro เลือก Qty ก่อน Set)

`BONUSBUYS` เคสนี้ถูกแล้ว — ไม่ต้องแก้:

- `buy.field9: 2`, `get.getQuantity: 2`
- `buy.field8: "99"`, `get.fieldP: "100"`
- ไม่มี `get.field8`

---

## พฤติกรรมหลัง implement (สรุป)

1. ลูกค้ากรอก Compensate + Charge Back ที่**แถวสินค้า** (และ/หรือ HEADER)
2. Convert gen `CONDITIONS[]` อัตโนมัติ
3. ถ้ามี Condition block กรอกไว้แล้ว — ตกลงทีมว่า **gen ทับ** หรือ **merge** (แนะนำ: gen จาก Mer C เป็นหลัก ตาม macro)
4. ไม่มี contract (ไม่มี Z2 / ไม่มี compensate) → `CONDITIONS: []`

---

## วิธีเทสหลังแก้ครบ (งาน 1 + 2)

### เทส rebateChargeback

1. HEADER: Reason = X · Contract Type = Z200
2. Convert (profile ใดก็ได้)
3. เช็ค:
   - [ ] `HEADER.rebateChargeback` = `"X"`
   - [ ] `HEADER.contractType` = `"Z200"`
   - [ ] `BONUSBUYS` ไม่พัง

### เทส Fill_Contract + Compensate

1. HEADER: Reason X · Z200
2. MATERIALS: `comp_qty_in_sap: 8` · `comp_set: 16` · ไม่กรอก Condition block (หรือกรอกแล้วดูว่า gen ทับถูกไหม)
3. Convert D001 + 2For
4. เช็ค:
   - [ ] `CONDITIONS[0].conditionType[0].cond_type_condition_rate` = **8** (ไม่ใช่ 1 หรือ 16)
   - [ ] `cond_hdr_reason` = `"X"` · `cond_hdr_contract_type` = `"Z200"`
   - [ ] วันที่ valid from/to ตรง period
   - [ ] `BONUSBUYS` ยังถูก

### เทส Compensate แบบ Set อย่างเดียว

1. `comp_qty_in_sap` ว่าง · `comp_set: 16`
2. Expected rate = **16**

### เทส Compensate แบบ %

1. `compensate_percent` มีค่า · Qty/Set ว่าง
2. Expected condition type แบบ % (ZR01) + rate ตาม macro normalize

---

## ไฟล์อ้างอิง

| ไฟล์ | หัวข้อ |
|------|--------|
| `Mer-C_Convert_To_STD.txt` | `set_RebateReason_n_ContractType` (~400) · `Fill_Contract` (~4625) · Compensate (~4908) · `Gen_D003` Coupon (~4090) · `Gen_F003` (~4282) |
| `Mer-C_Field_Mapping_Guide.md` | §G PromoTag+Compensate · §H Chargeback block |
| `MerC_to_AB_ExistsInAB_Mapping.md` | §2.3 Compensate / Charge Back col 59–77 |
| `payload_AB_Reference.md` | §4 CONDITIONS[] dictionary |
| `payload-keys.ts` | key map `comp_qty_in_sap`, `cond_type_*` |
| `profile/MerC_to_AB_D003.md` | Coupon merge · แยก (A)/(B) |
| `profile/MerC_to_AB_*.md` §8 | CONDITIONS — gen ตาม macro |

---

## สิ่งที่ผ่านแล้ว — ไม่ต้องแก้

- โครง `BONUSBUYS` ตาม 9 profile (buy/get, qty, ส่วนลด P/R/%)
- `MATERIALS: []` ฝั่ง AB
- copy `CONDITIONS` ทำงาน — ใช้เป็น fallback จนกว่า Fill_Contract จะเสร็จ
- forecast / pack_size / disc_deal / gp — **ไม่ส่ง AB** (macro ไม่ map)

---

# งาน 4 — D003 Coupon merge (`+B(B)` → ก้อนก่อนหน้า)

## สถานะตอนนี้ vs ที่ต้องเป็น

| | ตอนนี้ (เว็บ) | ต้องเป็น (macro `Gen_D003`) |
|--|---------------|----------------------------|
| แถว `A (Coupon)+B (A)` | 1 ก้อน BONUSBUYS (buy-only) | 1 ก้อน (buy-only) ✅ |
| แถว `A (Coupon)+B (B)` | **1 ก้อนใหม่** (get-only) | **merge `get` เข้าก้อนก่อนหน้า** ❌ |

Macro: `Mer-C_Convert_To_STD.txt` ~4090–4127 ใน `Gen_D003`

```
Mechanic = "A (Coupon)+B (A)"  →  เขียน buy ที่ก้อนปัจจุบัน
Mechanic = "A (Coupon)+B (B)"  →  เขียน get ที่ Bonus_Buy_Row_Index - 1 (ก้อนก่อนหน้า)
                                 ไม่สร้าง BONUSBUYS element ใหม่
```

## Implement แนะนำ

```ts
// ใน buildD003 / route หลัง map MATERIALS → BONUSBUYS
function buildD003Rows(materials) {
  const out = []
  for (const row of materials) {
    if (isCouponB(row.mechanic)) {
      mergeGetIntoPreviousBonusBuy(out, buildGetOnly(row))
      continue
    }
    out.push(buildSplitElement(row)) // (A) buy-only / (B) get-only ตามเดิม
  }
  return out
}
```

**ไม่ต้อง copy 9 profile** — เรียกจาก `buildD003` เท่านั้น  
Profile อื่น (D001, P011…) **ไม่มี** Coupon merge

---

## เคสเทส D003 Coupon

### C — 2 แถว MATERIALS

| แถว | mechanic | โครง |
|-----|----------|------|
| 1 | `A (Coupon)+B (A)` | buy-only |
| 2 | `A (Coupon)+B (B)` | get → **merge** |

### AB expected — **1 ก้อน** BONUSBUYS

```json
"BONUSBUYS": [{
  "bonusBuyHeader": { "mechanic": "A (Coupon)+B (A)", "bonusBuyProfile": "D003" },
  "buy": [{ "field4": "...", "field9": 1 }],
  "get": [{ "field4": "...", "getQuantity": 1, "fieldP": "999" }]
}]
```

**ผิด:** ได้ 2 ก้อน (ก้อน 2 มีแค่ get ว่าง buy)

---

# งาน 5 — F003 merge `(B)` (optional parity)

F003 ใช้ mechanic คนละชุด (`1A Get 1B (A)` / `(B)`) แต่ macro **merge get เข้า index ก่อนหน้า** เหมือนกัน (`Gen_F003` ~4307 — เขียน get ที่ `Get_Item_Index` ไม่ใช่ row ใหม่)

| | ตอนนี้ | macro |
|--|--------|-------|
| แยก (A)/(B) คนละก้อน | ✅ (รับได้) | merge (B) เข้า (A) |

ถ้าลูกค้าต้องการ parity macro F003 — reuse helper เดียวกับงาน 4 ใน `buildF003`

---

# งาน 6 — FOC / MEK1

## ไม่ใช่ BONUSBUYS · ไม่แยก profile

Trigger เมื่อ MATERIALS มี:

| C field | Mer C col | หมายเหตุ |
|---------|-----------|----------|
| `cost_foc` | 46 | มีค่า เช่น `"FOC"` |
| `cost_date_start` / `cost_date_end` | 47–48 | วันมีผล |

Mer C note: **`"Blank" = Not Create FOC & MEK1`** — ว่าง = ไม่สร้าง

## สิ่งที่ต้องทำ

1. หลัง convert BBY — scan `MATERIALS[]` ว่ามี FOC row ไหม
2. ถ้ามี → เรียก **`createFocMek1Output()`** (shared module)
3. Output **แยกจาก** payload AB หลัก (หรือ section แยกตาม backend กำหนด)

**ทุก profile** (D001, P011, …) ใช้ logic เดียวกัน — ขึ้นกับแถวสินค้า ไม่ใช่ `bonusBuyProfile`

## เทส FOC

1. MATERIALS: `cost_foc: "FOC"` · `cost_date_start` / `cost_date_end` มีค่า
2. Convert (profile ใดก็ได้)
3. เช็ค:
   - [ ] มี FOC/MEK1 output ตาม backend spec
   - [ ] `BONUSBUYS` / `CONDITIONS` ไม่พัง
4. MATERIALS: `cost_foc` ว่าง → **ไม่สร้าง** FOC

---

## วิธีเทสรวมหลังทำครบ checklist 1–6

| # | เทส | เช็ค |
|---|-----|------|
| 1 | HEADER Reason X + Z200 | `rebateChargeback` = `"X"` |
| 2 | D001 + comp_qty 8 / comp_set 16 | `condition_rate` = **8** |
| 3 | D003 Coupon 2 แถว (A)+(B) | **1 ก้อน** buy+get |
| 4 | F003 (B) merge (ถ้าทำงาน 5) | 1 ก้อน buy+get |
| 5 | MATERIALS + cost_foc | FOC output แยก |
| 6 | 9 profile smoke (promo 1 เคส) | BBY โครงถูก |
