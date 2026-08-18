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

ต้นทาง = path ใน JSON `LAYOUT: "C"` (ดู `layout-c-payload.example.json`)  
ปลายทาง = path ใน JSON `LAYOUT: "AB"` ภายใต้ `BONUSBUYS[*]`

| จาก C (JSON) | → AB (JSON) | สูตร / เงื่อนไข |
|--------------|-------------|----------------|
| `HEADER.bonusBuyProfile` / `MATERIALS[*].bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"P010"` |
| `MATERIALS[*].mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง |
| `HEADER.timeFrom` / `HEADER.timeTo` | — | **ไม่ map** ลง `validTime*` |
| `HEADER.referenceCode` / `MATERIALS[*].referenceCode` | `bonusBuyHeader.referenceCode` | **ใส่** |
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
