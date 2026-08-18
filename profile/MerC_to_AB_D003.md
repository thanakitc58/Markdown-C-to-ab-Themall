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

ต้นทาง = path ใน JSON `LAYOUT: "C"` (ดู `layout-c-payload.example.json`)  
ปลายทาง = path ใน JSON `LAYOUT: "AB"` ภายใต้ `BONUSBUYS[*]`

| จาก C (JSON) | → AB (JSON) | สูตร / เงื่อนไข |
|--------------|-------------|----------------|
| `HEADER.bonusBuyProfile` / `MATERIALS[*].bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"D003"` |
| `MATERIALS[*].mechanic` | `bonusBuyHeader.mechanic` + **ตัดสินฝั่ง** | parse suffix / รูปแบบ Coupon |
| `MATERIALS[*].mechanic` ชี้ buy | element: `buy: […]` · `get: []` | |
| `MATERIALS[*].mechanic` ชี้ get | element: `buy: []` · `get: […]` | |
| `HEADER.timeFrom` / `HEADER.timeTo` | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | **ใส่** |
| `HEADER.onlineDescriptionEnglish` / `HEADER.onlineDescriptionThai` | `bonusBuyHeader.onlineDescriptionEnglish` / `Thai` | เฉพาะ `HEADER.promotionArea = "P4"` |
| `MATERIALS[*].numberOfMaterialGrouping` / `MATERIALS[*].materialGroupName` | `field2` ฝั่งที่เขียน | MGPNew / MAT |
| `MATERIALS[*].material` / `MATERIALS[*].materialGroupName` | `field4` ฝั่งที่เขียน | ตาม type |
| `MATERIALS[*].mechanic` | `buy.field9` | เมื่อเป็นแถว buy |
| `MATERIALS[*].mechanic` | `get.getQuantity` | เมื่อเป็นแถว get |
| `MATERIALS[*].salesPricePromotion` | `get.fieldP` | เฉพาะ element ฝั่ง get · ถ้ามีใช้ช่องนี้ **อย่างเดียว** |
| `MATERIALS[*].discountAmount` | `get.fieldR` | ฝั่ง get · ใช้เมื่อไม่มี `salesPricePromotion` |
| `MATERIALS[*].discountPercentPlu` / `discountPercentForP015` / `discountPct` | `get.field22` | ฝั่ง get · ใช้เมื่อไม่มีโปรและไม่มีบาท · normalize เต็ม |

แนวทางแยก:

```
ถ้า mechanic ชี้ buy  → { buy: [mapBuy], get: [] }
ถ้า mechanic ชี้ get  → { buy: [], get: [mapGet] }   // ไม่ใส่ get.field8
```

อย่าบังคับ buy+get ใน element เดียวแบบ D001

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด (ฝั่ง get เท่านั้น)

| ลำดับ | จาก C (JSON) | → AB (JSON) |
|------|----------------|-------------|
| 1 | `MATERIALS[*].salesPricePromotion` | `get.fieldP` |
| 2 | `MATERIALS[*].discountAmount` | `get.fieldR` |
| 3 | `MATERIALS[*].discountPercentPlu` / `discountPercentForP015` / `discountPct` | `get.field22` (normalize เต็ม) |

### Normalize % แบบเต็ม

```
ถ้า (% / 100) < 1  → ใช้ค่าเดิม
ถ้า % > 100        → % / 100
ถ้า % < 1          → % * 100
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
