# C → AB — Profile P011 (Full)

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P011_Header` + `Gen_P011` |
| **โครง** | `buy: []` · `get: [1]` ต่อ 1 แถว `MATERIALS[*]` |
| **payload หลังบ้านหลัก** | `layout-ab-payload.example.json` |
| **ชื่อฟิลด์ AB อธิบายเพิ่ม** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Header_Shared.md` · `MerC_to_AB_Function_Split.md` |
| **ต่างจาก P001** | มี online เมื่อ P4 · รับ reference ได้แต่ไม่เขียน |

**อ่านยังไง:** ไฟล์นี้เป็น **สเปกครบทุก key ใน AB payload** สำหรับ P011 — แต่ละแถวบอกว่า map จาก C อย่างไร / ห้ามใส่ / เว้นว่าง / Auto  
ต้นทาง = JSON `LAYOUT: "C"` (ดู `layout-c-payload.example.json`)  
ปลายทาง = JSON `LAYOUT: "AB"`

**กติกา source of truth:** ถ้า key/shape ในไฟล์นี้ขัดกับ `layout-ab-payload.example.json` ให้ **ยึด payload หลังบ้านก่อน** แล้วใช้ไฟล์นี้เป็นคู่มือ logic ราย profile


**สัญลักษณ์คอลัมน์ การทำ**

| สัญลักษณ์ | ความหมาย |
|-----------|----------|
| **map** | แปลงจาก C → AB |
| **copy** | ส่งตรง (อาจมี alias ชื่อฟิลด์) |
| **calc** | คำนวณ / lookup / normalize |
| **omit** | profile นี้ **ไม่ใส่ key** นี้ |
| **auto** | ระบบ AB คำนวณเอง (Auto? = Y ใน To-Be) |
| **empty** | ใส่ `[]` หรือ omit ได้ |
| **ctx** | มาจาก context/header ไม่ใช่แถว material โดยตรง |


---

## 0) Pipeline P011

```
convertLayoutCToAbPayload(c)
  normalizeC(c)
  HEADER  = mapHeaderShared(c.HEADER)
  BONUSBUYS = c.MATERIALS
    .filter(shouldConvertRow)
    .filter(m => shortCode(m.bonusBuyProfile) === "P011")
    .map(m => buildGroupA(ctx))         // get[1] only
  CONDITIONS = mapConditionsShared(c.CONDITIONS)
  MATERIALS  = []
  LAYOUT = "AB"
  STATUS = c.STATUS
```

1 แถว `MATERIALS[i]` (profile P011) → 1 element `BONUSBUYS[i]`

---

## 1) Skip rules (ก่อนแปลง)

| เงื่อนไขใน C | ผล |
|--------------|-----|
| `MATERIALS[*].status` = `Reject` | ข้ามแถว |
| `MATERIALS[*].convertStatus` = `Done` | ไม่แปลงซ้ำ |
| `MATERIALS[*].mechanic` = `B1G1 (On Pack)` หรือ `B2G1 (On Pack)` | ข้ามทั้งกลุ่ม |
| `bonusBuyProfile` ≠ `P011` | ไม่เข้า router นี้ |
| profile ไม่อยู่ใน 9 ตัวลูกค้า | reject |

---

## 2) ระดับบนสุด

| จาก C (JSON) | → AB (JSON) | การทำ | เงื่อนไข P011 |
|--------------|-------------|-------|----------------|
| — | `LAYOUT` | **calc** | คงที่ `"AB"` |
| `STATUS` | `STATUS` | **copy** | ส่งตรง |
| `HEADER` | `HEADER` | **map** | §3 |
| `MATERIALS[*]` | `BONUSBUYS[*]` | **calc** | สร้างใหม่ §4–§7 · buildGroupA — get only |
| `CONDITIONS` | `CONDITIONS` | **copy** | §8 |
| `BONUSBUYS` (C) | — | **omit** | C มักเป็น `[]` — ไม่ใช้ |
| `MATERIALS` (AB) | `MATERIALS` | **empty** | `[]` |
| — | `SUPPLIERFILE` | **empty** | `null` ได้ถ้าต้องคง shape ตาม sample |

---


## 3) HEADER (`MerAB - Header`)

ใช้ `mapHeaderShared` — **ไม่ใส่ if ราย profile ใน util นี้**  
รายละเอียด util: `MerC_to_AB_Header_Shared.md`

