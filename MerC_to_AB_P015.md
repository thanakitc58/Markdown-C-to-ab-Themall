# C → AB — Profile P015

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P015_Header` + `Gen_P015` |
| **โครง** | `buy: []` · `get: [1]` (เหมือน P001) |
| **หมายเหตุ** | ต้นทาง % อาจมาช่อง `Discount % / For P015` — รับใน `discountPct` แล้วลง `get.field22` |
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
| `referenceCode` | ไม่ใส่ |
| Online EN/TH | ไม่ใส่ |

---

## 2) ตาราง C → AB (แถว Bonus Buy)

| จาก C | → AB (JSON) | สูตร / เงื่อนไข |
|-------|-------------|----------------|
| `bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"P015"` |
| `mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง |
| `timeFrom` / `timeTo` | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | **ใส่** |
| `wbsNo` / `wbsNumber` | `bonusBuyHeader.wbsNumber` | ส่งตรง |
| มี/ไม่มี Material Group | `get.field2` | MGPNew / MAT |
| material / group name | `get.field4` | ตาม `field2` |
| `salesPriceNormal` | `get.field8` | **ใส่** |
| Mechanic → get qty | `get.getQuantity` | lookup Mechanic |
| `salesUnit` | `get.unit` | ส่งตรง |
| `discountPct` (รวมช่อง For P015) | `get.field22` | normalize เต็ม หลัง `pickDiscount` |
| ส่วนลดอื่น (§3) | `get.fieldP` / `fieldR` / `field22` | อันแรกที่มี |
| — | `buy` | `[]` |

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด

| ลำดับ | จาก C | → AB |
|------|-------|------|
| 1 | `salesPricePromo` | `get.fieldP` |
| 2 | `discountAmt` | `get.fieldR` |
| 3 | `discountPct` (รวมช่อง P015) | `get.field22` (normalize เต็ม) |

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

## 4) Skeleton JSON (`BONUSBUYS[]` element)

```json
{
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "P015",
    "mechanic": "…",
    "validTimeFrom": "00:00:00",
    "validTimeTo": "23:59:59",
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

- [ ] โครงเหมือน P001 (`buy: []`, มี `field8`, มีเวลา)
- [ ] ไม่มี `referenceCode` / online
- [ ] % จากช่อง P015 ลง `get.field22` ไม่ใช่ชื่อฟิลด์อื่น
