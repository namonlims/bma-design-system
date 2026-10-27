# Mobile App Design System (bma.mobile)

> **Design system ของ Mobile App แยกจาก HCIS** (ผู้ใช้ 24 ต.ค. 69) — ไฟล์นี้คือต้นทาง แก้ที่ repo นี้ได้เลย (ไม่ใช่ mirror)
> **ส่ง dev ขึ้นแอปจริงได้ทั้งไฟล์ — ไม่ต้องเปิด design.md ของ HCIS** · token ทุกตัวที่แอปใช้ + ค่าที่แสดงจริง อยู่ใน §7.0 (สร้างจากโค้ดด้วย `npm run design:tokens`)
> Token: mih-design `packages/design-tokens/mobile-foundation.css` (เฉพาะ foundation ที่แอปใช้ · สร้างด้วย `npm run mobile:foundation` ใน mih-design) + `mobile.css` (`--mobile-*`) → `npm run tokens:pull` → `src/styles/tokens.css` + `src/styles/mobile-tokens.css`
> Figma: New Theme `XCuT5qG9E7V7Aa815X3usM` · app DS `opz7X9AAKGQf5prHhsPDGn`
> หัวข้อยังใช้เลข §7.x เดิม เพื่อให้ที่โค้ดอ้างไว้ยังถูก

## 7. Mobile Application Design System

The mobile citizen app (`/m/*` routes, see [src/features/](src/features/)) has its **own design system, fully separate from the HCIS web** (ผู้ใช้ 26 ต.ค. 69) — nothing is shared or read from the HCIS `design.md`. Foundation tokens (colour, typography family, icon size, motion) = mih-design `mobile-foundation.css`, full list + rendered values in §7.0. **Shape, height, and density** come from `--mobile-*` (§7.2), because thumb reach and one-handed use are the dominant constraints on a 390-wide viewport.

The mobile system is intentionally lean: ~7 primitives that compose every screen. There is no mobile `Table`, `Tabs`, `Accordion`, or `Dropdown` yet — when those land, they extend this layer, not duplicate it.

<!-- tokens:start (npm run design:tokens — ห้ามแก้มือ) -->
### 7.0. Token reference — ทุก token ที่แอปใช้ (สร้างอัตโนมัติ)

> สำหรับ dev ที่ขึ้นแอปจริง: ใช้ได้เฉพาะ token ในตารางนี้ · 594 ตัว = `src/styles/tokens.css` (foundation เฉพาะที่แอปใช้ · mih-design `mobile-foundation.css`) + `src/styles/mobile-tokens.css` (`--mobile-*` + ค่าที่ Mobile ทับ)
> **สว่าง** = ค่าที่แอปแสดงจริง (`<html data-app="mobile">`) · **มืด** = `<html data-mode="dark">` (ว่าง = เหมือนสว่าง) · **อ้างถึง** = ค่าที่เขียนในไฟล์ (ใช้ชื่อนี้ใน Figma variable)
> ใช้ในโค้ด: `var(--token, <ค่าสว่าง>)` — fallback ต้องเท่าค่าสว่าง · ค่าที่ไม่มีในตาราง = เพิ่มที่ mih-design ก่อน (`mobile.css`) แล้ว `npm run tokens:pull` · ตารางนี้สร้างใหม่ด้วย `npm run design:tokens`

#### Brand primitives (15)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--brand-p100` | `#E2F3EB` |  |  |
| `--brand-p200` | `#C5EDDA` |  |  |
| `--brand-p300` | `#91DFBB` |  |  |
| `--brand-p400` | `#6AD6A5` |  |  |
| `--brand-p50` | `#F4FBF8` |  |  |
| `--brand-p500` | `#54CF97` |  |  |
| `--brand-p600` | `#08A768` |  |  |
| `--brand-p600-rgb` | `8 167 104` |  |  |
| `--brand-p700` | `#007549` |  |  |
| `--brand-p800` | `#004C31` |  |  |
| `--brand-p900` | `#003322` |  |  |
| `--brand-p950` | `#002318` |  |  |
| `--brand-s100` | `#EFF5EF` |  |  |
| `--brand-s200` | `#E1F0E0` |  |  |
| `--brand-s700` | `#366233` |  |  |

#### Color primitives (9)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--color-bg-sky-cyan-400` | `rgb(64 170 242 / 0.20)` |  | `rgb(var(--color-bg-sky-cyan-400-rgb) / 0.20)` |
| `--color-bg-sky-cyan-400-rgb` | `64 170 242` |  |  |
| `--color-bg-sky-cyan-600` | `rgb(10 115 203 / 0.20)` |  | `rgb(var(--color-bg-sky-cyan-600-rgb) / 0.20)` |
| `--color-bg-sky-cyan-600-rgb` | `10 115 203` |  |  |
| `--color-bg-teal-400` | `rgb(62 195 178 / 0.20)` |  | `rgb(var(--color-bg-teal-400-rgb) / 0.20)` |
| `--color-bg-teal-400-rgb` | `62 195 178` |  |  |
| `--color-bg-teal-600` | `rgb(27 134 125 / 0.20)` |  | `rgb(var(--color-bg-teal-600-rgb) / 0.20)` |
| `--color-bg-teal-600-rgb` | `27 134 125` |  |  |
| `--color-sky-cyan-700` | `#0859A8` |  |  |

#### Dimension — space (13)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--dim-space-0` | `0px` |  |  |
| `--dim-space-050` | `2px` |  |  |
| `--dim-space-100` | `4px` |  |  |
| `--dim-space-1200` | `48px` |  |  |
| `--dim-space-150` | `6px` |  |  |
| `--dim-space-200` | `8px` |  |  |
| `--dim-space-2400` | `96px` |  |  |
| `--dim-space-300` | `12px` |  |  |
| `--dim-space-400` | `16px` |  |  |
| `--dim-space-500` | `20px` |  |  |
| `--dim-space-600` | `24px` |  |  |
| `--dim-space-700` | `28px` |  |  |
| `--dim-space-800` | `32px` |  |  |

#### Dimension — size (35)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--dim-size-100` | `8px` |  |  |
| `--dim-size-1000` | `48px` |  |  |
| `--dim-size-1050` | `50px` |  |  |
| `--dim-size-1100` | `52px` |  |  |
| `--dim-size-1200` | `56px` |  |  |
| `--dim-size-1300` | `60px` |  |  |
| `--dim-size-1400` | `64px` |  |  |
| `--dim-size-1500` | `68px` |  |  |
| `--dim-size-1600` | `72px` |  |  |
| `--dim-size-1700` | `80px` |  |  |
| `--dim-size-1800` | `96px` |  |  |
| `--dim-size-1900` | `112px` |  |  |
| `--dim-size-200` | `12px` |  |  |
| `--dim-size-2000` | `128px` |  |  |
| `--dim-size-2050` | `144px` |  |  |
| `--dim-size-2100` | `160px` |  |  |
| `--dim-size-2170` | `177px` |  |  |
| `--dim-size-2190` | `190px` |  |  |
| `--dim-size-2200` | `192px` |  |  |
| `--dim-size-2280` | `218px` |  |  |
| `--dim-size-2350` | `240px` |  |  |
| `--dim-size-2450` | `280px` |  |  |
| `--dim-size-250` | `14px` |  |  |
| `--dim-size-2500` | `320px` |  |  |
| `--dim-size-2600` | `384px` |  |  |
| `--dim-size-2610` | `402px` |  |  |
| `--dim-size-2700` | `448px` |  |  |
| `--dim-size-2900` | `640px` |  |  |
| `--dim-size-300` | `16px` |  |  |
| `--dim-size-400` | `20px` |  |  |
| `--dim-size-500` | `24px` |  |  |
| `--dim-size-600` | `32px` |  |  |
| `--dim-size-700` | `36px` |  |  |
| `--dim-size-800` | `40px` |  |  |
| `--dim-size-900` | `44px` |  |  |

#### Dimension — radius · stroke · blur (18)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--dim-radius-100` | `4px` |  |  |
| `--dim-radius-1000` | `100px` |  |  |
| `--dim-radius-150` | `6px` |  |  |
| `--dim-radius-200` | `8px` |  |  |
| `--dim-radius-300` | `10px` |  |  |
| `--dim-radius-400` | `12px` |  |  |
| `--dim-radius-450` | `14px` |  |  |
| `--dim-radius-500` | `16px` |  |  |
| `--dim-radius-550` | `22px` |  |  |
| `--dim-radius-600` | `24px` |  |  |
| `--dim-radius-700` | `38px` |  |  |
| `--dim-radius-full` | `9999px` |  |  |
| `--dim-stroke-100` | `1px` |  |  |
| `--dim-stroke-150` | `1.5px` |  |  |
| `--dim-stroke-200` | `2px` |  |  |
| `--dim-stroke-400` | `4px` |  |  |
| `--dim-stroke-500` | `5px` |  |  |
| `--dim-stroke-800` | `8px` |  |  |

#### Typography (20)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--type-family-body` | `'Sarabun', system-ui, -apple-system, BlinkMacSystemFont, sans-serif` |  |  |
| `--type-family-display` | `'Sao Chingcha', 'Sarabun', system-ui, sans-serif` |  | `var(--type-family-heading-graphic)` |
| `--type-family-heading-content` | `'Sarabun', system-ui, -apple-system, BlinkMacSystemFont, sans-serif` |  |  |
| `--type-family-heading-graphic` | `'Sao Chingcha', 'Sarabun', system-ui, sans-serif` |  |  |
| `--type-family-heading-graphic2` | `'Anuphan', 'Sarabun', system-ui, sans-serif` |  |  |
| `--type-family-mono` | `ui-monospace, SFMono-Regular, Menlo, monospace` |  |  |
| `--type-size-2xl` | `24px` |  |  |
| `--type-size-3xl` | `30px` |  |  |
| `--type-size-4xl` | `36px` |  |  |
| `--type-size-5xl` | `48px` |  |  |
| `--type-size-6xl` | `60px` |  |  |
| `--type-size-base` | `16px` |  |  |
| `--type-size-lg` | `18px` |  |  |
| `--type-size-sm` | `14px` |  |  |
| `--type-size-xl` | `20px` |  |  |
| `--type-size-xs` | `12px` |  |  |
| `--type-size-xxs` | `10px` |  |  |
| `--type-weight-bold` | `700` |  |  |
| `--type-weight-medium` | `500` |  |  |
| `--type-weight-regular` | `400` |  |  |

#### Shadow · effect (13)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--shadow-brand-drop-bottom-100` | `0 1px 4px rgb(8 167 104 / 0.08)` |  | `0 1px 4px var(--PrimaryShadow-600)` |
| `--shadow-brand-drop-bottom-200` | `inset 0 0 0 1px #C5EDDA` | `inset 0 0 0 1px #363B3F` | `inset 0 0 0 1px var(--mobile-theme-border-card)` |
| `--shadow-brand-drop-bottom-400` | `0 4px 4px -4px rgb(8 167 104 / 0.08), 0 16px 32px -4px rgb(8 167 104 / 0.10)` |  | `0 4px 4px -4px var(--PrimaryShadow-600), 0 16px 32px -4px var(--PrimaryShadow-601)` |
| `--shadow-brand-drop-top-100` | `0 -1px 4px rgb(8 167 104 / 0.08)` |  | `0 -1px 4px var(--PrimaryShadow-600)` |
| `--shadow-brand-top-200` | `0px -1px 4px rgb(8 167 104 / 0.08), 0px -1px 8px rgb(8 167 104 / 0.10)` |  | `0px -1px 4px var(--PrimaryShadow-600), 0px -1px 8px var(--PrimaryShadow-601)` |
| `--shadow-card` | `inset 0 0 0 1px #C5EDDA` | `inset 0 0 0 1px #363B3F` | `inset 0 0 0 1px var(--mobile-theme-border-card)` |
| `--shadow-card-hover` | `0px 1px 4px rgb(8 167 104 / 0.08), 0px 1px 8px rgb(8 167 104 / 0.10)` |  | `0px 1px 4px var(--PrimaryShadow-600), 0px 1px 8px var(--PrimaryShadow-601)` |
| `--shadow-drop-bottom-100` | `0 1px 4px rgb(100 116 139 / 0.10)` |  |  |
| `--shadow-drop-bottom-200` | `0 1px 4px rgba(100, 116, 139, 0.15), 0 1px 2px rgba(100, 116, 139, 0.10)` |  | `var(--shadow-smallalert-error)` |
| `--shadow-drop-bottom-400` | `0 4px 4px rgb(100 116 139 / 0.10), 0 16px 32px rgb(100 116 139 / 0.15)` |  |  |
| `--shadow-header` | `0 1px 4px rgb(8 167 104 / 0.08)` |  | `var(--shadow-brand-drop-bottom-100)` |
| `--shadow-primarysearch` | `0 1px 4px rgb(8 167 104 / 0.08), 0 1px 8px rgb(8 167 104 / 0.10)` |  | `0 1px 4px var(--PrimaryShadow-600), 0 1px 8px var(--PrimaryShadow-601)` |
| `--shadow-smallalert-error` | `0 1px 4px rgba(100, 116, 139, 0.15), 0 1px 2px rgba(100, 116, 139, 0.10)` |  |  |

#### Motion (9)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--motion-duration-calendar` | `320ms` |  |  |
| `--motion-duration-modal` | `220ms` |  |  |
| `--motion-duration-popover` | `160ms` |  |  |
| `--motion-easing-enter` | `cubic-bezier(0.16, 1, 0.3, 1)` |  |  |
| `--motion-easing-popover` | `ease-out` |  |  |
| `--motion-easing-standard` | `cubic-bezier(0.4, 0, 0.2, 1)` |  |  |
| `--motion-offset-calendar-x` | `24px` |  | `var(--dim-space-600, 24px)` |
| `--motion-panel-offset-y` | `8px` |  | `var(--dim-space-200, 8px)` |
| `--motion-panel-scale-from` | `0.98` |  |  |

#### Background (7)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--background-bg-blue-400` | `rgb(64 170 242 / 0.20)` | `rgb(10 115 203 / 0.20)` | `var(--color-bg-sky-cyan-400)` |
| `--background-bg-teal-400` | `rgb(62 195 178 / 0.20)` | `rgb(27 134 125 / 0.20)` | `var(--color-bg-teal-400)` |
| `--background-danger-tertiary-hover` | `#FECACA` | `#991B1B` |  |
| `--background-info-tertiary-hover` | `#BFDBFE` | `#1E40AF` |  |
| `--background-positive-tertiary-hover` | `#C5EDDA` | `#003322` | `var(--brand-p200)` |
| `--background-screen-brand-100` | `#FFFFFF` | `#222629` | `var(--surface-card-100)` |
| `--background-warning-tertiary-hover` | `#FED7AA` | `#9A3412` |  |

#### Surface (114)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--surface-alert-error` | `#FFF0F0` | `#450A0A` |  |
| `--surface-alert-error-default` | `#DC2626` |  |  |
| `--surface-alert-info` | `#F0F6FF` | `#172554` |  |
| `--surface-alert-success` | `#F4FBFA` | `#082B2A` |  |
| `--surface-alert-warning` | `#FFF9F0` | `#431407` |  |
| `--surface-alert-warning-default` | `#EA580C` |  |  |
| `--surface-alertdialog-default` | `#FFFFFF` | `#222629` | `var(--surface-card-100)` |
| `--surface-brandPrimary-default` | `#007549` |  | `var(--brand-p700)` |
| `--surface-brandPrimary-quaternary` | `#F4FBF8` | `#002318` | `var(--brand-p50)` |
| `--surface-brandPrimary-quaternary-hover` | `#E2F3EB` | `#002318` | `var(--brand-p100)` |
| `--surface-brandPrimary-secondary` | `#C5EDDA` | `#003322` | `var(--brand-p200)` |
| `--surface-brandPrimary-tertiary` | `#E2F3EB` | `#003322` | `var(--brand-p100)` |
| `--surface-brandPrimaryButton-tertiary` | `#E2F3EB` | `#002318` | `var(--brand-p100)` |
| `--surface-brandtag-default` | `#FFFFFF` | `#363B3F` |  |
| `--surface-brandtag-selected` | `#F4FBF8` | `#4D5358` | `var(--brand-p50)` |
| `--surface-calendar-current` | `#E2F3EB` | `#002318` | `var(--brand-p100)` |
| `--surface-calendar-selected-mobile` | `#08A768` |  | `var(--brand-p600)` |
| `--surface-camera-viewport` | `#111827` |  |  |
| `--surface-card-100` | `#FFFFFF` | `#222629` |  |
| `--surface-card-200` | `#FDFDFD` | `#363B3F` |  |
| `--surface-card-brand-100` | `#F4FBF8` | `#222629` | `var(--brand-p50)` |
| `--surface-card-brand100` | `#F4FBF8` | `#003322` | `var(--brand-p50)` |
| `--surface-card-skycyan-100` | `#F4F9FD` | `#081E40` | `var(--sky-cyan-50)` |
| `--surface-colorfulbadge-blue` | `#DBEAFE` | `#1E3A8A` |  |
| `--surface-colorfulbadge-blue-bold` | `#3B82F6` |  |  |
| `--surface-colorfulbadge-green` | `#E2F3EB` | `#002318` | `var(--brand-p100)` |
| `--surface-colorfulbadge-green-bold` | `#54CF97` |  |  |
| `--surface-colorfulbadge-grey` | `#F5F7F9` | `#363B3F` |  |
| `--surface-colorfulbadge-grey-bold` | `#858C92` |  |  |
| `--surface-colorfulbadge-indigo` | `#EEF2FF` | `#3730A3` |  |
| `--surface-colorfulbadge-indigo-bold` | `#818CF8` |  |  |
| `--surface-colorfulbadge-magenta` | `#FAE8FF` |  |  |
| `--surface-colorfulbadge-magenta-bold` | `#C926E0` |  |  |
| `--surface-colorfulbadge-orange` | `#FFEDD5` | `#7C2D12` |  |
| `--surface-colorfulbadge-orange-bold` | `#F97316` |  |  |
| `--surface-colorfulbadge-pink` | `#FCE7F3` | `#831843` |  |
| `--surface-colorfulbadge-pink-bold` | `#EC4899` |  |  |
| `--surface-colorfulbadge-purple` | `#F3E8FF` | `#581C87` |  |
| `--surface-colorfulbadge-purple-bold` | `#A855F7` |  |  |
| `--surface-colorfulbadge-red` | `#FEE2E2` | `#7F1D1D` |  |
| `--surface-colorfulbadge-red-bold` | `#EF4444` |  |  |
| `--surface-colorfulbadge-teal` | `#D1F6EE` | `#194845` |  |
| `--surface-colorfulbadge-teal-bold` | `#24A899` |  |  |
| `--surface-colorfulbadge-turquoise` | `#D1F6EE` | `#194845` | `var(--surface-colorfulbadge-teal)` |
| `--surface-colorfulbadge-turquoise-bold` | `#24A899` |  | `var(--surface-colorfulbadge-teal-bold)` |
| `--surface-colorfulbadge-yellow` | `#FEF9C3` | `#713F12` |  |
| `--surface-colorfulbadge-yellow-bold` | `#EAB308` |  |  |
| `--surface-content-default` | `#FFFFFF` | `#222629` |  |
| `--surface-content-secondary` | `#F5F6F7` | `#111314` |  |
| `--surface-danger-default` | `#DC2626` | `#F87171` |  |
| `--surface-danger-quaternary` | `#FFF0F0` | `#450A0A` | `var(--surface-dangerButton-quaternary)` |
| `--surface-danger-tertiary` | `#FEE2E2` | `#7F1D1D` |  |
| `--surface-dangerButton-quaternary` | `#FFF0F0` | `#450A0A` |  |
| `--surface-disabled-secondary-hover` | `#C9CDD0` | `#4D5358` |  |
| `--surface-disabled-tertiary` | `#F4F5F5` | `#363B3F` | `var(--surface-input-disabled)` |
| `--surface-disabledButton-tertiary` | `#F4F5F5` | `#4D5358` |  |
| `--surface-info-default` | `#3B82F6` |  |  |
| `--surface-input-default` | `#FFFFFF` | `#363B3F` |  |
| `--surface-input-disabled` | `#F4F5F5` | `#363B3F` |  |
| `--surface-list-default` | `#FFFFFF` | `#363B3F` |  |
| `--surface-list-hover` | `#F4FBF8` | `#363B3F` | `var(--brand-p50)` |
| `--surface-navigation-default` | `#FFFFFF` | `#222629` |  |
| `--surface-neutral-primary` | `#FFFFFF` | `#222629` |  |
| `--surface-neutral-quaternary` | `#F9FBFB` | `#222629` |  |
| `--surface-neutral-tertiary` | `#F4F5F5` | `#222629` |  |
| `--surface-neutralButton-default` | `#4D5358` |  |  |
| `--surface-neutralbutton-quinary` | `#FFFFFF` | `#222629` |  |
| `--surface-overlay-default` | `rgb(0 0 0 / 0.50)` |  | `var(--mobile-theme-overlay)` |
| `--surface-pill-blue` | `#DBEAFE` | `#1E3A8A` |  |
| `--surface-pill-orange` | `#FFEDD5` | `#7C2D12` |  |
| `--surface-pill-red` | `#FEE2E2` | `#7F1D1D` |  |
| `--surface-pill-teal` | `#D1F6EE` | `#194845` |  |
| `--surface-pill-yellow` | `#FEF9C3` | `#713F12` |  |
| `--surface-pink-quaternary` | `#FBF4F8` | `#500724` |  |
| `--surface-positive-default` | `#196C65` | `#6CDCCA` |  |
| `--surface-positive-tertiary` | `#D1F6EE` | `#194845` |  |
| `--surface-quickaction-blue-pressed` | `#BFDBFE` | `#1E40AF` |  |
| `--surface-quickaction-green-pressed` | `#C5EDDA` | `#003322` | `var(--brand-p200)` |
| `--surface-quickaction-indigo-pressed` | `#C7D2FE` | `#4F46E5` |  |
| `--surface-quickaction-orange-pressed` | `#FED7AA` | `#9A3412` |  |
| `--surface-quickaction-pink-pressed` | `#FBCFE8` | `#9D174D` |  |
| `--surface-quickaction-purple-pressed` | `#E9D5FF` | `#6B21A8` |  |
| `--surface-quickaction-teal-pressed` | `#A2EDDE` | `#195652` |  |
| `--surface-quickaction-yellow-pressed` | `#FEF08A` | `#854D0E` |  |
| `--surface-scrollbar-error-bar` | `#FEE2E2` | `#3A1C1C` |  |
| `--surface-scrollbar-error-pill` | `#FCA5A5` | `#B91C1C` |  |
| `--surface-scrollbar-info-bar` | `#DBEAFE` | `#14263D` |  |
| `--surface-scrollbar-info-pill` | `#93C5FD` | `#1D4ED8` |  |
| `--surface-scrollbar-success-bar` | `#D1F6EE` | `#0F2E2C` |  |
| `--surface-scrollbar-success-pill` | `#6CDCCA` | `#196C65` |  |
| `--surface-scrollbar-warning-bar` | `#FFEDD5` | `#33220F` |  |
| `--surface-scrollbar-warning-pill` | `#FDBA74` | `#C2410C` |  |
| `--surface-search-default` | `#FFFFFF` | `#363B3F` |  |
| `--surface-statusbadge-error` | `#FEE2E2` | `#3A1C1C` |  |
| `--surface-statusbadge-info` | `#DBEAFE` | `#14263D` |  |
| `--surface-statusbadge-neutral` | `#F5F7F9` | `#4D5358` |  |
| `--surface-statusbadge-success` | `#D1F6EE` | `#0F2E2C` |  |
| `--surface-statusbadge-warning` | `#FFEDD5` | `#33220F` |  |
| `--surface-statusbadge-yellow` | `#FEF9C3` | `#713F12` | `var(--surface-pill-yellow)` |
| `--surface-switch-checkbox` | `#08A768` |  | `var(--brand-p600)` |
| `--surface-switch-checkbox-default-hover` | `#54CF97` |  | `var(--brand-p500)` |
| `--surface-switch-checkbox-hover` | `#007549` |  | `var(--brand-p700)` |
| `--surface-switch-default` | `#FFFFFF` | `#222629` |  |
| `--surface-switch-knob-disabled` | `#FFFFFF` | `#222629` |  |
| `--surface-switch-on` | `#08A768` |  | `var(--brand-p600)` |
| `--surface-switch-unchecked-default` | `#E2E4E6` | `#363B3F` |  |
| `--surface-switch-unchecked-disabled` | `#F4F5F5` | `#222629` |  |
| `--surface-switch-unchecked-hover` | `#C9CDD0` | `#4D5358` |  |
| `--surface-table-default` | `#FFFFFF` | `#222629` |  |
| `--surface-table-header` | `#F4FBF8` | `#003322` | `var(--brand-p50)` |
| `--surface-table-hover` | `#F4FBF8` | `#003322` | `var(--brand-p50)` |
| `--surface-topic-default` | `#FFFFFF` | `#222629` |  |
| `--surface-topic-hover` | `#F4FBF8` | `#002318` | `var(--brand-p50, #F4FBF8)` |
| `--surface-warning-quaternary` | `#FFF9F0` | `#431407` |  |

