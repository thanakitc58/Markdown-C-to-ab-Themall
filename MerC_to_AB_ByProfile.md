# C → AB — แยกตาม Profile (9 ตัว)

 converter: **ของร่วมอยู่ไฟล์อื่น** ส่วนไฟล์นี้เจาะว่าแต่ละ profile ต่างกันตรงไหน

| เอกสาร | ใช้เมื่อ |
|--------|---------|
| `MerC_to_AB_Phase1_Shared.md` | ของร่วมทุก profile (header / skip / lookup) |
| `MerC_to_AB_Function_Split.md` | โครงโค้ด shared → router กลุ่ม → if ราย profile |
| `payload_AB_Reference.md` | ชื่อ field ปลายทาง AB (JSON contract) |
| **ไฟล์นี้** | สเปกราย profile — copy หัวข้อไป ticket ได้ |

**อ่านยังไง:** ทำ Phase 1 shared ก่อน → router กลุ่ม A/B/C → ค่อยเปิดหัวข้อ profile ที่รับผิดชอบ

---

## ตารางเปรียบเทียบเร็ว

| Profile | กลุ่ม | เขียนฝั่ง | `get.field8` ราคาขายปกติ | % normalize | `validTime*` | `referenceCode` | Online EN/TH (ถ้า P4) |
|---------|------|-----------|:------------------------:|:-----------:|:------------:|:---------------:|:---------------------:|
| **P001** | A | `get[]` เท่านั้น | ✅ | เต็ม | ✅ | ❌ | ❌ |
| **P010** | A | `get[]` เท่านั้น | ✅ | เต็ม | ❌ | ✅ | ✅ |
| **P011** | A | `get[]` เท่านั้น | ✅ | เต็ม | ✅ | ❌ (รับได้แต่ไม่เขียน) | ✅ |
| **P015** | A | `get[]` เท่านั้น | ✅ | เต็ม | ✅ | ❌ | ❌ |
| **D001** | B | `buy[]` + `get[]` | ❌ | เต็ม | ✅ | ❌ | ✅ |
| **F001** | B | `buy[]` + `get[]` | ❌ | **ง่าย** | ✅ | ❌ | ✅ |
| **D002** | C | `buy[]` + `get[]` (เหมือน B) | ❌ | เต็ม | ✅ | ❌ | ✅ |
| **D003** | C | buy **หรือ** get คนละ element | ❌ | เต็ม | ✅ | ❌ | ✅ |
| **F003** | C | buy **หรือ** get แยก index | ❌ | **ง่าย** | ✅ | ❌ | ✅ |

**Normalize เต็ม:** ถ้า `%/100 < 1` ใช้ค่าเดิม; ถ้า `>100` หาร 100; ถ้า `<1` คูณ 100  
**Normalize ง่าย:** `field22 = % / 100` ตรงๆ

**ลำดับส่วนลด (ทุก profile):** `promoPrice` → `fieldP` · `discountAmt` → `fieldR` · `discountPct` → `field22` (อันแรกที่มี)

**Out of scope:** P100, P103, F002

---

## ของร่วม (อย่า copy ซ้ำในแต่ละ profile)

ทำครั้งเดียวใน shared แล้วทุก profile ใช้:

- Header: วันที่ `DD.MM.YYYY`, vendor ว่าง→`NOBP`, rebate ตัดอักษร, contractType เฉพาะ `Z2…`
- แถว: `field2` = MAT / MGPNew, `field4` = material หรือ group name
- Mechanic → `buyQty` / `getQty`
- ส่วนลด `pickDiscount` ตามลำดับด้านบน
- `stores[]`, `card/tender/…` = เว้น `[]` ได้

รายละเอียด: `MerC_to_AB_Phase1_Shared.md` + หมวด 2 ใน `MerC_to_AB_Function_Split.md`

---

# กลุ่ม A — Get-only

โครงร่วมกลุ่ม A:

```js
{
  buy: [],
  get: [mapGetShared(ctx, { includeNormalSellPrice: true })],
  bonusBuyHeader: mapBonusBuyHeaderByProfile(ctx),
  // ...EMPTY_BB_CHILDREN, stores
}
```

---

## P001

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P001_Header` + `Gen_P001` |
| **ใช้เมื่อ** | โปรโมแบบลดราคา / ได้ของฝั่ง Get อย่างเดียว |

### กฎเฉพาะ

| หัวข้อ | ค่า |
|--------|-----|
| เขียน | `get[]` เท่านั้น · `buy: []` |
| `get.field8` | **ใส่** ราคาขายปกติ |
| % | normalize **เต็ม** |
| `validTimeFrom/To` | **ใส่** จาก header time |
| `referenceCode` | ไม่ใส่ |
| Online EN/TH | ไม่ใส่ |

### Checklist

- [ ] `buy` ว่าง
- [ ] `get` มี `field2`, `field4`, `field8`, `getQuantity`, `unit`
- [ ] ส่วนลดอันเดียว: `fieldP` หรือ `fieldR` หรือ `field22`
- [ ] header มีเวลา ไม่มี ref/online

### Skeleton output

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
  "get": [{
    "bonusBuyNumber": "1",
    "field2": "MAT - Material",
    "field4": "123456",
    "field8": 990,
    "getQuantity": 1,
    "unit": "EA",
    "fieldP": 891
  }]
}
```