| จาก C (JSON) | → AB `HEADER` | ชื่อใน To-Be | การทำ | สูตร / เงื่อนไข |
|--------------|---------------|--------------|-------|------------------|
| `HEADER.group` | `group` | กลุ่มงาน / Group | **copy** | ส่งตรง |
| `HEADER.promotionName` | `promotionName` | Promotion Name | **copy** | ส่งตรง |
| `HEADER.purchasingGroup` | `purchasingGroup` | Purchasing Group | **copy** / lookup | รหัส → ชื่อเต็ม (optional) |
| `HEADER.theme` | `theme` | Theme | **copy** / lookup | รหัส 4 ตัว → ชื่อเต็ม (optional) |
| `HEADER.bonusBuyProfile` | `bonusBuyProfile` | Bonus Buy Profile | **calc** | `shortCode` → `"P011"` |
| `HEADER.rebateChargeback` | `rebateChargeback` | Rebate Chargeback | **calc** | `normalizeRebate` — ตัดตัวแรก; ไม่ใช่ Z2 และตัวแรก ≠ `A` → เติม `0` |
| `HEADER.contractType` | `contractType` | Contract Type | **calc** | `normalizeContractType` — ขึ้นต้น `Z2` ถึงใส่ |
| `HEADER.wbsNumber` / `wbsNo` | `wbsNumber` | WBS No. | **copy** | สำรอง `wbsNo` |
| `HEADER.vendorCode` | `vendorCode` | Vendor | **calc** | `vendorOrNobp` — ว่าง → `"NOBP"` |
| `HEADER.vendorName` | `vendorName` | Vendor (ชื่อ) | **copy** | ส่งตรง |
| `HEADER.periodFrom` | `periodFrom` | จัดรายการ … Start | **copy** | คง `YYYY-MM-DD` ตาม backend payload |
| `HEADER.periodTo` | `periodTo` | จัดรายการ … End | **copy** | คง `YYYY-MM-DD` ตาม backend payload |
| `HEADER.days` | `days` | วันจัดรายการ | **copy** | เช่น `["All"]` |
| `HEADER.singleMultiple` | `singleMultiple` | Single / Multiple | **copy** | ส่งตรง |
| `HEADER.volume` / `vol` | `volume` | — | **copy** | UI/เอกสาร |
| `HEADER.status` | — | — | **omit** | backend payload sample ไม่มี |
| `HEADER.timeFrom` | `timeFrom` | — | **copy** | คงไว้ใน `HEADER` และใช้สร้าง `bonusBuyHeader.validTimeFrom` ตามกติกา profile |
| `HEADER.timeTo` | `timeTo` | — | **copy** | คงไว้ใน `HEADER` และใช้สร้าง `bonusBuyHeader.validTimeTo` ตามกติกา profile |

---

## 4) `BONUSBUYS[*]` — โครงก้อน

| AB block | P011 |
|----------|------|
| `lineNumber` | **copy** จาก `MATERIALS[*].lineNumber` (optional) |
| `bonusBuyNumber` | **copy** ซ้ำระดับ root (optional) |
| `bonusBuyHeader` | §5 |
| `buy[]` | `[]` เสมอ |
| `get[]` | ความยาว `[1]` — §7 |
| `stores[]` | §7.1 |
| `card[]` | **empty** |
| `tender[]` | **empty** |
| `installment[]` | **empty** |
| `posTerminal[]` | **empty** |
| `premium[]` | **empty** |
| `coupon[]` | **empty** |
| `limitControl[]` | **empty** |

---

## 5) `bonusBuyHeader` (r31–r53)

