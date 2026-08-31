# Dev Handoff — Convert จาก MATERIALS (งานค้าง)

| | |
|--|--|
| **อัปเดต** | 2026-08-31 |
| **สำหรับ** | Dev implement convert C → AB |
| **เอกสารอื่น** | [`Add-rebateChargeback-normalize.md`](Add-rebateChargeback-normalize.md) · [`Dev-handoff-Multiple-CONDITIONS.md`](Dev-handoff-Multiple-CONDITIONS.md) |

---

## สรุป 1 ประโยค

> **Convert อ่านจากสิ่งที่ user กรอกในแถวสินค้า (`MATERIALS[]`) + Condition block (`CONDITIONS[]`) — ไม่ยึด HEADER Mer C เป็นแหล่งข้อมูลหลัก**  
> HEADER = หัวเอกสาร (ชื่อโปร · วันที่ · WBS · group) · copy ไป AB ได้ · **ห้ามใช้ trigger / route / gen contract**

---

## หลักการ — HEADER Mer C ทำอะไร / ไม่ทำอะไร

| HEADER field | หน้าที่ | ใช้ convert ไหม |
|--------------|---------|-----------------|
| Group, Theme, Promotion Name, WBS, Period, Time, Days, purchasingGroup | หัวเอกสาร · copy ไป AB HEADER | ✅ copy เท่านั้น |
| `singleMultiple` | บอกโหมด Single/Multiple | ✅ อ่anได้ |
| `bonusBuyProfile` | ไม่บังคับใน Multiple | ❌ **ห้าม trigger BBY** |
| `rebateChargeback` | ไม่บังคับใน Multiple | ❌ **ห้าม gen CONDITIONS** |
| `contractType` | ไม่บังคับใน Multiple | ❌ **ห้าม gen CONDITIONS** |
| `vendorCode` | ว่าง → AB `NOBP` (copy rule) | ❌ **ห้ามทับ vendor ใน CONDITIONS / แถวสินค้า** |

**แหล่งข้อมูล convert จริง:**

```
MATERIALS[]  →  BONUSBUYS[]     (profile, mechanic, ราคา, ร้าน, compensate, charge back)
MATERIALS[]  →  CONDITIONS[]    (หลัง #2–3 — Fill_Contract)
CONDITIONS[] →  CONDITIONS[]    (ก่อน #2 — passthrough ถ้ากรอกมือ)
MATERIALS[]  →  FOC/MEK1        (งาน #6 — output แยก)
```

---

## ลำดับ implement (สั่งทำตามนี้)

| ลำดับ | งาน | สถานะ QA ล่าสุด | ทำที่ |
|-------|-----|-----------------|-------|
| **A** | Mat-group grouping (col 28) | ❌ 1 แถว ≈ 1 BBY | convert pipeline |
| **B** | #9 `bonusBuyNumber` ← `noof_bonus_buy` | ❌ ซ้ำ `"1"` ทุกก้อน | `buildGroup*` / shared |
| **C** | #2 `Fill_Contract` — เรียก `mapConditionsFromMerC()` | ❌ มี function ยังไม่เรียก | shared |
| **D** | #3 Map Compensate (col 59–61 → rate) | ❌ ผูกกับ C | ใน `mapConditionsFromMerC()` |
| **E** | #6 FOC / MEK1 | ❌ ยังไม่เริ่ม | module แยก |

---

## งาน A — Mat-group grouping (col 28)

### ปัญหาตอนนี้

- Loop แถว → **1 แถว = 1 BONUSBUY** เสมอ
- ไม่รวมแถวที่ **`noof_material_grouping`** เดียวกัน ภายใน **`noof_promotion`** เดียวกัน

### กติกา (ตาม macro)

| Col | JSON key | ความหมาย |
|-----|----------|----------|
| 27 | `noof_promotion` | ชุดโปร — 1 promo มีหลายแถวได้ |
| 28 | `noof_material_grouping` | กลุ่มสินค้าใน promo — **mat group เดียวกัน → รวม 1 BBY** |
| 29 | `noof_bonus_buy` | เลขก้อน BBY → AB `bonusBuyNumber` |

```
Group key แนะนำ:
  (noof_promotion, noof_material_grouping, bonus_buy_profile, mechanic?)
→ 1 ก้อน BONUSBUYS ต่อ group
→ buy[] / get[] รวมหลาย material ตาม macro (ถ้า profile รองรับ multi-line)
```

