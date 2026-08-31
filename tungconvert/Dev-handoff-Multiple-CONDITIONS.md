# Dev Handoff — Multiple mode · HEADER ไม่ trigger · CONDITIONS

| | |
|--|--|
| **อัปเดต** | 2026-08-31 |
| **เอกสารหลัก (checklist 1–10)** | [`Add-rebateChargeback-normalize.md`](Add-rebateChargeback-normalize.md) |
| **จากเทสจริง** | QA บนเว็บ Mer C → AB · ส่ง payload C/AB เปรียบเทียบ |

เอกสารนี้รวม **กติกา Mer C โหมด Multiple**, **bug convert ที่เจอ**, และ **CONDITIONS ถูก merge ผิด** — ให้ dev ยึดตอน implement งาน #8–9 และก่อน/หลัง Fill_Contract

---

## สรุป 1 ประโยค

> **Multiple:** convert BBY จาก **`MATERIALS[]` per-row (+ grouping ตาม macro)** — **HEADER เป็น metadata ร่วม ไม่ใช่ trigger**  
> **CONDITIONS:** ก่อน Fill_Contract เสร็จ → **passthrough ล้วน** ห้ามทับ vendor/rate แบบครึ่งๆ กลางๆ

---

## 1) โหมด Multiple บน Mer C — user กรอกยังไง

เมื่อ **Single / Multiple Promotion = Multiple** (* บังคับ):

| ช่อง HEADER | บังคับ? | หมายเหตุ |
|-------------|---------|----------|
| Group, Theme, Promotion Name, WBS, Period, Time | ✅ | ใช้ร่วมทั้งใบ |
| **Single / Multiple = Multiple** | ✅ | |
| **Bonus Buy Profile** | ❌ | profile อยู่ที่แถว `MATERIALS.bonus_buy_profile` · คนละแถวคนละ profile ได้ |
| **Rebate Chargeback** | ❌ | user เลือกใส่หรือไม่ใส่ · contract ไปที่ **Condition block** หรือแถวสินค้า |
| **Contract Type** | ❌ | มักว่าง/disable · ไปอยู่ `CONDITIONS` |
| **Vendor (HEADER)** | ❌ | vendor จริงอยู่แถวสินค้า / `CONDITIONS` |

**User ไม่ควรถูกบังคับกรอก HEADER profile / rebate เพื่อให้ convert ผ่าน**

---

## 2) โครง Excel / MATERIALS — 3 เลข (อย่าสับสน)

| Col Mer C | JSON key | ความหมาย |
|-----------|----------|----------|
| 27 No. of **Promotion** | `noof_promotion` | **ชุดโปร** — 1 promo มี **หลายแถว** ได้ (5 แถว promo 1 ฯลฯ) |
| 28 **Material Grouping** | `noof_material_grouping` | กลุ่มสินค้าใน promo — แถว mat group เดียวกัน → อาจ **รวม 1 BBY** |
| 29 **Bonus Buy** | `noof_bonus_buy` | เลขก้อน BBY — macro **เขียนกลับ** ตอน convert → `bonusBuyNumber` ใน AB |

```
noof_promotion     = กล่องใหญ่ (ชุดโปร 1, 2, 3…)
noof_material_grouping = กลุ่มสินค้าในกล่อง
noof_bonus_buy     = ลำดับก้อน BBY  →  bonusBuyNumber
bonus_buy_profile  = D001 / P010… (route builder)
```

**ไม่ใช่:** 1 แถว = 1 promotion · **ไม่ใช่:** `bonusBuyNumber` ← `noof_promotion`

### ตัวอย่าง HHC (หลายแถว / promo)

