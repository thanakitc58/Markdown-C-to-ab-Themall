# C → AB — Profile P010 (Full)

| | |
|--|--|
| **กลุ่ม** | A — Get-only |
| **Macro** | `P010_Header` + `Gen_P010` |
| **โครง** | `buy: []` · `get: [1]` ต่อ 1 แถว `MATERIALS[*]` |
| **payload หลังบ้านหลัก** | `layout-ab-payload.example.json` |
| **ชื่อฟิลด์ AB อธิบายเพิ่ม** | `payload_AB_Reference.md` |
| **ของร่วม** | `MerC_to_AB_Phase1_Shared.md` · `MerC_to_AB_Header_Shared.md` · `MerC_to_AB_Function_Split.md` |
| **ต่างจาก P001** | ไม่ใส่ `validTime*` · ใส่ `referenceCode` · online เมื่อ P4 |

**อ่านยังไง:** ไฟล์นี้เป็น **สเปกครบทุก key ใน AB payload** สำหรับ P010 — แต่ละแถวบอกว่า map จาก C อย่างไร / ห้ามใส่ / เว้นว่าง / Auto  
ต้นทาง = JSON `LAYOUT: "C"` (ดู `layout-c-payload.example.json`)  
ปลายทาง = JSON `LAYOUT: "AB"`

**กติกา source of truth:** ถ้า key/shape ในไฟล์นี้ขัดกับ `layout-ab-payload.example.json` ให้ **ยึด payload หลังบ้านก่อน** แล้วใช้ไฟล์นี้เป็นคู่มือ logic ราย profile


**สัญลักษณ์คอลัมน์ การทำ**

| สัญลักษณ์ | ความหมาย |
|-----------|----------|
| **map** | แปลงจาก C → AB |
| **copy** | ส่งตรง (อาจมี alias ชื่อฟิลด์) |
| **calc** | คำนวณ / lookup / normalize |
| **omit** | profile นี้ **ไม่ใส่ key** นี้ |
| **auto** | ระบบ AB คำนวณเอง (Auto? = Y ใน To-Be) |
| **empty** | ใส่ `[]` หรือ omit ได้ |
| **ctx** | มาจาก context/header ไม่ใช่แถว material โดยตรง |


---

## 0) Pipeline P010

```
convertLayoutCToAbPayload(c)
  normalizeC(c)
  HEADER  = mapHeaderShared(c.HEADER)
  BONUSBUYS = c.MATERIALS
    .filter(shouldConvertRow)
    .filter(m => shortCode(m.bonusBuyProfile) === "P010")
    .map(m => buildGroupA(ctx))         // get[1] only
  CONDITIONS = mapConditionsShared(c.CONDITIONS)
  MATERIALS  = []
  LAYOUT = "AB"
  STATUS = c.STATUS
```

1 แถว `MATERIALS[i]` (profile P010) → 1 element `BONUSBUYS[i]`

---

## อ่านก่อนแปลง (ภาษาคน — P010)

**P010 อยู่กลุ่ม A (Get-only)**  
User กรอกแถวสินค้าใน C ภายใต้ `MATERIALS`  
ตอนแปลงเป็น AB แถวนั้นกลายเป็น 1 ก้อนใน `BONUSBUYS` ที่มีแค่ฝั่ง **ได้ของ / ได้ส่วนลด (`get`)**  
**ไม่มีฝั่งซื้อ (`buy` ว่าง `[]`)** เพราะโปรไฟล์นี้ไม่บังคับ “ซื้อกี่ชิ้นก่อน” แบบ Buy+Get

### สิ่งที่ต้องจำก่อนเขียนโค้ด

1. **ของที่ส่งตรงเกือบหมด:** `STATUS`, `CONDITIONS`, และหลายฟิลด์ใน `HEADER`  
2. **ของที่ต้องประกอบใหม่:** ทุกแถว `MATERIALS` → 1 (หรือมากกว่า) ก้อน `BONUSBUYS` ด้านล่างนี้  
3. **ส่วนลดฝั่ง get เลือกได้แค่แบบเดียว:** มีราคาจัดรายการ (P) หรือ ส่วนลดบาท (R) หรือ ส่วนลด% — **ห้ามส่งสองแบบพร้อมกัน**

