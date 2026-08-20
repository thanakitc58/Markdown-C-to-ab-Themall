# C → AB — Profile P015

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P015_Header` + `Gen_P015` |
| **โครง** | `buy: []` · `get: [1]` ต่อ 1 แถว `MATERIALS[*]` |
| **จุดต่างหลัก** | `%` อาจมาจาก `discountPercentForP015` แต่ลงปลายทาง `get.field22` |
| **payload หลังบ้านหลัก** | `layout-ab-payload.example.json` |
| **ชื่อฟิลด์ AB อธิบายเพิ่ม** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Header_Shared.md` · `MerC_to_AB_Function_Split.md` |

ยึด payload หลังบ้านเป็นหลัก: `HEADER.period*` = `YYYY-MM-DD`, `HEADER.time*` อยู่ใน header, `field2` ใช้ `"Material"` / `"Material Group"`.

---

## 0) Pipeline P015

```text
normalizeC -> mapHeaderShared -> filter P015 -> buildGroupA -> copy CONDITIONS -> MATERIALS=[]
```

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า P015 |
|--------|----------|
| เขียนฝั่ง | `get[]` เท่านั้น |
| `buy[]` | `[]` |
| `get.field8` | **ใส่** |
| Normalize % | **เต็ม** |
| `bonusBuyHeader.validTimeFrom/To` | **ใส่** |
| `bonusBuyHeader.referenceCode` | omit |
| `bonusBuyHeader.onlineDescriptionEnglish/Thai` | omit |

---

## 2) HEADER

| key | การทำ P015 |
|-----|------------|
| `group`, `purchasingGroup`, `promotionName`, `theme`, `volume`, `singleMultiple`, `vendorName`, `days` | copy |
| `bonusBuyProfile` | `"P015"` |
| `rebateChargeback`, `contractType`, `vendorCode` | shared normalize |
| `wbsNumber` | copy |
| `periodFrom`, `periodTo` | copy เป็น `YYYY-MM-DD` |
| `timeFrom`, `timeTo` | copy ไว้ใน `HEADER` |
| `status` | omit |

---

## 3) `BONUSBUYS[*]`

| AB block | P015 |
|----------|------|
| `lineNumber`, `bonusBuyNumber` | copy / optional |
| `bonusBuyHeader` | ใช้ |
| `buy[]` | `[]` |
| `get[]` | `[1]` |
| `card[]`, `tender[]`, `installment[]`, `posTerminal[]`, `premium[]`, `coupon[]`, `limitControl[]` | `[]` |
| `stores[]` | copy / `[]` |

### 3.1 `bonusBuyHeader`

| key | การทำ P015 |
|-----|------------|
| `promotionNumber`, `description`, `purchasingGroup`, `product`, `department` | auto / optional |
| `bonusBuyNumber` | copy |
| `bonusBuyProfile` | `"P015"` |
| `mechanic` | copy |
| `validTimeFrom`, `validTimeTo` | ใช้จาก `HEADER.timeFrom/timeTo` |
| `promotionArea`, `wbsNumber` | copy |
| `referenceCode` | omit |
| `onlineDescriptionEnglish`, `onlineDescriptionThai` | omit |
| `limitNumber`, `priceTag`, `referenceBonusBuy`, `legacyPromotionNumber`, `allowDiscountAfterGetExtraMPoint`, `notAcceptAnyDiscountCoupon`, `notAcceptAnyDiscountCard`, `notAllowForEmployee` | omit |

---

## 4) `get[0]` ครบทุก field

| key | จาก C | การทำ P015 |
|-----|--------|------------|
| `bonusBuyNumber` | `numberOfBonusBuy` | copy |
| `field2` | material/group | `"Material"` หรือ `"Material Group"` |
| `char` | — | omit |
| `field4` | `material` / `materialGroupName` | calc |
| `field5` | `materialDescription` | copy |
| `sapMasterDescription` | `materialDescription2` | copy |
| `field7` | `costNormal` | copy |
| `field8` | `salesPriceNormal` | **ใส่** |
| `vat` | `vat` | copy / omit |
| `grossProfit` | — | auto |
| `getQuantity` | `mechanic` | lookup qty |
| `tierNumber`, `recursive`, `progressive`, `tierQuantityA`, `tierAmountB` | — | omit |
| `field17` | `costPromotion` | copy |
| `fieldP` | `salesPricePromotion` | ส่วนลดลำดับ 1 |
| `vat2` | — | omit |
| `grossProfit2` | — | auto |
| `fieldR` | `discountAmount` | ส่วนลดลำดับ 2 |
| `field22` | `discountPercentForP015` / `discountPercentPlu` / `discountPct` | ส่วนลดลำดับ 3 + normalize เต็ม |
| `newGrossProfitRate` | — | omit |
| `ean` | `barcode` | copy |
| `serialNumber` | — | omit |
| `unit` | `salesUnit` | copy |
| `priceUnit`, `unitOfMeasure` | — | omit |
| `basicPoint`, `extraPoint`, `pointAmount`, `exclusion` | — | omit |
| `noDiscount` | — | auto |
| `promotionTagSizeA4Cut1/2/4/6` | — | omit |

---

## 5) `CONDITIONS[]` / `MATERIALS[]` / `SUPPLIERFILE`

- `CONDITIONS[]` = shared `copy + compact`
- `MATERIALS[]` ฝั่ง AB = `[]`
- `SUPPLIERFILE` = `null` ถ้าต้องคง shape ตาม sample

---

## 6) สูตรร่วม

1. `salesPricePromotion` → `fieldP`
2. `discountAmount` → `fieldR`
3. `discountPercentForP015` / `discountPercentPlu` / `discountPct` → `field22`

Normalize % แบบเต็ม:

```text
ถ้า (% / 100) < 1  -> ใช้ค่าเดิม
ถ้า % > 100        -> % / 100
ถ้า % < 1          -> % * 100
```

---

## 7) Skeleton JSON

```json
{
  "bonusBuyNumber": "1",
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "P015",
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

## 8) Checklist

- [ ] `HEADER.period*` เป็น `YYYY-MM-DD`
- [ ] `HEADER.time*` อยู่ใน header
- [ ] `bonusBuyHeader.validTime*` มีค่า
- [ ] ไม่มี `referenceCode` และ online description
- [ ] `buy = []`
- [ ] `%` จากช่อง P015 ลง `get.field22`