#### Text (77)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--text-alert-error` | `#B91C1C` | `#FCA5A5` |  |
| `--text-alert-info` | `#1D4ED8` | `#93C5FD` |  |
| `--text-alert-success` | `#196C65` | `#6CDCCA` |  |
| `--text-alert-warning` | `#C2410C` | `#FDBA74` |  |
| `--text-brandPrimary-default` | `#007549` | `#6AD6A5` | `var(--brand-p700)` |
| `--text-brandPrimary-on-brand` | `#FFFFFF` | `#111314` |  |
| `--text-brandPrimary-secondary` | `#08A768` | `#6AD6A5` | `var(--brand-p600)` |
| `--text-brandtag-default` | `#4D5358` | `#F4F5F5` |  |
| `--text-brandtag-selected` | `#007549` | `#6AD6A5` | `var(--brand-p700)` |
| `--text-calendar-current` | `#007549` | `#6AD6A5` | `var(--brand-p700)` |
| `--text-calendar-default` | `#4D5358` | `#C9CDD0` |  |
| `--text-calendar-disabled` | `#CBD5E1` | `#334155` |  |
| `--text-calendar-month` | `#4D5358` | `#C9CDD0` |  |
| `--text-calendar-selected` | `#FFFFFF` | `#111314` |  |
| `--text-colorfulbadge-badge-on` | `#FFFFFF` | `#111314` |  |
| `--text-colorfulbadge-blue` | `#1D4ED8` | `#93C5FD` |  |
| `--text-colorfulbadge-green` | `#007549` | `#6AD6A5` | `var(--brand-p700)` |
| `--text-colorfulbadge-grey` | `#4D5358` | `#C9CDD0` |  |
| `--text-colorfulbadge-indigo` | `#4F46E5` | `#C7D2FE` |  |
| `--text-colorfulbadge-magenta` | `#A21CAF` |  |  |
| `--text-colorfulbadge-orange` | `#C2410C` | `#FDBA74` |  |
| `--text-colorfulbadge-pink` | `#BE185D` | `#F9A8D4` |  |
| `--text-colorfulbadge-purple` | `#7E22CE` | `#D8B4FE` |  |
| `--text-colorfulbadge-red` | `#B91C1C` | `#FCA5A5` |  |
| `--text-colorfulbadge-teal` | `#196C65` | `#6CDCCA` |  |
| `--text-colorfulbadge-turquoise` | `#196C65` | `#6CDCCA` | `var(--text-colorfulbadge-teal)` |
| `--text-colorfulbadge-yellow` | `#A16207` | `#FDE047` |  |
| `--text-content-default` | `#363B3F` | `#F4F5F5` |  |
| `--text-content-on-content` | `#FFFFFF` | `#111314` |  |
| `--text-content-primary` | `#363B3F` | `#F4F5F5` | `var(--text-content-default)` |
| `--text-content-quaternary` | `#ADB2B7` | `#858C92` |  |
| `--text-content-secondary` | `#636B72` | `#ADB2B7` |  |
| `--text-content-tertiary` | `#858C92` |  |  |
| `--text-danger-default` | `#B91C1C` | `#F87171` |  |
| `--text-danger-secondary` | `#DC2626` | `#FCA5A5` |  |
| `--text-disabled-secondary` | `#636B72` | `#ADB2B7` |  |
| `--text-disabled-tertiary` | `#ADB2B7` | `#636B72` |  |
| `--text-disabledButton-tertiary` | `#ADB2B7` | `#858C92` |  |
| `--text-info-default` | `#1D4ED8` | `#93C5FD` |  |
| `--text-info-emphasis` | `#3B82F6` | `#BFDBFE` |  |
| `--text-info-tertiary` | `#60A5FA` | `#2563EB` |  |
| `--text-input-default` | `#ADB2B7` | `#858C92` |  |
| `--text-input-disabled` | `#ADB2B7` | `#858C92` |  |
| `--text-input-error` | `#ADB2B7` | `#858C92` |  |
| `--text-input-filled` | `#4D5358` | `#F4F5F5` |  |
| `--text-input-hover` | `#ADB2B7` | `#858C92` |  |
| `--text-input-label` | `#4D5358` | `#F4F5F5` |  |
| `--text-input-require` | `#EC3FAA` | `#F87171` | `var(--mobile-theme-ci-pink)` |
| `--text-input-typing` | `#4D5358` | `#F4F5F5` |  |
| `--text-list-default` | `#4D5358` | `#F4F5F5` |  |
| `--text-list-hover` | `#007549` | `#6AD6A5` | `var(--brand-p700)` |
| `--text-list-subtext` | `#858C92` |  |  |
| `--text-neutral-default` | `#4D5358` | `#F4F5F5` | `var(--text-neutralbutton-default)` |
| `--text-neutral-on-neutral` | `#FFFFFF` |  |  |
| `--text-neutral-tertiary` | `#ADB2B7` | `#858C92` |  |
| `--text-neutralbutton-default` | `#4D5358` | `#F4F5F5` |  |
| `--text-payment-total` | `#24A899` |  |  |
| `--text-pill-blue` | `#1D4ED8` | `#BFDBFE` |  |
| `--text-pill-orange` | `#C2410C` | `#FED7AA` |  |
| `--text-pill-red` | `#B91C1C` | `#FECACA` |  |
| `--text-pill-teal` | `#196C65` | `#A2EDDE` |  |
| `--text-pill-yellow` | `#A16207` | `#FEF08A` |  |
| `--text-primarysearch-default` | `#ADB2B7` | `#858C92` |  |
| `--text-primarysearch-filled` | `#4D5358` | `#F4F5F5` |  |
| `--text-primarysearch-focus` | `#ADB2B7` | `#858C92` | `var(--text-primarysearch-default)` |
| `--text-primarysearch-hover` | `#4D5358` | `#F4F5F5` |  |
| `--text-primarysearch-typing` | `#4D5358` | `#F4F5F5` |  |
| `--text-secondarysearch-default` | `#ADB2B7` | `#858C92` |  |
| `--text-statusbadge-error` | `#B91C1C` | `#F58B8B` |  |
| `--text-statusbadge-info` | `#1D4ED8` | `#6BA5F5` |  |
| `--text-statusbadge-neutral` | `#4D5358` | `#F4F5F5` |  |
| `--text-statusbadge-success` | `#196C65` | `#3FCE93` |  |
| `--text-statusbadge-warning` | `#C2410C` | `#F5A76B` |  |
| `--text-statusbadge-yellow` | `#A16207` | `#FEF08A` | `var(--text-pill-yellow)` |
| `--text-table-title` | `#004C31` | `#6AD6A5` | `var(--brand-p800)` |
| `--text-topic-tertiary-title` | `#363B3F` | `#F4F5F5` |  |
| `--text-warning-default` | `#C2410C` | `#FDBA74` |  |

#### Border (50)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--border-alert-error` | `#FEE2E2` | `#7F1D1D` |  |
| `--border-alert-info` | `#DBEAFE` | `#1E3A8A` |  |
| `--border-alert-success` | `#D1F6EE` | `#194845` |  |
| `--border-alert-warning` | `#FFEDD5` | `#7C2D12` |  |
| `--border-brandPrimary-default` | `#007549` |  | `var(--brand-p700)` |
| `--border-brandPrimary-quaternary` | `#E2F3EB` | `#004C31` | `var(--brand-p100)` |
| `--border-brandPrimary-secondary` | `#08A768` | `#363B3F` | `var(--brand-p600)` |
| `--border-brandPrimary-tertiary` | `#C5EDDA` | `#007549` | `var(--brand-p200)` |
| `--border-brandPrimaryButton-default` | `#007549` |  | `var(--brand-p700)` |
| `--border-brandPrimaryButton-tertiary` | `#C5EDDA` | `#08A768` | `var(--brand-p200)` |
| `--border-brandprimarybutton-default` | `#007549` |  | `var(--border-brandPrimaryButton-default)` |
| `--border-brandprimarybutton-tertiary` | `#C5EDDA` | `#08A768` | `var(--brand-p200)` |
| `--border-brandtag-default` | `#E2E4E6` | `#4D5358` |  |
| `--border-brandtag-selected` | `#C5EDDA` | `#08A768` | `var(--brand-p200)` |
| `--border-card-default` | `#E2E4E6` | `#4D5358` |  |
| `--border-colorfulbadge-brand` | `#C5EDDA` | `#004C31` |  |
| `--border-colorfulbadge-error` | `#FECACA` | `#991B1B` |  |
| `--border-colorfulbadge-info` | `#BFDBFE` | `#1E40AF` |  |
| `--border-colorfulbadge-neutral` | `#E2E4E6` | `#363B3F` |  |
| `--border-colorfulbadge-success` | `#A2EDDE` | `#195652` |  |
| `--border-colorfulbadge-warning` | `#FEF08A` | `#854D0E` |  |
| `--border-colorfulbadge-warning2` | `#FED7AA` | `#9A3412` |  |
| `--border-content-tertiary` | `#E1E3E5` | `#4D5358` |  |
| `--border-dangerButton-default` | `#DC2626` | `#FCA5A5` |  |
| `--border-dangerButton-tertiary` | `#FECACA` | `#7F1D1D` |  |
| `--border-image-default` | `#FFFFFF` | `#222629` |  |
| `--border-info-quaternary` | `#DBEAFE` | `#1E3A8A` |  |
| `--border-info-tertiary` | `#BFDBFE` | `#1E40AF` |  |
| `--border-input-default` | `#91DFBB` | `#4D5358` | `var(--mobile-theme-border-choice)` |
| `--border-input-disabled` | `#E2E4E6` | `#4D5358` |  |
| `--border-input-error` | `#DC2626` | `#F87171` |  |
| `--border-input-filled` | `#91DFBB` | `#4D5358` | `var(--mobile-theme-border-choice)` |
| `--border-input-hover` | `#08A768` |  | `var(--brand-p600)` |
| `--border-neutral-quat` | `#F4F5F5` | `#363B3F` |  |
| `--border-neutral-quaternary` | `#F4F5F5` | `#363B3F` | `var(--border-neutral-quat)` |
| `--border-neutral-secondary` | `#C8CDD1` |  |  |
| `--border-neutral-tertiary` | `#E2E4E6` | `#363B3F` |  |
| `--border-neutralbutton-tertiary` | `#E2E4E6` | `#4D5358` |  |
| `--border-popover-grey` | `#E2E4E6` | `#4D5358` |  |
| `--border-primarysearch-default` | `#08A768` |  | `var(--brand-p600)` |
| `--border-primarysearch-filled` | `#08A768` |  | `var(--brand-p600)` |
| `--border-primarysearch-focus` | `#08A768` |  | `var(--border-primarysearch-default)` |
| `--border-primarysearch-hover` | `#007549` |  | `var(--brand-p700)` |
| `--border-primarysearch-typing` | `#007549` |  | `var(--brand-p700)` |
| `--border-statusbadge-neutral` | `#E2E4E6` | `#4D5358` |  |
| `--border-switch-default` | `#C9CDD0` | `#4D5358` |  |
| `--border-switch-disabled` | `#E2E4E6` | `#4D5358` |  |
| `--border-telemed-queue-orange-blink` | `rgba(254, 215, 170, 0.4)` |  |  |
| `--border-telemed-queue-teal-blink` | `rgba(209, 246, 238, 0.4)` |  |  |
| `--border-warning-default` | `#FDBA74` |  |  |

#### Icon (19)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--icon-brandPrimary-default` | `#007549` | `#6AD6A5` | `var(--brand-p700)` |
| `--icon-content-tertiary` | `#7A7F83` |  |  |
| `--icon-input-default` | `#ADB2B7` | `#636B72` |  |
| `--icon-input-disabled` | `#ADB2B7` | `#636B72` |  |
| `--icon-input-error` | `#ADB2B7` | `#636B72` |  |
| `--icon-input-filled` | `#4D5358` | `#636B72` |  |
| `--icon-input-hover` | `#08A768` | `#6AD6A5` | `var(--brand-p600)` |
| `--icon-list-secondary-default` | `#858C92` | `#C9CDD0` |  |
| `--icon-neutral-default` | `#4D5358` | `#C9CDD0` |  |
| `--icon-neutral-tertiary` | `#ADB2B7` | `#636B72` |  |
| `--icon-positive-default` | `#196C65` | `#6CDCCA` |  |
| `--icon-positive-success` | `#24A899` |  |  |
| `--icon-primarysearch-default` | `#08A768` | `#6AD6A5` | `var(--brand-p600)` |
| `--icon-primarysearch-filled` | `#08A768` | `#6AD6A5` | `var(--brand-p600)` |
| `--icon-primarysearch-focus` | `#08A768` | `#6AD6A5` | `var(--brand-p600)` |
| `--icon-primarysearch-hover` | `#007549` | `#91DFBB` | `var(--brand-p700)` |
| `--icon-primarysearch-typing` | `#007549` | `#91DFBB` | `var(--brand-p700)` |
| `--icon-step-accent` | `#FD879F` |  |  |
| `--icon-switch-checkbox-tick` | `#FFFFFF` | `#111314` |  |

