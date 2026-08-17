# C → AB — Profile F003

| | |
|--|--|
| **กลุ่ม** | C — แยก index |
| **Macro** | `F003_Header` + `Gen_F003` |
| **โครง** | `…(A)` → buy · `…(B)` → get (คนละ index/element) |
| **จุดต่างจาก D003** | Normalize % แบบ **ง่าย** |
| **ชื่อฟิลด์ AB** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Function_Split.md` |
| **เทียบ** | `MerC_to_AB_D003.md` · `MerC_to_AB_F001.md` (% ง่าย) |

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า |
|--------|-----|
| เขียนฝั่ง | `…(A)` → buy · `…(B)` → get (คนละ element) |
| `get.field8` | **ไม่ใส่** |
| Normalize % | **ง่าย** → `field22 = pct / 100` |
| `validTime*` / online | ใส่เวลา · online ถ้า P4 |
| `referenceCode` | ไม่ใส่ |

---

## 2) ตาราง C → AB

| จาก C | → AB (JSON) | สูตร / เงื่อนไข |
|-------|-------------|----------------|
| `bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"F003"` |
| `mechanic` suffix `(A)` | element buy-only | `buy: […]` · `get: []` |
| `mechanic` suffix `(B)` | element get-only | `buy: []` · `get: […]` |
| `timeFrom` / `timeTo` | `validTimeFrom` / `validTimeTo` | **ใส่** |
| Online EN / TH | `onlineDescriptionEnglish` / `Thai` | เฉพาะ P4 |
| มี/ไม่มี Material Group | `field2` ฝั่งที่เขียน | MGPNew / MAT |
| material / group name | `field4` ฝั่งที่เขียน | ตาม type |
| Mechanic → buy qty | `buy.field9` | แถว `(A)` |
| Mechanic → get qty | `get.getQuantity` | แถว `(B)` |
| ส่วนลด (§3) | `get.fieldP` / `fieldR` / `field22` | % ใช้ normalize **ง่าย** |

แนวทางแยก:

```
ถ้า mechanic ลงท้าย (A) → { buy: [mapBuy], get: [] }
ถ้า mechanic ลงท้าย (B) → { buy: [], get: [mapGet] }   // ไม่ใส่ get.field8
```

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด (ฝั่ง get)

| ลำดับ | จาก C | → AB |
|------|-------|------|
| 1 | `salesPricePromo` | `get.fieldP` |
| 2 | `discountAmt` | `get.fieldR` |
| 3 | `discountPct` | `get.field22` |

### Normalize % แบบง่าย (F001 / F003)

```
field22 = discountPct / 100
```

### Header ร่วม

| จาก C | → AB HEADER | สูตร |
|-------|-------------|------|
| `periodFrom` / `periodTo` | `periodFrom` / `periodTo` | `YYYY-MM-DD` → `DD.MM.YYYY` |
| `vendorCode` ว่าง | `vendorCode` | `"NOBP"` |
| `rebateChargeback` | `rebateChargeback` | ตัดอักษรแรก / เติม `0` ตามกฎ |
| `contractType` | `contractType` | ขึ้นต้น `Z2` ถึงใส่ |

---

## 4) Skeleton JSON (หลาย element)

```json
[
  {
    "bonusBuyHeader": {
      "bonusBuyNumber": "1",
      "bonusBuyProfile": "F003",
      "mechanic": "…(A)",
      "validTimeFrom": "00:00:00",
      "validTimeTo": "23:59:59"
    },
    "buy": [
      {
        "field2": "MAT - Material",
        "field4": "123456",
        "field9": 1
      }
    ],
    "get": []
  },
  {
    "bonusBuyHeader": {
      "bonusBuyNumber": "2",
      "bonusBuyProfile": "F003",
      "mechanic": "…(B)",
      "validTimeFrom": "00:00:00",
      "validTimeTo": "23:59:59"
    },
    "buy": [],
    "get": [
      {
        "field2": "MAT - Material",
        "field4": "123456",
        "getQuantity": 1,
        "unit": "EA",
        "field22": 0.1
      }
    ]
  }
]
```

---

## 5) Checklist

- [ ] แยก index ตาม suffix `(A)` / `(B)`
- [ ] % แบบง่ายเหมือน F001
- [ ] `get.field8` ไม่ใส่
- [ ] รอบแรก stub ได้เหมือน D003
