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

| จาก C | → AB (JSON) | สูตร / เงื่อนไข |
|-------|-------------|----------------|
| `bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"F001"` |
| `mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง + lookup qty |
| `timeFrom` / `timeTo` | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | **ใส่** |
| Online EN / TH | `onlineDescriptionEnglish` / `Thai` | เฉพาะ P4 |
| มี/ไม่มี Material Group | `buy.field2` / `get.field2` | MGPNew / MAT |
| material / group name | `buy.field4` / `get.field4` | ตาม type |
| `salesPriceNormal` | `buy.field8` ได้; **ไม่ใส่** `get.field8` | |
| Mechanic → buy qty | `buy.field9` | lookup Mechanic |
| Mechanic → get qty | `get.getQuantity` | lookup Mechanic |
| `salesUnit` | `buy.salesUnit` / `get.unit` | ส่งตรง |
| ส่วนลด (§3) | `get.fieldP` / `fieldR` / `field22` | % ใช้ normalize **ง่าย** |

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด

| ลำดับ | จาก C | → AB |
|------|-------|------|
| 1 | `salesPricePromo` | `get.fieldP` |
| 2 | `discountAmt` | `get.fieldR` |
| 3 | `discountPct` | `get.field22` |

### Normalize % แบบง่าย (F001 / F003)

```
field22 = discountPct / 100
```

ไม่ปรับช่วงแบบเต็ม

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
