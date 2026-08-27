# Bug / แจ้งเพื่อน — `rebateChargeback` หายหลัง convert

| | |
|--|--|
| **วันที่พบ** | 2026-08-27 |
| **เคสที่เทส** | F001 · Reason `X - สร้าง Contract พร้อม BBY` · Contract Type `Z200` |
| **สถานะ** | BBY + CONDITIONS ผ่าน · **HEADER.rebateChargeback เพี้ยน** |

---

## สรุปสั้นๆ ให้เพื่อน

ตอนเลือก **Reason = X (สร้าง Contract พร้อม BBY)** + **Contract Type = Z200** แล้ว convert:

- `BONUSBUYS` ถูก (F001 / B3G1 ฯลฯ)
- `CONDITIONS` copy จาก Condition Header ถูก — ยังมี `cond_hdr_reason: "X"`
- แต่ **`HEADER.rebateChargeback` ใน AB กลายเป็น `""`** ทั้งที่ C เป็น `"X"`

ขอช่วยไล่ `normalizeRebate` (shared header) ให้ตรง macro

---

## Expected vs Actual

| ฟิลด์ AB | Expected (ตาม macro) | Actual (เว็บตอนนี้) |
|----------|----------------------|---------------------|
| `HEADER.contractType` | `"Z200"` | `"Z200"` ✅ |
| `HEADER.rebateChargeback` | `"X"` | `""` ❌ |
| `CONDITIONS[0].conditionHeader.cond_hdr_reason` | `"X"` | `"X"` ✅ |

### C (ก่อนแปลง — ส่วนที่เกี่ยว)

```json
"HEADER": {
  "contractType": "Z200",
  "rebateChargeback": "X",
  "bonusBuyProfile": "F001"
}
```

```json
"CONDITIONS": [{
  "conditionHeader": {
    "cond_hdr_reason": "X",
    "cond_hdr_contract_type": "Z200",
    "cond_hdr_vendor": "OIS05"
  }
}]
```

### AB (หลังแปลง — จุดพัง)

```json
"HEADER": {
  "rebateChargeback": "",
  "contractType": "Z200"
}
```

---

## กติกาที่ถูก (จาก macro `set_RebateReason_n_ContractType`)

อ่านจาก `Mer-C_Convert_To_STD.txt`:

```
ถ้า Contract Type ขึ้นต้น "Z2":
  contractType  = ค่าเต็ม (เช่น Z200)
  rebateChargeback = ตัวอักษรแรกของ Reason   → "X"

ถ้า Contract Type ไม่ใช่ Z2:
  contractType  = ""
  ถ้าตัวแรกของ Reason ≠ "A":
    rebateChargeback = "0" + ตัวอักษรแรก     → เช่น "0X"
  ไม่งั้น:
    rebateChargeback = "A"
```

**สำคัญ:** macro ใช้แบบ **เอา Left(1)** ของ Reason  
ไม่ใช่ตัดตัวแรกทิ้งแล้วเหลือส่วนที่เหลือ

บนเว็บค่าใน C มักเหลือแค่ `"X"` อยู่แล้ว  
ถ้าโค้ดทำ `slice(1)` / ตัดตัวแรกทิ้ง → ได้ `""` ← ตรงกับบั๊กที่เจอ

---

## สาเหตุที่น่าจะเป็น

`normalizeRebate` ใน shared header **ตัดตัวอักษรแรกทิ้ง** จากค่าที่มีความยาว 1 แล้ว (`"X"`)  
เลยว่างทั้งก้อน

ควรเป็นประมาณ:

- ถ้าว่าง → ว่าง
- ถ้ามี Contract แบบ Z2 → ใช้ตัวอักษรแรกของ reason/rebate (= `"X"` ถ้าค่าเป็น `"X"` หรือ `"X - สร้าง…"`)
- ถ้าไม่ใช่ Z2 และตัวแรก ≠ `A` → เติม `0` นำหน้า (`"0X"`)

อย่า `substring(1)` บนสตริงที่เหลือรหัสตัวเดียวแล้ว

---

## ไฟล์อ้างอิงในสเปก

| ไฟล์ | หัวข้อ |
|------|--------|
| `Mer-C_Convert_To_STD.txt` | `set_RebateReason_n_ContractType` (~บรรทัด 400) |
| `MerC_to_AB_Header_Shared.md` | `normalizeRebate` |
| `MerC_to_AB_Phase1_Shared.md` | Charge Back / Reason → `rebateChargeback` |
| `profile/MerC_to_AB_*.md` §3 | HEADER map ร่วมทุกโปรไฟล์ |

---

## วิธีเทสซ้ำหลังแก้

1. Condition Header: Reason = `X - สร้าง Contract พร้อม BBY` · Contract Type = `Z200`
2. Convert (โปรไฟล์อะไรก็ได้ เช่น F001)
3. เช็ค AB:
   - [ ] `HEADER.rebateChargeback` = `"X"`
   - [ ] `HEADER.contractType` = `"Z200"`
   - [ ] `CONDITIONS` ยังมี reason / contract type
   - [ ] `BONUSBUYS` ไม่พัง

เคสเสริม (ถ้ามีเวลา):

| Reason | Contract Type | `rebateChargeback` ที่ควรได้ |
|--------|---------------|------------------------------|
| `X…` | `Z200` | `"X"` |
| `A…` | (ว่าง / ไม่ใช่ Z2) | `"A"` |
| `X…` | (ว่าง / ไม่ใช่ Z2) | `"0X"` |

---

## สิ่งที่ไม่ต้องแก้จากเคสนี้

- โครง F001 buy+get / mechanic qty — ผ่านแล้ว
- copy `CONDITIONS` จาก Condition Header — ผ่านแล้ว
- การแยก BBY ตามโปรไฟล์ — ไม่เกี่ยวกับบั๊กนี้

---

## ข้อความสั้นๆ คัดลอกไปแชทได้

> เจอบั๊ก shared header: เลือก Reason `X` + Contract Type `Z200` แล้ว convert  
> C มี `rebateChargeback: "X"` แต่ AB ได้ `""`  
> CONDITIONS ยังมี `cond_hdr_reason: "X"` อยู่ — น่าจะพังที่ `normalizeRebate` ตัดตัวแรกทิ้งทั้งที่ค่าเหลือตัวเดียว  
> ตาม macro ต้องได้ `"X"` (Z2 แล้วเอา Left(1) ของ reason)  
> รายละเอียดอยู่ใน `tungconvert/BUG-rebateChargeback-normalize.md`
