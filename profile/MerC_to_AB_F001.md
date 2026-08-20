# C → AB — Profile F001

| | |
|--|--|
| **กลุ่ม** | B — Buy + Get แถวเดียว |
| **Macro** | `F001_Header` + `Gen_F001` |
| **โครง** | `buy: [1]` · `get: [1]` ต่อ 1 แถว `MATERIALS[*]` |
| **จุดต่างหลัก** | เหมือน D001 แต่ `%` ใช้ normalize แบบง่าย |
| **payload หลังบ้านหลัก** | `layout-ab-payload.example.json` |
| **ชื่อฟิลด์ AB อธิบายเพิ่ม** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Header_Shared.md` · `MerC_to_AB_Function_Split.md` |

ยึด payload หลังบ้านเป็นหลัก: `HEADER.period*` = `YYYY-MM-DD`, `HEADER.time*` อยู่ใน header, `field2` ใช้ `"Material"` / `"Material Group"`.

---

## 0) Pipeline F001

```text
normalizeC -> mapHeaderShared -> filter F001 -> buildGroupB -> copy CONDITIONS -> MATERIALS=[]
```

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า F001 |
|--------|----------|
| เขียนฝั่ง | `buy[]` + `get[]` |
| `get.field8` | **omit** |
| Normalize % | **ง่าย** → `pct / 100` |
| `bonusBuyHeader.validTimeFrom/To` | **ใส่** |
| `bonusBuyHeader.referenceCode` | omit |
| `bonusBuyHeader.onlineDescriptionEnglish/Thai` | ใส่เมื่อ `promotionArea = "P4"` |

---

## 2) HEADER

| key | การทำ F001 |
|-----|------------|
| `group`, `purchasingGroup`, `promotionName`, `theme`, `volume`, `singleMultiple`, `vendorName`, `days` | copy |
| `bonusBuyProfile` | `"F001"` |
| `rebateChargeback`, `contractType`, `vendorCode` | shared normalize |
| `wbsNumber` | copy |
| `periodFrom`, `periodTo` | copy เป็น `YYYY-MM-DD` |
| `timeFrom`, `timeTo` | copy ไว้ใน `HEADER` |
| `status` | omit |

---

## 3) `BONUSBUYS[*]`

| AB block | F001 |
|----------|------|
| `lineNumber`, `bonusBuyNumber` | copy / optional |
| `bonusBuyHeader` | ใช้ |
| `buy[]` | `[1]` |
| `get[]` | `[1]` |
| `card[]`, `tender[]`, `installment[]`, `posTerminal[]`, `premium[]`, `coupon[]`, `limitControl[]` | `[]` |
| `stores[]` | copy / `[]` |

### 3.1 `bonusBuyHeader`

| key | การทำ F001 |
|-----|------------|
| `promotionNumber`, `description`, `purchasingGroup`, `product`, `department` | auto / optional |
| `bonusBuyNumber` | copy |
| `bonusBuyProfile` | `"F001"` |
| `mechanic` | copy |
| `validTimeFrom`, `validTimeTo` | ใช้จาก `HEADER.timeFrom/timeTo` |
| `promotionArea`, `wbsNumber` | copy |
| `referenceCode` | omit |
| `onlineDescriptionEnglish`, `onlineDescriptionThai` | ใส่เมื่อ `promotionArea = "P4"` |
| field ที่เหลือใน header block | omit ถ้าไม่มี requirement |

---

## 4) `buy[0]` ครบทุก field

| key | จาก C | การทำ F001 |
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

## 5) `get[0]` ครบทุก field

| key | จาก C | การทำ F001 |
|-----|--------|------------|
| `bonusBuyNumber` | `numberOfBonusBuy` | copy |
| `field2` | material/group | `"Material"` หรือ `"Material Group"` |
| `char` | — | omit |
| `field4` | `material` / `materialGroupName` | calc |
| `field5` | `materialDescription` | copy |
| `sapMasterDescription` | `materialDescription2` | copy |
| `field7` | `costNormal` | copy |
| `field8` | `salesPriceNormal` | **omit** |
| `vat` | `vat` | copy / omit |
| `grossProfit` | — | auto |
| `getQuantity` | `mechanic` | lookup get qty |
| `tierNumber`, `recursive`, `progressive`, `tierQuantityA`, `tierAmountB` | — | omit |
| `field17` | `costPromotion` | copy |
| `fieldP` | `salesPricePromotion` | ส่วนลดลำดับ 1 |
| `vat2` | — | omit |
| `grossProfit2` | — | auto |
| `fieldR` | `discountAmount` | ส่วนลดลำดับ 2 |
| `field22` | percent aliases | ส่วนลดลำดับ 3 + `% / 100` |
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

1. `salesPricePromotion` → `fieldP`
2. `discountAmount` → `fieldR`
3. `discountPercentPlu` / `discountPercentForP015` / `discountPct` → `field22`

Normalize % แบบง่าย:

```text
field22 = pct / 100
```

---

## 8) Skeleton JSON

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

## 9) Checklist

- [ ] `HEADER.period*` เป็น `YYYY-MM-DD`
- [ ] `HEADER.time*` อยู่ใน header
- [ ] `bonusBuyHeader.validTime*` มีค่า
- [ ] `get.field8` ไม่มี
- [ ] `%` ใช้ `pct / 100`
- [ ] online description ใส่เฉพาะ `P4`
