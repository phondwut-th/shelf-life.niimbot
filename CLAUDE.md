# CLAUDE.md — ร้าน KACHA · เว็บและแอปจัดการหลังบ้าน

โฟลเดอร์นี้ใช้ทำเว็บและแอปหลังบ้านของร้าน · อ่านไฟล์นี้ก่อนเริ่มงานทุกครั้ง · อัปเดตล่าสุด 2026-11-23

## ร้าน

- **KACHA** ร้านชาสไตล์จีน (ชื่อเดิม YICHA เลิกใช้แล้ว · บางแอปเก่ายังใช้ชื่อ YICHA หรือ KachaMD) · ศรีราชา ชลบุรี ย่านหลัง มก.ศรีราชา
- เปิดกับหุ้นส่วน · Boon = เจ้าของ/คนคุยกับ Claude
- ลูกค้า: นักศึกษา มก.ศรีราชา + พนักงานนิคมแหลมฉบัง · หน้าร้าน + เดลิเวอรี (GP ~33%)
- คาดว่าจะขายได้ ~230 แก้ว/วัน (ยังเป็นค่าประมาณ) · 2 กะ (09–15 / 15–21) กะละ 2 คน
- เมนู: เบสชา 8 ตัว × ชาใส/ชานม (+ท็อปครีมชีส) และชาองุ่นปั่น · ไม่ทำกาแฟ มัทฉะ พีช
- รายละเอียดสูตร ต้นทุน และราคา ดูใน `shared/` (ตัวเลขจากที่นั่นที่เดียว ห้ามฝังเลขลงในโค้ด)

## วิธีทำงาน

- ตอบและเขียน UI เป็น **ภาษาไทย** · ตัวเลขใส่หน่วยชัดเจน (ก. มล. °C นาที ฿)
- คนใช้งานจริงคือพนักงานหน้าร้าน ซึ่งส่วนใหญ่ใช้ **มือถือ** → ต้องใช้บนจอเล็กได้ ปุ่มใหญ่ กรอกเร็ว
- ข้อมูลที่ยังไม่ยืนยัน ให้บอกตรงๆ หรือขึ้นป้าย "ยังไม่ยืนยัน" ใน UI
- ห้ามใช้ชื่อเมนู โลโก้ หรือหน้าตาแก้วของ CHAGEE หรือแบรนด์อื่น
- ลบหรือเขียนทับข้อมูลจริงใน db ต้อง backup ลง `<แอป>/data/` ก่อนทุกครั้ง

## โครงสร้าง

```
~/kacha/
├── CLAUDE.md
├── shared/            ข้อมูลกลาง (JSON) ที่ทุกแอปใช้ร่วมกัน · ดู shared/README.md
│   ├── bases.json       เบสชา 8 ตัว + ระบบชง
│   ├── menu.json        ราคา สูตรผสม สูตรเตรียม ต้นทุน/แก้ว
│   ├── ingredients.json วัตถุดิบ/บรรจุภัณฑ์ (id ตรงกับแอปสต็อก) + ร้านที่ซื้อ
│   └── suppliers.json   ร้านที่สั่งของ · ของถึงกี่วัน · สั่งทุกกี่วัน
├── fund/              แอปกองกลาง
│   ├── index.html       ซอร์ส (ย้ายมาจาก ~/yicha-fund.html)
│   └── data/            backup ข้อมูล
└── stock/             แอปวัตถุดิบ (สต็อก · ซื้อของ · บิล · ยอดขาย)
    ├── index.html       ซอร์ส
    └── seed.json        บิลสั่งของเปิดร้าน 23 พ.ย. (นำเข้า db แล้ว)
```

## รันที่ไหน

- โฟลเดอร์นี้เป็น git repo แบบ private: `phondwutth/kacha` · ใช้ได้ทั้งบน Mac (`~/kacha`) และบน Claude Code cloud
- `fund/data/` อยู่ใน .gitignore (เป็นข้อมูลการเงินจริง) → **บน cloud จะไม่มีโฟลเดอร์นี้** · path นอก repo (`~/xuebot`, `~/Desktop/KachaMD`, `~/Downloads`) ก็ไม่มีบน cloud เหมือนกัน
- บน cloud ใช้ Artifact/ArtifactData ได้แล้ว (publish แอป stock จาก cloud ครั้งแรก 2026-11-23) · แต่ `fund/data/` ไม่มีบน cloud ถ้าต้อง backup ข้อมูล fund ให้ทำจาก Mac
- ตอนนี้ยังไม่มี dependency (มีแค่ HTML + JSON) ไม่ต้องติดตั้งอะไร · ถ้าเพิ่ม package ภายหลัง ให้อัปเดต setup script ของ environment ด้วย

## แอปที่มีอยู่

### fund — กองกลาง (Artifact)
- URL: https://claude.ai/artifact/RidyLzPPEyhfCD5gWDiyrL · ซอร์ส `fund/index.html`
- แก้ไข: แก้ `fund/index.html` แล้ว publish ที่ URL เดิม (ใส่ `url`) · แก้ข้อมูลผ่าน ArtifactData
- capabilities: `db` + `user` + `assets` · collection เดียวคือ `tx`
  - `{kind:'expense'|'topup', amount, date:'YYYY-MM-DD', desc, createdAt, imgs:[assetId], payer?:'friend'}`
  - ถ้าไม่มี `payer` = จ่ายจากกองกลาง · ยอดกองกลาง = Σtopup − Σexpense ที่ไม่ใช่ friend