---

## P010

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P010_Header` + `Gen_P010` |
| **ใช้เมื่อ** | คล้าย P001 แต่มี Reference + Online; **ไม่ใส่เวลา** |

### กฎเฉพาะ

| หัวข้อ | ค่า |
|--------|-----|
| เขียน | `get[]` เท่านั้น · `buy: []` |
| `get.field8` | **ใส่** |
| % | normalize **เต็ม** |
| `validTimeFrom/To` | **ไม่ใส่** |
| `referenceCode` | **ใส่** |
| Online EN/TH | ใส่เมื่อ `promotionArea = "P4"` |

### Checklist

- [ ] ไม่มี `validTimeFrom` / `validTimeTo` ใน header
- [ ] มี `referenceCode`
- [ ] online เฉพาะ P4
- [ ] ที่เหลือเหมือน P001 (get-only + field8)

### Skeleton output

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
  "get": [{
    "field8": 990,
    "field22": 10
  }]
}
```

---

## P011

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P011_Header` + `Gen_P011` |

### กฎเฉพาะ

| หัวข้อ | ค่า |
|--------|-----|
| เขียน | `get[]` เท่านั้น · `buy: []` |
| `get.field8` | **ใส่** |
| % | normalize **เต็ม** |
| `validTimeFrom/To` | **ใส่** |
| `referenceCode` | รับได้จากต้นทาง แต่ **ไม่เขียน** ลง AB |
| Online EN/TH | ใส่เมื่อ `promotionArea = "P4"` |

### Checklist

- [ ] มีเวลา
- [ ] มี online ถ้า P4
- [ ] **ไม่มี** `referenceCode` ใน output
- [ ] get-only + `field8`

---

## P015

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P015_Header` + `Gen_P015` |
| **หมายเหตุ** | โครงเหมือน P001; ต้นทาง % อาจมาช่อง `Discount % / For P015` — บนเว็บรับค่าใน `discountPct` พอ |

### กฎเฉพาะ

| หัวข้อ | ค่า |
|--------|-----|
| เขียน | `get[]` เท่านั้น · `buy: []` |
| `get.field8` | **ใส่** |
| % | normalize **เต็ม** |
| `validTimeFrom/To` | **ใส่** |
| `referenceCode` | ไม่ใส่ |
| Online EN/TH | ไม่ใส่ |

### Checklist

- [ ] เหมือน P001
- [ ] % มาจากช่อง P015 ได้ แต่ปลายทางยังเป็น `get.field22`

---

# กลุ่ม B — Buy + Get แถวเดียว

โครงร่วมกลุ่ม B:

```js
{
  buy: [mapBuyShared(ctx)],   // field4 + field9(buyQty)
  get: [mapGetShared(ctx, { includeNormalSellPrice: false })],
  bonusBuyHeader: mapBonusBuyHeaderByProfile(ctx),
}
```

---

## D001

| | |
|--|--|
| **กลุ่ม** | B — Buy+Get แถวเดียว |
| **Macro** | `D001_Header` + `Gen_D001` |

### กฎเฉพาะ

| หัวข้อ | ค่า |
|--------|-----|
| เขียน | `buy[]` + `get[]` (element เดียวต่อฝั่ง) |
| `get.field8` | **ไม่ใส่** |
| `buy` | `field2`, `field4`, `field9` (= buyQty จาก Mechanic) |
| % | normalize **เต็ม** |
| `validTimeFrom/To` | **ใส่** |
| Online EN/TH | ใส่เมื่อ P4 |

### Checklist

- [ ] มีทั้ง buy และ get
- [ ] get **ไม่มี** `field8`
- [ ] buy มี qty ที่ `field9`
- [ ] % แบบเต็ม

### Skeleton output

```json
{
  "bonusBuyHeader": {
    "bonusBuyProfile": "D001",
    "validTimeFrom": "…",
    "validTimeTo": "…",
    "mechanic": "B1G1"
  },
  "buy": [{
    "field2": "MAT - Material",
    "field4": "123456",
    "field9": 1
  }],
  "get": [{
    "field2": "MAT - Material",
    "field4": "123456",
    "getQuantity": 1,
    "unit": "EA",
    "fieldR": 50
  }]
}
```

---

## F001

