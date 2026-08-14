# C → AB: แบ่ง function — ของร่วมก่อน แล้วค่อย if/else ตาม Profile

สำหรับเพื่อนที่เริ่มเขียน converter: **อย่า if/else 9 ทางตั้งแต่บรรทัดแรก**

ลำดับงาน:
1. **ของร่วมทุก profile** (shared) → ทำก่อน รันได้เลยแม้ยังไม่มี buy/get ครบ
2. **router 3 กลุ่ม** (A / B / C) → แยกโครง `buy[]` / `get[]`
3. **if/else ราย profile เฉพาะจุดที่ต่าง** → time / referenceCode / % normalize / online

อ้างอิงชื่อฟิลด์: `payload_AB_Reference.md`  
อ้างอิงของร่วม: `MerC_to_AB_Phase1_Shared.md`  
อ้างอิงราย profile (ส่งเพื่อน): `MerC_to_AB_ByProfile.md`

**9 profile ลูกค้า**

| กลุ่ม | Profile | โครงหลัก |
|------|---------|----------|
| A | P001, P010, P011, P015 | `get[]` อย่างเดียว |
| B | D001, F001 | `buy[]` + `get[]` แถวเดียว |
| C | D002, D003, F003 | แยกตาม mechanic (D002 ≈ B; D003/F003 แยก element) |

---

## 1) โครงไฟล์แนะนำ

```
convertLayoutCToAbPayload(payload)     // entry
  normalizeC(payload)                  // ชื่อ field / section ให้เป็น camel + UPPERCASE
  mapHeaderShared(header)              // ของร่วม
  mapConditionsShared(conditions)      // ของร่วม (รอบแรก copy + compact พอ)
  materialsToBonusBuys(...)            // วนแต่ละแถว → router
    buildSharedLineContext(material, header)   // ของร่วมต่อแถว
    routeByProfile(ctx)                        // if/else กลุ่ม + ราย profile
```

อย่าใส่ logic ราคา/ส่วนลด/ประเภทสินค้า ลงใน `if (profile === "P010")` — ใส่ใน shared แล้วให้กลุ่มตัดสินแค่ **เขียนฝั่งไหน**

---

## 2) ของร่วม — เขียนก่อน (ทุก profile ใช้ชุดเดียวกัน)

### 2.1 Constants

```js
const GROUP_A = ["P001", "P010", "P011", "P015"]; // Get-only
const GROUP_B = ["D001", "F001"];                 // Buy+Get แถวเดียว
const GROUP_C = ["D002", "D003", "F003"];         // แยกแถว / หลาย index
const ALL_9 = [...GROUP_A, ...GROUP_B, ...GROUP_C];

const EMPTY_BB_CHILDREN = {
  card: [], tender: [], installment: [], posTerminal: [],
  premium: [], coupon: [], limitControl: [],
};
```

### 2.2 Shared helpers (ไม่ขึ้นกับ profile)

| function | ทำอะไร | ผลลัพธ์ |
|----------|--------|---------|
| `isoToSapDate` | `YYYY-MM-DD` → `DD.MM.YYYY` | วันที่ AB |
| `vendorOrNobp` | ว่าง → `"NOBP"` | `vendorCode` |
| `normalizeRebate` | ตัดอักษรแรก / เติม `0` ตามกฎ | `rebateChargeback` |
| `normalizeContractType` | ขึ้นต้น `Z2` ถึงใส่ | `contractType` |
| `materialType` | มี mat group → MGPNew; ไม่มี → Material | `field2` |
| `materialOrGroup` | ตาม type เลือก material / group name | `field4` |
| `pickDiscount` | promo → amt → % (อันแรกที่มี) | `{ kind, value }` |
| `normalizePctFull` / `normalizePctSimple` | ปรับ % | `field22` |
| `lookupBuyGetQty` | จาก Mechanic | `{ buyQty, getQty }` |

ค่า `field2` ที่ถูก (อย่า hardcode `MGPSAP`):

```js
// ชื่อเต็มตาม macro / To-Be
"MAT - Material"
"MGPNew - New Material Group"
// หรือรูปแบบ dropdown ของ UI เช่น "1 - Material" — ล็อกกับ To-Be/UI ให้ตรงกันทีหลัง
```

ส่วนลด (ร่วมทุก profile):