- ยอด ณ 2026-11-10: 77 รายการ · รายจ่ายรวม 39,637฿ · เติมกอง 35,000฿ · บัญชีเพื่อนจ่าย 5,131฿ · กองกลางเหลือ 494฿
- backup: `fund/data/tx-snapshot-2026-11-10.json` (ทั้ง collection) · `fund-seed-backup.json` + `สรุปเงินเบิก-YICHA.xlsx` (ประวัติเบิกเงินจาก Discord ช่วง ก.ค.–ต.ค.)
- กับดักที่เคยเจอ: `d.data()` คืนค่าเป็น object ที่ freeze แล้ว ต้องก๊อปก่อนเพิ่ม field · ปุ่มเลือกไฟล์ใช้ `<label for>` + input `opacity:0` (ถ้าใช้ `display:none` บน Android จะเปิดไม่ขึ้น)

### stock — วัตถุดิบร้าน (Artifact)
- URL: https://claude.ai/artifact/LLgy8HBaccmTPCPxChq2UU · ซอร์ส `stock/index.html`
- publish: ไฟล์หน้า `stock/index.html` + `files` = `shared/*.json` (path เดิม) และ `seed.json` ← `stock/seed.json` · หน้าเว็บ fetch ตัวเลขทั้งหมดจาก `shared/` ตอนเปิด → **แก้ตัวเลขใน shared/ แล้ว publish ใหม่** ไม่ต้องแก้โค้ด
- capabilities: `db` + `assets` · collections:
  - `bills/<id>` `{supplier, supplierName?, round, orderDate, dueDate?, ref, link?, lines:[{item, packs, price}], shipping, discount, amount, estimate, payments:[{amount, date, time, method, ref, payer, note, slips:[{id, pdf}]}], confirmed, received, receivedDate?, cancelled?, note, createdAt}`
  - `moves/<auto>` `{item, kind:'count'|'in'|'out', qty (หน่วยตาม ingredients.unit), date, eff? (count: วันที่ยอดนับมีผลสิ้นวัน), at, billId?, note?}`
  - `sales/<YYYY-MM-DD>` `{date, cups:{menuId: แก้ว}, at}` · `settings/plan` `{cups, mode:'grade'|'equal'|'custom', custom?, cheese, safety, openDate, autoDeduct, round}`
- สต็อก = ยอดนับล่าสุด + รับเข้า − ใช้/ทิ้ง − วัตถุดิบตามยอดขายที่บันทึก (ถ้าตั้งวันเปิดร้าน วันที่ไม่บันทึกยอดจะหักตามแผน) · ของในบิลที่ยังไม่ได้กด "รับของ" = รอของเข้า
- seed บิลเปิดร้าน 5 ใบ ลง db แล้ว 2026-11-23

### แอปอื่นของร้านที่ยังไม่ได้ย้ายเข้ามา
- **สต็อก KachaMD** (Artifact, db · แอปเก่า แอป stock ด้านบนทำแทนได้แล้ว): https://claude.ai/code/artifact/b6d95f60-4a86-42ec-ba16-4fce915fd70b · collection `items` + `days/<YYYY-MM-DD>`
- **xuebot** Discord bot (`~/xuebot`, รันบน Railway) · ระบบเบิกเงินเลิกใช้แล้ว ย้ายไปใช้แอป fund แทน
- **ผังบาร์ 3D** (`~/kacha-bar`) · ข้อมูลอยู่ใน `bar-layout.json`

## ที่มาของข้อมูล

- สูตรและงาน R&D: Obsidian vault `~/Desktop/KachaMD` (มี CLAUDE.md ของตัวเองสำหรับงาน R&D) · ไฟล์หลักคือ `สูตรเบสพร้อมขาย_v*` ตัวล่าสุด
- `~/Downloads/KACHA-ข้อมูลร้าน-ผังบาร์_v1.0_2026-10-29.md` เป็นข้อมูลร้านชุดใหม่สุดที่ใช้ตั้งต้น `shared/`
- อายุเบส **24 ชม.** (Boon ยืนยัน 2026-11-10)
- สูตรครีมชีสโฟม **ชุด Millac Gold** (Boon ยืนยัน 2026-11-10)
- 2026-11-23 อัปเดต shared/ จากสรุปแชทสั่งของเปิดร้าน + สูตร v3.37: ราคาใบชาจริง · ทิกวนอิมเปลี่ยนเป็น Overlord (it37) · มะลิองุ่นใช้ Chaiwala ลี่ลี่เซียง (it38) · คามิเลีย Chaiwala 30 + Yiwulong 10 · ไซรัปชานม 30 มล. · เกลือชมพู 3.5 ก./ล็อต · ยังไม่ยืนยัน: ไซรัปชาใส (ใช้ 30 มล.), บุก 50 ก./แก้ว, ท็อปครีมชีส 20%, lead/รอบสั่งของแต่ละร้าน
