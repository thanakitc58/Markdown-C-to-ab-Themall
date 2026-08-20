# AB Payload — Reference ฝั่ง AB (LAYOUT = "AB")

ไฟล์นี้ล็อก **payload ฝั่ง AB** โดยยึด **payload หลังบ้านตัวอย่าง** เป็นหลัก

**Source of truth ปัจจุบัน:** `layout-ab-payload.example.json`  
**เอกสารนี้มีหน้าที่:** อธิบายชื่อ key / section / ความหมายเชิง To-Be ให้คนทำ convert อ่านง่ายขึ้น

**อ่านยังไง (สำคัญ):** เพื่อนที่ทำต่อ **ทำงานที่ชื่อ field ใน JSON ล้วนๆ ไม่ต้องแตะ Excel cell**
- ถ้าเอกสารนี้ขัดกับ `layout-ab-payload.example.json` ให้ **ยึด payload หลังบ้านก่อน**
- ทุก key ใน payload → ชี้ไปที่ **ชื่อฟิลด์จริงในไฟล์ To-Be** `Copy of The To-Be Promotion Template Structure.xlsx`
  - `HEADER` → ชีต **`MerAB - Header`**
  - `BONUSBUYS` / `CONDITIONS` → ชีต **`MerAB - item Details`** (คอลัมน์ `B/C/D` = ชื่อฟิลด์, `G` = required, `H` = Auto)
- ลำดับ key ใน payload **เรียงตรงกับลำดับแถวในชีต To-Be แบบ 1:1** (ไล่ทีละแถวได้เลย)
- Excel cell (BM/BW/BF…) = ใช้เฉพาะตอน **export กลับเป็นไฟล์ AB** → ย้ายไปไว้ **ภาคผนวก A** ท้ายไฟล์

**Scope ลูกค้า: 9 profile เท่านั้น**

| กลุ่ม | Profile |
|------|---------|
| A — Get-only | P001, P010, P011, P015 |
| B — Buy+Get แถวเดียว | D001, F001 |
| C — แยกแถว / หลาย index | D002, D003, F003 |

**Out of scope:** P100, P103, F002 (มีใน macro แต่ลูกค้าไม่ใช้)

อ้างอิง logic แปลง: `MerC_to_AB_Phase1_Shared.md`, `MerC_to_AB_ByProfile.md` (แยก 9 profile), `Mer-C_Convert_To_STD.txt` (`Gen_Pxxx` บรรทัด 3620–4600)

---

## 0. โครงสร้างระดับบนสุด

```
{
  "LAYOUT": "AB",
  "STATUS": "DRAFT",
  "HEADER": { ... },          // ระดับโปรโมชัน → ชีต MerAB - Header
  "BONUSBUYS": [ ... ],       // 1 element = 1 Bonus Buy → บล็อก Store/BBY Header/Buy/Get ในชีต item Details
  "CONDITIONS": [ ... ],      // สัญญา/rebate/chargeback → บล็อก Condition ในชีต item Details
  "MATERIALS": [ ... ]        // ระดับสินค้า (planogram/forecast) — เว้นว่างได้
}
```

---

## 1. HEADER — dictionary (ชีต `MerAB - Header`)

| payload key | ชื่อใน To-Be (`MerAB - Header`) | แถว | Input | Auto | หมายเหตุ convert |
|-------------|--------------------------------|-----|-------|------|------------------|
| `group` | กลุ่มงาน / Group | r5 | Dropdown | Y | MerA/MerB/MerC |
| `promotionName` | Promotion Name | r6 | Text | N | ส่งตรง |
| `purchasingGroup` | Purchasing Group | r7 | Dropdown | N | lookup ชื่อเต็ม |
| `theme` | Theme | r8 | Dropdown | N | lookup รหัส 4 ตัว → ชื่อเต็ม |
| `bonusBuyProfile` | Bonus Buy Profile | r9 | Dropdown | N | **routing key** (9 ค่า) |
| `rebateChargeback` | Rebate Chargeback | r10 | Dropdown | N | ตัดตัวอักษรแรก; ไม่ใช่ Z2 และตัวแรก≠`A` → เติม `0` |
| `contractType` | Contract Type | r11 | Dropdown | Y | ใส่เมื่อขึ้นต้น `Z2`; ไม่ใช่ → (none) |
| `wbsNumber` | WBS No. | r12 | Text | Y | default ตาม Group |
| `periodFrom` | จัดรายการระหว่างวันที่ … Start | r13 | Text | N | `DD.MM.YYYY` (ค.ศ.) |
| `periodTo` | จัดรายการระหว่างวันที่ … End | r14 | Text | N | เหมือน periodFrom |
| `vendorCode` / `vendorName` | Vendor | r15 | Text | Y | ว่าง → `NOBP` |
| `days[]` | วันจัดรายการ | r16 | Dropdown | N | BBY เป็นตัวบอกว่าต้องเลือกวันไหม |
| `singleMultiple` | Single / Multiple Promotion | r17 | Dropdown | N | — |
| `timeFrom` / `timeTo` | *(ไม่มีใน To-Be Header แต่มีใน backend payload sample)* | — | — | — | backend sample เก็บไว้ทั้งใน `HEADER` และใช้ไปสร้าง `bonusBuyHeader.validTimeFrom/To` ตาม profile |
| `volume` / `status` | *(ไม่มีใน To-Be)* | — | — | — | ระดับ UI/เอกสาร |

