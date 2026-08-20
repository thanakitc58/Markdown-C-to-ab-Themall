# C → AB — Profile D001 (Full)

| | |
|--|--|
| **กลุ่ม** | B — Buy + Get แถวเดียว |
| **Macro** | `D001_Header` + `Gen_D001` |
| **โครง** | `buy: [1]` · `get: [1]` ต่อ 1 แถว `MATERIALS[*]` |
| **payload หลังบ้านหลัก** | `layout-ab-payload.example.json` |
| **ชื่อฟิลด์ AB อธิบายเพิ่ม** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Header_Shared.md` · `MerC_to_AB_Function_Split.md` |
| **fixture / ใบคีย์** | `tungconvert/D001/` · `tungconvert/D001/KEYING-SHEET-D001.md` |

**อ่านยังไง:** ไฟล์นี้เป็น **สเปกครบทุก key ใน AB payload** สำหรับ D001 — แต่ละแถวบอกว่า map จาก C อย่างไร / ห้ามใส่ / เว้นว่าง / Auto  
ต้นทาง = JSON `LAYOUT: "C"` (ดู `layout-c-payload.example.json`)  
ปลายทาง = JSON `LAYOUT: "AB"`

**กติกา source of truth:** ถ้า key/shape ในไฟล์นี้ขัดกับ `layout-ab-payload.example.json` ให้ **ยึด payload หลังบ้านก่อน** แล้วใช้ไฟล์นี้เป็นคู่มือ logic ราย profile

**สัญลักษณ์คอลัมน์ การทำ (D001)**

| สัญลักษณ์ | ความหมาย |
|-----------|----------|
| **map** | แปลงจาก C → AB |
| **copy** | ส่งตรง (อาจมี alias ชื่อฟิลด์) |
| **calc** | คำนวณ / lookup / normalize |
| **omit** | D001 **ไม่ใส่ key** นี้ |
| **auto** | ระบบ AB คำนวณเอง (Auto? = Y ใน To-Be) |
| **empty** | ใส่ `[]` หรือ omit ได้ |
| **ctx** | มาจาก context/header ไม่ใช่แถว material โดยตรง |

---

## 0) Pipeline D001

```
convertLayoutCToAbPayload(c)
  normalizeC(c)                         // alias snake_case → camelCase
  HEADER  = mapHeaderShared(c.HEADER)   // ของร่วม — ดู §2
  BONUSBUYS = c.MATERIALS
    .filter(shouldConvertRow)           // §1 skip rules
    .filter(m => shortCode(m.bonusBuyProfile) === "D001")
    .map(m => buildGroupB(ctx))         // buy[1] + get[1]
  CONDITIONS = mapConditionsShared(c.CONDITIONS)  // §8 — copy + compact
  MATERIALS  = []                       // §9
  LAYOUT = "AB"
  STATUS = c.STATUS
