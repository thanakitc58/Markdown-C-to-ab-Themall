# Dev Handoff — Material Grouping ลง BBY

ใช้ไฟล์นี้ส่งต่อ Dev เพื่อทำงาน `material grouping -> BONUSBUYS` ให้ตรง macro และตรงกับ payload ใหม่

## สรุปงานที่ต้องทำ

ตอนนี้ flow `material grouping` เดินมาเกือบครบแล้ว:

- มี `number_of_material_grouping` ใน `MATERIALS`
- สร้าง `MATERIALGROUPINGS[]` ได้แล้ว
- group `BONUSBUYS[]` ตาม `(number_of_promotion, number_of_material_grouping)` ได้แล้ว

**สิ่งที่ยังต้องทำต่อ** คือเอา `material grouping` ไปใช้ใน `BONUSBUYS[].buy[]` และ `BONUSBUYS[].get[]` ให้ตรง macro

## สิ่งที่ต้องแก้

### 1) ถ้าเป็น mat-group แล้ว `BONUSBUYS` ต้องอ้างกลุ่ม ไม่ใช่ material รายตัว

ถ้าแถว C มี `number_of_material_grouping` และระบบสร้าง `MATERIALGROUPINGS[]` แล้ว:

- `buy[].material_or_group_type` ต้องเป็น `Material Group`
- `buy[].material_or_group_code` ต้องเป็น `grouping_name`
- `get[].material_or_group_type` ต้องเป็น `Material Group`
- `get[].material_or_group_code` ต้องเป็น `grouping_name`

**ห้าม** ใช้แบบนี้เมื่อเป็น mat-group:

- `material_or_group_type = "Material"`
- `material_or_group_code = material`

## Material จาก C ไปอยู่ตรงไหน

material 1 แถวจาก C จะถูกใช้ 2 ส่วน:

1. ไปเป็นสมาชิกใน `MATERIALGROUPINGS[].component`
2. ไปผูกกับ `BONUSBUYS` ผ่าน `grouping_name`

ดังนั้นเวลาเป็น mat-group:

- `MATERIALGROUPINGS[].component` = material จริง
- `BONUSBUYS[].buy/get[].material_or_group_code` = `grouping_name`

## Rule ของ line number

ให้ใช้ `1-based` ทุก section

- `BONUSBUYS[].line_number` เริ่มที่ `1`
- `CONDITIONS[].line_number` เริ่มที่ `1`
- `MATERIALS[].line_number` เริ่มที่ `1`
- `MATERIALGROUPINGS[].line_number` เริ่มที่ `1`

ถ้าฝั่ง backend ใช้ `lineNo` ก็ให้ส่งค่าเดียวกันแบบ 1-based

## ต้องแยกตาม profile ไหม

ถ้าในใบเดียวมีหลาย profile หรือหลาย mechanic ต้องแยก BBY คนละก้อน

group key ที่ปลอดภัย:

```text
(number_of_promotion, number_of_material_grouping, bonus_buy_profile, mechanic)
```

สรุป:

- ถ้า profile และ mechanic เหมือนกันทั้งก้อน -> group ตาม `(promotion, mat-group)`
- ถ้า profile หรือ mechanic ต่างกัน -> แยกคนละ BBY

## ตัวอย่าง Input C

```json
{
  "LAYOUT": "C",
  "MATERIALS": [
    {
      "line_number": 1,
      "number_of_promotion": 1,
      "number_of_material_grouping": 1,
      "material": "1000728050",
      "bonus_buy_profile": "F001",
      "mechanic": "B1G1"
    },
    {
      "line_number": 2,
      "number_of_promotion": 1,
      "number_of_material_grouping": 1,
      "material": "1000728050",
      "bonus_buy_profile": "F001",
      "mechanic": "B1G1"
    },
    {
      "line_number": 3,
      "number_of_promotion": 2,
      "number_of_material_grouping": 2,
      "material": "1000728050",
      "bonus_buy_profile": "F001",
      "mechanic": "B1G1"
    }
  ]
}
```

## ตัวอย่าง `MATERIALGROUPINGS` ที่ควรได้