#### Mobile — platform (`--mobile-*`) (177)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--mobile-badge-padding-x` | `8px` |  | `var(--dim-space-200, 8px)` |
| `--mobile-badge-padding-y` | `2px` |  |  |
| `--mobile-badge-radius` | `9999px` |  | `var(--dim-radius-full, 9999px)` |
| `--mobile-badge-type-size` | `12px` |  | `var(--type-size-xs, 12px)` |
| `--mobile-border-brandPrimaryButton-default` | `#007549` | `#91DFBB` | `var(--brand-p700)` |
| `--mobile-border-brandPrimaryButton-default-hover` | `#004C31` | `#C5EDDA` | `var(--brand-p800)` |
| `--mobile-border-brandPrimaryButton-quaternary` | `#E2F3EB` | `#003322` |  |
| `--mobile-border-brandPrimaryButton-quaternary-hover` | `#C5EDDA` | `#004C31` |  |
| `--mobile-border-brandPrimaryButton-secondary` | `#6AD6A5` | `#08A768` |  |
| `--mobile-border-brandPrimaryButton-secondary-hover` | `#54CF97` |  |  |
| `--mobile-border-brandPrimaryButton-tertiary` | `#C5EDDA` | `#004C31` |  |
| `--mobile-border-brandPrimaryButton-tertiary-hover` | `#91DFBB` | `#007549` |  |
| `--mobile-border-brandSecondaryButton-default` | `#427C3D` | `#78B573` |  |
| `--mobile-border-brandSecondaryButton-default-hover` | `#366233` | `#A7D2A3` |  |
| `--mobile-border-brandSecondaryButton-quaternary` | `#EFF5EF` | `#274126` |  |
| `--mobile-border-brandSecondaryButton-quaternary-hover` | `#E1F0E0` | `#2E4F2C` |  |
| `--mobile-border-brandSecondaryButton-secondary` | `#78B573` | `#427C3D` |  |
| `--mobile-border-brandSecondaryButton-secondary-hover` | `#53954E` |  |  |
| `--mobile-border-brandSecondaryButton-tertiary` | `#E1F0E0` | `#2E4F2C` |  |
| `--mobile-border-brandSecondaryButton-tertiary-hover` | `#A7D2A3` | `#366233` |  |
| `--mobile-border-dangerButton-default` | `#DC2626` | `#F87171` |  |
| `--mobile-border-dangerButton-default-hover` | `#B91C1C` | `#FCA5A5` |  |
| `--mobile-border-dangerButton-quaternary` | `#FEE2E2` | `#7F1D1D` |  |
| `--mobile-border-dangerButton-quaternary-hover` | `#FECACA` | `#991B1B` |  |
| `--mobile-border-dangerButton-secondary` | `#F87171` | `#DC2626` |  |
| `--mobile-border-dangerButton-secondary-hover` | `#EF4444` |  |  |
| `--mobile-border-dangerButton-tertiary` | `#FECACA` | `#991B1B` |  |
| `--mobile-border-dangerButton-tertiary-hover` | `#FCA5A5` | `#B91C1C` |  |
| `--mobile-border-disabledButton-default` | `#636B72` | `#ADB2B7` |  |
| `--mobile-border-disabledButton-quaternary` | `#F4F5F5` | `#222629` |  |
| `--mobile-border-disabledButton-secondary` | `#ADB2B7` | `#636B72` |  |
| `--mobile-border-disabledButton-tertiary` | `#E2E4E6` | `#363B3F` |  |
| `--mobile-border-neutralButton-default` | `#636B72` | `#ADB2B7` |  |
| `--mobile-border-neutralButton-default-hover` | `#4D5358` | `#C9CDD0` |  |
| `--mobile-border-neutralButton-quaternary` | `#F4F5F5` | `#222629` |  |
| `--mobile-border-neutralButton-quaternary-hover` | `#E2E4E6` | `#363B3F` |  |
| `--mobile-border-neutralButton-secondary` | `#ADB2B7` | `#636B72` |  |
| `--mobile-border-neutralButton-secondary-hover` | `#858C92` |  |  |
| `--mobile-border-neutralButton-tertiary` | `#E2E4E6` | `#363B3F` |  |
| `--mobile-border-neutralButton-tertiary-hover` | `#C9CDD0` | `#4D5358` |  |
| `--mobile-button-gap` | `8px` |  | `var(--dim-space-200, 8px)` |
| `--mobile-button-gap-sm` | `6px` |  | `var(--dim-space-150, 6px)` |
| `--mobile-button-height-lg` | `56px` |  |  |
| `--mobile-button-height-md` | `48px` |  |  |
| `--mobile-button-height-sm` | `40px` |  |  |
| `--mobile-button-padding-x` | `24px` |  | `var(--dim-space-600, 24px)` |
| `--mobile-button-radius` | `9999px` |  | `var(--dim-radius-full, 9999px)` |
| `--mobile-button-shadow-fill` | `0 4px 4px -1px rgb(8 167 104 / 0.08), 0 4px 4px -1px rgb(8 167 104 / 0.10)` |  | `0 var(--dim-space-100, 4px) var(--dim-space-100, 4px) -1px var(--PrimaryShadow-600), 0 …` |
| `--mobile-button-type-size` | `16px` |  | `var(--type-size-base, 16px)` |
| `--mobile-button-type-size-lg` | `18px` |  | `var(--type-size-lg, 18px)` |
| `--mobile-button-type-weight` | `600` |  | `var(--mobile-theme-weight-semibold)` |
| `--mobile-card-padding` | `16px` |  | `var(--dim-space-400, 16px)` |
| `--mobile-card-radius` | `12px` |  | `var(--dim-radius-400)` |
| `--mobile-focus-ring-width` | `4px` |  |  |
| `--mobile-header-height` | `56px` |  |  |
| `--mobile-icon-brandPrimaryButton-default` | `#007549` | `#91DFBB` | `var(--brand-p700)` |
| `--mobile-icon-brandPrimaryButton-on-brand` | `#FFFFFF` | `#002318` |  |
| `--mobile-icon-brandPrimaryButton-quaternary` | `#91DFBB` | `#007549` | `var(--brand-p300)` |
| `--mobile-icon-brandPrimaryButton-secondary` | `#54CF97` | `#6AD6A5` | `var(--brand-p500)` |
| `--mobile-icon-brandPrimaryButton-tertiary` | `#6AD6A5` | `#54CF97` | `var(--brand-p400)` |
| `--mobile-icon-brandSecondaryButton-default` | `#366233` | `#A7D2A3` |  |
| `--mobile-icon-brandSecondaryButton-on-default` | `#FFFFFF` |  |  |
| `--mobile-icon-brandSecondaryButton-quaternary` | `#A7D2A3` | `#366233` |  |
| `--mobile-icon-brandSecondaryButton-secondary` | `#427C3D` | `#78B573` |  |
| `--mobile-icon-brandSecondaryButton-tertiary` | `#78B573` | `#427C3D` |  |
| `--mobile-icon-dangerButton-default` | `#B91C1C` | `#FCA5A5` |  |
| `--mobile-icon-dangerButton-on-default` | `#FFFFFF` | `#020617` |  |
| `--mobile-icon-dangerButton-quaternary` | `#FCA5A5` | `#B91C1C` |  |
| `--mobile-icon-dangerButton-secondary` | `#DC2626` | `#F87171` |  |
| `--mobile-icon-dangerButton-tertiary` | `#F87171` | `#DC2626` |  |
| `--mobile-icon-disabledButton-default` | `#4D5358` | `#C9CDD0` |  |
| `--mobile-icon-disabledButton-on-disabled` | `#FFFFFF` | `#111314` |  |
| `--mobile-icon-disabledButton-quaternary` | `#C9CDD0` | `#4D5358` |  |
| `--mobile-icon-disabledButton-secondary` | `#636B72` | `#ADB2B7` |  |
| `--mobile-icon-disabledButton-tertiary` | `#ADB2B7` | `#636B72` |  |
| `--mobile-icon-neutralButton-default` | `#4D5358` | `#C9CDD0` |  |
| `--mobile-icon-neutralButton-on-neutral` | `#FFFFFF` | `#111314` |  |
| `--mobile-icon-neutralButton-quaternary` | `#C9CDD0` | `#4D5358` |  |
| `--mobile-icon-neutralButton-secondary` | `#636B72` | `#ADB2B7` |  |
| `--mobile-icon-neutralButton-tertiary` | `#ADB2B7` | `#636B72` |  |
| `--mobile-input-height` | `48px` |  |  |
| `--mobile-input-padding-x` | `16px` |  | `var(--dim-space-400, 16px)` |
| `--mobile-input-radius` | `12px` |  | `var(--dim-radius-400, 12px)` |
| `--mobile-input-type-size` | `16px` |  | `var(--type-size-base, 16px)` |
| `--mobile-status-bar-height` | `44px` |  |  |
| `--mobile-surface-brandPrimaryButton-default` | `#007549` | `#91DFBB` | `var(--brand-p700)` |
| `--mobile-surface-brandPrimaryButton-default-hover` | `#004C31` | `#C5EDDA` | `var(--brand-p800)` |
| `--mobile-surface-brandPrimaryButton-quaternary` | `#F4FBF8` | `#002318` | `var(--brand-p50)` |
| `--mobile-surface-brandPrimaryButton-quaternary-hover` | `#E2F3EB` | `#003322` | `var(--brand-p100)` |
| `--mobile-surface-brandPrimaryButton-quinary` | `#FFFFFF` | `#020617` |  |
| `--mobile-surface-brandPrimaryButton-quinary-hover` | `#F4FBF8` | `#002318` | `var(--brand-p50)` |
| `--mobile-surface-brandPrimaryButton-secondary` | `#C5EDDA` | `#004C31` | `var(--brand-p200)` |
| `--mobile-surface-brandPrimaryButton-secondary-hover` | `#91DFBB` | `#007549` | `var(--brand-p300)` |
| `--mobile-surface-brandPrimaryButton-tertiary` | `#E2F3EB` | `#003322` | `var(--brand-p100)` |
| `--mobile-surface-brandPrimaryButton-tertiary-hover` | `#C5EDDA` | `#004C31` | `var(--brand-p200)` |
| `--mobile-surface-brandSecondaryButton-default` | `#427C3D` | `#78B573` |  |
| `--mobile-surface-brandSecondaryButton-default-hover` | `#366233` | `#A7D2A3` |  |
| `--mobile-surface-brandSecondaryButton-quaternary` | `#FAFCFA` | `#102310` |  |
| `--mobile-surface-brandSecondaryButton-quaternary-hover` | `#EFF5EF` | `#274126` |  |
| `--mobile-surface-brandSecondaryButton-quinary` | `#FFFFFF` | `#020617` |  |
| `--mobile-surface-brandSecondaryButton-quinary-hover` | `#FAFCFA` | `#102310` |  |
| `--mobile-surface-brandSecondaryButton-secondary` | `#E1F0E0` | `#2E4F2C` |  |
| `--mobile-surface-brandSecondaryButton-secondary-hover` | `#A7D2A3` | `#366233` |  |
| `--mobile-surface-brandSecondaryButton-tertiary` | `#EFF5EF` | `#274126` |  |
| `--mobile-surface-brandSecondaryButton-tertiary-hover` | `#E1F0E0` | `#2E4F2C` |  |
| `--mobile-surface-dangerButton-default` | `#B91C1C` | `#FCA5A5` |  |
| `--mobile-surface-dangerButton-default-hover` | `#991B1B` | `#FCA5A5` |  |
| `--mobile-surface-dangerButton-quaternary` | `#FFF0F0` | `#450A0A` |  |
| `--mobile-surface-dangerButton-quaternary-hover` | `#FEE2E2` | `#7F1D1D` |  |
| `--mobile-surface-dangerButton-quinary` | `#FFFFFF` | `#DADADA` |  |
| `--mobile-surface-dangerButton-quinary-hover` | `#FFF0F0` | `#450A0A` |  |
| `--mobile-surface-dangerButton-secondary` | `#FECACA` | `#DC2626` |  |
| `--mobile-surface-dangerButton-secondary-hover` | `#FCA5A5` | `#EF4444` |  |
| `--mobile-surface-dangerButton-tertiary` | `#FEE2E2` | `#7F1D1D` |  |
| `--mobile-surface-dangerButton-tertiary-hover` | `#FECACA` | `#991B1B` |  |
| `--mobile-surface-disabledButton-default` | `#4D5358` | `#C9CDD0` |  |
| `--mobile-surface-disabledButton-default-hover` | `#363B3F` | `#E2E4E6` |  |
| `--mobile-surface-disabledButton-quaternary` | `#F9FBFB` | `#111314` |  |
| `--mobile-surface-disabledButton-quaternary-hover` | `#F4F5F5` | `#222629` |  |
| `--mobile-surface-disabledButton-secondary` | `#E2E4E6` | `#363B3F` |  |
| `--mobile-surface-disabledButton-secondary-hover` | `#C9CDD0` | `#4D5358` |  |
| `--mobile-surface-disabledButton-tertiary` | `#F4F5F5` | `#363B3F` |  |
| `--mobile-surface-disabledButton-tertiary-hover` | `#E2E4E6` | `#4D5358` |  |
| `--mobile-surface-keypadButton-pressed` | `color-mix( in srgb, #FFFFFF 20%, transparent )` | `color-mix( in srgb, #111314 20%, transparent )` | `color-mix( in srgb, var(--text-content-on-content, #FFFFFF) 20%, transparent )` |
| `--mobile-surface-neutralButton-default` | `#4D5358` | `#C9CDD0` |  |
| `--mobile-surface-neutralButton-default-hover` | `#363B3F` | `#E2E4E6` |  |
| `--mobile-surface-neutralButton-quaternary` | `#F9FBFB` | `#111314` |  |
| `--mobile-surface-neutralButton-quaternary-hover` | `#F4F5F5` | `#222629` |  |
| `--mobile-surface-neutralButton-quinary` | `#FFFFFF` | `#111314` |  |
| `--mobile-surface-neutralButton-quinary-hover` | `#F9FBFB` | `#111314` |  |
| `--mobile-surface-neutralButton-secondary` | `#E2E4E6` | `#363B3F` |  |
| `--mobile-surface-neutralButton-secondary-hover` | `#C9CDD0` | `#4D5358` |  |
| `--mobile-surface-neutralButton-tertiary` | `#F4F5F5` | `#222629` |  |
| `--mobile-surface-neutralButton-tertiary-hover` | `#E2E4E6` | `#363B3F` |  |
| `--mobile-text-brandPrimaryButton-default` | `#007549` | `#91DFBB` | `var(--brand-p700)` |
| `--mobile-text-brandPrimaryButton-on-brand` | `#FFFFFF` | `#002318` |  |
| `--mobile-text-brandPrimaryButton-quaternary` | `#91DFBB` | `#007549` | `var(--brand-p300)` |
| `--mobile-text-brandPrimaryButton-secondary` | `#54CF97` | `#6AD6A5` | `var(--brand-p500)` |
| `--mobile-text-brandPrimaryButton-tertiary` | `#6AD6A5` | `#54CF97` | `var(--brand-p400)` |
| `--mobile-text-brandSecondaryButton-default` | `#366233` | `#A7D2A3` |  |
| `--mobile-text-brandSecondaryButton-on-default` | `#FFFFFF` | `#102310` |  |
| `--mobile-text-brandSecondaryButton-quaternary` | `#A7D2A3` | `#366233` |  |
| `--mobile-text-brandSecondaryButton-secondary` | `#427C3D` | `#78B573` |  |
| `--mobile-text-brandSecondaryButton-tertiary` | `#78B573` | `#427C3D` |  |
| `--mobile-text-dangerButton-default` | `#B91C1C` | `#FCA5A5` |  |
| `--mobile-text-dangerButton-on-default` | `#FFFFFF` | `#020617` |  |
| `--mobile-text-dangerButton-quaternary` | `#FCA5A5` | `#B91C1C` |  |
| `--mobile-text-dangerButton-secondary` | `#DC2626` | `#F87171` |  |
| `--mobile-text-dangerButton-tertiary` | `#F87171` | `#DC2626` |  |
| `--mobile-text-disabledButton-default` | `#4D5358` | `#C9CDD0` |  |
| `--mobile-text-disabledButton-on-disabled` | `#FFFFFF` | `#111314` |  |
| `--mobile-text-disabledButton-quaternary` | `#C9CDD0` | `#4D5358` |  |
| `--mobile-text-disabledButton-secondary` | `#636B72` | `#ADB2B7` |  |
| `--mobile-text-disabledButton-tertiary` | `#ADB2B7` | `#636B72` |  |
| `--mobile-text-neutralButton-default` | `#4D5358` | `#C9CDD0` |  |
| `--mobile-text-neutralButton-on-neutral` | `#FFFFFF` | `#111314` |  |
| `--mobile-text-neutralButton-quaternary` | `#C9CDD0` | `#4D5358` |  |
| `--mobile-text-neutralButton-secondary` | `#636B72` | `#ADB2B7` |  |
| `--mobile-text-neutralButton-tertiary` | `#ADB2B7` | `#636B72` |  |
| `--mobile-theme-accent-green` | `#50A622` |  |  |
| `--mobile-theme-accent-pink` | `#EC3FAA` |  |  |
| `--mobile-theme-border-card` | `#C5EDDA` | `#363B3F` | `var(--brand-p200)` |
| `--mobile-theme-border-choice` | `#91DFBB` | `#4D5358` | `var(--brand-p300)` |
| `--mobile-theme-ci-blue` | `#0071CE` |  |  |
| `--mobile-theme-ci-emerald` | `#00744B` |  |  |
| `--mobile-theme-ci-forest` | `#205A41` |  |  |
| `--mobile-theme-ci-gold` | `#FFB449` |  |  |
| `--mobile-theme-ci-mint` | `#00C08B` |  |  |
| `--mobile-theme-ci-pink` | `#EC3FAA` |  | `var(--mobile-theme-accent-pink)` |
| `--mobile-theme-family-display` | `'Sao Chingcha', 'Sarabun', system-ui, sans-serif` |  | `var(--type-family-heading-graphic)` |
| `--mobile-theme-family-label` | `'Anuphan Medium', 'Anuphan', 'Sarabun', system-ui, sans-serif` |  |  |
| `--mobile-theme-filter-shadow-pink` | `drop-shadow(0 4px 8px rgb(236 63 170 / 0.25))` |  | `drop-shadow(0 var(--dim-space-100, 4px) var(--dim-space-200, 8px) rgb(236 63 170 / 0.25))` |
| `--mobile-theme-header` | `#007549` |  | `var(--surface-brandPrimary-default)` |
| `--mobile-theme-overlay` | `rgb(0 0 0 / 0.50)` |  |  |
| `--mobile-theme-shadow-pink` | `0 4px 16px rgb(236 63 170 / 0.20)` |  | `0 var(--dim-space-100, 4px) var(--dim-space-400, 16px) rgb(236 63 170 / 0.20)` |
| `--mobile-theme-weight-semibold` | `600` |  |  |
| `--mobile-touch-target-min` | `44px` |  |  |

#### อื่น ๆ (18)

| Token | สว่าง | มืด | อ้างถึง |
|---|---|---|---|
| `--PrimaryShadow-600` | `rgb(8 167 104 / 0.08)` |  | `rgb(var(--brand-p600-rgb) / 0.08)` |
| `--PrimaryShadow-601` | `rgb(8 167 104 / 0.10)` |  | `rgb(var(--brand-p600-rgb) / 0.10)` |
| `--d-bg` | `—` | `#111314` |  |
| `--d-border-1` | `—` | `#363B3F` |  |
| `--d-border-2` | `—` | `#4D5358` |  |
| `--d-hover` | `—` | `#363B3F` |  |
| `--d-icon` | `—` | `#C9CDD0` |  |
| `--d-surface-1` | `—` | `#222629` |  |
| `--d-surface-2` | `—` | `#363B3F` |  |
| `--d-surface-3` | `—` | `#4D5358` |  |
| `--d-text-1` | `—` | `#F4F5F5` |  |
| `--d-text-2` | `—` | `#ADB2B7` |  |
| `--d-text-3` | `—` | `#858C92` |  |
| `--labels-primary` | `#000000` | `#FFFFFF` |  |
| `--opacity-disabled` | `0.5` |  |  |
| `--primaryBorder-600` | `rgb(8 167 104 / 0.15)` |  | `rgb(var(--brand-p600-rgb) / 0.15)` |
| `--sky-cyan-50` | `#F4F9FD` |  |  |
| `--sky-cyan-950` | `#081E40` |  |  |

<!-- tokens:end -->

### 7.1. Layering rules

```
┌──────────────────────────────────────────────────┐
│  Mobile foundation                               │  §7.0
│  --brand-p*, --type-family-*, --text-*, etc.     │
│  (mih-design mobile-foundation.css)              │
└──────────────────────────────────────────────────┘
                         ▲
┌──────────────────────────────────────────────────┐
│  Mobile tokens  --mobile-*                       │  §7.2
└──────────────────────────────────────────────────┘
                         ▲
                Mobile components (§7.3)
```

**Hard rules:**

| Rule | What it means | Where it's enforced |
|------|---------------|--------------------|
| ✅ Mobile components alias `--mobile-*` and foundation tokens | shape/height/density = `--mobile-*` · colour / type / icon = mobile foundation (§7.0) | [`src/shared/ui/*`](src/shared/ui/) — every primitive |
| ❌ No token outside §7.0 | a value the app needs but §7.0 lacks → add it to mih-design `mobile.css` first, then `npm run tokens:pull` | review |
| ❌ Mobile components MUST NOT redefine colour / typography family / icon | brand fork = brand broken | review |
| ❌ Mobile components MUST NOT alias `--btn-*`, `--card-*`, web-only component tokens | those carry web-only geometry (radius 6, height 44/56 fixed shape) | review |
| ✅ Touch target floor = `var(--mobile-touch-target-min, 44px)` | iOS HIG / Material 3 minimum — even for icon-only taps | every interactive primitive |

If you find yourself reaching for a literal pixel in a mobile component, the answer is **add a new `--mobile-*` token** — not paste in `12px`.

### 7.2. Platform tokens (`--mobile-*`)

Defined in [`src/index.css`](src/index.css) — a single `:root` block right above the Spotlight CSS. Each token aliases a foundation `--dim-*` / `--type-*` value where possible, so a foundation rescale still propagates.

#### Button

