# C → AB — Profile P010

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P010_Header` + `Gen_P010` |
| **โครง** | `buy: []` · `get: [1]` ต่อ 1 แถว `MATERIALS[*]` |
| **ต่างจาก P001** | ไม่ใส่ `validTime*` ใน `bonusBuyHeader` · ใส่ `referenceCode` · online เมื่อ `P4` |
| **payload หลังบ้านหลัก** | `layout-ab-payload.example.json` |
| **ชื่อฟิลด์ AB อธิบายเพิ่ม** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Header_Shared.md` · `MerC_to_AB_Function_Split.md` |

ไฟล์นี้ยึด payload หลังบ้านเป็นหลัก:

- `HEADER.periodFrom` / `periodTo` = `YYYY-MM-DD`
- `HEADER.timeFrom` / `timeTo` ยังอยู่ใน `HEADER`
- `field2` ใช้ค่า `"Material"` หรือ `"Material Group"`

---

## 0) Pipeline P010

```text
convertLayoutCToAbPayload(c)
  normalizeC(c)
  HEADER = mapHeaderShared(c.HEADER)
  BONUSBUYS = c.MATERIALS
    .filter(shouldConvertRow)
    .filter(m => shortCode(m.bonusBuyProfile) === "P010")
    .map(m => buildGroupA(ctx))
  CONDITIONS = mapConditionsShared(c.CONDITIONS)
  MATERIALS = []
  SUPPLIERFILE = null
```

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า P010 |
|--------|----------|
| เขียนฝั่ง | `get[]` เท่านั้น |
| `buy[]` | `[]` เสมอ |
| `get.field8` | **ใส่** |
| Normalize % | **เต็ม** |
| `bonusBuyHeader.validTimeFrom/To` | **omit** |
| `bonusBuyHeader.referenceCode` | **ใส่** |
| `bonusBuyHeader.onlineDescriptionEnglish/Thai` | ใส่เมื่อ `promotionArea = "P4"` |

---

## 2) ระดับบนสุด

| จาก C (JSON) | → AB (JSON) | การทำ | เงื่อนไข P010 |
|--------------|-------------|-------|----------------|
| — | `LAYOUT` | calc | `"AB"` |
| `STATUS` | `STATUS` | copy | ส่งตรง |
| `HEADER` | `HEADER` | map | §3 |
| `MATERIALS[*]` | `BONUSBUYS[*]` | calc | สร้างใหม่ |
| `CONDITIONS` | `CONDITIONS` | copy | shared |
| `BONUSBUYS` (C) | — | omit | |
| `MATERIALS` (AB) | `MATERIALS` | empty | `[]` |
| — | `SUPPLIERFILE` | empty | `null` ถ้าจะคง shape |

---

## 3) HEADER

| จาก C (JSON) | → AB `HEADER` | การทำ | หมายเหตุ |
|--------------|---------------|-------|----------|
| `HEADER.group` | `group` | copy | |
| `HEADER.purchasingGroup` | `purchasingGroup` | copy | |
| `HEADER.promotionName` | `promotionName` | copy | |
| `HEADER.theme` | `theme` | copy | |
| `HEADER.volume` / `vol` | `volume` | copy | optional |
| `HEADER.contractType` | `contractType` | calc | shared rule |
| `HEADER.bonusBuyProfile` | `bonusBuyProfile` | calc | `"P010"` |
| `HEADER.rebateChargeback` | `rebateChargeback` | calc | shared rule |
| `HEADER.wbsNumber` / `wbsNo` | `wbsNumber` | copy | |
| `HEADER.vendorCode` | `vendorCode` | calc | ว่าง → `NOBP` |
| `HEADER.vendorName` | `vendorName` | copy | optional |
| `HEADER.singleMultiple` | `singleMultiple` | copy | optional |
| `HEADER.periodFrom` | `periodFrom` | copy | `YYYY-MM-DD` |
| `HEADER.periodTo` | `periodTo` | copy | `YYYY-MM-DD` |
| `HEADER.timeFrom` | `timeFrom` | copy | คงไว้ใน header |
| `HEADER.timeTo` | `timeTo` | copy | คงไว้ใน header |
| `HEADER.days` | `days` | copy | |
| `HEADER.status` | — | omit | sample ไม่มี |

---

## 4) `BONUSBUYS[*]`

| AB block | P010 |
|----------|------|
| `lineNumber` | copy / optional |
| `bonusBuyNumber` | copy / optional |
| `bonusBuyHeader` | §5 |
| `buy[]` | `[]` |
| `get[]` | `[1]` |
| `card[]` / `tender[]` / `installment[]` / `posTerminal[]` / `premium[]` / `coupon[]` / `limitControl[]` | `[]` |
| `stores[]` | copy / empty |

---

## 5) `bonusBuyHeader`

