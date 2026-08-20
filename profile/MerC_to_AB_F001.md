# C → AB — Profile F001 (Full)

| | |
|--|--|
| **กลุ่ม** | B — Buy + Get แถวเดียว |
| **Macro** | `F001_Header` + `Gen_F001` |
| **โครง** | `buy: [1]` · `get: [1]` ต่อ 1 แถว `MATERIALS[*]` |
| **payload หลังบ้านหลัก** | `layout-ab-payload.example.json` |
| **ชื่อฟิลด์ AB อธิบายเพิ่ม** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Header_Shared.md` · `MerC_to_AB_Function_Split.md` |
| **จุดต่างจาก D001** | Normalize % แบบ **ง่าย** (`pct / 100`) |
| **เทียบ** | `MerC_to_AB_D001.md` |

**อ่านยังไง:** ไฟล์นี้เป็น **สเปกครบทุก key ใน AB payload** สำหรับ F001 — แต่ละแถวบอกว่า map จาก C อย่างไร / ห้ามใส่ / เว้นว่าง / Auto  
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

## 0) Pipeline F001

```
convertLayoutCToAbPayload(c)
  normalizeC(c)
  HEADER  = mapHeaderShared(c.HEADER)
  BONUSBUYS = c.MATERIALS
    .filter(shouldConvertRow)
    .filter(m => shortCode(m.bonusBuyProfile) === "F001")
    .map(m => buildGroupB(ctx))         // buy[1] + get[1]
  CONDITIONS = mapConditionsShared(c.CONDITIONS)
  MATERIALS  = []
  LAYOUT = "AB"
  STATUS = c.STATUS

