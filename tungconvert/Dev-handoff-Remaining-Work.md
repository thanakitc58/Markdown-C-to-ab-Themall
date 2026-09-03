# Dev Handoff — งานค้าง Convert Mer C → AB

| | |
|--|--|
| **อัปเดต** | 2026-09-03 |
| **สำหรับ** | Dev implement / QA เช็คครบ |
| **อ้างอิง macro** | `Mer-C_Convert_To_STD.txt` |
| **Fixture** | โฟลเดอร์ `JsonBBY/` |
| **เอกสารเก่า** | [`Dev-handoff-Convert-Remaining.md`](Dev-handoff-Convert-Remaining.md) · [`Add-rebateChargeback-normalize.md`](Add-rebateChargeback-normalize.md) |

---

## ข้อความสั้นส่ง Dev (copy ได้)

> งานค้าง convert อยู่ในไฟล์นี้ — **อ่านก่อนแก้**
>
> **ลำดับทำ:** **A + B คู่กัน** → **1** → **C+D** → **4** → **E/6** → **5 / 10** (optional)
>
> **A** Group แถวด้วย `(noof_promotion, noof_material_grouping)` — ตอนนี้ 1 แถว = 1 BBY ❌  
> **B** `bonusBuyNumber` ไล่ `"1"`,`"2"`,`"3"` หลัง group — ตอนนี้ซ้ำ `"1"` ทุกก้อน ❌  
> ลูกค้ากรอก `noof_bonus_buy` เองได้ แต่ **ห้ามใช้ทับกติกา group** (mat-group ต่าง = คนละก้อน แม้ใส่ `1`,`1`)
>
> Macro: `Mer-C_Convert_To_STD.txt` บรรทัด **8–10**, **209–356**, **554–559**, **622–677**, **1348–1414**  
> Fixture: `JsonBBY/case-*.c.json` (อย่าใช้ `case-3-*.ab.json` เป็น golden — ยังเป็น output ผิด)

---

## สถานะรวม (เช็คครบหรือยัง)

| # | งาน | สถานะ | ทำที่ |
|---|-----|--------|------|
| **8** | Multiple per-row router (profile/mechanic ต่อแถว) | ✅ ผ่านแล้ว (ประมาณ) | convert pipeline |
| **A** | Material Grouping (col 28) | ❌ ยัง 1 แถว ≈ 1 BBY | shared / group |
| **B / 9** | `bonusBuyNumber` | ❌ ยังซ้ำ `"1"` ทุกก้อน | shared / group |
| **1** | `normalizeRebate` (Reason X → `"X"`) | ❌ | `mapHeaderShared` |
| **C / 2** | เรียก `mapConditionsFromMerC()` / Fill_Contract | ❌ มี function ยังไม่เรียก | shared |
| **D / 3** | Map Compensate col 59–61 → rate | ❌ ผูกกับ C | ใน `mapConditionsFromMerC` |
| **4** | D003 Coupon merge `(B)` → ก้อนก่อนหน้า | ❌ | `buildD003` |
| **5** | F003 `(B)` merge | ❌ optional | `buildF003` |
| **E / 6** | FOC / MEK1 | ❌ ยังไม่เริ่ม | module แยก |
| **10** | Dedupe `stores[]` | nice-to-have | shared |

**ขั้นต่ำส่งลูกค้า:** A + B + 1 + C+D + 8  
**parity macro เต็ม:** + 4 + E/6 (+ 5 ถ้าต้องการ)

**ผ่านแล้ว (อย่าแก้พัง):** buy/get ตาม profile · CONDITIONS passthrough ชั่วคราว · HEADER ไม่ trigger BBY

---

## 3 คอลัมน์ Multiple (source of truth)

| Col Excel | JSON key | ความหมาย | → AB |
|-----------|----------|----------|------|
| 27 | `noof_promotion` | ชุดโปร | **ห้าม** map เป็น `bonusBuyNumber` |
| 28 | `noof_material_grouping` | กลุ่มสินค้าใน promo | ใช้ **group** |
| 29 | `noof_bonus_buy` | เลขก้อน BBY | → **`bonusBuyNumber`** |