---

## 2. BONUSBUYS[] — dictionary (ชีต `MerAB - item Details`)

1 element = 1 Bonus Buy line ประกอบด้วย `bonusBuyNumber` + `bonusBuyHeader` + `buy[]` + `get[]` (+ บล็อกเสริม `card/tender/…/limitControl` + `stores[]`)

### 2.1 `bonusBuyHeader` (Main Header = "Bonus Buy Header", r31–r54)

| payload key | ชื่อใน To-Be | แถว | required | Auto |
|-------------|--------------|-----|----------|------|
| `bonusBuyNumber` | Bonus buy No. (root ของ element) | — | Optional | — |
| `promotionNumber` | Promotion No | r31 | Optional | Y |
| `bonusBuyNumber` | Bonus buy No. | r32 | Mandatory | N |
| `description` | Description | r33 | Depends on Formula | Y |
| `purchasingGroup` | Purchasing Group | r34 | Optional | Y |
| `bonusBuyProfile` | Bonus Buy Profile | r35 | Optional | Y |
| `mechanic` | Mechanic | r36 | Optional | N |
| `limitNumber` | Limit No. | r37 | Optional | N |
| `validTimeFrom` | Valid Time from | r38 | Optional | N |
| `validTimeTo` | Valid Time to | r39 | Optional | N |
| `product` | Product | r40 | Depends on Formula | Y |
| `priceTag` | Price Tag | r41 | Optional | N |
| `promotionArea` | Promotion Area | r42 | Depends on Formula | Y |
| `wbsNumber` | WBS No. | r43 | Depends on Formula | Y |
| `referenceBonusBuy` | Ref. Bonus buy | r44 | Optional | N |
| `legacyPromotionNumber` | Legacy Promotion No. | r45 | Optional | N |
| `allowDiscountAfterGetExtraMPoint` | Allow discount after get extra M point | r46 | Optional | N |
| `notAcceptAnyDiscountCoupon` | Not accept any discount coupon | r47 | Optional | N |
| `notAcceptAnyDiscountCard` | Not accept any discount card | r48 | Optional | N |
| `notAllowForEmployee` | Not allow for employee | r49 | Optional | N |
| `referenceCode` | Reference code | r50 | Optional | N |
| `department` | Department | r51 | Depends on Formula | Y |
| `onlineDescriptionEnglish` | Online Description EN | r52 | Optional | N |
| `onlineDescriptionThai` | Online Description TH | r53 | Optional | N |

### 2.2 `buy[]` (Main Header = "Buy", r54–r69)

| payload key | ชื่อใน To-Be | แถว | required |
|-------------|--------------|-----|----------|
| `bonusBuyNumber` | Bonus Buy No. | r54 | Optional |
| `field2` | ประเภท (Material / Material Group) | r55 | Optional |
| `char` | *(characteristic เสริม — ไม่มีแถวตรงใน To-Be)* | — | — |
| `field4` | รหัสสินค้า / ชื่อกลุ่ม / Running No. | r56 | Optional |
| `description` | Description | r57 | Optional |
| `sapMasterDescription` | คำอธิบายจาก SAP Master (Long Thai) | r58 | Optional |
| `field7` | ราคาปกติ — **ราคาทุน (ไม่รวม VAT)** | r59 | Optional |
| `field8` | ราคาปกติ — **ราคาขาย** | r60 | Optional |
| `field9` | จำนวนชิ้นขั้นต่ำ | r61 | Optional |
| `field10` | มูลค่าขั้นต่ำ | r62 | Optional |
| `ean` | EAN | r63 | Optional |
| `serial` | Serial | r64 | Optional |
| `salesUnit` | Sales unit | r65 | Optional |
| `promotionTagSizeA4Cut1` | Pro Tag Size A4 Cut 1 | r66 | Optional |
| `promotionTagSizeA4Cut2` | Pro Tag Size A4 Cut 2 | r67 | Optional |
| `promotionTagSizeA4Cut4` | Pro Tag Size A4 Cut 4 | r68 | Optional |
| `promotionTagSizeA4Cut6` | Pro Tag Size A4 Cut 6 | r69 | Optional |

