# C → AB — HEADER Shared Utils

สำหรับเพื่อนที่อยากทำ **util เรียกครั้งเดียว** ครอบทุกโปรไฟล์ (9 ตัว)

| | |
|--|--|
| **ขอบเขต** | เฉพาะ `HEADER` ฝั่ง AB |
| **ใช้ร่วม** | P001, P010, P011, P015, D001, F001, D002, D003, F003 |
| **อย่าใส่ if รายโปรไฟล์ที่นี่** | จุดต่างโปรไฟล์ไปที่ `bonusBuyHeader` / `buy` / `get` |
| **อ้างอิง** | `MerC_to_AB_Function_Split.md` · `MerC_to_AB_Phase1_Shared.md` · `payload_AB_Reference.md` |

---

## 1) หลักการ

- ระดับ `HEADER` ของ AB = **ของร่วมเกือบทั้งก้อน**
- เรียก `mapHeaderShared(header)` ครั้งเดียวทุกโปรไฟล์
- ใช้ `compact(...)` — ไม่มีค่า / ไม่ใช้ → **ไม่ใส่ key** (อย่าใส่ `""` รก)

จุดที่ต่างรายโปรไฟล์ (เช่น `validTime*`, `referenceCode`, online EN/TH) อยู่ที่ **`BONUSBUYS[].bonusBuyHeader`** ไม่ใช่ HEADER

---

## 2) Util ย่อยที่ควรมี

| Util | Input | Output / กติกา |
|------|-------|----------------|
| `isoToSapDate` | `YYYY-MM-DD` (หรือเทียบเท่า) | `DD.MM.YYYY` |
| `vendorOrNobp` | `vendorCode` | ว่าง / null → `"NOBP"` · ไม่งั้นค่าเดิม |
| `normalizeRebate` | `rebateChargeback` (+ `contractType` ถ้าต้องใช้ประกอบ) | ตัดตัวอักษรแรก; ถ้าไม่ใช่ Z2 และตัวแรก ≠ `A` → เติม `0` นำหน้า |
| `normalizeContractType` | `contractType` | ขึ้นต้น `Z2` ถึงใส่ · ไม่งั้นว่าง / omit |
| `shortCode` / `profileCode` | `bonusBuyProfile` | ตัดเหลือรหัส เช่น `D001` |
| `capitalizeStatus` | `status` | `draft` → `Draft` |
| `normalizeDays` | `days[]` | มี All → `["All"]` · ไม่มีเก็บรายวัน (Phase 1 ส่งตามที่ user เลือกได้) |

Lookup (รอบหลัง / เมื่อมีตาราง):

| Util | กติกา |
|------|-------|
| `lookupTheme` | รหัส 4 ตัว → ชื่อเต็ม (ถ้า UI ส่งชื่อเต็มแล้ว = ส่งตรง) |
| `lookupPurchasingGroup` | รหัส → ชื่อเต็ม |

---

## 3) ตาราง HEADER ที่ map ร่วมทุกโปรไฟล์

ต้นทาง = path ใน JSON `LAYOUT: "C"`

| จาก C (JSON) | → AB `HEADER` | วิธี | Util |
|--------------|---------------|------|------|
| `HEADER.theme` | `theme` | ส่งตรง / lookup | copy · `lookupTheme` |
| `HEADER.promotionName` | `promotionName` | ส่งตรง | copy |
| `HEADER.group` | `group` | ส่งตรง | copy |
| `HEADER.purchasingGroup` | `purchasingGroup` | ส่งตรง / lookup | copy · lookup |
| `HEADER.bonusBuyProfile` / `MATERIALS[*].bonusBuyProfile` | `bonusBuyProfile` | ตัดรหัสโปรไฟล์ | `shortCode` |
| `HEADER.rebateChargeback` | `rebateChargeback` | สูตรตัด / เติม 0 | `normalizeRebate` |
| `HEADER.contractType` | `contractType` | เฉพาะ `Z2…` | `normalizeContractType` |
| `HEADER.wbsNumber` | `wbsNumber` | ส่งตรง (สำรอง `HEADER.wbsNo`) | copy |
| `HEADER.vendorCode` | `vendorCode` | ว่าง → `NOBP` | `vendorOrNobp` |
| `HEADER.vendorName` | `vendorName` | ส่งตรง | copy |
| `HEADER.periodFrom` | `periodFrom` | แปลงวันที่ | `isoToSapDate` |
| `HEADER.periodTo` | `periodTo` | แปลงวันที่ | `isoToSapDate` |
| `HEADER.days` | `days` | ส่งผ่าน / normalize | `normalizeDays` |
| `HEADER.singleMultiple` | `singleMultiple` | ส่งตรง | copy |
| `HEADER.volume` | `volume` | ส่งตรง (สำรอง `HEADER.vol`) | copy |
| `HEADER.status` | `status` | ตัวพิมพ์ต้น | `capitalizeStatus` |