**ราคาขายปกติ (`field8`)**  
อยู่ฝั่ง **`get`** ได้ (กลุ่ม A)  
ตัวอย่าง: C มี `salesPriceNormal: 1199` → AB `get[0].field8 = 1199`

**ส่วนลด %** ใช้แบบเต็ม: ถ้ากรอก `10` ถือว่าเป็น 10% อยู่แล้ว; ถ้ากรอก `1668` แปลว่าคนใส่ทศนิยมผิดสเกล → หาร 100 ได้ `16.68`

**เวลา Valid** — P010 **ไม่ใส่** `validTimeFrom` / `validTimeTo` ใน BONUSBUY

**Reference code** — P010 **ใส่** จาก HEADER หรือ MATERIALS

**Online EN/TH** ใส่เฉพาะเมื่อพื้นที่โปรเป็น `P4` · นอกนั้นไม่ต้องมี key

### ตัวอย่างภาพรวม (สมมติ 1 แถวสินค้า)

```
C.MATERIALS[0] = {
  material: "1000473882",
  mechanic: "Get Discount",
  salesPriceNormal: 1199,
  salesPricePromotion: 999,   // มีราคาจัดรายการ
  discountAmount: 50,         // มีบาทด้วย แต่ไม่ใช้ เพราะมีราคาจัดรายการแล้ว
  salesUnit: "EA"
}
```

แปลงแล้วฝั่ง AB (ย่อ) — กลุ่ม A มีแค่ get:

```
BONUSBUYS[0].buy = []
BONUSBUYS[0].get[0].field8 = 1199          // ราคาปกติอยู่ฝั่ง get
BONUSBUYS[0].get[0].fieldP = 999           // ส่วนลดแบบราคาจัดรายการ
// ไม่มี fieldR เพราะเลือก P แล้ว
BONUSBUYS[0].get[0].unit = "EA"
```

ตารางด้านล่างเป็นรายฟิลด์ — อ่านคอลัมน์ **อ่านยังไง** เป็นภาษาคนว่า C เป็นแบบนี้แล้ว AB ได้อะไร

---
## 1) Skip rules (ก่อนแปลง)

คิดง่ายๆ: **ยังไม่แปลง** จนกว่าแถวจะผ่านประตูนี้  
แถวที่ไม่ผ่าน = ไม่สร้าง `BONUSBUYS` จากแถวนั้น

| เงื่อนไขใน C | ผล | อ่านยังไง (มีตัวอย่าง) |
|--------------|-----|-------------------------|
| `status` = `Reject` | ข้ามแถว | คนกด reject แถวนี้แล้ว · ตัวอย่าง: แถวที่ 2 เป็น Reject → แถว 2 ไม่เข้า AB แต่แถว 1,3 ยังแปลงได้ |
| `convertStatus` = `Done` | ไม่แปลงซ้ำ | แถวนี้แปลงไปรอบก่อนแล้ว · อย่าสร้าง BONUSBUY ซ้ำ |
| `mechanic` = `B1G1 (On Pack)` หรือ `B2G1 (On Pack)` | ข้ามทั้งกลุ่ม | โปรติดแพ็ก ไม่เข้า flow นี้ · ทั้งชุดโปรที่ผูกกันข้ามไป |
| `bonusBuyProfile` ไม่ใช่ `P010` | ไม่เข้าไฟล์นี้ | แถวเป็น P001 แต่คุณเปิดสเปก P010 → ส่งไป router อื่น ไม่ใช่บั๊ก |
| โปรไฟล์นอก 9 ตัวลูกค้า | reject ทั้งก้อน | รองรับแค่ P001/P010/P011/P015/D001/F001/D002/D003/F003 |

---

## 2) ระดับบนสุด

| จาก C (JSON) | → AB (JSON) | การทำ | เงื่อนไข P010 |
|--------------|-------------|-------|----------------|
| — | `LAYOUT` | **calc** | คงที่ `"AB"` |
| `STATUS` | `STATUS` | **copy** | ส่งตรง |
| `HEADER` | `HEADER` | **map** | §3 |
| `MATERIALS[*]` | `BONUSBUYS[*]` | **calc** | สร้างใหม่ §4–§7 · buildGroupA — get only |
| `CONDITIONS` | `CONDITIONS` | **copy** | §8 |
| `BONUSBUYS` (C) | — | **omit** | C มักเป็น `[]` — ไม่ใช้ |
| `MATERIALS` (AB) | `MATERIALS` | **empty** | `[]` |
| — | `SUPPLIERFILE` | **empty** | `null` ได้ถ้าต้องคง shape ตาม sample |