| Promotion | แถว MATERIALS | Mat group | BBY ที่ควรได้ (macro) |
|-----------|---------------|-----------|-------------------------|
| 1 | 5 แถว | ทั้งหมด `1` | **1 ก้อน** BBY (`bonusBuyNumber "1"`) |
| 2 | 3–4 แถว | ทั้งหมด `2` | **1 ก้อน** (`"2"` ถ้าไล่ทั้งใบ) |
| 3 | 4 แถว | `3` | **1 ก้อน** |
| 4 | 1 แถว | `4` | **1 ก้อน** |

รวม ~14 แถว → **~4 ก้อน BONUSBUYS** ไม่ใช่ 14 ก้อน

---

## 3) Bug #8 — อย่า trigger convert จาก HEADER

### อาการ (reproduce)

| เคส | `HEADER.bonusBuyProfile` | แถว `bonus_buy_profile` | `BONUSBUYS` | `MATERIALS` AB |
|-----|--------------------------|-------------------------|-------------|----------------|
| A | `""` | D001 × 2 | `[]` ❌ | ยังอยู่ ❌ |
| B | `"D001"` | D001 × 2 | 2 ก้อน ✅ | `[]` ✅ |
| C | `""` | P010 | `[]` ❌ | ยังอยู่ ❌ |
| D | `"D001"` | D001 · 2For 1 แถว | 1 ก้อน ✅ | `[]` ✅ |

### Expected

```ts
function convertMaterialsToBonusBuys(header, materials) {
  const out = []
  for (const row of materials) {
    if (!shouldConvertRow(row)) continue
    const profile = shortCode(row.bonus_buy_profile ?? row.bonusBuyProfile)
    if (!profile) continue
    out.push(getProfileBuilder(profile)({ header, row }))
  }
  return out
}
// HEADER.bonusBuyProfile — copy ได้ แต่ห้ามเป็นเงื่อนไขเดียวที่จะ convert
```

- รองรับ **mixed profile** (D001 + P011 ในใบเดียว)
- รองรับ **mixed mechanic** ต่อแถว (qty lookup แยกแถว)
- ต้องมี **grouping** ตาม mat group / macro (ไม่ใช่ 1 แถว = 1 BBY เสมอ)

---

## 4) Bug #9 — `bonusBuyNumber`

| จาก C | → AB |
|-------|------|
| **`noof_bonus_buy`** / `numberOfBonusBuy` | **`bonusBuyNumber`** (หลัก) |
| `noof_promotion` | **ไม่ map** เป็น bonusBuyNumber |
| `bonusBuyProfile` | **ไม่ใช่** bonusBuyNumber |

```ts
function resolveBonusBuyNumber(row) {
  return String(row.noof_bonus_buy ?? row.numberOfBonusBuy ?? 1)
}
```

ใส่ให้ตรงกัน: `bonusBuyHeader`, `buy[0]`, `get[0]`

ภายใน promo เดียว หลาย BBY: `"1"`, `"2"`, `"3"`… ตาม **`noof_bonus_buy`** ไม่ใช่จำนวนแถว

---

## 5) CONDITIONS — passthrough vs merge ผิด (เทส 2026-08-31)

### คำว่า passthrough

**Passthrough** = copy `C.CONDITIONS` → `AB.CONDITIONS` **ตรงๆ ไม่แก้ field**

### เคส D001 · กรอก Condition block มือ · HEADER rebate/contract ว่าง

User **ไม่กรอก** Rebate Chargeback / Contract Type ที่ HEADER (ถูกตาม Multiple)  
กรอก contract ใน **Condition block**: Reason X, Z200, vendor MSH02, rate 1

| field | C (กรอก) | AB (ได้) | ถูกไหม |
|-------|----------|----------|--------|
| `HEADER.rebateChargeback` | `""` | `""` | ✅ ไม่ได้กรอก HEADER |
| `cond_hdr_vendor` | **MSH02** | **OIS05** | ❌ ถูกแทนด้วย `HEADER.vendorCode` |
| `cond_type_condition_rate` | **1** | **0** | ❌ ถูกทับจาก `comp_set: 0` |

