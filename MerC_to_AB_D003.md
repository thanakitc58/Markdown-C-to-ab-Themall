# C → AB — Profile D003

| | |
|--|--|
| **กลุ่ม** | C — แยก element |
| **Macro** | `D003_Header` + `Gen_D003` |
| **โครง** | buy **หรือ** get คนละ `BONUSBUYS` element ตาม mechanic |
| **ชื่อฟิลด์ AB** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Function_Split.md` |
| **Macro รายละเอียด** | `Mer-C_Convert_To_STD.txt` → `Gen_D003` |

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า |
|--------|-----|
| เขียนฝั่ง | ตาม mechanic: ฝั่ง `(A)` → buy · ฝั่ง `(B)` → get |
| Coupon พิเศษ | `A(Coupon)+B(A)` → buy · `+B(B)` → get ของ element ก่อนหน้า |
| `get.field8` | **ไม่ใส่** |
| Normalize % | **เต็ม** |
| `validTime*` / online | ใส่เวลา · online ถ้า P4 |
| `referenceCode` | ไม่ใส่ |

---

## 2) ตาราง C → AB

| จาก C | → AB (JSON) | สูตร / เงื่อนไข |
|-------|-------------|----------------|
| `bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"D003"` |
| `mechanic` | `bonusBuyHeader.mechanic` + **ตัดสินฝั่ง** | parse suffix / รูปแบบ Coupon |
| `mechanic` ชี้ buy | element: `buy: […]` · `get: []` | |
| `mechanic` ชี้ get | element: `buy: []` · `get: […]` | |
| `timeFrom` / `timeTo` | `validTimeFrom` / `validTimeTo` | **ใส่** |
| Online EN / TH | `onlineDescriptionEnglish` / `Thai` | เฉพาะ P4 |
| มี/ไม่มี Material Group | `field2` ฝั่งที่เขียน | MGPNew / MAT |
| material / group name | `field4` ฝั่งที่เขียน | ตาม type |
| Mechanic → buy qty | `buy.field9` | เมื่อเป็นแถว buy |
| Mechanic → get qty | `get.getQuantity` | เมื่อเป็นแถว get |
| ส่วนลด (§3) | `get.fieldP` / `fieldR` / `field22` | เฉพาะ element ฝั่ง get |

แนวทางแยก:

```
ถ้า mechanic ชี้ buy  → { buy: [mapBuy], get: [] }
ถ้า mechanic ชี้ get  → { buy: [], get: [mapGet] }   // ไม่ใส่ get.field8
```

อย่าบังคับ buy+get ใน element เดียวแบบ D001

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด (ฝั่ง get เท่านั้น)

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

## 4) Skeleton JSON (หลาย element)

```json
[
  {
    "bonusBuyHeader": {
      "bonusBuyNumber": "1",
      "bonusBuyProfile": "D003",
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
      "bonusBuyProfile": "D003",
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
        "fieldR": 50
      }
    ]
  }
]
```

---

## 5) Checklist

- [ ] parse `mechanic` ก่อนตัดสิน buy หรือ get
- [ ] ไม่บังคับ buy+get ใน element เดียว
- [ ] `get.field8` ไม่ใส่
- [ ] % แบบเต็ม
- [ ] รอบแรก stub ได้ — อย่า block ของร่วม