---


## 3) HEADER (`MerAB - Header`)

HEADER คือหัวโปรทั้งใบ — **ใช้ util ร่วม** (`mapHeaderShared`) ไม่ต้องแยก if P010  
ด้านล่างคือ “userอกอะไร → ส่งอะไร / ต้องแก้ไหม”

| จาก C | → AB | การทำ | อ่านยังไง |
|-------|------|-------|-----------|
| `group` | `group` | **copy** | ส่งชื่อกลุ่มงานตามที่กรอก |
| `promotionName` | `promotionName` | **copy** | ส่งชื่อโปรตามที่กรอก |
| `purchasingGroup` | `purchasingGroup` | **copy** / lookup | ส่งรหัส/ชื่อตามที่กรอก · ถ้ามีตาราง lookup จะขยายเป็นชื่อเต็มได้ |
| `theme` | `theme` | **copy** / lookup | เหมือน purchasingGroup |
| `bonusBuyProfile` | `bonusBuyProfile` | **calc** | อยากได้รหัสสั้น · ตัวอย่าง: `"D001 - Same group"` → `"P010"` |
| `rebateChargeback` | `rebateChargeback` | **calc** | ตัดตัวอักษรแรกของค่าที่กรอกทิ้ง · ถ้าผลลัพธ์ไม่ขึ้นต้น `Z2` และตัวแรกไม่ใช่ `A` ให้เติม `0` นำหน้า · ตัวอย่าง: ต้นทาง `"X0ABC"` → ตัด `X` เหลือ `"0ABC"` · ถ้าว่างก็ว่าง |
| `contractType` | `contractType` | **calc** | ใส่เมื่อค่าขึ้นต้นด้วย `"Z2"` (เช่น `"Z202"`) · ถ้าไม่ใช่สัญญาแบบนี้ → ส่งว่าง `""` |
| `wbsNumber` / `wbsNo` | `wbsNumber` | **copy** | ส่งเลข WBS · ชื่อฟิลด์ C อาจเป็น `wbsNo` ก็ได้ |
| `vendorCode` | `vendorCode` | **calc** | **ถ้าว่าง ให้ใส่ `NOBP`** · ตัวอย่าง: `""` → `"NOBP"` · `"OIS05"` → `"OIS05"` |
| `vendorName` | `vendorName` | **copy** | ส่งชื่อ vendor ตามที่กรอก |
| `periodFrom` / `periodTo` | เหมือนกัน | **copy** | วันที่รูปแบบ `YYYY-MM-DD` (ตาม payload หลังบ้าน) |
| `days` | `days` | **copy** | เช่น `["All"]` = จัดทุกวัน |
| `singleMultiple` | `singleMultiple` | **copy** | `"Single"` หรือ `"Multiple"` ตามที่เลือก |
| `volume` / `vol` | `volume` | **copy** | ส่งตรง · ชื่อสั้น `vol` ก็ map เป็น `volume` |
| `status` | — | **omit** | ฝั่ง AB HEADER ไม่รับฟิลด์นี้ → ไม่ส่ง |
| `timeFrom` / `timeTo` | เก็บใน HEADER | **copy** | เก็บไว้ที่หัวใบ และเอาไปใส่เวลา Valid ใน BONUSBUY (§5) ถ้าโปรไฟล์นั้นใช้ |

รายละเอียด util เพิ่ม: `MerC_to_AB_Header_Shared.md`

---

## 4) `BONUSBUYS[*]` — โครงก้อน

| AB block | P010 |
|----------|------|
| `lineNumber` | **copy** จาก `MATERIALS[*].lineNumber` (optional) |
| `bonusBuyNumber` | **copy** ซ้ำระดับ root (optional) |
| `bonusBuyHeader` | §5 |
| `buy[]` | `[]` เสมอ |
| `get[]` | ความยาว `[1]` — §7 |
| `stores[]` | §7.1 |
| `card[]` | **empty** |
| `tender[]` | **empty** |
| `installment[]` | **empty** |
| `posTerminal[]` | **empty** |
| `premium[]` | **empty** |
| `coupon[]` | **empty** |
| `limitControl[]` | **empty** |