### ตัวอย่าง HHC (~14 แถว → ~4 BBY)

| Promo | แถว | Mat group | BBY ที่ควรได้ |
|-------|-----|-----------|---------------|
| 1 | 5 | 1 | 1 ก้อน |
| 2 | 3–4 | 2 | 1 ก้อน |
| 3 | 4 | 3 | 1 ก้อน |
| 4 | 1 | 4 | 1 ก้อน |

### เทสจาก QA (2026-08-31)

**INPUT:** promo 1 (1 แถว) + promo 2 (2 แถว) · D001 2For · HEADER ว่าง

| ตอนนี้ | ควรได้ (หลัง grouping) |
|--------|------------------------|
| 3 BBY แยก | **2 BBY** (promo 2 รวม 2 แถว ถ้า mat group เดียวกัน) |

### Checklist dev

- [ ] Group แถวด้วย `(noof_promotion, noof_material_grouping)` ก่อน build BBY
- [ ] ไม่ hardcode 1 row = 1 BBY
- [ ] หลัง group แล้ว route `buildD001` / `buildP011` ตาม `bonus_buy_profile`
- [ ] Mixed profile ในใบเดียวยังทำงาน (group แยก profile)

---

## งาน B — #9 `bonusBuyNumber`

### ปัญหาตอนนี้

ทุกก้อนได้ `bonusBuyNumber: "1"` แม้ `noof_promotion` ต่างกัน

### กติกา map

| จาก C | → AB | ห้าม |
|-------|------|------|
| **`noof_bonus_buy`** / `numberOfBonusBuy` | **`bonusBuyNumber`** | — |
| (ไม่มีค่า) | default `"1"` | — |
| `noof_promotion` | — | **ไม่ map** |
| `bonusBuyProfile` | — | **ไม่ map** |

```ts
function resolveBonusBuyNumber(row) {
  return String(row.noof_bonus_buy ?? row.numberOfBonusBuy ?? 1)
}
```

ใส่ให้ตรงกัน: `BONUSBUYS[i].bonusBuyNumber` · `bonusBuyHeader` · `buy[0]` · `get[0]`

### Checklist dev

- [ ] อ่an `noof_bonus_buy` จากแถว (หรือจาก group leader หลัง mat-group)
- [ ] ไม่ default `"1"` ทุกก้อน
- [ ] ไม่ใช้ `noof_promotion` แทน

---

## งาน C + D — Fill_Contract (#2) + Compensate (#3)

### ปัญหาตอนนี้

- `mapConditionsFromMerC()` **มีใน code แต่ยังไม่ถูกเรียก**
- ตอนนี้ `CONDITIONS` = **passthrough** จาก Condition block ที่กรอกมือ
- มี bug ครึ่งๆ กลางๆ: `comp_set: 0` บางครั้งทับ rate (ก่อน passthrough ชัด)

### หลัง implement — พฤติกรรมที่ต้องการ

1. **เรียก `mapConditionsFromMerC(header, materials)`** ตอน convert
2. **Gen `CONDITIONS[]` ทั้งก้อน** จากแถว MATERIALS (+ charge back cols 69–77)
3. **ไม่ merge ครึ่งๆ** — gen ทั้งก้อน หรือ passthrough ทั้งก้อน · ห้ามทับบาง field

### แหล่งข้อมูล (จากแถวสินค้า — ไม่ใช่ HEADER)

| Mer C col | JSON key | → AB |
|-----------|----------|------|
| 59 | `comp_qty_in_sap` | `cond_type_condition_rate` (ลำดับ 1) |
| 60 | `comp_set` | `cond_type_condition_rate` (ลำดับ 2) |
| 61 | `comp_%` / `comp_f3` | rate % + type ZR01 |
| 69 | Charge Back / Reason | `cond_hdr_reason` |
| 70 | Contract Type | `cond_hdr_contract_type` |
| 72–77 | Settlement, table, field combo… | `conditionHeader`, `businessVolume*`, `conditionType` |
| แถว | `vendor` | `bv_sales_vendor`, `cond_hdr_vendor` |

Macro อ้างอิง: `Fill_Contract` ~บรรทัด 4625 · Compensate ~4908 ใน `Mer-C_Convert_To_STD.txt`

### กติกาเลือก Compensate rate

