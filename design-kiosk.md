# Kiosk Design System (bma.kiosk)

> **Design system ของ Kiosk แยกจาก HCIS** (ผู้ใช้ 24 ต.ค. 69) — ไฟล์นี้คือต้นทาง แก้ที่ repo นี้ได้เลย
> Token: mih-design `packages/design-tokens/kiosk.css` (`--kiosk-*` · ไฟล์เดียวจบ ไม่อ้าง token ของ HCIS) → `npm run tokens:pull` → `src/styles/kiosk-tokens.css`
> Figma Kiosk `AGes4Mc8odyTEYARFeFqcj` · variables collection "Kiosk" (code syntax = `var(--kiosk-*)`) · effect styles `kiosk/shadow/*`

## 8. Kiosk Design System (bma.kiosk)

> Kiosk = ตู้บริการตนเองสำนักอนามัย 1080×1920 แนวตั้ง · repo `namonlims/bma.kiosk` (React 19 + Tailwind v4) · ลงทะเบียน 24 ต.ค. 69
> สีหลักเป็น**น้ำเงิน** (ไม่ใช่เขียวมรกตของเว็บ) และตัวอักษรใหญ่สำหรับอ่านจากระยะยืน — จึงเป็นชั้น platform แยก `--kiosk-*` แบบเดียวกับ `--mobile-*` (§7)

### 8.1. Layering rules

| ชั้น | ที่อยู่ | ใครใช้ |
|---|---|---|
| Foundation `--dim-*` · `--type-weight-*` · primitives | `:root` (§1–§3) | ทุกแอป |
| Kiosk platform `--kiosk-*` | mih-design `packages/design-tokens/kiosk.css` | **bma.kiosk เท่านั้น** — เว็บ/มือถือห้ามใช้ |
| Kiosk re-point ของ semantic ที่ใช้ร่วม | `:root[data-app='kiosk']` (ยังไม่มี) | bma.kiosk ตั้ง `<html data-app="kiosk">` แล้ว |
| Tailwind `@theme` | `bma.kiosk/src/index.css` | alias `var(--kiosk-*)` เท่านั้น — ห้ามใส่ค่าดิบ |

- token ใหม่ของ kiosk → เพิ่มใน **mih-design** `packages/design-tokens/kiosk.css` + ตารางในไฟล์นี้ + variable ใน Figma Kiosk collection "Kiosk" → `npm run tokens:pull` → alias ใน `@theme` → commit mirror `src/styles/kiosk-tokens.css`
- ในโค้ด kiosk ใช้ class ของ theme (`text-kiosk-blue` · `bg-kiosk-card` · `text-kiosk-48`) — ห้าม `text-[#0d4477]` / `text-[48px]`
- ยกเว้น: canvas (`receiptDownload.ts` — canvas อ่าน `var()` ไม่ได้) · โลโก้แบรนด์ภายนอก (ThaID) · illustration ใน `src/assets/`

### 8.2. Platform tokens (`--kiosk-*`)

#### Colour

| Token | Value | Tailwind | ใช้ที่ |
|---|---|---|---|
| `--kiosk-blue` | #0D4477 | `kiosk-blue` | หัวข้อ · ข้อความหลัก · ปุ่ม Fill |
| `--kiosk-blue-pressed` | #022D56 | `kiosk-blue-pressed` | ปุ่ม Fill ตอนกด |
| `--kiosk-blue-muted` | #466E93 | `kiosk-blue-muted` | ข้อความรอง · คำอธิบาย |
| `--kiosk-blue-subdued` | rgb(13 68 119 / 0.5) | `kiosk-blue-subdued` | Outline Subdued ขอบ/พื้นกด (Figma 268:14108) |
| `--kiosk-field-border` | rgb(13 68 119 / 0.2) | `kiosk-field-border` | ช่องเลขบัตร · passport · แท็บเอกสารที่ไม่ได้เลือก |
| `--kiosk-ink` | #1D1F23 | `kiosk-ink` | ค่าตัวเลขเข้มสุด (ใบเสร็จ / ใบคิว) |
| `--kiosk-disabled` | #C2C8CE | `kiosk-disabled` | ปุ่ม disabled พื้น/ขอบ/ข้อความ |
| `--kiosk-green` | #41614D | `kiosk-green` | เนื้อความ/คำอธิบาย (ใช้ที่ 70%) |
| `--kiosk-surface-card` | #EEFFFC | `kiosk-card` | พื้นการ์ด / ไฮไลต์ |
| `--kiosk-mint-50 / 100 / 200` | #EAFFFB · #F7FFFD · #C8FFF5 | `kiosk-mint-*` | พื้นหลังหน้าจอไล่สี ต้น · กลาง · ปลาย |
| `--kiosk-pink-50 / 100 / 200` | #FFEBF6 · #FFF4FA · #FFC1E2 | `kiosk-pink-*` | พื้นหลังหน้าจอผิดพลาดไล่สี |