### ไม่ใช่ปลายทาง HEADER ของ AB

| จาก C (JSON) | หมายเหตุ |
|--------------|----------|
| `HEADER.timeFrom` / `HEADER.timeTo` | พักค่าไว้ใน context แล้วไปใส่ `BONUSBUYS[*].bonusBuyHeader.validTimeFrom` / `validTimeTo` **ตามโปรไฟล์** — อย่าคงไว้ใน AB `HEADER` |

---

## 4) โครงโค้ดแนะนำ

```js
function mapHeaderShared(header) {
  return compact({
    theme: header.theme,                          // หรือ lookupTheme(header.theme)
    promotionName: header.promotionName,
    group: header.group,
    purchasingGroup: header.purchasingGroup,      // หรือ lookup
    bonusBuyProfile: shortCode(header.bonusBuyProfile),
    rebateChargeback: normalizeRebate(header.rebateChargeback, header.contractType),
    contractType: normalizeContractType(header.contractType),
    wbsNumber: header.wbsNumber ?? header.wbsNo,
    vendorCode: vendorOrNobp(header.vendorCode),
    vendorName: header.vendorName,
    periodFrom: isoToSapDate(header.periodFrom),
    periodTo: isoToSapDate(header.periodTo),
    days: normalizeDays(header.days),
    singleMultiple: header.singleMultiple,
    volume: header.volume ?? header.vol,
    status: capitalizeStatus(header.status),
  });
}
```

เรียกใช้:

```js
const ab = {
  LAYOUT: "AB",
  STATUS: payload.STATUS,
  HEADER: mapHeaderShared(payload.HEADER),
  BONUSBUYS: materialsToBonusBuys(...),  // แยกรายโปรไฟล์ตรงนี้
  CONDITIONS: mapConditionsShared(payload.CONDITIONS),
  MATERIALS: [],
};
```

**ห้าม**

```js
if (profile === "D001") { header.xxx = ... }  // อย่าทำใน HEADER util
```

---

## 5) ตัวอย่าง / fixture ที่เกี่ยวกับ HEADER

| เคส | ไฟล์ | จุดตรวจ HEADER |
|-----|------|----------------|
| วันที่แปลงแล้ว | `tungconvert/D001/1-D001-promo.json` | `periodFrom`/`periodTo` = `DD.MM.YYYY` |
| vendor ว่าง | `tungconvert/D001/4-D001-nobp.json` | `vendorCode` = `"NOBP"` |
| status | ทุกเคส D001 | `Draft` |

---

## 6) Checklist ก่อนถือว่า HEADER util เสร็จ

- [ ] `isoToSapDate` แปลงถูก
- [ ] `vendorOrNobp` ว่างได้ `NOBP`
- [ ] `normalizeContractType` เก็บเฉพาะ `Z2…`
- [ ] `normalizeRebate` ตามสูตร shared
- [ ] `shortCode` ได้รหัสโปรไฟล์สั้น
- [ ] ไม่เหลือ `timeFrom`/`timeTo` ใน AB `HEADER`
- [ ] ไม่มี `if (profile)` ใน `mapHeaderShared`
- [ ] ใช้ `compact` — ไม่ยัด `""` ทุกช่อง

---

## 7) ของที่ทำคนละที่ (ไม่ใช่ HEADER util)

| หัวข้อ | อยู่ที่ |
|--------|--------|
| `validTimeFrom` / `validTimeTo` | `bonusBuyHeader` (บางโปรไฟล์ใส่ / บางตัวไม่ใส่) |
| `referenceCode` | `bonusBuyHeader` |
| online EN/TH เมื่อ P4 | `bonusBuyHeader` |
| `buy[]` / `get[]` / ส่วนลด / MAT·MGP / mechanic qty | ระดับแถว BBY |

รายละเอียดแยกโปรไฟล์: `MerC_to_AB_ByProfile.md` + ไฟล์ใน `profile/`