Macro const บรรทัด **8–10**:

```
col_no_Number_of_Promotion = 27
col_no_Mat_Group           = 28
col_no_BBY_no              = 29
```

Alias เก่าที่รองรับได้: `numberOfPromotion` · `numberOfMaterialGrouping` · `numberOfBonusBuy`

---

# งาน A + B — Grouping + `bonusBuyNumber` (ทำคู่กัน)

## ปัญหาตอนนี้

- Loop แถว → **1 แถว = 1 BONUSBUY** เสมอ
- ไม่รวมแถวที่ mat-group เดียวกันใน promo เดียวกัน
- `bonusBuyNumber` default / ซ้ำ **`"1"`** ทุกก้อน

## กติกา group (ตาม macro)

**Group key (ขั้นต่ำ):**

```
(noof_promotion, noof_material_grouping)
```

**Group key (แนะนำเมื่อ mixed profile/mechanic):**

```
(noof_promotion, noof_material_grouping, bonus_buy_profile, mechanic)
```

| สถานการณ์ | ผลที่ต้องการ |
|-----------|----------------|
| promo เดียว · mat-group **เดียวกัน** · หลายแถว | **1 BBY** |
| promo เดียว · mat-group **ต่าง** | **คนละ BBY** |
| promo **ต่าง** | **คนละ BBY** |
| col 28 **ว่างทั้งใบ** | ไม่ merge แบบ Material Group · แยกตาม **promo** |
| col 28 มีบางแถว · บางแถวว่าง | แถวว่าง **ไม่** รวมกับกลุ่มที่มีเลข |

### หลัง group — buy/get

| กรณี | AB `field2` / `field4` |
|------|------------------------|
| มีกลุ่ม (col 28 + ชื่อกลุ่ม) | `"Material Group"` / `grouping_name` |
| สินค้ารายตัว | `"Material"` / `material` |

Payload AB ใช้ `"Material Group"` (ไม่ใช้ข้อความ macro `"MGPNew - New Material Group"`)

## กติกา `bonusBuyNumber`

1. Group แถวก่อน  
2. นับก้อนทั้งใบ → `"1"`, `"2"`, `"3"` …  
3. ใส่ให้ตรงกันทั้งก้อน:  
   `BONUSBUYS[].bonusBuyNumber` · `bonusBuyHeader.bonusBuyNumber` · `buy[].bonusBuyNumber` · `get[].bonusBuyNumber`

### ถ้าลูกค้ากรอก `noof_bonus_buy` (col 29) เอง

| สถานการณ์ | พฤติกรรม |
|-----------|----------|
| **ว่าง** | generate `"1"`, `"2"`, … หลัง group |
| กรอกแล้ว **และสอดคล้อง group** (ก้อนเดียวกันเลขเดียวกัน) | ใช้เลขที่กรอกได้ |
| กรอก **ขัดกับ group** เช่น mat-group ต่าง แต่ใส่ `1`,`1` | **ยังต้องได้ 2 ก้อน** · เลข `"1"`,`"2"` — **ห้าม merge เพราะเลขซ้ำ** |

**สำคัญ (parity macro):**  
Macro **เขียนทับ** col 29 ด้วย `Bonus_Buy_Index` — **ไม่อ่าน** ค่าลูกค้ามาเป็นตัวตัดสินใจ group  
→ col 29 = hint / output · **ห้ามใช้ทับกติกา `(promo, mat-group)`**

### ห้าม

- default `"1"` ทุกก้อน  
- map `noof_promotion` → `bonusBuyNumber`  
- 1 แถว = 1 BBY เสมอ  
- ใช้เลข col 29 เป็นตัว merge ถ้า mat-group ต่าง

## Macro อ้างอิง