```js
function pickDiscount(material) {
  if (material.salesPricePromo) return { kind: "P", value: material.salesPricePromo }; // → get.fieldP
  if (material.discountAmt)     return { kind: "R", value: material.discountAmt };     // → get.fieldR
  if (material.discountPct)     return { kind: "PCT", value: material.discountPct };   // → get.field22
  return null;
}
```

### 2.3 `mapHeaderShared` — ทำรอบแรกได้เลย

```js
function mapHeaderShared(header) {
  return compact({
    theme: header.theme,
    promotionName: header.promotionName,
    group: header.group,
    purchasingGroup: header.purchasingGroup,
    bonusBuyProfile: header.bonusBuyProfile,
    rebateChargeback: normalizeRebate(header.rebateChargeback, header.contractType),
    contractType: normalizeContractType(header.contractType),
    wbsNumber: header.wbsNumber ?? header.wbsNo,
    vendorCode: vendorOrNobp(header.vendorCode),
    vendorName: header.vendorName,
    periodFrom: isoToSapDate(header.periodFrom),
    periodTo: isoToSapDate(header.periodTo),
    timeFrom: header.timeFrom,   // copy ลงแถว BBY ตาม profile ทีหลัง
    timeTo: header.timeTo,
    days: header.days,
    singleMultiple: header.singleMultiple,
    volume: header.volume ?? header.vol,
    status: capitalize(header.status),
  });
}
```

### 2.4 `buildSharedLineContext` — วัตถุดิบแถว (ยังไม่แยก buy/get)

```js
function buildSharedLineContext(material, header) {
  const profile = shortCode(material.bonusBuyProfile || header.bonusBuyProfile);
  if (!ALL_9.includes(profile)) throw new Error(`Unsupported profile: ${profile}`);

  const type = materialType(material);           // MAT vs MGPNew
  const code = materialOrGroup(material, type);  // field4
  const qty = lookupBuyGetQty(material.mechanic);
  const discount = pickDiscount(material);

  return {
    profile,
    material,
    header,
    type,
    code,
    buyQty: qty.buyQty,
    getQty: qty.getQty,
    discount,
    normalSellPrice: material.salesPriceNormal,  // → get.field8 (เฉพาะกลุ่ม A)
    costPrice: material.costNormal,              // → field7 (optional)
    unit: material.salesUnit,
    stores: material.stores || [],
    mechanic: material.mechanic,
  };
}
```

ตรงนี้คือ **“ของเหมือนกัน”** ที่เพื่อนเริ่มเขียนได้ก่อน — ยังไม่ต้องมี if profile

---

## 3) Router — if/else แบ่ง 3 ชั้น (อย่าแตก 9 ทางตั้งแต่แรก)

### ชั้น 1: กลุ่ม A / B / C (โครง buy/get)

```js
function routeByProfile(ctx) {
  const { profile } = ctx;

  if (GROUP_A.includes(profile)) {
    return buildGroupA(ctx); // get only
  }
  if (GROUP_B.includes(profile)) {
    return buildGroupB(ctx); // buy + get
  }
  if (GROUP_C.includes(profile)) {
    return buildGroupC(ctx); // D002 like B; D003/F003 special
  }
  throw new Error(`Unsupported profile: ${profile}`);
}
```

### ชั้น 2: ของร่วมใน get / buy (ใช้ซ้ำข้ามกลุ่ม)

```js
function mapGetShared(ctx, { includeNormalSellPrice }) {
  const row = {
    bonusBuyNumber: ctx.bonusBuyNumber,
    field2: ctx.type,
    field4: ctx.code,
    field7: ctx.costPrice,
    getQuantity: ctx.getQty,
    unit: ctx.unit,
  };

  if (includeNormalSellPrice) {
    row.field8 = ctx.normalSellPrice; // กลุ่ม A เท่านั้น
  }

  // ส่วนลด — อันเดียวตามลำดับ
  applyDiscount(row, ctx.discount, ctx.profile);

  return compact(row);
}

function mapBuyShared(ctx) {
  return compact({
    bonusBuyNumber: ctx.bonusBuyNumber,
    field2: ctx.type,
    field4: ctx.code,
    field7: ctx.costPrice,
    field8: ctx.normalSellPrice,
    field9: ctx.buyQty, // Buy qty จาก Mechanic
    salesUnit: ctx.unit,
  });
}

function applyDiscount(row, discount, profile) {
  if (!discount) return;
  if (discount.kind === "P") {
    row.fieldP = discount.value;
  } else if (discount.kind === "R") {
    row.fieldR = discount.value;
  } else if (discount.kind === "PCT") {
    row.field22 = needsSimplePct(profile)
      ? normalizePctSimple(discount.value)
      : normalizePctFull(discount.value);
  }
}

function needsSimplePct(profile) {
  return profile === "F001" || profile === "F003";
}
```

