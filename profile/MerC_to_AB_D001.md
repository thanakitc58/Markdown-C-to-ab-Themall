# C → AB — Profile D001

| | |
|--|--|
| **กลุ่ม** | B — Buy + Get แถวเดียว |
| **Macro** | `D001_Header` + `Gen_D001` |
| **โครง** | `buy: [1]` · `get: [1]` |
| **ชื่อฟิลด์ AB** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Function_Split.md` |

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า |
|--------|-----|
| เขียนฝั่ง | `buy[]` + `get[]` (element เดียวต่อฝั่ง) |
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
| `HEADER.bonusBuyProfile` / `MATERIALS[*].bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"D001"` |
| `MATERIALS[*].mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง + ใช้ lookup qty |
| `HEADER.timeFrom` / `HEADER.timeTo` | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | **ใส่** (ว่าง → `00:00:00` / `23:59:59`) |
| `HEADER.onlineDescriptionEnglish` / `HEADER.onlineDescriptionThai` | `bonusBuyHeader.onlineDescriptionEnglish` / `Thai` | เฉพาะเมื่อ `HEADER.promotionArea = "P4"` |
| `HEADER.wbsNumber` | `bonusBuyHeader.wbsNumber` | ส่งตรง (สำรอง `HEADER.wbsNo`) |
| `MATERIALS[*].numberOfMaterialGrouping` / `MATERIALS[*].materialGroupName` | `buy.field2` และ `get.field2` | มีกลุ่ม → `"MGPNew - New Material Group"` · ไม่มี → `"MAT - Material"` |
| `MATERIALS[*].material` / `MATERIALS[*].materialGroupName` | `buy.field4` และ `get.field4` | ตาม type |
| `MATERIALS[*].salesPriceNormal` | `buy.field8` | ใส่ฝั่ง buy ได้; **ไม่ใส่** `get.field8` |
| `MATERIALS[*].mechanic` | `buy.field9` | lookup Mechanic → buy qty |
| `MATERIALS[*].salesUnit` | `buy.salesUnit` / `get.unit` | ส่งตรง |
| `MATERIALS[*].mechanic` | `get.getQuantity` | lookup Mechanic → get qty |
| `MATERIALS[*].salesPricePromotion` | `get.fieldP` | ถ้ามีค่านี้ ใช้ช่องนี้ **อย่างเดียว** (ราคาโปร) |
| `MATERIALS[*].discountAmount` | `get.fieldR` | ใช้เมื่อ **ไม่มี** `salesPricePromotion` (ส่วนลดบาท) |
| `MATERIALS[*].discountPercentPlu` / `discountPercentForP015` / `discountPct` | `get.field22` | ใช้เมื่อ **ไม่มี** โปรและไม่มีบาท (ส่วนลด %) · normalize เต็ม |

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
    "bonusBuyProfile": "D001",
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

- [ ] มีทั้ง `buy` และ `get`
- [ ] `get` **ไม่มี** `field8`
- [ ] `buy` มี `field9` (buy qty)
- [ ] % แบบเต็ม
