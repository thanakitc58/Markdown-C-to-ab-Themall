# C → AB — Profile P001

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P001_Header` + `Gen_P001` |
| **โครง** | `buy: []` · `get: [1]` ต่อ 1 แถว `MATERIALS[*]` |
| **payload หลังบ้านหลัก** | `layout-ab-payload.example.json` |
| **ชื่อฟิลด์ AB อธิบายเพิ่ม** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Header_Shared.md` · `MerC_to_AB_Function_Split.md` |

ไฟล์นี้ยึด **shape / key / รูปแบบค่า** ตาม `layout-ab-payload.example.json` ก่อนเสมอ  
ดังนั้นตัวอย่างในเอกสารนี้จะใช้แนวเดียวกับ backend sample เช่น:

- `HEADER.periodFrom` / `periodTo` = `YYYY-MM-DD`
- `HEADER.timeFrom` / `timeTo` = อยู่ใน `HEADER` ด้วย
- `field2` ใช้ค่า `"Material"` หรือ `"Material Group"` เชิง payload

---

## 0) Pipeline P001

```text
convertLayoutCToAbPayload(c)
  normalizeC(c)
  HEADER = mapHeaderShared(c.HEADER)
  BONUSBUYS = c.MATERIALS
    .filter(shouldConvertRow)
    .filter(m => shortCode(m.bonusBuyProfile) === "P001")
    .map(m => buildGroupA(ctx))        // get[1] only
  CONDITIONS = mapConditionsShared(c.CONDITIONS)
  MATERIALS = []
  SUPPLIERFILE = null                  // ถ้าต้องคง shape ตาม backend sample
```

1 แถว `MATERIALS[i]` (profile P001) → 1 element `BONUSBUYS[i]`

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า P001 |
|--------|----------|
| เขียนฝั่ง | `get[]` เท่านั้น |
| `buy[]` | `[]` เสมอ |
| `get.field8` (normal sell price) | **ใส่** |
| Normalize % | **เต็ม** |
| `bonusBuyHeader.validTimeFrom/To` | **ใส่** |
| `bonusBuyHeader.referenceCode` | **omit** |
| `bonusBuyHeader.onlineDescriptionEnglish/Thai` | **omit** |

---

## 2) ระดับบนสุด

| จาก C (JSON) | → AB (JSON) | การทำ | เงื่อนไข P001 |
|--------------|-------------|-------|----------------|
| — | `LAYOUT` | **calc** | คงที่ `"AB"` |
| `STATUS` | `STATUS` | **copy** | ส่งตรง |
| `HEADER` | `HEADER` | **map** | §3 |
| `MATERIALS[*]` | `BONUSBUYS[*]` | **calc** | สร้างใหม่ §4–§7 |
| `CONDITIONS` | `CONDITIONS` | **copy** | §8 |
| `BONUSBUYS` (C) | — | **omit** | ไม่ใช้เป็นต้นทาง |
| `MATERIALS` (AB) | `MATERIALS` | **empty** | `[]` |
| — | `SUPPLIERFILE` | **empty** | ใช้ `null` ถ้าต้องคง shape ตาม sample |

---

## 3) HEADER

P001 ใช้ shared header rules และ **เก็บ `timeFrom/timeTo` ไว้ใน `HEADER` ด้วย** ตาม payload หลังบ้าน

| จาก C (JSON) | → AB `HEADER` | การทำ | หมายเหตุ |
|--------------|---------------|-------|----------|
| `HEADER.group` | `group` | **copy** | ส่งตรง |
| `HEADER.purchasingGroup` | `purchasingGroup` | **copy** | lookup ชื่อเต็มได้ถ้ามี |
| `HEADER.promotionName` | `promotionName` | **copy** | ส่งตรง |
| `HEADER.theme` | `theme` | **copy** | lookup รหัสได้ |
| `HEADER.volume` / `vol` | `volume` | **copy** | optional |
| `HEADER.contractType` | `contractType` | **calc** | shared rule |
| `HEADER.bonusBuyProfile` | `bonusBuyProfile` | **calc** | `"P001"` |
| `HEADER.rebateChargeback` | `rebateChargeback` | **calc** | shared rule |
| `HEADER.wbsNumber` / `wbsNo` | `wbsNumber` | **copy** | fallback ได้ |
| `HEADER.vendorCode` | `vendorCode` | **calc** | ว่าง → `"NOBP"` |
| `HEADER.vendorName` | `vendorName` | **copy** | optional |
| `HEADER.singleMultiple` | `singleMultiple` | **copy** | optional |
| `HEADER.periodFrom` | `periodFrom` | **copy** | ใช้ `YYYY-MM-DD` ตาม backend payload |
| `HEADER.periodTo` | `periodTo` | **copy** | ใช้ `YYYY-MM-DD` |
| `HEADER.timeFrom` | `timeFrom` | **copy** | อยู่ใน `HEADER` |
| `HEADER.timeTo` | `timeTo` | **copy** | อยู่ใน `HEADER` |
| `HEADER.days` | `days` | **copy** | เช่น `["mon","wed"]` |
| `HEADER.status` | — | **omit** | payload sample ไม่มี |