| จาก C (JSON) | → AB (JSON) | ชื่อใน To-Be | req | การทำ | เงื่อนไข P011 |
|--------------|-------------|--------------|-----|-------|----------------|
| `MATERIALS[*].numberOfBonusBuy` / `noof_bonus_buy` | `bonusBuyNumber` | Bonus buy No. | M | **copy** | default `"1"` |
| `MATERIALS[*].bonusBuyProfile` / `bonus_buy_profile` | `bonusBuyProfile` | Bonus Buy Profile | — | **calc** | `"P011"` |
| `MATERIALS[*].mechanic` | `mechanic` | Mechanic | — | **copy** | + ใช้ lookup qty §10 |
| `HEADER.timeFrom` | `validTimeFrom` | Valid Time from | — | **calc** | **ใส่** — ว่าง → `00:00:00` |
| `HEADER.timeTo` | `validTimeTo` | Valid Time to | — | **calc** | **ใส่** — ว่าง → `23:59:59` |
| `HEADER.wbsNumber` / `wbsNo` | `wbsNumber` | WBS No. | — | **ctx** | copy จาก header |
| `HEADER.promotionArea` **หรือ** `MATERIALS[*].promotionArea` | `promotionArea` | Promotion Area | — | **copy** | fallback: header ก่อน แล้ว material |
| `HEADER.onlineDescriptionEnglish` **หรือ** `MATERIALS[*].onlineDescriptionEnglish` | `onlineDescriptionEnglish` | Online Description EN | — | **copy** | **เฉพาะ** `promotionArea = "P4"` |
| `HEADER.onlineDescriptionThai` **หรือ** `MATERIALS[*].onlineDescriptionThai` | `onlineDescriptionThai` | Online Description TH | — | **copy** | **เฉพาะ** `promotionArea = "P4"` |
| `HEADER.referenceCode` / `MATERIALS[*].referenceCode` | `referenceCode` | Reference code | — | **omit** | ต้นทางมีได้ แต่ P011 **ไม่เขียน** |
| — | `promotionNumber` | Promotion No. | — | **auto** | |
| — | `description` | Description | — | **auto** | prefix + ชื่อโปร (TBD macro) |
| — | `purchasingGroup` | Purchasing Group | — | **auto** | |
| — | `limitNumber` | Limit No. | — | **omit** | ไม่ใช้รอบนี้ |
| — | `product` | Product | — | **auto** | |
| — | `priceTag` | Price Tag | — | **omit** | |
| — | `referenceBonusBuy` | Ref. Bonus buy | — | **omit** | |
| — | `legacyPromotionNumber` | Legacy Promotion No. | — | **omit** | |
| — | `allowDiscountAfterGetExtraMPoint` | Allow discount after… | — | **omit** | |
| — | `notAcceptAnyDiscountCoupon` | Not accept coupon | — | **omit** | |
| — | `notAcceptAnyDiscountCard` | Not accept card | — | **omit** | |
| — | `notAllowForEmployee` | Not allow employee | — | **omit** | |
| — | `department` | Department | — | **auto** | |

---

## 6) `buy[]` (r54–r69)

P011 เป็น **Get-only** → `buy = []` เสมอ  
field ใน buy ทั้งหมด = **ไม่ใช้** สำหรับ profile นี้

| จาก C (JSON) | → AB `buy[0]` | ชื่อใน To-Be | req | การทำ | เงื่อนไข P011 |
|--------------|---------------|--------------|-----|-------|----------------|
| — | ทั้งก้อน `buy` | Buy block | — | **empty** | `[]` |

---

## 7) `get[0]` (r70–r106)

| จาก C (JSON) | → AB `get[0]` | ชื่อใน To-Be | req | การทำ | เงื่อนไข P011 |
|--------------|---------------|--------------|-----|-------|----------------|
| `numberOfBonusBuy` / `noof_bonus_buy` | `bonusBuyNumber` | Bonus Buy No. | **M** | **copy** | |
| มีกลุ่ม? | `field2` | ประเภท MAT / Group | **M** | **calc** | มีกลุ่ม → `"Material Group"` · ไม่มี → `"Material"` |
| `material` / `materialGroupName` | `field4` | รหัส / ชื่อกลุ่ม | **M** | **calc** | ตาม type |
| `materialDescription` / `material_des` | `field5` | Description | — | **copy** | optional |
| `materialDescription2` / `material_en_des` | `sapMasterDescription` | SAP desc | — | **copy** | optional |
| `costNormal` | `field7` | ราคาทุน | — | **copy** | optional |
| `salesPriceNormal` / `sales_price_normal` | `field8` | **ราคาขายปกติ** | — | **copy** | **ใส่** (กลุ่ม A) |
| — | `vat` | VAT | — | **omit** | |
| — | `grossProfit` | GP% ปกติ | — | **auto** | |
| `mechanic` | `getQuantity` | Get Qty | **M** | **calc** | lookup Mechanic → get qty |
| — | `tierNumber` … `tierAmountB` | Tier | — | **omit** | |
| `costPromotion` / `cost_promotion` | `field17` | ราคาทุนจัดรายการ | — | **copy** | optional |
| `salesPricePromotion` / `sales_price_promo` | `fieldP` | ราคาจัดรายการ [P] | — | **calc** | `pickDiscount` ลำดับ 1 |
| — | `vat2` | VAT | — | **omit** | |
| — | `grossProfit2` | GP% ใหม่ | — | **auto** | |
| `discountAmount` / `discountAmt` | `fieldR` | ส่วนลดบาท [R] | — | **calc** | `pickDiscount` ลำดับ 2 |
| `discountPercentPlu` / `discountPercentForP015` / `discountPct` | `field22` | ส่วนลด % [%] | — | **calc** | ลำดับ 3 + `normalizePctFull` |
| — | `newGrossProfitRate` | New GP Rate | — | **omit** | |
| `barcode` | `ean` | EAN | — | **copy** | optional |
| — | `serialNumber` | Serial No. | — | **omit** | |
| `salesUnit` | `unit` | Unit | **M** | **copy** | |
| — | `priceUnit` / `unitOfMeasure` | Price unit / UOM | — | **omit** | |
| — | `basicPoint` … `pointAmount` | Point | — | **omit** | |
| — | `exclusion` | Exclusion | — | **omit** | |
| — | `noDiscount` | No discount | — | **auto** | On Top X=0.01% → mark X |
| `promotionTag` | `promotionTagSizeA4Cut1..6` | Pro Tag | — | **omit** | |