```
1. มี comp_qty_in_sap และ Contract Type ≠ Z222 → rate = Qty
2. ไม่มี Qty แต่มี comp_set → rate = Set
3. มี Compensate % → type ZR01 · rate = %
```

**Qty มาก่อน Set** ถ้ามีทั้งคู่

### เทส QA หลังเรียก function แล้ว

| เคส MATERIALS | Expected AB rate |
|---------------|------------------|
| `comp_qty_in_sap: 8`, `comp_set: 16` | **8** |
| `comp_qty_in_sap: 0.5`, `comp_set: 1` | **0.5** |
| `comp_set: 0` อย่างเดียว | **0** |
| ไม่มี compensate + ไม่มี Z2 | `CONDITIONS: []` หรือตาม macro |

**หมายเหตุ QA:** ก่อน C+D เสร็จ — เทส passthrough จาก Condition block มือ · **หลัง C+D เสร็จ** — เทส gen จากแถว (ไม่ต้องกรอก Condition block)

### Checklist dev

- [ ] Wire `mapConditionsFromMerC()` ใน convert pipeline
- [ ] Gen CONDITIONS จาก MATERIALS rows
- [ ] Map compensate col 59–61 → rate ตามลำดับ macro
- [ ] Vendor contract จาก **`MATERIALS.vendor`** — ไม่จาก HEADER
- [ ] หยุด logic ที่ `comp_set` ทับ rate แบบ merge ครึ่งๆ

---

## งาน E — FOC / MEK1 (#6)

### Trigger (จากแถวสินค้า)

| C field | Col | เงื่อนไข |
|---------|-----|----------|
| `cost_foc` | 46 | มีค่า เช่น `"FOC"` |
| `cost_date_start` / `cost_date_end` | 47–48 | วันที่มีผล |

Mer C: **`Blank` = ไม่สร้าง FOC & MEK1**

### สิ่งที่ต้องทำ

1. หลัง convert BBY — scan `MATERIALS[]`
2. แถวไหนมี FOC → เรียก **`createFocMek1Output()`** (shared · ทุก profile)
3. Output **แยกจาก** `BONUSBUYS` / `CONDITIONS` หลัก

### Checklist dev

- [ ] Scan `cost_foc` + `cost_date_*` ใน MATERIALS
- [ ] สร้าง FOC output แยก
- [ ] `cost_foc` ว่าง → ไม่สร้าง
- [ ] BBY / CONDITIONS ไม่พัง

---

## สิ่งที่ผ่านแล้ว (ไม่ต้องแก้)

- BBY 9 profile — โครง buy/get, qty, field8/fieldP/R/% ✅
- Per-row router — HEADER profile ว่าง + แถวมี profile → BBY ออก ✅ (2026-08-31)
- CONDITIONS passthrough — เคสกรอก Condition block 3 ก้อน copy ตรง ✅
- `MATERIALS: []` ฝั่ง AB ✅

---

## QA จะเทสอะไรหลังแต่ละงาน

### หลัง A + B (mat-group + bonusBuyNumber)

1. promo 1 (1 แถว) + promo 2 (2 แถว, mat group เดียว) → **2 BBY** · number `"1"`, `"2"`
2. HHC ~14 แถว / 4 mat group → **~4 BBY**
3. BBY buy/get/qty/fieldP ยังถูก profile

### หลัง C + D (Fill_Contract + Compensate)

1. แถว `comp_qty_in_sap: 8` → rate **8**
2. Vendor MSH02 ในแถว → CONDITIONS vendor **MSH02**
3. ไม่กรอก Condition block → CONDITIONS gen จากแถวได้

### หลัง E (FOC)

1. แถว `cost_foc: "FOC"` + วันที่ → มี FOC output
2. ว่าง → ไม่สร้าง

---

## ไฟล์อ้างอิง

| ไฟล์ | ใช้ทำอะไร |
|------|-----------|
| [`mechanic-qty-lookup.json`](mechanic-qty-lookup.json) | mechanic → buyQty/getQty |
| [`profile/MerC_to_AB_*.md`](../profile/) | Spec buy/get ต่อ profile |
| [`Mer-C_Convert_To_STD.txt`](../Mer-C_Convert_To_STD.txt) | Macro Fill_Contract · grouping |
| [`Add-rebateChargeback-normalize.md`](Add-rebateChargeback-normalize.md) | Checklist 1–10 เต็ม |