#### Elevation · shape

| Token | Value | Tailwind |
|---|---|---|
| `--kiosk-shadow-card` / `-hover` | 0 10 24 0 rgba(145,232,216, .2 / .6) | `shadow-kiosk-card` / `-hover` |
| `--kiosk-shadow-outline-button` / `-pressed` | 0 10 48 0 · 0 10 24 0 rgba(145,232,216,.2) | `shadow-kiosk-outline-button` / `-pressed` |
| `--kiosk-radius-card` | 24px | `rounded-kiosk-card` |

#### Type (Prompt · large-screen scale)

| Token | px | Tailwind | บทบาท |
|---|---:|---|---|
| `--kiosk-type-family` | Prompt, Noto Sans Thai | `font-prompt` | ทุกข้อความ |
| `--kiosk-type-family-display` | Noto Sans Thai, Prompt | `font-thai-display` | แทน wordmark "Sao Chingcha" |
| `--kiosk-type-size-100` | 100 | `text-kiosk-100` | เลขคิว · ตัวเลขเด่น (semibold) |
| `--kiosk-type-size-58` · `-56` | 58 · 56 | `text-kiosk-58` · `-56` | หัวข้อหน้าจอ |
| `--kiosk-type-size-48` | 48 | `text-kiosk-48` | หัวข้อส่วน · ป้ายปุ่ม |
| `--kiosk-type-size-40` · `-36` | 40 · 36 | `text-kiosk-40` · `-36` | เนื้อความใหญ่ · หัวการ์ด |
| `--kiosk-type-size-32` · `-28` · `-24` | 32 · 28 · 24 | `text-kiosk-32` … | เนื้อความ · ป้าย · caption |
| `--kiosk-type-size-20` · `-16` · `-14` | 20 · 16 · 14 | `text-kiosk-20` … | ใบคิว/ใบเสร็จที่พิมพ์ |

ชื่อ = ขนาด px ตาม text style ใน Figma ของ kiosk เพื่อให้ dev จับคู่ 1:1 · `text-kiosk-*` ตั้งเฉพาะ font-size (line-height ยังกำหนดต่อ element)

### 8.3. Components (bma.kiosk `src/design-system/`)

`PrimaryButton` · `OutlineButton` · `OutlineSubduedButton` · `Card` · `Keypad` / `NumberKey` · ปุ่มรวม `kioskButtonStyles.ts` · หน้าดูรวม `/Kiosk/design-system`
Shell: `KioskShell` · `Header` · `IdNumberDisplay` · `PrintingScreen` · `LottieAsset` (ไฟล์ Lottie/ภาพจาก `@mih/assets`)

### 8.4. Figma reference