### 7.1 `stores[]`

| จาก C (JSON) | → AB | การทำ | เงื่อนไข |
|--------------|------|-------|----------|
| `MATERIALS[*].stores` | `BONUSBUYS[*].stores` | **copy** / แปลงรหัส | เช่น `store_13_ka` → `"13KA"` |
| ไม่มีค่า | `stores` | **empty** | `[]` |

รหัส plant อ้างอิง: `MerC_to_AB_ExistsInAB_Mapping.md` §5.2

---


## 8) `CONDITIONS[]`

ใช้เมื่อ `contractType` ขึ้นต้น `Z2` / `Z3`  
รอบแรก: **copy + compact** จาก C — ไม่ gen ใหม่จาก MATERIALS

| AB block | การทำ |
|----------|-------|
| `conditionHeader` | **copy** |
| `businessVolumePurchase[]` | **copy** |
| `businessVolumeSales[]` | **copy** |
| `conditionType[]` | **copy** |
| `settlementCalendar[]` | **copy** |
| `combineCheck[]` | **copy** |
| `allocation[]` | **copy** |

Compensate / Settlement / Payment ใน Mer C → อยู่ใน CONDITIONS ไม่ใช่ BBY line

---


## 9) `MATERIALS[]` ฝั่ง AB

| จาก C | → AB |
|--------|------|
| `MATERIALS[*]` (forecast, planogram, GP, …) | `MATERIALS: []` |

ข้อมูล planogram / forecast ใน C **ไม่เข้า BBY** ในรอบนี้

---

## 10) สูตรและ helper (P011)

### 10.1 กฎเฉพาะ profile (สรุป)

| หัวข้อ | ค่า P011 |
|--------|----------|
| เขียนฝั่ง | `get[1]` เท่านั้น |
| `get.field8` | **ใส่** |
| Normalize % | **เต็ม** |
| `validTimeFrom` / `validTimeTo` | **ใส่** |
| `referenceCode` | รับได้แต่ **ไม่เขียน** |
| Online EN/TH | **ใส่เมื่อ** `promotionArea = "P4"` |

### 10.2 `materialType` + `materialOrGroup`

```
ถ้ามี materialGroupName หรือ numberOfMaterialGrouping ชี้กลุ่ม:
  field2 = "Material Group"
  field4 = materialGroupName
ไม่งั้น:
  field2 = "Material"
  field4 = material
```

### 10.3 `pickDiscount` — เลือกช่องเดียว

| ลำดับ | จาก C | → AB `get[0]` | เงื่อนไข |
|------|--------|---------------|----------|
| 1 | `salesPricePromotion` / `sales_price_promo` | `fieldP` | มีค่า → ใช้อันนี้อย่างเดียว |
| 2 | `discountAmount` / `discountAmt` | `fieldR` | เมื่อไม่มีข้อ 1 |
| 3 | `discountPercentPlu` / `discountPercentForP015` / `discountPct` | `field22` | เมื่อไม่มี 1 และ 2 · ใช้ `normalizePctFull` |

### 10.4 `normalizePctFull`

```
ถ้า (% / 100) < 1  → ใช้ค่าเดิม     (10 → 10)
ถ้า % > 100        → % / 100       (1668 → 16.68)
ถ้า % < 1          → % * 100       (0.15 → 15)
```

### 10.5 `lookupBuyGetQty(mechanic)`

→ `get.getQuantity` จากชีต Mechanic  
(กลุ่ม A ไม่เขียน buy qty)