---

## 4) `BONUSBUYS[*]` — โครงก้อน

| AB block | P001 |
|----------|------|
| `lineNumber` | copy / optional |
| `bonusBuyNumber` | copy / optional ที่ root |
| `bonusBuyHeader` | §5 |
| `buy[]` | `[]` เสมอ |
| `get[]` | ความยาว `[1]` — §7 |
| `card[]` | empty |
| `tender[]` | empty |
| `installment[]` | empty |
| `posTerminal[]` | empty |
| `premium[]` | empty |
| `coupon[]` | empty |
| `limitControl[]` | empty |
| `stores[]` | copy / empty |

---

## 5) `bonusBuyHeader`

| key | P001 | การทำ / หมายเหตุ |
|-----|------|-------------------|
| `promotionNumber` | auto / optional | ปล่อยระบบหรือ copy ถ้ามี |
| `bonusBuyNumber` | ใช้ | copy จาก `numberOfBonusBuy` |
| `description` | auto / optional | prefix + promo name ได้ |
| `purchasingGroup` | auto / optional | ดึงจาก header |
| `bonusBuyProfile` | ใช้ | `"P001"` |
| `mechanic` | ใช้ | copy จาก material row |
| `limitNumber` | omit | |
| `validTimeFrom` | ใช้ | จาก `HEADER.timeFrom`, ว่าง → default |
| `validTimeTo` | ใช้ | จาก `HEADER.timeTo`, ว่าง → default |
| `product` | auto | |
| `priceTag` | omit | |
| `promotionArea` | copy | จาก header/material |
| `wbsNumber` | ใช้ | จาก header |
| `referenceBonusBuy` | omit | |
| `legacyPromotionNumber` | omit | |
| `allowDiscountAfterGetExtraMPoint` | omit | |
| `notAcceptAnyDiscountCoupon` | omit | |
| `notAcceptAnyDiscountCard` | omit | |
| `notAllowForEmployee` | omit | |
| `referenceCode` | omit | P001 ไม่ใช้ |
| `department` | auto | |
| `onlineDescriptionEnglish` | omit | P001 ไม่ใช้ |
| `onlineDescriptionThai` | omit | P001 ไม่ใช้ |

---

## 6) `buy[]`

P001 เป็น get-only ดังนั้น:

- `buy = []`
- ไม่สร้าง `buy[0]`
- field ใน buy ทั้งหมดถือว่า **not used for this profile**

---

## 7) `get[0]`

| key | จาก C | การทำ | เงื่อนไข P001 |
|-----|--------|-------|----------------|
| `bonusBuyNumber` | `numberOfBonusBuy` | copy | mandatory |
| `field2` | material vs group | calc | `"Material"` หรือ `"Material Group"` |
| `char` | — | omit | |
| `field4` | `material` / `materialGroupName` | calc | ตาม `field2` |
| `field5` | `materialDescription` | copy | optional |
| `sapMasterDescription` | `materialDescription2` | copy | optional |
| `field7` | `costNormal` | copy | optional |
| `field8` | `salesPriceNormal` | copy | **ใส่** |
| `vat` | `vat` | copy / omit | optional |
| `grossProfit` | — | auto | |
| `getQuantity` | `mechanic` | calc | lookup qty |
| `tierNumber` | — | omit | |
| `recursive` | — | omit | |
| `progressive` | — | omit | |
| `tierQuantityA` | — | omit | |
| `tierAmountB` | — | omit | |
| `field17` | `costPromotion` | copy | optional |
| `fieldP` | `salesPricePromotion` | calc | ส่วนลดลำดับ 1 |
| `vat2` | — | omit | |
| `grossProfit2` | — | auto | |
| `fieldR` | `discountAmount` | calc | ส่วนลดลำดับ 2 |
| `field22` | `discountPercentPlu` / aliases | calc | ส่วนลดลำดับ 3 + normalize เต็ม |
| `newGrossProfitRate` | — | omit | |
| `ean` | `barcode` | copy | optional |
| `serialNumber` | — | omit | |
| `unit` | `salesUnit` | copy | mandatory |
| `priceUnit` | — | omit | |
| `unitOfMeasure` | — | omit | |
| `basicPoint` | — | omit | |
| `extraPoint` | — | omit | |
| `pointAmount` | — | omit | |
| `exclusion` | — | omit | |
| `noDiscount` | — | auto | ตาม rule On Top |
| `promotionTagSizeA4Cut1` | — | omit | |
| `promotionTagSizeA4Cut2` | — | omit | |
| `promotionTagSizeA4Cut4` | — | omit | |
| `promotionTagSizeA4Cut6` | — | omit | |