```json
[
  {
    "line_number": 1,
    "running_number": "1",
    "grouping_name": "MM8D0908D1",
    "description": "MM8D0908D1",
    "category": "1 - Material No.",
    "component": "1000728050"
  },
  {
    "line_number": 2,
    "running_number": "1",
    "grouping_name": "MM8D0908D1",
    "description": "MM8D0908D1",
    "category": "1 - Material No.",
    "component": "1000728050"
  },
  {
    "line_number": 3,
    "running_number": "2",
    "grouping_name": "MM8D0908D2",
    "description": "MM8D0908D2",
    "category": "1 - Material No.",
    "component": "1000728050"
  }
]
```

## Output AB ที่ผิดตอนนี้

ตอนนี้ output ยังอ้าง material รายตัวอยู่:

```json
{
  "LAYOUT": "AB",
  "BONUSBUYS": [
    {
      "line_number": 1,
      "bonus_buy_number": "1",
      "buy": [
        {
          "bonus_buy_number": "1",
          "material_or_group_type": "Material",
          "material_or_group_code": "1000728050"
        }
      ],
      "get": [
        {
          "bonus_buy_number": "1",
          "material_or_group_type": "Material",
          "material_or_group_code": "1000728050"
        }
      ]
    }
  ]
}
```

## Output AB ที่ถูก

เมื่อเป็น mat-group แล้ว `BONUSBUYS` ต้องอ้าง `grouping_name`

```json
{
  "LAYOUT": "AB",
  "BONUSBUYS": [
    {
      "line_number": 1,
      "bonus_buy_number": "1",
      "bonusBuyHeader": {
        "bonus_buy_number": "1",
        "bonus_buy_profile": "F001",
        "mechanic": "B1G1"
      },
      "buy": [
        {
          "bonus_buy_number": "1",
          "material_or_group_type": "Material Group",
          "material_or_group_code": "MM8D0908D1"
        }
      ],
      "get": [
        {
          "bonus_buy_number": "1",
          "material_or_group_type": "Material Group",
          "material_or_group_code": "MM8D0908D1"
        }
      ]
    },
    {
      "line_number": 2,
      "bonus_buy_number": "2",
      "bonusBuyHeader": {
        "bonus_buy_number": "2",
        "bonus_buy_profile": "F001",
        "mechanic": "B1G1"
      },
      "buy": [
        {
          "bonus_buy_number": "2",
          "material_or_group_type": "Material Group",
          "material_or_group_code": "MM8D0908D2"
        }
      ],
      "get": [
        {
          "bonus_buy_number": "2",
          "material_or_group_type": "Material Group",
          "material_or_group_code": "MM8D0908D2"
        }
      ]
    }
  ]
}
```

## Mapping ที่ Dev ใช้ได้เลย

### C -> MATERIALGROUPINGS

- `MATERIALS[].line_number` -> `MATERIALGROUPINGS[].line_number`
- `MATERIALS[].number_of_material_grouping` -> `MATERIALGROUPINGS[].running_number`
- `grouping_name` -> generate / reuse ตาม `running_number`
- `description` -> ใช้ค่าเดียวกับ `grouping_name` ตามตัวอย่างปัจจุบัน
- `category` -> `"1 - Material No."`
- `component` -> `MATERIALS[].material`

### MATERIALGROUPINGS -> BONUSBUYS

หลัง group BBY แล้ว:

- หา `grouping_name` ของกลุ่มนั้นจาก `running_number`
- set `buy[].material_or_group_type = "Material Group"`
- set `buy[].material_or_group_code = grouping_name`
- set `get[].material_or_group_type = "Material Group"`
- set `get[].material_or_group_code = grouping_name`

## Acceptance Checklist

- [ ] ถ้ามี mat-group แล้ว `BONUSBUYS.buy/get` ใช้ `grouping_name`
- [ ] ไม่ใช้ material code ตรงๆ ใน `buy/get` ของก้อนที่เป็น mat-group
- [ ] `MATERIALGROUPINGS[].component` ยังเก็บ material จริงครบทุกแถว
- [ ] `running_number` เดียวกันได้ `grouping_name` เดียวกัน
- [ ] `line_number` ทุก section เริ่มที่ `1`
- [ ] ถ้ามีหลาย profile/mechanic ในใบเดียว BBY แยกตาม profile/mechanic

## หมายเหตุ

- ตอนนี้ตัวอย่างใน `matG.json` ถูกปรับให้เป็นแนวทางที่ถูกแล้วสำหรับเคส mat-group
- ถ้าทีม backend ใช้ enum short code แทนคำว่า `Material Group` ให้ยืนยันอีกครั้งก่อนปล่อยจริง