| Token | Value | Resolves to |
|-------|-------|-------------|
| `--mobile-button-radius` | `var(--dim-radius-full)` | `9999px` (pill) |
| `--mobile-button-height-sm` | `40px` | Figma Size=Small (`dimension/size/800`) |
| `--mobile-button-height-md` | `48px` | Figma Size=Medium (`dimension/size/1000`) |
| `--mobile-button-height-lg` | `56px` | Figma Size=Large (`dimension/size/1200`) |
| `--mobile-button-padding-x` | `var(--dim-space-600)` | `24px` |
| `--mobile-button-gap` | `var(--dim-space-200)` | `8px` (Medium / Large) |
| `--mobile-button-gap-sm` | `var(--dim-space-150)` | `6px` (Small) |
| `--mobile-button-type-size` | `var(--type-size-base)` | `16px` (Small) |
| `--mobile-button-type-size-lg` | `var(--type-size-lg)` | `18px` (Medium / Large) |
| `--mobile-button-type-weight` | `var(--type-weight-regular)` | `400` (heading-graphic2) |
| `--mobile-button-shadow-fill` | Brand Drop Shadow Bottom/300 | Fill · Default / Hover / Focus |
| `--mobile-surface-keypadButton-pressed` | white @ 20% (`color-mix` on `--text-content-on-content`) | KeypadButton · Pressed fill (Archive `314:43835`) |

> **Why pill, not 6 px?** The brief is "ทรงมนโค้ง" — mobile buttons read as soft chips, not rectangular form-controls. Pill survives at any width without re-tuning the radius (a fixed `12 px` looks pill at width 60, square at width 280).

#### Input

| Token | Value | Resolves to |
|-------|-------|-------------|
| `--mobile-input-radius` | `var(--dim-radius-400)` | `12px` |
| `--mobile-input-height` | `48px` | comfortable thumb-tap target |
| `--mobile-input-padding-x` | `var(--dim-space-400)` | `16px` |
| `--mobile-input-type-size` | `var(--type-size-base)` | `16px` ⚠️ |

> ⚠️ **iOS Safari auto-zoom**: input `font-size < 16 px` triggers the viewport zoom on focus. Never override `--mobile-input-type-size` below 16.

#### Card

| Token | Value | Resolves to |
|-------|-------|-------------|
| `--mobile-card-radius` | `var(--dim-radius-500)` | `16px` |
| `--mobile-card-padding` | `var(--dim-space-400)` | `16px` |

#### Badge

| Token | Value | Resolves to |
|-------|-------|-------------|
| `--mobile-badge-radius` | `var(--dim-radius-full)` | `9999px` (pill) |
| `--mobile-badge-padding-x` | `var(--dim-space-200)` | `8px` |
| `--mobile-badge-padding-y` | — | `2px` |
| `--mobile-badge-type-size` | `var(--type-size-xs)` | `12px` |

#### Shell (frame chrome)

| Token | Value | Resolves to |
|-------|-------|-------------|
| `--mobile-header-height` | — | `56px` |
| `--mobile-status-bar-height` | — | `44px` |
| `--mobile-touch-target-min` | — | `44px` |

#### Semantic colour (`--mobile-surface-*` / `--mobile-text-*` / `--mobile-border-*` / `--mobile-icon-*`)

Mobile colour tokens are **separate from web** (web uses `--surface-*` / `--text-*` directly). This split lets the citizen app run a different palette — currently a brighter emerald (`#08A768`) versus the desktop deep-green (`#007549`) — without re-skinning the desktop EMR. Both palettes inherit foundation tokens (typography family, motion, icon families).

Source of truth: Figma file `opz7X9AAKGQf5prHhsPDGn` Light/Dark variable collections → mih-design `packages/design-tokens/mobile.css` → `npm run tokens:pull` → [`src/styles/mobile-tokens.css`](src/styles/mobile-tokens.css) (generated mirror — never edit by hand). Light values live under `:root`, dark overrides under `[data-mobile-theme="dark"]` (toggleable per-subtree).

Each tone covers 4 categories × 5 surface tiers + on-* text variants. The button system reads from these tokens via the `buildPalette(tone, options)` helper in [_base.jsx](src/shared/ui/button/_base.jsx):

| Tone | Used by | Default fill (light) | Default fill (dark) |
|------|---------|----------------------|---------------------|
| `brandPrimaryButton` | BrandButton, BrandSubdueButton, BrandIconButton, BrandSubdueIconButton, BrandCircleIconButton, BrandCircleSubdueIconButton, BrandSubdueFloatButton | `var(--brand-p700)` #007549 (hover p800 #004C31) | `#91DFBB` |
| `brandSecondaryButton` | reserved for second-tier brand actions | `#427C3D` | `#78B573` |
| `dangerButton` | DangerButton, DangerSubdueButton, DangerIconButton, DangerSubdueIconButton | `#B91C1C` | `#FCA5A5` |
| `neutralButton` | NeutralButton, NeutralSubdueButton, NeutralIconButton, NeutralSubdueIconButton | `#4D5358` | `#C9CDD0` |
| `disabledButton` | every variant's `disabled` state (all tones) — reads the **`tertiary`** tier: bg `#F4F5F5` · text `#ADB2B7` · outline border `#E2E4E6` | `#F4F5F5` | `#363B3F` |

Per-tone surface tiers — same shape for every tone:

| Tier | Where it shows up |
|------|-------------------|
| `default` | Fill bg for solid (fill) variant |
| `default-hover` | Hover bg for solid fill |
| `secondary` | Active bg for solid fill (pressed) |
| `secondary-hover` | (reserved — currently unused by buttons) |
| `tertiary` | Active bg for ghost / outline-active fallback |
| `tertiary-hover` | (reserved) |
| `quaternary` | Default bg for subdue (soft) fill |
| `quaternary-hover` | Hover bg for subdue fill / outline-active |
| `quinary` | (reserved — lightest tier) |
| `quinary-hover` | Hover bg for outline / ghost variant |

**Full token list per tone** (every name exists in `mobile-tokens.css`; `{tone}` = `brandPrimaryButton` · `brandSecondaryButton` · `dangerButton` · `neutralButton` · `disabledButton`):

| Family | Keys |
|---|---|
| `--mobile-surface-{tone}-*` | `default` · `default-hover` · `secondary` · `secondary-hover` · `tertiary` · `tertiary-hover` · `quaternary` · `quaternary-hover` · `quinary` · `quinary-hover` (disabled has no `quinary*`) |
| `--mobile-text-{tone}-*` | `default` · `secondary` · `tertiary` · `quaternary` · `on-*` |
| `--mobile-border-{tone}-*` | `default` · `secondary` · `tertiary` · `quaternary` |
| `--mobile-icon-{tone}-*` | `default` · `secondary` · `tertiary` · `quaternary` · `on-*` |

Text + icon tokens follow the same naming (`text-{tone}-default | secondary | tertiary | quaternary | on-brand | on-default | on-neutral`). `on-*` keys carry the contrast colour painted ON TOP of the matching solid fill (`text-brandPrimaryButton-on-brand` = white in light mode, `#002318` in dark mode).