### 7.1 บล็อกเสริม

| block | P001 |
|-------|------|
| `card[]` | `[]` |
| `tender[]` | `[]` |
| `installment[]` | `[]` |
| `posTerminal[]` | `[]` |
| `premium[]` | `[]` |
| `coupon[]` | `[]` |
| `limitControl[]` | `[]` |
| `stores[]` | copy ถ้ามี, ไม่งั้น `[]` |

---

## 8) `CONDITIONS[]`

ใช้ shared contract shape ตาม backend payload:

- `conditionHeader`
- `businessVolumePurchase[]`
- `businessVolumeSales[]`
- `conditionType[]`
- `settlementCalendar[]`
- `combineCheck[]`
- `allocation[]`

P001 ไม่มี logic เฉพาะเพิ่ม: **copy + compact** จาก C

---

## 9) `MATERIALS[]` ฝั่ง AB

`MATERIALS = []` สำหรับรอบ convert BBY

---

## 10) สูตรและ helper

### 10.1 ลำดับส่วนลด

1. `salesPricePromotion` → `fieldP`
2. `discountAmount` → `fieldR`
3. `discountPercentPlu` / `discountPercentForP015` / `discountPct` → `field22`

ใช้ **ช่องเดียว**

### 10.2 Normalize % แบบเต็ม

```text
ถ้า (% / 100) < 1  -> ใช้ค่าเดิม
ถ้า % > 100        -> % / 100
ถ้า % < 1          -> % * 100
```

### 10.3 Alias ฝั่ง C

| camelCase | alias ที่พบ |
|-----------|-------------|
| `salesPriceNormal` | `sales_price_normal` |
| `salesPricePromotion` | `sales_price_promo` |
| `bonusBuyProfile` | `bonus_buy_profile` |
| `numberOfBonusBuy` | `noof_bonus_buy` |
| `materialDescription` | `material_des`, `material_th_des` |
| `materialDescription2` | `material_des_2`, `material_en_des` |
| `discountAmount` | `discountAmt` |

---

## 11) Skeleton JSON

```json
{
  "lineNumber": 0,
  "bonusBuyNumber": "1",
  "bonusBuyHeader": {
    "promotionNumber": "1",
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "P001",
    "mechanic": "Discount",
    "validTimeFrom": "08:30",
    "validTimeTo": "22:00",
    "promotionArea": "All",
    "wbsNumber": "WBS-2026-002"
  },
  "buy": [],
  "get": [
    {
      "bonusBuyNumber": "1",
      "field2": "Material",
      "field4": "MAT-100",
      "field5": "Get item description",
      "field7": "80.00",
      "field8": "129.00",
      "getQuantity": "1",
      "fieldP": "99.00",
      "unit": "EA"
    }
  ],
  "card": [],
  "tender": [],
  "installment": [],
  "posTerminal": [],
  "premium": [],
  "coupon": [],
  "limitControl": [],
  "stores": []
}
```

---

## 12) Checklist

- [ ] `HEADER.periodFrom/periodTo` ใช้ `YYYY-MM-DD`
- [ ] `HEADER.timeFrom/timeTo` อยู่ใน header
- [ ] `bonusBuyHeader.validTimeFrom/To` มีค่า
- [ ] `buy = []`
- [ ] `get` มี `bonusBuyNumber`, `field2`, `field4`, `field8`, `getQuantity`, `unit`
- [ ] ส่วนลดมีแค่ `fieldP` หรือ `fieldR` หรือ `field22`
- [ ] ไม่มี `referenceCode` และ online description