### 2.3 `get[]` (Main Header = "Get", r70–r106)

| payload key | ชื่อใน To-Be | แถว | required | หมายเหตุ |
|-------------|--------------|-----|----------|----------|
| `bonusBuyNumber` | Bonus Buy No. | r70 | **Mandatory** | |
| `field2` | ประเภท (Material / Material Group) | r71 | **Mandatory** | |
| `char` | *(characteristic เสริม)* | — | — | |
| `field4` | รหัสสินค้า / ชื่อกลุ่ม / Running No. | r72 | **Mandatory** | |
| `field5` | Description | r73 | Optional | |
| `sapMasterDescription` | คำอธิบายจาก SAP Master (Long Thai) | r74 | Optional | |
| `field7` | ราคาปกติ — **ราคาทุน (ไม่รวม VAT)** | r75 | Optional | |
| `field8` | ราคาปกติ — **ราคาขาย** | r76 | Optional | **= normal price** (P-series ใส่ตัวนี้) |
| `vat` | VAT | r77 | Optional | |
| `grossProfit` | GP% ปกติ (default) | r78 | Depends on Formula | Auto |
| `getQuantity` | Get Qty | r79 | **Mandatory** | จาก Mechanic |
| `tierNumber` | Tier — Tier No. | r80 | Optional | |
| `recursive` | Tier — Recursive | r81 | Optional | |
| `progressive` | Tier — Progressive | r82 | Optional | |
| `tierQuantityA` | Tier — Tier Quantity [A] | r83 | Optional | |
| `tierAmountB` | Tier — Tier Amount [B] | r84 | Optional | |
| `field17` | ราคาทุนจัดรายการ (ไม่รวม VAT) | r85 | Optional | |
| `fieldP` | ราคาจัดรายการ (THB) **[P]** | r86 | Optional | **ส่วนลดลำดับ 1** |
| `vat2` | VAT | r87 | Optional | |
| `grossProfit2` | GP% ใหม่ (auto) | r88 | Depends on Formula | Auto |
| `fieldR` | ส่วนลด จำนวนเงิน (THB) **[R]** | r89 | Optional | **ส่วนลดลำดับ 2** |
| `field22` | ส่วนลด **% [%]** | r90 | Optional | **ส่วนลดลำดับ 3** |
| `newGrossProfitRate` | New GP Rate (%) | r91 | Optional | |
| `ean` | EAN | r92 | Optional | |
| `serialNumber` | Serial No. | r93 | Optional | |
| `unit` | Unit | r94 | **Mandatory** | |
| `priceUnit` | Price unit | r95 | Optional | |
| `unitOfMeasure` | UOM | r96 | Optional | |
| `basicPoint` | Basic Point | r97 | Optional | |
| `extraPoint` | Extra Point | r98 | Optional | |
| `pointAmount` | Point amount | r99 | Optional | |
| `exclusion` | Exclusion | r100 | Optional | |
| `noDiscount` | No discount | r101 | Depends on Formula | Auto: On Top X=0.01% → mark X |
| `promotionTagSizeA4Cut1..6` | Pro Tag Size A4 Cut 1/2/4/6 | r102–r105 | Optional | |

> **ตัวเลขยืนยัน mapping** (จาก payload ตัวอย่าง, ลด 10% จากราคาขาย 990):
> `fieldP` 891 = 990×0.9 · `fieldR` 99 = 990×0.1 · `field17` 405 = 450×0.9 (ราคาทุน) → พิสูจน์ว่า `field8`=ราคาขาย, `field7`=ราคาทุน

### 2.4 บล็อกเสริม (Optional ทั้งหมด — เว้น `[]` ได้)