---

## 5) `bonusBuyHeader` — หัวของแต่ละก้อน BONUSBUY

นี่คือหัวของ **1 โปรย่อย** หลังแตกจากแถว MATERIALS  
(คนละชั้นกับ HEADER ทั้งใบ)

| จาก C | → AB | ชื่อบนจอ | req | การทำ | อ่านยังไง |
|-------|------|----------|-----|-------|-----------|
| `numberOfBonusBuy` | `bonusBuyNumber` | Bonus buy No. | M | **copy** | เลขลำดับก้อน · ไม่กรอก → ใส่ `"1"` |
| `bonusBuyProfile` | `bonusBuyProfile` | Bonus Buy Profile | — | **calc** | ตัดให้เหลือรหัสสั้น `"P010"` เท่านั้น |
| `mechanic` | `mechanic` | Mechanic | — | **copy** | ส่งข้อความ mechanic ตามที่เลือกในฟอร์ม · ข้อความนี้เอาไปเปิดตาราง Mechanic เพื่อรู้ว่าซื้อกี่ชิ้น / ได้กี่ชิ้น (กลุ่ม A ใช้แค่จำนวนฝั่งได้) |
| `HEADER.timeFrom` | `validTimeFrom` | Valid Time from | — | **omit** | **P010 ไม่ส่งฟิลด์นี้** · อย่าใส่ `validTimeFrom` ใน JSON ผลลัพธ์ |
| `HEADER.timeTo` | `validTimeTo` | Valid Time to | — | **omit** | **P010 ไม่ส่งฟิลด์นี้** · อย่าใส่ `validTimeTo` |
| `HEADER.wbsNumber` | `wbsNumber` | WBS No. | — | **ctx** | ยืมเลข WBS จากหัวใบมาใส่ทุกก้อน |
| `promotionArea` (HEADER ก่อน แล้วค่อย MATERIALS) | `promotionArea` | Promotion Area | — | **copy** | บอกพื้นที่โปร · ค่า `P4` = ออนไลน์ → ถึงจะใส่คำอธิบาย online |
| online EN | `onlineDescriptionEnglish` | Online EN | — | **copy** | ใส่เมื่อโปรเป็นออนไลน์พื้นที่ **P4** เท่านั้น · ตัวอย่าง: `promotionArea="P4"` และมีข้อความ EN → ส่ง · ถ้าพื้นที่เป็นอย่างอื่น → **ไม่มี key นี้เลย** |
| online TH | `onlineDescriptionThai` | Online TH | — | **copy** | เหมือน EN แต่เป็นข้อความไทย · นอก P4 ไม่ส่ง |
| `referenceCode` (HEADER หรือ MATERIALS) | `referenceCode` | Reference | — | **copy** | **P010 ใส่ได้** · มีก็ส่ง ไม่มีก็ `""` |
| — | `promotionNumber` / `description` / `purchasingGroup` / `product` / `department` | (หลายช่อง) | — | **auto** | **ไม่ต้อง map** · ระบบ AB ใส่เอง |
| — | `limitNumber`, `priceTag`, flags ต่างๆ | — | — | **omit** | รอบนี้ไม่ใช้ → **อย่าใส่ key** ใน JSON |

---

## 6) `buy[]` — ฝั่งซื้อ

**P010 ไม่มีฝั่งซื้อ**  
ตั้งค่าเป็น array ว่างเสมอ:

```json
"buy": []
```

อย่าเอา material / ราคา ไปยัดใน buy

---

> **กลุ่ม A:** มีแค่ฝั่ง get · สร้าง `get: [ {...} ]` ความยาว 1 ต่อแถว MATERIALS

## 7) `get[0]` — ฝั่งได้ (ได้อะไร / กี่ชิ้น / ส่วนลดแบบไหน)

ฝั่งนี้บอกว่าเมื่อเข้าเงื่อนไขแล้ว **ได้อะไร** และ **ส่วนลดเป็นแบบไหน**

### กติาส่วนลด (สำคัญมาก)

ดูใน MATERIALS ตามลำดับนี้ — **เจออันแรกแล้วหยุด**:

1. มี `salesPricePromotion` (ราคาจัดรายการ) → ใส่แค่ `fieldP`  
2. ไม่มีข้อ 1 แต่มี `discountAmount` (ส่วนลดบาท) → ใส่แค่ `fieldR`  
3. ไม่มีข้อ 1–2 แต่มี % → ใส่แค่ `field22` หลังปรับสเกลแล้ว  
4. ไม่มีเลย → ไม่ใส่ทั้งสามช่อง

**ผิด:** ส่ง `fieldP` กับ `fieldR` พร้อมกัน  
**ถูก:** มีราคาจัดรายการ 999 และมีส่วนลดบาท 50 ในฟอร์ม → AB ได้แค่ `fieldP: 999`

| จาก C | → AB `get[0]` | ชื่อบนจอ | การทำ | อ่านยังไง |
|-------|---------------|----------|-------|-----------|
| `numberOfBonusBuy` | `bonusBuyNumber` | Bonus Buy No. | **copy** | เลขเดียวกับหัวก้อน |
| มีกลุ่มไหม? | `field2` | ประเภท | **calc** | เหมือนฝั่ง buy: `"Material"` หรือ `"Material Group"` |
| `material` / ชื่อกลุ่ม | `field4` | รหัส / ชื่อกลุ่ม | **calc** | รหัสสินค้า หรือชื่อกลุ่มตามประเภท |
| คำอธิบายสินค้า | `field5` / `sapMasterDescription` | Description | **copy** | มีก็ส่ง |
| `costNormal` | `field7` | ราคาทุน | **copy** | มีก็ส่ง |
| `salesPriceNormal` | `field8` | ราคาขายปกติ | **copy** | **ใส่ได้** · C `salesPriceNormal: 1199` → `get[0].field8 = 1199` |
| `mechanic` | `getQuantity` | Get Qty | **calc** | เปิดตาราง Mechanic เอา **get qty** · ตัวอย่างได้ 1 ชิ้น → `getQuantity = 1` |
| `costPromotion` | `field17` | ราคาทุนจัดรายการ | **copy** | มีก็ส่ง · ไม่เกี่ยวกับการเลือก P/R/% |
| `salesPricePromotion` | `fieldP` | ราคาจัดรายการ [P] | **calc** | ลำดับ 1 ของส่วนลด · ตัวอย่าง `999` → `fieldP: 999` แล้ว **ไม่ส่ง** R/% |
| `discountAmount` | `fieldR` | ส่วนลดบาท [R] | **calc** | ลำดับ 2 · ใช้เมื่อ **ไม่มี** ราคาจัดรายการ · ตัวอย่าง `50` → `fieldR: 50` |
| ช่อง % (`discountPercentPlu` / `discountPercentForP015` / `discountPct`) | `field22` | ส่วนลด % | **calc** | ลำดับ 3 · ใช้เมื่อไม่มี P และ R · ตัวอย่าง: กรอก `1668` → ส่ง `16.68` · กรอก `10` → ส่ง `10` |
| `barcode` | `ean` | EAN | **copy** | มีก็ส่ง |
| `salesUnit` | `unit` | Unit | **copy** | หน่วยขาย · ชื่อ key ฝั่ง get เป็น `unit` (ไม่ใช่ `salesUnit`) เช่น `"EA"` |
| vat / GP / tier / point / Pro Tag | — | — | **omit** หรือ **auto** | ไม่ map จาก C รอบนี้ · auto = ระบบคิดเอง |

### 7.1 `stores[]` — สาขาที่ร่วมรายการ

| จาก C | → AB | อ่านยังไง |
|-------|------|-----------|
| `MATERIALS[*].stores` | `stores` | แปลงรหัสสาขาให้อ่านง่าย · ตัวอย่าง `store_13_ka` → `"13KA"` · ได้ array เช่น `["13KA","14KB"]` |
| ไม่มีสาขา | `stores` | ใส่ `[]` |

---

## 8) `CONDITIONS[]`

ใช้เมื่อ `contractType` ขึ้นต้น `Z2` / `Z3`  
รอบแรก: **copy + compact** จาก C — ไม่ gen ใหม่จาก MATERIALS

| AB block | การทำ |
|----------|-------|
| `conditionHeader` | **copy** |
| `businessVolumePurchase[]` | **copy** |
| `businessVolumeSales[]` | **copy** |
| `conditionType[]` | **copy** |
| `settlementCalendar[]` | **copy** |
| `combineCheck[]` | **copy** |
| `allocation[]` | **copy** |

