# C → AB — Profile P001

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P001_Header` + `Gen_P001` |
| **โครง** | `buy: []` · `get: [1]` |
| **ชื่อฟิลด์ AB** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Function_Split.md` |

---

## 1) กฎเฉพาะ profile

| หัวข้อ | ค่า |
|--------|-----|
| เขียนฝั่ง | `get[]` เท่านั้น |
| `get.field8` (ราคาขายปกติ) | **ใส่** |
| Normalize % | **เต็ม** |
| `validTimeFrom` / `validTimeTo` | **ใส่** จาก header time |
| `referenceCode` | ไม่ใส่ |
| Online EN/TH | ไม่ใส่ |

---

## 2) ตาราง C → AB (แถว Bonus Buy)

| จาก C | → AB (JSON) | สูตร / เงื่อนไข |
|-------|-------------|----------------|
| `bonusBuyProfile` | `bonusBuyHeader.bonusBuyProfile` | `"P001"` |
| `mechanic` | `bonusBuyHeader.mechanic` | ส่งตรง |
| `timeFrom` / `timeTo` (header) | `bonusBuyHeader.validTimeFrom` / `validTimeTo` | ส่งตรง |
| `wbsNo` / `wbsNumber` | `bonusBuyHeader.wbsNumber` | ส่งตรง / default ตาม Group |
| มี/ไม่มี Material Group | `get.field2` | มีกลุ่ม → `"MGPNew - New Material Group"`; ไม่มี → `"MAT - Material"` |
| material / group name | `get.field4` | ตาม `field2` |
| `costNormal` | `get.field7` | optional |
| `salesPriceNormal` | `get.field8` | **ใส่** |
| Mechanic → get qty | `get.getQuantity` | lookup ชีต Mechanic |
| `salesUnit` | `get.unit` | ส่งตรง |
| ส่วนลด (ดู §3) | `get.fieldP` **หรือ** `fieldR` **หรือ** `field22` | อันแรกที่มีเท่านั้น |
| — | `buy` | `[]` เสมอ |

---

## 3) สูตรที่ใช้

### ลำดับส่วนลด (ทุก profile)

| ลำดับ | จาก C | → AB |
|------|-------|------|
| 1 | `salesPricePromo` / `promoPrice` | `get.fieldP` |
| 2 | `discountAmt` | `get.fieldR` |
| 3 | `discountPct` | `get.field22` (หลัง normalize) |

### Normalize % แบบเต็ม

```
ถ้า (% / 100) < 1  → ใช้ค่าเดิม
ถ้า % > 100        → % / 100
ถ้า % < 1          → % * 100
```

### Header ร่วม (ก่อนเข้าแถว)

| จาก C | → AB HEADER | สูตร |
|-------|-------------|------|
| `periodFrom` / `periodTo` | `periodFrom` / `periodTo` | `YYYY-MM-DD` → `DD.MM.YYYY` |
| `vendorCode` ว่าง | `vendorCode` | `"NOBP"` |
| `rebateChargeback` | `rebateChargeback` | ตัดอักษรแรก; ไม่ใช่ Z2 และตัวแรก≠`A` → เติม `0` |
| `contractType` | `contractType` | ขึ้นต้น `Z2` ถึงใส่; ไม่ใช่ → ว่าง |

---

## 4) Skeleton JSON (`BONUSBUYS[]` element)

```json
{
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "P001",
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

- [ ] `buy` = `[]`
- [ ] `get` มี `field2`, `field4`, `field8`, `getQuantity`, `unit`
- [ ] ส่วนลดอันเดียว: `fieldP` หรือ `fieldR` หรือ `field22`
- [ ] header มี `validTime*` ไม่มี `referenceCode` / online