| payload block | Main Header ใน To-Be | แถว |
|---------------|----------------------|-----|
| `card[]` | Card Type | r107–r108 |
| `tender[]` | Tender Type | r109–r110 |
| `installment[]` | Installment (Bank, Interest rate, No.of month) | r111–r114 |
| `posTerminal[]` | POS Terminal | r115–r116 |
| `premium[]` | Premium Redemption | r117–r121 |
| `coupon[]` | Coupon (Logo/Header1-5/Detail1-5/Trailer1-5) | r122–r138 |
| `limitControl[]` | Limit Control (Per BBY/Per Card/Limit/MON–SUN/Max per ticket/card) | r139–r152 |
| `stores[]` | Store (X) (All / 13KA…67KA / Lazada / Shopee / …) | r2–r31 |

---

## 3. ต้องแก้ตรงไหนตามแต่ละ Profile (9 ตัว)

ทุก profile ใช้ HEADER + ลำดับส่วนลดเดียวกัน (`fieldP → fieldR → field22`) ต่างกันที่ **เขียนฝั่งไหน / ใส่ราคาขายปกติ (`get.field8`) ไหม / normalize % แบบไหน / header ใส่ช่องไหน**

| Profile | กลุ่ม | ฝั่งที่เขียน | normal price `get.field8` | Normalize % | `validTime*` | `referenceCode` | Online EN/TH (P4) | หมายเหตุ |
|---------|------|--------------|:-------------------------:|:-----------:|:------------:|:---------------:|:-----------------:|----------|
| **P001** | A | `get[]` | ✅ | เต็ม¹ | ✅ | — | — | มาตรฐาน Get-only |
| **P010** | A | `get[]` | ✅ | เต็ม¹ | ❌ ไม่ใส่ | ✅ | ✅ | ไม่ใส่เวลา; มี ref + online |
| **P011** | A | `get[]` | ✅ | เต็ม¹ | ✅ | — | ✅ | รับ ref แต่ไม่เขียน |
| **P015** | A | `get[]` | ✅ | เต็ม¹ | ✅ | — | — | คล้าย P001 |
| **D001** | B | `buy[]` + `get[]` | ❌ ไม่ใส่ | เต็ม¹ | ✅ | — | ✅ | buy: field4/field9(qty) |
| **F001** | B | `buy[]` + `get[]` | ❌ | ง่าย² | ✅ | — | ✅ | % แบบง่าย |
| **D002** | C | `buy[]` + `get[]` | ❌ | เต็ม¹ | ✅ | — | ✅ | โครงเหมือน D001 |
| **D003** | C | buy **หรือ** get (คนละ element) | ❌ | เต็ม¹ | ✅ | — | ✅ | Coupon: `A(Coupon)+B(A)`→buy, `+B(B)`→get element ก่อนหน้า |
| **F003** | C | buy **หรือ** get (แยก index) | ❌ | ง่าย² | ✅ | — | ✅ | `…(A)`→buy; `…(B)`→get |

¹ **Normalize เต็ม** (P001/P010/P011/P015/D001/D002/D003): ถ้า `%/100 < 1` ใช้ค่าเดิม; ถ้า `>100` หาร 100; ถ้า `<1` คูณ 100 (กัน 0.1 / 1000)

² **Normalize ง่าย** (F001/F003): `field22 = % / 100` ตรงๆ ไม่ปรับช่วง

### สรุปเชิงปฏิบัติสำหรับ backend

1. **routing** ด้วย `HEADER.bonusBuyProfile` → reject ถ้าไม่ใช่ 9 ตัวข้างบน
   - กลุ่ม A → เขียนแค่ `get[]` + ใส่ `get.field8` (ราคาขายปกติ)
   - กลุ่ม B → เขียนทั้ง `buy[]` + `get[]`, **ไม่ใส่** `get.field8`
   - กลุ่ม C → เหมือน B แต่ D003/F003 แยก buy/get คนละ element ตาม `bonusBuyHeader.mechanic`
2. **ราคาขายปกติ (`get.field8`):** ใส่เฉพาะกลุ่ม A
3. **% normalization:** F001/F003 = แบบง่าย, ที่เหลือ = แบบเต็ม
4. **header ราย profile:** P010 ไม่ใส่ `validTimeFrom/To` แต่ใส่ `referenceCode` + online; P011 ใส่เวลา+online ไม่มี ref; P001/P015 ใส่เวลา ไม่มี ref/online
5. **Online EN/TH:** เขียนเมื่อ `promotionArea = "P4"` เท่านั้น (ทุก profile ยกเว้น P001/P015)
6. **validation:** ใช้คอลัมน์ `G (required)` ของ To-Be — Get: `bonusBuyNumber`/`field2`/`field4`/`getQuantity`/`unit` = **Mandatory**

