# C → AB — Profile P010

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P010_Header` + `Gen_P010` |
| **โครง** | `buy: []` · `get: [1]` |
| **ต่างจาก P001** | ไม่ใส่เวลา · ใส่ `referenceCode` · Online เมื่อ P4 |
| **ชื่อฟิลด์ AB** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Function_Split.md` |

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า |
|--------|-----|
| เขียนฝั่ง | `get[]` เท่านั้น |
| `get.field8` (ราคาขายปกติ) | **ใส่** |
| Normalize % | **เต็ม** |
| `validTimeFrom` / `validTimeTo` | **ไม่ใส่** |
| `referenceCode` | **ใส่** |
| Online EN/TH | ใส่เมื่อ `promotionArea = "P4"` |

---

## 2) ตาราง C → AB (แถว Bonus Buy)

| จาก C | → AB (JSON) | สูตร / เงื่อนไข |
|-------|-------------|----------------|
| `bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"P010"` |
| `mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง |
| `timeFrom` / `timeTo` | — | **ไม่ map** ลง `validTime*` |
| reference (ต้นทาง) | `bonusBuyHeader.referenceCode` | **ใส่** |
| Online EN / TH | `bonusBuyHeader.onlineDescriptionEnglish` / `Thai` | เฉพาะ `promotionArea = "P4"` |
| `wbsNo` / `wbsNumber` | `bonusBuyHeader.wbsNumber` | ส่งตรง |
| มี/ไม่มี Material Group | `get.field2` | MGPNew / MAT |
| material / group name | `get.field4` | ตาม `field2` |
| `salesPriceNormal` | `get.field8` | **ใส่** |
| Mechanic → get qty | `get.getQuantity` | lookup Mechanic |
| `salesUnit` | `get.unit` | ส่งตรง |
| ส่วนลด (§3) | `get.fieldP` / `fieldR` / `field22` | อันแรกที่มี |
| — | `buy` | `[]` |

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด

| ลำดับ | จาก C | → AB |
|------|-------|------|
| 1 | `salesPricePromo` | `get.fieldP` |
| 2 | `discountAmt` | `get.fieldR` |
| 3 | `discountPct` | `get.field22` (normalize เต็ม) |

### Normalize % แบบเต็ม

```
ถ้า (% / 100) < 1  → ใช้ค่าเดิม
ถ้า % > 100        → % / 100
ถ้า % < 1          → % * 100
```

### Header ร่วม

| จาก C | → AB HEADER | สูตร |
|-------|-------------|------|
| `periodFrom` / `periodTo` | `periodFrom` / `periodTo` | `YYYY-MM-DD` → `DD.MM.YYYY` |
| `vendorCode` ว่าง | `vendorCode` | `"NOBP"` |
| `rebateChargeback` | `rebateChargeback` | ตัดอักษรแรก / เติม `0` ตามกฎ |
| `contractType` | `contractType` | ขึ้นต้น `Z2` ถึงใส่ |

---

## 4) Skeleton JSON (`BONUSBUYS[]` element)

```json
{
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "P010",
    "referenceCode": "…",
    "onlineDescriptionEnglish": "…",
    "onlineDescriptionThai": "…",
    "wbsNumber": "…"
  },
  "buy": [],
  "get": [
    {
      "bonusBuyNumber": "1",
      "field2": "MAT - Material",
      "field4": "123456",
      "field8": 990,
      "getQuantity": 1,
      "unit": "EA",
      "field22": 10
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

## 5) Checklist

- [ ] ไม่มี `validTimeFrom` / `validTimeTo`
- [ ] มี `referenceCode`
- [ ] online เฉพาะเมื่อ `promotionArea = "P4"`
- [ ] get-only + มี `field8`
