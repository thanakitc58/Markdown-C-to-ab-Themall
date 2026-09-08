# Material Grouping Notes

ใช้เอกสารนี้เพื่ออธิบายว่า `materialGrouping` ถูก generate อย่างไร, material จาก C ไปอยู่ตรงไหนใน `BONUSBUYS`, และ frontend/backend ต้องใช้ `line_number` แบบใด

## ภาพรวม

`materialGrouping` เป็นข้อมูลที่ save ลง table `promotion_material_grouping`

field หลักมีดังนี้:

- `running_number`
- `grouping_name`
- `description`
- `category`
- `component`
- `lineNo` ใช้เป็นตำแหน่งแถวตอน save และ map ไป `line_number`

## Rule สำคัญที่ต้องส่งให้ Dev

1. ถ้าแถว C มี `number_of_material_grouping` และระบบสร้าง `MATERIALGROUPINGS` แล้ว
   ฝั่ง AB ใน `BONUSBUYS[].buy[]` / `BONUSBUYS[].get[]` ต้องอ้าง **กลุ่มสินค้า**
   ไม่ใช่รหัส material รายตัว
2. ค่าอ้างอิงของกลุ่มคือ `grouping_name`
3. `material_or_group_type` ควรเป็นกลุ่มสินค้า เช่น `Material Group`
4. `material_or_group_code` ควรเป็น `grouping_name`
5. `line_number` / `lineNo` ของทุก section ให้เริ่มที่ `1` ไม่ใช่ `0`

## Material ไปอยู่ตรงไหนใน BBY

### ตอนเป็น material รายตัว

- `BONUSBUYS[].buy[].material_or_group_type` = `Material`
- `BONUSBUYS[].buy[].material_or_group_code` = `material`
- `BONUSBUYS[].get[].material_or_group_type` = `Material`
- `BONUSBUYS[].get[].material_or_group_code` = `material`

### ตอนเป็น material grouping

- `BONUSBUYS[].buy[].material_or_group_type` = `Material Group`
- `BONUSBUYS[].buy[].material_or_group_code` = `grouping_name`
- `BONUSBUYS[].get[].material_or_group_type` = `Material Group`
- `BONUSBUYS[].get[].material_or_group_code` = `grouping_name`

สรุปคือ material จาก C ไม่ได้หายไป แต่ย้ายไปอยู่ 2 ที่:

1. `MATERIALGROUPINGS[].component` เก็บ member ของกลุ่ม
2. `BONUSBUYS[].buy/get[].material_or_group_code` เก็บ `grouping_name` ของกลุ่มนั้น

ตัวอย่าง:

```js
{
  line_number: 1,
  running_number: "1",
  grouping_name: "MM8D0908D1",
  description: "MM8D0908D1",
  category: "1 - Material No.",
  component: "1000728050"
}
```

```js
{
  line_number: 1,
  bonus_buy_number: "1",
  buy: [
    {
      bonus_buy_number: "1",
      material_or_group_type: "Material Group",
      material_or_group_code: "MM8D0908D1"
    }
  ],
  get: [
    {
      bonus_buy_number: "1",
      material_or_group_type: "Material Group",
      material_or_group_code: "MM8D0908D1"
    }
  ]
}
```

## Rule ของ line number

เพื่อให้ตรงกันทั้ง FE / convert / backend ให้ใช้กติกานี้:

- `BONUSBUYS[].line_number` เริ่มที่ `1`
- `CONDITIONS[].line_number` เริ่มที่ `1`
- `MATERIALS[].line_number` เริ่มที่ `1`
- `MATERIALGROUPINGS[].line_number` เริ่มที่ `1`
- ถ้า payload ฝั่ง backend ใช้ `lineNo` ให้ส่งค่าเดียวกันแบบ 1-based

ตัวอย่าง:

```js
[
  { line_number: 1, running_number: "1", component: "1000727961" },
  { line_number: 2, running_number: "1", component: "1000727962" },
  { line_number: 3, running_number: "2", component: "BR-9" }
]
```

## ต้องแยกตาม profile ไหม

ถ้าในใบเดียวมีหลาย profile หรือหลาย mechanic ให้แยกคนละก้อน BBY

group key ที่ปลอดภัยคือ:

```text
(number_of_promotion, number_of_material_grouping, bonus_buy_profile, mechanic)
```

แต่ถ้าในใบเดียวเป็น profile/mechanic เดียวกันทั้งหมด
ก็ group ตาม `(number_of_promotion, number_of_material_grouping)` ได้

## สูตร generate `grouping_name`

ฝั่ง backend มีฟังก์ชัน `materialGroupingName()` อยู่ใน `tmg-supplier-portal-backend/modules/promotion/service.ts`

สูตรตามโค้ด:

```text
grouping_name = "MM" + HEX(seconds_since_2022_01_01_UTC) + running_number
```

โดย:

- `"MM"` เป็น prefix คงที่
- `seconds_since_2022_01_01_UTC` คือจำนวนวินาทีตั้งแต่ `2022-01-01 00:00:00 UTC`
- `HEX(...)` คือแปลงเลขวินาทีเป็นเลขฐาน 16 ตัวพิมพ์ใหญ่
- `running_number` คือเลขลำดับของ Material Grouping ใน sheet

สูตรนี้สอดคล้องกับ comment ในโค้ด:

```text
="MM" & DEC2HEX((NOW()-DATE(2022,1,1))*86400) & A4
```

## ตัวแปรที่ใช้ในสูตร

ฟังก์ชัน generate ใช้ตัวแปรเพียง 2 ตัว:

- `runningNumber: string`
- `at: Date`

โค้ดจริง:

```ts
const EPOCH_2022 = Date.UTC(2022, 0, 1);

export const materialGroupingName = (runningNumber: string, at: Date): string => {
  const seconds = Math.floor((at.getTime() - EPOCH_2022) / 1000);
  return `MM${seconds.toString(16).toUpperCase()}${runningNumber}`;
};
```

## เงื่อนไขตอน backend generate

backend จะ generate `grouping_name` เฉพาะกรณีนี้:

1. `grouping_name` ยังว่าง
2. `running_number` มีค่า

ถ้า `grouping_name` ถูกส่งมาแล้ว backend จะใช้ค่านั้นตรง ๆ และไม่ generate ใหม่

ถ้ามีหลายแถวที่ใช้ `running_number` เดียวกัน backend จะ reuse `grouping_name` เดิมให้ทุกแถวนั้น

logic ตอน save:

```ts
const stampedAt = new Date();

for (const [i, group] of (payload.materialGroupings || []).entries()) {
  const { lineNo, ...cols } = group || {};
  const running = `${cols.running_number ?? ""}`;
  let name = cols.grouping_name;

  if (!notBlank(name) && notBlank(running)) {
    name = groupingNames[running] ?? materialGroupingName(running, stampedAt);
    groupingNames[running] = name;
  }

  await m.save(
    m.create(PromotionMaterialGrouping, {
      promotion_id: promotionId,
      line_number: lineNo ?? i,
      ...cols,
      ...(notBlank(name) ? { grouping_name: name } : {}),
    })
  );
}
```

## Payload ที่ frontend ส่งไป backend

ฝั่ง frontend map key ของ `materialGrouping` ไว้ใน `tmg-supplier-portal-frontend/constants/promotion-payload.js`

mapping:

```js
materialGrouping: {
  running_number: "running_number",
  grouping_name: "grouping_name",
  description: "description",
  category: "category",
  component: "component",
}
```

สรุปว่า object แต่ละ row ใน `payload.materialGroupings` จะมี shape ประมาณนี้:

```js
{
  lineNo: 1,
  running_number: "1",
  grouping_name: "MM858533A1",
  description: "buds5",
  category: "1 - Material No.",
  component: "1000727961"
}
```

หมายเหตุ:

- `lineNo` ไม่ได้เป็น column ธุรกิจ แต่ใช้ map ไป `line_number` ใน DB
- ถ้าทีมตกลงใช้ 1-based index ให้ FE ส่ง `lineNo` เริ่มที่ `1`
- `grouping_name` ส่งมาได้ แต่ถ้าว่าง backend สามารถ generate ให้

## Field ใน database

entity ของ `promotion_material_grouping` รองรับ field ต่อไปนี้:

- `promotion_id`
- `line_number`
- `running_number`
- `grouping_name`
- `description`
- `category`
- `component`

metadata อื่น ๆ ที่ backend ดูแลเอง:

- `id`
- `created_by`
- `updated_by`
- `created_date`
- `updated_date`
- `version`

## ตัวอย่างผลลัพธ์

ถ้า save ที่เวลาเดียวกัน และมีข้อมูล:

```js
[
  { running_number: "1", component: "1000727961" },
  { running_number: "1", component: "1000727962" },
  { running_number: "2", component: "BR-9" }
]
```

ผลที่คาด:

- row 1 และ row 2 จะได้ `grouping_name` เดียวกัน เพราะ `running_number` เท่ากัน
- row 3 จะได้ `grouping_name` อีกค่า เพราะเป็น `running_number = "2"`

ตัวอย่างจาก test:

```js
[
  { lineNo: 1, running_number: "1", grouping_name: "MM858533A1", description: "buds5", category: "1 - Material No.", component: "1000727961" },
  { lineNo: 2, running_number: "1", grouping_name: "MM858533A1", description: "buds5", category: "1 - Material No.", component: "1000727962" },
  { lineNo: 3, running_number: "2", grouping_name: "MM858533A2", description: "cases", category: "4 - Brand", component: "BR-9" }
]
```

## Query ตรวจใน DB

```sql
SELECT
  id,
  promotion_id,
  line_number,
  running_number,
  grouping_name,
  description,
  category,
  component,
  created_date,
  updated_date
FROM promotion_material_grouping
WHERE promotion_id = <promotion_id>
ORDER BY line_number, id;
```

## สรุปสั้น

- สูตร generate ฝังอยู่ใน backend
- สูตรคือ `MM + hex(timestamp-from-2022) + running_number`
- frontend ส่ง `running_number`, `grouping_name`, `description`, `category`, `component`
- ถ้า `grouping_name` ว่าง backend จะ generate ให้เอง
- ถ้า `running_number` ซ้ำกันหลายแถว จะได้ `grouping_name` เดียวกัน
- ถ้าเป็น mat-group ฝั่ง `BONUSBUYS` ต้องอ้าง `grouping_name` ไม่ใช่ `material`
- `line_number` / `lineNo` ให้เริ่มที่ `1`