### ชั้น 3: if/else ราย profile — เฉพาะ header ที่ต่าง

อย่า copy ทั้ง buy/get ใหม่ทีละ profile — แยกแค่ `bonusBuyHeader`:

```js
function mapBonusBuyHeaderByProfile(ctx) {
  const { profile, header, material } = ctx;

  // --- ของร่วมทุก profile ---
  const base = {
    bonusBuyNumber: ctx.bonusBuyNumber,
    bonusBuyProfile: profile,
    mechanic: ctx.mechanic,
    wbsNumber: header.wbsNumber ?? header.wbsNo,
    description: buildDescription(header, material), // formula ร่วม (หรือ TBD)
    promotionArea: resolvePromotionArea(header, material), // TBD macro → ใส่ทีหลังได้
  };

  // --- จุดที่ต่าง: เวลา / reference / online ---
  if (profile === "P010") {
    // ไม่ใส่ validTime*; ใส่ referenceCode; online ถ้า P4
    return compact({
      ...base,
      referenceCode: material.referenceCode,
      ...onlineIfP4(base.promotionArea, material),
    });
  }

  if (profile === "P001" || profile === "P015") {
    // ใส่เวลา; ไม่มี ref / online
    return compact({
      ...base,
      validTimeFrom: header.timeFrom,
      validTimeTo: header.timeTo,
    });
  }

  if (profile === "P011") {
    // ใส่เวลา + online ถ้า P4; ไม่เขียน reference
    return compact({
      ...base,
      validTimeFrom: header.timeFrom,
      validTimeTo: header.timeTo,
      ...onlineIfP4(base.promotionArea, material),
    });
  }

  // D001, D002, D003, F001, F003 — ใส่เวลา + online ถ้า P4
  return compact({
    ...base,
    validTimeFrom: header.timeFrom,
    validTimeTo: header.timeTo,
    ...onlineIfP4(base.promotionArea, material),
  });
}

function onlineIfP4(promotionArea, material) {
  if (promotionArea !== "P4") return {};
  return {
    onlineDescriptionEnglish: material.onlineEn,
    onlineDescriptionThai: material.onlineTh,
  };
}
```

### กลุ่ม A / B / C ประกอบก้อนสุดท้าย

```js
function buildGroupA(ctx) {
  return {
    lineNumber: ctx.lineNumber,
    bonusBuyNumber: ctx.bonusBuyNumber,
    bonusBuyHeader: mapBonusBuyHeaderByProfile(ctx),
    buy: [],                                    // ← สำคัญ
    get: [mapGetShared(ctx, { includeNormalSellPrice: true })],
    ...EMPTY_BB_CHILDREN,
    stores: ctx.stores,
  };
}

function buildGroupB(ctx) {
  return {
    lineNumber: ctx.lineNumber,
    bonusBuyNumber: ctx.bonusBuyNumber,
    bonusBuyHeader: mapBonusBuyHeaderByProfile(ctx),
    buy: [mapBuyShared(ctx)],
    get: [mapGetShared(ctx, { includeNormalSellPrice: false })],
    ...EMPTY_BB_CHILDREN,
    stores: ctx.stores,
  };
}

function buildGroupC(ctx) {
  const { profile } = ctx;

  // D002 — โครงเหมือนกลุ่ม B (Buy+Get แถวเดียว)
  if (profile === "D002") {
    return buildGroupB(ctx);
  }

  // D003 / F003 — รอบหลัง: แยกตาม mechanic (A→buy, B→get)
  // รอบแรก: stub ไว้ก่อน อย่า block ของร่วม
  if (profile === "D003" || profile === "F003") {
    return buildSplitByMechanic(ctx); // TODO รอบ 2
  }

  throw new Error(`Unhandled group C: ${profile}`);
}
```