**สรุป:** ไม่ใช่ passthrough ล้วน — มี logic ครึ่งๆ กลางๆ ทับค่าที่ user กรอก

### ต้องการจาก dev

| ช่วง | พฤติกรรม |
|------|-----------|
| **ก่อน Fill_Contract (#2) เสร็จ** | **Passthrough ล้วน** — อย่า merge/touch vendor, rate, หรือ field อื่นใน CONDITIONS |
| **หลัง Fill_Contract (#2–3)** | Gen จาก MATERIALS + HEADER ตาม macro · **gen ทั้งก้อน** ไม่ merge บาง field |
| Vendor ใน contract | จาก **แถวสินค้า / charge back cols** — **ไม่ใช่** `HEADER.vendorCode` แทน vendor แถว |
| `comp_set: 0` / ว่าง | อย่าทับ rate ที่กรอกใน Condition block จนกว่าจะ gen ทั้งก้อนจาก Mer C ชัดเจน |

---

## 6) เทส BBY ที่ผ่านแล้ว (reference)

### D001 · 2For · fieldP · HEADER มี D001

```
buy.field9 = 2, get.getQuantity = 2
buy.field8 = sales_price_normal (490)
get.fieldP = sales_price_promo (1000)
ไม่มี get.field8
MATERIALS = []
```

### D001 · Multiple · 2 แถว · HEADER D001

- 2 ก้อน BBY · stores แยกกัน ✅  
- `bonusBuyNumber` ทั้งคู่ `"1"` ยัง ❌ (รอ #9)

---

## 7) Checklist dev (ลำดับแนะนำ)

- [ ] **#8** Per-row router + mat group grouping — ไม่ trigger จาก HEADER profile
- [ ] **CONDITIONS passthrough** — หยุดทับ vendor/rate จน Fill_Contract พร้อม
- [ ] **#9** `bonusBuyNumber` ← `noof_bonus_buy`
- [ ] **#1** `normalizeRebate` (เมื่อ user กรอก Reason ที่ HEADER)
- [ ] **#2–3** Fill_Contract + Compensate (gen ทั้งก้อน ไม่ merge ครึ่งๆ)
- [ ] **#4–6** D003 Coupon · F003 · FOC
- [ ] **#10** Dedupe stores (nice-to-have)

---

## 8) ไฟล์อ้างอิง

| ไฟล์ | ใช้ทำอะไร |
|------|-----------|
| [`Add-rebateChargeback-normalize.md`](Add-rebateChargeback-normalize.md) | Checklist 1–10 · rebate · Fill_Contract · Coupon |
| [`profile/MerC_to_AB_*.md`](../profile/) | Spec ต่อ profile (buy/get, field8, P/R/%) |
| [`mechanic-qty-lookup.json`](mechanic-qty-lookup.json) | mechanic → buyQty / getQty |
| [`Mer-C_Convert_To_STD.txt`](../Mer-C_Convert_To_STD.txt) | Macro · col 27–29 · Fill_Contract |
| [`Mer-C_Field_Mapping_Guide.md`](../Mer-C_Field_Mapping_Guide.md) | col 27 Promotion · 28 Mat Group · 29 BBY no |

---

## 9) Done = QA จะเทสอะไรหลัง fix

1. Multiple · HEADER profile **ว่าง** · แถวมี P010 / D001 → BBY ออก
2. Mixed profile 3 แถว (D001 + P011 + D001) → 3 ก้อน · โครง buy/get ถูก profile
3. Condition block กรอก vendor MSH02 rate 1 → AB **ไม่เปลี่ยน** (จนกว่า Fill_Contract จะ gen ทั้งก้อน)
4. HHC หลายแถว / mat group เดียว → จำนวน BBY ตาม grouping ไม่ใช่จำนวนแถว
5. `bonusBuyNumber` ไล่ 1, 2, 3… ตาม `noof_bonus_buy`
