# C → AB — Profile F001

| | |
|--|--|
| **กลุ่ม** | B — Buy + Get แถวเดียว |
| **Macro** | `F001_Header` + `Gen_F001` |
| **โครง** | `buy: [1]` · `get: [1]` (เหมือน D001) |
| **จุดต่างจาก D001** | Normalize % แบบ **ง่าย** |
| **ชื่อฟิลด์ AB** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Function_Split.md` |

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า |
|--------|-----|
| เขียนฝั่ง | `buy[]` + `get[]` |
| `get.field8` (ราคาขายปกติ) | **ไม่ใส่** |
| Normalize % | **ง่าย** → `field22 = pct / 100` |
| `validTimeFrom` / `validTimeTo` | **ใส่** |
| `referenceCode` | ไม่ใส่ |
| Online EN/TH | ใส่เมื่อ `promotionArea = "P4"` |

---

## 2) ตาราง C → AB (แถว Bonus Buy)

ต้นทาง = path ใน JSON `LAYOUT: "C"` (ดู `layout-c-payload.example.json`)  
ปลายทาง = path ใน JSON `LAYOUT: "AB"` ภายใต้ `BONUSBUYS[*]`

| จาก C (JSON) | → AB (JSON) | สูตร / เงื่อนไข |
|--------------|-------------|----------------|
| `HEADER.bonusBuyProfile` / `MATERIALS[*].bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"F001"` |
| `MATERIALS[*].mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง + lookup qty |
| `HEADER.timeFrom` / `HEADER.timeTo` | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | **ใส่** |
| `HEADER.onlineDescriptionEnglish` / `HEADER.onlineDescriptionThai` | `bonusBuyHeader.onlineDescriptionEnglish` / `Thai` | เฉพาะ `HEADER.promotionArea = "P4"` |
| `MATERIALS[*].numberOfMaterialGrouping` / `MATERIALS[*].materialGroupName` | `buy.field2` / `get.field2` | MGPNew / MAT |
| `MATERIALS[*].material` / `MATERIALS[*].materialGroupName` | `buy.field4` / `get.field4` | ตาม type |
| `MATERIALS[*].salesPriceNormal` | `buy.field8` | ใส่ฝั่ง buy ได้; **ไม่ใส่** `get.field8` |
| `MATERIALS[*].mechanic` | `buy.field9` | lookup Mechanic → buy qty |
| `MATERIALS[*].mechanic` | `get.getQuantity` | lookup Mechanic → get qty |
| `MATERIALS[*].salesUnit` | `buy.salesUnit` / `get.unit` | ส่งตรง |
| `MATERIALS[*].salesPricePromotion` | `get.fieldP` | ถ้ามีค่านี้ ใช้ช่องนี้ **อย่างเดียว** |
| `MATERIALS[*].discountAmount` | `get.fieldR` | ใช้เมื่อ **ไม่มี** `salesPricePromotion` |
| `MATERIALS[*].discountPercentPlu` / `discountPercentForP015` / `discountPct` | `get.field22` | ใช้เมื่อไม่มีโปรและไม่มีบาท · `% / 100` |

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด

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

ไม่ปรับช่วงแบบเต็ม

### Header ร่วม

| จาก C (JSON) | → AB `HEADER` | สูตร |
|--------------|----------------|------|
| `HEADER.periodFrom` / `HEADER.periodTo` | `HEADER.periodFrom` / `periodTo` | `YYYY-MM-DD` → `DD.MM.YYYY` |
| `HEADER.vendorCode` ว่าง | `HEADER.vendorCode` | `"NOBP"` |
| `HEADER.rebateChargeback` | `HEADER.rebateChargeback` | ตัดอักษรแรก / เติม `0` ตามกฎ |
| `HEADER.contractType` | `HEADER.contractType` | ขึ้นต้น `Z2` ถึงใส่ |

---

## 4) Skeleton JSON (`BONUSBUYS[]` element)

```json
{
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "F001",
    "mechanic": "B1G1",
    "validTimeFrom": "00:00:00",
    "validTimeTo": "23:59:59",
    "wbsNumber": "…"
  },
  "buy": [
    {
      "bonusBuyNumber": "1",
      "field2": "MAT - Material",
      "field4": "123456",
      "field9": 1,
      "salesUnit": "EA"
    }
  ],
  "get": [
    {
      "bonusBuyNumber": "1",
      "field2": "MAT - Material",
      "field4": "123456",
      "getQuantity": 1,
      "unit": "EA",
      "field22": 0.1
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

- [ ] โครงเหมือน D001 (buy + get, ไม่มี `get.field8`)
- [ ] ถ้ามีส่วนลด % → ใช้ `pct / 100` ไม่ใช่ normalize เต็ม