### 10.6 Alias ชื่อฟิลด์ C (normalizeC)

| camelCase | snake_case ที่พบใน fixture |
|-----------|----------------------------|
| `salesPriceNormal` | `sales_price_normal` |
| `salesPricePromotion` | `sales_price_promo` |
| `bonusBuyProfile` | `bonus_buy_profile` |
| `numberOfBonusBuy` | `noof_bonus_buy` |
| `materialDescription` | `material_des`, `material_th_des` |
| `materialDescription2` | `material_des_2`, `material_en_des` |
| `discountAmount` | `discountAmt` |

---

## 11) เคสทดสอบ

| # | จุดตรวจ |
|---|--------|
| 1 | ส่วนลด promo → `get.fieldP` อย่างเดียว |
| 2 | ส่วนลดบาท → `get.fieldR` อย่างเดียว |
| 3 | ส่วนลด % → `get.field22` หลัง normalize |
| 4 | `HEADER.vendorCode` ว่าง → `NOBP` |
| 5 | Material Group → `field2` / `field4` |

---

## 12) Skeleton JSON (`BONUSBUYS[]` element)

```json
{
  "lineNumber": 0,
  "bonusBuyNumber": "1",
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "P011",
    "mechanic": "Discount",
    "validTimeFrom": "08:30",
    "validTimeTo": "22:00",
    "promotionArea": "All",
    "wbsNumber": "WBS-2026-002"
    "onlineDescriptionEnglish": "…",
    "onlineDescriptionThai": "…",
  },
  "buy": [],
  "get": [
    {
      "bonusBuyNumber": "1",
      "field2": "Material",
      "field4": "MAT-100",
      "field5": "Get item description",
      "field8": "129.00",
      "getQuantity": "1",
      "fieldP": "99.00",
      "unit": "EA"
    }
  ],
  "stores": [],
  "card": [],
  "tender": [],
  "installment": [],
  "posTerminal": [],
  "premium": [],
  "coupon": [],
  "limitControl": []
}
```

---

## 13) Checklist (P011 ครบ)

### โครง
- [ ] `LAYOUT` = `"AB"`
- [ ] 1 แถว C → 1 `BONUSBUYS` element
- [ ] `buy = []` · มี `get[0]`
- [ ] `MATERIALS` ฝั่ง AB = `[]`

### HEADER
- [ ] วันที่ใน `HEADER.periodFrom/periodTo` เป็น `YYYY-MM-DD`
- [ ] `vendorCode` ว่าง → `NOBP`
- [ ] มี `timeFrom`/`timeTo` ใน AB `HEADER`

### bonusBuyHeader / get
- [ ] `buy = []`
- [ ] `get.field8` มีค่า
- [ ] มี `validTime*`
- [ ] **ไม่มี** `referenceCode` ใน output
- [ ] online เฉพาะ P4
- [ ] % normalize เต็ม
- [ ] ส่วนลด **ช่องเดียว**: P หรือ R หรือ field22
- [ ] Get Mandatory: `bonusBuyNumber`, `field2`, `field4`, `getQuantity`, `unit`

### อื่นๆ
- [ ] `CONDITIONS` copy จาก C
- [ ] `card/tender/…` = `[]`
- [ ] skip rules §1 ทำงาน

---


## 14) สิ่งที่ยัง TBD ในโค้ด (ไม่ใช่ลืม map)

| หัวข้อ | หมายเหตุ |
|--------|----------|
| `lookupBuyGetQty` | fixture มัก hardcode `1` |
| `normalizeRebate` เต็ม | บางเคส clear เป็น `""` |
| `bonusBuyHeader.description` | macro prefix |
| theme / purchasingGroup lookup | optional |
| Pro Tag, Tier, Point | out of scope รอบนี้ |

---

## 15) การยืนยัน key (audit)

เทียบ: `payload_AB_Reference.md` · `Gen_P011` / `P011_Header` ใน `Mer-C_Convert_To_STD.txt` · `layout-ab-payload.example.json`

### ✅ Core P011

| หัวข้อ | สถานะ |
|--------|--------|
| Get-only (`buy=[]`) | ✅ ตาม macro กลุ่ม A |
| `get.field8` ใส่ | ✅ ตามกลุ่ม A |
| ส่วนลดช่องเดียว + normalize ตามตาราง | ✅ |
| validTime / referenceCode / online ตามตาราง §10.1 | ✅ ตาม macro header |
| AB key name ตาม payload | ✅ |
| Golden test ครบทุก key | ❌ ยังไม่มี |
