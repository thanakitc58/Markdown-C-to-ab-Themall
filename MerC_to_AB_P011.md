# C → AB — Profile P011

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P011_Header` + `Gen_P011` |
| **โครง** | `buy: []` · `get: [1]` |
| **ต่างจาก P001** | ใส่ Online เมื่อ P4 · รับ reference ได้แต่**ไม่เขียน**ลง AB |
| **ชื่อฟิลด์ AB** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Function_Split.md` |

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า |
|--------|-----|
| เขียนฝั่ง | `get[]` เท่านั้น |
| `get.field8` (ราคาขายปกติ) | **ใส่** |
| Normalize % | **เต็ม** |
| `validTimeFrom` / `validTimeTo` | **ใส่** |
| `referenceCode` | รับจากต้นทางได้ แต่ **ไม่เขียน** ลง output |
| Online EN/TH | ใส่เมื่อ `promotionArea = "P4"` |

---

## 2) ตาราง C → AB (แถว Bonus Buy)

| จาก C | → AB (JSON) | สูตร / เงื่อนไข |
|-------|-------------|----------------|
| `bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"P011"` |
| `mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง |
| `timeFrom` / `timeTo` | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | **ใส่** |
| reference (ต้นทาง) | — | **ไม่ map** ลง `referenceCode` |
| Online EN / TH | `onlineDescriptionEnglish` / `Thai` | เฉพาะ `promotionArea = "P4"` |
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
    "bonusBuyProfile": "P011",
    "validTimeFrom": "00:00:00",
    "validTimeTo": "23:59:59",
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
      "fieldP": 891
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

- [ ] มี `validTime*`
- [ ] มี online ถ้า P4
- [ ] **ไม่มี** `referenceCode` ใน output
- [ ] get-only + มี `field8`
