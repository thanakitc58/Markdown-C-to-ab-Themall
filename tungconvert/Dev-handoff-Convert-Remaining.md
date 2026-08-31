# Dev Handoff — Convert จาก MATERIALS (งานค้าง)

| | |
|--|--|
| **อัปเดต** | 2026-08-31 (v3 — mat-group ครบ + ตารางชื่อ field) |
| **สำหรับ** | Dev implement convert C → AB |
| **เอกสารอื่น** | [`Add-rebateChargeback-normalize.md`](Add-rebateChargeback-normalize.md) · [`Dev-handoff-Multiple-CONDITIONS.md`](Dev-handoff-Multiple-CONDITIONS.md) |
| **อ่านไฟล์นี้ก่อน** | งานค้าง A→E · HEADER ไม่ trigger · เทสจริงจาก QA |

---

## ข้อความสำหรับ Dev (copy ส่งได้)

> งานค้าง convert อยู่ในไฟล์นี้ — **อ่านก่อน implement**
>
> **HEADER Mer C = หัวเอกสารเท่านั้น** · convert จาก **`MATERIALS[]` per-row** (+ grouping) · ห้าม trigger BBY/contract จาก HEADER
>
> **ลำดับทำ:**  
> **A** Mat-group grouping (col 28) — ตอนนี้ 1 แถว = 1 BBY ❌  
> **B** `bonusBuyNumber` ← `noof_bonus_buy` (col 29) — ตอนนี้ซ้ำ `"1"` ทุกก้อน ❌  
> **C+D** เรียก `mapConditionsFromMerC()` + map Compensate col 59–61  
> **E** FOC/MEK1 ทีหลัง
>
> **เคส QA ล่าสุด (กรอก mat-group แล้ว):** promo1/mat1 (1 แถว) + promo2/mat2 (2 แถว mat-group เดียวกัน)  
> → ควรได้ **2 BBY** · number `"1"`, `"2"` · ตอนนี้ได้ **3 BBY** number ซ้ำ `"1"` ทั้งหมด  
> **Material Grouping (col 28):** รวมแถวเมื่อ **`noof_promotion` + `noof_material_grouping` ตรงกัน** → 1 BBY · ดู edge cases + ตารางชื่อ field ในไฟล์  
> **JSON keys:** `noof_promotion` · `noof_material_grouping` · `noof_bonus_buy` · `comp_qty_in_sap` · `comp_set` · `comp_f3` (ไม่ใช่ `comp_%`)

---

## คำศัพท์ + ชื่อ field ให้ตรง (source of truth)

ใช้ชื่อ **`payload-keys.ts`** (export JSON บนเว็บ) · อย่าสับสนกับชื่อ Excel / DB

### 3 คอลัมน์ Multiple (อย่าใช้คำผิด)

| Col Excel | ชื่อบนฟอร์ม Mer C | **JSON ที่เทส/convert ใช้** | DB column | ห้ามเรียกผิด |
|-----------|-------------------|------------------------------|-----------|--------------|
| 27 | No. of Promotion | **`noof_promotion`** | `number_of_promotion` | ไม่ใช่เลข BBY |
| 28 | Material Grouping | **`noof_material_grouping`** | `number_of_material_grouping` | ไม่ใช่ `noof_promotion` |
| 29 | Bonus Buy | **`noof_bonus_buy`** | `number_of_bonus_buy` | → AB **`bonusBuyNumber`** |

**Alias ที่อาจเจอใน fixture เก่า** (รองรับได้ แต่ spec ใช้แถวบน):

| Alias (camelCase) | แปลเป็น |
|-------------------|---------|
| `numberOfPromotion` | `noof_promotion` |
| `numberOfMaterialGrouping` | `noof_material_grouping` |
| `numberOfBonusBuy` | `noof_bonus_buy` |

### Compensate + Charge Back (แถว MATERIALS)