| | |
|--|--|
| **กลุ่ม** | B — Buy+Get แถวเดียว |
| **Macro** | `F001_Header` + `Gen_F001` |
| **จุดต่างจาก D001** | % ใช้ normalize **ง่าย** |

### กฎเฉพาะ

| หัวข้อ | ค่า |
|--------|-----|
| เขียน | `buy[]` + `get[]` |
| `get.field8` | **ไม่ใส่** |
| % | **ง่าย** → `field22 = pct / 100` |
| `validTimeFrom/To` | **ใส่** |
| Online EN/TH | ใส่เมื่อ P4 |

### Checklist

- [ ] โครงเหมือน D001
- [ ] ถ้ามีส่วนลด % → ใช้ `normalizePctSimple` ไม่ใช่แบบเต็ม

---

# กลุ่ม C — แยกแถว / หลาย index

---

## D002

| | |
|--|--|
| **กลุ่ม** | C |
| **Macro** | `D002_Header` + `Gen_D002` |
| **จุดสำคัญ** | **โครงเหมือน D001 (กลุ่ม B)** — implement โดยเรียก `buildGroupB` ได้ |

### กฎเฉพาะ

| หัวข้อ | ค่า |
|--------|-----|
| เขียน | `buy[]` + `get[]` แถวเดียว |
| `get.field8` | **ไม่ใส่** |
| % | normalize **เต็ม** |
| `validTime*` / online | เหมือน D001 |

### Checklist

- [ ] อย่าเขียน logic ซ้ำ — reuse กลุ่ม B
- [ ] % แบบเต็ม (ต่างจาก F001)

```js
if (profile === "D002") return buildGroupB(ctx);
```

---

## D003

| | |
|--|--|
| **กลุ่ม** | C — แยก element |
| **Macro** | `D003_Header` + `Gen_D003` |
| **ความยาก** | buy กับ get **อยู่คนละ BONUSBUYS element** ตาม mechanic |

### กฎเฉพาะ

| หัวข้อ | ค่า |
|--------|-----|
| เขียน | ตาม mechanic: ฝั่ง `(A)` → buy · ฝั่ง `(B)` → get |
| Coupon แบบพิเศษ | `A(Coupon)+B(A)` → buy · `+B(B)` → get ของ element ก่อนหน้า |
| `get.field8` | **ไม่ใส่** |
| % | normalize **เต็ม** |
| `validTime*` / online | ใส่เวลา + online ถ้า P4 |

### Checklist

- [ ] อย่าบังคับ buy+get ใน element เดียวแบบ D001
- [ ] parse `mechanic` เพื่อตัดสินว่าแถวนี้เป็น buy หรือ get
- [ ] รอบแรกของทีม: stub / TODO ได้ — **อย่า block ของร่วม** (`MerC_to_AB_Function_Split.md` รอบ 5)

### แนวทางแยก

```js
// แนวคิด — รายละเอียดสูตร mechanic ดู macro Gen_D003
if (mechanicIndicatesBuy(ctx.mechanic)) {
  return { buy: [mapBuyShared(ctx)], get: [], ... };
}
if (mechanicIndicatesGet(ctx.mechanic)) {
  return { buy: [], get: [mapGetShared(ctx, { includeNormalSellPrice: false })], ... };
}
```

---

## F003

| | |
|--|--|
| **กลุ่ม** | C — แยก index |
| **Macro** | `F003_Header` + `Gen_F003` |
| **จุดต่างจาก D003** | % ใช้ normalize **ง่าย**; แยก `(A)`→buy · `(B)`→get |

### กฎเฉพาะ

| หัวข้อ | ค่า |
|--------|-----|
| เขียน | `…(A)` → buy · `…(B)` → get (คนละ index/element) |
| `get.field8` | **ไม่ใส่** |
| % | **ง่าย** |
| `validTime*` / online | ใส่เวลา + online ถ้า P4 |

### Checklist

- [ ] แยก index ตาม suffix mechanic
- [ ] % แบบง่ายเหมือน F001
- [ ] รอบแรก stub ได้เหมือน D003

---

## สรุปมอบงานเพื่อน

| คน | รับ | พึ่งไฟล์ |
|----|-----|---------|
| คน A | ของร่วม + HEADER | `Phase1_Shared` + `Function_Split` §2 |
| คน B | กลุ่ม A: P001 / P010 / P011 / P015 | หัวข้อกลุ่ม A ในไฟล์นี้ |
| คน C | กลุ่ม B: D001 / F001 + D002 | หัวข้อกลุ่ม B + D002 |
| คน D | D003 / F003 (รอบหลัง) | หัวข้อ D003 / F003 |

ชื่อ field ปลายทางทุกคนล็อกกับ `payload_AB_Reference.md` — **อย่าตั้งชื่อใหม่**