Focus ring stroke comes from `--mobile-focus-ring-width` (4px); colour is `--mobile-border-brandPrimaryButton-tertiary` regardless of tone (the focus ring's job is to mark _focus_, not _intent_).

> **How to add a new tone**: append the full tier+text+border+icon set to BOTH `:root` and `[data-mobile-theme="dark"]` in mih-design `mobile.css` → `npm run tokens:pull`, then call `buildPalette('newTone')` from the consuming button file. No changes to `_base.jsx` should be needed.

### 7.3. Component map

Each primitive lives in its own file under [src/shared/ui/](src/shared/ui/). Pages can import from `'@/shared/ui'` (barrel) or from `'./MobileShell'` (legacy re-exports — kept so the existing 19 pages don't need to be touched).

**Naming convention** — mobile components use the **same names as the Figma file** (no `Mobile` prefix). In this repo every mobile primitive lives under `src/shared/ui/`; the web counterparts (36/40/44h, radius-150) live in bma.health `src/components/button/`. Import path is the source of truth:

```js
import { BrandButton } from '@/shared/ui';         // barrel (40/48/56/64h, pill)
import { BrandButton } from '@/shared/ui/button';  // explicit button folder
```

#### Button family (Figma App `opz7X9AAKGQf5prHhsPDGn` · Brand Button `231:29` + siblings)

All text buttons live under [src/shared/ui/button/](src/shared/ui/button/). They share `MobileButtonBase` + `createFigmaTextButton` ([_base.jsx](src/shared/ui/button/_base.jsx)) — each file builds a tone palette then exposes the **same Figma property surface**:

| Figma property | Code prop | Values |
|----------------|-----------|--------|
| **Style** | `variant` / `buttonStyle` | `Fill` · `Outline` · `Ghost` (Subdue families: Fill · Outline only) |
| **State** | `state` or `hovered` / `active` / `focused` / `disabled` | `Default` · `Hover` · `Active` · `Focus` · `Disable` |
| **Size** | `size` | `Large` (56) · `Medium` (48) · `Small` (40) — also `lg` / `md` / `sm` |
| **Show Leading icon** | `showLeadingIcon` + `leadingIcon` | boolean + Material Symbol name |
| **Show Tailing icon** | `showTailingIcon` + `trailingIcon` | boolean + Material Symbol name (Figma spelling *Tailing*) |

**Size matrix (Figma App):**

| Size | Height | Icon | Gap | Type (Anuphan) |
|------|--------|------|-----|------|
| Small | 40px | 16 | 6px | `--type-size-button-sm` (14) |
| Medium | 48px | 20 | 8px | `--type-size-button-md` (16) |
| Large | 56px | 24 | 8px | `--type-size-button-lg` (16) |
| X Large | 64px | 24 | 8px | `--type-size-button-xl` (18) |

Padding X = `24px` (`--dim-space-600`) · radius = pill · font = `--type-family-heading-graphic2` (re-pointed to **Anuphan** in `mobile.css` — the app no longer loads Mitr · user 25 Oct 2569) Regular (same as Topic). Fill · Default uses `--mobile-button-shadow-fill` (Brand Drop Shadow Bottom/300).

**State matrix** — every Style resolves to 5 states (`default` / `hover` / `active` / `focus` / `disabled`). The renderer tracks hover/focus/pressed; callers can force a state via props (used by `/m/components`). Focus ring uses CSS `outline` outside the box.

| Component | Figma node | Styles | Notes |
|-----------|-----------|--------|-------|
| [`BrandButton`](src/shared/ui/button/BrandButton.jsx) | [231:29](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=231-29) | Fill · Outline · Ghost | Primary CTA |
| [`BrandSubdueButton`](src/shared/ui/button/BrandSubdueButton.jsx) | [2024:3589](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3589) | Fill · Outline | Soft brand · no Ghost |
| [`DangerButton`](src/shared/ui/button/DangerButton.jsx) | [2024:3590](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3590) | Fill · Outline · Ghost | Destructive |
| [`DangerSubdueButton`](src/shared/ui/button/DangerSubdueButton.jsx) | [2024:3592](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3592) | Fill · Outline | Soft destructive · no Ghost |
| [`NeutralButton`](src/shared/ui/button/NeutralButton.jsx) | [2024:3591](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3591) | Fill · Outline · Ghost | Greyscale secondary |
| [`NeutralSubdueButton`](src/shared/ui/button/NeutralSubdueButton.jsx) | [2024:2940](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-2940) | Fill · Outline | Soft neutral · no Ghost |
| [`BrandIconButton`](src/shared/ui/button/BrandIconButton.jsx) | 377:11596 | pill (square) | Icon-only |
| [`BrandSubdueIconButton`](src/shared/ui/button/BrandSubdueIconButton.jsx) | 2024:4352 | pill (square) | Icon-only · soft brand |
| [`DangerIconButton`](src/shared/ui/button/DangerIconButton.jsx) | 2024:3701 | pill (square) | Icon-only · destructive |
| [`DangerSubdueIconButton`](src/shared/ui/button/DangerSubdueIconButton.jsx) | 2024:4434 | pill (square) | Icon-only · soft destructive |
| [`NeutralIconButton`](src/shared/ui/button/NeutralIconButton.jsx) | 2024:4189 | pill (square) | Icon-only · neutral |
| [`NeutralSubdueIconButton`](src/shared/ui/button/NeutralSubdueIconButton.jsx) | 2024:4516 | pill (square) | Icon-only · soft neutral |
| [`BrandCircleIconButton`](src/shared/ui/button/BrandCircleIconButton.jsx) | 2103:548 | circle (50%) | Force-circular |
| [`BrandCircleSubdueIconButton`](src/shared/ui/button/BrandCircleSubdueIconButton.jsx) | 2103:645 | circle (50%) | Force-circular · soft brand |
| [`BrandSubdueFloatButton`](src/shared/ui/button/BrandSubdueFloatButton.jsx) | 2293:11088 | circle (50%) | FAB · drop-shadow |
| [`HistoryListFormButton`](src/shared/ui/button/HistoryListFormButton.jsx) | 3360:2711 | pill | Wide row · forced Medium |
| [`KeypadButton`](src/shared/ui/button/KeypadButton.jsx) | [314:43835](https://www.figma.com/design/4WXeI8fgBLQGgQ9VheM2yD/-Archive--Mobile-Application?node-id=314-43835) | circle 72 | PIN keypad · Number / icon · Default / Pressed |

#### BrandButton

> Mobile · Figma App node [`231:29`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=231-29) · Live: [`/m/components`](/m/components)

##### Component Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `variant` / `buttonStyle` | `Fill` \| `Outline` \| `Ghost` | `Fill` | Figma **Style** |
| `size` | `Large` \| `Medium` \| `Small` | `Large` | 56 / 48 / 40 px height |
| `state` | `Default` \| `Hover` \| `Active` \| `Focus` \| `Disable` | `Default` | Force state (demos) |
| `showLeadingIcon` | `boolean` | `true` | Show leading icon when `leadingIcon` set |
| `showTailingIcon` | `boolean` | `true` | Show trailing icon when `trailingIcon` set (Figma name) |
| `leadingIcon` / `trailingIcon` | Material Symbol name | — | Icon slots |
| `fullWidth` | `boolean` | `true` | Span container width |
| `disabled` | `boolean` | `false` | Non-interactive |

##### Spacing

| Size | Height | Padding X | Gap | Icon | Type |
|---|---|---|---|---|---|
| Large | 56px | 24px | 8px | 24×24 | 18px regular |
| Medium | 48px | 24px | 8px | 20×20 | 18px regular |
| Small | 40px | 24px | 6px | 16×16 | 16px regular |

```jsx
<BrandButton
  buttonStyle="Fill"
  size="Large"
  leadingIcon="image"
  trailingIcon="chevron_right"
  showLeadingIcon
  showTailingIcon
>
  Button
</BrandButton>
```

#### BrandSubdueButton

> Mobile · Figma [`2024:3589`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3589) · Soft brand · **Style: Fill / Outline only** (no Ghost) · same Size / State / icon toggles as BrandButton.

#### DangerButton

> Mobile · Figma [`2024:3590`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3590) · Destructive · Style: Fill / Outline / Ghost · same Size / State / icon toggles.

#### DangerSubdueButton

> Mobile · Figma [`2024:3592`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3592) · Soft destructive · **Style: Fill / Outline only** · same Size / State / icon toggles.

#### NeutralButton

> Mobile · Figma [`2024:3591`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3591) · Greyscale · Style: Fill / Outline / Ghost · same Size / State / icon toggles.

#### NeutralSubdueButton

> Mobile · Figma [`2024:2940`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-2940) · Soft neutral · **Style: Fill / Outline only** · same Size / State / icon toggles.

**Props (shared via `createFigmaTextButton`):**

```jsx
<BrandButton
  variant="Fill" | "Outline" | "Ghost"   // or buttonStyle=…
  size="Large" | "Medium" | "Small"      // or lg | md | sm
  state="Default"                        // optional force for demos
  leadingIcon="event_available"
  trailingIcon="arrow_forward"
  showLeadingIcon
  showTailingIcon
  disabled
  fullWidth                              // default true
  onClick={...}
>
  Label
</BrandButton>
```

> ⚠️ **Deprecated**: `MobileButton` with `variant='primary' | 'secondary' | 'ghost' | 'danger' | 'neutral'` is kept as a translation shim ([MobileButton.jsx](src/shared/ui/MobileButton.jsx)) so existing `/m/*` pages keep working. New code MUST use the canonical Figma names above.

#### KeypadButton — Archive `314:43835`

> Mobile · [`KeypadButton.jsx`](src/shared/ui/button/KeypadButton.jsx) · Live: [`/m/components#keypad-button`](/m/components#keypad-button)

PIN / numeric keypad key. 72×72 circle (`--dim-size-1600`). Default: 1px `--brand-p400` ring · transparent fill. Pressed: 1.5px `--brand-p300` ring + `--mobile-surface-keypadButton-pressed` (white @ 20%). Digit = Sarabun Bold 24 / on-content. Icon type uses Material `backspace` (Figma Delete).

| Property | Type | Default | Description |
|---|---|---|---|
| `type` | `Number` \| `icon` | `Number` | Digit vs delete |
| `state` | `Default` \| `Pressed` | interactive | Force for demos |
| `digit` / `children` | `ReactNode` | — | Number label |
| `icon` | Material Symbol | `backspace` | When `type=icon` |

```jsx
<KeypadButton type="Number" digit="1" onClick={() => press('1')} />
<KeypadButton type="icon" onClick={backspace} aria-label="ลบ" />
```

#### Primitives

| Component | File | Web equivalent | What differs on mobile |
|-----------|------|----------------|------------------------|
| [`Password`](src/shared/ui/Password.jsx) | Password.jsx | `PasswordField` (§6) | Figma App [`5021:1008`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=5021-1008) · pill · lock + eye · Large 56 / Medium 48 |
| [`Textarea`](src/shared/ui/Textarea.jsx) | Textarea.jsx | `Textarea` (§6.12) | Figma App [`268:26`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=268-26) · min-h 96 · radius-200 · Default/Press/Filled/Error/Disabled |
| [`RadioButton`](src/shared/ui/RadioButton.jsx) | RadioButton.jsx | `RadioButton` (§6.21) | Figma App [`2860:3497`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2860-3497) · card radio · min-h 56 · radius-200 · inset ring 1px default / 2px brand selected · inline icon slot · `RadioButtonGroup` = equal-column grid (ตรวจเต้านมด้วยตนเอง) |
| [`SuccessCard`](src/shared/ui/SuccessCard.jsx) | SuccessCard.jsx | — | Archive [`178:122765`](https://www.figma.com/design/4WXeI8fgBLQGgQ9VheM2yD/?node-id=178-122765) · teal gradient + arch card (radius-1000 / 600) · award hero · optional rows / dual CTA |
| [`MobileCard`](src/shared/ui/Card.jsx) | Card.jsx | `Card` (web) | Radius 16 vs 8; subtle elevation opt-in |
| [`ServiceRequestCard`](src/features/requests/components/ServiceRequestCard.jsx) | ServiceRequestCard.jsx | — | Archive [`194:153209`](https://www.figma.com/design/4WXeI8fgBLQGgQ9VheM2yD/?node-id=194-153209) · 160w · radius-600 · Colorful icon Badge pill · Home ส่งคำขอ |
| [`MobileInput`](src/shared/ui/Input.jsx) | Input.jsx | `InputField` (§6.12) | 16px base font (iOS zoom guard); 48h; pill-leaning radius |
| [`MobileField`](src/shared/ui/Field.jsx) | Field.jsx | `FormControl` wrapper | Stacks label / hint / error around any control |
| [`MobileBadge`](src/shared/ui/Badge.jsx) | Badge.jsx | soft tone chips | countdown / distance pills — prefer ColorfulBadge for categorical tags |
| [`ColorfulBadge`](src/shared/ui/badge/ColorfulBadge.jsx) | shared web | Figma App [`2076:8413`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2076-8413) · 11 styles × Small/Default × Default/Outline × Primary/Secondary |
| [`MobileHeader`](src/shared/ui/Header.jsx) | Header.jsx | `Header` (§6.19) | 56h app-bar pattern; back arrow + title + right-slot; `transparent` mode over hero gradient |
| [`MobileSectionLabel`](src/shared/ui/SectionLabel.jsx) | SectionLabel.jsx | — | Section header above content blocks |
| [`MobileShell`](src/shared/ui/MobileShell.jsx) | MobileShell.jsx | — | Phone frame + status bar + bottom nav + auth gate |
| [`Sym`](src/shared/ui/Sym.jsx) | Sym.jsx | `Sym` (web) | Same shape, separate file so the mobile tree has no web-only coupling |
| [`TimelineItem`](src/shared/ui/TimelineItem.jsx) · `TimelineRow` | TimelineItem.jsx | Figma New Theme `2790:109178` | Date-sorted list (appointments · history · vaccines · home visits): left date/time/icon on a rail · right small card (centre · clinic · doctor · label/value · indicator icons) · `TimelineRow lines={2}` cuts a long value at 2 lines + "…" (home-visit summary) |
| [`TimeSlotList`](src/features/appointments/components/TimeSlotList.jsx) | TimeSlotList.jsx | — | Pick a time: hourly rows "09:00 - 10:00 น. · ว่าง N คิว" → action sheet (`BookingSheet`) with the 15-min slots |
| [`ListRow`](src/shared/ui/ListRow.jsx) · `ListSection` | ListRow.jsx | — | Divider list row (icon · title · 1-line summary · right value/status · chevron) + section heading with summary |
| [`PageSection`](src/shared/ui/PageSection.jsx) · `InfoGrid` | PageSection.jsx | — | Frameless page section (heading + 1px divider) + aligned label/value grid |
| [`BottomActionBar`](src/shared/ui/BottomActionBar.jsx) | BottomActionBar.jsx | sticky footer | Bottom button bar (column \| row · sticky · safe area) — §7.3.11 |
| [`BottomSheet`](src/shared/ui/BottomSheet.jsx) · `SheetCloseButton` | BottomSheet.jsx | `Popup` | The one slide-up sheet of the app: ‹ / centred title / × · footer · variant page · portal — §7.3.12 |
| [`EmptyState`](src/shared/ui/EmptyState.jsx) | EmptyState.jsx | empty state | Icon 48 → title → text → action · framed / compact — §7.3.13 |
| [`InlineNotice`](src/shared/ui/InlineNotice.jsx) | InlineNotice.jsx | `Alert` (§6) | Compact alert: icon + text, 4 tones, md / sm, optional × — §7.3.14 |
| [`Divider`](src/shared/ui/Divider.jsx) | Divider.jsx | — | 1px solid / dashed line, optional centred label ("หรือ") — §7.3.15 |
| [`AmountRow`](src/features/payment/components/AmountRow.jsx) · `AmountRows` | AmountRow.jsx | — | Label left / amount right, total row (`strong`), value tone — §7.3.16 |
| [`StepProgress`](src/shared/ui/StepProgress.jsx) | StepProgress.jsx | `Stepper` | "ขั้นตอนที่ n จาก N" + segmented / continuous bar — §7.3.17 |
| [`Avatar`](src/shared/ui/Avatar.jsx) | Avatar.jsx | `Avatar` (§6.11) | Photo circle → initial → icon fallback · ring — §7.3.18 |
| [`CenterDialog`](src/shared/ui/CenterDialog.jsx) | CenterDialog.jsx | `AlertDialog` (§6) | Centred popup: illustration / icon → title → text → Fill + Outline · danger — §7.3.19 |
| [`ChoiceButtonGroup`](src/shared/ui/ChoiceButtonGroup.jsx) | ChoiceButtonGroup.jsx | `SelectableCardButton` | Pick one answer with pill buttons (Fill selected · Subdue outline) · row \| column — §7.3.20 |
| [`Toast`](src/shared/ui/Toast.jsx) · `useToast` | Toast.jsx | `SuccessToast` / `PhcisTopToast` | Short floating message above the bottom bar, auto-hides 1.8 s, wraps — §7.3.21 |
| [`Tabs`](src/shared/ui/Tabs.jsx) | Tabs.jsx | `TabUnderline` | Underline tabs, 48 tall, count, scroll inside the bar — §7.3.22 |

> 📝 **Naming exception**: Primitives that are conceptually "mobile shell" (Card, Input, Field, Badge, Header, SectionLabel, Shell) keep the `Mobile*` prefix because there is no Figma master of these yet — the prefix marks them as mobile-only until a Figma source-of-truth exists. Buttons, **Password**, and **Textarea** drop the prefix because their Figma App masters (`319:2012` / `5021:1008` / `268:26`) ARE the source-of-truth. **SuccessCard** drops the prefix as a shared flow template (Archive success screens), not a shell chrome piece.

#### 7.3.1 MobileButton — props & variants

```jsx
<MobileButton
  variant='primary' | 'secondary' | 'ghost' | 'danger' | 'neutral'
  size='md' | 'lg'           // default 'lg' (56h)
  icon='event_available'     // optional leading Material Symbol
  iconRight='arrow_forward'  // optional trailing
  disabled
  fullWidth                  // default true — mobile CTAs span the safe area
  onClick={...}
>
  จองนัด
</MobileButton>
```

| Variant | Surface token | Text token | Border |
|---------|---------------|------------|--------|
| `primary` | `--surface-brandPrimary-default` | `#FFFFFF` | transparent |
| `secondary` | `--surface-brandPrimary-quaternary` | `--text-brandPrimary-default` | `--border-brandPrimary-default` |
| `ghost` | transparent | `--text-brandPrimary-default` | transparent |
| `danger` | `--surface-warning-default` | `#FFFFFF` | transparent |
| `neutral` | `--surface-content-default` | `--text-content-primary` | `--border-content-tertiary` |

#### 7.3.2 MobileCard — props

```jsx
<MobileCard
  padding='var(--dim-space-500, 20px)'   // override default 16
  onClick={...}                          // makes the card a button (role/keyboard)
  interactive                            // visual cursor: pointer without onClick
  elevated                               // adds --shadow-brand-drop-bottom-200
>
  …
</MobileCard>
```

#### 7.3.3 Password — Figma App `5021:1008`

> Mobile · [`Password.jsx`](src/shared/ui/Password.jsx) · Live: [`/m/components#mobile-password`](/m/components#mobile-password)

| Property | Type | Default | Description |
|---|---|---|---|
| `size` | `Large` \| `Medium` \| `lg` \| `md` | `lg` | 56 / 48 px height |
| `state` | `default` \| `press` \| `filled` \| `error` \| `disabled` | derived | Force for demos |
| `showFieldName` | `boolean` | `true` | Show label |
| `required` | `boolean` | `false` | Show `*` |
| `showLeadingIcon` | `boolean` | `true` | Leading `lock` (override via `leadingIcon`) |
| `showPassword` | `boolean` | `true` | Eye toggle (`visibility` / `visibility_off`) |
| `helperText` / `error` | `string` | — | Helper under field; `error` forces Error state |

```jsx
<Password
  label="รหัสผ่าน"
  required
  size="Large"
  value={pw}
  onChange={(e) => setPw(e.target.value)}
  placeholder="กรอกรหัสผ่าน"
  helperText="อย่างน้อย 8 ตัวอักษร"
/>
```

#### 7.3.4 Textarea — Figma App `268:26`

> Mobile · [`Textarea.jsx`](src/shared/ui/Textarea.jsx) · Live: [`/m/components#mobile-textarea`](/m/components#mobile-textarea)

| Property | Type | Default | Description |
|---|---|---|---|
| `state` | `default` \| `press` \| `filled` \| `error` \| `disabled` | derived | Force for demos |
| `showFieldName` | `boolean` | `true` | Show label |
| `required` | `boolean` | `false` | Show `*` |
| `helperText` / `error` | `string` | — | Helper under field; `error` forces Error state |
| `minHeight` | `number` | `96` | Box min-height (px) |
| `rows` | `number` | `3` | Native rows |
| `resize` | CSS resize | `none` | Override if needed |

```jsx
<Textarea
  label="อาการเบื้องต้น / หมายเหตุ"
  required
  value={note}
  onChange={(e) => setNote(e.target.value)}
  placeholder="Placeholder"
  helperText="Helper text"
/>
```

#### 7.3.5 SuccessCard — Archive `178:122765`

> Mobile · [`SuccessCard.jsx`](src/shared/ui/SuccessCard.jsx) · Live: [`/m/components#mobile-success-card`](/m/components#mobile-success-card)

Reusable post-flow success template: teal gradient shell + white arch card (top `--dim-radius-1000`, bottom `--dim-radius-600`) + award hero + title + optional description / info rows + primary `BrandButton` (+ optional `BrandSubdueButton` Outline).

Used by register identity success, **register complete** ([Archive `171:70337`](https://www.figma.com/design/4WXeI8fgBLQGgQ9VheM2yD/-Archive--Mobile-Application?node-id=171-70337)), and booking success. Export `SUCCESS_CARD_BG` for `MobileShell` `background` when chrome is hidden.

| Property | Type | Default | Description |
|---|---|---|---|
| `title` | `ReactNode` | — | Brand heading |
| `description` | `ReactNode` | — | Supporting copy under title |
| `rows` | `{ label, value }[]` | — | Soft brand summary card |
| `primaryAction` / `secondaryAction` | `{ label, onClick, showTailingIcon?, … }` | — | CTA(s); primary defaults chevron |
| `heroSrc` / `heroSize` | `string` / `number` | award PNG / `160` | Hero illustration |
| `fill` | `boolean` | `true` | `flex:1` + min-height 100% |
| `background` | `string` | `SUCCESS_CARD_BG` | Gradient override |

```jsx
<SuccessCard
  title="ยืนยันตัวตนสำเร็จ"
  rows={[
    { label: 'ชื่อ–สกุล', value: fullName },
    { label: 'เลขประจำตัวประชาชน', value: formatCid(cid) },
  ]}
  primaryAction={{ label: 'ดำเนินการต่อ', onClick: onNext }}
/>

{/* Archive 171:70337 — register complete */}
<SuccessCard
  title="ลงทะเบียนสำเร็จ"
  description="ยินดีต้อนรับเข้าสู่แอปพลิเคชัน PHCIS ข้อมูลของคุณถูกบันทึกเข้าระบบเรียบร้อยแล้ว"
  primaryAction={{ label: 'เข้าสู่ระบบ', onClick: enterApp }}
/>

<SuccessCard
  title="สร้างนัดหมายสำเร็จ"
  description={<>ระบบได้บันทึก…</>}
  primaryAction={{ label: 'ดูการนัดหมาย', showTailingIcon: false, onClick: goAppts }}
  secondaryAction={{ label: 'กลับสู่หน้าหลัก', onClick: goHome }}
/>
```

#### 7.3.6 MobileInput + MobileField — pattern

```jsx
<MobileField label='เลขประจำตัวประชาชน' required hint='13 หลัก' error={errors.cid}>
  <MobileInput inputMode='numeric' maxLength={13} value={cid} onChange={…} />
</MobileField>
```

`MobileField` accepts **any control** as children — `MobileInput`, `<textarea>`, `<select>`, or a custom picker. The label / hint / error treatment is the same.

#### 7.3.7 MobileBadge — tones

`tone` = `'neutral'` | `'brand'` | `'success'` | `'warning'` | `'info'`. Soft countdown / distance chips. For categorical tags use **ColorfulBadge** (Figma App `2076:8413`) instead.

#### 7.3.8 ColorfulBadge — Figma App `2076:8413`

> Shared · [`ColorfulBadge.jsx`](src/shared/ui/badge/ColorfulBadge.jsx) · Live: [`/m/components#mobile-colorful-badge`](/m/components#mobile-colorful-badge) · Web: `/components`

11 styles × Small (24) / Default (32) × Default / Outline × Primary (`*-bold` + badge-on) / Secondary (100 + 700). Toggles: Show Dot · Show Icon · Show Text.

```jsx
<ColorfulBadge style="green" size="small" hierarchy="secondary" showDot icon="pill">
  Text
</ColorfulBadge>
<ColorfulBadge style="green" size="small" hierarchy="primary" showDot>Text</ColorfulBadge>
<ColorfulBadge style="grey" size="small" fill="outline" showDot>Text</ColorfulBadge>
```

#### 7.3.9 MobileHeader — props

```jsx
<MobileHeader
  title='นัดหมาย'
  subtitle='TOR §4.2'      // optional small line beneath title
  onBack={...}              // defaults to history.goBack()
  right={<BellIcon />}      // any node — usually an icon-button or badge
  hideBack                  // omit the back arrow (e.g. tab-root pages)
  transparent               // for hero screens that own their background
/>
```

#### 7.3.10 MobileShell — frame

```jsx
<MobileShell
  showStatusBar={true}      // hide for full-bleed onboarding
  showBottomNav={true}      // hide on auth screens / wizard steps
  requireAuth={false}       // public pages (login, register, reset)
  background='var(--surface-content-secondary)'
>
  <MobileHeader title='…' />
  {/* page body */}
</MobileShell>
```

The shell wraps the page in a 402×874 iPhone 17 mock so desktop reviewers can demo the citizen flow. On viewports ≤ 440 wide the chrome (bezel · notch · side keys) collapses and the app fills the viewport — `.mih-phone-stage / -device / -screen / -chrome` rules in `src/index.css` (คทง.12 §6 no horizontal scroll at 360 · 390 · 430).

#### 7.3.11–7.3.18 Common primitives (ผู้ใช้ 22 ต.ค. 69)

Seven of these were hand-drawn again and again across pages; `BottomSheet` was already made the single sheet of the app and is listed so the set is complete (counts from a scan of every `src/features/*` + `src/shared/ui/*` file). New screens use the shared component; old copies move to it when their page is touched. All are exported from `@/components/mobile`; live demos at `/m/components?component=<slug>`.

| # | Component | Slug | Replaces (copies found) | Key props |
|---|-----------|------|-------------------------|-----------|
| 7.3.11 | [`BottomActionBar`](src/shared/ui/BottomActionBar.jsx) | `mobile-bottom-action-bar` | `Footer` · `StickyFooter` · `BookingFooter` + inline footers (~20) | `direction` column \| row · `sticky` · `safeArea` · `border` brand \| neutral \| none · `shadow` |
| 7.3.12 | [`BottomSheet`](src/shared/ui/BottomSheet.jsx) (`BookingSheet` = alias) · `SheetCloseButton` · `SHEET_TITLE` | `mobile-bottom-sheet` | the one sheet of the whole app — every sheet already moved to it (ผู้ใช้ 21–22 ต.ค. 69 · `.claude/rules/mobile-ui-layout.md`) | `open` · `title` · `onClose` · `onBack` · `footer` · `headerRight` · `padded` · `dismissOnOverlay` · `variant` sheet \| page · `zIndex` · portals into `.mih-phone-screen` |
| 7.3.13 | [`EmptyState`](src/shared/ui/EmptyState.jsx) | `mobile-empty-state` | `EmptyState` · `EmptyTimeline` + "ไม่พบ…" screens (~14) | `icon` (null = none) · `illustration` (Lottie/ภาพแทนไอคอน · ไม่มียอดค้างชำระ) · `tone` · `title` · `description` · `action` · `framed` · `compact` · `titleSize` (ค่าตั้งต้น base · ไม่มียอดค้างชำระ = xl) |
| 7.3.14 | [`InlineNotice`](src/shared/ui/InlineNotice.jsx) | `mobile-inline-notice` | `ErrorAlert` · `ErrorNote` · `InfoAlert` + inline alert boxes (~10) | `tone` info \| success \| warning \| error · `title` · `icon` · `size` md \| sm · `action` · `onClose` |
| 7.3.15 | [`Divider`](src/shared/ui/Divider.jsx) | `mobile-divider` | `SettingsList` Divider + inline 1px / dashed lines (~9) | `variant` solid \| dashed · `tone` neutral \| brand \| strong · `label` · `spacing` |
| 7.3.16 | [`AmountRow`](src/features/payment/components/AmountRow.jsx) · `AmountRows` | `mobile-amount-row` | `CostRow` · `InfoRow` · `AmountRow` · receipt `Row` (~8) | `label` · `value` · `sub` · `strong` · `tone` danger \| success \| brand · `size` sm \| xs |
| 7.3.17 | [`StepProgress`](src/shared/ui/StepProgress.jsx) | `mobile-step-progress` | `StepBar` · `BookingStepper` · `RequestStepProgress` · `ProgressBar` (5) | `current` (1-based) · `total` · `label` · `caption` · `variant` segmented \| continuous |
| 7.3.18 | [`Avatar`](src/shared/ui/Avatar.jsx) | `mobile-avatar` | `ListAvatar` + home / doctor / switcher / profile photo circles (7) | `src` · `name` (first letter fallback, title prefix stripped) · `fallbackIcon` · `size` · `ring` white \| brand |

Rules that apply to all eight:

- **Which one, when.** Buttons at the bottom of a page or sheet → `BottomActionBar` (one Fill CTA, the rest Subdue). Any slide-up panel → `BottomSheet` (never re-draw the overlay; do not stack sheets — change the content and use `onBack`). Nothing to show → `EmptyState`. A message inside the page → `InlineNotice` (the web `Alert` stays for full-width Figma 297:59 alerts only). Money rows → `AmountRow` (right-aligned value); other label/value pairs → `InfoGrid` (left-aligned two columns).
- **Tokens.** Sheet panel `--surface-card-100` + top radius `--dim-radius-600` + `--shadow-brand-top-200`, up to 92% tall · overlay `--surface-overlay-default` (black 50% in the app) · notice `--surface-/--border-/--text-alert-{tone}` · bar border `--border-brandPrimary-quaternary` + `--shadow-brand-drop-top-100` · step fill `--surface-brandPrimary-default` / current `--surface-positive-default` / rest `--surface-brandPrimary-tertiary` (Archive `2119:51183`).
- **Touch.** `InlineNotice` × is 44×44. `SheetCloseButton` and the sheet ‹ are 44×44 hit areas; the visible circle stays 40 (Figma) — ผู้ใช้ 22 ต.ค. 69 ข้อ 2ก.
- **Close.** `BottomSheet` closes on ×, tap on the overlay (unless `dismissOnOverlay={false}`), and Esc.

```jsx
<BottomSheet title='เลือกขนาดตัวอักษร' onClose={close}
  footer={<BrandButton fullWidth onClick={save}>บันทึก</BrandButton>}>
  …
</BottomSheet>

<BottomActionBar sticky safeArea>
  <BrandButton fullWidth>ยืนยันนัดหมาย</BrandButton>
  <BrandSubdueButton fullWidth>ย้อนกลับ</BrandSubdueButton>
</BottomActionBar>

<AmountRows>
  <AmountRow label='ค่ายา' value='฿320.00' />
  <AmountRow label='สิทธิการรักษา' value='-฿320.00' tone='success' />
  <Divider variant='dashed' tone='strong' />
  <AmountRow label='ยอดที่ต้องชำระ' value='฿0.00' strong />
</AmountRows>

<StepProgress current={2} total={4} label='ยืนยันตัวตน' />
<EmptyState icon='event_busy' title='ยังไม่มีนัดหมาย' description='นัดหมายที่จองไว้จะแสดงที่นี่' framed />
<InlineNotice tone='warning' size='sm'>ยาบางรายการต้องเก็บในตู้เย็น</InlineNotice>
<Avatar src={user.avatar} name={user.name} size={48} ring='white' />
```

**Not migrated yet** (copies still in pages — move them when the page is next touched; sheets are already done): footers in Telemed · PharmacyPayment · Consent · Register; empty states in Book · Appointments · Records; step bars in Register · BookingWizard · RequestDetailSheet · SelfAssessment.

#### 7.3.19–7.3.22 Common primitives รอบ 2 (ผู้ใช้ 23 ต.ค. 69)

Second scan of every `src/features/*` + `src/shared/ui/*` file (23 ต.ค. 69): four more patterns were still hand-drawn in several places. Search fields were checked too and are already shared (`BookingSearchRow` · `MobileInputField leadingIcon='search'`), so no new search component.

| # | Component | Slug | Replaces (copies found) | Key props |
|---|-----------|------|-------------------------|-----------|
| 7.3.19 | [`CenterDialog`](src/shared/ui/CenterDialog.jsx) | `mobile-center-dialog` | `CameraPermissionPopup` · `LocationPermissionPopup` · `ToggleConfirmPopup` (ตั้งค่า) · `MicrophonePermissionPopup` · `CancelAppointmentPopup` · `ConsentDeclinedPopup` (Telemed) (6) | `illustration` (96) \| `icon` + `iconTone` (badge 64) · `title` · `children` · `primaryAction` / `secondaryAction` `{label, onClick, icon?, disabled?}` · `danger` · `onClose` · `dismissOnOverlay` · `inline` |
| 7.3.20 | [`ChoiceButtonGroup`](src/shared/ui/ChoiceButtonGroup.jsx) | `mobile-choice-button-group` | `ConsentChoicePair` · self-assessment answers · OTP channel · medical-certificate yes/no on Book · RatingSheet / BseSelfRecordSheet choices (~6) | `options=[{value,label,icon?,disabled?}]` · `value` · `onChange` · `label` · `layout` row \| column · `size` |
| 7.3.21 | [`Toast`](src/shared/ui/Toast.jsx) · `useToast` | `mobile-toast` | floating pills in Consent · Settings · Profile (raw `rgba` / `#fff` / 12px + `nowrap` that overflowed at 360) (3) | `useToast(duration=1800)` → `{show(message, icon?), hide, node}` · `Toast` `message` · `icon` · `bottom` · `inline` |
| 7.3.22 | [`Tabs`](src/shared/ui/Tabs.jsx) | `mobile-tabs` | `SegmentTab` on `/m/records` — §7.4 listed Tabs as missing (1) | `tabs=[{key,label,count?}]` · `value` · `onChange` · `label` |
| 7.3.26 | [`ChartTapReadout`](src/shared/ui/ChartTapReadout.jsx) · `viewBoxX` · `nearestIndex` | `mobile-chart-tap-readout` | กล่องลอยบนกราฟการเจริญเติบโต (กรมอนามัย · TSPE · รอบศีรษะ) — ตัวเลขย้ายมาใต้กราฟ (ผู้ใช้ 26 ต.ค. 69) (3) | `title` (ว่าง = ข้อความสีชมพู `hint` "แตะบนกราฟเพื่อดูค่า" + ไอคอน touch_app) · `measured {value, date}` · `measuredLabel` · `rows=[[label, value]]` · `unit` · กล่อง `--surface-brandPrimary-quaternary` radius-300 · aria-live |

Which one, when:

- **Centred popup vs sheet.** Ask for a permission (camera · location · microphone), confirm something that can't be undone, or a result the user must acknowledge → `CenterDialog`. Anything with fields, a list, or more than ~3 lines → `BottomSheet` (§7.3.12). One Fill button (top) + one Outline (bottom); `danger` turns the title and Fill red (`DangerButton`) and sets `role='alertdialog'`.
- **Pick one answer.** 2 short options (มี/ไม่มี · ยินยอม/ไม่ยินยอม) → `ChoiceButtonGroup layout='row'` (equal width). 3+ options or long text → `layout='column'` (left aligned). Options that need a description line → `RadioButtonGroup`. Filtering a list → `FilterChip`.
- **Short feedback.** Result of a tap that needs no decision ("คัดลอกแล้ว" · "บันทึกการตั้งค่าแล้ว" · "ยังไม่เปิดใช้") → `useToast().show(...)`, ≤ 2 lines, gone after 1.8 s. Needs a decision → `CenterDialog`. Success of a whole task → `SuccessCard`. A message that must stay on the page → `InlineNotice`.
- **Switch views of one page.** 3+ views → `Tabs` (scrolls sideways inside its own bar, never widens the page). 2 views or filters → `FilterChip` / `ChoiceButtonGroup`.

Tokens: dialog `--surface-card-100` · border `--border-popover-grey` · radius `--dim-radius-500` · `--shadow-drop-bottom-400` · max width `--dim-size-2600` (384) · overlay `--surface-overlay-default` · title `--type-size-xl` `--text-brandPrimary-default` (danger `--text-danger-default`) · body `--type-size-sm` `--text-content-tertiary`. Toast `--surface-neutralButton-default` + `--text-content-on-content` (inverts in dark mode) · `--type-size-sm` · pill · max width = screen − `--dim-space-800`. Tabs underline `--dim-stroke-400` (4px, same as web `TabUnderline`) `--border-brandPrimary-default` · bar line `--border-brandPrimary-quaternary` · height `--dim-size-1000` (48).

Touch: every choice button and tab is ≥ 48 tall; dialog buttons are Large (56). `CenterDialog` closes on overlay tap (unless `dismissOnOverlay={false}`) and Esc. `CenterDialog` and `Toast` portal into `.mih-phone-screen`; `inline` draws them inside the nearest positioned parent (docs page only).

```jsx
<CenterDialog
  illustration='/illustration/mobile/camera-permission-hero.png'
  title='อนุญาตให้เข้าถึงกล้อง'
  primaryAction={{ label: 'อนุญาต', onClick: allow }}
  secondaryAction={{ label: 'ไม่อนุญาต' }}
  onClose={deny}
>
  แอปพลิเคชันจำเป็นต้องเข้าถึงกล้องถ่ายรูปของคุณ เพื่อสแกนใบหน้ายืนยันตัวตน
</CenterDialog>

<ChoiceButtonGroup label='ความยินยอม' value={consent} onChange={setConsent}
  options={[{ value: 'yes', label: 'ยินยอม' }, { value: 'no', label: 'ไม่ยินยอม' }]} />

const toast = useToast();
toast.show('คัดลอกเลขที่คำขอแล้ว', 'content_copy');
return <>{page}{toast.node}</>;

<Tabs label='ประวัติ' value={tab} onChange={setTab}
  tabs={[{ key: 'treatment', label: 'ประวัติการรักษา' }, { key: 'vaccines', label: 'วัคซีนที่เคยได้รับ', count: 12 }]} />
```

**Not migrated yet** (move when the page is next touched): `CameraPermissionPopup` · `LocationPermissionPopup` (keep their file names as thin wrappers) · Settings `ToggleConfirmPopup` · Telemed `MicrophonePermissionPopup` · `CancelAppointmentPopup` · `ConsentDeclinedPopup` → `CenterDialog`; `ConsentChoicePair` · self-assessment answers · Register OTP channel · Book certificate yes/no · RatingSheet · BseSelfRecordSheet → `ChoiceButtonGroup`; floating toasts in Consent · Settings · Profile → `useToast` (the BreastSelfExam / Records messages are inline status lines, not toasts — keep); `/m/records` `SegmentTab` → `Tabs`.

#### SettingsList · HomeMenuSheet (คทง.12 HOME-03 · THEME-03)

| Export | File | Use |
|---|---|---|
| `Card` · `SectionTitle` · `Divider` · `NavRow` · `ToggleRow` · `SettingsSheet` | [`SettingsList.jsx`](src/features/profile/components/SettingsList.jsx) | rows + bottom sheet shared by `/m/settings` and the ☰ menu — one copy only; rows ≥ 44px, label `--type-size-base`, switch has a 44×44 tap area |
| `FontSizeRow` | same | "ขนาดตัวอักษร" เล็ก / ปกติ / ใหญ่ (`FilterChip`) → `setFontSize` (`src/shared/lib/mobileFontSize.js`, `<html data-font-size>`) |
| `INFO_SHEETS` · `InfoBody` · `ContactBody` · `CONTACT_CHANNELS` | same | ข้อกำหนดบริการ · คำถามที่พบบ่อย (FAQ) · ติดต่อเจ้าหน้าที่ (Line UMSC · โทร) |
| `usePersistedSettings` | same | `phcis_mobile_settings_v1` store (camera · microphone · notifications · news) |
| `HomeMenuSheet` | [`HomeMenuSheet.jsx`](src/features/home/components/HomeMenuSheet.jsx) | ☰ on หน้าแรก: ตั้งค่า (กล้อง · ไมโครโฟน · ขนาดตัวอักษร) → บัญชีผู้ใช้ (แก้ไขข้อมูล · สลับผู้ใช้งาน → `AccountSwitcherSheet`) → ความช่วยเหลือ → footer `DangerButton` outline "ออกจากระบบ" |
| `AccountSwitcherLauncher` | [`AccountSwitcherLauncher.jsx`](src/features/auth/components/AccountSwitcherLauncher.jsx) | `AccountSwitcherSheet` + `AddAccountSheet` pre-wired (accounts · PIN unlock via `/m/login?switch=<id>&next=<current>` · remove · add) — open from any page with `open` / `onClose` only (AUTH-06) |

#### Appointment requests · RatingSheet (คทง.12 APPT-02/04/05 · EVAL-01)

| Export | File | Use |
|---|---|---|
| `AppointmentRequestStatus` | [`AppointmentRequestStatus.jsx`](src/features/appointments/components/AppointmentRequestStatus.jsx) | คำขอเลื่อน/ยกเลิกนัด: "ขอเลื่อนนัด" / "ขอยกเลิกนัด" + `StatusBadge` รออนุมัติ (warning) · อนุมัติแล้ว (success) · ปฏิเสธ (error) → label/value วัน-เวลาที่ขอ · เหตุผล — การ์ดนัด · รายละเอียดนัด |
| `AppointmentRulesList` · `LabelValueRows` | same | กติกานัดหมาย (`BOOKING_RULES`) เป็น label/value 2 คอลัมน์ — หน้ายืนยันจอง · รายละเอียดนัด |
| `RequestReasonPicker` · `reasonText` | same | ตัวเลือกเหตุผล + "อื่นๆ (โปรดระบุ)" เยื้องใต้ตัวเลือก — ชีตขอยกเลิกนัด · หน้าขอเลื่อนนัด · ขอเยี่ยมบ้าน |
| `RatingSheet` | [`RatingSheet.jsx`](src/features/rating/components/RatingSheet.jsx) | ดาว 5 ระดับ (บังคับ) · คะแนนรายด้านเป็นปุ่มดาววงกลม `BrandCircleSubdueIconButton` md (ไม่บังคับ) · ข้อเสนอแนะไม่บังคับ · "ประเมินภายหลัง" — ส่งแล้ว = หน้า success เต็มจอ (`SuccessCard` + Lottie `feedback-rating-recolored.json`) · `onSubmit` ยิงตอนกด "เสร็จสิ้น" — ประเมินแอป + ประเมินบริการ ศบส. (Onsite · Telemed) |
| `RatingNudge` | [`RatingNudge.jsx`](src/features/rating/components/RatingNudge.jsx) | `AppNotification` ย่อบนหน้าแรก แตะแล้วเปิด `RatingSheet` — บริการ Onsite ที่ยังไม่ประเมิน ก่อน · แอปเดือนละครั้ง (`src/features/rating/lib/mobileRatings.js`) |

#### Carousel dots · Booking/pay blocks · Scan · Vaccine photos (ผู้ใช้ 23 ต.ค. 69)

| Export | File | Use |
|---|---|---|
| `CarouselDots` | [`CarouselDots.jsx`](src/shared/ui/CarouselDots.jsx) | จุดบอกหน้าใต้แถบเลื่อนแนวนอน · `count` · `index` · ชมพูธีม 6px หน้าปัจจุบันทึบ ที่เหลือ 35% · หน้าเดียว = ไม่แสดง — ใช้ที่: แบนเนอร์หน้าแรก (เลื่อนทีละแผ่น) · quick menu หน้าแรก (เลื่อนอิสระ: จำนวน = ความกว้างที่เลื่อนได้ ÷ ความกว้างจอ) |
| `UnpaidBookingBlockSheet` · `pendingBills` | [`UnpaidBookingBlockSheet.jsx`](src/features/appointments/components/UnpaidBookingBlockSheet.jsx) | แผ่นดัก "ยังจองนัดหมายไม่ได้" เมื่อมียอดค้างชำระ · Lottie `cash-money-recolored.json` · N รายการ · รวม X บาท · ปุ่ม "ชำระเงิน" (`onPay` → `/m/payment`) + "ไว้ภายหลัง" — ใช้ที่: ทางลัด "จองนัดหมาย" หน้าแรก · ด่านใน `/m/book` (ทุกทางเข้า · `?reschedule` ไม่ติด) · แบบเดียวกับ `TelemedNewPatientSheet` |
| `UnpaidBillsSheet` | [`UnpaidBillsSheet.jsx`](src/features/payment/components/UnpaidBillsSheet.jsx) | รายการบิลค้าง แตะบิล = `onPick(bill)` → `PaymentDetailSheet` · `title` · `intro` · `footer` — แบนเนอร์ยอดค้างหน้าแรก |
| `MedPickupSection` · `DeliveryMethodSheet` · `PostalAddressSheet` · `MedicationRow` | [`MedPickupDetails.jsx`](src/features/pharmacy/components/MedPickupDetails.jsx) | "รายละเอียดการรับยา" (ปุ่ม "เพิ่มรายละเอียด" → แผ่น "ช่องทางรับยา" ศูนย์บริการ \| ไปรษณีย์ → ที่อยู่) · `value={method, note, address}` · `onChange` · `sheetZIndex` (ซ้อนบนแผ่นอื่นให้สูงกว่า) — ใช้ที่: `/m/pharmacy/payment` · บิลที่มี `bill.meds` ใน `PaymentDetailBody` (ต้องเลือกช่องทางรับยาก่อน ช่องทางชำระจึงกดได้ · ผู้ใช้ 27 ต.ค. 69) |
| `QrScanSheet` | [`QrScanSheet.jsx`](src/features/home/components/QrScanSheet.jsx) | ตัวสแกน QR เป็น bottom sheet · ไม่ส่ง `modes` = เปิดกล้องทันที, `resolveScan()` คืน `foundTitle` / `foundActionLabel` / `autoProceed` ของ QR ที่เจอ (ถุงยา → "ยืนยันการรับยา" · ใบยินยอม → เปิดฟอร์มทันที) · ปุ่มสแกนอยู่หัวเขียวหน้าแรกหน้ากระดิ่ง |
| `AddVaccineSheet` | [`AddVaccineSheet.jsx`](src/features/records/components/AddVaccineSheet.jsx) | เพิ่มวัคซีน = ส่งรูปอย่างเดียว (ถ่าย/แนบ ≤ 6 รูป · ลบทีละรูป) ไม่มีชนิด/วันที่ · `onSave({ photos })` → `selfVaccines.addSelfVaccinePhotos` · การ์ด "รูปที่ส่งให้เจ้าหน้าที่ · รอตรวจสอบ" เหนือตาราง (PHCIS ลงเข็มให้ที่ `MobileVaccinePhotosPanel`) |

#### Lab results · Home BP (คทง.12 ประชุม 19 ต.ค. 69 · EMR-09/10/11 · HBP-01)

| Component | File | Contract |
|---|---|---|
| `LabResults` | [`LabResults.jsx`](src/features/records/components/LabResults.jsx) | ผลแล็บจัดหมวด (เลือด · เคมีคลินิก · ปัสสาวะ · อื่นๆ) กดขยายทีละหมวด หมวดแรกเปิด · ทุกผลมี "ค่าปกติ" · ผิดปกติ/วิกฤต = ไอคอน + ตัวอักษร `--text-danger-secondary` ข้างค่า · ผลอ่อนไหว (`src/features/records/lib/sensitiveLabs.js`) ไม่แสดงค่า → การ์ด "ผลตรวจนี้รับฟังที่ ศบส. เท่านั้น" + วันนัดฟังผล · สถานที่ |
| `MobileHomeBpPage` (`/m/home-bp`) | [`MobileHomeBpPage.jsx`](src/features/self-care/pages/MobileHomeBpPage.jsx) | คำสั่งแพทย์ (label/value) → บันทึกตัวบน/ตัวล่าง/ชีพจร/วันเวลา → กราฟ 7·30 วัน + เส้นเป้าหมายประ · ค่าสูงกว่าเป้าหมายจุดแดง · store `phcisLink.homeBpOrders/homeBpReadings` |

Rules + labels live in [`src/features/appointments/lib/appointmentRules.js`](src/features/appointments/lib/appointmentRules.js) (สถานะคำขอ · สถานะนัด · Onsite/Telemed/เยี่ยมบ้าน · ช่วงเวลา · วันทำการ · กฎ 15 นาที · ผู้ป่วยใหม่ไม่เห็น Telemed). Store: `phcisLink.js` `requestAppointmentChange` · `decideAppointmentChange` · `requestHomeVisit`.

#### 7.3.23 Fields · pickers (ผู้ใช้ 21 ต.ค. 69)

| Component | File | Props | Notes |
|---|---|---|---|
| `MobileSelectField` | [SelectField.jsx](src/shared/ui/SelectField.jsx) | `label` · `required` · `value` · `onChange` · `options` · `placeholder='เลือก'` · `sheetTitle` · `disabled` | Input Field look (Figma App DS `267:50`) → `BottomSheet` list. Replaces native `<select>`. |
| `MobileDateField` / `MobileTimeField` | [DateTimeFields.jsx](src/shared/ui/DateTimeFields.jsx) | `value` (ISO / `HH:MM`) · `onChange` · `label` · `min` / `max` · `sheetTitle` · `placeholder` · `minuteStep` (time) · `expanded` + `onToggle` (inline) | DS calendar / 2-column hour:minute sheet instead of the browser picker. Also exports `DatePickerPanel` · `PickerExpand` · `thaiDateLabel(iso)` (พ.ศ.). |
| `MobileOtpField` | [OtpField.jsx](src/shared/ui/OtpField.jsx) | `label` · `required` · `helperText` · `error` · `size='lg'` · `state` · `disabled` · `value` · `onChange` · `onComplete` · `length=6` | 1:1 Figma "OTP Field". `OTP_LEN = 6`. |
| `Checkbox` / `CheckboxBox` | [checkbox/](src/shared/ui/checkbox/) | `Checkbox`: box + label (forwardRef) · `CheckboxBox`: `checked` · `hover` · `disabled` | Figma `269:28` / `306:1099` · 16×16 atom · stroke 1.5px every state. |
| `MobileOptionCard` | [OptionCard.jsx](src/shared/ui/OptionCard.jsx) | `title` · `subtitle` · `illustration` (36×36) · `badge` · `selected` · `onClick` · `width` · `align='center'\|'start'` | Figma DS Card `39:407` — selectable service-type / department row. Border `--mobile-theme-border-choice`. |

#### 7.3.24 Navigation · sheets · shell

| Component | File | Props | Notes |
|---|---|---|---|
| `NavigationMenu` | [NavigationMenu.jsx](src/shared/ui/NavigationMenu.jsx) | `items=NAV_MENU_ITEMS` · `activeKey='home'` · `onSelect` | Figma DS "Navigation". Lottie icons: active filled, inactive = first frame. Press = icon shrinks · newly-selected = bounce (user 24 Oct 2569). |
| `TabletLayout` | [tablet/TabletLayout.jsx](src/shared/ui/tablet/TabletLayout.jsx) | `renderRoutes` | iPad (Figma "iPad · Designs" `6104:2`): side nav + list/detail columns; full-screen routes skip the split. Context in `tablet/tabletPane`. |
| `PageSheet` | [PageSheet.jsx](src/shared/ui/PageSheet.jsx) | `open` · `onClose` · `label` · `children` | Opens an existing full page (with its own `MobileNavHeader`) as a bottom sheet. |
| `InAppBrowser` | [InAppBrowser.jsx](src/shared/ui/InAppBrowser.jsx) | `open` · `url` · `title` · `onClose` | External web pages inside the app (user 19 Oct 2569). |
| `CameraCaptureSheet` | [CameraCaptureSheet.jsx](src/shared/ui/CameraCaptureSheet.jsx) | `open` · `onClose` · `onCapture` · `onFallback` · `maxPx=320` · `title` · `shape='circle'\|'document'` | In-page camera (getUserMedia) for "ถ่ายรูป" — circle = profile (front, mirrored), document = vaccine book. Dark backdrop = gradient exception. |
| `AppKeyboard` · `AppKeyboardPanel` | [AppKeyboard.jsx](src/shared/ui/AppKeyboard.jsx) | Keyboard: none (mounted once in `MobileShell` PhoneFrame) — Panel: `kind` (`numeric` · `decimal` · `text` · `latin`) · `onKey(ch)` · `onBackspace` · `onDone` · `onEnter` · `keyboardKindFor(el)` | In-app keyboard inside the phone mock (user 27 Oct 2569). Focus any input in `.mih-phone-screen` → keyboard docks at the bottom, content above shrinks, home indicator hides; its height is published as `--mih-app-keyboard-h` so every `[role=dialog][aria-modal]` (BottomSheet · page sheet · CenterDialog) ends above the keyboard and the focused field scrolls into view. Number-only fields (`inputMode` numeric · tel · decimal or `type` number · tel) = 0–9 pad only (decimal adds "."). Others = Thai Kedmanee ⇄ ABC ⇄ 123 (email/password start on ABC). Top bar "เสร็จ" blurs. Keys don't steal focus; `maxLength` respected; physical keyboard still works. Not shown on real phones/tablets (`isMobileBrowser`) — they use the OS keyboard picked by the same `inputMode`. `inputMode='none'` / readOnly = no keyboard. |
| `InlineRow` · `EditSheet` | [InlineRow.jsx](src/shared/ui/InlineRow.jsx) | Row: `label` · `value` · `required` · `emptyText='ยังไม่ได้ระบุ'` · `badge` · `onClick` · `last` — Sheet: `title` · `onBack` · `onSubBack` · `onSave` · `saveDisabled` · `saveLabel` · `variant='page'` | Left/right data rows + edit-one-item sheet (user 27 Oct 2569, moved from the profile page). Row: label 16 left (red * when required) · value sm secondary right, one line, ellipsis (empty = tertiary `emptyText`) · › when tappable, else a 20px spacer so values align · 1px divider. Tap → `EditSheet` — a full page by default (`BottomSheet variant='page'`: ‹ back · centred title · Fill save in footer · user 28 Oct 2569; `variant='sheet'` = old bottom sheet) holding the real field (`MobileSelectField` · `MobileInputField` · `MobileDateField` · `DeliveryAddressForm`); edit a draft, save applies, close discards. Used by โปรไฟล์ › ข้อมูลประจำตัว and ลงทะเบียน › สร้างบัญชีผู้ใช้. |
| `DeleteAccountSheet` | [DeleteAccountSheet.jsx](src/shared/ui/DeleteAccountSheet.jsx) | `onClose` | ตั้งค่า › "ลบบัญชี" (user 24 Oct 2569). Exports `DELETE_REASONS` · `DELETED_HN_KEY`. |
| `ConsentDocStep` | [ConsentDocStep.jsx](src/features/auth/components/ConsentDocStep.jsx) | `title` · `body` (`TERMS_BODY` · `POLICY_BODY`) · `consentLabel` · `onAccept` · `stepIndex=0` · `stepCount=1` · `stepTag` | Read a document, then consent (user 27 Oct 2569 ข้อ 2ก). Step bar ("ขั้นตอนที่ n จาก N" + right tag) → centred title → body folded into numbered sections (one open at a time) → full-width Fill button whose label is the consent text (no checkbox). Used by ลงทะเบียน (step 1 of 5 "ข้อมูลส่วนตัว") and first ThaiD sign-in (steps 1–2 of 2 "ก่อนเข้าใช้งานครั้งแรก"). Page chrome `Footer` · `FormCard` · `Title` = [RegisterLayout.jsx](src/features/auth/components/RegisterLayout.jsx). Figma Mobile Application `10448:2` · `10449:2`. |
| `ThaiIdNotInstalledSheet` · `ThaiIdNotInstalledContent` | [ThaiIdNotInstalledSheet.jsx](src/features/auth/components/ThaiIdNotInstalledSheet.jsx) | Sheet: `open` · `onClose` · `onUseCitizenId` — Content: `onDownload=openThaiIdStore` · `onUseCitizenId` · `showTitle` | "เข้าสู่ระบบด้วย ThaiD" with no ThaiD app (user 27 Oct 2569). ThaiD logo 72 → 1-line summary → group "ขั้นตอน" (3 numbered rows) → Fill "ดาวน์โหลดแอป ThaiD" (store by OS · `THAID_STORE_URL` · Google Play = search page, mock) + Outline "เข้าสู่ระบบด้วยเลขบัตรประชาชน". Content = same body inside another sheet (no nested sheets). Demo account `no-thaid` (นายวิชัย รุ่งเรือง). Figma Mobile Application `10443:2`. |

#### 7.3.25 Notifications · journey · icons

| Component | File | Props | Notes |
|---|---|---|---|
| `NotificationCard` | [NotificationCard.jsx](src/shared/ui/NotificationCard.jsx) | `title` · `time` · `body` · `icon` · `tone` · `showButton` · `buttonLabel='ชำระเงิน'` · `onButtonClick` · `onClick` · `unread` · `onSeen` · `state` | Figma DS Card `59:148`. Unread = light CI-pink bg + pink left bar; read = white (user 25 Oct 2569). |
| `NotificationBellButton` · `NotificationBellIcon` | [NotificationBellButton.jsx](src/shared/ui/NotificationBellButton.jsx) · [NotificationBellIcon.jsx](src/shared/ui/NotificationBellIcon.jsx) | Button: `unread` · `color` — Icon: `tone` · `size` · `fallbackColor` | Bell at the right of tab headers (ส่งคำขอ · การนัดหมาย · ประวัติ). Lottie, white-line, replays every minute. |
| `SmsPush` · `EmailPush` | [SmsPush.jsx](src/shared/ui/SmsPush.jsx) · [EmailPush.jsx](src/shared/ui/EmailPush.jsx) | `code` · `reference` · `sender` · `appName` · `duration=9000` · `onDismiss` (+ `subject` · `fromAddress` on email) | Simulated OS push banner carrying the OTP (demo only). |
| `EmailOtpView` · `EmailOtpMessage` | [EmailOtpView.jsx](src/shared/ui/EmailOtpView.jsx) | View: `open` · `onClose` · message props — Message: `code` · `reference` · `purpose='ลงทะเบียน BMA Health'` · `subject` · `sender` · `fromAddress` · `to` (masked) · `validMinutes=5` | The OTP email the user opens (user 27 Oct 2569). Tap `EmailPush` (`onOpen`) in ลงทะเบียน / ลืมรหัสผ่าน → full-page mail view (`BottomSheet variant='page'`, header "อีเมล"). Subject → จาก / ถึง / เวลา (label/value) → body: purpose line → OTP 36px brand on brand-quaternary box + "รหัสอ้างอิง · ใช้ได้ภายใน 5 นาที" → warning `InlineNotice` "ห้ามเปิดเผยรหัสนี้กับผู้อื่น" → "หากคุณไม่ได้ทำรายการนี้…" → footer สำนักอนามัย. Footer: "คัดลอกรหัส" (Outline) · "ไปกรอกรหัส" (Fill). |
| `JourneyStepper` | [JourneyStepper.jsx](src/shared/ui/JourneyStepper.jsx) | `steps=JOURNEY_STEPS` · `orientation='horizontal'\|'vertical'` | Care-journey bar (Figma `371:45332` · states `284:37331`). `JOURNEY_STEPS_DONE` for closed visits. |
| `StepIcon` | [stepIcons.jsx](src/shared/ui/stepIcons.jsx) | `name` · `size=20` | Exact Figma two-tone step glyphs; accent = CI pink (user 21 Oct 2569). `hasStepIcon(name)`. |
| `FigmaIcon` | [figmaIcons.jsx](src/shared/ui/figmaIcons.jsx) | `name` · `size=16` · `color` | Feather glyphs from Figma Home `2042:26021` / Appointment Card `367:41054`; Material names aliased via `resolveFigmaIconKey`. |
| `LottieIcon` | [LottieIcon.jsx](src/shared/ui/LottieIcon.jsx) | `src` · `fallbackIcon` · `size` (24) · `fallbackColor` · `replayMs=30000` · `autoplay` · `fallbackFill` | Icon-sized Lottie (bell · scan · quick menu). `autoplay=false` = frozen first frame. |
| `LottieIllustration` | [illustrations/LottieIllustration.jsx](src/shared/ui/illustrations/LottieIllustration.jsx) | `animationData` · `size=112` · `loop` · `autoplay` · `still` · `ariaLabel` | Empty states / success art. Pass imported JSON, not a `path`. Files come from `@mih/assets`. |
| `Barcode128` | [lab/Barcode128.jsx](src/shared/ui/lab/Barcode128.jsx) | value | Code 128-B inline SVG (lab specimen / queue slip). |


#### 7.3.27 Feature cards · sheets · calendar (ผู้ใช้ 26 ต.ค. 69)

| Component | File | Props | Notes |
|---|---|---|---|
| `ClinicCard` · `ClinicRow` | [ClinicCard.jsx](src/features/clinics/components/ClinicCard.jsx) | `number` · `name` · `distanceKm` · `hoursRows=[{label, value}]` (`hoursRowsOf(hours)`) · `badgeLabel='พรุ่งนี้มีคิวว่าง'` · `selected` · `checked` · `onClick` · `onMap` · `mapLabel='ดูแผนที่'` · `show*` (Number · TextGroupAndBadge · Title · Subtext · Badge · Checkbox · Hours · Map) — Row: `number` · `name` · `distanceKm` · `selected` · `first` · `onClick` · `onMap` | Figma DS Card / Clinic `E5DppVK8yi0EKJp1SUuKa9 39:197`. Card = centre summary in booking sheets (`BookingSheets`) · Row = divider list on /m/clinics. Hours = 3 rows ในเวลาราชการ · นอกเวลาราชการ · วันหยุด. |
| `DoctorCard` | [DoctorCard.jsx](src/features/appointments/components/DoctorCard.jsx) | `name` · `subtitle` · `dept` · `license` · `centre` · `centreLabel='ศูนย์:'` · `avatar` · `anyone` · `badgeLabel='คิวว่างวันนี้'` · `changeLabel='เปลี่ยน'` · `selected` · `checked` · `onClick` · `onChange` · `headerLabel='แพทย์ผู้รักษา'` · `show*` (Header · Profile · Title · Badge · Button · Checkbox) | Figma DS Card / Doctor `E5DppVK8yi0EKJp1SUuKa9 1:703`. `anyone` = "ไม่ระบุแพทย์" (avatar `/illustration/mobile/no-doctor-avatar.png` + `subtitle`). Not used by a page yet (barrel only). |
| `BookingCalendar` | [BookingCalendar.jsx](src/features/appointments/components/BookingCalendar.jsx) | `selectedKey` · `availableKeys` · `fullKeys` · `onSelect(key)` · `fullLabel='เต็ม'` | Figma DS Calendar / Single (Archive `152:46901`) · cell states `143:23705`. Keys = `toBeKey(date)` (พ.ศ. `YYYY-MM-DD`). Today = `surface/calendar/current` + "วันนี้" · selected = `surface/calendar/selected-mobile` · full = "เต็ม" non-interactive · outside month / not available = disabled. Chevron 40×40. |
| `HealthInfoSheet` | [HealthInfoSheet.jsx](src/features/records/components/HealthInfoSheet.jsx) | `open` · `onClose` · `onSave({weight, height, systolic, diastolic, bmi})` | Archive `262:33351` — opened from the health-form notification. Lottie `online-health-checkup.json` → 4 `MobileInputField` (น้ำหนัก · ส่วนสูง · ความดันบน/ล่าง) → live BMI + `StatusBadge` (น้ำหนักน้อย < 18.5 · ปกติ < 25 · น้ำหนักเกิน < 30 · อ้วน) → "บันทึก". |
| `ReceiptSheet` · `ReceiptCard` | [ReceiptSheet.jsx](src/features/payment/components/ReceiptSheet.jsx) | Sheet: `open` · `onClose` · `onSave` · `title='ใบเสร็จรับเงิน'` · `saveLabel='บันทึกใบเสร็จ (e-slip)'` · `...receipt` — Card: `title` · `org` · `statusLabel='ชำระแล้ว'` · `statusStyle` (success \| warning \| info \| error) · `receiptNo` · `receiptNoLabel` · `dateTime` · `paidVia` · `patient` · `itemsTitle` · `items=[{label, amount}]` · `subtotal` · `claimDeduct` · `serviceFee` · `netLabel='ยอดชำระสุทธิ'` · `net` | Figma Mobile Application `2042:29168`. Build the props with `buildReceipt(bill, method)` (PaymentDetailSheet). Before payment `receiptNoLabel='เลขที่ใบแจ้งหนี้'` (PAY-01). Sheet = `BottomSheet` + one Fill CTA. |
| `MedicationListSheet` · `MedCard` | [MedicationListSheet.jsx](src/features/pharmacy/components/MedicationListSheet.jsx) | Sheet: `open` · `meds` · `onClose` · `onConfirm` · `confirmLabel='ยืนยันการรับยา'` · `title='รายการยา'` · `showJourney` — MedCard: `med {name, strength, dose, usage, warning, image, since, dispensedAt}` · `flat` · `divider` | Figma Mobile Application `2387:49519`. No frame / no grey fill per med (ผู้ใช้ 21 ต.ค. 69) · divider between meds · `MedCard` reused on the visit page. |

<!-- components:start (npm run design:catalog — ห้ามแก้มือ) -->
#### 7.3.0 Component registry — ทุก component ที่แอปใช้ (สร้างอัตโนมัติ)

> สร้างจาก `src/shared/ui/index.js` (export ทั้งหมด) + `src/features/dev/pages/MobileComponentsPage.jsx` · `src/features/dev/pages/MobileFeatureDemos.jsx` (เดโม) ด้วย `npm run design:catalog` · 68 รายการ · ยังไม่มีเดโม 0 · ยังไม่มีสเปก 0
> CI รันให้เองทุกครั้งที่ merge เข้า main (`.github/workflows/publish-design-md.yml`) แล้วส่งเมนูแท็บ Mobile (`design-mobile.json`) ขึ้นเว็บ Design System

| Component | ไฟล์ | เดโม (/m/components) | สเปก (หัวข้อใน design.md) |
|---|---|---|---|
| `MobileShell` · `MobileAppProvider` | [`shared/ui/MobileShell.jsx`](src/shared/ui/MobileShell.jsx) | [`mobile-shell`](/m/components?component=mobile-shell) | 7.3.10 MobileShell — frame |
| `Sym` | [`shared/ui/Sym.jsx`](src/shared/ui/Sym.jsx) | [`sym`](/m/components?component=sym) | Primitives |
| `FigmaIcon` | [`shared/ui/figmaIcons.jsx`](src/shared/ui/figmaIcons.jsx) | [`mobile-figma-icon`](/m/components?component=mobile-figma-icon) | 7.3.25 Notifications · journey · icons |
| `MobileButton` | [`shared/ui/MobileButton.jsx`](src/shared/ui/MobileButton.jsx) | [`brand-button`](/m/components?component=brand-button) | 7.3.1 MobileButton — props & variants |
| `MobileCard` | [`shared/ui/Card.jsx`](src/shared/ui/Card.jsx) | [`mobile-card`](/m/components?component=mobile-card) | 7.3.2 MobileCard — props |
| `MobileOptionCard` | [`shared/ui/OptionCard.jsx`](src/shared/ui/OptionCard.jsx) | [`mobile-option-card`](/m/components?component=mobile-option-card) | 7.3.23 Fields · pickers (ผู้ใช้ 21 ต.ค. 69) |
| `ClinicCard` · `ClinicRow` | [`features/clinics/components/ClinicCard.jsx`](src/features/clinics/components/ClinicCard.jsx) | [`mobile-clinic-card-states`](/m/components?component=mobile-clinic-card-states) | 7.3.27 Feature cards · sheets · calendar (ผู้ใช้ 26 ต.ค. 69) |
| `DoctorCard` | [`features/appointments/components/DoctorCard.jsx`](src/features/appointments/components/DoctorCard.jsx) | [`mobile-doctor-card`](/m/components?component=mobile-doctor-card) | 7.3.27 Feature cards · sheets · calendar (ผู้ใช้ 26 ต.ค. 69) |
| `BookingCalendar` | [`features/appointments/components/BookingCalendar.jsx`](src/features/appointments/components/BookingCalendar.jsx) | [`mobile-booking-calendar`](/m/components?component=mobile-booking-calendar) | 7.3.27 Feature cards · sheets · calendar (ผู้ใช้ 26 ต.ค. 69) |
| `BottomSheet` | [`shared/ui/BottomSheet.jsx`](src/shared/ui/BottomSheet.jsx) | [`mobile-bottom-sheet`](/m/components?component=mobile-bottom-sheet) | Primitives |
| `ServiceRequestCard` | [`features/requests/components/ServiceRequestCard.jsx`](src/features/requests/components/ServiceRequestCard.jsx) | [`mobile-service-request-card`](/m/components?component=mobile-service-request-card) | Primitives |
| `RequestDetailSheet` · `RequestIconBadge` | [`features/requests/components/RequestDetailSheet.jsx`](src/features/requests/components/RequestDetailSheet.jsx) | [`mobile-request-detail-sheet`](/m/components?component=mobile-request-detail-sheet) | 7.6. Mobile New Theme (Design Option A) |
| `MobileInput` | [`shared/ui/Input.jsx`](src/shared/ui/Input.jsx) | [`mobile-input-field`](/m/components?component=mobile-input-field) | 7.3.6 MobileInput + MobileField — pattern |
| `MobileField` | [`shared/ui/Field.jsx`](src/shared/ui/Field.jsx) | [`mobile-input-field`](/m/components?component=mobile-input-field) | 7.3.6 MobileInput + MobileField — pattern |
| `MobileInputField` | [`shared/ui/InputField.jsx`](src/shared/ui/InputField.jsx) | [`mobile-input-field`](/m/components?component=mobile-input-field) | 7.3.27 Feature cards · sheets · calendar (ผู้ใช้ 26 ต.ค. 69) |
| `MobileSelectField` | [`shared/ui/SelectField.jsx`](src/shared/ui/SelectField.jsx) | [`mobile-select-field`](/m/components?component=mobile-select-field) | 7.3.23 Fields · pickers (ผู้ใช้ 21 ต.ค. 69) |
| `Password` | [`shared/ui/Password.jsx`](src/shared/ui/Password.jsx) | [`mobile-password`](/m/components?component=mobile-password) | 7.3.3 Password — Figma App 5021:1008 |
| `Textarea` | [`shared/ui/Textarea.jsx`](src/shared/ui/Textarea.jsx) | [`mobile-textarea`](/m/components?component=mobile-textarea) | 7.3.4 Textarea — Figma App 268:26 |
| `RadioButton` · `RadioButtonGroup` | [`shared/ui/RadioButton.jsx`](src/shared/ui/RadioButton.jsx) | [`mobile-radio-button`](/m/components?component=mobile-radio-button) | Primitives |
| `SuccessCard` | [`shared/ui/SuccessCard.jsx`](src/shared/ui/SuccessCard.jsx) | [`mobile-success-card`](/m/components?component=mobile-success-card) | 7.3.5 SuccessCard — Archive 178:122765 |
| `NavigationMenu` | [`shared/ui/NavigationMenu.jsx`](src/shared/ui/NavigationMenu.jsx) | [`mobile-navigation-menu`](/m/components?component=mobile-navigation-menu) | 7.3.24 Navigation · sheets · shell |
| `MobileBadge` | [`shared/ui/Badge.jsx`](src/shared/ui/Badge.jsx) | [`mobile-badge`](/m/components?component=mobile-badge) | 7.3.7 MobileBadge — tones |
| `ColorfulBadge` | [`shared/ui/badge/ColorfulBadge.jsx`](src/shared/ui/badge/ColorfulBadge.jsx) | [`mobile-colorful-badge`](/m/components?component=mobile-colorful-badge) | 7.3.8 ColorfulBadge — Figma App 2076:8413 |
| `SmsPush` | [`shared/ui/SmsPush.jsx`](src/shared/ui/SmsPush.jsx) | [`mobile-sms-push`](/m/components?component=mobile-sms-push) | 7.3.25 Notifications · journey · icons |
| `EmailPush` | [`shared/ui/EmailPush.jsx`](src/shared/ui/EmailPush.jsx) | [`mobile-email-push`](/m/components?component=mobile-email-push) | 7.3.25 Notifications · journey · icons |
| `AppNotification` | [`shared/ui/AppNotification.jsx`](src/shared/ui/AppNotification.jsx) | [`mobile-app-notification`](/m/components?component=mobile-app-notification) | Appointment requests · RatingSheet (คทง.12 APPT-02/04/05 · EVAL-01) |
| `NotificationCard` | [`shared/ui/NotificationCard.jsx`](src/shared/ui/NotificationCard.jsx) | [`mobile-notification-card`](/m/components?component=mobile-notification-card) | 7.3.25 Notifications · journey · icons |
| `HealthInfoSheet` | [`features/records/components/HealthInfoSheet.jsx`](src/features/records/components/HealthInfoSheet.jsx) | [`mobile-health-info-sheet`](/m/components?component=mobile-health-info-sheet) | 7.3.27 Feature cards · sheets · calendar (ผู้ใช้ 26 ต.ค. 69) |
| `AccountSwitcherSheet` | [`features/auth/components/AccountSwitcherSheet.jsx`](src/features/auth/components/AccountSwitcherSheet.jsx) | [`mobile-account-switcher-sheet`](/m/components?component=mobile-account-switcher-sheet) | SettingsList · HomeMenuSheet (คทง.12 HOME-03 · THEME-03) |
| `AccountSwitcherLauncher` | [`features/auth/components/AccountSwitcherLauncher.jsx`](src/features/auth/components/AccountSwitcherLauncher.jsx) | [`mobile-account-switcher-launcher`](/m/components?component=mobile-account-switcher-launcher) | SettingsList · HomeMenuSheet (คทง.12 HOME-03 · THEME-03) |
| `AddAccountSheet` | [`features/auth/components/AddAccountSheet.jsx`](src/features/auth/components/AddAccountSheet.jsx) | [`mobile-add-account-sheet`](/m/components?component=mobile-add-account-sheet) | SettingsList · HomeMenuSheet (คทง.12 HOME-03 · THEME-03) |
| `ThaiIdNotInstalledSheet` · `ThaiIdNotInstalledContent` | [`features/auth/components/ThaiIdNotInstalledSheet.jsx`](src/features/auth/components/ThaiIdNotInstalledSheet.jsx) | [`mobile-thaid-not-installed-sheet`](/m/components?component=mobile-thaid-not-installed-sheet) | 7.3.24 Navigation · sheets · shell |
| `ConsentDocStep` | [`features/auth/components/ConsentDocStep.jsx`](src/features/auth/components/ConsentDocStep.jsx) | [`mobile-consent-doc-step`](/m/components?component=mobile-consent-doc-step) | 7.3.24 Navigation · sheets · shell |
| `ReceiptSheet` · `ReceiptCard` | [`features/payment/components/ReceiptSheet.jsx`](src/features/payment/components/ReceiptSheet.jsx) | [`mobile-receipt-sheet`](/m/components?component=mobile-receipt-sheet) | 7.3.27 Feature cards · sheets · calendar (ผู้ใช้ 26 ต.ค. 69) |
| `PaymentDetailSheet` · `PaymentDetailBody` | [`features/payment/components/PaymentDetailSheet.jsx`](src/features/payment/components/PaymentDetailSheet.jsx) | [`mobile-payment-detail-sheet`](/m/components?component=mobile-payment-detail-sheet) | Carousel dots · Booking/pay blocks · Scan · Vaccine photos (ผู้ใช้ 23 ต.ค. 69) |
| `MedicationListSheet` | [`features/pharmacy/components/MedicationListSheet.jsx`](src/features/pharmacy/components/MedicationListSheet.jsx) | [`mobile-medication-list-sheet`](/m/components?component=mobile-medication-list-sheet) | 7.3.27 Feature cards · sheets · calendar (ผู้ใช้ 26 ต.ค. 69) |
| `LabResults` | [`features/records/components/LabResults.jsx`](src/features/records/components/LabResults.jsx) | [`mobile-lab-results`](/m/components?component=mobile-lab-results) | Lab results · Home BP (คทง.12 ประชุม 19 ต.ค. 69 · EMR-09/10/11 · HBP-01) |
| `JourneyStepper` | [`shared/ui/JourneyStepper.jsx`](src/shared/ui/JourneyStepper.jsx) | [`mobile-journey-stepper`](/m/components?component=mobile-journey-stepper) | 7.3.25 Notifications · journey · icons |
| `CameraPermissionPopup` | [`shared/ui/CameraPermissionPopup.jsx`](src/shared/ui/CameraPermissionPopup.jsx) | [`mobile-camera-permission-popup`](/m/components?component=mobile-camera-permission-popup) | 7.3.19–7.3.22 Common primitives รอบ 2 (ผู้ใช้ 23 ต.ค. 69) |
| `LocationPermissionPopup` | [`shared/ui/LocationPermissionPopup.jsx`](src/shared/ui/LocationPermissionPopup.jsx) | [`mobile-location-permission-popup`](/m/components?component=mobile-location-permission-popup) | 7.3.19–7.3.22 Common primitives รอบ 2 (ผู้ใช้ 23 ต.ค. 69) |
| `MobileHeader` · `MobileNavHeader` | [`shared/ui/Header.jsx`](src/shared/ui/Header.jsx) | [`mobile-header`](/m/components?component=mobile-header) | 7.3.9 MobileHeader — props |
| `FilterChip` · `FilterChipBar` | [`shared/ui/FilterChip.jsx`](src/shared/ui/FilterChip.jsx) | [`mobile-filter-chip`](/m/components?component=mobile-filter-chip) | 7.3.19–7.3.22 Common primitives รอบ 2 (ผู้ใช้ 23 ต.ค. 69) |
| `MobileSectionLabel` | [`shared/ui/SectionLabel.jsx`](src/shared/ui/SectionLabel.jsx) | [`mobile-section-label`](/m/components?component=mobile-section-label) | Primitives |
| `HomeMenuSheet` | [`features/home/components/HomeMenuSheet.jsx`](src/features/home/components/HomeMenuSheet.jsx) | [`mobile-home-menu-sheet`](/m/components?component=mobile-home-menu-sheet) | SettingsList · HomeMenuSheet (คทง.12 HOME-03 · THEME-03) |
| `FontSizeRow` | [`features/profile/components/SettingsList.jsx`](src/features/profile/components/SettingsList.jsx) | [`mobile-font-size-row`](/m/components?component=mobile-font-size-row) | SettingsList · HomeMenuSheet (คทง.12 HOME-03 · THEME-03) |
| `AppointmentRequestStatus` · `AppointmentRulesList` · `LabelValueRows` · `RequestReasonPicker` | [`features/appointments/components/AppointmentRequestStatus.jsx`](src/features/appointments/components/AppointmentRequestStatus.jsx) | [`mobile-appointment-request-status`](/m/components?component=mobile-appointment-request-status) | Appointment requests · RatingSheet (คทง.12 APPT-02/04/05 · EVAL-01) |
| `RatingSheet` | [`features/rating/components/RatingSheet.jsx`](src/features/rating/components/RatingSheet.jsx) | [`mobile-rating-sheet`](/m/components?component=mobile-rating-sheet) | Appointment requests · RatingSheet (คทง.12 APPT-02/04/05 · EVAL-01) |
| `RatingNudge` | [`features/rating/components/RatingNudge.jsx`](src/features/rating/components/RatingNudge.jsx) | [`mobile-rating-nudge`](/m/components?component=mobile-rating-nudge) | Appointment requests · RatingSheet (คทง.12 APPT-02/04/05 · EVAL-01) |
| `TimelineItem` · `TimelineRow` | [`shared/ui/TimelineItem.jsx`](src/shared/ui/TimelineItem.jsx) | [`mobile-timeline-item`](/m/components?component=mobile-timeline-item) | Primitives |
| `TimeSlotList` | [`features/appointments/components/TimeSlotList.jsx`](src/features/appointments/components/TimeSlotList.jsx) | [`mobile-time-slot-list`](/m/components?component=mobile-time-slot-list) | Primitives |
| `ListRow` · `ListSection` | [`shared/ui/ListRow.jsx`](src/shared/ui/ListRow.jsx) | [`mobile-list-row`](/m/components?component=mobile-list-row) | Primitives |
| `PageSection` · `InfoGrid` | [`shared/ui/PageSection.jsx`](src/shared/ui/PageSection.jsx) | [`mobile-page-section`](/m/components?component=mobile-page-section) | Primitives |
| `BottomActionBar` | [`shared/ui/BottomActionBar.jsx`](src/shared/ui/BottomActionBar.jsx) | [`mobile-bottom-action-bar`](/m/components?component=mobile-bottom-action-bar) | Primitives |
| `EmptyState` | [`shared/ui/EmptyState.jsx`](src/shared/ui/EmptyState.jsx) | [`mobile-empty-state`](/m/components?component=mobile-empty-state) | Primitives |
| `InlineNotice` | [`shared/ui/InlineNotice.jsx`](src/shared/ui/InlineNotice.jsx) | [`mobile-inline-notice`](/m/components?component=mobile-inline-notice) | Primitives |
| `Divider` | [`shared/ui/Divider.jsx`](src/shared/ui/Divider.jsx) | [`mobile-divider`](/m/components?component=mobile-divider) | Primitives |
| `AmountRow` · `AmountRows` | [`features/payment/components/AmountRow.jsx`](src/features/payment/components/AmountRow.jsx) | [`mobile-amount-row`](/m/components?component=mobile-amount-row) | Primitives |
| `StepProgress` | [`shared/ui/StepProgress.jsx`](src/shared/ui/StepProgress.jsx) | [`mobile-step-progress`](/m/components?component=mobile-step-progress) | Primitives |
| `Avatar` | [`shared/ui/Avatar.jsx`](src/shared/ui/Avatar.jsx) | [`mobile-avatar`](/m/components?component=mobile-avatar) | Primitives |
| `CenterDialog` | [`shared/ui/CenterDialog.jsx`](src/shared/ui/CenterDialog.jsx) | [`mobile-center-dialog`](/m/components?component=mobile-center-dialog) | Primitives |
| `ChoiceButtonGroup` | [`shared/ui/ChoiceButtonGroup.jsx`](src/shared/ui/ChoiceButtonGroup.jsx) | [`mobile-choice-button-group`](/m/components?component=mobile-choice-button-group) | Primitives |
| `Toast` | [`shared/ui/Toast.jsx`](src/shared/ui/Toast.jsx) | [`mobile-toast`](/m/components?component=mobile-toast) | Primitives |
| `Tabs` | [`shared/ui/Tabs.jsx`](src/shared/ui/Tabs.jsx) | [`mobile-tabs`](/m/components?component=mobile-tabs) | Primitives |
| `ChartTapReadout` | [`shared/ui/ChartTapReadout.jsx`](src/shared/ui/ChartTapReadout.jsx) | [`mobile-chart-tap-readout`](/m/components?component=mobile-chart-tap-readout) | 7.3.19–7.3.22 Common primitives รอบ 2 (ผู้ใช้ 23 ต.ค. 69) |
| `InAppBrowser` | [`shared/ui/InAppBrowser.jsx`](src/shared/ui/InAppBrowser.jsx) | [`mobile-in-app-browser`](/m/components?component=mobile-in-app-browser) | 7.3.24 Navigation · sheets · shell |
| `EmailOtpView` · `EmailOtpMessage` | [`shared/ui/EmailOtpView.jsx`](src/shared/ui/EmailOtpView.jsx) | [`mobile-email-otp-view`](/m/components?component=mobile-email-otp-view) | 7.3.25 Notifications · journey · icons |
| `AppKeyboard` · `AppKeyboardPanel` | [`shared/ui/AppKeyboard.jsx`](src/shared/ui/AppKeyboard.jsx) | [`mobile-app-keyboard`](/m/components?component=mobile-app-keyboard) | 7.3.24 Navigation · sheets · shell |
| `InlineRow` · `EditSheet` | [`shared/ui/InlineRow.jsx`](src/shared/ui/InlineRow.jsx) | [`mobile-inline-row`](/m/components?component=mobile-inline-row) | 7.3.24 Navigation · sheets · shell |
<!-- components:end -->

### 7.4. What does **not** exist yet on mobile

These web primitives have **no** mobile equivalent today. Pages that need them must either compose from existing mobile primitives or motivate the new addition in a PR:

- `Table` — long list patterns use stacked `MobileCard` rows instead.
- ~~`Tabs`~~ — now `Tabs` (§7.3.22, underline). A pill segmented control is still not built; use `ChoiceButtonGroup` (§7.3.20) for 2-option switches.
- `Accordion` — `MobileCard` + collapsible state is the current pattern.
- ~~`Dropdown` / `Select`~~ — now `MobileSelectField` (§7.3.23): Input-Field look, opens a `BottomSheet` picker.
- ~~`DatePicker`~~ — now `MobileDateField` / `MobileTimeField` (§7.3.23). `TimeSlot` booking grids still use [TimeSlotList.jsx](src/features/appointments/components/TimeSlotList.jsx).

When adding a new mobile primitive: create the file under [src/shared/ui/](src/shared/ui/), add tokens to §7.2, and register the entry in §7.3's component-map table.

### 7.7. Navigation stack (คทง.12 · 21 ต.ค. 69)

แอปเป็น layer ซ้อนทับ: ทุกหน้าที่เปิดจากหน้าอื่นซ้อนบนหน้าเดิม · **← ย้อนกลับ / ✕ ปิด = ปิด layer บนสุด กลับหน้าที่เปิดมันจริง** (ไม่ใช่ route ตายตัว) · scroll/แท็บ/ตัวกรองของหน้าเดิมยังอยู่

| ระดับ | คืออะไร | เปิด | ปิด |
|---|---|---|---|
| ฐาน | 5 แท็บล่าง + เข้าสู่ระบบ | `history.replace` (สลับแท็บ ไม่ซ้อน) | — |
| หน้าเต็ม | รายละเอียด · รายการย่อย · ฟอร์ม | `history.push(path)` — `useStackHistory` (MobileShell) ใส่ `{ mobileTransition: 'slide-in' }` ให้อัตโนมัติ | ← `goBackOr(history, fallback)` |
| แผ่น/ทับ | bottom sheet · popup · หน้าแบบแผ่น | state ในหน้า หรือ `{ mobileTransition: 'sheet' }` | ✕ / แตะฉากหลัง / ปัดลง |
| จบ flow | หน้าสำเร็จ | `history.replace` ทับหน้าฟอร์ม | ปุ่มปลายทางใช้ `replace` — back ไม่เปิดฟอร์มที่ส่งแล้ว |

- `goBackOr(history, fallback)` ([Header.jsx](src/shared/ui/Header.jsx)): มีหน้าก่อนหน้าในแอป → `goBack()` · เปิดจากลิงก์ตรง → `replace(fallback)` · `MobileNavHeader` / `MobileHeader` รับ prop `fallback`
- ห้ามเขียน `onBack={() => history.push('/m/…')}` — ใช้ `goBackOr` หรือส่ง `fallback`

### 7.6. Mobile New Theme (Design Option A)

ลูกค้าเลือก 18 ต.ค. 69 · Figma `XCuT5qG9E7V7Aa815X3usM` — Theme [`2790:109177`](https://www.figma.com/design/XCuT5qG9E7V7Aa815X3usM/Mobile-Application--New-Theme-?node-id=2790-109177) · DS page "New Theme · Design System" [`6003:47665`](https://www.figma.com/design/XCuT5qG9E7V7Aa815X3usM/Mobile-Application--New-Theme-?node-id=6003-47665). Approved exception to §7.1 "must not redefine colour / typography family" — the customer changed the mobile brand look.

**Scope:** `mobile.css` block `:root[data-app='mobile']:not([data-mode='dark'])`. Only bma.mobile sets `<html data-app="mobile">` → web pages (bma.health, including its `/m/*` preview) keep the default values. Dark mode unchanged.

| Token | Value | Use |
|---|---|---|
| `--mobile-theme-header` | `var(--surface-brandPrimary-default)` #007549 | solid header / splash / login / profile hero (was teal gradient #13B292 → #00A08E) |
| `--mobile-theme-border-card` | `var(--brand-p200)` #C5EDDA | 1px card border |
| `--mobile-theme-accent-pink` | #EC3FAA | appointment banner outline + title, bell dot |
| `--mobile-theme-accent-green` | #50A622 | HN badge (solid, white text) |
| `--mobile-theme-overlay` · `--surface-overlay-default` *(re-pointed)* | `rgb(0 0 0 / 0.50)` black 50% | backdrop of every bottom sheet / popup in the mobile app (was slate-500 @ 40%) — user 21 Oct 2569 |
| `--mobile-theme-ci-*` | emerald #00744B · forest #205A41 · leaf #67A33B · mint #00C08B · gold #FFB449 · yellow #F8DF52 · blue #0071CE · pink → accent-pink | BMA CI secondary palette (THEME-01) — primary stays emerald; never re-tone to look like "หมอ กทม." |
| `--mobile-theme-filter-shadow-pink` | `drop-shadow(0 4px 8px rgb(236 63 170 / 0.25))` | `filter:` twin of shadow-pink for clip-path shapes (home notice banner) — user 25 Oct 2569 |
| `--mobile-theme-border-choice` | `var(--brand-p300)` #91DFBB | unselected choice / option-card border (ผู้ใช้ 19 ต.ค. 69) |
| `--mobile-theme-shadow-pink` | `0 4px 16px rgb(236 63 170 / 0.20)` CI pink 20% | drop shadow of the pink outstanding-balance card (ชำระเงิน › ยอดที่ต้องชำระ) — user 25 Oct 2569 |
| `--text-input-require` *(re-pointed)* | `var(--mobile-theme-ci-pink)` #EC3FAA | required-field `*` on every mobile form (was red #DC2626) — user 25 Oct 2569 |
| `--mobile-theme-family-display` | `var(--type-family-heading-graphic)` Sao Chingcha | splash · home hero · `MobileNavHeader` title (THEME-02) — `@font-face` self-hosted in bma.mobile `public/fonts/sao-chingcha/` |
| `--mobile-theme-weight-semibold` | 600 | section titles (Anuphan SemiBold 14, brand) |
| `--background-screen-brand-100` *(re-pointed)* | `var(--surface-card-100)` white | page background (was mint #F4FBF8) |
| `--type-family-heading-graphic2` *(re-pointed)* | Anuphan | headings / labels / buttons — light **and dark** mode (was Mitr; app no longer loads Mitr · user 25 Oct 2569) |
| `--mobile-theme-family-label` | `'Anuphan Medium'` (500, self-hosted) → Anuphan | home quick-menu labels — lighter than headings (user 25 Oct 2569) |
| `--shadow-brand-drop-bottom-200` · `--shadow-card` *(re-pointed)* | `inset 0 0 0 1px var(--mobile-theme-border-card)` | cards: border instead of drop shadow |
| `--surface-neutral-primary` *(re-pointed)* | #FFFFFF · dark `var(--d-surface-1)` #222629 | neutral surface — value from Figma variable `surface/neutral/primary` (ds:sync pull 26 Oct 2569) |

**Patterns (Figma DS components):** Header/Home (avatar · greeting · solid green HN badge · outline pill "บัตรประจำตัว" · outline bell) · Home/Quick Menu (8 actions, 4 columns, 60px gradient circle + white glyph — gradients inline: green #9BC781→#77B757 · mint #10DFB5→#00D29D · orange #FFCF8A→#FFC15E · blue #6AA8E6→#008CDE · pink #FF90CD→#FF63BC · purple #BB96D2→#772FA5) · Banner/Appointment (white, 1px pink outline, cut corners, solid pink offset) · Chip/Request (pill, 1px border) · Card/Visit · Card/News · History/Timeline Item · Background/Splash. Bottom nav and tabs keep their green drop shadow.

**BMA CI colours (Color System 01/02):** primary emerald #00744B (PANTONE 3415C) · secondary #205A41 · #67A33B · #00C08B · #FFB449 · #F8DF52 · #0071CE · pink #EC3FAA (PANTONE 232C). Illustrations use CI only: green parts #00744B · pink #EC3FAA (light pink = #F69FD5, 50% tint) · blue #0071CE — Figma variables `color/ci/*` in collection "New Theme". Titles + button labels = Anuphan SemiBold.

**Icon colours:** every icon badge (Colorful icon Badge · MobileIconBadge · RequestIconBadge · ColorfulIconBadge on mobile) = theme gradient by tone (green → Green · purple → Purple · blue/indigo → Blue · pink/magenta → Pink · teal/turquoise → Mint · orange/yellow → Orange) + white glyph — `MOBILE_THEME_GRADIENTS` / `mobileIconSolidStyle(tone)` in bma.mobile `mobileIconTones.js`. Red (danger) and grey badges keep the pastel palette. Small inline glyphs in cards use BMA accents (medication #50A622 · lab #0073D6 · X-ray/notification #EC3FAA). Nav: active #00744B · inactive #ADB2B7.

### 7.5. Figma reference

**Mobile button source-of-truth** — file `opz7X9AAKGQf5prHhsPDGn` (MIH-Design-System-Foundation Copy), Button section `319:2012`. The 16 frames in that section map 1:1 to the 16 files in [src/shared/ui/button/](src/shared/ui/button/) — see §7.3 table for per-component node IDs.

**Web button source-of-truth** — file `GXI1CGpLhpImm8JYpW9ksh`, Button section `319:2012`. Web buttons live in bma.health `src/components/button/` and use the same component names (BrandButton, etc.) with web-specific tokens (radius-150 / 4-6 px, no pill).

**Password source-of-truth** — Figma App file `opz7X9AAKGQf5prHhsPDGn`, Password set [`5021:1008`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=5021-1008) → [`Password.jsx`](src/shared/ui/Password.jsx).

**Textarea source-of-truth** — Figma App file `opz7X9AAKGQf5prHhsPDGn`, Textarea set [`268:26`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=268-26) → [`Textarea.jsx`](src/shared/ui/Textarea.jsx).

Other mobile primitives (Card / Input / Field / Badge / Header / SectionLabel / Shell) do not yet have a Figma master — when they do, add Figma node IDs to the §7.3 component map and drop the `Mobile*` prefix following the button-family convention.

---