1 แถว MATERIALS[i] → 1 BONUSBUYS[i]
```

---

## 1) Skip rules (ก่อนแปลง)

| เงื่อนไขใน C | ผล |
|--------------|-----|
| `MATERIALS[*].status` = `Reject` | ข้ามแถว |
| `MATERIALS[*].convertStatus` = `Done` | ไม่แปลงซ้ำ |
| `MATERIALS[*].mechanic` = `B1G1 (On Pack)` หรือ `B2G1 (On Pack)` | ข้ามทั้งกลุ่ม |
| `bonusBuyProfile` ≠ `F001` | ไม่เข้า router นี้ |
| profile ไม่อยู่ใน 9 ตัวลูกค้า | reject |

---

## 2) ระดับบนสุด

| จาก C (JSON) | → AB (JSON) | การทำ | เงื่อนไข F001 |
|--------------|-------------|-------|----------------|
| — | `LAYOUT` | **calc** | คงที่ `"AB"` |
| `STATUS` | `STATUS` | **copy** | ส่งตรง |
| `HEADER` | `HEADER` | **map** | §3 |
| `MATERIALS[*]` | `BONUSBUYS[*]` | **calc** | สร้างใหม่ §4–§7 · buildGroupB — buy[1]+get[1] |
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
| `HEADER.bonusBuyProfile` | `bonusBuyProfile` | Bonus Buy Profile | **calc** | `shortCode` → `"F001"` |
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

| AB block | F001 |
|----------|------|
| `lineNumber` | **copy** จาก `MATERIALS[*].lineNumber` (optional) |
| `bonusBuyNumber` | **copy** ซ้ำระดับ root (optional) |
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

---

## 5) `bonusBuyHeader` (r31–r53)

| จาก C (JSON) | → AB (JSON) | ชื่อใน To-Be | req | การทำ | เงื่อนไข F001 |
|--------------|-------------|--------------|-----|-------|----------------|
| `MATERIALS[*].numberOfBonusBuy` / `noof_bonus_buy` | `bonusBuyNumber` | Bonus buy No. | M | **copy** | default `"1"` |
| `MATERIALS[*].bonusBuyProfile` / `bonus_buy_profile` | `bonusBuyProfile` | Bonus Buy Profile | — | **calc** | `"F001"` |
| `MATERIALS[*].mechanic` | `mechanic` | Mechanic | — | **copy** | + ใช้ lookup qty / ตัดสินฝั่ง §10 |
| `HEADER.timeFrom` | `validTimeFrom` | Valid Time from | — | **calc** | **ใส่** — ว่าง → `00:00:00` |
| `HEADER.timeTo` | `validTimeTo` | Valid Time to | — | **calc** | **ใส่** — ว่าง → `23:59:59` |
| `HEADER.wbsNumber` / `wbsNo` | `wbsNumber` | WBS No. | — | **ctx** | copy จาก header |
| `HEADER.promotionArea` **หรือ** `MATERIALS[*].promotionArea` | `promotionArea` | Promotion Area | — | **copy** | fallback: header ก่อน แล้ว material |
| `HEADER.onlineDescriptionEnglish` **หรือ** `MATERIALS[*].onlineDescriptionEnglish` | `onlineDescriptionEnglish` | Online Description EN | — | **copy** | **เฉพาะ** `promotionArea = "P4"` |
| `HEADER.onlineDescriptionThai` **หรือ** `MATERIALS[*].onlineDescriptionThai` | `onlineDescriptionThai` | Online Description TH | — | **copy** | **เฉพาะ** `promotionArea = "P4"` |
| `MATERIALS[*].referenceCode` | `referenceCode` | Reference code | — | **omit** | F001 **ไม่ใส่** |
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

## 6) `buy[0]` (r54–r69)

| จาก C (JSON) | → AB `buy[0]` | ชื่อใน To-Be | req | การทำ | เงื่อนไข F001 |
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

| จาก C (JSON) | → AB `get[0]` | ชื่อใน To-Be | req | การทำ | เงื่อนไข F001 |
|--------------|---------------|--------------|-----|-------|----------------|
| `numberOfBonusBuy` / `noof_bonus_buy` | `bonusBuyNumber` | Bonus Buy No. | **M** | **copy** | |
| มีกลุ่ม? | `field2` | ประเภท MAT / Group | **M** | **calc** | มีกลุ่ม → `"Material Group"` · ไม่มี → `"Material"` |
| `material` / `materialGroupName` | `field4` | รหัส / ชื่อกลุ่ม | **M** | **calc** | ตาม type |
| `materialDescription` / `material_des` | `field5` | Description | — | **copy** | optional |
| `materialDescription2` / `material_en_des` | `sapMasterDescription` | SAP desc | — | **copy** | optional |
| `costNormal` | `field7` | ราคาทุน | — | **copy** | optional |
| `salesPriceNormal` / `sales_price_normal` | `field8` | **ราคาขายปกติ** | — | **omit** | **F001 ห้ามใส่** (กลุ่ม B/C) |
| — | `vat` | VAT | — | **omit** | |
| — | `grossProfit` | GP% ปกติ | — | **auto** | |
| `mechanic` | `getQuantity` | Get Qty | **M** | **calc** | `lookupBuyGetQty` → get qty |
| — | `tierNumber` … `tierAmountB` | Tier | — | **omit** | |
| `costPromotion` / `cost_promotion` | `field17` | ราคาทุนจัดรายการ | — | **copy** | optional |
| `salesPricePromotion` / `sales_price_promo` | `fieldP` | ราคาจัดรายการ [P] | — | **calc** | `pickDiscount` ลำดับ 1 |
| — | `vat2` | VAT | — | **omit** | |
| — | `grossProfit2` | GP% ใหม่ | — | **auto** | |
| `discountAmount` / `discountAmt` | `fieldR` | ส่วนลดบาท [R] | — | **calc** | `pickDiscount` ลำดับ 2 |
| `discountPercentPlu` / `discountPercentForP015` / `discountPct` | `field22` | ส่วนลด % [%] | — | **calc** | ลำดับ 3 + `normalizePctSimple` |
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

## 10) สูตรและ helper (F001)

### 10.1 กฎเฉพาะ profile (สรุป)

| หัวข้อ | ค่า F001 |
|--------|----------|
| เขียนฝั่ง | `buy[1]` + `get[1]` |
| `get.field8` | **omit** |
| Normalize % | **ง่าย** (`pct / 100`) |
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
| 3 | `discountPercentPlu` / `discountPercentForP015` / `discountPct` | `field22` | เมื่อไม่มี 1 และ 2 · ใช้ `normalizePctSimple` |

### 10.4 `normalizePctSimple`

```
field22 = pct / 100
```

ไม่ปรับช่วงแบบเต็ม (ต่างจาก D001)

### 10.5 `lookupBuyGetQty(mechanic)`

→ `{ buyQty, getQty }` จากชีต Mechanic  
→ `buy.field9` และ `get.getQuantity`


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
| 3 | ส่วนลด % → `get.field22` ตาม normalize ของ F001 |
| 4 | `HEADER.vendorCode` ว่าง → `NOBP` |
| 5 | Material Group → `field2` / `field4` |
| 6 | `promotionArea = P4` → online EN/TH |

---

## 12) Skeleton JSON

```json
{
  "bonusBuyNumber": "1",
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "F001",
    "mechanic": "B1G1",
    "validTimeFrom": "08:30",
    "validTimeTo": "22:00",
    "promotionArea": "All",
    "wbsNumber": "WBS-2026-002"
  },
  "buy": [
    {
      "bonusBuyNumber": "1",
      "field2": "Material",
      "field4": "MAT-100",
      "field8": "129.00",
      "field9": "1",
      "salesUnit": "EA"
    }
  ],
  "get": [
    {
      "bonusBuyNumber": "1",
      "field2": "Material",
      "field4": "MAT-100",
      "getQuantity": "1",
      "field22": "0.10",
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

## 13) Checklist (F001 ครบ)

### โครง
- [ ] `LAYOUT` = `"AB"`
- [ ] `MATERIALS` ฝั่ง AB = `[]`
- [ ] 1 แถว C → 1 `BONUSBUYS` มีทั้ง buy+get
- [ ] % ใช้ `pct / 100` ไม่ใช่ normalize เต็ม
- [ ] มี `validTime*`
- [ ] ไม่มี `referenceCode`
- [ ] online เฉพาะ P4
- [ ] ส่วนลด **ช่องเดียว**: P หรือ R หรือ field22
- [ ] `get` **ไม่มี** `field8`
- [ ] Get Mandatory เมื่อมี get: `bonusBuyNumber`, `field2`, `field4`, `getQuantity`, `unit`

### HEADER
- [ ] วันที่ใน `HEADER.periodFrom/periodTo` เป็น `YYYY-MM-DD`
- [ ] `vendorCode` ว่าง → `NOBP`
- [ ] มี `timeFrom`/`timeTo` ใน AB `HEADER`

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

เทียบ: `payload_AB_Reference.md` · `Gen_F001` / `F001_Header` ใน `Mer-C_Convert_To_STD.txt` · `layout-ab-payload.example.json`

| หัวข้อ | สถานะ |
|--------|--------|
| AB key name ตาม payload | ✅ |
| Core logic ตามตาราง §10.1 | ✅ ตาม macro |
| `get.field8` omit | ✅ กลุ่ม B/C |
| % แบบง่าย (`/100`) | ✅ ตรง `Gen_F001` |
| Golden test ครบทุก key | ❌ ยังไม่มี |