| Col | Excel header | **JSON** | หมายเหตุ |
|-----|--------------|----------|----------|
| 59 | Compensate / ฿/Qty In SAP | **`comp_qty_in_sap`** | → rate ลำดับ 1 |
| 60 | Compensate / ฿/Set | **`comp_set`** | → rate ลำดับ 2 |
| 61 | Compensate / % | **`comp_f3`** | ไม่ใช่ `comp_%` |
| 69–77 | Charge Back … | ดูตารางงาน C+D | จากแถว / col charge back |
| 36 | Vendor | **`vendor`** | ไม่ใช่ `HEADER.vendorCode` |

### Material Grouping — ชื่อกลุ่ม (นอกแถว grid)

ชีท Excel แยก **Material Grouping** · ใน payload อาจเป็น array `materialGrouping[]` หรือ field บนแถว:

| แหล่ง | JSON key | ใช้ทำอะไร |
|-------|----------|-----------|
| ชีท / master | `grouping_name` | ชื่อกลุ่ม → AB `buy.field4` / `get.field4` เมื่อเป็นกลุ่ม |
| บาง fixture | `materialGroupName` | เทียบเท่า `grouping_name` |
| บาง fixture | `materialGroup` | รหัสกลุ่ม (ไม่ใช่ col 28) |

Col **28** = เลข **`noof_material_grouping`** (1, 2, 3…) — **คนละอย่างกับชื่อกลุ่ม**

### AB buy/get เมื่อเป็นกลุ่มสินค้า

| เงื่อนไข | `field2` | `field4` |
|----------|----------|----------|
| มีกลุ่ม (col 28 + ชื่อกลุ่ม) | **`"Material Group"`** | **`materialGroupName`** / `grouping_name` |
| สินค้ารายตัว | **`"Material"`** | **`material`** (รหัสสินค้า) |

