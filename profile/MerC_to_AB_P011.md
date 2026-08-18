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

ต้นทาง = path ใน JSON `LAYOUT: "C"` (ดู `layout-c-payload.example.json`)  
ปลายทาง = path ใน JSON `LAYOUT: "AB"` ภายใต้ `BONUSBUYS[*]`

| จาก C (JSON) | → AB (JSON) | สูตร / เงื่อนไข |
|--------------|-------------|----------------|
| `HEADER.bonusBuyProfile` / `MATERIALS[*].bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"P011"` |
| `MATERIALS[*].mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง |
| `HEADER.timeFrom` / `HEADER.timeTo` | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | **ใส่** |
| `HEADER.referenceCode` / `MATERIALS[*].referenceCode` | — | **ไม่ map** ลง `referenceCode` |
| `HEADER.onlineDescriptionEnglish` / `HEADER.onlineDescriptionThai` | `bonusBuyHeader.onlineDescriptionEnglish` / `Thai` | เฉพาะ `HEADER.promotionArea = "P4"` |
| `HEADER.wbsNumber` | `bonusBuyHeader.wbsNumber` | ส่งตรง (สำรอง `HEADER.wbsNo`) |
| `MATERIALS[*].numberOfMaterialGrouping` / `MATERIALS[*].materialGroupName` | `get.field2` | MGPNew / MAT |
| `MATERIALS[*].material` / `MATERIALS[*].materialGroupName` | `get.field4` | ตาม `field2` |
| `MATERIALS[*].salesPriceNormal` | `get.field8` | **ใส่** |
| `MATERIALS[*].mechanic` | `get.getQuantity` | lookup Mechanic |
| `MATERIALS[*].salesUnit` | `get.unit` | ส่งตรง |
| `MATERIALS[*].salesPricePromotion` | `get.fieldP` | ถ้ามีค่านี้ ใช้ช่องนี้ **อย่างเดียว** |
| `MATERIALS[*].discountAmount` | `get.fieldR` | ใช้เมื่อ **ไม่มี** `salesPricePromotion` |
| `MATERIALS[*].discountPercentPlu` / `discountPercentForP015` / `discountPct` | `get.field22` | ใช้เมื่อไม่มีโปรและไม่มีบาท · normalize เต็ม |
| — | `buy` | `[]` |

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
