# C → AB — Profile D003

| | |
|--|--|
| **กลุ่ม** | C — แยก element |
| **Macro** | `D003_Header` + `Gen_D003` |
| **โครง** | buy-only หรือ get-only คนละ `BONUSBUYS` element |
| **payload หลังบ้านหลัก** | `layout-ab-payload.example.json` |
| **ชื่อฟิลด์ AB อธิบายเพิ่ม** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Header_Shared.md` · `MerC_to_AB_Function_Split.md` |

ยึด payload หลังบ้านเป็นหลัก: `HEADER.period*` = `YYYY-MM-DD`, `HEADER.time*` อยู่ใน header, `field2` ใช้ `"Material"` / `"Material Group"`.

---

## 0) Pipeline D003

```text
normalizeC -> mapHeaderShared -> filter D003 -> splitByMechanic(A/B) -> copy CONDITIONS -> MATERIALS=[]
```

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า D003 |
|--------|----------|
| เขียนฝั่ง | `(A)` -> buy-only, `(B)` -> get-only |
| Coupon พิเศษ | `A(Coupon)+B(A)` -> buy, `+B(B)` -> get ของ element ก่อนหน้า |
| `get.field8` | omit |
| Normalize % | เต็ม |
| `bonusBuyHeader.validTimeFrom/To` | ใส่ |
| `referenceCode` | omit |
| online | ใส่เมื่อ `promotionArea = "P4"` |

---

## 2) HEADER

| key | การทำ D003 |
|-----|------------|
| `group`, `purchasingGroup`, `promotionName`, `theme`, `volume`, `singleMultiple`, `vendorName`, `days` | copy |
| `bonusBuyProfile` | `"D003"` |
| `rebateChargeback`, `contractType`, `vendorCode` | shared normalize |
| `wbsNumber` | copy |
| `periodFrom`, `periodTo` | copy เป็น `YYYY-MM-DD` |
| `timeFrom`, `timeTo` | copy ไว้ใน `HEADER` |
| `status` | omit |

---

## 3) `BONUSBUYS[*]`

| block | D003 |
|-------|------|
| `lineNumber`, `bonusBuyNumber` | copy / optional |
| `bonusBuyHeader` | ใช้ทุก element |
| `buy[]` | `[1]` หรือ `[]` ตาม mechanic |
| `get[]` | `[1]` หรือ `[]` ตาม mechanic |
| `card[]`, `tender[]`, `installment[]`, `posTerminal[]`, `premium[]`, `coupon[]`, `limitControl[]` | `[]` เว้นแต่มี rule พิเศษภายหลัง |
| `stores[]` | copy / `[]` |

### 3.1 `bonusBuyHeader`

| key | การทำ D003 |
|-----|------------|
| `promotionNumber`, `description`, `purchasingGroup`, `product`, `department` | auto / optional |
| `bonusBuyNumber` | copy |
| `bonusBuyProfile` | `"D003"` |
| `mechanic` | copy และใช้ตัดสินฝั่ง |
| `validTimeFrom`, `validTimeTo` | ใช้จาก `HEADER.timeFrom/timeTo` |
| `promotionArea`, `wbsNumber` | copy |
| `referenceCode` | omit |
| `onlineDescriptionEnglish`, `onlineDescriptionThai` | ใส่เมื่อ `promotionArea = "P4"` |
| field ที่เหลือใน header block | omit ถ้าไม่มี requirement |

---

## 4) `buy[0]` เมื่อ mechanic ชี้ `(A)`

| key | จาก C | การทำ D003 |
|-----|--------|------------|
| `bonusBuyNumber` | `numberOfBonusBuy` | copy |
| `field2` | material/group | `"Material"` หรือ `"Material Group"` |
| `char` | — | omit |
| `field4` | `material` / `materialGroupName` | calc |
| `description` | `materialDescription` | copy |
| `sapMasterDescription` | `materialDescription2` | copy |
| `field7` | `costNormal` | copy |
| `field8` | `salesPriceNormal` | copy |
| `field9` | `mechanic` | lookup buy qty |
| `field10` | — | omit |
| `ean` | `barcode` | copy |
| `serial` | — | omit |
| `salesUnit` | `salesUnit` | copy |
| `promotionTagSizeA4Cut1/2/4/6` | — | omit |

## 5) `get[0]` เมื่อ mechanic ชี้ `(B)`

| key | จาก C | การทำ D003 |
|-----|--------|------------|
| `bonusBuyNumber` | `numberOfBonusBuy` | copy |
| `field2` | material/group | `"Material"` หรือ `"Material Group"` |
| `char` | — | omit |
| `field4` | `material` / `materialGroupName` | calc |
| `field5` | `materialDescription` | copy |
| `sapMasterDescription` | `materialDescription2` | copy |
| `field7` | `costNormal` | copy |
| `field8` | `salesPriceNormal` | omit |
| `vat` | `vat` | copy / omit |
| `grossProfit` | — | auto |
| `getQuantity` | `mechanic` | lookup get qty |
| `tierNumber`, `recursive`, `progressive`, `tierQuantityA`, `tierAmountB` | — | omit |
| `field17` | `costPromotion` | copy |
| `fieldP` | `salesPricePromotion` | ส่วนลดลำดับ 1 |
| `vat2` | — | omit |
| `grossProfit2` | — | auto |
| `fieldR` | `discountAmount` | ส่วนลดลำดับ 2 |
| `field22` | percent aliases | ส่วนลดลำดับ 3 + normalize เต็ม |
| `newGrossProfitRate` | — | omit |
| `ean` | `barcode` | copy |
| `serialNumber` | — | omit |
| `unit` | `salesUnit` | copy |
| `priceUnit`, `unitOfMeasure` | — | omit |
| `basicPoint`, `extraPoint`, `pointAmount`, `exclusion` | — | omit |
| `noDiscount` | — | auto |
| `promotionTagSizeA4Cut1/2/4/6` | — | omit |

---

## 6) `CONDITIONS[]` / `MATERIALS[]` / `SUPPLIERFILE`

- `CONDITIONS[]` = shared `copy + compact`
- `MATERIALS[]` ฝั่ง AB = `[]`
- `SUPPLIERFILE` = `null` ถ้าต้องคง shape ตาม sample

---

## 7) สูตรร่วม

1. `salesPricePromotion` -> `fieldP`
2. `discountAmount` -> `fieldR`
3. `discountPercentPlu` / `discountPercentForP015` / `discountPct` -> `field22`

Normalize % แบบเต็ม:

```text
ถ้า (% / 100) < 1  -> ใช้ค่าเดิม
ถ้า % > 100        -> % / 100
ถ้า % < 1          -> % * 100
```

---

## 8) Skeleton JSON

```json
[
  {
    "bonusBuyNumber": "1",
    "bonusBuyHeader": {
      "bonusBuyNumber": "1",
      "bonusBuyProfile": "D003",
      "mechanic": "Coupon (A)",
      "validTimeFrom": "08:30",
      "validTimeTo": "22:00"
    },
    "buy": [
      {
        "field2": "Material",
        "field4": "MAT-100",
        "field9": "1"
      }
    ],
    "get": []
  },
  {
    "bonusBuyNumber": "1",
    "bonusBuyHeader": {
      "bonusBuyNumber": "1",
      "bonusBuyProfile": "D003",
      "mechanic": "Coupon (B)",
      "validTimeFrom": "08:30",
      "validTimeTo": "22:00"
    },
    "buy": [],
    "get": [
      {
        "field2": "Material",
        "field4": "MAT-100",
        "getQuantity": "1",
        "fieldR": "30.00",
        "unit": "EA"
      }
    ]
  }
]
```

---

## 9) Checklist

- [ ] parse `mechanic` ก่อนตัดสิน buy/get
- [ ] ไม่บังคับ buy+get ใน element เดียว
- [ ] `get.field8` ไม่มี
- [ ] `%` ใช้ normalize เต็ม