---

## 4. CONDITIONS[] — dictionary (ชีต `MerAB - item Details`)

ใช้เมื่อ `contractType` ขึ้นต้น `Z2`/`Z3` แต่ละ element = 1 contract

| payload block | Main Header ใน To-Be | แถว | Mandatory เด่น |
|---------------|----------------------|-----|-----------------|
| `conditionHeader` | Condition Header | r153–r178 | Contract No. (r154), Payment Method (r162), Additional Text (r175) |
| `businessVolumePurchase[]` | Business Volume Selection Criteria Purchase | r179–r192 | — |
| `businessVolumeSales[]` | Business Volume Selection Criteria Sales | r193–r216 | Contract No. (r193), Field Combination (r195), Billing Type (r212) |
| `conditionType[]` | Condition Type | r217–r235 | Contract No./Condition Table/Condition Type/Condition Rate |
| `settlementCalendar[]` | Settlement Calendar | r236–r238 | — |
| `combineCheck[]` | Combine Check | r239–r250 | — |
| `allocation[]` | Allocation Detail | r251–r255 | — |

> รอบแรก (Phase 1) contract อยู่นอก scope การ gen แถว BBY — แต่ payload ต้องรับโครงนี้ไว้

## 5. MATERIALS[]

ระดับสินค้า (planogram/forecast/ราคาต้นทุน/GP) — **ไม่เข้า Promotion BBY โดยตรง** ใช้ประกอบ/ตรวจ เว้น `[]` ได้

---

## 6. Checklist backend (AB)

- [ ] parse ตาม section `HEADER/BONUSBUYS/CONDITIONS/MATERIALS` + `LAYOUT="AB"`
- [ ] validate ด้วยคอลัมน์ `required` ของ To-Be (Mandatory ก่อน)
- [ ] HEADER: lookup theme/purGroup/profile, vendor ว่าง→NOBP, rebate ตัดอักษร, contractType Z2, รวมวันที่
- [ ] routing ตาม `bonusBuyProfile` → **เฉพาะ 9 profile**
- [ ] เขียน `get[]` ทุกกลุ่ม; `buy[]` เฉพาะ B/C
- [ ] `get.field8` (ราคาขายปกติ) เฉพาะกลุ่ม A
- [ ] % normalize: เต็ม vs ง่าย ตามตารางข้อ 3
- [ ] header ราย profile (validTime / referenceCode / online)
- [ ] D003/F003 แยก buy/get ตาม `mechanic`
- [ ] ลำดับส่วนลด `fieldP → fieldR → field22`

---

## ภาคผนวก A — Excel cell (ใช้เฉพาะตอน export กลับไฟล์ AB `Promotion BBY`)

เพื่อน **ไม่ต้องใช้ตารางนี้** — มีไว้ให้คนเขียนตัว export Excel เท่านั้น

| payload key | Excel cell (Promotion BBY) |
|-------------|----------------------------|
| `bonusBuyHeader.bonusBuyNumber` | T |
| `bonusBuyHeader.description` | U (prefix + ชื่อ) |
| `bonusBuyHeader.validTimeFrom` / `validTimeTo` | W / X |
| `bonusBuyHeader.promotionArea` | AA |
| `bonusBuyHeader.wbsNumber` | AB |
| `bonusBuyHeader.referenceCode` | AI |
| `bonusBuyHeader.onlineDescriptionEnglish` / `Thai` | AK / AL |
| `buy.bonusBuyNumber` | AN |
| `buy.field2` (type) | AO |
| `buy.field4` (material) | AQ |
| `buy.field9` (qty) | AV |
| `get.bonusBuyNumber` | BF |
| `get.field2` (type) | BG |
| `get.field4` (material) | BI |
| `get.field8` (ราคาขายปกติ) | BM |
| `get.fieldP` (promo [P]) | BW |
| `get.fieldR` (discount [R]) | BZ |
| `get.field22` (discount %) | CA |
| `get.getQuantity` | BP |
| `get.unit` | CE |

Header cell: `theme`→F9, `promotionName`→C8, `purchasingGroup`→C9, `bonusBuyProfile`→C10, `rebateChargeback`→C12, `contractType`→E12, `vendorCode`→L12, `periodFrom`→P11, `periodTo`→P12, `days`→P17/P19–P25