Macro Excel ใช้ข้อความ `"MGPNew - New Material Group"` — **payload AB ใช้ `"Material Group"`** (ตาม `profile/MerC_to_AB_D001.md`)

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
| **A** | Mat-group grouping (col 28) | ❌ 1 แถว ≈ 1 BBY · แม้กรอก mat-group แล้ว | convert pipeline (ต่อจาก #8 ✅) |
| **B** | #9 `bonusBuyNumber` ← `noof_bonus_buy` | ❌ ซ้ำ `"1"` ทุกก้อน | `buildGroup*` / shared |
| **C** | #2 `Fill_Contract` — เรียก `mapConditionsFromMerC()` | ❌ มี function ยังไม่เรียก | shared |
| **D** | #3 Map Compensate (col 59–61 → rate) | ❌ ผูกกับ C | ใน `mapConditionsFromMerC()` |
| **E** | #6 FOC / MEK1 | ❌ ยังไม่เริ่ม | module แยก |

---

## งาน A — Material Grouping (col 28)

### ปัญหาตอนนี้

- Loop แถว → **1 แถว = 1 BONUSBUY** เสมอ
- ไม่รวมแถวที่ **`noof_material_grouping`** เดียวกัน ภายใน **`noof_promotion`** เดียวกัน
- **`bonusBuyNumber`** default `"1"` ทุกก้อน (งาน B — แก้คู่กัน)

### กติกา grouping (ตาม macro)

**Group key (ขั้นต่ำ):**

```
(noof_promotion, noof_material_grouping)
```

**Group key (แนะนำเมื่อ mixed profile/mechanic):**

```
(noof_promotion, noof_material_grouping, bonus_buy_profile, mechanic)
```

→ แถวที่ key เดียวกัน = **1 ก้อน `BONUSBUYS[]`**

| Col | JSON key | ความหมาย |
|-----|----------|----------|
| 27 | `noof_promotion` | ชุดโปร — loop ตัดช่วง first→last ตาม promo |
| 28 | `noof_material_grouping` | กลุ่มสินค้าใน promo — **เลขเดียวกัน → รวม 1 BBY** |
| 29 | `noof_bonus_buy` | เลขก้อน BBY → AB `bonusBuyNumber` (macro เขียนกลับ col 29) |

### Edge cases — dev ต้อง handle

| สถานการณ์ | พฤติกรรมที่ต้องการ |
|-----------|---------------------|
| Promo เดียว · mat-group **เดียวกัน** · หลายแถว | **1 BBY** · buy/get ตาม profile (หลาย material ในก้อน หรือ 1 กลุ่ม — ดูด้านล่าง) |
| Promo เดียว · mat-group **ต่างกัน** (1 vs 2) | **คนละ BBY** (2 ก้อน) |
| Promo **ต่างกัน** · mat-group เท่าไหร่ก็ได้ | **คนละ BBY** ตาม promo+mat-group |
| Col 28 **ว่าง** ทุกแถว | `is_Mat_Group = false` — ใช้ **`material`** รายตัว · **ไม่ merge** ตาม mat-group (1 แถวที่ convert ได้ ≈ 1 BBY ถ้า promo แยก) |
| Col 28 มีบางแถว · บางแถวว่าง | แถวว่าง **ไม่** merge เข้ากลุ่มที่มีเลข — treat แยกหรือ skip ตาม macro reject rules |
| Mechanic `(On Pack)` B1G1/B2G1 | **skip** แถว (macro ไม่ convert) |
| Mixed profile (D001 + P011) ในใบเดียว | group **แยก profile** — ห้ามบังคับ mechanic เดียวทั้งใบ |

### หลัง merge 1 BBY — buy[] / get[]

| กรณี | AB |
|------|-----|
| กลุ่มสินค้า (มีชื่อกลุ่ม) | `field2: "Material Group"` · `field4: grouping_name` · qty/ราคาตาม mechanic **ก้อน** |
| หลาย material ในกลุ่มเดียว (macro) | อาจ **หลายแถว** ใน buy[] / get[] หรือ 1 กลุ่ม — parity กับ `Gen_D001` + `is_Mat_Group=True` |
| สินค้าเดียวซ้ำ 2 แถว (เคส QA) | หลัง merge ควร **1 ก้อน** · ไม่ duplicate BBY 3 ก้อน |

Fixture อ้างอิง mat-group: `tungconvert/D001/5-D001-mgp.json`

### Macro อ้างอิง (`Mer-C_Convert_To_STD.txt`)

| หัวข้อ | บรรทัดโดยประมาณ |
|--------|------------------|
| `col_no_Mat_Group = 28` | ~9 |
| Check `is_Mat_Group` (col 28 มีค่าไหม) | ~552–559 |
| Fill ชีท Material Grouping | ~576–637 |
| Dedupe รายชื่อ/เลขกลุ่ม | ~649–677 |
| `Gen_D001` + `is_Mat_Group` branch | ~3620+ (`MGPNew` / `Mat_Group_Name_List`) |

### ตัวอย่าง HHC (~14 แถว → ~4 BBY)

| Promo | แถว | Mat group | BBY ที่ควรได้ |
|-------|-----|-----------|---------------|
| 1 | 5 | 1 | 1 ก้อน |
| 2 | 3–4 | 2 | 1 ก้อน |
| 3 | 4 | 3 | 1 ก้อน |
| 4 | 1 | 4 | 1 ก้อน |

#### เคส 3 — promo เดียว · mat-group ต่างกัน

| แถว | `noof_promotion` | `noof_material_grouping` | BBY |
|-----|------------------|--------------------------|-----|
| 0 | 1 | 1 | ก้อน 1 |
| 1 | 1 | 2 | ก้อน 2 |

→ **2 BBY** ใน promo 1 (ไม่ merge เพราะ mat-group ต่าง)

### เทสจาก QA (2026-08-31)

#### เคส 1 — ไม่กรอก mat-group

**INPUT:** promo 1 (1 แถว) + promo 2 (2 แถว) · D001 2For · HEADER ว่าง · ไม่มี `noof_material_grouping`

| ตอนนี้ | ควรได้ (หลัง grouping) |
|--------|------------------------|
| 3 BBY แยก · number `"1"` ทั้งหมด | **2 BBY** · `"1"`, `"2"` |

#### เคส 2 — กรอก mat-group แล้ว (reproduce ล่าสุด)

**INPUT:** HEADER ว่าง · D001 2For · 3 CONDITIONS passthrough · `days: ["Sun"]`

| แถว C | `noof_promotion` | `noof_material_grouping` | `noof_bonus_buy` | AB ตอนนี้ | AB ควรได้ |
|-------|------------------|--------------------------|------------------|-----------|-----------|
| 0 | 1 | **1** | (ว่าง) | BBY[0] `"1"` | BBY[0] `"1"` ✅ |
| 1 | 2 | **2** | (ว่าง) | BBY[1] `"1"` ❌ | รวมกับแถว 2 |
| 2 | 2 | **2** | (ว่าง) | BBY[2] `"1"` ❌ | **BBY[1] `"2"`** (1 ก้อน) |

**สรุปเคส 2:**

| | ตอนนี้ | หลัง A+B |
|--|--------|----------|
| จำนวน BBY | 3 | **2** |
| `bonusBuyNumber` | `"1"`, `"1"`, `"1"` | **`"1"`, `"2"`** |
| BBY buy/get/qty/fieldP | ถูก (2For · 490/500) | ยังต้องถูกหลัง merge |
| CONDITIONS | passthrough 3 ก้อน ✅ | ไม่กระทบจน C+D |

**Console ที่เห็น:** `BONUSBUYS: Array(3)` · ทุกก้อน `bonusBuyNumber: '1'`

### Checklist dev — งาน A

- [ ] Group แถวด้วย `(noof_promotion, noof_material_grouping)` ก่อน build BBY
- [ ] ไม่ hardcode 1 row = 1 BBY
- [ ] Promo เดียว + mat-group ต่างกัน → BBY แยก (เคส 3)
- [ ] Col 28 ว่าง → ไม่ merge mat-group · ใช้ Material รายตัว
- [ ] หลัง group → `field2` / `field4` ถูก (Material Group + ชื่อกลุ่ม หรือ Material + รหัส)
- [ ] หลัง group แล้ว route `buildD001` / `buildP011` ตาม `bonus_buy_profile`
- [ ] Mixed profile ในใบเดียวยังทำงาน
- [ ] ทำคู่กับ **งาน B** — `bonusBuyNumber` ไล่ `"1"`, `"2"`, … หลัง group

```ts
function groupMaterialRows(materials) {
  const groups = new Map()
  for (const row of materials) {
    if (!shouldConvertRow(row)) continue
    const key = [
      row.noof_promotion ?? row.numberOfPromotion,
      row.noof_material_grouping ?? row.numberOfMaterialGrouping ?? '',
      row.bonus_buy_profile,
      row.mechanic,
    ].join('|')
    if (!groups.has(key)) groups.set(key, [])
    groups.get(key).push(row)
  }
  return [...groups.values()]
}
```

---

## งาน B — #9 `bonusBuyNumber`

### ปัญหาตอนนี้

ทุกก้อนได้ `bonusBuyNumber: "1"` แม้ `noof_promotion` ต่างกัน

### กติกา map

| จาก C | → AB | ห้าม |
|-------|------|------|
| **`noof_bonus_buy`** / `numberOfBonusBuy` | **`bonusBuyNumber`** | — |
| (ไม่มีค่า) | ดู **หมายเหตุด้านล่าง** | — |
| `noof_promotion` | — | **ไม่ map** เป็น bonusBuyNumber |
| `bonusBuyProfile` | — | **ไม่ map** |

```ts
function resolveBonusBuyNumber(row, groupIndex?) {
  // 1) col 29 จากแถวหรือ group leader หลัง mat-group
  const fromRow = row.noof_bonus_buy ?? row.numberOfBonusBuy
  if (fromRow != null && fromRow !== '') return String(fromRow)
  // 2) fallback ตาม macro: ลำดับก้อน BBY ทั้งใบ (1, 2, 3…) — ไม่ใช่ noof_promotion โดยตรง
  if (groupIndex != null) return String(groupIndex + 1)
  return '1'
}
```

ใส่ให้ตรงกัน: `BONUSBUYS[i].bonusBuyNumber` · `bonusBuyHeader` · `buy[0]` · `get[0]`

### หมายเหตุ — `noof_bonus_buy` ว่าง (เคส QA)

- บน Excel macro มัก **เขียนกลับ** col 29 ตอน convert
- ถ้า web ยังไม่ส่ง col 29 → หลัง mat-group ใช้ **ลำดับก้อน BBY ทั้งใบ** (`"1"`, `"2"`, …) ไม่ใช่ default `"1"` ทุกก้อน
- **ห้าม** map `noof_promotion` → `bonusBuyNumber` (col 27 ≠ col 29)

### Checklist dev

- [ ] อ่an `noof_bonus_buy` จากแถว (หรือจาก group leader หลัง mat-group)
- [ ] ถ้าว่าง — fallback ลำดับก้อนหลัง group ไม่ใช่ `"1"` ซ้ำทุกก้อน
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
| 61 | Compensate / % | **`comp_f3`** | rate % + type ZR01 |
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
- **#8 Per-row router** — HEADER profile/rebate/contract ว่าง + แถวมี `bonus_buy_profile` → BBY ออก ✅ (2026-08-31)
- CONDITIONS passthrough — เคสกรอก Condition block 3 ก้อน · MSH02 · rate 1 copy ตรง ✅
- `MATERIALS: []` ฝั่ง AB ✅
- HEADER copy — vendor ว่าง → `NOBP` ✅

---

## QA จะเทสอะไรหลังแต่ละงาน

### หลัง A + B (mat-group + bonusBuyNumber)

1. **เคส 2 ด้านบน** — promo1/mat1 + promo2/mat2×2 → **2 BBY** · `"1"`, `"2"` (ไม่ใช่ 3 ก้อน number ซ้ำ)
2. HHC ~14 แถว / 4 mat group → **~4 BBY** (ไม่ใช่ 14)
3. BBY buy/get/qty/fieldP ยังถูก profile หลัง merge
4. Mixed profile (D001 + P011) ในใบเดียว → ก้อนแยกตาม profile + group

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
| [`payload-keys.ts`](../payload-keys.ts) | **ชื่อ JSON มาตรฐาน** (material · compensate · col 27–29) |
| [`Mer-C_Field_Mapping_Guide.md`](../Mer-C_Field_Mapping_Guide.md) | col Excel · § B ตัวแบ่งกลุ่ม · § D mat-group |
| [`MerC_to_AB_ExistsInAB_Mapping.md`](../MerC_to_AB_ExistsInAB_Mapping.md) | col 28/29/59–61/69–77 → cell AB |
| [`mechanic-qty-lookup.json`](mechanic-qty-lookup.json) | mechanic → buyQty/getQty |
| [`profile/MerC_to_AB_*.md`](../profile/) | Spec buy/get · `field2` Material Group |
| [`Mer-C_Convert_To_STD.txt`](../Mer-C_Convert_To_STD.txt) | Macro mat-group ~552–677 · Fill_Contract ~4625 |
| [`payload_AB_Reference.md`](../payload_AB_Reference.md) | §4 CONDITIONS structure · § bonusBuyHeader |
| [`tungconvert/D001/5-D001-mgp.json`](D001/5-D001-mgp.json) | Fixture Material Group |
| [`Add-rebateChargeback-normalize.md`](Add-rebateChargeback-normalize.md) | Checklist 1–10 เต็ม |
| [`Dev-handoff-Multiple-CONDITIONS.md`](Dev-handoff-Multiple-CONDITIONS.md) | Multiple · CONDITIONS · #8 |

---

## Out of scope ไฟล์นี้ (ดู doc อื่น)

| งาน | ไฟล์ |
|-----|------|
| #1 normalizeRebate | `Add-rebateChargeback-normalize.md` |
| #4 D003 Coupon merge · #5 F003 merge | `Add-rebateChargeback-normalize.md` |
| #10 dedupe stores | `Add-rebateChargeback-normalize.md` |
