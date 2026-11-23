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