| key | P010 | การทำ / หมายเหตุ |
|-----|------|-------------------|
| `promotionNumber` | auto / optional | |
| `bonusBuyNumber` | ใช้ | copy |
| `description` | auto / optional | |
| `purchasingGroup` | auto / optional | |
| `bonusBuyProfile` | ใช้ | `"P010"` |
| `mechanic` | ใช้ | copy |
| `limitNumber` | omit | |
| `validTimeFrom` | omit | P010 ไม่ใส่ |
| `validTimeTo` | omit | P010 ไม่ใส่ |
| `product` | auto | |
| `priceTag` | omit | |
| `promotionArea` | copy | |
| `wbsNumber` | ใช้ | จาก header |
| `referenceBonusBuy` | omit | |
| `legacyPromotionNumber` | omit | |
| `allowDiscountAfterGetExtraMPoint` | omit | |
| `notAcceptAnyDiscountCoupon` | omit | |
| `notAcceptAnyDiscountCard` | omit | |
| `notAllowForEmployee` | omit | |
| `referenceCode` | ใช้ | จาก `HEADER.referenceCode` หรือ row |
| `department` | auto | |
| `onlineDescriptionEnglish` | conditional | เฉพาะ `promotionArea = "P4"` |
| `onlineDescriptionThai` | conditional | เฉพาะ `promotionArea = "P4"` |

---

## 6) `buy[]`

P010 เป็น get-only:

- `buy = []`
- ไม่มี `buy[0]`

---

## 7) `get[0]`

| key | จาก C | การทำ | เงื่อนไข P010 |
|-----|--------|-------|----------------|
| `bonusBuyNumber` | `numberOfBonusBuy` | copy | mandatory |
| `field2` | material vs group | calc | `"Material"` หรือ `"Material Group"` |
| `char` | — | omit | |
| `field4` | `material` / `materialGroupName` | calc | |
| `field5` | `materialDescription` | copy | optional |
| `sapMasterDescription` | `materialDescription2` | copy | optional |
| `field7` | `costNormal` | copy | optional |
| `field8` | `salesPriceNormal` | copy | **ใส่** |
| `vat` | `vat` | copy / omit | optional |
| `grossProfit` | — | auto | |
| `getQuantity` | `mechanic` | calc | lookup qty |
| `tierNumber` / `recursive` / `progressive` / `tierQuantityA` / `tierAmountB` | — | omit | |
| `field17` | `costPromotion` | copy | optional |
| `fieldP` | `salesPricePromotion` | calc | ลำดับ 1 |
| `vat2` | — | omit | |
| `grossProfit2` | — | auto | |
| `fieldR` | `discountAmount` | calc | ลำดับ 2 |
| `field22` | discount percent aliases | calc | ลำดับ 3 + normalize เต็ม |
| `newGrossProfitRate` | — | omit | |
| `ean` | `barcode` | copy | optional |
| `serialNumber` | — | omit | |
| `unit` | `salesUnit` | copy | mandatory |
| `priceUnit` / `unitOfMeasure` | — | omit | |
| `basicPoint` / `extraPoint` / `pointAmount` / `exclusion` | — | omit | |
| `noDiscount` | — | auto | |
| `promotionTagSizeA4Cut1/2/4/6` | — | omit | |

---

## 8) `CONDITIONS[]` และ `MATERIALS[]`

- `CONDITIONS[]` = shared `copy + compact`
- `MATERIALS[]` ฝั่ง AB = `[]`

---

## 9) สูตรและ helper

1. `salesPricePromotion` → `fieldP`
2. `discountAmount` → `fieldR`
3. `discountPercentPlu` / `discountPercentForP015` / `discountPct` → `field22`

Normalize % แบบเต็ม:

```text
ถ้า (% / 100) < 1  -> ใช้ค่าเดิม
ถ้า % > 100        -> % / 100
ถ้า % < 1          -> % * 100
```

---

## 10) Skeleton JSON

```json
{
  "lineNumber": 0,
  "bonusBuyNumber": "1",
  "bonusBuyHeader": {
    "promotionNumber": "1",
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "P010",
    "mechanic": "Discount",
    "promotionArea": "P4",
    "wbsNumber": "WBS-2026-002",
    "referenceCode": "REF-001",
    "onlineDescriptionEnglish": "Online EN",
    "onlineDescriptionThai": "Online TH"
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
      "field22": "10",
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

## 11) Checklist

- [ ] `HEADER.periodFrom/periodTo` ใช้ `YYYY-MM-DD`
- [ ] `HEADER.timeFrom/timeTo` ยังอยู่ใน header
- [ ] `bonusBuyHeader` **ไม่มี** `validTimeFrom/To`
- [ ] `bonusBuyHeader.referenceCode` มีค่า
- [ ] online description ใส่เฉพาะ `promotionArea = "P4"`
- [ ] `buy = []`
- [ ] `get.field8` มีค่า
