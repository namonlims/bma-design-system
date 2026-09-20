# bma-design-system — เว็บ MIH Design System (ไฟล์ที่ build แล้ว)

เว็บนี้ generate อัตโนมัติ — **อย่าแก้ไฟล์ในนี้ด้วยมือ** ต้นทางอยู่ที่

- source ของเว็บ: `namonlims/mih-design` → `site/src` (ds-site/build.mjs + docs/design-system.html)
- component · design.md · tokens: `namonlims/mih-design` (sync จาก `namonlims/bma.health` ทุกครั้งที่ merge เข้า main)

ทุก push เข้า main ของ bma.health → workflow *Sync design system → mih-design* → build → push มาที่นี่ → GitHub Pages อัปเดตเอง