Compensate / Settlement / Payment ใน Mer C → อยู่ใน CONDITIONS ไม่ใช่ BBY line

---


## 9) `MATERIALS[]` ฝั่ง AB

| จาก C | → AB |
|--------|------|
| `MATERIALS[*]` (forecast, planogram, GP, …) | `MATERIALS: []` |

ข้อมูล planogram / forecast ใน C **ไม่เข้า BBY** ในรอบนี้

---

## 10) สูตรและ helper (P010)

### 10.1 กฎเฉพาะ profile (สรุป)

| หัวข้อ | ค่า P010 |
|--------|----------|
| เขียนฝั่ง | `get[1]` เท่านั้น |
| `get.field8` | **ใส่** |
| Normalize % | **เต็ม** |
| `validTimeFrom` / `validTimeTo` | **omit** |
| `referenceCode` | **ใส่** |
| Online EN/TH | **ใส่เมื่อ** `promotionArea = "P4"` |

### 10.2 `materialType` + `materialOrGroup`

```
ถ้ามี materialGroupName หรือ numberOfMaterialGrouping ชี้กลุ่ม:
  field2 = "Material Group"
  field4 = materialGroupName
ไม่งั้น:
  field2 = "Material"
  field4 = material
```

### 10.3 เลือกส่วนลด (`pickDiscount`) — อ่านเป็นขั้นตอน

อย่าคิดว่าต้องส่งครบทุกช่องส่วนลด  
**ส่งช่องเดียว** ตามลำดับ:

```
ถ้ามีราคาจัดรายการ (salesPricePromotion)
    → ใส่ get.fieldP = ราคานั้น
    → ไม่ใส่ fieldR และ field22

ไม่งั้น ถ้ามีส่วนลดบาท (discountAmount)
    → ใส่ get.fieldR = จำนวนบาท
    → ไม่ใส่ fieldP และ field22

ไม่งั้น ถ้ามี %
    → ปรับสเกล % ก่อน แล้วใส่ get.field22
    → ไม่ใส่ fieldP และ fieldR

ไม่งั้น
    → ไม่มีส่วนลดใน get
```

ตัวอย่างจากชีวิตจริง:
- กรอกโปรราคา 999 และกรอกบาท 50 พร้อมกันในฟอร์ม → AB เหลือแค่ `fieldP: 999`
- กรอกแค่บาท 50 → AB ได้ `fieldR: 50`
- กรอกแค่ % → AB ได้ `field22` หลังปรับสเกล

### 10.4 ปรับสเกล % แบบเต็ม (`normalizePctFull`) — ใช้ใน P010

บางคนกรอก `10` (หมาย 10%) บางคนกรอก `1668` (พลาดสเกล)

```
ถ้าคิดว่าเป็น % อยู่แล้ว (เช่น 10)     → คงไว้ 10
ถ้าใหญ่ผิดปกติ (>100 เช่น 1668)     → หาร 100 ได้ 16.68
ถ้าเป็นทศนิยมเล็ก (<1 เช่น 0.15)     → คูณ 100 ได้ 15
```

ตรวจกับ fixture: `7-D001-pct-normalize` ใช้แนว `1668 → 16.68`

### 10.5 จำนวนชิ้นจาก Mechanic (`lookupBuyGetQty`)

User เลือกข้อความ mechanic ในฟอร์ม เช่น `"1A Get 1B (A)"`  
ระบบไปเปิด **ตาราง Mechanic** หาแถวชื่อตรงกัน แล้วอ่าน:
- **buy qty** → ใส่ `buy[0].field9` (จำนวนชิ้นขั้นต่ำฝั่งซื้อ)
- **get qty** → ใส่ `get[0].getQuantity` (จำนวนที่ได้)

ถ้าตารางยังไม่มีแถวนั้น → ใช้ `1` กับ `1` ชั่วคราว (fixture หลายไฟล์ทำแบบนี้)

หมายเหตุ: mechanic แบบ On Pack ถูกข้ามตั้งแต่ Skip rules แล้ว จะไม่มาถึงขั้นนี้

### 10.6 Alias ชื่อฟิลด์ C (normalizeC)