Node ที่โค้ดอ้าง: outline Subdued `268:14108` · Card `237:9258` · `51:14038` / `51:14039` · `249:13756` · ไฟล์ Figma ของ kiosk: `AGes4Mc8odyTEYARFeFqcj` (https://www.figma.com/design/AGes4Mc8odyTEYARFeFqcj/Kiosk) · PHCIS|KIOSK เดิมในไฟล์เว็บ `4002:928` (`--dim-radius-450`)

---

### 8.0. กติกาการทำงาน (บังคับทุกงาน Kiosk UI)

- ทุกค่าสี · ขนาดตัวอักษร · รัศมี · เงา ต้องอ้าง `--kiosk-*` (หรือ `--dim-*` ของ foundation) ผ่าน class ของ Tailwind เท่านั้น
- ค่าที่ยังเขียนดิบอยู่ในโค้ดปัจจุบันถือเป็น **drift** ดู §8.9 — งานใหม่ห้ามเพิ่ม · งานที่แตะต้องแก้ตาม §8.9 ในงานเดียวกัน
- Kiosk ไม่มีเมาส์ · ไม่มีคีย์บอร์ดจริง · ไม่มี hover สำหรับการสัมผัส — สถานะกดใช้ `:active` เสมอ
- ก่อนปิดงาน UI ของ Kiosk: เปิดหน้า `/Kiosk/design-system` ดูจริงที่ความกว้าง 1080 (หน้าจอจริง) และเช็กว่าไม่มีส่วนล้นกรอบ

### 8.5. Canvas และ layout

- ผืนผ้าใบเดียว **1080 × 1920 แนวตั้ง** (ไม่มี breakpoint อื่น — ไม่ต้องทำ responsive ตามความกว้างเบราว์เซอร์)
- คอลัมน์เนื้อหาหลักกว้างสูงสุด **1560px** (อ้างจาก `Keypad.tsx` — คีย์บอร์ด 10 ปุ่มต้องพอดีในคอลัมน์นี้)
- ระยะซ้าย-ขวาของแถวการ์ด `px-[76px]` (อ้างจาก `Card.tsx` ตัวอย่าง Service Selection Grid)
- ระยะระหว่างปุ่ม/คีย์ ใช้ `gap-[24px]` (ตัวเลข) · `gap-[16px]` / `gap-[12px]` (ตัวอักษร) — ยังไม่ได้เป็น token
- พื้นหลังหน้าปกติไล่สีจาก `--kiosk-mint-50 → 100 → 200` · หน้าผิดพลาดไล่สีจาก `--kiosk-pink-50 → 100 → 200`

### 8.6. Component specs

ทุกชิ้นอยู่ใน `src/design-system/` และ export ผ่าน `index.ts` เท่านั้น — ห้าม deep-import จากหน้า

#### 8.6.1. PrimaryButton — ปุ่ม CTA หลัก (Figma 166:39216)

| หัวข้อ | ค่า |
|---|---|
| ใช้ทำอะไร | การกระทำหลักของหน้าเดียว เช่น「ยืนยัน」「ถ่ายภาพ」 — **หนึ่งปุ่ม Fill ต่อหน้า** |
| ขนาด | สูง `160px` · ความกว้างกำหนดโดยผู้เรียก (Figma: 400px · 480px หรือกว้างตามข้อความ) |
| รูป | radius `24px` · ตัวอักษร `64px` medium สีขาว · กึ่งกลาง |
| สถานะ | Default `bg-kiosk-blue` + `shadow-kiosk-card` · Pressed `:active` → `bg-kiosk-blue-pressed` · Disabled `bg-kiosk-disabled` ไม่มีเงา |
| ไฟล์ | `PrimaryButton.tsx` · class พื้นฐานใน `kioskButtonStyles.ts` |

#### 8.6.2. OutlineButton — ปุ่มรอง (Figma 181:43393)

| หัวข้อ | ค่า |
|---|---|
| ใช้ทำอะไร | ทางเลือกรองที่ไม่ใช่การยืนยัน เช่น「ถ่ายใหม่」 |
| ขนาด | สูง `160px` · เส้นขอบ `4px` · radius `24px` · ตัวอักษร `64px` medium |
| สถานะ | Default ขอบ/ตัวอักษร `--kiosk-blue` · Pressed → พื้นและขอบ `--kiosk-blue-subdued` ตัวอักษรขาว · Disabled ขอบและตัวอักษร `--kiosk-disabled` |
| ไฟล์ | `OutlineButton.tsx` · class พื้นฐานใน `kioskButtonStyles.ts` |

#### 8.6.3. OutlineSubduedButton — ปุ่มยกเลิก (Figma 268:14108)

- ใช้กับ**ทุกปุ่มยกเลิก** (「ยกเลิก」) ตามคอมเมนต์ในโค้ด
- ขอบ `--kiosk-blue-subdued` (50%) · ตัวอักษร `--kiosk-blue` · Pressed เติมพื้น `--kiosk-blue-subdued` + ตัวอักษรขาว · Disabled เหมือน OutlineButton
- ไฟล์ `OutlineSubduedButton.tsx`

> หมายเหตุ: โค้ดระบุ node Figma ของ Outline Subdued สองค่า (`268:14108` ใน comment ของ OutlineSubduedButton · `181:43393` ใน kioskButtonStyles สำหรับ Outline ทั่วไป) ต้องยืนยันกับ Figma ก่อนอ้างอิง

#### 8.6.4. Card — การ์ดเลือกตัวเลือก (Figma 151:36499)

| หัวข้อ | ค่า |
|---|---|
| ใช้ทำอะไร | ตัวเลือกหลักที่แตะได้ทั้งการ์ด เช่น เลือกบริการ · เลือกวิธีชำระ |
| โครง | ไอคอน (กล่อง 96 × 112) → หัวข้อ `58px` semibold `--kiosk-blue` → คำอธิบาย (ถ้ามี) `36px` |
| ขนาด | ค่าเริ่มต้น **กว้าง 768px** · `fluid` = ยืดเต็มแถว (`flex-1`) |
| รูป | padding `32px × 56px` · radius `24px` · เงา `--kiosk-shadow-card` |
| สถานะ | Default · `:hover` / `:active` / `:focus-visible` เงาเข้มขึ้น (`--kiosk-shadow-card-hover`) · `:focus-visible` มี outline `2px` สี `--kiosk-blue` |
| ไฟล์ | `Card.tsx` |

#### 8.6.5. NumberKey · NumericKeypad · AlphanumericKeyboard — คีย์บอร์ดบนจอ (Figma 172:42119)

| ชิ้น | ขนาดปุ่ม | ใช้ทำอะไร |
|---|---|---|
| `NumberKey` size `default` | 240 × 160 · ตัวอักษร `56px` bold | คีย์ตัวเลขเลขบัตรประชาชน 13 หลัก |
| `NumberKey` size `compact` | 140 × 124 · ตัวอักษร `44px` | คีย์ของ AlphanumericKeyboard (เลขหนังสือเดินทาง) |
| `NumericKeypad` | 3 แถว 1–9 + แถวล่าง [ว่าง] · 0 · ลบ | กรอกเลขบัตร — แถวห่าง `24px` |
| `AlphanumericKeyboard` | 4 แถว: 1234567890 · QWERTYUIOP · ASDFGHJKL · ZXCVBNM + ปุ่มลบกว้าง `292px` ท้ายแถวสุดท้าย | กรอกหนังสือเดินทาง — แถวห่าง `16px` |

- สถานะกด = `:active` เปลี่ยนเงาเข้มขึ้น (ไม่มี hover)
- ทุกปุ่มมี `aria-label` (ตัวอักษรหรือ「ลบ / Delete」)
- ไฟล์ `NumberKey.tsx` · `Keypad.tsx` · ไอคอนลบ `components/icons/DeleteIcon`

### 8.7. การสัมผัสและการเข้าถึง

- เป้าสัมผัสขั้นต่ำที่โค้ดใช้จริง: ปุ่มและคีย์สูง **124px ขึ้นไป** (ปุ่มเต็ม 160px) — ใหญ่กว่า 44px ของเว็บมาก
- ไม่มี hover เป็นตัวบอกสถานะ — ทุกสถานะที่ผู้ใช้ต้องเห็นต้องมีจากการกด (`:active`) หรือการเปลี่ยนค่า
- ข้อความบนปุ่มและการ์ดต้องอ่านจากระยะยืนได้ (ตัวอักษรขั้นต่ำ `14px` ใช้เฉพาะใบพิมพ์ — §8.2)
- ยังไม่ได้วัด contrast ของคู่สีทั้งหมดในระบบนี้ — ต้องตรวจก่อนอ้างว่าผ่าน WCAG

### 8.8. ไอคอนและภาพ

- ไอคอนใช้ **Figma ก่อน** (เช่น `insert-id-card/delete.svg` สำหรับปุ่มลบ) แล้วค่อย Lucide ตามกติกาไอคอนของ Mih — ไม่วาง SVG ดิบซ้ำในคอมโพเนนต์
- Lottie และภาพจาก `@mih/assets` เท่านั้น (ดู `LottieAsset`)
- ไม่ใช้ไอคอนผสมหลายตระกูลในหน้าเดียว

### 8.9. Drift ที่ต้องแก้ (ค่าดิบในโค้ดปัจจุบัน)

| ไฟล์ | ค่าดิบ | ควรเป็น |
|---|---|---|
| `kioskButtonStyles.ts` | `h-[160px]` `rounded-[24px]` `text-[64px]` `border-4` | ตั้ง token ขนาดสำหรับ CTA ใน mih-design แล้วอ้างผ่าน class |
| `kioskButtonStyles.ts` (numberKeyBase) | `shadow-[0px_12px_20px_rgba(5,150,105,0.08)]` · `border-[rgba(188,202,192,0.1)]` · `rounded-[16px]` | เงาและขอบของคีย์ควรเป็น token ของ kiosk (ยังไม่มี) |
| `NumberKey.tsx` | `text-[44px]` · `h-[124px] w-[140px]` | token type-size และขนาดคีย์ compact |
| `Card.tsx` | `text-[#457499]` · `w-[768px]` · `px-[32px] py-[56px]` | สีคำอธิบายเป็น token (`--kiosk-blue-muted` ใกล้เคียงแต่ไม่เท่ากัน) · ที่เหลือเป็น token ขนาด |
| `Keypad.tsx` | `gap-[24px]` `gap-[16px]` `gap-[12px]` `w-[292px]` | token ระยะห่างสำหรับ kiosk |

ข้อ drift เหล่านี้ไม่ได้ทำให้หน้าเพี้ยนในตอนนี้ แต่ทำให้เปลี่ยนค่าจาก Figma ทีหลังยาก

### 8.10. ส่วนที่ยังไม่มีเทียบกับ design.md ของเว็บ

- ยังไม่มี **states matrix** ครบทุกคอมโพเนนต์ (เช่น loading · error · empty ของการ์ด/คีย์)
- ยังไม่มี **สีแบบ error / warning** ที่ใช้จริงบนหน้าจอ (มีแค่พื้นหลังไล่สีชมพู)
- ยังไม่มี **Figma node** ของ `Header` · `KioskShell` · `IdNumberDisplay` · `PrintingScreen` ในเอกสาร
- ยังไม่มี **สเปกใบพิมพ์** (ใบคิว · ใบเสร็จ) นอกจากชื่อ token ขนาด 14–20px
- ยังไม่มี **การเข้าถึง** ที่วัดแล้ว (contrast · ลำดับโฟกัส · ตัวอ่านหน้าจอ) — §8.7 เป็นสิ่งที่โค้ดทำอยู่ ไม่ใช่ผลการตรวจ
- ยังไม่มี **แบบฟอร์มและการเลือก** (ช่องกรอก · checkbox · radio) เพราะยังไม่มีในชุดคอมโพเนนต์ของ kiosk

### 8.11. ความสัมพันธ์กับระบบอื่น (จาก reference หน้า HCIS เดิม — ส่วน Kiosk)

ที่มา: หัวข้อ "Mobile App · Kiosk — design system แยกของตัวเอง" ของหน้า reference เดิม (`bma-design-system.mih.co.th/reference.html`) เฉพาะแถวและกติกาของ Kiosk

| ระบบ | เอกสาร | Token | Figma |
|---|---|---|---|
| Kiosk | bma.kiosk `design.md` (ไฟล์นี้) | `kiosk.css` (`--kiosk-*` · ไฟล์เดียวจบ) | `AGes4Mc8odyTEYARFeFqcj` |

กติกาที่ใช้กับ Kiosk:
- HCIS (เว็บ) ห้ามใช้ `--kiosk-*` · Kiosk ไม่เพิ่ม token ของตัวเองลงใน `tokens.css` ของเว็บ
- token ของ Kiosk แก้ที่ mih-design `packages/design-tokens/kiosk.css` เท่านั้น แล้วดึงกลับด้วย `npm run tokens:pull` ตามที่ §8.1 ระบุ
- การถ่ายรูปยืนยันตัวตนแบบ 360 องศา (ท่าศีรษะ 4 ท่า แบบ Face ID) ที่หน้าลงทะเบียนของ HCIS อ้างถึง bma.kiosk เป็นต้นแบบ · ถ้าแก้ท่าทางหรือกรอบใน Kiosk ให้ตรวจหน้า HCIS ที่ใช้ร่วมด้วย