---

## 4) Entry — ประกอบทั้งหมด

```js
export function convertLayoutCToAbPayload(basePayload) {
  if (!basePayload || basePayload.LAYOUT !== "C") return basePayload;

  const header = mapHeaderShared(basePayload.HEADER || {});
  const conditions = mapConditionsShared(basePayload.CONDITIONS || [], header);

  const bonusBuys = (basePayload.MATERIALS || []).map((material, idx) => {
    const ctx = buildSharedLineContext(material, header);
    ctx.lineNumber = material.lineNumber ?? idx;
    ctx.bonusBuyNumber = String(material.bonusBuyNo ?? idx + 1);
    return routeByProfile(ctx);
  });

  return {
    LAYOUT: "AB",
    STATUS: basePayload.STATUS,
    HEADER: header,
    BONUSBUYS: bonusBuys,
    CONDITIONS: conditions,
    MATERIALS: [],
  };
}
```

> หมายเหตุ: ภายหลังต้องจัดกลุ่มตาม `noof_bonus_buy` / `noof_promotion` — รอบแรก map ทีละ material เพื่อให้ pipeline วิ่งได้ก่อน

---

## 5) แผนลงมือเป็นรอบ (อย่าทำทีเดียว)

| รอบ | ทำอะไร | Done เมื่อ |
|-----|--------|-----------|
| **0** | normalize C + `mapHeaderShared` + CONDITIONS copy/compact | ได้ `LAYOUT:"AB"` + HEADER วันที่/NOBP |
| **1** | `buildSharedLineContext` + `pickDiscount` + `materialType` | มี ctx ครบต่อแถว ยังไม่แยก buy/get |
| **2** | router กลุ่ม A/B + `mapGetShared` / `mapBuyShared` | P001/P015/D001/F001 ออกโครงถูก |
| **3** | `mapBonusBuyHeaderByProfile` (P010 time/ref) | P010/P011 ต่างตรง header |
| **4** | % normalize เต็ม vs ง่าย | F001 ใช้แบบง่าย |
| **5** | กลุ่ม C: D002 = B; D003/F003 ตาม mechanic | ครบ 9 profile |
| **6** | promotionArea / description prefix / BV formulas | เมื่อ business ส่งสูตร |

**อย่าเริ่มที่รอบ 5** — เพื่อนควรมีรอบ 0–2 ทำงานก่อน แล้วค่อยเพิ่ม if profile

---

## 6) Checklist ส่งเพื่อน (copy ไป ticket ได้)

- [ ] ของร่วม: วันที่, vendor→NOBP, rebate, contractType Z2, field2 MAT/MGP, ส่วนลด P→R→%
- [ ] **ยังไม่** hardcode `MGPSAP` / `char: "40"`
- [ ] router 3 กลุ่มก่อน แล้วค่อย if ราย profile ที่ header
- [ ] กลุ่ม A: `buy: []` + `get.field8`
- [ ] กลุ่ม B: `buy` + `get` ไม่มี `field8`
- [ ] P010: ไม่ใส่ `validTime*`, ใส่ `referenceCode`
- [ ] F001/F003: % แบบง่าย
- [ ] D003/F003: TODO แยก — ไม่ block ของร่วม
- [ ] CONDITIONS: รอบแรก map เท่าที่มีใน C พอ สูตร TBD ใส่ registry ทีหลัง

---

## 7) สิ่งที่ผิดในโค้ดเดิม — ย้ายไปอยู่ตรงไหนในโครงใหม่

| ของเดิม | ปัญหา | ย้ายไปที่ |
|---------|--------|-----------|
| `field2: "MGPSAP"` | ค่าผิด | `materialType()` ใน shared |
| `char: "40"` | ไม่มีใน To-Be | ลบจนกว่า business ยืนยัน |
| `buy` ทุกแถว | กลุ่ม A พัง | `buildGroupA` → `buy: []` |
| `getQuantity` เช็คแค่ B1G1 | ไม่พอ | `lookupBuyGetQty` shared |
| `mapHeader` spread ทั้งก้อน | วันที่/ชื่อ field ไม่แปลง | `mapHeaderShared` |
| if ทุกอย่างใน formula registry | สับสน shared vs profile | registry เฉพาะสูตร TBD; โครง buy/get อยู่ใน router |