| camelCase | snake_case ที่พบใน fixture |
|-----------|----------------------------|
| `salesPriceNormal` | `sales_price_normal` |
| `salesPricePromotion` | `sales_price_promo` |
| `bonusBuyProfile` | `bonus_buy_profile` |
| `numberOfBonusBuy` | `noof_bonus_buy` |
| `materialDescription` | `material_des`, `material_th_des` |
| `materialDescription2` | `material_des_2`, `material_en_des` |
| `discountAmount` | `discountAmt` |

---

## 11) เคสทดสอบ

| # | จุดตรวจ |
|---|--------|
| 1 | ส่วนลด promo → `get.fieldP` อย่างเดียว |
| 2 | ส่วนลดบาท → `get.fieldR` อย่างเดียว |
| 3 | ส่วนลด % → `get.field22` หลัง normalize |
| 4 | `HEADER.vendorCode` ว่าง → `NOBP` |
| 5 | Material Group → `field2` / `field4` |

---

## 12) Skeleton JSON (`BONUSBUYS[]` element)

```json
{
  "lineNumber": 0,
  "bonusBuyNumber": "1",
  "bonusBuyHeader": {
    "bonusBuyNumber": "1",
    "bonusBuyProfile": "P010",
    "mechanic": "Discount",
    "promotionArea": "All",
    "wbsNumber": "WBS-2026-002"
    "referenceCode": "REF-001",
    "onlineDescriptionEnglish": "…",
    "onlineDescriptionThai": "…",
  },
  "buy": [],
  "get": [
    {
      "bonusBuyNumber": "1",
      "field2": "Material",
      "field4": "MAT-100",
      "field5": "Get item description",
      "field8": "129.00",
      "getQuantity": "1",
      "field22": "10",
      "unit": "EA"
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

## 13) Checklist (P010 ครบ)

### โครง
- [ ] `LAYOUT` = `"AB"`
- [ ] 1 แถว C → 1 `BONUSBUYS` element
- [ ] `buy = []` · มี `get[0]`
- [ ] `MATERIALS` ฝั่ง AB = `[]`

### HEADER
- [ ] วันที่ใน `HEADER.periodFrom/periodTo` เป็น `YYYY-MM-DD`
- [ ] `vendorCode` ว่าง → `NOBP`
- [ ] มี `timeFrom`/`timeTo` ใน AB `HEADER`

### bonusBuyHeader / get
- [ ] `buy = []`
- [ ] `get.field8` มีค่า
- [ ] **ไม่มี** `validTime*`
- [ ] มี `referenceCode`
- [ ] online เฉพาะ P4
- [ ] % normalize เต็ม
- [ ] ส่วนลด **ช่องเดียว**: P หรือ R หรือ field22
- [ ] Get Mandatory: `bonusBuyNumber`, `field2`, `field4`, `getQuantity`, `unit`

### อื่นๆ
- [ ] `CONDITIONS` copy จาก C
- [ ] `card/tender/…` = `[]`
- [ ] skip rules §1 ทำงาน

---


## 14) สิ่งที่ยัง TBD ในโค้ด (ไม่ใช่ลืม map)

| หัวข้อ | หมายเหตุ |
|--------|----------|
| `lookupBuyGetQty` | fixture มัก hardcode `1` |
| `normalizeRebate` เต็ม | บางเคส clear เป็น `""` |
| `bonusBuyHeader.description` | macro prefix |
| theme / purchasingGroup lookup | optional |
| Pro Tag, Tier, Point | out of scope รอบนี้ |

---

## 15) การยืนยัน key (audit)

เทียบ: `payload_AB_Reference.md` · `Gen_P010` / `P010_Header` ใน `Mer-C_Convert_To_STD.txt` · `layout-ab-payload.example.json`

### ✅ Core P010

| หัวข้อ | สถานะ |
|--------|--------|
| Get-only (`buy=[]`) | ✅ ตาม macro กลุ่ม A |
| `get.field8` ใส่ | ✅ ตามกลุ่ม A |
| ส่วนลดช่องเดียว + normalize ตามตาราง | ✅ |
| validTime / referenceCode / online ตามตาราง §10.1 | ✅ ตาม macro header |
| AB key name ตาม payload | ✅ |
| Golden test ครบทุก key | ❌ ยังไม่มี |
