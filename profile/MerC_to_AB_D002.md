# C → AB — Profile D002

| | |
|--|--|
| **กลุ่ม** | C |
| **Macro** | `D002_Header` + `Gen_D002` |
| **โครง** | `buy: [1]` · `get: [1]` — **เหมือน D001 (กลุ่ม B)** |
| **Implement** | reuse logic กลุ่ม B (`buildGroupB`) ได้ |
| **ชื่อฟิลด์ AB** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Function_Split.md` |
| **เทียบ** | `MerC_to_AB_D001.md` |

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า |
|--------|-----|
| เขียนฝั่ง | `buy[]` + `get[]` แถวเดียว |
| `get.field8` (ราคาขายปกติ) | **ไม่ใส่** |
| Normalize % | **เต็ม** |
| `validTimeFrom` / `validTimeTo` | **ใส่** |
| `referenceCode` | ไม่ใส่ |
| Online EN/TH | ใส่เมื่อ `promotionArea = "P4"` |

---

## 2) ตาราง C → AB (แถว Bonus Buy)

ต้นทาง = path ใน JSON `LAYOUT: "C"` (ดู `layout-c-payload.example.json`)  
ปลายทาง = path ใน JSON `LAYOUT: "AB"` ภายใต้ `BONUSBUYS[*]`

| จาก C (JSON) | → AB (JSON) | สูตร / เงื่อนไข |
|--------------|-------------|----------------|
| `HEADER.bonusBuyProfile` / `MATERIALS[*].bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"D002"` |
| `MATERIALS[*].mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง + lookup qty |
| `HEADER.timeFrom` / `HEADER.timeTo` | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | **ใส่** |
| `HEADER.onlineDescriptionEnglish` / `HEADER.onlineDescriptionThai` | `bonusBuyHeader.onlineDescriptionEnglish` / `Thai` | เฉพาะ `HEADER.promotionArea = "P4"` |
| `MATERIALS[*].numberOfMaterialGrouping` / `MATERIALS[*].materialGroupName` | `buy.field2` / `get.field2` | MGPNew / MAT |
| `MATERIALS[*].material` / `MATERIALS[*].materialGroupName` | `buy.field4` / `get.field4` | ตาม type |
| `MATERIALS[*].salesPriceNormal` | `buy.field8` | ใส่ฝั่ง buy ได้; **ไม่ใส่** `get.field8` |
| `MATERIALS[*].mechanic` | `buy.field9` | lookup Mechanic → buy qty |
| `MATERIALS[*].mechanic` | `get.getQuantity` | lookup Mechanic → get qty |
| `MATERIALS[*].salesPricePromotion` | `get.fieldP` | ถ้ามีค่านี้ ใช้ช่องนี้ **อย่างเดียว** |
| `MATERIALS[*].discountAmount` | `get.fieldR` | ใช้เมื่อ **ไม่มี** `salesPricePromotion` |
| `MATERIALS[*].discountPercentPlu` / `discountPercentForP015` / `discountPct` | `get.field22` | ใช้เมื่อไม่มีโปรและไม่มีบาท · normalize เต็ม |

Router:

```
ถ้า profile === "D002" → ใช้โครงเดียวกับ D001 (กลุ่ม B)
```

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด

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

## 4) Skeleton JSON (`BONUSBUYS[]` element)

```json
{
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "D002",
    "mechanic": "…",
    "validTimeFrom": "00:00:00",
    "validTimeTo": "23:59:59",
    "wbsNumber": "…"
  },
  "buy": [
    {
      "field2": "MAT - Material",
      "field4": "123456",
      "field9": 1
    }
  ],
  "get": [
    {
      "field2": "MAT - Material",
      "field4": "123456",
      "getQuantity": 1,
      "unit": "EA",
      "fieldR": 50
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

- [ ] อย่าเขียน logic ซ้ำ — reuse กลุ่ม B / D001
- [ ] % แบบเต็ม (ต่างจาก F001)
- [ ] `get` ไม่มี `field8`
