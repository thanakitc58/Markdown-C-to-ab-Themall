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

ต้นทาง = path ใน JSON `LAYOUT: "C"` (ดู `layout-c-payload.example.json`)  
ปลายทาง = path ใน JSON `LAYOUT: "AB"` ภายใต้ `BONUSBUYS[*]`

| จาก C (JSON) | → AB (JSON) | สูตร / เงื่อนไข |
|--------------|-------------|----------------|
| `HEADER.bonusBuyProfile` / `MATERIALS[*].bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"F003"` |
| `MATERIALS[*].mechanic` suffix `(A)` | element buy-only | `buy: […]` · `get: []` |
| `MATERIALS[*].mechanic` suffix `(B)` | element get-only | `buy: []` · `get: […]` |
| `HEADER.timeFrom` / `HEADER.timeTo` | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | **ใส่** |
| `HEADER.onlineDescriptionEnglish` / `HEADER.onlineDescriptionThai` | `bonusBuyHeader.onlineDescriptionEnglish` / `Thai` | เฉพาะ `HEADER.promotionArea = "P4"` |
| `MATERIALS[*].numberOfMaterialGrouping` / `MATERIALS[*].materialGroupName` | `field2` ฝั่งที่เขียน | MGPNew / MAT |
| `MATERIALS[*].material` / `MATERIALS[*].materialGroupName` | `field4` ฝั่งที่เขียน | ตาม type |
| `MATERIALS[*].mechanic` | `buy.field9` | แถว `(A)` |
| `MATERIALS[*].mechanic` | `get.getQuantity` | แถว `(B)` |
| `MATERIALS[*].salesPricePromotion` | `get.fieldP` | แถว `(B)` · ถ้ามีใช้ช่องนี้ **อย่างเดียว** |
| `MATERIALS[*].discountAmount` | `get.fieldR` | แถว `(B)` · ใช้เมื่อไม่มี `salesPricePromotion` |
| `MATERIALS[*].discountPercentPlu` / `discountPercentForP015` / `discountPct` | `get.field22` | แถว `(B)` · ใช้เมื่อไม่มีโปรและไม่มีบาท · `% / 100` |

แนวทางแยก:

```
ถ้า mechanic ลงท้าย (A) → { buy: [mapBuy], get: [] }
ถ้า mechanic ลงท้าย (B) → { buy: [], get: [mapGet] }   // ไม่ใส่ get.field8
```

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด (ฝั่ง get)

| ลำดับ | จาก C (JSON) | → AB (JSON) |
|------|----------------|-------------|
| 1 | `MATERIALS[*].salesPricePromotion` | `get.fieldP` |
| 2 | `MATERIALS[*].discountAmount` | `get.fieldR` |
| 3 | `MATERIALS[*].discountPercentPlu` / `discountPercentForP015` / `discountPct` | `get.field22` |

### Normalize % แบบง่าย (F001 / F003)

```
field22 = MATERIALS[*].discountPercentPlu / 100
(หรือ discountPercentForP015 / discountPct ถ้า payload ใช้ชื่อนี้)
```

### Header ร่วม

| จาก C (JSON) | → AB `HEADER` | สูตร |
|--------------|----------------|------|
| `HEADER.periodFrom` / `HEADER.periodTo` | `HEADER.periodFrom` / `periodTo` | `YYYY-MM-DD` → `DD.MM.YYYY` |
| `HEADER.vendorCode` ว่าง | `HEADER.vendorCode` | `"NOBP"` |
| `HEADER.rebateChargeback` | `HEADER.rebateChargeback` | ตัดอักษรแรก / เติม `0` ตามกฎ |
| `HEADER.contractType` | `HEADER.contractType` | ขึ้นต้น `Z2` ถึงใส่ |

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