| หัวข้อ | บรรทัดโดยประมาณ |
|--------|------------------|
| col 27 / 28 / 29 | **8–10** |
| ตัดช่วงตาม promo (แยกไฟล์ Excel) | **209–356** |
| เช็คมี mat-group ไหม (`is_Mat_Group`) | **554–559** |
| fill + dedupe `Mat_Group_Number_List` | **622–677** |
| `Bonus_Buy_Index = 1` | **1216** |
| ไม่มี mat-group branch | **1235–1344** |
| loop ตาม mat-group + เขียน col 29 | **1348–1352** |
| `Bonus_Buy_Index + 1` (เช่น D001) | **1414** |

หมายเหตุ: Excel macro **แยกไฟล์ตาม promo** · mat-group แยกแค่ก้อน BBY **ในไฟล์**  
Web convert = **1 JSON** · หลายก้อนใน `BONUSBUYS[]`

## ตัวอย่างเทส A+B

| INPUT | Expected |
|-------|----------|
| promo1/mat1 ×2 · col 29 ว่าง | **1 BBY** · `"1"` |
| promo 1×2 + 2×2 + 3×2 (mat ตามกลุ่ม) | **3 BBY** · `"1"`,`"2"`,`"3"` |
| promo1 D001 ×2 + promo2 F001 ×1 | **2 BBY** · `"1"`,`"2"` |
| promo28 · mat 93 + 94 · col 29 = 1,2 | **2 BBY** · `"1"`,`"2"` |
| promo28 · mat 93 + 94 · ลูกค้าใส่ **1,1** | **ยัง 2 BBY** · `"1"`,`"2"` (ห้าม `"1"`,`"1"`) |
| ไม่มี mat-group · promo `1,1,2,2` | **2 BBY** · `"1"`,`"2"` |

### Fixture `JsonBBY/`

| ไฟล์ | Expected |
|------|----------|
| `case-1-111.c.json` | 1 BBY · `"1"` |
| `case-2-11-22-333.c.json` | 3 BBY · `"1"`,`"2"`,`"3"` |
| `case-3-1 2 3.c.json` | 2 BBY · `"1"`,`"2"` |

> **อย่าใช้** `case-3-1 2 3.ab.json` เป็น golden — เป็น output ผิดของ dev (หลายก้อน · number ซ้ำ `"1"`)  
> ไฟล์ `.ab.json` อื่นถ้ายังสะท้อน bug เดิม ก็ regenerate หลังแก้ A+B

### เช็คเร็วหลัง convert

```js
const nums = ab.BONUSBUYS.map(b => b.bonusBuyNumber)
console.log(ab.BONUSBUYS.length, nums)
// เคส 2 ต้องได้: 3 ["1","2","3"]
// ถ้า length = จำนวนแถว หรือ nums ซ้ำ ["1","1","1"] → ยังพัง
```

## Checklist A+B

- [ ] Group ด้วย `(noof_promotion, noof_material_grouping)` ก่อน build BBY
- [ ] Mixed profile/mechanic → ใส่ profile + mechanic ใน key ด้วย
- [ ] ไม่ hardcode 1 row = 1 BBY
- [ ] `bonusBuyNumber` ไล่หลัง group · ไม่ซ้ำ `"1"` ทุกก้อน
- [ ] ห้าม map จาก `noof_promotion`
- [ ] ลูกค้ากรอก col 29 ขัด group → ยังแยกตาม mat-group
- [ ] เลขตรงกันทั้ง header / buy / get
- [ ] รัน `JsonBBY` เคส 1–3 ผ่าน

---

# งานอื่นที่ยังค้าง

## งาน 1 — `normalizeRebate`

| INPUT | AB ที่ถูก |
|-------|-----------|
| Reason `X` + contract `Z200` | `HEADER.rebateChargeback` = **`"X"`** ไม่ใช่ `""` |

- [ ] แก้แล้ว · เทส Reason X ไม่หายหลัง convert

## งาน C+D — Fill_Contract + Compensate

| ตอนนี้ | ควรได้ |
|--------|--------|
| `mapConditionsFromMerC()` มีแต่ยังไม่เรียก | เรียกตอน convert |
| `CONDITIONS` = passthrough จากมือ | gen จาก MATERIALS + compensate |
| `comp_qty_in_sap: 8` / `comp_set: 16` → rate ยัง `1` | `cond_type_condition_rate` = **8** (ลำดับ qty ก่อน set) |