```

1 แถว `MATERIALS[i]` (profile D001) → 1 element `BONUSBUYS[i]`

---

## 1) Skip rules (ก่อนแปลง)

| เงื่อนไขใน C | ผล |
|--------------|-----|
| `MATERIALS[*].status` = `Reject` | ข้ามแถว |
| `MATERIALS[*].convertStatus` = `Done` | ไม่แปลงซ้ำ |
| `MATERIALS[*].mechanic` = `B1G1 (On Pack)` หรือ `B2G1 (On Pack)` | ข้ามทั้งกลุ่ม |
| `bonusBuyProfile` ≠ `D001` | ไม่เข้า router นี้ |
| profile ไม่อยู่ใน 9 ตัวลูกค้า | reject |

---

## 2) ระดับบนสุด

| จาก C (JSON) | → AB (JSON) | การทำ | เงื่อนไข D001 |
|--------------|-------------|-------|----------------|
| — | `LAYOUT` | **calc** | คงที่ `"AB"` |
| `STATUS` | `STATUS` | **copy** | ส่งตรง |
| `HEADER` | `HEADER` | **map** | §3 |
| `MATERIALS[*]` | `BONUSBUYS[*]` | **calc** | สร้างใหม่ §4–§7 |
| `CONDITIONS` | `CONDITIONS` | **copy** | §8 |
| `BONUSBUYS` (C) | — | **omit** | C มักเป็น `[]` — ไม่ใช้ |
| `MATERIALS` (AB) | `MATERIALS` | **empty** | `[]` |
| `SUPPLIERFILE` | — | **omit** | นอก scope |

---

## 3) HEADER (`MerAB - Header`)

ใช้ `mapHeaderShared` — **ไม่ใส่ if D001 ใน util นี้**  
รายละเอียด util: `MerC_to_AB_Header_Shared.md`

| จาก C (JSON) | → AB `HEADER` | ชื่อใน To-Be | การทำ | สูตร / เงื่อนไข |
|--------------|---------------|--------------|-------|------------------|
| `HEADER.group` | `group` | กลุ่มงาน / Group | **copy** | ส่งตรง |
| `HEADER.promotionName` | `promotionName` | Promotion Name | **copy** | ส่งตรง |
| `HEADER.purchasingGroup` | `purchasingGroup` | Purchasing Group | **copy** / lookup | รหัส → ชื่อเต็ม (optional) |
| `HEADER.theme` | `theme` | Theme | **copy** / lookup | รหัส 4 ตัว → ชื่อเต็ม (optional) |
| `HEADER.bonusBuyProfile` | `bonusBuyProfile` | Bonus Buy Profile | **calc** | `shortCode` → `"D001"` |
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
| `HEADER.timeFrom` | `timeFrom` | — | **copy** | คงไว้ใน `HEADER` และใช้สร้าง `bonusBuyHeader.validTimeFrom` |
| `HEADER.timeTo` | `timeTo` | — | **copy** | คงไว้ใน `HEADER` และใช้สร้าง `bonusBuyHeader.validTimeTo` |

---

## 4) `BONUSBUYS[*]` — โครงก้อน

| AB block | D001 |
|----------|------|
| `bonusBuyHeader` | §5 |
| `buy[]` | ความยาว `[1]` — §6 |
| `get[]` | ความยาว `[1]` — §7 |
| `stores[]` | §7.1 |
| `card[]` | **empty** |
| `tender[]` | **empty** |
| `installment[]` | **empty** |
| `posTerminal[]` | **empty** |
| `premium[]` | **empty** |
| `coupon[]` | **empty** |
| `limitControl[]` | **empty** |
| `lineNumber` | **copy** | จาก `MATERIALS[*].lineNumber` (optional) |
| `numberOfBonusBuy` | `bonusBuyNumber` | **copy** | ซ้ำระดับ root (optional — ตาม `layout-ab-payload.example.json`) |

---

## 5) `bonusBuyHeader` (r31–r53)

| จาก C (JSON) | → AB (JSON) | ชื่อใน To-Be | req | การทำ | เงื่อนไข D001 |
|--------------|-------------|--------------|-----|-------|----------------|
| `MATERIALS[*].numberOfBonusBuy` / `noof_bonus_buy` | `bonusBuyNumber` | Bonus buy No. | M | **copy** | default `"1"` |
| `MATERIALS[*].bonusBuyProfile` / `bonus_buy_profile` | `bonusBuyProfile` | Bonus Buy Profile | — | **calc** | `"D001"` |
| `MATERIALS[*].mechanic` | `mechanic` | Mechanic | — | **copy** | + ใช้ lookup qty §10 |
| `HEADER.timeFrom` | `validTimeFrom` | Valid Time from | — | **calc** | **ใส่** — ว่าง → `00:00:00` |
| `HEADER.timeTo` | `validTimeTo` | Valid Time to | — | **calc** | **ใส่** — ว่าง → `23:59:59` |
| `HEADER.wbsNumber` / `wbsNo` | `wbsNumber` | WBS No. | — | **ctx** | copy จาก header |
| `HEADER.promotionArea` **หรือ** `MATERIALS[*].promotionArea` | `promotionArea` | Promotion Area | — | **copy** | fallback: header ก่อน แล้ว material |
| `HEADER.onlineDescriptionEnglish` **หรือ** `MATERIALS[*].onlineDescriptionEnglish` | `onlineDescriptionEnglish` | Online Description EN | — | **copy** | **เฉพาะ** `promotionArea = "P4"` |
| `HEADER.onlineDescriptionThai` **หรือ** `MATERIALS[*].onlineDescriptionThai` | `onlineDescriptionThai` | Online Description TH | — | **copy** | **เฉพาะ** `promotionArea = "P4"` |
| `MATERIALS[*].referenceCode` | `referenceCode` | Reference code | — | **omit** | D001 **ไม่ใส่** |
| — | `promotionNumber` | Promotion No. | — | **auto** | |
| — | `description` | Description | — | **auto** | prefix + ชื่อโปร (TBD macro) |
| — | `purchasingGroup` | Purchasing Group | — | **auto** | |
| — | `limitNumber` | Limit No. | — | **omit** | ไม่ใช้ D001 รอบนี้ |
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

## 6) `buy[0]` (r54–r69)

| จาก C (JSON) | → AB `buy[0]` | ชื่อใน To-Be | req | การทำ | เงื่อนไข D001 |
|--------------|---------------|--------------|-----|-------|----------------|
| `numberOfBonusBuy` / `noof_bonus_buy` | `bonusBuyNumber` | Bonus Buy No. | — | **copy** | |
| มี `materialGroupName` / `materialGroup` / `numberOfMaterialGrouping`? | `field2` | ประเภท MAT / Group | — | **calc** | มีกลุ่ม → `"Material Group"` · ไม่มี → `"Material"` |
| `material` (MAT) / `materialGroupName` (MGP) | `field4` | รหัส / ชื่อกลุ่ม | — | **calc** | ตาม type |
| `materialDescription` / `material_des` / `material_th_des` | `description` | Description | — | **copy** | optional |
| `materialDescription2` / `material_en_des` / `material_des_2` | `sapMasterDescription` | Long Thai / SAP desc | — | **copy** | optional |
| `costNormal` / `cost_normal` | `field7` | ราคาทุน (ไม่รวม VAT) | — | **copy** | optional |
| `salesPriceNormal` / `sales_price_normal` | `field8` | **ราคาขายปกติ** | — | **copy** | ใส่ฝั่ง buy ได้ |
| `mechanic` | `field9` | จำนวนชิ้นขั้นต่ำ | — | **calc** | `lookupBuyGetQty` → buy qty |
| — | `field10` | มูลค่าขั้นต่ำ | — | **omit** | |
| `barcode` | `ean` | EAN | — | **copy** | optional |
| — | `serial` | Serial | — | **omit** | |
| `salesUnit` | `salesUnit` | Sales unit | — | **copy** | |
| `promotionTag` | `promotionTagSizeA4Cut1..6` | Pro Tag A4 | — | **omit** | รอบหลัง |
| — | `char` | characteristic | — | **omit** | ไม่มีแถว To-Be |

---

## 7) `get[0]` (r70–r106)

| จาก C (JSON) | → AB `get[0]` | ชื่อใน To-Be | req | การทำ | เงื่อนไข D001 |
|--------------|---------------|--------------|-----|-------|----------------|
| `numberOfBonusBuy` / `noof_bonus_buy` | `bonusBuyNumber` | Bonus Buy No. | **M** | **copy** | |
| มีกลุ่ม? | `field2` | ประเภท MAT / Group | **M** | **calc** | เหมือน buy |
| `material` / `materialGroupName` | `field4` | รหัส / ชื่อกลุ่ม | **M** | **calc** | เหมือน buy |
| `materialDescription` / `material_des` | `field5` | Description | — | **copy** | optional |
| `materialDescription2` / `material_en_des` | `sapMasterDescription` | SAP desc | — | **copy** | optional |
| `costNormal` | `field7` | ราคาทุน | — | **copy** | optional |
| `salesPriceNormal` / `sales_price_normal` | `field8` | **ราคาขายปกติ** | — | **omit** | **D001 ห้ามใส่** |
| — | `vat` | VAT | — | **omit** | |
| — | `grossProfit` | GP% ปกติ | — | **auto** | |
| `mechanic` | `getQuantity` | Get Qty | **M** | **calc** | `lookupBuyGetQty` → get qty |
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

| AB block | การทำ D001 |
|----------|------------|
| `conditionHeader` | **copy** |
| `businessVolumePurchase[]` | **copy** |
| `businessVolumeSales[]` | **copy** |
| `conditionType[]` | **copy** |
| `settlementCalendar[]` | **copy** |
| `combineCheck[]` | **copy** |
| `allocation[]` | **copy** |

Compensate / Settlement / Payment ใน Mer C (col BG–BY) → อยู่ใน CONDITIONS / contract block ไม่ใช่ BBY line

---

## 9) `MATERIALS[]` ฝั่ง AB

| จาก C | → AB |
|--------|------|
| `MATERIALS[*]` (forecast, planogram, GP, …) | `MATERIALS: []` |

ข้อมูล planogram / forecast ใน C **ไม่เข้า BBY** ในรอบนี้

---

## 10) สูตรและ helper (D001)

### 10.1 กฎเฉพาะ profile (สรุป)

| หัวข้อ | ค่า D001 |
|--------|----------|
| เขียนฝั่ง | `buy[1]` + `get[1]` |
| `get.field8` | **omit** |
| Normalize % | **เต็ม** (`normalizePctFull`) |
| `validTimeFrom` / `validTimeTo` | **ใส่** |
| `referenceCode` | **omit** |
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
| 3 | `discountPercentPlu` / `discountPercentForP015` / `discountPct` | `field22` | เมื่อไม่มี 1 และ 2 |

### 10.4 `normalizePctFull` (D001)

```
ถ้า (% / 100) < 1  → ใช้ค่าเดิม     (10 → 10)
ถ้า % > 100        → % / 100       (1668 → 16.68)
ถ้า % < 1          → % * 100       (0.15 → 15)
```

### 10.5 `lookupBuyGetQty(mechanic)`

→ `{ buyQty, getQty }` จากชีต Mechanic  
→ `buy.field9` และ `get.getQuantity`  
default fixture ใช้ `1` — **โค้ดจริงต้อง lookup**

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

## 11) เคสทดสอบ (fixture)

| # | ไฟล์ | จุดตรวจ |
|---|------|---------|
| 1 | `tungconvert/D001/1-D001-promo.json` | `get.fieldP` · ไม่มี `fieldR`/`field22` |
| 2 | `tungconvert/D001/2-D001-amt.json` | `get.fieldR` = 50 |
| 3 | `tungconvert/D001/3-D001-pct.json` | `get.field22` = 16.68 |
| 4 | `tungconvert/D001/4-D001-nobp.json` | `HEADER.vendorCode` = `NOBP` |
| 5 | `tungconvert/D001/5-D001-mgp.json` | `field2` = MGPNew · `field4` = ชื่อกลุ่ม |
| 6 | `tungconvert/D001/6-D001-p4-online.json` | online EN/TH เมื่อ P4 |
| 7 | `tungconvert/D001/7-D001-pct-normalize.json` | 1668 → 16.68 |

---

## 12) Skeleton JSON (`BONUSBUYS[]` element)

```json
{
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "D001",
    "mechanic": "1A Get 1B (A)",
    "validTimeFrom": "00:00:00",
    "validTimeTo": "23:59:59",
    "wbsNumber": "AP.26.8883.10.CP.01"
  },
  "buy": [
    {
      "bonusBuyNumber": "1",
      "field2": "Material",
      "field4": "1000473882",
      "description": "โอเลย์ …",
      "field8": "1199",
      "field9": 1,
      "ean": "4987176041340",
      "salesUnit": "EA"
    }
  ],
  "get": [
    {
      "bonusBuyNumber": "1",
      "field2": "Material",
      "field4": "1000473882",
      "getQuantity": 1,
      "unit": "EA",
      "fieldP": "1199"
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

## 13) Checklist (D001 ครบ)

### โครง
- [ ] `LAYOUT` = `"AB"`
- [ ] 1 แถว C → 1 `BONUSBUYS` element
- [ ] มีทั้ง `buy[0]` และ `get[0]`
- [ ] `MATERIALS` ฝั่ง AB = `[]`

### HEADER
- [ ] วันที่ใน `HEADER.periodFrom/periodTo` เป็น `YYYY-MM-DD`
- [ ] `vendorCode` ว่าง → `NOBP`
- [ ] `rebateChargeback` / `contractType` ตาม shared
- [ ] มี `timeFrom`/`timeTo` ใน AB `HEADER`

### bonusBuyHeader
- [ ] `bonusBuyProfile` = `D001`
- [ ] `validTimeFrom` / `validTimeTo` มีค่า
- [ ] **ไม่มี** `referenceCode`
- [ ] online EN/TH เฉพาะ P4

### buy / get
- [ ] `buy.field9` จาก mechanic
- [ ] `get.getQuantity` จาก mechanic
- [ ] `get` **ไม่มี** `field8`
- [ ] ส่วนลด **ช่องเดียว**: P หรือ R หรือ field22
- [ ] % ใช้ normalize **เต็ม**

### Get Mandatory (To-Be)
- [ ] `bonusBuyNumber`, `field2`, `field4`, `getQuantity`, `unit`

### อื่นๆ
- [ ] `CONDITIONS` copy จาก C
- [ ] `card/tender/…` = `[]`
- [ ] skip rules §1 ทำงาน

---

## 14) สิ่งที่ยัง TBD ในโค้ด (ไม่ใช่ลืม map)

| หัวข้อ | หมายเหตุ |
|--------|----------|
| `lookupBuyGetQty` | fixture hardcode `1` |
| `normalizeRebate` เต็ม | บาง fixture clear เป็น `""` |
| `bonusBuyHeader.description` | macro prefix |
| theme / purchasingGroup lookup | optional |
| Pro Tag, Tier, Point | out of scope รอบนี้ |

---

## 15) การยืนยัน key (audit)

เทียบ 3 แหล่ง: `payload_AB_Reference.md` · `Gen_D001`/`D001_Header` ใน `Mer-C_Convert_To_STD.txt` · fixture `tungconvert/D001/`

### ✅ AB key หลัก — ตรง contract + VBA

| AB key | ชื่อ To-Be | VBA cell | Gen_D001 เขียน? |
|--------|------------|----------|-----------------|
| `bonusBuyHeader.bonusBuyNumber` | Bonus buy No. | T | ✅ |
| `bonusBuyHeader.validTimeFrom` | Valid Time from | W | ✅ (D001_Header) |
| `bonusBuyHeader.validTimeTo` | Valid Time to | X | ✅ |
| `bonusBuyHeader.promotionArea` | Promotion Area | AA | ✅ |
| `bonusBuyHeader.wbsNumber` | WBS No. | AB | ✅ |
| `bonusBuyHeader.mechanic` | Mechanic | — | ✅ (จาก C ไม่ใช่ cell เดียว) |
| `bonusBuyHeader.onlineDescriptionEnglish` | Online EN | AK | ✅ เมื่อ P4 |
| `bonusBuyHeader.onlineDescriptionThai` | Online TH | AL | ✅ เมื่อ P4 |
| `buy.bonusBuyNumber` | Bonus Buy No. | AN | ✅ |
| `buy.field2` | ประเภท MAT/Group | AO | ✅ |
| `buy.field4` | รหัส/ชื่อกลุ่ม | AQ | ✅ |
| `buy.field9` | จำนวนชิ้นขั้นต่ำ | AV | ✅ (BuyAndGet[0]) |
| `get.bonusBuyNumber` | Bonus Buy No. | BF | ✅ |
| `get.field2` | ประเภท MAT/Group | BG | ✅ |
| `get.field4` | รหัส/ชื่อกลุ่ม | BI | ✅ |
| `get.fieldP` | ราคาจัดรายการ [P] | BW | ✅ (New_Price) |
| `get.fieldR` | ส่วนลดบาท [R] | BZ | ✅ |
| `get.field22` | ส่วนลด % | CA | ✅ + normalize |
| `get.getQuantity` | Get Qty | BP | ✅ (BuyAndGet[1]) |
| `get.unit` | Unit | CE | ✅ (Sale_Unit) |
| `get.field8` | ราคาขายปกติ | BM | ❌ **D001 ไม่เขียน** (ถูกต้อง) |
| `bonusBuyHeader.referenceCode` | Reference code | AI | ❌ **D001 ส่ง `""`** (ถูกต้อง) |

### ⚠️ อยู่ใน To-Be + fixture แต่ VBA `Gen_D001` ไม่เขียน

| AB key | ชื่อ To-Be | หมายเหตุ |
|--------|------------|----------|
| `buy.field8` | ราคาขาย (Buy r60) | fixture `1-D001-promo` map จาก `salesPriceNormal` — **ถูกตาม To-Be/ใบคีย์** แต่ macro เก่าไม่ใส่ |
| `buy.salesUnit` | Sales unit (r65) | fixture map จาก `salesUnit` — macro ใส่แค่ `get.unit` (CE) |
| `buy.description` / `buy.ean` | Description / EAN | optional ใน To-Be — fixture มี, macro ไม่มี |

→ ถ้าโฟกัส **เทียบ Excel macro 100%** = ตัด 3 แถวบนออก  
→ ถ้าโฟกัส **JSON contract To-Be + ใบคีย์** = **เก็บไว้ถูกแล้ว**

### ⚠️ ต้นทาง C — ใช้ fallback

| AB key | ต้นทาง C ที่ถูก |
|--------|----------------|
| `promotionArea` | `HEADER.promotionArea` **หรือ** `MATERIALS[*].promotionArea` |
| `onlineDescriptionEnglish` | `HEADER…` **หรือ** `MATERIALS[*]…` (fixture `6-D001-p4-online`) |
| `onlineDescriptionThai` | เหมือนด้านบน |
| ส่วนลด [P] | `salesPricePromotion` **หรือ** `sales_price_promo` (normalizeC ก่อน) |
| ราคาขาย buy | `salesPriceNormal` **หรือ** `sales_price_normal` |

### ✅ get ใช้ `field5` ไม่ใช่ `description`

To-Be แถว Get Description = key **`field5`** (r73) — ไม่ใช่ `description` (นั่นของ buy r57)

### สรุปความมั่นใจ

| ระดับ | สถานะ |
|-------|--------|
| **AB key name** ตาม `payload_AB_Reference.md` | ✅ ถูก |
| **Core D001** (buy+get, ส่วนลด, เวลา, ไม่มี get.field8) | ✅ ตรง VBA |
| **Optional buy** (field8, salesUnit, ean, description) | ✅ ตรง To-Be/fixture · ⚠️ เกิน macro |
| **Golden test ครบทุก key** | ❌ ยังไม่มี — มีแค่ fixture 7 เคส |