JSON keys: `comp_qty_in_sap` · `comp_set` · `comp_f3` (ไม่ใช่ `comp_%`)

- [ ] เรียก `mapConditionsFromMerC()`
- [ ] map compensate → rate ถูก
- [ ] vendor จากแถว MATERIALS ไม่ทับด้วย `HEADER.vendorCode` ผิดๆ

## งาน 4 — D003 Coupon merge

| แถว mechanic | ผลที่ถูก |
|--------------|----------|
| `A (Coupon)+B (A)` | 1 ก้อน (buy) |
| `A (Coupon)+B (B)` | **merge get เข้าก้อนก่อนหน้า** · ไม่สร้างก้อนใหม่ |

- [ ] 2 แถว A+B → **1** BONUSBUY มีทั้ง buy + get

## งาน 5 — F003 `(B)` merge (optional)

Logic คล้ายงาน 4 · ทำถ้าต้องการ parity macro F003

- [ ] (optional) merge `(B)` เข้า `(A)`

## งาน E / 6 — FOC / MEK1

- trigger จาก `cost_foc` + `cost_date_*` ใน MATERIALS  
- output **แยก** จาก `BONUSBUYS`  
- `cost_foc` ว่าง = **ไม่สร้าง**

- [ ] มี module FOC/MEK1 · ไม่พัง BBY/CONDITIONS

## งาน 10 — Dedupe `stores[]` (nice-to-have)

- [ ] (optional) ร้านซ้ำจาก C ไม่ copy ซ้ำไป AB

## งานเสริมที่ควรมีตอน convert

- [ ] `promotionArea` จาก `disp_mprice_category` (ถ้ายังขาดใน AB header ก้อน)
- [ ] regenerate golden `JsonBBY/case-3-*.ab.json` หลัง A+B ผ่าน

---

# วิธีเช็คว่าครบใน project จริง

## 1) Search ในโค้ด

| ค้นหา | ต้องเจอ |
|--------|---------|
| `noof_material_grouping` / group key | group ก่อน build BBY |
| `noof_bonus_buy` / `bonusBuyNumber` | resolve หลัง group · ได้ `"2"` ได้ |
| `mapConditionsFromMerC` | **ถูกเรียก** ใน pipeline |
| `normalizeRebate` | Reason X ไม่เป็น `""` |
| Coupon / merge `(B)` | D003 |
| `cost_foc` / FOC / MEK1 | module แยก |

มีแค่ define แต่ไม่เรียก = **ยังไม่ครบ**

## 2) รัน fixture แล้วเทียบ

เอา `JsonBBY/case-*.c.json` ยิง convert → เทียบตาราง Expected ด้านบน

## 3) เทสรวมหลังทำครบ

| # | เทส | ผ่านเมื่อ |
|---|-----|-----------|
| 1 | Reason X + Z200 | `rebateChargeback === "X"` |
| 2 | D001 + comp 8 / set 16 | `condition_rate === 8` |
| 3 | D003 Coupon A+B | 1 ก้อน buy+get |
| 4 | FOC มี `cost_foc` | มี output แยก |
| 5 | Multiple · HEADER profile ว่าง | ยังได้ BBY ตามแถว |
| 6 | `JsonBBY` เคส 1–3 | จำนวนก้อน + เลขถูก |
| 7 | mat ต่าง · ลูกค้าใส่ bbynumber `1`,`1` | ยังได้ `"1"`,`"2"` |

---

## Done = อะไร

| ระดับ | เงื่อนไข |
|-------|----------|
| **ส่งลูกค้าขั้นต่ำ** | A + B + 1 + C+D + (#8 ผ่านแล้ว) |
| **parity macro** | + 4 + E/6 (+ 5) |
| **ไม่ต้องทำ** | `pack_size`, `forecast_*`, `gp_*` ฯลฯ (macro ก็ไม่ส่ง AB) |
| **Out of scope** | P100, P103, F002 (ตาม handoff เดิม) |
