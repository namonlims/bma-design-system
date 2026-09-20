# Design System Token Reference

> **Canonical package:** [`@mih/design-tokens`](https://github.com/namonlims/mih-design) (`packages/design-tokens`).  
> CSS custom properties ship from that package (`tokens.css`); this `design.md` is the human-readable mirror.  
> Edit tokens in **mih-design**, then run `npm run tokens:pull` here to refresh this file.
>
> Auto-generated from Figma Variable exports. Last updated: July 2026 (§1–§4 synced from tokens/figma/ (primitive · brand · breakpoint · semantic))

## Token Architecture

The MIH token system has **3 vertical layers + 2 orthogonal overlays**.
Each layer aliases the layer above it — components never consume raw
primitives, and primitives never know which surface uses them. This is
what makes the 10-brand theme swap atomic, the mobile platform overlay
non-invasive, and the design tokens-only rule in
[`CLAUDE.md`](CLAUDE.md) enforceable.

```
┌─────────────────────────────────────────────────────────────────────┐
│ LAYER 1 · PRIMITIVES — raw values · never used directly in code     │
│ ─────────────────────────────────────────────────────────────────── │
│  COLOR              DIMENSION         TYPOGRAPHY      EFFECTS       │
│  --brand-p{50…900}  --dim-radius-*    --type-size-*   inner-shadow  │
│  --color-{red,      --dim-stroke-*    --type-weight-* drop-shadow   │
│   orange, blue,     --dim-space-*     --type-family-* blur          │
│   teal, indigo,     --dim-size-*                       (effect      │
│   …}                --dim-blur-*                        primitives) │
│  --PrimaryShadow-*                                                   │
│                                                                      │
│  ~89 color · 37 dim · 11 typography · effect primitives             │
└──────────────────────────────────┬──────────────────────────────────┘
                                   │ aliased by ↓
┌──────────────────────────────────▼──────────────────────────────────┐
│ LAYER 2 · SEMANTIC — purpose-based · brand-aware                    │
│ ─────────────────────────────────────────────────────────────────── │
│  --surface-*    --border-*    --text-*      --icon-*                │
│  --shadow-*     --background-*                                       │
│                                                                      │
│  examples (each alias resolves to a Layer 1 primitive):              │
│   --surface-card-default          → var(--brand-p50)                │
│   --text-content-primary          → ink-dark                        │
│   --shadow-card                   → drop-shadow-bottom.200 stack    │
│   --shadow-brand-drop-bottom-100  → uses --PrimaryShadow-600        │
│                                                                      │
│  ~552 semantic tokens                                                │
└──────────────────────────────────┬──────────────────────────────────┘
                                   │ aliased by ↓
┌──────────────────────────────────▼──────────────────────────────────┐
│ LAYER 3 · COMPONENT — per-component · per-state · per-variant       │
│ ─────────────────────────────────────────────────────────────────── │
│  --surface-table-header           --surface-table-footer            │
│  --surface-button-brand-*         --border-alert-{info,warn,error}  │
│  --text-calendar-selected         --surface-pill-{red,orange,…}     │
│                                                                      │
│  Components consume these DIRECTLY (Table.jsx, BrandButton.jsx, …). │
│  Adding a new variant → add a new component token here, alias it    │
│  to a semantic (Layer 2) token, then reference the variable.        │
└─────────────────────────────────────────────────────────────────────┘


      ┌───────────────────────────────────────────────────────────┐
      │ ORTHOGONAL · PLATFORM OVERLAY — --mobile-*                │
      │ ───────────────────────────────────────────────────────── │
      │ Mirrors the Layer 2 + 3 structure with mobile-specific    │
      │ values (bigger touch targets, pill radius default):       │
      │                                                            │
      │   --mobile-button-height-sm   = 40   (Figma Small)         │
      │   --mobile-button-height-md   = 48   (Figma Medium)        │
      │   --mobile-button-height-lg   = 56   (Figma Large)         │
      │   --mobile-button-radius      = pill (full)               │
      │   --mobile-surface-* / --mobile-text-* / --mobile-border-*│
      │                                                            │
      │ ~290 mobile-namespaced tokens. Mobile components read     │
      │ ONLY --mobile-*; desktop components read the unprefixed   │
      │ tokens. The two trees never cross.                        │
      └───────────────────────────────────────────────────────────┘

      ┌───────────────────────────────────────────────────────────┐
      │ ORTHOGONAL · BRAND-MODE SWAP — [data-brand="…"]           │
      │ ───────────────────────────────────────────────────────── │
      │ Overrides selected color PRIMITIVES (Layer 1) under each  │
      │ brand selector. Every semantic + component token that     │
      │ aliases --brand-p* retints atomically — no component      │
      │ change, no JS, no re-render contract.                     │
      │                                                            │
      │ 10 brand modes published (SettingsModal → ProfilePopover):│
      │   default · soft-peach · muted-teal · sakura · soft-sky · │
      │   rose-mist · blue-serenity · dusty-mauve ·               │
      │   soft-lavender · warm-gray                               │
      │                                                            │
      │ Applied on <html data-brand="…"> — global swap; the user  │
      │ selects in the Settings modal and the host page sets it.  │
      └───────────────────────────────────────────────────────────┘
```

### Alias rules

1. **A token may ONLY alias a token from the layer above it.** Never raw
   values inside a component, never sideways (semantic → semantic of
   unrelated purpose), never upward (a component token bypassing semantic
   to reach a primitive).
2. **Components consume ONLY Layer 3 (component) or Layer 2 (semantic)
   tokens.** Never Layer 1. Gradients are the only documented exception —
   see [`CLAUDE.md`](CLAUDE.md) § "Design tokens are the only source of truth".
3. **The fallback in `var(--token, literal)` must match the variable's
   actual resolved value.** A mismatched fallback (e.g.
   `var(--dim-radius-100, 6px)` when `radius-100` is 4 px) flags that the
   wrong token was chosen — pick the token whose value matches.
4. **Adding a new value to Figma:** first add the primitive in §1,
   wire it into `src/index.css` `:root`, then alias downward to semantic
   (§4) and (where needed) component (§5/§6). Do not introduce literals
   that bypass the cascade.
5. **Brand swap is orthogonal:** never hard-code a brand colour at the
   semantic or component layer. Reference `--brand-p{50…900}` so the
   `[data-brand]` overrides flow through.

### File map

| Layer | Source of truth (Figma) | Spec file | CSS variable file |
|---|---|---|---|
| 1 · Primitives | MIH Foundation library | `design.md` | `tokens.css` (`:root`) |
| 2 · Semantic | MIH Foundation library | `design.md` | `tokens.css` (`:root`) |
| 2 · Semantic (import) | Figma W3C export drop zone | `figma/*.tokens.json` (mih-design) | — (`sync-design-md-from-figma-tokens.mjs` regenerates design.md) |
| 3 · Component | Per-component Figma file | `design.md` | `tokens.css` (`:root`) |
| Mobile overlay | MIH Mobile section | `design.md` | `tokens.css` (`--mobile-*`) |
| Brand swap | (no Figma file — runtime override) | `design.md` | `tokens.css` (`[data-brand="…"]`) |

`tokens.css` = `@mih/design-tokens/tokens.css` ใน repo mih-design (mirror ที่ `src/styles/tokens.css` ของ bma.health)

---

## 1. Primitive Tokens

Primitive tokens are the raw, mode-independent values. They are the source of truth referenced by all other layers.

### 1a. Color Palettes

Each palette has 11 steps (50 → 950). Values are in HEX.


#### `emerald`

| Step | Hex |
|------|-----|
| 50 | `#F4FBF8` |
| 100 | `#E2F3EB` |
| 200 | `#C5EDDA` |
| 300 | `#91DFBB` |
| 400 | `#6AD6A5` |
| 500 | `#54CF97` |
| 600 | `#08A768` |
| 700 | `#007549` |
| 800 | `#004C31` |
| 900 | `#003322` |
| 950 | `#002318` |

#### `forest`

| Step | Hex |
|------|-----|
| 50 | `#FAFCFA` |
| 100 | `#EFF5EF` |
| 200 | `#E1F0E0` |
| 300 | `#A7D2A3` |
| 400 | `#78B573` |
| 500 | `#53954E` |
| 600 | `#427C3D` |
| 700 | `#366233` |
| 800 | `#2E4F2C` |
| 900 | `#274126` |
| 950 | `#102310` |

#### `neutral`

| Step | Hex |
|------|-----|
| 50 | `#F9FBFB` |
| 100 | `#F4F5F5` |
| 200 | `#E2E4E6` |
| 300 | `#C9CDD0` |
| 400 | `#ADB2B7` |
| 500 | `#858C92` |
| 600 | `#636B72` |
| 700 | `#4D5358` |
| 800 | `#363B3F` |
| 900 | `#222629` |
| 950 | `#111314` |

#### `slate`

| Step | Hex |
|------|-----|
| 50 | `#F5F6FA` |
| 100 | `#F5F7F9` |
| 200 | `#E9EDF5` |
| 300 | `#CBD5E1` |
| 400 | `#94A3B8` |
| 500 | `#64748B` |
| 600 | `#475569` |
| 700 | `#334155` |
| 800 | `#1E293B` |
| 900 | `#0F172A` |
| 950 | `#020617` |

#### `grey`

| Step | Hex |
|------|-----|
| 50 | `#FFFFFF` |
| 100 | `#FDFDFD` |
| 200 | `#FAFAFA` |
| 300 | `#F7F7F7` |
| 400 | `#F4F4F4` |
| 500 | `#F0F0F0` |
| 600 | `#EBEBEB` |
| 700 | `#E8E8E8` |
| 800 | `#E3E3E3` |
| 900 | `#DEDEDE` |
| 950 | `#DADADA` |

#### `pink`

| Step | Hex |
|------|-----|
| 50 | `#FBF4F8` |
| 100 | `#FCE7F3` |
| 200 | `#FBCFE8` |
| 300 | `#F9A8D4` |
| 400 | `#F472B6` |
| 500 | `#EC4899` |
| 600 | `#DB2777` |
| 700 | `#BE185D` |
| 800 | `#9D174D` |
| 900 | `#831843` |
| 950 | `#500724` |

#### `yellow`

| Step | Hex |
|------|-----|
| 50 | `#FBFAF4` |
| 100 | `#FEF9C3` |
| 200 | `#FEF08A` |
| 300 | `#FDE047` |
| 400 | `#FACC15` |
| 500 | `#EAB308` |
| 600 | `#CA8A04` |
| 700 | `#A16207` |
| 800 | `#854D0E` |
| 900 | `#713F12` |
| 950 | `#422006` |

#### `orange`

| Step | Hex |
|------|-----|
| 50 | `#FFF9F0` |
| 100 | `#FFEDD5` |
| 200 | `#FED7AA` |
| 300 | `#FDBA74` |
| 400 | `#FB923C` |
| 500 | `#F97316` |
| 600 | `#EA580C` |
| 700 | `#C2410C` |
| 800 | `#9A3412` |
| 900 | `#7C2D12` |
| 950 | `#431407` |

#### `teal`

| Step | Hex |
|------|-----|
| 50 | `#F4FBFA` |
| 100 | `#D1F6EE` |
| 200 | `#A2EDDE` |
| 300 | `#6CDCCA` |
| 400 | `#3EC3B2` |
| 500 | `#24A899` |
| 600 | `#1B867D` |
| 700 | `#196C65` |
| 800 | `#195652` |
| 900 | `#194845` |
| 950 | `#082B2A` |

#### `purple`

| Step | Hex |
|------|-----|
| 50 | `#F7F3FC` |
| 100 | `#F3E8FF` |
| 200 | `#E9D5FF` |
| 300 | `#D8B4FE` |
| 400 | `#C084FC` |
| 500 | `#A855F7` |
| 600 | `#9333EA` |
| 700 | `#7E22CE` |
| 800 | `#6B21A8` |
| 900 | `#581C87` |
| 950 2 | `#3B0764` |

#### `magenta`

| Step | Hex |
|------|-----|
| 50 | `#FAF3FC` |
| 100 | `#F9D6FE` |
| 200 | `#F3ADFD` |
| 300 | `#EA78F9` |
| 400 | `#DC4CF2` |
| 500 | `#C926E0` |
| 600 | `#A81BBD` |
| 700 | `#871598` |
| 800 | `#6E1379` |
| 900 | `#5A125F` |
| 950 | `#380040` |

#### `blue`

| Step | Hex |
|------|-----|
| 50 | `#F0F6FF` |
| 100 | `#DBEAFE` |
| 200 | `#BFDBFE` |
| 300 | `#93C5FD` |
| 400 | `#60A5FA` |
| 500 | `#3B82F6` |
| 600 | `#2563EB` |
| 700 | `#1D4ED8` |
| 800 | `#1E40AF` |
| 900 | `#1E3A8A` |
| 950 | `#172554` |

#### `indigo`

| Step | Hex |
|------|-----|
| 50 | `#F3F5FC` |
| 100 | `#EEF2FF` |
| 200 | `#E0E7FF` |
| 300 | `#C7D2FE` |
| 400 | `#A5B4FC` |
| 500 | `#818CF8` |
| 600 | `#6366F1` |
| 700 | `#4F46E5` |
| 800 | `#4338CA` |
| 900 | `#3730A3` |
| 950 | `#1E1B4B` |

#### `red`

| Step | Hex |
|------|-----|
| 50 | `#FFF0F0` |
| 100 | `#FEE2E2` |
| 200 | `#FECACA` |
| 300 | `#FCA5A5` |
| 400 | `#F87171` |
| 500 | `#EF4444` |
| 600 | `#DC2626` |
| 700 | `#B91C1C` |
| 800 | `#991B1B` |
| 900 | `#7F1D1D` |
| 950 | `#450A0A` |

#### Special Color Tokens

Alpha-baked shadow primitives. Alpha is pre-multiplied into the swatch — consumers use them as drop-shadow / glow color values directly.

| Token | Hex | Opacity |
|-------|-----|---------|
| `color.shadow.050` | `#64748B` | 10 % |
| `color.shadow.100` | `#64748B` | 15 % |
| `color.shadow.400` | `#64748B` | 25 % |
| `color.shadow.indigo-050` | `#818CF8` | 10 % |
| `color.shadow.indigo-100` | `#818CF8` | 15 % |
| `color.shadow.indigo-400` | `#818CF8` | 25 % |

> **Indigo shadow primitives** alias `indigo/400` (`#818CF8` — the same swatch used by `--surface-colorfulbadge-indigo-bold`) at three opacity tiers. Used wherever a brand-violet ambient glow is needed — e.g. Doctor AI panel hover and indigo-bold pill drops.

Alpha-baked **background tint** primitives (Figma scope `EFFECT_COLOR`). Each is a `teal`/`sky-cyan` swatch pre-multiplied to 20 % — used directly as a translucent fill (e.g. card / chip ambient wash). The light and dark modes alias these via `background.bg.*` (§4).

| Token | Hex | Opacity |
|-------|-----|---------|
| `color.bg.sky-cyan-400` | `#40AAF2` | 20 % |
| `color.bg.sky-cyan-600` | `#0A73CB` | 20 % |
| `color.bg.teal-400` | `#3EC3B2` | 20 % |
| `color.bg.teal-600` | `#1B867D` | 20 % |

Solid **sky-cyan/700** (`--color-sky-cyan-700 = #0859A8`) — the ศูนย์ number text
inside the Clinic Card badge (Figma DS Card `E5DppVK8yi0EKJp1SUuKa9` 39:197); the
badge fill itself is a 141° cyan gradient (token exception, inline).

---

### 1a-2. Additional Color Palettes (from JSON)

> Palettes below come from `primitive.tokens.json` and are addressable as primitives. Brand modes (§2) reach into a subset of these for their `primary`/`secondary` maps.


#### `blue-serenity`

| Step | Hex |
|------|-----|
| 50 | `#F0F2FF` |
| 100 | `#E4E8FF` |
| 200 | `#D0D7FF` |
| 300 | `#BBC6FE` |
| 400 | `#A5B3FC` |
| 500 | `#8EA0F8` |
| 600 | `#7A8CF0` |
| 700 | `#6676E0` |
| 800 | `#5060C8` |
| 900 | `#3A4AAA` |
| 950 | `#2A3488` |

#### `peachy-blush`

| Step | Hex |
|------|-----|
| 50 | `#FFF5F0` |
| 100 | `#FDE8DE` |
| 200 | `#FAC8B8` |
| 300 | `#F5A595` |
| 400 | `#EE8278` |
| 500 | `#E0625E` |
| 600 | `#C84858` |
| 700 | `#A93556` |
| 800 | `#882549` |
| 900 | `#65183A` |
| 950 | `#440E28` |

#### `sage`

| Step | Hex |
|------|-----|
| 50 | `#EEF5EF` |
| 100 | `#D8EBDA` |
| 200 | `#B2D5B8` |
| 300 | `#88BA93` |
| 400 | `#5F9D6E` |
| 500 | `#4A8459` |
| 600 | `#3D6E4A` |
| 700 | `#34593E` |
| 800 | `#2B4733` |
| 900 | `#233829` |
| 950 | `#17261B` |

#### `warm-gray`

| Step | Hex |
|------|-----|
| 50 | `#F5F3F1` |
| 100 | `#EBE7E4` |
| 200 | `#D5CEC9` |
| 300 | `#BDB5AF` |
| 400 | `#A39A94` |
| 500 | `#8A8078` |
| 600 | `#726760` |
| 700 | `#5C504A` |
| 800 | `#463C37` |
| 900 | `#312B26` |
| 950 | `#211D19` |

#### `periwinkle`

| Step | Hex |
|------|-----|
| 50 | `#F0F0FF` |
| 100 | `#E4E3FE` |
| 200 | `#C9C8FC` |
| 300 | `#A8A5F8` |
| 400 | `#8B87F2` |
| 500 | `#7068E8` |
| 600 | `#5C50D4` |
| 700 | `#4B3FBA` |
| 800 | `#3A3092` |
| 900 | `#272070` |
| 950 | `#1A1550` |

#### `sky-cyan`

| Step | Hex |
|------|-----|
| 50 | `#F4F9FD` |
| 100 | `#E5F2FC` |
| 200 | `#BCDDF9` |
| 300 | `#80C3F7` |
| 400 | `#40AAF2` |
| 500 | `#0E90E8` |
| 600 | `#0A73CB` |
| 700 | `#0859A8` |
| 800 | `#0A4383` |
| 900 | `#0C3062` |
| 950 | `#081E40` |

#### `rose`

| Step | Hex |
|------|-----|
| 50 | `#FEF2F2` |
| 100 | `#FCE4E5` |
| 200 | `#F8C5C7` |
| 300 | `#F2A0A3` |
| 400 | `#EA8285` |
| 500 | `#E27B7F` |
| 600 | `#CC5E62` |
| 700 | `#AB4548` |
| 800 | `#8C3437` |
| 900 | `#6B2527` |
| 950 | `#4A181A` |

#### `muted-teal`

| Step | Hex |
|------|-----|
| 50 | `#F0F8F7` |
| 100 | `#DAEEED` |
| 200 | `#B5DBD9` |
| 300 | `#8DC3C0` |
| 400 | `#7AABA8` |
| 500 | `#6F9693` |
| 600 | `#5A7D7A` |
| 700 | `#486462` |
| 800 | `#374E4C` |
| 900 | `#283A38` |
| 950 | `#1A2726` |

#### `lavender`

| Step | Hex |
|------|-----|
| 50 | `#F7F4FF` |
| 100 | `#EDE7FF` |
| 200 | `#DDD0FF` |
| 300 | `#D0BEFF` |
| 400 | `#CBBCFF` |
| 500 | `#C8B6FF` |
| 600 | `#A98EF0` |
| 700 | `#8A6CD8` |
| 800 | `#6B4EB8` |
| 900 | `#4E3490` |
| 950 | `#342068` |

#### `lime`

| Step | Hex |
|------|-----|
| 50 | `#F5FCE6` |
| 100 | `#EAF7CC` |
| 200 | `#D5EF9A` |
| 300 | `#BDE36A` |
| 400 | `#ADD654` |
| 500 | `#A5CF4F` |
| 600 | `#88B033` |
| 700 | `#6B8E26` |
| 800 | `#526C1C` |
| 900 | `#3B4E14` |
| 950 | `#27340D` |

#### `soft-rose`

| Step | Hex |
|------|-----|
| 50 | `#FEF5F6` |
| 100 | `#FDE8EA` |
| 200 | `#FBD1D4` |
| 300 | `#F8B8BC` |
| 400 | `#F6AEB2` |
| 500 | `#F4A4AA` |
| 600 | `#E07880` |
| 700 | `#C25560` |
| 800 | `#9E3A44` |
| 900 | `#78272F` |
| 950 | `#521A20` |

#### `soft-lavender`

| Step | Hex |
|------|-----|
| 50 | `#FAF7FF` |
| 100 | `#F2ECFF` |
| 200 | `#E5D9FF` |
| 300 | `#D6C6FA` |
| 400 | `#C5B2F5` |
| 500 | `#B49EEE` |
| 600 | `#9D87D8` |
| 700 | `#836FC0` |
| 800 | `#6655A0` |
| 900 | `#4A3D7A` |
| 950 | `#302754` |

#### `soft-sky`

| Step | Hex |
|------|-----|
| 50 | `#F5FBFF` |
| 100 | `#E8F5FD` |
| 200 | `#D6ECFA` |
| 300 | `#D6ECFA` |
| 400 | `#A8D3EE` |
| 500 | `#8EC4E5` |
| 600 | `#8EC4E5` |
| 700 | `#5594C4` |
| 800 | `#3D78A8` |
| 900 | `#285A82` |
| 950 | `#183D5C` |

#### `soft-yellow`

| Step | Hex |
|------|-----|
| 50 | `#FFFDF5` |
| 100 | `#FDF8E2` |
| 200 | `#FAF0C4` |
| 300 | `#F8E7AA` |
| 400 | `#F6DE9C` |
| 500 | `#F5DD90` |
| 600 | `#DEC472` |
| 700 | `#C4A852` |
| 800 | `#A08838` |
| 900 | `#786420` |
| 950 | `#4E4010` |

#### `soft-peach`

| Step | Hex |
|------|-----|
| 50 | `#FFFAF7` |
| 100 | `#FEF0E8` |
| 200 | `#FDE2D0` |
| 300 | `#FCD1B8` |
| 400 | `#FBBEA0` |
| 500 | `#FAA381` |
| 600 | `#E88A66` |
| 700 | `#D0714D` |
| 800 | `#B05A38` |
| 900 | `#844026` |
| 950 | `#562814` |

#### `dusty-mauve`

| Step | Hex |
|------|-----|
| 50 | `#F9F4F5` |
| 100 | `#F2E6E8` |
| 200 | `#E4CDD0` |
| 300 | `#D0ADB2` |
| 400 | `#B58E94` |
| 500 | `#977177` |
| 600 | `#7E5A60` |
| 700 | `#664750` |
| 800 | `#503840` |
| 900 | `#3C2A30` |
| 950 | `#281C21` |

#### `cool-lavender`

| Step | Hex |
|------|-----|
| 50 | `#F5F0FF` |
| 100 | `#EAE0FF` |
| 200 | `#D0BFFF` |
| 300 | `#AC8AE8` |
| 400 | `#8B62D4` |
| 500 | `#6E45BC` |
| 600 | `#5230A0` |
| 700 | `#3C1E80` |
| 800 | `#281260` |
| 900 | `#180945` |
| 950 | `#0C0528` |

#### `sakura`

| Step | Hex |
|------|-----|
| 50 | `#FFF8FA` |
| 100 | `#FFEDF3` |
| 200 | `#FFE0EC` |
| 300 | `#FFCFDF` |
| 400 | `#FFC0D4` |
| 500 | `#F5AFCA` |
| 600 | `#E89DBA` |
| 700 | `#D687A8` |
| 800 | `#C07494` |
| 900 | `#A35C7C` |
| 950 | `#8A4665` |

#### `rose-mist`

| Step | Hex |
|------|-----|
| 50 | `#FFF9F9` |
| 100 | `#FCEEED` |
| 200 | `#F8E0DF` |
| 300 | `#F4D0CF` |
| 400 | `#EEBFBE` |
| 500 | `#E5ADAC` |
| 600 | `#D49A99` |
| 700 | `#BC8584` |
| 800 | `#A06F6E` |
| 900 | `#7E5554` |
| 950 | `#5E3C3C` |

### 1b. Typography

#### Font Size (px)

| Token | Value |
| --- | --- |
| `typography.size.xxs` | `10` |
| `typography.size.xs` | `12` |
| `typography.size.sm` | `14` |
| `typography.size.base` | `16` |
| `typography.size.lg` | `18` |
| `typography.size.xl` | `20` |
| `typography.size.2xl` | `24` |
| `typography.size.3xl` | `30` |
| `typography.size.4xl` | `36` |
| `typography.size.5xl` | `48` |
| `typography.size.6xl` | `60` |

#### Font Weight

| Token | Value |
| --- | --- |
| `typography.weight.thin` | `100` |
| `typography.weight.light` | `300` |
| `typography.weight.regular` | `400` |
| `typography.weight.medium` | `500` |
| `typography.weight.bold` | `700` |

#### Letter Spacing

| Token | Value |
| --- | --- |
| `typography.letter-spacing.tighter` | `-2` |
| `typography.letter-spacing.tight` | `-1.5` |
| `typography.letter-spacing.normal` | `0` |
| `typography.letter-spacing.wide` | `2.5` |
| `typography.letter-spacing.wider` | `5` |
| `typography.letter-spacing.widest` | `10` |

#### Font Family

| Token | Value |
| --- | --- |
| `typography.family.heading-graphic` | `Sao Chingcha` |
| `typography.family.heading-graphic2` | `Mitr` |
| `typography.family.heading-content` | `Sarabun` |
| `typography.family.body` | `Sarabun` |
| `typography.family.ui` | `Sarabun` |

#### Font Style

| Token | Value |
| --- | --- |
| `typography.style.thin` | `Thin` |
| `typography.style.light` | `Light` |
| `typography.style.regular` | `Regular` |
| `typography.style.medium` | `Medium` |
| `typography.style.bold` | `Bold` |

> `heading-graphic` (Sao Chingcha) and `heading-graphic2` (Mitr — Google Fonts) are the two **brand display** typefaces; Mitr has broader weight coverage (200–900). Body / heading / UI use Sarabun.

### 1c. Dimension

#### Spacing (px)

| Token | Value |
| --- | --- |
| `dimension.space.0` | `0` |
| `dimension.space.100` | `4` |
| `dimension.space.150` | `6` |
| `dimension.space.200` | `8` |
| `dimension.space.300` | `12` |
| `dimension.space.400` | `16` |
| `dimension.space.600` | `24` |
| `dimension.space.800` | `32` |
| `dimension.space.1200` | `48` |
| `dimension.space.1600` | `64` |
| `dimension.space.2400` | `96` |
| `dimension.space.4000` | `160` |
| `dimension.space.050` | `2` |
| `dimension.space.negative-100` | `-4` |
| `dimension.space.negative-200` | `-8` |
| `dimension.space.negative-300` | `-12` |
| `dimension.space.negative-400` | `-16` |
| `dimension.space.negative-600` | `-24` |

#### Border Radius (px)

| Token | Value |
| --- | --- |
| `dimension.radius.0` | `0` |
| `dimension.radius.100` | `4` |
| `dimension.radius.150` | `6` |
| `dimension.radius.200` | `8` |
| `dimension.radius.300` | `10` |
| `dimension.radius.400` | `12` |
| `dimension.radius.500` | `16` |
| `dimension.radius.600` | `24` |
| `dimension.radius.700` | `32` |
| `dimension.radius.800` | `40` |
| `dimension.radius.full` | `9999` |

#### Size Scale (px)

| Token | Value |
| --- | --- |
| `dimension.size.1` | `1` |
| `dimension.size.50` | `4` |
| `dimension.size.100` | `8` |
| `dimension.size.200` | `12` |
| `dimension.size.250` | `14` |
| `dimension.size.300` | `16` |
| `dimension.size.400` | `20` |
| `dimension.size.500` | `24` |
| `dimension.size.600` | `32` |
| `dimension.size.700` | `36` |
| `dimension.size.800` | `40` |
| `dimension.size.900` | `44` |
| `dimension.size.1000` | `48` |
| `dimension.size.1100` | `52` |
| `dimension.size.1200` | `56` |
| `dimension.size.1300` | `60` |
| `dimension.size.1400` | `64` |
| `dimension.size.1500` | `68` |
| `dimension.size.1600` | `72` |
| `dimension.size.1700` | `80` |
| `dimension.size.1800` | `96` |
| `dimension.size.1900` | `112` |
| `dimension.size.2000` | `128` |
| `dimension.size.2100` | `160` |
| `dimension.size.2200` | `192` |
| `dimension.size.2300` | `224` |
| `dimension.size.2400` | `256` |
| `dimension.size.2500` | `320` |
| `dimension.size.2600` | `384` |
| `dimension.size.2700` | `448` |
| `dimension.size.2800` | `512` |
| `dimension.size.2900` | `640` |
| `dimension.size.3000` | `768` |
| `dimension.size.3100` | `960` |
| `dimension.size.3200` | `1200` |

#### Blur (px)

| Token | Value |
| --- | --- |
| `dimension.blur.100` | `4` |

#### Depth / Z-index

| Token | Value |
| --- | --- |
| `dimension.depth.0` | `0` |
| `dimension.depth.100` | `4` |
| `dimension.depth.200` | `8` |
| `dimension.depth.400` | `16` |
| `dimension.depth.800` | `32` |
| `dimension.depth.1200` | `48` |
| `dimension.depth.025` | `1` |
| `dimension.depth.negative-025` | `-1` |
| `dimension.depth.negative-100` | `-4` |
| `dimension.depth.negative-200` | `-8` |
| `dimension.depth.negative-400` | `-16` |
| `dimension.depth.negative-800` | `-32` |
| `dimension.depth.negative-1200` | `-48` |

#### Stroke Width (px)

| Token | Value |
| --- | --- |
| `dimension.stroke.100` | `1` |
| `dimension.stroke.150` | `1.5` |
| `dimension.stroke.200` | `2` |
| `dimension.stroke.300` | `3` |
| `dimension.stroke.400` | `4` |
| `dimension.stroke.500` | `5` |
| `dimension.stroke.600` | `6` |
| `dimension.stroke.700` | `7` |
| `dimension.stroke.800` | `8` |
| `dimension.stroke.900` | `9` |

> `dimension.radius.1000` (100px) — success card top arch (Archive Mobile [178:122765](https://www.figma.com/design/4WXeI8fgBLQGgQ9VheM2yD/-Archive--Mobile-Application?node-id=178-122765)).

### 1d. Effect Styles

Effect styles define shadows and blurs. Multi-layer effects are separated by `+`.

**Shadow color notation:** `#RRGGBBAA` where AA is alpha in hex (e.g. `1A` = 10%, `26` = 15%, `40` = 25%)

| Token | Effect Layers |
|-------|---------------|
| `effect.input.100` | `drop_shadow blur=0px x=0 y=0 #08A76826` |
| `effect.drop-shadow-top.100` | `drop_shadow blur=4px x=0 y=-1 #64748B1A` |
| `effect.drop-shadow-top.200` | `drop_shadow blur=4px x=0 y=-1 #64748B1A` + `drop_shadow blur=8px x=0 y=-1 #64748B26` |
| `effect.drop-shadow-top.300` | `drop_shadow blur=4px x=0 y=-4 #64748B1A` + `drop_shadow blur=4px x=0 y=-4 #64748B26` |
| `effect.drop-shadow-top.400` | `drop_shadow blur=4px x=0 y=-4 #64748B1A` + `drop_shadow blur=32px x=0 y=-16 #64748B26` |
| `effect.drop-shadow-top.500` | `drop_shadow blur=4px x=0 y=-4 #64748B1A` + `drop_shadow blur=16px x=0 y=-16 #64748B26` |
| `effect.drop-shadow-top.600` | `drop_shadow blur=32px x=0 y=-16 #64748B40` |
| `effect.drop-shadow-bottom.100` | `drop_shadow blur=4px x=0 y=1 #64748B1A` |
| `effect.drop-shadow-bottom.200` | `drop_shadow blur=4px x=0 y=1 #64748B1A` + `drop_shadow blur=8px x=0 y=1 #64748B26` |
| `effect.drop-shadow-bottom.300` | `drop_shadow blur=4px x=0 y=4 #64748B1A` + `drop_shadow blur=4px x=0 y=4 #64748B26` |
| `effect.drop-shadow-bottom.400` | `drop_shadow blur=4px x=0 y=4 #64748B1A` + `drop_shadow blur=32px x=0 y=16 #64748B26` |
| `effect.drop-shadow-bottom.500` | `drop_shadow blur=4px x=0 y=4 #64748B1A` + `drop_shadow blur=16px x=0 y=16 #64748B26` |
| `effect.drop-shadow-bottom.600` | `drop_shadow blur=32px x=0 y=16 #64748B40` |
| `effect.brand-drop-shadow-top.100` | `drop_shadow blur=4px x=0 y=-1 #08A76814` |
| `effect.brand-drop-shadow-top.200` | `drop_shadow blur=4px x=0 y=-1 #08A76814` + `drop_shadow blur=8px x=0 y=-1 #08A7681A` |
| `effect.brand-drop-shadow-top.300` | `drop_shadow blur=4px x=0 y=-4 #08A76814` + `drop_shadow blur=4px x=0 y=-4 #08A7681A` |
| `effect.brand-drop-shadow-top.400` | `drop_shadow blur=4px x=0 y=-4 #08A76814` + `drop_shadow blur=32px x=0 y=-16 #08A7681A` |
| `effect.brand-drop-shadow-top.500` | `drop_shadow blur=4px x=0 y=-4 #08A76814` + `drop_shadow blur=16px x=0 y=-16 #08A7681A` |
| `effect.brand-drop-shadow-top.600` | `drop_shadow blur=32px x=0 y=-16 #08A76833` |
| `effect.brand-drop-shadow-bottom.100` | `drop_shadow blur=4px x=0 y=1 #08A76814` |
| `effect.brand-drop-shadow-bottom.200` | `drop_shadow blur=4px x=0 y=1 #08A76814` + `drop_shadow blur=8px x=0 y=1 #08A7681A` |
| `effect.brand-drop-shadow-bottom.300` | `drop_shadow blur=4px x=0 y=4 #08A76814` + `drop_shadow blur=4px x=0 y=4 #08A7681A` |
| `effect.brand-drop-shadow-bottom.400` | `drop_shadow blur=4px x=0 y=4 #08A76814` + `drop_shadow blur=32px x=0 y=16 #08A7681A` |
| `effect.brand-drop-shadow-bottom.500` | `drop_shadow blur=4px x=0 y=4 #08A76814` + `drop_shadow blur=16px x=0 y=16 #08A7681A` |
| `effect.brand-drop-shadow-bottom.600` | `drop_shadow blur=32px x=0 y=16 #08A76833` |
| `effect.brand-inner-shadow.100` | `inner_shadow blur=2px x=0 y=1 #08A76814` |
| `effect.brand-inner-shadow.200` | `inner_shadow blur=4px x=0 y=2 #08A7681A` |
| `effect.brand-inner-shadow.300` | `inner_shadow blur=6px x=0 y=4 #08A7681F` |
| `effect.brand-inner-shadow.400` | `inner_shadow blur=8px x=0 y=4 #08A76826` |
| `effect.brand-inner-shadow.500` | `inner_shadow blur=12px x=0 y=8 #08A7682E` |
| `effect.brand-inner-shadow.600` | `inner_shadow blur=16px x=0 y=12 #08A76838` |
| `effect.blur.overlay` | `background_blur 8px` |
| `effect.blur.layer` | `layer_blur 6px` |
| `effect.blur.glass` | `background_blur 12px` |
| `effect.indigo-drop-shadow-bottom.100` | `drop_shadow blur=4px x=0 y=1 #818CF814` |
| `effect.indigo-drop-shadow-bottom.200` | `drop_shadow blur=4px x=0 y=1 #818CF814` + `drop_shadow blur=8px x=0 y=1 #818CF81A` |
| `effect.telemed-fab-hover` (→ `--shadow-telemed-fab-hover`) | `drop_shadow blur=28px spread=-4 x=0 y=10 #7A4CE580` + `drop_shadow blur=12px x=0 y=4 #D833A152` — purple→magenta glow on the telemed floating CTA hover, matches its gradient |

#### Motion tokens

CSS custom properties for enter/exit transitions. Declared in [`src/index.css`](src/index.css) `:root`; consumed by `.mih-popup-*` (centered modals) and `.mih-popover-in` (anchored popovers).

| Token | CSS variable | Value | Role |
|---|---|---|---|
| Modal duration | `--motion-duration-modal` | **220 ms** | Overlay + panel enter/exit for `<Popup>` · `<AlertDialog>` |
| Popover duration | `--motion-duration-popover` | **160 ms** | Anchored popover entrance (`Popover`, `ProfilePopover`, search panels) |
| Standard easing | `--motion-easing-standard` | `cubic-bezier(0.4, 0, 0.2, 1)` | Modal fade + lift |
| Popover easing | `--motion-easing-popover` | `ease-out` | Popover scale-in |
| Panel offset Y | `--motion-panel-offset-y` | `var(--dim-space-200)` (**8 px**) | Modal panel start position (enter from below) |
| Panel scale from | `--motion-panel-scale-from` | **0.98** | Modal panel start scale |
| Calendar duration | `--motion-duration-calendar` | **320 ms** | Appointment calendar period slide (`AppointmentCalendarView`) |
| Calendar offset X | `--motion-offset-calendar-x` | `var(--dim-space-600)` (**24 px**) | Horizontal slide distance |
| Enter easing | `--motion-easing-enter` | `cubic-bezier(0.16, 1, 0.3, 1)` | Calendar slide · step transitions |

> **Reduced motion:** `@media (prefers-reduced-motion: reduce)` disables transitions on `.mih-popup-overlay` / `.mih-popup-dialog` and snaps to the visible state.

---

## 2. Brand Tokens

Brand tokens alias Primitive color palettes. There are **10 brand modes**. Each mode defines:
- **primary** — maps to one Primitive color palette
- **secondary** — maps to another Primitive color palette

Switching the brand mode automatically changes all components that alias brand tokens — including Button, Form Input, Checkbox, Radio, Step, Calendar, Timeslot, Search, and more.

### Implementation — CSS Variable System

The preview uses a two-layer CSS variable system so every brand-aliased token updates atomically when the mode changes.

**Layer 1 — Brand Primitives** (switch these per mode):

```css
/* In :root — default (emerald/forest) */
--brand-p50: #F4FBF8;  --brand-p100: #E2F3EB;  --brand-p200: #C5EDDA;
--brand-p300: #84DBB4; --brand-p400: #6AD6A5;  --brand-p500: #54CF97;
--brand-p600: #08A768; --brand-p700: #007549;  --brand-p800: #004C31;
--brand-p900: #003322; --brand-p950: #002318;
/* RGB triplets for rgba() */
--brand-p600-rgb: 8, 167, 104;
--brand-p700-rgb: 0, 117, 73;
```

**Layer 2 — Semantic Tokens** (reference brand primitives):

```css
/* Button */
--surface-brandPrimaryButton-default:       var(--brand-p700);
--surface-brandPrimaryButton-default-hover: var(--brand-p800);
--surface-brandPrimaryButton-tertiary:      var(--brand-p100);
--border-brandPrimaryButton-default:        var(--brand-p700);
--focus-ring-brand: rgba(var(--brand-p600-rgb), 0.25);

/* Form Input */
--border-input-hover:        var(--brand-p600);
--border-input-typing:       var(--brand-p600);
--focus-ring-input-hover:    rgba(var(--brand-p600-rgb), 0.14);

/* Checkbox / Radio */
--surface-checkbox-checked:  var(--brand-p700);
--border-checkbox-hover:     var(--brand-p600);

/* Step */
--sdc-circle-def-bg:  var(--brand-p100);   /* surface/steps/default */
--sdc-circle-cur-bg:  var(--brand-p700);   /* surface/steps/selected */
--sdc-circle-cur-ring: rgba(var(--brand-p600-rgb), 0.15);

/* Calendar, Timeslot, Search, Action List … all alias var(--brand-p*) */
```

**Mode switching** — set `data-brand` on `<html>`:

```js
// JS
function setBrand(mode) {
  document.documentElement.setAttribute('data-brand', mode);
}
```

```css
/* CSS override per mode */
[data-brand="blue-serenity"] {
  --brand-p700: #6676E0;
  --brand-p600: #7A8CF0;
  --brand-p600-rgb: 122, 140, 240;
  --brand-p700-rgb: 102, 118, 224;
  /* … all steps 50–950 … */
}
```

### Brand Modes Overview

| Mode | primary palette | secondary palette | Primary-700 | Primary-600 |
|------|-----------------|-------------------|-------------|-------------|
| `default` | `emerald` | `forest` | `#007549` | `#08A768` |
| `blue-serenity` | `blue-serenity` | `blue-serenity` | `#6676E0` | `#7A8CF0` |
| `dusty-mauve` | `dusty-mauve` | `dusty-mauve` | `#664750` | `#7E5A60` |
| `muted-teal` | `muted-teal` | `muted-teal` | `#486462` | `#5A7D7A` |
| `rose-mist` | `rose-mist` | `rose-mist` | `#BC8584` | `#D49A99` |
| `sakura` | `sakura` | `sakura` | `#D687A8` | `#E89DBA` |
| `soft-lavender` | `soft-lavender` | `soft-lavender` | `#836FC0` | `#9D87D8` |
| `soft-peach` | `soft-peach` | `soft-peach` | `#D0714D` | `#E88A66` |
| `soft-sky` | `soft-sky` | `soft-sky` | `#5594C4` | `#8EC4E5` |
| `warm-gray` | `warm-gray` | `warm-gray` | `#5C504A` | `#726760` |

### Brand Color Values per Mode

Each brand token step (`50`–`950`) resolves to the corresponding step in the mapped Primitive palette.

> **Usage in code:** Reference `brand.primary.{step}` or `brand.secondary.{step}` and switch the brand mode to automatically change colors. Alpha tokens (`primaryBorder`, `PrimaryShadow`, `primaryText`) follow the same brand-mode switch and are pre-baked transparency variants used for shadows, focus rings, and subdued text.

#### Brand: `default`

| Step | primary (`emerald`) | secondary (`forest`) |
|------|------------------------|------------------------|
| 50 | `#F4FBF8` | `#FAFCFA` |
| 100 | `#E2F3EB` | `#EFF5EF` |
| 200 | `#C5EDDA` | `#E1F0E0` |
| 300 | `#91DFBB` | `#A7D2A3` |
| 400 | `#6AD6A5` | `#78B573` |
| 500 | `#54CF97` | `#53954E` |
| 600 | `#08A768` | `#427C3D` |
| 700 | `#007549` | `#366233` |
| 800 | `#004C31` | `#2E4F2C` |
| 900 | `#003322` | `#274126` |
| 950 | `#002318` | `#102310` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#6AD6A5` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#08A768` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#6AD6A5` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#6AD6A5` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#6AD6A5` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#08A768` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#08A768` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#08A768` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#7DE0B3` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#007549` | `50%` | `primaryText/700` |

#### Brand: `blue-serenity`

| Step | primary (`blue-serenity`) | secondary (`blue-serenity`) |
|------|------------------------|------------------------|
| 50 | `#F0F2FF` | `#F0F2FF` |
| 100 | `#E4E8FF` | `#E4E8FF` |
| 200 | `#D0D7FF` | `#D0D7FF` |
| 300 | `#BBC6FE` | `#BBC6FE` |
| 400 | `#A5B3FC` | `#A5B3FC` |
| 500 | `#8EA0F8` | `#8EA0F8` |
| 600 | `#7A8CF0` | `#7A8CF0` |
| 700 | `#6676E0` | `#6676E0` |
| 800 | `#5060C8` | `#5060C8` |
| 900 | `#3A4AAA` | `#3A4AAA` |
| 950 | `#2A3488` | `#2A3488` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#A5B3FC` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#8EA0F8` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#A5B3FC` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#A5B3FC` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#A5B3FC` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#8EA0F8` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#8EA0F8` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#8EA0F8` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#BBC6FE` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#6676E0` | `50%` | `primaryText/700` |

#### Brand: `dusty-mauve`

| Step | primary (`dusty-mauve`) | secondary (`dusty-mauve`) |
|------|------------------------|------------------------|
| 50 | `#F9F4F5` | `#F9F4F5` |
| 100 | `#F2E6E8` | `#F2E6E8` |
| 200 | `#E4CDD0` | `#E4CDD0` |
| 300 | `#D0ADB2` | `#D0ADB2` |
| 400 | `#B58E94` | `#B58E94` |
| 500 | `#977177` | `#977177` |
| 600 | `#7E5A60` | `#7E5A60` |
| 700 | `#664750` | `#664750` |
| 800 | `#503840` | `#503840` |
| 900 | `#3C2A30` | `#3C2A30` |
| 950 | `#281C21` | `#281C21` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#B58E94` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#7E5A60` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#B58E94` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#B58E94` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#B58E94` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#7E5A60` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#7E5A60` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#7E5A60` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#D0ADB2` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#664750` | `50%` | `primaryText/700` |

#### Brand: `muted-teal`

| Step | primary (`muted-teal`) | secondary (`muted-teal`) |
|------|------------------------|------------------------|
| 50 | `#F0F8F7` | `#F0F8F7` |
| 100 | `#DAEEED` | `#DAEEED` |
| 200 | `#B5DBD9` | `#B5DBD9` |
| 300 | `#8DC3C0` | `#8DC3C0` |
| 400 | `#7AABA8` | `#7AABA8` |
| 500 | `#6F9693` | `#6F9693` |
| 600 | `#5A7D7A` | `#5A7D7A` |
| 700 | `#486462` | `#486462` |
| 800 | `#374E4C` | `#374E4C` |
| 900 | `#283A38` | `#283A38` |
| 950 | `#1A2726` | `#1A2726` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#7AABA8` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#6F9693` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#7AABA8` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#7AABA8` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#7AABA8` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#6F9693` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#6F9693` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#6F9693` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#8DC3C0` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#486462` | `50%` | `primaryText/700` |

#### Brand: `rose-mist`

| Step | primary (`rose-mist`) | secondary (`rose-mist`) |
|------|------------------------|------------------------|
| 50 | `#FFF9F9` | `#FFF9F9` |
| 100 | `#FCEEED` | `#FCEEED` |
| 200 | `#F8E0DF` | `#F8E0DF` |
| 300 | `#F4D0CF` | `#F4D0CF` |
| 400 | `#EEBFBE` | `#EEBFBE` |
| 500 | `#E5ADAC` | `#E5ADAC` |
| 600 | `#D49A99` | `#D49A99` |
| 700 | `#BC8584` | `#BC8584` |
| 800 | `#A06F6E` | `#A06F6E` |
| 900 | `#7E5554` | `#7E5554` |
| 950 | `#5E3C3C` | `#5E3C3C` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#EEBFBE` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#D49A99` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#EEBFBE` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#EEBFBE` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#EEBFBE` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#D49A99` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#D49A99` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#D49A99` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#F4D0CF` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#BC8584` | `50%` | `primaryText/700` |

#### Brand: `sakura`

| Step | primary (`sakura`) | secondary (`sakura`) |
|------|------------------------|------------------------|
| 50 | `#FFF8FA` | `#FFF8FA` |
| 100 | `#FFEDF3` | `#FFEDF3` |
| 200 | `#FFE0EC` | `#FFE0EC` |
| 300 | `#FFCFDF` | `#FFCFDF` |
| 400 | `#FFC0D4` | `#FFC0D4` |
| 500 | `#F5AFCA` | `#F5AFCA` |
| 600 | `#E89DBA` | `#E89DBA` |
| 700 | `#D687A8` | `#D687A8` |
| 800 | `#C07494` | `#C07494` |
| 900 | `#A35C7C` | `#A35C7C` |
| 950 | `#8A4665` | `#8A4665` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#FFC0D4` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#E89DBA` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#FFC0D4` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#FFC0D4` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#FFC0D4` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#E89DBA` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#E89DBA` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#E89DBA` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#F9A8D4` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#D687A8` | `50%` | `primaryText/700` |

#### Brand: `soft-lavender`

| Step | primary (`soft-lavender`) | secondary (`soft-lavender`) |
|------|------------------------|------------------------|
| 50 | `#FAF7FF` | `#FAF7FF` |
| 100 | `#F2ECFF` | `#F2ECFF` |
| 200 | `#E5D9FF` | `#E5D9FF` |
| 300 | `#D6C6FA` | `#D6C6FA` |
| 400 | `#C5B2F5` | `#C5B2F5` |
| 500 | `#B49EEE` | `#B49EEE` |
| 600 | `#9D87D8` | `#9D87D8` |
| 700 | `#836FC0` | `#836FC0` |
| 800 | `#6655A0` | `#6655A0` |
| 900 | `#4A3D7A` | `#4A3D7A` |
| 950 | `#302754` | `#302754` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#C5B2F5` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#9D87D8` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#C5B2F5` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#C5B2F5` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#C5B2F5` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#9D87D8` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#9D87D8` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#9D87D8` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#D6C6FA` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#836FC0` | `50%` | `primaryText/700` |

#### Brand: `soft-peach`

| Step | primary (`soft-peach`) | secondary (`soft-peach`) |
|------|------------------------|------------------------|
| 50 | `#FFFAF7` | `#FFFAF7` |
| 100 | `#FEF0E8` | `#FEF0E8` |
| 200 | `#FDE2D0` | `#FDE2D0` |
| 300 | `#FCD1B8` | `#FCD1B8` |
| 400 | `#FBBEA0` | `#FBBEA0` |
| 500 | `#FAA381` | `#FAA381` |
| 600 | `#E88A66` | `#E88A66` |
| 700 | `#D0714D` | `#D0714D` |
| 800 | `#B05A38` | `#B05A38` |
| 900 | `#844026` | `#844026` |
| 950 | `#562814` | `#562814` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#FBBEA0` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#E88A66` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#FBBEA0` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#FBBEA0` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#FBBEA0` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#E88A66` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#E88A66` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#E88A66` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#FCD1B8` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#D0714D` | `50%` | `primaryText/700` |

#### Brand: `soft-sky`

| Step | primary (`soft-sky`) | secondary (`soft-sky`) |
|------|------------------------|------------------------|
| 50 | `#F5FBFF` | `#F5FBFF` |
| 100 | `#E8F5FD` | `#E8F5FD` |
| 200 | `#D6ECFA` | `#D6ECFA` |
| 300 | `#D6ECFA` | `#D6ECFA` |
| 400 | `#A8D3EE` | `#A8D3EE` |
| 500 | `#8EC4E5` | `#8EC4E5` |
| 600 | `#8EC4E5` | `#8EC4E5` |
| 700 | `#5594C4` | `#5594C4` |
| 800 | `#3D78A8` | `#3D78A8` |
| 900 | `#285A82` | `#285A82` |
| 950 | `#183D5C` | `#183D5C` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#A8D3EE` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#8EC4E5` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#A8D3EE` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#A8D3EE` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#A8D3EE` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#8EC4E5` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#8EC4E5` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#8EC4E5` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#A8D3EE` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#5594C4` | `50%` | `primaryText/700` |

#### Brand: `warm-gray`

| Step | primary (`warm-gray`) | secondary (`warm-gray`) |
|------|------------------------|------------------------|
| 50 | `#F5F3F1` | `#F5F3F1` |
| 100 | `#EBE7E4` | `#EBE7E4` |
| 200 | `#D5CEC9` | `#D5CEC9` |
| 300 | `#BDB5AF` | `#BDB5AF` |
| 400 | `#A39A94` | `#A39A94` |
| 500 | `#8A8078` | `#8A8078` |
| 600 | `#726760` | `#726760` |
| 700 | `#5C504A` | `#5C504A` |
| 800 | `#463C37` | `#463C37` |
| 900 | `#312B26` | `#312B26` |
| 950 | `#211D19` | `#211D19` |

| Alpha Token | Hex | Alpha | Source step |
|-------------|-----|-------|-------------|
| `brand.primaryBorder.400` | `#A39A94` | `15%` | `primaryBorder/400` |
| `brand.primaryBorder.600` | `#8A8078` | `15%` | `primaryBorder/600` |
| `brand.PrimaryShadow.400` | `#A39A94` | `8%` | `PrimaryShadow/400` |
| `brand.PrimaryShadow.401` | `#A39A94` | `10%` | `PrimaryShadow/401` |
| `brand.PrimaryShadow.402` | `#A39A94` | `20%` | `PrimaryShadow/402` |
| `brand.PrimaryShadow.600` | `#8A8078` | `8%` | `PrimaryShadow/600` |
| `brand.PrimaryShadow.601` | `#8A8078` | `10%` | `PrimaryShadow/601` |
| `brand.PrimaryShadow.602` | `#8A8078` | `20%` | `PrimaryShadow/602` |
| `brand.primaryText.300` | `#BDB5AF` | `50%` | `primaryText/300` |
| `brand.primaryText.700` | `#5C504A` | `50%` | `primaryText/700` |

---

## 3. Breakpoint Tokens

Breakpoint tokens define responsive layout widths aligned with Tailwind CSS breakpoints.

| Mode | `default` (px) |
|------|----------------|
| `xs` | `402` |
| `sm` | `640` |
| `md` | `768` |
| `lg` | `1024` |
| `xl` | `1280` |
| `2xl` | `1536` |

---

## 4. Semantic Tokens

Semantic tokens assign design meaning to values. They have two modes — **light** and **dark** — and reference Brand (for primary/secondary) or Primitive tokens.

**Categories:** `background` · `text` · `border` · `icon` · `surface`

> **Primary/Secondary tokens change** when the Brand mode is switched. All other semantic tokens (neutral, disabled, positive, warning, progress, danger, info, screen) reference Primitive directly and remain stable across brands.

### How light → dark works

Dark mode **inverts the palette step**, it does not pick new colours:

| Light step | Dark step |
|---|---|
| 50 | 950 |
| 100 | 900 |
| 200 | 800 |
| 300 | 700 |
| 400 | 600 |
| 500 | 500 (unchanged — the midpoint) |

Examples: `neutral/100` `#F4F5F5` → `neutral/900` `#222629` · `red/100` `#FEE2E2` →
`red/900` `#7F1D1D` · `blue/200` `#BFDBFE` → `blue/800` `#1E40AF`. The dark column of
every table below already follows this rule — read the value from the table rather than
computing it, and never sample a colour by eye.

**Implementation notes** (`tokens.css`):

- Overrides live in `[data-mode="dark"]` blocks, applied via `<html data-mode="dark">`.
- They are built from the `--d-*` neutral ramp, which **must be declared inside a
  `[data-mode="dark"]` block**. A ramp declared only under `[data-mobile-theme="dark"]`
  is out of scope on `<html>`, so every `var(--d-*)` reference fails to compute and
  silently falls back to its light value — the whole site stays light with no error.
- Tokens the app reads but `:root` never declares (`var(--x, #light)`) must also be
  declared in the dark block, otherwise they resolve to their light inline fallback in
  both modes.
- `--mobile-*` is scoped to the phone frame and must **not** follow `<html data-mode>`.
- Text/icon tokens paired with a tinted surface must flip together — a dark surface with
  dark text is unreadable.

### 4.1. `background`

| Token | Light | Dark | Alias (Light) |
|-------|-------|------|----------------|
| `background.brandPrimary.default` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `background.brandPrimary.default-hover` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `background.brandPrimary.secondary` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `background.brandPrimary.secondary-hover` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `background.brandPrimary.tertiary` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `background.brandPrimary.tertiary-hover` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `background.brandSecondary.default` | `#53954E` | `#53954E` | `Brand/secondary/500` |
| `background.brandSecondary.default-hover` | `#427C3D` | `#53954E` | `Brand/secondary/600` |
| `background.brandSecondary.secondary` | `#E1F0E0` | `#2E4F2C` | `Brand/secondary/200` |
| `background.brandSecondary.secondary-hover` | `#A7D2A3` | `#366233` | `Brand/secondary/300` |
| `background.brandSecondary.tertiary` | `#EFF5EF` | `#274126` | `Brand/secondary/100` |
| `background.brandSecondary.tertiary-hover` | `#E1F0E0` | `#2E4F2C` | `Brand/secondary/200` |
| `background.neutral.default` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `background.neutral.default-hover` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `background.neutral.secondary` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `background.neutral.secondary-hover` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `background.neutral.tertiary` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `background.neutral.tertiary-hover` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `background.disabled.default` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `background.disabled.default-hover` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `background.disabled.secondary` | `#E2E4E6` | `#E2E4E6` | `Primitive/color/neutral/200` |
| `background.disabled.secondary-hover` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `background.disabled.tertiary` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `background.disabled.tertiary-hover` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `background.positive.default` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `background.positive.default-hover` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `background.positive.secondary` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `background.positive.secondary-hover` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `background.positive.tertiary` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `background.positive.tertiary-hover` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `background.warning.default` | `#F97316` | `#F97316` | `Primitive/color/orange/500` |
| `background.warning.default-hover` | `#EA580C` | `#FB923C` | `Primitive/color/orange/600` |
| `background.warning.secondary` | `#FED7AA` | `#9A3412` | `Primitive/color/orange/200` |
| `background.warning.secondary-hover` | `#FDBA74` | `#C2410C` | `Primitive/color/orange/300` |
| `background.warning.tertiary` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `background.warning.tertiary-hover` | `#FED7AA` | `#9A3412` | `Primitive/color/orange/200` |
| `background.progress.default` | `#EAB308` | `#EAB308` | `Primitive/color/yellow/500` |
| `background.progress.default-hover` | `#CA8A04` | `#FACC15` | `Primitive/color/yellow/600` |
| `background.progress.secondary` | `#FEF08A` | `#854D0E` | `Primitive/color/yellow/200` |
| `background.progress.secondary-hover` | `#FDE047` | `#A16207` | `Primitive/color/yellow/300` |
| `background.progress.tertiary` | `#FEF9C3` | `#713F12` | `Primitive/color/yellow/100` |
| `background.progress.tertiary-hover` | `#FEF08A` | `#854D0E` | `Primitive/color/yellow/200` |
| `background.danger.default` | `#EF4444` | `#EF4444` | `Primitive/color/red/500` |
| `background.danger.default-hover` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `background.danger.secondary` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `background.danger.secondary-hover` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `background.danger.tertiary` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `background.danger.tertiary-hover` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `background.info.default` | `#3B82F6` | `#3B82F6` | `Primitive/color/blue/500` |
| `background.info.default-hover` | `#2563EB` | `#60A5FA` | `Primitive/color/blue/600` |
| `background.info.secondary` | `#BFDBFE` | `#1E40AF` | `Primitive/color/blue/200` |
| `background.info.secondary-hover` | `#93C5FD` | `#1D4ED8` | `Primitive/color/blue/300` |
| `background.info.tertiary` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `background.info.tertiary-hover` | `#BFDBFE` | `#1E40AF` | `Primitive/color/blue/200` |
| `background.screen.100` | `#F5F6FA` | `#0F172A` | `Primitive/color/slate/50` |
| `background.screen.200` | `#F5F7F9` | `#1E293B` | `Primitive/color/slate/100` |
| `background.screen.300` | `#E9EDF5` | `#334155` | `Primitive/color/slate/200` |
| `background.screen.400` | `#CBD5E1` | `#475569` | `Primitive/color/slate/300` |
| `background.screen.default` | `#FFFFFF` | `#0F172A` | `Primitive/color/grey/50` |
| `background.screen.brand-100` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `background.screen.brand-200` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `background.screen.disabled` | `#64748B` | `#94A3B8` | `Primitive/color/slate/500` |
| `background.overlay.default` | `#64748B @ 50%` | `#64748B @ 50%` | `` |
| `background.bg.teal-400` | `#3EC3B2 @ 20%` | `#1B867D @ 20%` | `Primitive/color/bg/teal-400` |
| `background.bg.blue-400` | `#40AAF2 @ 20%` | `#0A73CB @ 20%` | `Primitive/color/bg/sky-cyan-400` |

> `background.bg.*` are translucent ambient-wash fills (alpha-baked primitives, §1 *Special Color Tokens*). Light aliases `color/bg/teal-400` & `color/bg/sky-cyan-400`; dark aliases the deeper `color/bg/teal-600` & `color/bg/sky-cyan-600` — both at 20 % opacity.

### 4.2. `text`

| Token | Light | Dark | Alias (Light) |
|-------|-------|------|----------------|
| `text.brandPrimary.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.brandPrimary.secondary` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `text.brandPrimary.tertiary` | `#6AD6A5` | `#08A768` | `Brand/primary/400` |
| `text.brandPrimary.quaternary` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `text.brandPrimary.on-brand` | `#FFFFFF` | `#002318` | `Primitive/color/grey/50` |
| `text.brandSecondary.default` | `#366233` | `#A7D2A3` | `Brand/secondary/700` |
| `text.brandSecondary.secondary` | `#427C3D` | `#78B573` | `Brand/secondary/600` |
| `text.brandSecondary.tertiary` | `#78B573` | `#427C3D` | `Brand/secondary/400` |
| `text.brandSecondary.quaternary` | `#A7D2A3` | `#366233` | `Brand/secondary/300` |
| `text.brandSecondary.on-default` | `#FFFFFF` | `#102310` | `Primitive/color/grey/50` |
| `text.neutral.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.neutral.secondary` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.neutral.tertiary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.neutral.quaternary` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `text.neutral.on-neutral` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.disabled.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.disabled.secondary` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.disabled.tertiary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.disabled.quaternary` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `text.disabled.on-disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.positive.default` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `text.positive.secondary` | `#1B867D` | `#3EC3B2` | `Primitive/color/teal/600` |
| `text.positive.tertiary` | `#3EC3B2` | `#1B867D` | `Primitive/color/teal/400` |
| `text.positive.quaternary` | `#6CDCCA` | `#196C65` | `Primitive/color/teal/300` |
| `text.positive.on-positive` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.warning.default` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `text.warning.secondary` | `#EA580C` | `#FB923C` | `Primitive/color/orange/600` |
| `text.warning.tertiary` | `#FB923C` | `#EA580C` | `Primitive/color/orange/400` |
| `text.warning.quaternary` | `#FDBA74` | `#C2410C` | `Primitive/color/orange/300` |
| `text.warning.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.progress.default` | `#A16207` | `#FDE047` | `Primitive/color/yellow/700` |
| `text.progress.secondary` | `#CA8A04` | `#FACC15` | `Primitive/color/yellow/600` |
| `text.progress.tertiary` | `#FACC15` | `#CA8A04` | `Primitive/color/yellow/400` |
| `text.progress.quaternary` | `#FDE047` | `#A16207` | `Primitive/color/yellow/300` |
| `text.progress.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.danger.default` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `text.danger.secondary` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `text.danger.tertiary` | `#F87171` | `#DC2626` | `Primitive/color/red/400` |
| `text.danger.quaternary` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `text.danger.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.info.default` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `text.info.secondary` | `#2563EB` | `#60A5FA` | `Primitive/color/blue/600` |
| `text.info.tertiary` | `#60A5FA` | `#2563EB` | `Primitive/color/blue/400` |
| `text.info.quaternary` | `#93C5FD` | `#1D4ED8` | `Primitive/color/blue/300` |
| `text.info.on-info` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.navy.default` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `text.navy.secondary` | `#6366F1` | `#A5B4FC` | `Primitive/color/indigo/600` |
| `text.navy.tertiary` | `#A5B4FC` | `#6366F1` | `Primitive/color/indigo/400` |
| `text.navy.quaternary` | `#C7D2FE` | `#4F46E5` | `Primitive/color/indigo/300` |
| `text.navy.on-info` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.magenta.default` | `#871598` | `#EA78F9` | `Primitive/color/magenta/700` |
| `text.magenta.secondary` | `#A81BBD` | `#DC4CF2` | `Primitive/color/magenta/600` |
| `text.magenta.tertiary` | `#DC4CF2` | `#A81BBD` | `Primitive/color/magenta/400` |
| `text.magenta.quaternary` | `#EA78F9` | `#871598` | `Primitive/color/magenta/300` |
| `text.magenta.on-info` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.pink.default` | `#BE185D` | `#F9A8D4` | `Primitive/color/pink/700` |
| `text.pink.secondary` | `#DB2777` | `#F472B6` | `Primitive/color/pink/600` |
| `text.pink.tertiary` | `#F472B6` | `#DB2777` | `Primitive/color/pink/400` |
| `text.pink.quaternary` | `#F9A8D4` | `#BE185D` | `Primitive/color/pink/300` |
| `text.pink.on-info` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.breadcrumb.pagename` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `text.breadcrumb.current` | `#363B3F` | `#E2E4E6` | `Primitive/color/neutral/800` |
| `text.content.default` | `#363B3F` | `#E2E4E6` | `Primitive/color/neutral/800` |
| `text.content.secondary` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.content.tertiary` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `text.content.quaternary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.content.on-content` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.input.default` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.input.hover` | `#ADB2B7` | `#C9CDD0` | `Primitive/color/neutral/400` |
| `text.input.typing` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.input.filled` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.input.error` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.input.disabled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.input.label` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.input.helper` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.input.errorHelper` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `text.input.require` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `text.primarySearch.default` | `#ADB2B7` | `#858C92` | `Primitive/color/neutral/400` |
| `text.primarySearch.hover` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.primarySearch.typing` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.primarySearch.filled` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.secondarySearch.default` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.secondarySearch.hover` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.secondarySearch.typing` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.secondarySearch.filled` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.brandPrimaryButton.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.brandPrimaryButton.secondary` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `text.brandPrimaryButton.tertiary` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `text.brandPrimaryButton.quaternary` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `text.brandPrimaryButton.on-brand` | `#FFFFFF` | `#002318` | `Primitive/color/grey/50` |
| `text.brandSecondaryButton.default` | `#366233` | `#A7D2A3` | `Brand/secondary/700` |
| `text.brandSecondaryButton.secondary` | `#427C3D` | `#78B573` | `Brand/secondary/600` |
| `text.brandSecondaryButton.tertiary` | `#78B573` | `#427C3D` | `Brand/secondary/400` |
| `text.brandSecondaryButton.quaternary` | `#A7D2A3` | `#366233` | `Brand/secondary/300` |
| `text.brandSecondaryButton.on-default` | `#FFFFFF` | `#102310` | `Primitive/color/grey/50` |
| `text.disabledButton.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.disabledButton.secondary` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.disabledButton.tertiary` | `#ADB2B7` | `#858C92` | `Primitive/color/neutral/400` |
| `text.disabledButton.quaternary` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `text.disabledButton.on-disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.dangerButton.default` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `text.dangerButton.secondary` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `text.dangerButton.tertiary` | `#F87171` | `#DC2626` | `Primitive/color/red/400` |
| `text.dangerButton.quaternary` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `text.dangerButton.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.neutralButton.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.neutralButton.secondary` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.neutralButton.tertiary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.neutralButton.quaternary` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `text.neutralButton.on-neutral` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.neutralTag.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.neutralTag.hover` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.neutralTag.selected` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.neutralTag.selected-hover` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.brandTag.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.brandTag.title` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `text.brandTag.hover` | `#007549` | `#C9CDD0` | `Brand/primary/700` |
| `text.brandTag.selected` | `#007549` | `#C9CDD0` | `Brand/primary/700` |
| `text.brandTag.selected-hover` | `#007549` | `#C9CDD0` | `Brand/primary/700` |
| `text.statuslBadge.neutral` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.statuslBadge.brand` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.statuslBadge.info` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `text.statuslBadge.success` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `text.statuslBadge.warning` | `#A16207` | `#FDE047` | `Primitive/color/yellow/700` |
| `text.statuslBadge.in progress` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `text.statuslBadge.error` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `text.colorfulBadge.badge-on` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.colorfulBadge.grey` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.colorfulBadge.green` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.colorfulBadge.blue` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `text.colorfulBadge.teal` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `text.colorfulBadge.yellow` | `#A16207` | `#FDE047` | `Primitive/color/yellow/700` |
| `text.colorfulBadge.orange` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `text.colorfulBadge.red` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `text.colorfulBadge.indigo` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `text.colorfulBadge.pink` | `#BE185D` | `#F9A8D4` | `Primitive/color/pink/700` |
| `text.colorfulBadge.megenta` | `#871598` | `#EA78F9` | `Primitive/color/magenta/700` |
| `text.colorfulBadge.purple` | `#7E22CE` | `#D8B4FE` | `Primitive/color/purple/700` |
| `text.pagination.default` | `#4D5358` | `#E2E4E6` | `Primitive/color/neutral/700` |
| `text.pagination.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.pagination.selected` | `#007549` | `#FFFFFF` | `Brand/primary/700` |
| `text.tooltip.tooltip-on` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.tabUnderline.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.tabUnderline.hover` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `text.tabUnderline.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.tabUnderline.disabled` | `#C9CDD0` | `#636B72` | `Primitive/color/neutral/300` |
| `text.tabPill.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.tabPill.subtext-default` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.tabPill.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.tabPill.subtext-hover` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.tabPill.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.tabPill.selected-selected` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.tabPill.disabled` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `text.tabPill.selected-disabled` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `text.tabCapsule.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.tabCapsule.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.tabCapsule.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.tabCapsule.disabled` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `text.tabCapsule.inProgress` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.tabCapsule.success` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.timeslot.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.timeslot.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.timeslot.disabled` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `text.timeslot.selected` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `text.accordion.collapsed` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.accordion.expanded` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.accordion.disabled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.accordion.content` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.alert.info` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `text.alert.success` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `text.alert.warning` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `text.alert.error` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `text.alertDialog.text` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.alertDialog.subtext` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.calendar.date-default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.calendar.date-current` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.calendar.date-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.calendar.date-selected` | `#FFFFFF` | `#DADADA` | `Primitive/color/grey/50` |
| `text.calendar.date-disabled` | `#CBD5E1` | `#334155` | `Primitive/color/slate/300` |
| `text.calendar.month` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.calendar slot.date-default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.calendar slot.date-current` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.calendar slot.date-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.calendar slot.date-selected` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `text.calendar slot.date-disabled` | `#CBD5E1` | `#334155` | `Primitive/color/slate/300` |
| `text.calendar slot.month` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.calendar slot.slot-default` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `text.calendar slot.slot-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.calendar slot.slot-selected` | `#FFFFFF` | `#DADADA` | `Primitive/color/grey/50` |
| `text.calendar slot.slot-disabled` | `#CBD5E1` | `#334155` | `Primitive/color/slate/300` |
| `text.calendar slot.text` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.calendar slot.cta-default` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `text.calendar slot.cta-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.calendar slot.cta-disabled` | `#CBD5E1` | `#334155` | `Primitive/color/slate/300` |
| `text.list.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.list.subtext` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `text.list.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.list.subtext-hover` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `text.list.selected` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.list.selected-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.modal.text` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.modal.subtext` | `#ADB2B7` | `#ADB2B7` | `Primitive/color/neutral/400` |
| `text.modal.content` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.sideMenu.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.sideMenu.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.sideMenu.disabled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.sideMenu.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.sideMenu.selected-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.topic.primary-title` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.topic.primary-subtext-default` | `#858C92` | `#ADB2B7` | `Primitive/color/neutral/500` |
| `text.topic.primary-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.topic.primary-subtext-hover` | `#858C92` | `#ADB2B7` | `Primitive/color/neutral/500` |
| `text.topic.primary-disabled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.topic.primary-subtext-disabled` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `text.topic.secondary-title` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.topic.secondary-subtext` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.topic.secondary-subtext-on` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `text.topic.tertiary-title` | `#363B3F` | `#E2E4E6` | `Primitive/color/neutral/800` |
| `text.topic.tertiary-subtext` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `text.topic.tertiary-subtext-on` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `text.navigation.title` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.navigation.subtext` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `text.statusSummaryCard.Info-number` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `text.statusSummaryCard.Indigo-number` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `text.statusSummaryCard.lavender-number` | `#8A6CD8` | `#D0BEFF` | `Primitive/color/lavender/700` |
| `text.statusSummaryCard.sakura-number` | `#D687A8` | `#FFCFDF` | `Primitive/color/sakura/700` |
| `text.statusSummaryCard.Positive-number` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `text.statusSummaryCard.Lime-number` | `#6B8E26` | `#BDE36A` | `Primitive/color/lime/700` |
| `text.statusSummaryCard.Inprogress-number` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `text.statusSummaryCard.Destructive-number` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `text.statusSummaryCard.subtext` | `#858C92` | `#ADB2B7` | `Primitive/color/neutral/500` |
| `text.statusSummaryCard.increase` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.statusSummaryCard.decrease` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `text.categorySummaryCard.A-number` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `text.categorySummaryCard.B-number` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.categorySummaryCard.C-number` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `text.categorySummaryCard.D-number` | `#871598` | `#EA78F9` | `Primitive/color/magenta/700` |
| `text.categorySummaryCard.E-number` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `text.categorySummaryCard.F-number` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `text.categorySummaryCard.G-number` | `#A16207` | `#FDE047` | `Primitive/color/yellow/700` |
| `text.categorySummaryCard.H-number` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `text.categorySummaryCard.Title` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `text.categorySummaryCard.Subtext` | `#858C92` | `#ADB2B7` | `Primitive/color/neutral/500` |
| `text.categorySummaryCard.Warning` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `text.formBuilder.text` | `#007549` | `#54CF97` | `Brand/primary/700` |
| `text.formBuilder.subtext` | `#858C92` | `#C9CDD0` | `Primitive/color/neutral/500` |
| `text.formBuilder.error` | `#DC2626` | `#FCA5A5` | `Primitive/color/red/600` |
| `text.steps.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.steps.select` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.steps.success` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `text.steps.title` | `#007549 @ 50%` | `#7DE0B3 @ 50%` | `Brand/primaryText/700` |
| `text.steps.title-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.steps.title-select` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `text.steps.title-success` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `text.steps.subtitle` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `text.table.title` | `#004C31` | `#C5EDDA` | `Brand/primary/800` |
| `text.table.subtext` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `text.table.footer` | `#004C31` | `#C5EDDA` | `Brand/primary/800` |

### 4.3. `border`

| Token | Light | Dark | Alias (Light) |
|-------|-------|------|----------------|
| `border.brandPrimary.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.brandPrimary.secondary` | `#6AD6A5` | `#08A768` | `Brand/primary/400` |
| `border.brandPrimary.tertiary` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.brandPrimary.quaternary` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `border.brandSecondary.default` | `#427C3D` | `#78B573` | `Brand/secondary/600` |
| `border.brandSecondary.secondary` | `#78B573` | `#427C3D` | `Brand/secondary/400` |
| `border.brandSecondary.tertiary` | `#E1F0E0` | `#2E4F2C` | `Brand/secondary/200` |
| `border.brandSecondary.quaternary` | `#EFF5EF` | `#274126` | `Brand/secondary/100` |
| `border.neutral.default` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `border.neutral.secondary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `border.neutral.tertiary` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.neutral.quaternary` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `border.disabled.default` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `border.disabled.secondary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `border.disabled.tertiary` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.disabled.quaternary` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `border.positive.default` | `#1B867D` | `#3EC3B2` | `Primitive/color/teal/600` |
| `border.positive.secondary` | `#3EC3B2` | `#1B867D` | `Primitive/color/teal/400` |
| `border.positive.tertiary` | `#A2EDDE` | `#195652` | `Primitive/color/teal/200` |
| `border.positive.quaternary` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `border.warning.default` | `#EA580C` | `#FB923C` | `Primitive/color/orange/600` |
| `border.warning.secondary` | `#FB923C` | `#EA580C` | `Primitive/color/orange/400` |
| `border.warning.tertiary` | `#FED7AA` | `#9A3412` | `Primitive/color/orange/200` |
| `border.warning.quaternary` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `border.progress.default` | `#CA8A04` | `#FACC15` | `Primitive/color/yellow/600` |
| `border.progress.secondary` | `#FACC15` | `#CA8A04` | `Primitive/color/yellow/400` |
| `border.progress.tertiary` | `#FEF08A` | `#854D0E` | `Primitive/color/yellow/200` |
| `border.progress.quaternary` | `#FEF9C3` | `#713F12` | `Primitive/color/yellow/100` |
| `border.danger.default` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `border.danger.secondary` | `#F87171` | `#DC2626` | `Primitive/color/red/400` |
| `border.danger.tertiary` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `border.danger.quaternary` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `border.info.default` | `#2563EB` | `#60A5FA` | `Primitive/color/blue/600` |
| `border.info.secondary` | `#60A5FA` | `#2563EB` | `Primitive/color/blue/400` |
| `border.info.tertiary` | `#BFDBFE` | `#1E40AF` | `Primitive/color/blue/200` |
| `border.info.quaternary` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `border.input.default` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `border.input.hover` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `border.input.typing` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `border.input.filled` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.input.error` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `border.input.disabled` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.primarySearch.default` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `border.primarySearch.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.primarySearch.typing` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.primarySearch.filled` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `border.aiSearch.default` | `#818CF8` | `#818CF8` | `Primitive/color/indigo/500` |
| `border.aiSearch.hover` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `border.aiSearch.typing` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `border.aiSearch.filled` | `#818CF8` | `#818CF8` | `Primitive/color/indigo/500` |
| `border.secondarySearch.default` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `border.secondarySearch.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.secondarySearch.typing` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.secondarySearch.filled` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `border.image.default` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `border.brandPrimaryButton.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.brandPrimaryButton.default-hover` | `#004C31` | `#C5EDDA` | `Brand/primary/800` |
| `border.brandPrimaryButton.secondary` | `#6AD6A5` | `#08A768` | `Brand/primary/400` |
| `border.brandPrimaryButton.secondary-hover` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `border.brandPrimaryButton.tertiary` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.brandPrimaryButton.tertiary-hover` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `border.brandPrimaryButton.quaternary` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `border.brandPrimaryButton.quaternary-hover` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.brandSecondaryButton.default` | `#427C3D` | `#78B573` | `Brand/secondary/600` |
| `border.brandSecondaryButton.default-hover` | `#366233` | `#A7D2A3` | `Brand/secondary/700` |
| `border.brandSecondaryButton.secondary` | `#78B573` | `#427C3D` | `Brand/secondary/400` |
| `border.brandSecondaryButton.secondary-hover` | `#53954E` | `#53954E` | `Brand/secondary/500` |
| `border.brandSecondaryButton.tertiary` | `#E1F0E0` | `#2E4F2C` | `Brand/secondary/200` |
| `border.brandSecondaryButton.tertiary-hover` | `#A7D2A3` | `#366233` | `Brand/secondary/300` |
| `border.brandSecondaryButton.quaternary` | `#EFF5EF` | `#274126` | `Brand/secondary/100` |
| `border.brandSecondaryButton.quaternary-hover` | `#E1F0E0` | `#2E4F2C` | `Brand/secondary/200` |
| `border.disabledButton.default` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `border.disabledButton.secondary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `border.disabledButton.tertiary` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.disabledButton.quaternary` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `border.dangerButton.default` | `#DC2626` | `#FCA5A5` | `Primitive/color/red/600` |
| `border.dangerButton.default-hover` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `border.dangerButton.secondary` | `#F87171` | `#DC2626` | `Primitive/color/red/400` |
| `border.dangerButton.secondary-hover` | `#EF4444` | `#EF4444` | `Primitive/color/red/500` |
| `border.dangerButton.tertiary` | `#FECACA` | `#7F1D1D` | `Primitive/color/red/200` |
| `border.dangerButton.tertiary-hover` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `border.dangerButton.quaternary` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `border.dangerButton.quaternary-hover` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `border.neutralButton.default` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `border.neutralButton.default-hover` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `border.neutralButton.secondary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `border.neutralButton.secondary-hover` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `border.neutralButton.tertiary` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `border.neutralButton.tertiary-hover` | `#C9CDD0` | `#636B72` | `Primitive/color/neutral/300` |
| `border.neutralButton.quaternary` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `border.neutralButton.quaternary-hover` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.neutralTag.default` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.neutralTag.default-hover` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.neutralTag.selected` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.neutralTag.quaternary` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.brandTag.default` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.brandTag.default-hover` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.brandTag.selected` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.brandTag.quaternary` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.modal.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.modal.secondary` | `#6AD6A5` | `#08A768` | `Brand/primary/400` |
| `border.modal.tertiary` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.modal.quaternary` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `border.modal.cardBorder` | `#F5F6FA` | `#004C31` | `Primitive/color/slate/50` |
| `border.navigation.default` | `#F4F5F5` | `#4D5358` | `Primitive/color/neutral/100` |
| `border.calendar slot.slot-default` | `#E2F3EB` | `#002318` | `Brand/primary/100` |
| `border.calendar slot.slot-hover` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.calendar slot.slot-selected` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `border.calendar slot.slot-disabled` | `#F5F7F9` | `#0F172A` | `Primitive/color/slate/100` |
| `border.statusBadge.brand` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.statusBadge.neutral` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.statusBadge.info` | `#BFDBFE` | `#1E40AF` | `Primitive/color/blue/200` |
| `border.statusBadge.success` | `#A2EDDE` | `#195652` | `Primitive/color/teal/200` |
| `border.statusBadge.warning` | `#FEF08A` | `#854D0E` | `Primitive/color/yellow/200` |
| `border.statusBadge.warning2` | `#FED7AA` | `#9A3412` | `Primitive/color/orange/200` |
| `border.statusBadge.error` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `border.colorfulBadge.brand` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.colorfulBadge.neutral` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.colorfulBadge.info` | `#BFDBFE` | `#1E40AF` | `Primitive/color/blue/200` |
| `border.colorfulBadge.success` | `#A2EDDE` | `#195652` | `Primitive/color/teal/200` |
| `border.colorfulBadge.warning` | `#FEF08A` | `#854D0E` | `Primitive/color/yellow/200` |
| `border.colorfulBadge.warning2` | `#FED7AA` | `#9A3412` | `Primitive/color/orange/200` |
| `border.colorfulBadge.error` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `border.popover.grey` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.tab.hover` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `border.tab.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.tab.selected-hover` | `#004C31` | `#C5EDDA` | `Brand/primary/800` |
| `border.tab.line` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `border.accordion.default` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `border.accordion.line` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.alert.info` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `border.alert.success` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `border.alert.warning` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `border.alert.error` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `border.alertDialog.default` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.sideMenu.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `border.sideMenu.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.sideMenu.disabled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `border.sideMenu.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.sideMenu.selected-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.timeslot.default` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `border.pill.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `border.tabCapsule.default` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `border.tabCapsule.hover` | `#C5EDDA` | `#007549` | `Brand/primary/200` |
| `border.tabCapsule.selected` | `#C5EDDA` | `#007549` | `Brand/primary/200` |
| `border.tabCapsule.disabled` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `border.tabCapsule.inProgress` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `border.tabCapsule.success` | `#E2E4E6` | `#24A899` | `Primitive/color/neutral/200` |
| `border.card.default` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `border.card.brand-100` | `#C5EDDA` | `#4D5358` | `Primitive/color/emerald/200` |
| `border.card.indigo-100` | `#E0E7FF` | `#4F46E5` | `Primitive/color/indigo/200` |
| `border.card.blue-100` | `#BFDBFE` | `#1D4ED8` | `Primitive/color/blue/200` |
| `border.card.sakura-100` | `#FFE0EC` | `#D687A8` | `Primitive/color/sakura/200` |
| `border.switch.default` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `border.switch.disabled` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `border.formBuilder.default` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.formBuilder.hover` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `border.formBuilder.filled` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `border.formBuilder.error` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `border.steps.default` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `border.table.default` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `border.table.line` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `border.dropdown.default` | `#EBEBEB` | `#F4F4F4` | `Primitive/color/grey/600` |
| `border.actionList.default` | `#EBEBEB` | `#F4F4F4` | `Primitive/color/grey/600` |
| `border.slider.primary` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `border.slider.neutral` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `border.slider.blue` | `#3B82F6` | `#3B82F6` | `Primitive/color/blue/500` |
| `border.slider.indigo` | `#818CF8` | `#818CF8` | `Primitive/color/indigo/500` |
| `border.slider.lavender` | `#C8B6FF` | `#C8B6FF` | `Primitive/color/lavender/500` |
| `border.slider.teal` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `border.slider.lime` | `#A5CF4F` | `#A5CF4F` | `Primitive/color/lime/500` |
| `border.slider.sakura` | `#F5AFCA` | `#F5AFCA` | `Primitive/color/sakura/500` |
| `border.slider.orange` | `#F97316` | `#F97316` | `Primitive/color/orange/500` |
| `border.slider.red` | `#EF4444` | `#EF4444` | `Primitive/color/red/500` |

### 4.4. `icon`

| Token | Light | Dark | Alias (Light) |
|-------|-------|------|----------------|
| `icon.brandPrimary.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.brandPrimary.secondary` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `icon.brandPrimary.tertiary` | `#6AD6A5` | `#08A768` | `Brand/primary/400` |
| `icon.brandPrimary.quaternary` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `icon.brandPrimary.on-brand` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.brandSecondary.default` | `#366233` | `#A7D2A3` | `Brand/secondary/700` |
| `icon.brandSecondary.secondary` | `#427C3D` | `#78B573` | `Brand/secondary/600` |
| `icon.brandSecondary.tertiary` | `#78B573` | `#427C3D` | `Brand/secondary/400` |
| `icon.brandSecondary.quaternary` | `#A7D2A3` | `#366233` | `Brand/secondary/300` |
| `icon.brandSecondary.on-default` | `#FFFFFF` | `#102310` | `Primitive/color/grey/50` |
| `icon.neutral.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.neutral.secondary` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `icon.neutral.tertiary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.neutral.quaternary` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `icon.neutral.on-neutral` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `icon.disabled.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.disabled.secondary` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `icon.disabled.tertiary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.disabled.quaternary` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `icon.disabled.on-disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `icon.positive.default` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `icon.positive.secondary` | `#1B867D` | `#3EC3B2` | `Primitive/color/teal/600` |
| `icon.positive.tertiary` | `#3EC3B2` | `#1B867D` | `Primitive/color/teal/400` |
| `icon.positive.quaternary` | `#6CDCCA` | `#196C65` | `Primitive/color/teal/300` |
| `icon.positive.on-brand` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.positive.success` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `icon.warning.default` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `icon.warning.secondary` | `#EA580C` | `#FB923C` | `Primitive/color/orange/600` |
| `icon.warning.tertiary` | `#FB923C` | `#EA580C` | `Primitive/color/orange/400` |
| `icon.warning.quaternary` | `#FDBA74` | `#C2410C` | `Primitive/color/orange/300` |
| `icon.warning.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.progress.default` | `#A16207` | `#FDE047` | `Primitive/color/yellow/700` |
| `icon.progress.secondary` | `#CA8A04` | `#FACC15` | `Primitive/color/yellow/600` |
| `icon.progress.tertiary` | `#FACC15` | `#CA8A04` | `Primitive/color/yellow/400` |
| `icon.progress.quaternary` | `#FDE047` | `#A16207` | `Primitive/color/yellow/300` |
| `icon.progress.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.danger.default` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `icon.danger.secondary` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `icon.danger.tertiary` | `#F87171` | `#DC2626` | `Primitive/color/red/400` |
| `icon.danger.quaternary` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `icon.danger.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.info.default` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `icon.info.secondary` | `#2563EB` | `#60A5FA` | `Primitive/color/blue/600` |
| `icon.info.tertiary` | `#60A5FA` | `#2563EB` | `Primitive/color/blue/400` |
| `icon.info.quaternary` | `#93C5FD` | `#1D4ED8` | `Primitive/color/blue/300` |
| `icon.info.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.indigo.default` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `icon.indigo.secondary` | `#6366F1` | `#A5B4FC` | `Primitive/color/indigo/600` |
| `icon.indigo.tertiary` | `#A5B4FC` | `#6366F1` | `Primitive/color/indigo/400` |
| `icon.indigo.quaternary` | `#C7D2FE` | `#4F46E5` | `Primitive/color/indigo/300` |
| `icon.indigo.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.lavender.default` | `#8A6CD8` | `#D0BEFF` | `Primitive/color/lavender/700` |
| `icon.lavender.secondary` | `#A98EF0` | `#CBBCFF` | `Primitive/color/lavender/600` |
| `icon.lavender.tertiary` | `#CBBCFF` | `#A98EF0` | `Primitive/color/lavender/400` |
| `icon.lavender.quaternary` | `#D0BEFF` | `#8A6CD8` | `Primitive/color/lavender/300` |
| `icon.lavender.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.lime.default` | `#6B8E26` | `#BDE36A` | `Primitive/color/lime/700` |
| `icon.lime.secondary` | `#88B033` | `#ADD654` | `Primitive/color/lime/600` |
| `icon.lime.tertiary` | `#ADD654` | `#88B033` | `Primitive/color/lime/400` |
| `icon.lime.quaternary` | `#BDE36A` | `#6B8E26` | `Primitive/color/lime/300` |
| `icon.lime.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.sakura.default` | `#D687A8` | `#FFCFDF` | `Primitive/color/sakura/700` |
| `icon.sakura.secondary` | `#E89DBA` | `#FFC0D4` | `Primitive/color/sakura/600` |
| `icon.sakura.tertiary` | `#FFC0D4` | `#E89DBA` | `Primitive/color/sakura/400` |
| `icon.sakura.quaternary` | `#FFCFDF` | `#D687A8` | `Primitive/color/sakura/300` |
| `icon.sakura.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.pink.default` | `#BE185D` | `#F9A8D4` | `Primitive/color/pink/700` |
| `icon.pink.secondary` | `#DB2777` | `#F472B6` | `Primitive/color/pink/600` |
| `icon.pink.tertiary` | `#F472B6` | `#DB2777` | `Primitive/color/pink/400` |
| `icon.pink.quaternary` | `#F9A8D4` | `#BE185D` | `Primitive/color/pink/300` |
| `icon.pink.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.navy.default` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `icon.navy.secondary` | `#6366F1` | `#A5B4FC` | `Primitive/color/indigo/600` |
| `icon.navy.tertiary` | `#A5B4FC` | `#6366F1` | `Primitive/color/indigo/400` |
| `icon.navy.quaternary` | `#C7D2FE` | `#4F46E5` | `Primitive/color/indigo/300` |
| `icon.navy.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.magenta.default` | `#871598` | `#EA78F9` | `Primitive/color/magenta/700` |
| `icon.magenta.secondary` | `#A81BBD` | `#DC4CF2` | `Primitive/color/magenta/600` |
| `icon.magenta.tertiary` | `#DC4CF2` | `#A81BBD` | `Primitive/color/magenta/400` |
| `icon.magenta.quaternary` | `#EA78F9` | `#871598` | `Primitive/color/magenta/300` |
| `icon.magenta.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.input.default` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.input.hover` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `icon.input.typing` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.input.filled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.input.error` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.input.disabled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.primarySearch.default` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `icon.primarySearch.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.primarySearch.typing` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.primarySearch.filled` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `icon.aiSearch.default` | `#6366F1` | `#A5B4FC` | `Primitive/color/indigo/600` |
| `icon.aiSearch.hover` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `icon.aiSearch.typing` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `icon.aiSearch.filled` | `#6366F1` | `#A5B4FC` | `Primitive/color/indigo/600` |
| `icon.secondarySearch.default` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.secondarySearch.hover` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.secondarySearch.typing` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.secondarySearch.filled` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.brandPrimaryButton.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.brandPrimaryButton.default-hover` | `#004C31` | `#C5EDDA` | `Brand/primary/800` |
| `icon.brandPrimaryButton.secondary` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `icon.brandPrimaryButton.secondary-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.brandPrimaryButton.tertiary` | `#6AD6A5` | `#08A768` | `Brand/primary/400` |
| `icon.brandPrimaryButton.tertiary-hover` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `icon.brandPrimaryButton.quaternary` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `icon.brandPrimaryButton.quaternary-hover` | `#6AD6A5` | `#08A768` | `Brand/primary/400` |
| `icon.brandPrimaryButton.on-brand` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.brandSecondaryButton.default` | `#366233` | `#A7D2A3` | `Brand/secondary/700` |
| `icon.brandSecondaryButton.default-hover` | `#2E4F2C` | `#E1F0E0` | `Brand/secondary/800` |
| `icon.brandSecondaryButton.secondary` | `#427C3D` | `#78B573` | `Brand/secondary/600` |
| `icon.brandSecondaryButton.secondary-hover` | `#366233` | `#A7D2A3` | `Brand/secondary/700` |
| `icon.brandSecondaryButton.tertiary` | `#78B573` | `#427C3D` | `Brand/secondary/400` |
| `icon.brandSecondaryButton.tertiary-hover` | `#53954E` | `#53954E` | `Brand/secondary/500` |
| `icon.brandSecondaryButton.quaternary` | `#A7D2A3` | `#366233` | `Brand/secondary/300` |
| `icon.brandSecondaryButton.quaternary-hover` | `#78B573` | `#427C3D` | `Brand/secondary/400` |
| `icon.brandSecondaryButton.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.disablediconButton.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.disablediconButton.secondary` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `icon.disablediconButton.tertiary` | `#ADB2B7` | `#858C92` | `Primitive/color/neutral/400` |
| `icon.disablediconButton.quaternary` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `icon.disablediconButton.on-disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `icon.dangerButton.default` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `icon.dangerButton.secondary` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `icon.dangerButton.tertiary` | `#F87171` | `#DC2626` | `Primitive/color/red/400` |
| `icon.dangerButton.quaternary` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `icon.dangerButton.on-default` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `icon.neutralButton.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.neutralButton.secondary` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `icon.neutralButton.tertiary` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.neutralButton.quaternary` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `icon.neutralButton.on-neutral` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `icon.tag.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.tag.tailing` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `icon.tag.hover` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.tag.selected` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.tag.selected-hover` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.brandTag.default` | `#007549` | `#08A768` | `Brand/primary/700` |
| `icon.brandTag.tailing` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `icon.brandTag.hover` | `#007549` | `#08A768` | `Brand/primary/700` |
| `icon.brandTag.selected` | `#007549` | `#08A768` | `Brand/primary/700` |
| `icon.brandTag.selected-hover` | `#007549` | `#08A768` | `Brand/primary/700` |
| `icon.colorfulBadge.badge-on` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `icon.colorfulBadge.grey` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.colorfulBadge.green` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.colorfulBadge.blue` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `icon.colorfulBadge.teal` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `icon.colorfulBadge.yellow` | `#A16207` | `#FDE047` | `Primitive/color/yellow/700` |
| `icon.colorfulBadge.orange` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `icon.colorfulBadge.red` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `icon.colorfulBadge.indigo` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `icon.colorfulBadge.pink` | `#BE185D` | `#F9A8D4` | `Primitive/color/pink/700` |
| `icon.colorfulBadge.megenta` | `#871598` | `#EA78F9` | `Primitive/color/magenta/700` |
| `icon.colorfulBadge.purple` | `#7E22CE` | `#D8B4FE` | `Primitive/color/purple/700` |
| `icon.statuslBadge.neutral` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.statuslBadge.brand` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.statuslBadge.info` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `icon.statuslBadge.success` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `icon.statuslBadge.warning` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `icon.statuslBadge.error` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `icon.tabUnderline.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.tabUnderline.hover` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `icon.tabUnderline.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.tabUnderline.selected-hover` | `#004C31` | `#C5EDDA` | `Brand/primary/800` |
| `icon.tabUnderline.disabled` | `#C9CDD0` | `#636B72` | `Primitive/color/neutral/300` |
| `icon.tabCapsule.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.tabCapsule.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.tabCapsule.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.tabCapsule.disabled` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `icon.tabCapsule.inProgress` | `#F97316` | `#F97316` | `Primitive/color/orange/500` |
| `icon.tabCapsule.success` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `icon.tabPill.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.tabPill.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.tabPill.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.tabPill.disabled` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `icon.accordion.collapsed` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.accordion.expanded` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.accordion.disabled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.alert.info` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `icon.alert.success` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `icon.alert.warning` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `icon.alert.error` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `icon.alertDialog.info` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.alertDialog.success` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `icon.alertDialog.warning` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `icon.alertDialog.error` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `icon.calendar.selected` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.calendar.hover` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `icon.calendar slot.selected` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `icon.list.primary-default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.list.primary-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.list.primary-selected` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.list.primary-selected-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.list.secondary-default` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `icon.list.secondary-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.list.secondary-selected` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `icon.list.secondary-selected-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.sideMenu.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `icon.sideMenu.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.sideMenu.disabled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.sideMenu.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.sideMenu.selected-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.topic.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.topic.hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `icon.topic.disabled` | `#ADB2B7` | `#636B72` | `Primitive/color/neutral/400` |
| `icon.statusSummaryCard.Info` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `icon.statusSummaryCard.Positive` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `icon.statusSummaryCard.Inprogress` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `icon.statusSummaryCard.Destructive` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `icon.statusSummaryCard.increase` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `icon.statusSummaryCard.decrease` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `icon.formBuilder.default` | `#007549` | `#CBD5E1` | `Brand/primary/700` |
| `icon.formBuilder.iconBox-default` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `icon.formBuilder.content` | `#007549` | `#08A768` | `Brand/primary/700` |
| `icon.formBuilder.iconBox-hover` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `icon.formBuilder.error` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `icon.step.success-on` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |

### 4.5. `surface`

| Token | Light | Dark | Alias (Light) |
|-------|-------|------|----------------|
| `surface.brandPrimary.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `surface.brandPrimary.default-hover` | `#004C31` | `#C5EDDA` | `Brand/primary/800` |
| `surface.brandPrimary.secondary` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `surface.brandPrimary.secondary-hover` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `surface.brandPrimary.tertiary` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.brandPrimary.tertiary-hover` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `surface.brandPrimary.quaternary` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.brandPrimary.quaternary-hover` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.brandSecondary.default` | `#366233` | `#A7D2A3` | `Brand/secondary/700` |
| `surface.brandSecondary.default-hover` | `#2E4F2C` | `#E1F0E0` | `Brand/secondary/800` |
| `surface.brandSecondary.secondary` | `#E1F0E0` | `#2E4F2C` | `Brand/secondary/200` |
| `surface.brandSecondary.secondary-hover` | `#A7D2A3` | `#366233` | `Brand/secondary/300` |
| `surface.brandSecondary.tertiary` | `#EFF5EF` | `#274126` | `Brand/secondary/100` |
| `surface.brandSecondary.tertiary-hover` | `#E1F0E0` | `#2E4F2C` | `Brand/secondary/200` |
| `surface.brandSecondary.quaternary` | `#FAFCFA` | `#102310` | `Brand/secondary/50` |
| `surface.brandSecondary.quaternary-hover` | `#EFF5EF` | `#274126` | `Brand/secondary/100` |
| `surface.neutral.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `surface.neutral.default-hover` | `#363B3F` | `#E2E4E6` | `Primitive/color/neutral/800` |
| `surface.neutral.secondary` | `#363B3F` | `#E2E4E6` | `Primitive/color/neutral/800` |
| `surface.neutral.secondary-hover` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `surface.neutral.tertiary` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `surface.neutral.tertiary-hover` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `surface.neutral.quaternary` | `#F9FBFB` | `#363B3F` | `Primitive/color/neutral/50` |
| `surface.neutral.quaternary-hover` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `surface.disabled.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `surface.disabled.default-hover` | `#363B3F` | `#E2E4E6` | `Primitive/color/neutral/800` |
| `surface.disabled.secondary` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `surface.disabled.secondary-hover` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `surface.disabled.tertiary` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `surface.disabled.tertiary-hover` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `surface.disabled.quaternary` | `#F9FBFB` | `#111314` | `Primitive/color/neutral/50` |
| `surface.disabled.quaternary-hover` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `surface.positive.default` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `surface.positive.default-hover` | `#195652` | `#A2EDDE` | `Primitive/color/teal/800` |
| `surface.positive.secondary` | `#A2EDDE` | `#195652` | `Primitive/color/teal/200` |
| `surface.positive.secondary-hover` | `#6CDCCA` | `#196C65` | `Primitive/color/teal/300` |
| `surface.positive.tertiary` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `surface.positive.tertiary-hover` | `#A2EDDE` | `#195652` | `Primitive/color/teal/200` |
| `surface.positive.quaternary` | `#F4FBFA` | `#082B2A` | `Primitive/color/teal/50` |
| `surface.positive.quaternary-hover` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `surface.warning.default` | `#C2410C` | `#FDBA74` | `Primitive/color/orange/700` |
| `surface.warning.default-hover` | `#9A3412` | `#FED7AA` | `Primitive/color/orange/800` |
| `surface.warning.secondary` | `#FED7AA` | `#9A3412` | `Primitive/color/orange/200` |
| `surface.warning.secondary-hover` | `#FDBA74` | `#C2410C` | `Primitive/color/orange/300` |
| `surface.warning.tertiary` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `surface.warning.tertiary-hover` | `#FED7AA` | `#9A3412` | `Primitive/color/orange/200` |
| `surface.warning.quaternary` | `#FFF9F0` | `#431407` | `Primitive/color/orange/50` |
| `surface.warning.quaternary-hover` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `surface.progress.default` | `#A16207` | `#FDE047` | `Primitive/color/yellow/700` |
| `surface.progress.default-hover` | `#854D0E` | `#FEF08A` | `Primitive/color/yellow/800` |
| `surface.progress.secondary` | `#FEF08A` | `#854D0E` | `Primitive/color/yellow/200` |
| `surface.progress.secondary-hover` | `#FDE047` | `#A16207` | `Primitive/color/yellow/300` |
| `surface.progress.tertiary` | `#FEF9C3` | `#713F12` | `Primitive/color/yellow/100` |
| `surface.progress.tertiary-hover` | `#FEF08A` | `#854D0E` | `Primitive/color/yellow/200` |
| `surface.progress.quaternary` | `#FBFAF4` | `#422006` | `Primitive/color/yellow/50` |
| `surface.progress.quaternary-hover` | `#FEF9C3` | `#713F12` | `Primitive/color/yellow/100` |
| `surface.danger.default` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `surface.danger.default-hover` | `#991B1B` | `#FECACA` | `Primitive/color/red/800` |
| `surface.danger.secondary` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `surface.danger.secondary-hover` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `surface.danger.tertiary` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.danger.tertiary-hover` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `surface.danger.quaternary` | `#FFF0F0` | `#450A0A` | `Primitive/color/red/50` |
| `surface.danger.quaternary-hover` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.info.default` | `#1D4ED8` | `#93C5FD` | `Primitive/color/blue/700` |
| `surface.info.default-hover` | `#1E40AF` | `#BFDBFE` | `Primitive/color/blue/800` |
| `surface.info.secondary` | `#BFDBFE` | `#1E40AF` | `Primitive/color/blue/200` |
| `surface.info.secondary-hover` | `#93C5FD` | `#1D4ED8` | `Primitive/color/blue/300` |
| `surface.info.tertiary` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `surface.info.tertiary-hover` | `#BFDBFE` | `#1E40AF` | `Primitive/color/blue/200` |
| `surface.info.quaternary` | `#F0F6FF` | `#172554` | `Primitive/color/blue/50` |
| `surface.info.quaternary-hover` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `surface.navy.default` | `#4F46E5` | `#C7D2FE` | `Primitive/color/indigo/700` |
| `surface.navy.default-hover` | `#4338CA` | `#E0E7FF` | `Primitive/color/indigo/800` |
| `surface.navy.secondary` | `#E0E7FF` | `#4338CA` | `Primitive/color/indigo/200` |
| `surface.navy.secondary-hover` | `#C7D2FE` | `#4F46E5` | `Primitive/color/indigo/300` |
| `surface.navy.tertiary` | `#EEF2FF` | `#3730A3` | `Primitive/color/indigo/100` |
| `surface.navy.tertiary-hover` | `#E0E7FF` | `#4338CA` | `Primitive/color/indigo/200` |
| `surface.navy.quaternary` | `#F3F5FC` | `#1E1B4B` | `Primitive/color/indigo/50` |
| `surface.navy.quaternary-hover` | `#EEF2FF` | `#3730A3` | `Primitive/color/indigo/100` |
| `surface.magenta.default` | `#871598` | `#EA78F9` | `Primitive/color/magenta/700` |
| `surface.magenta.default-hover` | `#6E1379` | `#F3ADFD` | `Primitive/color/magenta/800` |
| `surface.magenta.secondary` | `#F3ADFD` | `#6E1379` | `Primitive/color/magenta/200` |
| `surface.magenta.secondary-hover` | `#EA78F9` | `#871598` | `Primitive/color/magenta/300` |
| `surface.magenta.tertiary` | `#F9D6FE` | `#5A125F` | `Primitive/color/magenta/100` |
| `surface.magenta.tertiary-hover` | `#F3ADFD` | `#6E1379` | `Primitive/color/magenta/200` |
| `surface.magenta.quaternary` | `#FAF3FC` | `#380040` | `Primitive/color/magenta/50` |
| `surface.magenta.quaternary-hover` | `#F9D6FE` | `#5A125F` | `Primitive/color/magenta/100` |
| `surface.pink.default` | `#BE185D` | `#F9A8D4` | `Primitive/color/pink/700` |
| `surface.pink.default-hover` | `#9D174D` | `#FBCFE8` | `Primitive/color/pink/800` |
| `surface.pink.secondary` | `#FBCFE8` | `#9D174D` | `Primitive/color/pink/200` |
| `surface.pink.secondary-hover` | `#F9A8D4` | `#BE185D` | `Primitive/color/pink/300` |
| `surface.pink.tertiary` | `#FCE7F3` | `#831843` | `Primitive/color/pink/100` |
| `surface.pink.tertiary-hover` | `#FBCFE8` | `#9D174D` | `Primitive/color/pink/200` |
| `surface.pink.quaternary` | `#FBF4F8` | `#500724` | `Primitive/color/pink/50` |
| `surface.pink.quaternary-hover` | `#FCE7F3` | `#831843` | `Primitive/color/pink/100` |
| `surface.input.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.input.disabled` | `#F4F5F5` | `#363B3F` | `Primitive/color/neutral/100` |
| `surface.search.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.modal.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.modal.disabled` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `surface.switch.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.switch.icon` | `#FFFFFF` | `#FFFFFF` | `Primitive/color/grey/50` |
| `surface.switch.icon-disabled` | `#F4F5F5` | `#636B72` | `Primitive/color/neutral/100` |
| `surface.switch.off` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `surface.switch.off-hover` | `#C9CDD0` | `#636B72` | `Primitive/color/neutral/300` |
| `surface.switch.on` | `#1B867D` | `#3EC3B2` | `Primitive/color/teal/600` |
| `surface.switch.hover` | `#3EC3B2` | `#1B867D` | `Primitive/color/teal/400` |
| `surface.switch.checkbox-dedault-hover` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `surface.switch.on-hover` | `#196C65` | `#6CDCCA` | `Primitive/color/teal/700` |
| `surface.switch.checkbox` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `surface.switch.checkbox-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `surface.switch.radio-dedault-hover` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `surface.switch.radio` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `surface.switch.radio-hover` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `surface.switch.disabled` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `surface.switch.A` | `#BFDBFE` | `#1E40AF` | `Primitive/color/blue/200` |
| `surface.switch.B` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `surface.switch.C` | `#A2EDDE` | `#195652` | `Primitive/color/teal/200` |
| `surface.switch.D` | `#F3ADFD` | `#6E1379` | `Primitive/color/magenta/200` |
| `surface.switch.E` | `#E0E7FF` | `#4338CA` | `Primitive/color/indigo/200` |
| `surface.switch.F` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `surface.switch.G` | `#FEF08A` | `#854D0E` | `Primitive/color/yellow/200` |
| `surface.switch.H` | `#FED7AA` | `#9A3412` | `Primitive/color/orange/200` |
| `surface.tooltip.default` | `#007549` | `#08A768` | `Brand/primary/700` |
| `surface.tooltip.white` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.notification.default` | `#DB2777` | `#F472B6` | `Primitive/color/pink/600` |
| `surface.navigation.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.card.100` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.card.200` | `#FDFDFD` | `#363B3F` | `Primitive/color/grey/100` |
| `surface.card.300` | `#F4F5F5` | `#363B3F` | `Primitive/color/neutral/100` |
| `surface.card.brand-100` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.card.indigo-100` | `#F3F5FC` | `#1E1B4B` | `Primitive/color/indigo/50` |
| `surface.card.sakura-200` | `#FFEDF3` | `#A35C7C` | `Primitive/color/sakura/100` |
| `surface.card.blue-100` | `#F0F6FF` | `#172554` | `Primitive/color/blue/50` |
| `surface.brandPrimaryButton.default` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `surface.brandPrimaryButton.default-hover` | `#004C31` | `#C5EDDA` | `Brand/primary/800` |
| `surface.brandPrimaryButton.secondary` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `surface.brandPrimaryButton.secondary-hover` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `surface.brandPrimaryButton.tertiary` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.brandPrimaryButton.tertiary-hover` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `surface.brandPrimaryButton.quaternary` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.brandPrimaryButton.quaternary-hover` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.brandPrimaryButton.quinary` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `surface.brandPrimaryButton.quinary-hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.brandSecondarybutton.default` | `#366233` | `#A7D2A3` | `Brand/secondary/700` |
| `surface.brandSecondarybutton.default-hover` | `#2E4F2C` | `#E1F0E0` | `Brand/secondary/800` |
| `surface.brandSecondarybutton.secondary` | `#E1F0E0` | `#2E4F2C` | `Brand/secondary/200` |
| `surface.brandSecondarybutton.secondary-hover` | `#A7D2A3` | `#366233` | `Brand/secondary/300` |
| `surface.brandSecondarybutton.tertiary` | `#EFF5EF` | `#274126` | `Brand/secondary/100` |
| `surface.brandSecondarybutton.tertiary-hover` | `#E1F0E0` | `#2E4F2C` | `Brand/secondary/200` |
| `surface.brandSecondarybutton.quaternary` | `#FAFCFA` | `#102310` | `Brand/secondary/50` |
| `surface.brandSecondarybutton.quaternary-hover` | `#EFF5EF` | `#274126` | `Brand/secondary/100` |
| `surface.brandSecondarybutton.quinary` | `#FFFFFF` | `#020617` | `Primitive/color/grey/50` |
| `surface.brandSecondarybutton.quinary-hover` | `#FAFCFA` | `#102310` | `Brand/secondary/50` |
| `surface.disabledButton.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `surface.disabledButton.default-hover` | `#363B3F` | `#E2E4E6` | `Primitive/color/neutral/800` |
| `surface.disabledButton.secondary` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `surface.disabledButton.secondary-hover` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `surface.disabledButton.tertiary` | `#F4F5F5` | `#363B3F` | `Primitive/color/neutral/100` |
| `surface.disabledButton.tertiary-hover` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `surface.disabledButton.quaternary` | `#F9FBFB` | `#111314` | `Primitive/color/neutral/50` |
| `surface.disabledButton.quaternary-hover` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `surface.dangerButton.default` | `#B91C1C` | `#FCA5A5` | `Primitive/color/red/700` |
| `surface.dangerButton.default-hover` | `#991B1B` | `#FECACA` | `Primitive/color/red/800` |
| `surface.dangerButton.secondary` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `surface.dangerButton.secondary-hover` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `surface.dangerButton.tertiary` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.dangerButton.tertiary-hover` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `surface.dangerButton.quaternary` | `#FFF0F0` | `#450A0A` | `Primitive/color/red/50` |
| `surface.dangerButton.quaternary-hover` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.dangerButton.quinary` | `#FFFFFF` | `#DADADA` | `Primitive/color/grey/50` |
| `surface.dangerButton.quinary-hover` | `#FFF0F0` | `#450A0A` | `Primitive/color/red/50` |
| `surface.neutralButton.default` | `#4D5358` | `#C9CDD0` | `Primitive/color/neutral/700` |
| `surface.neutralButton.default-hover` | `#363B3F` | `#E2E4E6` | `Primitive/color/neutral/800` |
| `surface.neutralButton.secondary` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `surface.neutralButton.secondary-hover` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `surface.neutralButton.tertiary` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `surface.neutralButton.tertiary-hover` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `surface.neutralButton.quaternary` | `#F9FBFB` | `#111314` | `Primitive/color/neutral/50` |
| `surface.neutralButton.quaternary-hover` | `#F4F5F5` | `#222629` | `Primitive/color/neutral/100` |
| `surface.neutralButton.quinary` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.neutralButton.quinary-hover` | `#F9FBFB` | `#111314` | `Primitive/color/neutral/50` |
| `surface.neutralTag.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.neutralTag.default-hover` | `#F9FBFB` | `#222629` | `Primitive/color/neutral/50` |
| `surface.neutralTag.selected` | `#F4F5F5` | `#363B3F` | `Primitive/color/neutral/100` |
| `surface.neutralTag.selectedy-hover` | `#E2E4E6` | `#4D5358` | `Primitive/color/neutral/200` |
| `surface.brandTag.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.brandTag.default-hover` | `#F4FBF8` | `#003322` | `Brand/primary/50` |
| `surface.brandTag.selected` | `#F4FBF8` | `#003322` | `Brand/primary/50` |
| `surface.brandTag.selected-hover` | `#E2F3EB` | `#004C31` | `Brand/primary/100` |
| `surface.statusBadge.brand` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.statusBadge.neutral` | `#F5F7F9` | `#334155` | `Primitive/color/slate/100` |
| `surface.statusBadge.info` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `surface.statusBadge.success` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `surface.statusBadge.warning` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `surface.statusBadge.error` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.colorfulBadge.green-bold` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `surface.colorfulBadge.green-medium` | `#91DFBB` | `#007549` | `Brand/primary/300` |
| `surface.colorfulBadge.green` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.colorfulBadge.grey-bold` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `surface.colorfulBadge.grey-medium` | `#C9CDD0` | `#4D5358` | `Primitive/color/neutral/300` |
| `surface.colorfulBadge.grey` | `#F5F7F9` | `#0F172A` | `Primitive/color/slate/100` |
| `surface.colorfulBadge.blue-bold` | `#3B82F6` | `#3B82F6` | `Primitive/color/blue/500` |
| `surface.colorfulBadge.blue-medium` | `#93C5FD` | `#1D4ED8` | `Primitive/color/blue/300` |
| `surface.colorfulBadge.blue` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `surface.colorfulBadge.teal-bold` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `surface.colorfulBadge.teal-medium` | `#6CDCCA` | `#196C65` | `Primitive/color/teal/300` |
| `surface.colorfulBadge.teal` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `surface.colorfulBadge.yellow-bold` | `#EAB308` | `#EAB308` | `Primitive/color/yellow/500` |
| `surface.colorfulBadge.yellow-medium` | `#FDE047` | `#A16207` | `Primitive/color/yellow/300` |
| `surface.colorfulBadge.yellow` | `#FEF9C3` | `#713F12` | `Primitive/color/yellow/100` |
| `surface.colorfulBadge.orange-bold` | `#F97316` | `#F97316` | `Primitive/color/orange/500` |
| `surface.colorfulBadge.orange-medium` | `#FDBA74` | `#C2410C` | `Primitive/color/orange/300` |
| `surface.colorfulBadge.orange` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `surface.colorfulBadge.red-bold` | `#EF4444` | `#EF4444` | `Primitive/color/red/500` |
| `surface.colorfulBadge.red-medium` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `surface.colorfulBadge.red` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.colorfulBadge.indigo-bold` | `#818CF8` | `#818CF8` | `Primitive/color/indigo/500` |
| `surface.colorfulBadge.indigo-medium` | `#C7D2FE` | `#4F46E5` | `Primitive/color/indigo/300` |
| `surface.colorfulBadge.indigo` | `#EEF2FF` | `#3730A3` | `Primitive/color/indigo/100` |
| `surface.colorfulBadge.pink-bold` | `#EC4899` | `#EC4899` | `Primitive/color/pink/500` |
| `surface.colorfulBadge.pink-medium` | `#F9A8D4` | `#BE185D` | `Primitive/color/pink/300` |
| `surface.colorfulBadge.pink` | `#FCE7F3` | `#831843` | `Primitive/color/pink/100` |
| `surface.colorfulBadge.megenta-bold` | `#C926E0` | `#C926E0` | `Primitive/color/magenta/500` |
| `surface.colorfulBadge.megenta-medium` | `#EA78F9` | `#871598` | `Primitive/color/magenta/300` |
| `surface.colorfulBadge.megenta` | `#F9D6FE` | `#5A125F` | `Primitive/color/magenta/100` |
| `surface.colorfulBadge.purple-bold` | `#A855F7` | `#A855F7` | `Primitive/color/purple/500` |
| `surface.colorfulBadge.purple-medium` | `#D8B4FE` | `#7E22CE` | `Primitive/color/purple/300` |
| `surface.colorfulBadge.purple` | `#F3E8FF` | `#581C87` | `Primitive/color/purple/100` |
| `surface.pagination.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.pagination.hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.pagination.selected` | `#E2F3EB` | `#007549` | `Brand/primary/100` |
| `surface.list.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.list.hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.list.selected` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.list.selected-hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.timeslot.default` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.timeslot.hover` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.timeslot.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `surface.timeslot.disabled` | `#FAFCFA` | `#102310` | `Brand/secondary/50` |
| `surface.timeslot.box` | `#FAFCFA` | `#102310` | `Brand/secondary/50` |
| `surface.accordion.collapsed` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.accordion.expanded` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.accordion.disabled` | `#F4F5F5` | `#363B3F` | `Primitive/color/neutral/100` |
| `surface.alert.info` | `#F0F6FF` | `#172554` | `Primitive/color/blue/50` |
| `surface.alert.success` | `#F4FBFA` | `#082B2A` | `Primitive/color/teal/50` |
| `surface.alert.warning` | `#FFF9F0` | `#431407` | `Primitive/color/orange/50` |
| `surface.alert.error` | `#FFF0F0` | `#450A0A` | `Primitive/color/red/50` |
| `surface.smallAlert.success` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `surface.smallAlert.error` | `#EF4444` | `#EF4444` | `Primitive/color/red/500` |
| `surface.alertDialog.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.calendar.date-default` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.calendar.date-current` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.calendar.date-hover` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.calendar.date-selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `surface.calendar.date-disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.calendar slot.date-default` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.calendar slot.date-current` | `#08A768` | `#6AD6A5` | `Brand/primary/600` |
| `surface.calendar slot.date-hover` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.calendar slot.date-selected` | `#007549` | `#08A768` | `Brand/primary/700` |
| `surface.calendar slot.date-disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.calendar slot.slot-default` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.calendar slot.slot-hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.calendar slot.slot-selected` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `surface.calendar slot.slot-disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.calendar slot.cta-default` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.calendar slot.cta-hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.calendar slot.cta-disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.sideMenu.default` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.sideMenu.hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.sideMenu.disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.sideMenu.selected` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.sideMenu.selected-hover` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.topic.default` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.topic.hover` | `#F4FBF8` | `#003322` | `Brand/primary/50` |
| `surface.topic.disabled` | `#F9FBFB` | `#363B3F` | `Primitive/color/neutral/50` |
| `surface.tabPill.default` | `#FFFFFF` | `#363B3F` | `Primitive/color/grey/50` |
| `surface.tabPill.hover` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `surface.tabPill.selected` | `#FFFFFF` | `#363B3F` | `Primitive/color/grey/50` |
| `surface.tabPill.disabled` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.tabPill.background` | `#E2F3EB` | `#003322` | `Primitive/color/emerald/100` |
| `surface.tabCapsule.default` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.tabCapsule.hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.tabCapsule.selected` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.tabCapsule.disabled` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.tabCapsule.success` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.statusSummaryCard.Info` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `surface.statusSummaryCard.Indigo` | `#EEF2FF` | `#3730A3` | `Primitive/color/indigo/100` |
| `surface.statusSummaryCard.Sakura` | `#FFEDF3` | `#A35C7C` | `Primitive/color/sakura/100` |
| `surface.statusSummaryCard.Positive` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `surface.statusSummaryCard.Lime` | `#EAF7CC` | `#3B4E14` | `Primitive/color/lime/100` |
| `surface.statusSummaryCard.Inprogress` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `surface.statusSummaryCard.Destructive` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.statusSummaryCard.increase` | `#D1F6EE` | `#003322` | `Primitive/color/teal/100` |
| `surface.statusSummaryCard.decrease` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.categorySummaryCard.A` | `#BFDBFE` | `#1E40AF` | `Primitive/color/blue/200` |
| `surface.categorySummaryCard.B` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `surface.categorySummaryCard.C` | `#A2EDDE` | `#195652` | `Primitive/color/teal/200` |
| `surface.categorySummaryCard.D` | `#F3ADFD` | `#6E1379` | `Primitive/color/magenta/200` |
| `surface.categorySummaryCard.E` | `#E0E7FF` | `#4338CA` | `Primitive/color/indigo/200` |
| `surface.categorySummaryCard.F` | `#FECACA` | `#991B1B` | `Primitive/color/red/200` |
| `surface.categorySummaryCard.G` | `#FEF08A` | `#854D0E` | `Primitive/color/yellow/200` |
| `surface.categorySummaryCard.H` | `#FED7AA` | `#9A3412` | `Primitive/color/orange/200` |
| `surface.formBuilder.default` | `#F4FBF8` | `#E8E8E8` | `Brand/primary/50` |
| `surface.formBuilder.default-on` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.formBuilder.hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.formBuilder.filled` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.formBuilder.error` | `#FFF0F0` | `#450A0A` | `Primitive/color/red/50` |
| `surface.formBuilder.card` | `#FFFFFF` | `#111314` | `Primitive/color/grey/50` |
| `surface.scrollBar.neutral-bar` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `surface.scrollBar.neutral-pill` | `#ADB2B7` | `#ADB2B7` | `Primitive/color/neutral/400` |
| `surface.scrollBar.info-bar` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `surface.scrollBar.info-pill` | `#93C5FD` | `#1D4ED8` | `Primitive/color/blue/300` |
| `surface.scrollBar.success-bar` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `surface.scrollBar.success-pill` | `#6CDCCA` | `#196C65` | `Primitive/color/teal/300` |
| `surface.scrollBar.warning-bar` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `surface.scrollBar.warning-pill` | `#FDBA74` | `#C2410C` | `Primitive/color/orange/300` |
| `surface.scrollBar.error-bar` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.scrollBar.error-pill` | `#FCA5A5` | `#B91C1C` | `Primitive/color/red/300` |
| `surface.indicator.grey` | `#636B72` | `#ADB2B7` | `Primitive/color/neutral/600` |
| `surface.indicator.blue` | `#2563EB` | `#60A5FA` | `Primitive/color/blue/600` |
| `surface.indicator.indigo` | `#6366F1` | `#A5B4FC` | `Primitive/color/indigo/600` |
| `surface.indicator.teal` | `#1B867D` | `#3EC3B2` | `Primitive/color/teal/600` |
| `surface.indicator.green` | `#08A768` | `#6AD6A5` | `Primitive/color/emerald/600` |
| `surface.indicator.orange` | `#EA580C` | `#FB923C` | `Primitive/color/orange/600` |
| `surface.indicator.yellow` | `#CA8A04` | `#FACC15` | `Primitive/color/yellow/600` |
| `surface.indicator.red` | `#DC2626` | `#F87171` | `Primitive/color/red/600` |
| `surface.indicator.pink` | `#DB2777` | `#F472B6` | `Primitive/color/pink/600` |
| `surface.indicator.purple` | `#9333EA` | `#C084FC` | `Primitive/color/purple/600` |
| `surface.indicator.magenta` | `#A81BBD` | `#DC4CF2` | `Primitive/color/magenta/600` |
| `surface.steps.default` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.steps.hover` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `surface.steps.selected` | `#007549` | `#91DFBB` | `Brand/primary/700` |
| `surface.steps.success` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `surface.steps.success-hover` | `#1B867D` | `#3EC3B2` | `Primitive/color/teal/600` |
| `surface.table.header` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.table.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.table.hover` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.table.selected` | `#F4FBF8` | `#002318` | `Brand/primary/50` |
| `surface.table.selected-hover` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.table.footer` | `#E2F3EB` | `#003322` | `Brand/primary/100` |
| `surface.dropdown.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.actionList.default` | `#FFFFFF` | `#222629` | `Primitive/color/grey/50` |
| `surface.progress Bar.neutral-bar` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `surface.progress Bar.neutral-pill` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `surface.progress Bar.blue-bar` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `surface.progress Bar.blue-pill` | `#3B82F6` | `#3B82F6` | `Primitive/color/blue/500` |
| `surface.progress Bar.indigo-bar` | `#EEF2FF` | `#3730A3` | `Primitive/color/indigo/100` |
| `surface.progress Bar.indigo-pill` | `#818CF8` | `#818CF8` | `Primitive/color/indigo/500` |
| `surface.progress Bar.lavender-bar` | `#EDE7FF` | `#4E3490` | `Primitive/color/lavender/100` |
| `surface.progress Bar.lavender-pill` | `#C8B6FF` | `#C8B6FF` | `Primitive/color/lavender/500` |
| `surface.progress Bar.teal-bar` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `surface.progress Bar.teal-pill` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `surface.progress Bar.lime-bar` | `#EAF7CC` | `#3B4E14` | `Primitive/color/lime/100` |
| `surface.progress Bar.lime-pill` | `#A5CF4F` | `#A5CF4F` | `Primitive/color/lime/500` |
| `surface.progress Bar.sakura-bar` | `#FFEDF3` | `#A35C7C` | `Primitive/color/sakura/100` |
| `surface.progress Bar.sakura-pill` | `#F5AFCA` | `#F5AFCA` | `Primitive/color/sakura/500` |
| `surface.progress Bar.orange-bar` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `surface.progress Bar.orange-pill` | `#F97316` | `#F97316` | `Primitive/color/orange/500` |
| `surface.progress Bar.red-bar` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.progress Bar.red-pill` | `#EF4444` | `#EF4444` | `Primitive/color/red/500` |
| `surface.slider.primary-bar` | `#C5EDDA` | `#004C31` | `Brand/primary/200` |
| `surface.slider.primary-pill` | `#54CF97` | `#54CF97` | `Brand/primary/500` |
| `surface.slider.neutral-bar` | `#E2E4E6` | `#363B3F` | `Primitive/color/neutral/200` |
| `surface.slider.neutral-pill` | `#858C92` | `#858C92` | `Primitive/color/neutral/500` |
| `surface.slider.blue-bar` | `#DBEAFE` | `#1E3A8A` | `Primitive/color/blue/100` |
| `surface.slider.blue-pill` | `#3B82F6` | `#3B82F6` | `Primitive/color/blue/500` |
| `surface.slider.indigo-bar` | `#EEF2FF` | `#3730A3` | `Primitive/color/indigo/100` |
| `surface.slider.indigo-pill` | `#818CF8` | `#818CF8` | `Primitive/color/indigo/500` |
| `surface.slider.lavender-bar` | `#EDE7FF` | `#4E3490` | `Primitive/color/lavender/100` |
| `surface.slider.lavender-pill` | `#C8B6FF` | `#C8B6FF` | `Primitive/color/lavender/500` |
| `surface.slider.teal-bar` | `#D1F6EE` | `#194845` | `Primitive/color/teal/100` |
| `surface.slider.teal-pill` | `#24A899` | `#24A899` | `Primitive/color/teal/500` |
| `surface.slider.lime-bar` | `#EAF7CC` | `#3B4E14` | `Primitive/color/lime/100` |
| `surface.slider.lime-pill` | `#A5CF4F` | `#A5CF4F` | `Primitive/color/lime/500` |
| `surface.slider.sakura-bar` | `#FFEDF3` | `#A35C7C` | `Primitive/color/sakura/100` |
| `surface.slider.sakura-pill` | `#F5AFCA` | `#F5AFCA` | `Primitive/color/sakura/500` |
| `surface.slider.orange-bar` | `#FFEDD5` | `#7C2D12` | `Primitive/color/orange/100` |
| `surface.slider.orange-pill` | `#F97316` | `#F97316` | `Primitive/color/orange/500` |
| `surface.slider.red-bar` | `#FEE2E2` | `#7F1D1D` | `Primitive/color/red/100` |
| `surface.slider.red-pill` | `#EF4444` | `#EF4444` | `Primitive/color/red/500` |

### 4.6. `typography`

#### Text Styles — overview

**45 text styles** · **3 font families** · **8 groups** (per Figma `3465:2`, `3467:2`, plus Display Graphic 2).

> 🔗 **Primitives:** the px / weight / letter-spacing values referenced by these styles live in [§1b. Typography](#1b-typography) (font-size scale `xxs`–`6xl`, weight `thin`–`bold`, letter-spacing `tighter`/`normal`).

##### Font families

| Family | Role | Styles | Weights | Sample |
|---|---|---|---|---|
| **Sao Chingcha** | Display graphic 1 — Thai-led hero, brand moments | 9 | Bold only | `Aa Bb Cc · กขคงจ · 0123456789` |
| **Mitr**         | Display graphic 2 + Heading (h1–h8) — brand display family driving both hero + section headings | 17 | Regular (heading) · Bold (graphic2) — broader scale available 200–900 | `Aa Bb Cc · กขคงจ · 0123456789` |
| **Sarabun**      | UI typography — display, body, labels, captions, buttons (no longer used for headings) | 19 | Regular · Medium · Bold | `Aa Bb Cc · กขคงจ · 0123456789` |

> Sao Chingcha + Mitr are the two **brand display** typefaces. Mitr now also drives **headings h1–h8** (Regular weight) — gives the product a softer, warmer brand voice than Sarabun Bold did previously. Sarabun remains the workhorse for body / display / labels / captions / buttons.

##### Groups

| Group | Count | Family | Weight | Size range | Common use |
|---|---|---|---|---|---|
| Display Graphic   | 9 | Sao Chingcha | Bold | 60 → 14 px | Hero, marketing splash, brand moments. LH 125%, LS 0. |
| Display Graphic 2 | 9 | Mitr | Bold (default) · Medium / SemiBold available | 60 → 14 px | Alternative display headings, posters, kiosk hero blocks where Mitr's wider Latin glyphs read better. LH 125%, LS 0. |
| Display / Heading | 9 | Sarabun | Bold | 60 → 14 px | Page-level display headlines. `display1` LS 0; `display2`–`display9` LS −2% (`tighter`). |
| Heading | 8 | Mitr | Regular | 36 → 12 px | Section headings. h1–h4 LH **137.5%**; h5–h8 LH **150%**. Brand-driven heading family (was Sarabun Bold; now Mitr Regular for warmer brand voice). |
| Body | 4 | Sarabun | Regular | 20 → 14 px | Body text, paragraphs, descriptions. LH 150%. |
| Label | 2 | Sarabun | Medium | 14, 12 px | Form labels, nav items, tab labels, tags, chips, overlines. |
| Caption | 2 | Sarabun | Regular | 12, 10 px | Supporting text, timestamps, metadata, legal copy, hints. |
| Button | 2 | Sarabun | Medium | 16, 14 px | `button.lg` — CTA / primary. `button.sm` — secondary / compact. |

#### Full token reference

Every style below maps to a Figma text style of the same name. The right-hand "Alias:" columns expose the primitive-token paths each style composes from (so retheming swaps one place, all styles follow).

| Token | Font Family | Weight | Size | Line Height | Letter Spacing | Alias: Family | Alias: Weight | Alias: Size | Alias: Tracking |
|-------|-------------|--------|------|-------------|----------------|---------------|---------------|-------------|-----------------|
| `typography.display.graphic1` | Sao Chingcha | Bold | 60px | 125% | 0% | `typography.family.heading-graphic` | `typography.weight.bold` | `typography.size.6xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic2` | Sao Chingcha | Bold | 48px | 125% | 0% | `typography.family.heading-graphic` | `typography.weight.bold` | `typography.size.5xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic3` | Sao Chingcha | Bold | 36px | 125% | 0% | `typography.family.heading-graphic` | `typography.weight.bold` | `typography.size.4xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic4` | Sao Chingcha | Bold | 30px | 125% | 0% | `typography.family.heading-graphic` | `typography.weight.bold` | `typography.size.3xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic5` | Sao Chingcha | Bold | 24px | 125% | 0% | `typography.family.heading-graphic` | `typography.weight.bold` | `typography.size.2xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic6` | Sao Chingcha | Bold | 20px | 125% | 0% | `typography.family.heading-graphic` | `typography.weight.bold` | `typography.size.xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic7` | Sao Chingcha | Bold | 18px | 125% | 0% | `typography.family.heading-graphic` | `typography.weight.bold` | `typography.size.lg` | `typography.letter-spacing.normal` |
| `typography.display.graphic8` | Sao Chingcha | Bold | 16px | 125% | 0% | `typography.family.heading-graphic` | `typography.weight.bold` | `typography.size.base` | `typography.letter-spacing.normal` |
| `typography.display.graphic9` | Sao Chingcha | Bold | 14px | 125% | 0% | `typography.family.heading-graphic` | `typography.weight.bold` | `typography.size.sm` | `typography.letter-spacing.normal` |
| `typography.display.graphic2-1` | Mitr | Bold | 60px | 125% | 0% | `typography.family.heading-graphic2` | `typography.weight.bold` | `typography.size.6xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic2-2` | Mitr | Bold | 48px | 125% | 0% | `typography.family.heading-graphic2` | `typography.weight.bold` | `typography.size.5xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic2-3` | Mitr | Bold | 36px | 125% | 0% | `typography.family.heading-graphic2` | `typography.weight.bold` | `typography.size.4xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic2-4` | Mitr | Bold | 30px | 125% | 0% | `typography.family.heading-graphic2` | `typography.weight.bold` | `typography.size.3xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic2-5` | Mitr | Bold | 24px | 125% | 0% | `typography.family.heading-graphic2` | `typography.weight.bold` | `typography.size.2xl` | `typography.letter-spacing.normal` |
| `typography.display.graphic2-6` | Mitr | Bold | 20px | 125% | 0% | `typography.family.heading-graphic2` | `typography.weight.bold` | `typography.size.xl`  | `typography.letter-spacing.normal` |
| `typography.display.graphic2-7` | Mitr | Bold | 18px | 125% | 0% | `typography.family.heading-graphic2` | `typography.weight.bold` | `typography.size.lg`  | `typography.letter-spacing.normal` |
| `typography.display.graphic2-8` | Mitr | Bold | 16px | 125% | 0% | `typography.family.heading-graphic2` | `typography.weight.bold` | `typography.size.base`| `typography.letter-spacing.normal` |
| `typography.display.graphic2-9` | Mitr | Bold | 14px | 125% | 0% | `typography.family.heading-graphic2` | `typography.weight.bold` | `typography.size.sm`  | `typography.letter-spacing.normal` |
| `typography.display1` | Sarabun | Bold | 60px | 125% | 0% | `typography.family.body` | `typography.weight.bold` | `typography.size.6xl` | `typography.letter-spacing.normal` |
| `typography.display2` | Sarabun | Bold | 48px | 125% | -2% | `typography.family.body` | `typography.weight.bold` | `typography.size.5xl` | `typography.letter-spacing.tighter` |
| `typography.display3` | Sarabun | Bold | 36px | 125% | -2% | `typography.family.body` | `typography.weight.bold` | `typography.size.4xl` | `typography.letter-spacing.tighter` |
| `typography.display4` | Sarabun | Bold | 30px | 125% | -2% | `typography.family.body` | `typography.weight.bold` | `typography.size.3xl` | `typography.letter-spacing.tighter` |
| `typography.display5` | Sarabun | Bold | 24px | 125% | -2% | `typography.family.body` | `typography.weight.bold` | `typography.size.2xl` | `typography.letter-spacing.tighter` |
| `typography.display6` | Sarabun | Bold | 20px | 125% | -2% | `typography.family.body` | `typography.weight.bold` | `typography.size.xl` | `typography.letter-spacing.tighter` |
| `typography.display7` | Sarabun | Bold | 18px | 125% | -2% | `typography.family.body` | `typography.weight.bold` | `typography.size.lg` | `typography.letter-spacing.tighter` |
| `typography.display8` | Sarabun | Bold | 16px | 125% | -2% | `typography.family.body` | `typography.weight.bold` | `typography.size.base` | `typography.letter-spacing.tighter` |
| `typography.display9` | Sarabun | Bold | 14px | 125% | -2% | `typography.family.body` | `typography.weight.bold` | `typography.size.sm` | `typography.letter-spacing.tighter` |
| `typography.h1` | Mitr | Regular | 36px | 137.5% | 0% | `typography.family.heading-graphic2` | `typography.weight.regular` | `typography.size.4xl`  | `typography.letter-spacing.normal` |
| `typography.h2` | Mitr | Regular | 30px | 137.5% | 0% | `typography.family.heading-graphic2` | `typography.weight.regular` | `typography.size.3xl`  | `typography.letter-spacing.normal` |
| `typography.h3` | Mitr | Regular | 24px | 137.5% | 0% | `typography.family.heading-graphic2` | `typography.weight.regular` | `typography.size.2xl`  | `typography.letter-spacing.normal` |
| `typography.h4` | Mitr | Regular | 20px | 137.5% | 0% | `typography.family.heading-graphic2` | `typography.weight.regular` | `typography.size.xl`   | `typography.letter-spacing.normal` |
| `typography.h5` | Mitr | Regular | 18px | 150%   | 0% | `typography.family.heading-graphic2` | `typography.weight.regular` | `typography.size.lg`   | `typography.letter-spacing.normal` |
| `typography.h5-emphasis` | Mitr | Medium | 18px | 150% | 0% | `typography.family.heading-graphic2` | `typography.weight.medium` | `typography.size.lg` | `typography.letter-spacing.normal` |
| `typography.h6` | Mitr | Regular | 16px | 150%   | 0% | `typography.family.heading-graphic2` | `typography.weight.regular` | `typography.size.base` | `typography.letter-spacing.normal` |
| `typography.h7` | Mitr | Regular | 14px | 150%   | 0% | `typography.family.heading-graphic2` | `typography.weight.regular` | `typography.size.sm`   | `typography.letter-spacing.normal` |
| `typography.h8` | Mitr | Regular | 12px | 150%   | 0% | `typography.family.heading-graphic2` | `typography.weight.regular` | `typography.size.xs`   | `typography.letter-spacing.normal` |
| `typography.body1` | Sarabun | Regular | 20px | 150% | 0% | `typography.family.body` | `typography.weight.regular` | `typography.size.xl` | `typography.letter-spacing.normal` |
| `typography.body2` | Sarabun | Regular | 18px | 150% | 0% | `typography.family.body` | `typography.weight.regular` | `typography.size.lg` | `typography.letter-spacing.normal` |
| `typography.body3` | Sarabun | Regular | 16px | 150% | 0% | `typography.family.body` | `typography.weight.regular` | `typography.size.base` | `typography.letter-spacing.normal` |
| `typography.body4` | Sarabun | Regular | 14px | 150% | 0% | `typography.family.body` | `typography.weight.regular` | `typography.size.sm` | `typography.letter-spacing.normal` |
| `typography.label1` | Sarabun | Medium | 14px | 150% | 0% | `typography.family.body` | `typography.weight.medium` | `typography.size.sm` | `typography.letter-spacing.normal` |
| `typography.label2` | Sarabun | Medium | 12px | 150% | 0% | `typography.family.body` | `typography.weight.medium` | `typography.size.xs` | `typography.letter-spacing.normal` |
| `typography.caption1` | Sarabun | Regular | 12px | 150% | 0px | `typography.family.body` | `typography.weight.regular` | `typography.size.xs` | `typography.letter-spacing.normal` |
| `typography.caption2` | Sarabun | Regular | 10px | 150% | 0px | `typography.family.body` | `typography.weight.regular` | `typography.size.xxs` | `typography.letter-spacing.normal` |
| `typography.button.lg` | Sarabun | Medium | 16px | 150% | 0% | `typography.family.body` | `typography.weight.medium` | `typography.size.base` | `typography.letter-spacing.normal` |
| `typography.button.sm` | Sarabun | Medium | 14px | 150% | 0% | `typography.family.body` | `typography.weight.medium` | `typography.size.sm` | `typography.letter-spacing.normal` |

---

## 5. Component Token Mapping

Component Token Mapping แสดงว่าแต่ละ property ของ component ใช้ Semantic token ใด แบ่งตาม variant, state และ size

> Source: Figma Button section (node `319:2012`) cross-referenced against Primitive/Semantic token layers.
> Brand mode: **default** (primary = emerald palette, secondary = forest palette).
> All hex values shown for **light / dark** modes respectively.

> 📘 **Usage Guidelines (Do / Don't):** Per-component Do/Don't rules — behavior · dos · donts — live in the live components page, **not** in this token document. The token tables below specify *what* each variant looks like; the Guidelines specify *when* and *why* to use each variant. To browse Do/Don't for any component:
>
> - **Live viewer:** [docs/design-system.html](docs/design-system.html) → pick a component in the sidebar → **Live** tab. Each component renders its `<Guidelines>` panel at the bottom of its Section.
> - **Canonical source code:** [src/pages/components/ComponentsLibraryPage.jsx](src/pages/components/ComponentsLibraryPage.jsx). Search for the component's `<Guidelines behavior={[...]} dos={[...]} donts={[...]}/>` block.
>
> All 118 routed components carry a `<Guidelines>` panel as of 2026-05-23. To update, edit `ComponentsLibraryPage.jsx` only — design.md token mappings stay focused on the token layer.

---

---

### Variants Matrix

The Figma file contains **14 button families**, each with Style × State × Size combinations.

| Button Family | Styles | States | Sizes | Figma Node |
|---|---|---|---|---|
| Brand Button | Fill, Outline, Ghost | Default, Hover, Disable, Selected | Small (36px h), Medium (40px h), Large (44px h) | `377-11596` |
| Brand Subdue Button | Fill, Outline | Default, Hover, Disable, Selected | Small, Medium, Large | `2024-3589` |
| Danger Button | Fill, Outline, Ghost | Default, Hover, Disable, Selected | Small, Medium, Large | `2024-3590` |
| Danger Subdue Button | Fill, Outline | Default, Hover, Disable, Selected | Small, Medium, Large | `2024-3592` |
| Neutral Button | Fill, Outline, Ghost | Default, Hover, Disable, Selected | Small, Medium, Large | `2024-3591` |
| Neutral Subdue Button | Fill, Outline | Default, Hover, Disable, Selected | Small, Medium, Large | `2024-2940` |
| Brand Icon Button | Fill, Outline, Ghost | Default, Hover, Disabled, Selected | Small (36px), Medium (40px), Large (44px) | `377-11596` |
| Brand Subdue Icon Button | Fill, Outline | Default, Hover, Disabled, Selected | Small, Medium, Large | `2024-4352` |
| Danger Icon Button | Fill, Outline, Ghost | Default, Hover, Disabled, Selected | Small, Medium, Large | `2024-3701` |
| Danger Subdue Icon Button | Fill, Outline | Default, Hover, Disabled, Selected | Small, Medium, Large | `2024-4434` |
| Neutral Icon Button | Fill, Outline, Ghost | Default, Hover, Disabled, Selected | Small, Medium, Large | `2024-4189` |
| Neutral Subdue Icon Button | Fill, Outline | Default, Hover, Disabled, Selected | Small, Medium, Large | `2024-4516` |
| Brand Circle Icon Button | Fill, Outline, Ghost | Default, Hover, Selected, Focus, Disabled | Small, Medium, Large | `2103-548` |
| Brand Circle Subdue Icon Button | Fill, Outline | Default, Hover, Active, Focus, Disabled | XSmall (32px), Small, Medium, Large | `2103-645` |

> **Note:** "Subdue" variants use lower-contrast background colors (tertiary/quaternary surface levels) compared to their standard counterparts.
> "Selected" is an additional interactive state meaning the button is toggled/active.

---

### Token Mapping by Variant

#### Key

- **Surface** = button container background
- **Text** = label color
- **Icon** = icon color (leading/trailing icon, or icon-only)
- **Border** = stroke color (Outline style only)
- The `surface.*` tokens are the canonical container tokens; `background.*` tokens exist for contextual usage.

---

### 5.1. Brand Button (Primary)

> 🔗 [ดู Button Preview](./button_preview.html)

#### Component Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `style` | `Fill` \| `Outline` \| `Ghost` | `Fill` | Visual style variant |
| `size` | `Large` \| `Medium` \| `Small` | `Large` | Button size (44px / 40px / 36px height) |
| `state` | `Default` \| `Hover` \| `Disable` \| `Selected` | `Default` | Interaction state |
| `showLeadingIcon` | `boolean` | `true` | แสดง icon **ด้านหน้า** (ซ้ายของ label) |
| `showTailingIcon` | `boolean` | `true` | แสดง icon **ด้านหลัง** (ขวาของ label) |

#### Spacing (verified from Figma node 377:12081)

| Size | Height | Padding X | Padding Y | Gap | Icon Size | Min Width | Max Width |
|---|---|---|---|---|---|---|---|
| Large | 44px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Medium | 40px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Small | 36px | `dimension/space/300` (12px) | `dimension/space/150` (6px) | `dimension/space/150` (6px) | 16 × 16px | 80px | 384px |

#### Icon Placement

| Size | Icon Size | Gap | Layout |
|---|---|---|---|
| Large (44px) | 20 × 20px | `dimension/space/200` (8px) | `[leading icon] [label] [tailing icon]` |
| Medium (40px) | 20 × 20px | `dimension/space/200` (8px) | `[leading icon] [label] [tailing icon]` |
| Small (36px) | 16 × 16px | `dimension/space/150` (6px) | `[leading icon] [label] [tailing icon]` |

> **Note:** ทั้ง `showLeadingIcon` และ `showTailingIcon` สามารถ toggle แยกกันได้ — สามารถแสดง icon ด้านเดียว, ทั้งสองด้าน, หรือไม่แสดงเลยก็ได้
>
> **Icon alignment:** icon container ต้อง `display: inline-flex; align-items: center; justify-content: center` เพื่อให้ icon align center ในแนว vertical กับ label เสมอ
>
> **Icon ใน Figma (Figma node 377:12082 / I377:12082;367:9926):** ใช้ Feather `image` icon เป็น placeholder slot — ทั้ง leading และ trailing ใช้ icon เดียวกัน Developer สามารถเปลี่ยนเป็น icon ใด ๆ จาก icon library ได้ตามต้องการ

#### Icon Implementation

```html
<!-- Icon slot — Feather image icon (ตาม Figma node 377:12082) -->
<!-- Leading icon (ซ้าย) -->
<span style="display:inline-flex;align-items:center;justify-content:center;flex-shrink:0;width:20px;height:20px;">
  <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor"
       stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
    <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
    <circle cx="8.5" cy="8.5" r="1.5"></circle>
    <polyline points="21 15 16 10 5 21"></polyline>
  </svg>
</span>

<!-- Label -->
<span>Button</span>

<!-- Trailing icon (ขวา) -->
<span style="display:inline-flex;align-items:center;justify-content:center;flex-shrink:0;width:20px;height:20px;">
  <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor"
       stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
    <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
    <circle cx="8.5" cy="8.5" r="1.5"></circle>
    <polyline points="21 15 16 10 5 21"></polyline>
  </svg>
</span>
```

> **Small size:** เปลี่ยน `width`/`height` ของ wrapper และ `<svg>` เป็น `16px`

---

#### 1a. Fill Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Label | Color | `text.brandPrimaryButton.on-brand` | `Primitive/color/grey/50` | `#FFFFFF` | `#002318` |
| Leading Icon | Color | `icon.brandPrimaryButton.on-brand` | `Primitive/color/grey/50` | `#FFFFFF` | `#020617` |
| Trailing Icon | Color | `icon.brandPrimaryButton.on-brand` | `Primitive/color/grey/50` | `#FFFFFF` | `#020617` |
| Border | — | *(no border on Fill style)* | — | — | — |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.default-hover` | `Brand/primary/800` | `#004C31` | `#C5EDDA` |
| Label | Color | `text.brandPrimaryButton.on-brand` | `Primitive/color/grey/50` | `#FFFFFF` | `#002318` |
| Leading Icon | Color | `icon.brandPrimaryButton.on-brand` | `Primitive/color/grey/50` | `#FFFFFF` | `#020617` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Label | Color | `text.disabledButton.on-disabled` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |
| Leading Icon | Color | `icon.disablediconButton.on-disabled` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.secondary` | `Brand/primary/200` | `#C5EDDA` | `#004C31` |
| Label | Color | `text.brandPrimaryButton.secondary` | `Brand/primary/600` | `#08A768` | `#6AD6A5` |
| Leading Icon | Color | `icon.brandPrimaryButton.secondary` | `Brand/primary/600` | `#08A768` | `#6AD6A5` |

---

#### 1b. Outline Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.quinary` | `Primitive/color/grey/50` | `#FFFFFF` | `#020617` |
| Label | Color | `text.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Leading Icon | Color | `icon.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Border | Stroke | `border.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.quinary-hover` | `Brand/primary/50` | `#F4FBF8` | `#002318` |
| Label | Color | `text.brandPrimaryButton.secondary` | `Brand/primary/600` | `#08A768` | `#6AD6A5` |
| Leading Icon | Color | `icon.brandPrimaryButton.default-hover` | `Brand/primary/800` | `#004C31` | `#C5EDDA` |
| Border | Stroke | `border.brandPrimaryButton.default-hover` | `Brand/primary/800` | `#004C31` | `#C5EDDA` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.quinary` | `Primitive/color/grey/50` | `#FFFFFF` | `#DADADA` |
| Label | Color | `text.disabledButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.disablediconButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Border | Stroke | `border.disabledButton.default` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.tertiary` | `Brand/primary/100` | `#E2F3EB` | `#003322` |
| Label | Color | `text.brandPrimaryButton.tertiary` | `Brand/primary/600` | `#08A768` | `#6AD6A5` |
| Leading Icon | Color | `icon.brandPrimaryButton.tertiary` | `Brand/primary/400` | `#6AD6A5` | `#08A768` |
| Border | Stroke | `border.brandPrimaryButton.tertiary` | `Brand/primary/200` | `#C5EDDA` | `#004C31` |

---

#### 1c. Ghost Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.quaternary` | `Brand/primary/50` | `#F4FBF8` | `#002318` |
| Label | Color | `text.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Leading Icon | Color | `icon.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Border | — | *(no border on Ghost style)* | — | — | — |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.quaternary-hover` | `Brand/primary/100` | `#E2F3EB` | `#003322` |
| Label | Color | `text.brandPrimaryButton.secondary` | `Brand/primary/600` | `#08A768` | `#6AD6A5` |
| Leading Icon | Color | `icon.brandPrimaryButton.secondary-hover` | `Brand/primary/700` | `#007549` | `#84DBB4` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#363B3F` |
| Label | Color | `text.disabledButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.disablediconButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.tertiary` | `Brand/primary/100` | `#E2F3EB` | `#003322` |
| Label | Color | `text.brandPrimaryButton.tertiary` | `Brand/primary/600` | `#08A768` | `#6AD6A5` |
| Leading Icon | Color | `icon.brandPrimaryButton.tertiary` | `Brand/primary/400` | `#6AD6A5` | `#08A768` |

---

### 5.2. Brand Subdue Button

> 🔗 [ดู Button Preview](./button_preview.html)

Subdue buttons use lower-contrast surface levels (tertiary/quaternary tiers), ideal for secondary actions where the primary button would be too prominent. Source: Figma node `2024-3589`.

#### Component Properties

| Property | Type | Default | Description |
|---|---|---|---|
| `style` | `Fill` \| `Outline` | `Fill` | Visual style variant (ไม่มี Ghost) |
| `size` | `Large` \| `Medium` \| `Small` | `Large` | Button size (44px / 40px / 36px height) |
| `state` | `Default` \| `Hover` \| `Disable` \| `Selected` | `Default` | Interaction state |
| `showLeadingIcon` | `boolean` | `true` | แสดง icon **ด้านหน้า** (ซ้ายของ label) |
| `showTailingIcon` | `boolean` | `true` | แสดง icon **ด้านหลัง** (ขวาของ label) |

#### Spacing (verified from Figma node 2024:3589)

| Size | Height | Padding X | Padding Y | Gap | Icon Size | Min Width | Max Width |
|---|---|---|---|---|---|---|---|
| Large | 44px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Medium | 40px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Small | 36px | `dimension/space/300` (12px) | `dimension/space/150` (6px) | `dimension/space/150` (6px) | 16 × 16px | 80px | 384px |

#### Font

| Size | Token | Value |
|---|---|---|
| Large / Medium | `typograpphy/size/base` | 16px, Sarabun Medium |
| Small | `typograpphy/size/sm` | 14px, Sarabun Medium |

#### Icon

> **Icon (Figma node I2024:2428;367:9926):** ใช้ Feather `image` icon เป็น placeholder slot — ทั้ง leading และ trailing ใช้ icon เดียวกัน
>
> **Icon alignment:** icon container ต้อง `display: inline-flex; align-items: center; justify-content: center` เพื่อให้ icon align center ในแนว vertical กับ label เสมอ

```html
<!-- Leading icon — Large/Medium (20px) -->
<span style="display:inline-flex;align-items:center;justify-content:center;flex-shrink:0;width:20px;height:20px;">
  <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor"
       stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
    <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
    <circle cx="8.5" cy="8.5" r="1.5"></circle>
    <polyline points="21 15 16 10 5 21"></polyline>
  </svg>
</span>
<!-- Small size: เปลี่ยน width/height wrapper และ svg เป็น 16px -->
```

---

#### 2a. Fill Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.tertiary` | `Brand/primary/100` | `#E2F3EB` | `#003322` |
| Label | Color | `text.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Leading Icon | Color | `icon.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.tertiary-hover` | `Brand/primary/200` | `#C5EDDA` | `#004C31` |
| Label | Color | `text.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Leading Icon | Color | `icon.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#363B3F` |
| Label | Color | `text.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |
| Leading Icon | Color | `icon.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.quaternary` | `Brand/primary/50` | `#F4FBF8` | `#002318` |
| Container | Border | `border.brandPrimary.quaternary` | `Brand/primary/100` | `#E2F3EB` | `#003322` |
| Label | Color | `text.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Leading Icon | Color | `icon.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |

---

#### 2b. Outline Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | *(transparent — no fill)* | — | — | — |
| Label | Color | `text.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Leading Icon | Color | `icon.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Border | Stroke | `border.brandPrimaryButton.tertiary` | `Brand/primary/200` | `#C5EDDA` | `#004C31` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.quaternary` | `Brand/primary/50` | `#F4FBF8` | `#002318` |
| Label | Color | `text.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Leading Icon | Color | `icon.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Border | Stroke | `border.brandPrimaryButton.quaternary-hover` | `Brand/primary/200` | `#C5EDDA` | `#004C31` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#363B3F` |
| Label | Color | `text.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |
| Leading Icon | Color | `icon.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |
| Border | Stroke | `border.disabledButton.tertiary` | `Primitive/color/neutral/200` | `#E2E4E6` | `#363B3F` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.brandPrimaryButton.quaternary` | `Brand/primary/50` | `#F4FBF8` | `#002318` |
| Label | Color | `text.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Leading Icon | Color | `icon.brandPrimaryButton.default` | `Brand/primary/700` | `#007549` | `#84DBB4` |
| Border | Stroke | `border.brandPrimary.quaternary` | `Brand/primary/100` | `#E2F3EB` | `#003322` |

---

### 5.2a. Brand Dashed Add Button

> **Figma node:** [`3360:2711`](https://www.figma.com/design/SBdh4TtY0KAa22s3dnyRNr/MIH-Design-System-Foundation?node-id=3360-2711) · MIH Design System Foundation  
> **Code:** [`src/components/button/BrandDashedAddButton.jsx`](src/components/button/BrandDashedAddButton.jsx) · used by [`HistoryListForm`](src/components/forms/HistoryListForm.jsx) for `schema.addLabel` (e.g. เพิ่มชื่อยา/สารเคมี) and by [`VitalsForm`](src/components/forms/VitalsForm.jsx) for **เพิ่มรอบการวัด** (gated with `disabled` until required vitals are valid).

Full-width **dashed** affordance for adding another row in a repeating history block. Matches Figma **HistoryListFormButton** (56px height tier `dimension/size/1200`).

#### Dimension tokens

| Property | Token | Value |
|---|---|---|
| Height | `dimension/size/1200` | **56 px** |
| Padding X | `dimension/space/400` | **16 px** |
| Padding Y | `dimension/space/200` | **8 px** |
| Icon ↔ label gap | `dimension/space/200` | **8 px** |
| Corner radius | `dimension/radius/200` | **8 px** |
| Border | `dimension/stroke/200` | **2 px** dashed |
| Leading icon | — | **20 × 20 px** (`plus`) |
| Min width | — | **80 px** (content may span full column width) |

#### Color tokens

| State | Layer | Semantic token | CSS variable |
|---|---|---|---|
| Default | Background | `surface.brandPrimary.quaternary` | `--surface-brandPrimary-quaternary` |
| Default | Border | `border.brandPrimaryButton.tertiary` | `--border-brandPrimaryButton-tertiary` |
| Default | Label + icon | `text.brandPrimaryButton.default` · `icon.brandPrimaryButton.default` | `--text-brandPrimaryButton-default` (inherits on `<Icon>` via `currentColor`) |
| Hover | Background | `surface.brandPrimary.quaternary-hover` | `--surface-brandPrimary-quaternary-hover` |
| Hover | Border | `border.brandPrimaryButton.tertiary-hover` | `--border-brandPrimaryButton-tertiary-hover` → `Brand/primary/300` |
| **Disabled** | Background | `surface.disabledButton.tertiary` | `--surface-disabledButton-tertiary` |
| **Disabled** | Border | `border.disabledButton.tertiary` | `--border-disabledButton-tertiary` |
| **Disabled** | Label + icon | `text.disabledButton.tertiary` · `icon.disabledButton.tertiary` | `--text-disabledButton-tertiary` · `--icon-disabledButton-tertiary` |

Disabled matches **Brand Subdue Button · Outline · Disabled** (design.md §7f / row ~2461): neutral fill + neutral stroke + muted label/icon — **not** an opacity wash (`opacity: 0.55` was removed). Cursor remains `not-allowed`; hover does not retint while `disabled`.

#### React props

| Prop | Type | Default | Description |
|---|---|---|---|
| `children` | `node` | — | Button label |
| `onClick` | `function` | — | Click handler |
| `disabled` | `boolean` | `false` | When `true`: disabledButton.tertiary surface/border/text/icon tokens, `cursor: not-allowed`, no hover |
| `icon` | `string` | `'plus'` | Passed to `<Icon name={…}>` |
| `iconSize` | `number` | `20` | Icon pixel size |
| `className` / `style` | — | — | Layout overrides (e.g. width in a flex parent) |

---

### 5.3. Danger Button

> 🔗 [ดู Button Preview](./button_preview.html)

**Component Properties:** style (Fill / Outline / Ghost) · size (Large / Medium / Small) · state (Default / Hover / Disable / Selected) · showLeadingIcon · showTailingIcon · Source: Figma node `2024-3590`

#### Spacing (shared with Brand Button — verified from Figma 377:12081)

| Size | Height | Padding X | Padding Y | Gap | Icon Size | Min Width | Max Width |
|---|---|---|---|---|---|---|---|
| Large | 44px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Medium | 40px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Small | 36px | `dimension/space/300` (12px) | `dimension/space/150` (6px) | `dimension/space/150` (6px) | 16 × 16px | 80px | 384px |

#### Font

| Size | Family | Weight | Size | Line Height |
|---|---|---|---|---|
| Large / Medium | Sarabun | Medium (500) | 16px | 1.5 |
| Small | Sarabun | Medium (500) | 14px | 1.5 |

#### Icon

> **Icon (Figma node 2024-3590):** Feather `image` icon as placeholder slot — ใช้ icon เดียวกันกับ Brand Button ทั้ง leading และ trailing. Danger Button รองรับ Ghost style ด้วย.

```html
<!-- Feather image icon — Danger Button icon slot -->
<span style="display:inline-flex;align-items:center;justify-content:center;flex-shrink:0;width:20px;height:20px">
  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
       stroke-linecap="round" stroke-linejoin="round" width="20" height="20">
    <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
    <circle cx="8.5" cy="8.5" r="1.5"></circle>
    <polyline points="21 15 16 10 5 21"></polyline>
  </svg>
</span>
```

| Placement | Size (Large / Medium) | Size (Small) |
|---|---|---|
| Leading Icon | 20 × 20px | 16 × 16px |
| Trailing Icon | 20 × 20px | 16 × 16px |

---

#### 3a. Fill Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Label | Color | `text.dangerButton.on-default` | `Primitive/color/grey/50` | `#FFFFFF` | `#020617` |
| Leading Icon | Color | `icon.dangerButton.on-default` | `Primitive/color/grey/50` | `#FFFFFF` | `#020617` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.default-hover` | `Primitive/color/red/800` | `#991B1B` | `#FECACA` |
| Label | Color | `text.dangerButton.on-default` | `Primitive/color/grey/50` | `#FFFFFF` | `#020617` |
| Leading Icon | Color | `icon.dangerButton.on-default` | `Primitive/color/grey/50` | `#FFFFFF` | `#020617` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Label | Color | `text.disabledButton.on-disabled` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |
| Leading Icon | Color | `icon.disablediconButton.on-disabled` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.secondary` | `Primitive/color/red/200` | `#FECACA` | `#991B1B` |
| Label | Color | `text.dangerButton.secondary` | `Primitive/color/red/600` | `#DC2626` | `#F87171` |
| Leading Icon | Color | `icon.dangerButton.secondary` | `Primitive/color/red/600` | `#DC2626` | `#F87171` |

---

#### 3b. Outline Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.quinary` | `Primitive/color/grey/50` | `#FFFFFF` | `#DADADA` |
| Label | Color | `text.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Leading Icon | Color | `icon.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Border | Stroke | `border.dangerButton.default` | `Primitive/color/red/600` | `#DC2626` | `#FCA5A5` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.quinary-hover` | `Primitive/color/red/50` | `#FFF0F0` | `#450A0A` |
| Label | Color | `text.dangerButton.secondary` | `Primitive/color/red/600` | `#DC2626` | `#F87171` |
| Leading Icon | Color | `icon.dangerButton.secondary` | `Primitive/color/red/600` | `#DC2626` | `#F87171` |
| Border | Stroke | `border.dangerButton.default-hover` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.quinary` | `Primitive/color/grey/50` | `#FFFFFF` | `#DADADA` |
| Label | Color | `text.disabledButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.disablediconButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Border | Stroke | `border.disabledButton.default` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.tertiary` | `Primitive/color/red/100` | `#FEE2E2` | `#7F1D1D` |
| Label | Color | `text.dangerButton.tertiary` | `Primitive/color/red/400` | `#F87171` | `#DC2626` |
| Leading Icon | Color | `icon.dangerButton.tertiary` | `Primitive/color/red/400` | `#F87171` | `#DC2626` |
| Border | Stroke | `border.dangerButton.tertiary` | `Primitive/color/red/200` | `#FECACA` | `#7F1D1D` |

---

#### 3c. Ghost Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.quaternary` | `Primitive/color/red/50` | `#FFF0F0` | `#450A0A` |
| Label | Color | `text.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Leading Icon | Color | `icon.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.quaternary-hover` | `Primitive/color/red/100` | `#FEE2E2` | `#7F1D1D` |
| Label | Color | `text.dangerButton.secondary` | `Primitive/color/red/600` | `#DC2626` | `#F87171` |
| Leading Icon | Color | `icon.dangerButton.secondary` | `Primitive/color/red/600` | `#DC2626` | `#F87171` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#363B3F` |
| Label | Color | `text.disabledButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.disablediconButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.tertiary` | `Primitive/color/red/100` | `#FEE2E2` | `#7F1D1D` |
| Label | Color | `text.dangerButton.tertiary` | `Primitive/color/red/400` | `#F87171` | `#DC2626` |
| Leading Icon | Color | `icon.dangerButton.tertiary` | `Primitive/color/red/400` | `#F87171` | `#DC2626` |

---

### 5.4. Danger Subdue Button

Subdue danger buttons use softer red surface tiers, for confirmations or secondary destructive actions. Source: Figma node `2024-3592`.

> 🔗 [ดู Button Preview](./button_preview.html)

**Component Properties:** style (Fill / Outline — **ไม่มี Ghost**) · size (Large / Medium / Small) · state (Default / Hover / Disable / Selected) · showLeadingIcon · showTailingIcon

#### Spacing (shared with Brand Button — verified from Figma 377:12081)

| Size | Height | Padding X | Padding Y | Gap | Icon Size | Min Width | Max Width |
|---|---|---|---|---|---|---|---|
| Large | 44px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Medium | 40px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Small | 36px | `dimension/space/300` (12px) | `dimension/space/150` (6px) | `dimension/space/150` (6px) | 16 × 16px | 80px | 384px |

#### Font

| Size | Family | Weight | Size | Line Height |
|---|---|---|---|---|
| Large / Medium | Sarabun | Medium (500) | 16px | 1.5 |
| Small | Sarabun | Medium (500) | 14px | 1.5 |

#### Icon

> **Icon (Figma node 2024-3592):** Feather `image` icon เหมือน Brand Button — ใช้เป็น placeholder slot. ทั้ง leading และ trailing ใช้ icon เดียวกัน. ไม่มี Ghost style สำหรับ Subdue variants.

```html
<!-- Feather image icon — Danger Subdue Button icon slot -->
<span style="display:inline-flex;align-items:center;justify-content:center;flex-shrink:0;width:20px;height:20px">
  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
       stroke-linecap="round" stroke-linejoin="round" width="20" height="20">
    <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
    <circle cx="8.5" cy="8.5" r="1.5"></circle>
    <polyline points="21 15 16 10 5 21"></polyline>
  </svg>
</span>
```

| Placement | Size (Large / Medium) | Size (Small) |
|---|---|---|
| Leading Icon | 20 × 20px | 16 × 16px |
| Trailing Icon | 20 × 20px | 16 × 16px |

---

#### 4a. Fill Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.tertiary` | `Primitive/color/red/100` | `#FEE2E2` | `#7F1D1D` |
| Label | Color | `text.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Leading Icon | Color | `icon.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.tertiary-hover` | `Primitive/color/red/200` | `#FECACA` | `#991B1B` |
| Label | Color | `text.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Leading Icon | Color | `icon.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#363B3F` |
| Label | Color | `text.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |
| Leading Icon | Color | `icon.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.quaternary` | `Primitive/color/red/50` | `#FFF0F0` | `#450A0A` |
| Container | Border | `border.danger.quaternary` | `Primitive/color/red/100` | `#FEE2E2` | `#7F1D1D` |
| Label | Color | `text.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Leading Icon | Color | `icon.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |

---

#### 4b. Outline Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | *(transparent — no fill)* | — | — | — |
| Label | Color | `text.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Leading Icon | Color | `icon.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Border | Stroke | `border.dangerButton.tertiary` | `Primitive/color/red/200` | `#FECACA` | `#7F1D1D` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.quaternary` | `Primitive/color/red/50` | `#FFF0F0` | `#450A0A` |
| Label | Color | `text.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Leading Icon | Color | `icon.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Border | Stroke | `border.dangerButton.quaternary-hover` | `Primitive/color/red/200` | `#FECACA` | `#991B1B` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#363B3F` |
| Label | Color | `text.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |
| Leading Icon | Color | `icon.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |
| Border | Stroke | `border.disabledButton.tertiary` | `Primitive/color/neutral/200` | `#E2E4E6` | `#363B3F` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.dangerButton.quaternary` | `Primitive/color/red/50` | `#FFF0F0` | `#450A0A` |
| Label | Color | `text.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Leading Icon | Color | `icon.dangerButton.default` | `Primitive/color/red/700` | `#B91C1C` | `#FCA5A5` |
| Border | Stroke | `border.danger.quaternary` | `Primitive/color/red/100` | `#FEE2E2` | `#7F1D1D` |

---

### 5.5. Neutral Button

> 🔗 [ดู Button Preview](./button_preview.html)

**Component Properties:** style (Fill / Outline / Ghost) · size (Large / Medium / Small) · state (Default / Hover / Disable / Selected) · showLeadingIcon · showTailingIcon · Source: Figma node `2024-3591`

#### Spacing (shared with Brand Button — verified from Figma 377:12081)

| Size | Height | Padding X | Padding Y | Gap | Icon Size | Min Width | Max Width |
|---|---|---|---|---|---|---|---|
| Large | 44px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Medium | 40px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Small | 36px | `dimension/space/300` (12px) | `dimension/space/150` (6px) | `dimension/space/150` (6px) | 16 × 16px | 80px | 384px |

#### Font

| Size | Family | Weight | Size | Line Height |
|---|---|---|---|---|
| Large / Medium | Sarabun | Medium (500) | 16px | 1.5 |
| Small | Sarabun | Medium (500) | 14px | 1.5 |

#### Icon

> **Icon (Figma node 2024-3591):** Feather `image` icon เหมือน Brand Button — ใช้เป็น placeholder slot. ทั้ง leading และ trailing ใช้ icon เดียวกัน. Neutral Button รองรับ Ghost style.

```html
<!-- Feather image icon — Neutral Button icon slot -->
<span style="display:inline-flex;align-items:center;justify-content:center;flex-shrink:0;width:20px;height:20px">
  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
       stroke-linecap="round" stroke-linejoin="round" width="20" height="20">
    <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
    <circle cx="8.5" cy="8.5" r="1.5"></circle>
    <polyline points="21 15 16 10 5 21"></polyline>
  </svg>
</span>
```

| Placement | Size (Large / Medium) | Size (Small) |
|---|---|---|
| Leading Icon | 20 × 20px | 16 × 16px |
| Trailing Icon | 20 × 20px | 16 × 16px |

---

#### 5a. Fill Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Label | Color | `text.neutralButton.on-neutral` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |
| Leading Icon | Color | `icon.neutralButton.on-neutral` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.default-hover` | `Primitive/color/neutral/800` | `#363B3F` | `#E2E4E6` |
| Label | Color | `text.neutralButton.on-neutral` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |
| Leading Icon | Color | `icon.neutralButton.on-neutral` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Label | Color | `text.disabledButton.on-disabled` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |
| Leading Icon | Color | `icon.disablediconButton.on-disabled` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.secondary` | `Primitive/color/neutral/200` | `#E2E4E6` | `#363B3F` |
| Label | Color | `text.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |

---

#### 5b. Outline Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.quinary` | `Primitive/color/grey/50` | `#FFFFFF` | `#111314` |
| Label | Color | `text.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Border | Stroke | `border.neutralButton.default` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.quinary-hover` | `Primitive/color/neutral/50` | `#F9FBFB` | `#111314` |
| Label | Color | `text.neutralButton.secondary` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |
| Leading Icon | Color | `icon.neutralButton.secondary` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |
| Border | Stroke | `border.neutralButton.default-hover` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.quinary` | `Primitive/color/grey/50` | `#FFFFFF` | `#DADADA` |
| Label | Color | `text.disabledButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.disablediconButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Border | Stroke | `border.disabledButton.default` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#222629` |
| Label | Color | `text.neutralButton.secondary` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |
| Leading Icon | Color | `icon.neutralButton.secondary` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |
| Border | Stroke | `border.neutralButton.tertiary` | `Primitive/color/neutral/200` | `#E2E4E6` | `#4D5358` |

---

#### 5c. Ghost Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.quaternary` | `Primitive/color/neutral/50` | `#F9FBFB` | `#111314` |
| Label | Color | `text.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.quaternary-hover` | `Primitive/color/neutral/100` | `#F4F5F5` | `#222629` |
| Label | Color | `text.neutralButton.secondary` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |
| Leading Icon | Color | `icon.neutralButton.secondary` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#363B3F` |
| Label | Color | `text.disabledButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.disablediconButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#222629` |
| Label | Color | `text.neutralButton.secondary` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |
| Leading Icon | Color | `icon.neutralButton.secondary` | `Primitive/color/neutral/600` | `#636B72` | `#ADB2B7` |

---

### 5.6. Neutral Subdue Button

Neutral subdue uses lighter neutral backgrounds compared to the standard neutral button. Source: Figma node `2024-2940`.

> 🔗 [ดู Button Preview](./button_preview.html)

**Component Properties:** style (Fill / Outline — **ไม่มี Ghost**) · size (Large / Medium / Small) · state (Default / Hover / Disable / Selected) · showLeadingIcon · showTailingIcon

#### Spacing (shared with Brand Button — verified from Figma 377:12081)

| Size | Height | Padding X | Padding Y | Gap | Icon Size | Min Width | Max Width |
|---|---|---|---|---|---|---|---|
| Large | 44px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Medium | 40px | `dimension/space/400` (16px) | `dimension/space/200` (8px) | `dimension/space/200` (8px) | 20 × 20px | 80px | 384px |
| Small | 36px | `dimension/space/300` (12px) | `dimension/space/150` (6px) | `dimension/space/150` (6px) | 16 × 16px | 80px | 384px |

#### Font

| Size | Family | Weight | Size | Line Height |
|---|---|---|---|---|
| Large / Medium | Sarabun | Medium (500) | 16px | 1.5 |
| Small | Sarabun | Medium (500) | 14px | 1.5 |

#### Icon

> **Icon (Figma node 2024-2940):** Feather `image` icon เหมือน Brand Button — ใช้เป็น placeholder slot. ทั้ง leading และ trailing ใช้ icon เดียวกัน. ไม่มี Ghost style สำหรับ Subdue variants.

```html
<!-- Feather image icon — Neutral Subdue Button icon slot -->
<span style="display:inline-flex;align-items:center;justify-content:center;flex-shrink:0;width:20px;height:20px">
  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
       stroke-linecap="round" stroke-linejoin="round" width="20" height="20">
    <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
    <circle cx="8.5" cy="8.5" r="1.5"></circle>
    <polyline points="21 15 16 10 5 21"></polyline>
  </svg>
</span>
```

| Placement | Size (Large / Medium) | Size (Small) |
|---|---|---|
| Leading Icon | 20 × 20px | 16 × 16px |
| Trailing Icon | 20 × 20px | 16 × 16px |

---

#### 6a. Fill Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.secondary` | `Primitive/color/neutral/200` | `#E2E4E6` | `#363B3F` |
| Label | Color | `text.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.secondary-hover` | `Primitive/color/neutral/300` | `#C9CDD0` | `#4D5358` |
| Label | Color | `text.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#363B3F` |
| Label | Color | `text.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |
| Leading Icon | Color | `icon.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.quaternary` | `Primitive/color/neutral/50` | `#F9FBFB` | `#111314` |
| Container | Border | `border.neutral.quaternary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#222629` |
| Label | Color | `text.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |

---

#### 6b. Outline Style

##### Default State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | *(transparent — no fill)* | — | — | — |
| Label | Color | `text.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Border | Stroke | `border.neutralButton.tertiary` | `Primitive/color/neutral/200` | `#E2E4E6` | `#4D5358` |

##### Hover State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.quaternary` | `Primitive/color/neutral/50` | `#F9FBFB` | `#111314` |
| Label | Color | `text.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Border | Stroke | `border.neutralButton.tertiary-hover` | `Primitive/color/neutral/300` | `#C9CDD0` | `#636B72` |

##### Disabled State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.disabledButton.tertiary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#363B3F` |
| Label | Color | `text.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |
| Leading Icon | Color | `icon.disabledButton.tertiary` | `Primitive/color/neutral/500` | `#ADB2B7` | `#636B72` |
| Border | Stroke | `border.disabledButton.tertiary` | `Primitive/color/neutral/200` | `#E2E4E6` | `#363B3F` |

##### Selected State

| Layer | Property | Semantic Token | Primitive Alias | Light | Dark |
|---|---|---|---|---|---|
| Container | Background | `surface.neutralButton.quaternary` | `Primitive/color/neutral/50` | `#F9FBFB` | `#111314` |
| Label | Color | `text.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Leading Icon | Color | `icon.neutralButton.default` | `Primitive/color/neutral/700` | `#4D5358` | `#C9CDD0` |
| Border | Stroke | `border.neutral.quaternary` | `Primitive/color/neutral/100` | `#F4F5F5` | `#222629` |

---

### 5.7. Icon-Only Buttons

Icon-only buttons are **square** containers (`width = height`) with a single centered icon. They use the same surface/border/icon tokens as their text counterparts — no `text.*` token applies since there is no label.

#### Dimension Summary

| Size | Width × Height | Icon Size | Border Radius | Dimension Token |
|---|---|---|---|---|
| Small | 36 × 36px | 16 × 16px | `dimension.radius.200` (8px) | `dimension.size.700` |
| Medium | 40 × 40px | 20 × 20px | `dimension.radius.200` (8px) | `dimension.size.800` |
| Large | 44 × 44px | 20 × 20px | `dimension.radius.200` (8px) | `dimension.size.900` |

> Circle variants (§7g, §7h) use `dimension.radius.600` (24px) — full pill radius.

---

#### 7a. Brand Icon Button (Square)

Source: Figma node `377-11596`. Styles: Fill, Outline, Ghost. Sizes: Small (36px), Medium (40px), Large (44px).

| Style | State | Surface Token | Icon Token |
|---|---|---|---|
| Fill | Default | `surface.brandPrimaryButton.default` → `#007549` / `#84DBB4` | `icon.brandPrimaryButton.on-brand` → `#FFFFFF` / `#020617` |
| Fill | Hover | `surface.brandPrimaryButton.default-hover` → `#004C31` / `#C5EDDA` | `icon.brandPrimaryButton.on-brand` → `#FFFFFF` / `#020617` |
| Fill | Disabled | `surface.disabledButton.default` → `#4D5358` / `#C9CDD0` | `icon.disablediconButton.on-disabled` → `#FFFFFF` / `#111314` |
| Fill | Selected | `surface.brandPrimaryButton.secondary` → `#C5EDDA` / `#004C31` | `icon.brandPrimaryButton.secondary` → `#08A768` / `#6AD6A5` |
| Outline | Default | `surface.brandPrimaryButton.quinary` → `#FFFFFF` / `#020617` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Hover | `surface.brandPrimaryButton.quinary-hover` → `#F4FBF8` / `#002318` | `icon.brandPrimaryButton.default-hover` → `#004C31` / `#C5EDDA` |
| Outline | Disabled | `surface.disabledButton.quinary` → `#FFFFFF` / `#DADADA` | `icon.disablediconButton.default` → `#4D5358` / `#C9CDD0` |
| Outline | Selected | `surface.brandPrimaryButton.tertiary` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.tertiary` → `#6AD6A5` / `#08A768` |
| Ghost | Default | `surface.brandPrimaryButton.quaternary` → `#F4FBF8` / `#002318` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Ghost | Hover | `surface.brandPrimaryButton.quaternary-hover` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.secondary-hover` → `#007549` / `#84DBB4` |
| Ghost | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | `icon.disablediconButton.default` → `#4D5358` / `#C9CDD0` |
| Ghost | Selected | `surface.brandPrimaryButton.tertiary` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.tertiary` → `#6AD6A5` / `#08A768` |

---

#### 7b. Brand Subdue Icon Button (Square)

Source: Figma node `2024-4352`. Styles: Fill, Outline. No Ghost. Sizes: Small, Medium, Large.

| Style | State | Surface Token | Icon Token |
|---|---|---|---|
| Fill | Default | `surface.brandPrimaryButton.secondary` → `#C5EDDA` / `#004C31` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Fill | Hover | `surface.brandPrimaryButton.secondary-hover` → `#84DBB4` / `#007549` | `icon.brandPrimaryButton.secondary-hover` → `#007549` / `#84DBB4` |
| Fill | Disabled | `surface.disabledButton.secondary` → `#E2E4E6` / `#363B3F` | `icon.disablediconButton.default` → `#4D5358` / `#C9CDD0` |
| Fill | Selected | `surface.brandPrimaryButton.tertiary` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.tertiary` → `#6AD6A5` / `#08A768` |
| Outline | Default | `surface.brandPrimaryButton.quinary` → `#FFFFFF` / `#020617` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Hover | `surface.brandPrimaryButton.quinary-hover` → `#F4FBF8` / `#002318` | `icon.brandPrimaryButton.secondary-hover` → `#007549` / `#84DBB4` |
| Outline | Disabled | `surface.disabledButton.quinary` → `#FFFFFF` / `#DADADA` | `icon.disablediconButton.default` → `#4D5358` / `#C9CDD0` |
| Outline | Selected | `surface.brandPrimaryButton.tertiary` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.tertiary` → `#6AD6A5` / `#08A768` |

---

#### 7c. Danger Icon Button (Square)

Source: Figma node `2024-3701`. Styles: Fill, Outline, Ghost. Sizes: Small, Medium, Large.

| Style | State | Surface Token | Icon Token |
|---|---|---|---|
| Fill | Default | `surface.dangerButton.default` → `#B91C1C` / `#FCA5A5` | `icon.dangerButton.on-default` → `#FFFFFF` / `#020617` |
| Fill | Hover | `surface.dangerButton.default-hover` → `#991B1B` / `#FECACA` | `icon.dangerButton.on-default` → `#FFFFFF` / `#020617` |
| Fill | Disabled | `surface.disabledButton.default` → `#4D5358` / `#C9CDD0` | `icon.disablediconButton.on-disabled` → `#FFFFFF` / `#111314` |
| Fill | Selected | `surface.dangerButton.secondary` → `#FECACA` / `#991B1B` | `icon.dangerButton.secondary` → `#DC2626` / `#F87171` |
| Outline | Default | `surface.dangerButton.quinary` → `#FFFFFF` / `#DADADA` | `icon.dangerButton.default` → `#B91C1C` / `#FCA5A5` |
| Outline | Hover | `surface.dangerButton.quinary-hover` → `#FFF0F0` / `#450A0A` | `icon.dangerButton.secondary` → `#DC2626` / `#F87171` |
| Outline | Disabled | `surface.disabledButton.quinary` → `#FFFFFF` / `#DADADA` | `icon.disablediconButton.default` → `#4D5358` / `#C9CDD0` |
| Outline | Selected | `surface.dangerButton.tertiary` → `#FEE2E2` / `#7F1D1D` | `icon.dangerButton.tertiary` → `#F87171` / `#DC2626` |
| Ghost | Default | `surface.dangerButton.quaternary` → `#FFF0F0` / `#450A0A` | `icon.dangerButton.default` → `#B91C1C` / `#FCA5A5` |
| Ghost | Hover | `surface.dangerButton.quaternary-hover` → `#FEE2E2` / `#7F1D1D` | `icon.dangerButton.secondary` → `#DC2626` / `#F87171` |
| Ghost | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | `icon.disablediconButton.default` → `#4D5358` / `#C9CDD0` |
| Ghost | Selected | `surface.dangerButton.tertiary` → `#FEE2E2` / `#7F1D1D` | `icon.dangerButton.tertiary` → `#F87171` / `#DC2626` |

---

#### 7d. Neutral Icon Button (Square)

Source: Figma node `2024-4189`. Styles: Fill, Outline, Ghost. Sizes: Small, Medium, Large.

| Style | State | Surface Token | Icon Token |
|---|---|---|---|
| Fill | Default | `surface.neutralButton.default` → `#4D5358` / `#C9CDD0` | `icon.neutralButton.on-neutral` → `#FFFFFF` / `#111314` |
| Fill | Hover | `surface.neutralButton.default-hover` → `#363B3F` / `#E2E4E6` | `icon.neutralButton.on-neutral` → `#FFFFFF` / `#111314` |
| Fill | Disabled | `surface.disabledButton.default` → `#4D5358` / `#C9CDD0` | `icon.disablediconButton.on-disabled` → `#FFFFFF` / `#111314` |
| Fill | Selected | `surface.neutralButton.secondary` → `#E2E4E6` / `#363B3F` | `icon.neutralButton.default` → `#4D5358` / `#C9CDD0` |
| Outline | Default | `surface.neutralButton.quinary` → `#FFFFFF` / `#111314` | `icon.neutralButton.default` → `#4D5358` / `#C9CDD0` |
| Outline | Hover | `surface.neutralButton.quinary-hover` → `#F9FBFB` / `#111314` | `icon.neutralButton.secondary` → `#636B72` / `#ADB2B7` |
| Outline | Disabled | `surface.disabledButton.quinary` → `#FFFFFF` / `#DADADA` | `icon.disablediconButton.default` → `#4D5358` / `#C9CDD0` |
| Outline | Selected | `surface.neutralButton.tertiary` → `#F4F5F5` / `#222629` | `icon.neutralButton.secondary` → `#636B72` / `#ADB2B7` |
| Ghost | Default | `surface.neutralButton.quaternary` → `#F9FBFB` / `#111314` | `icon.neutralButton.default` → `#4D5358` / `#C9CDD0` |
| Ghost | Hover | `surface.neutralButton.quaternary-hover` → `#F4F5F5` / `#222629` | `icon.neutralButton.secondary` → `#636B72` / `#ADB2B7` |
| Ghost | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | `icon.disablediconButton.default` → `#4D5358` / `#C9CDD0` |
| Ghost | Selected | `surface.neutralButton.tertiary` → `#F4F5F5` / `#222629` | `icon.neutralButton.secondary` → `#636B72` / `#ADB2B7` |

---

#### 7e. Danger Subdue Icon Button

Source: Figma node `2024-4434`. Styles: Fill, Outline. No Ghost style on Subdue Icon variants.

> ⚠️ Note: Outline Hover uses `border.dangerButton.secondary` (`#F87171`) — one step darker than the text button Outline Hover — matching the tighter visual weight of an icon-only button.

| Style | State | Surface Token | Border Token | Icon Token |
|---|---|---|---|---|
| Fill | Default | `surface.dangerButton.tertiary` → `#FEE2E2` / `#7F1D1D` | — | `icon.dangerButton.default` → `#B91C1C` / `#FCA5A5` |
| Fill | Hover | `surface.dangerButton.tertiary-hover` → `#FECACA` / `#991B1B` | — | `icon.dangerButton.default` → `#B91C1C` / `#FCA5A5` |
| Fill | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | — | `icon.disabledButton.tertiary` → `#ADB2B7` / `#636B72` |
| Fill | Selected | `surface.dangerButton.quaternary` → `#FFF0F0` / `#450A0A` | `border.dangerButton.quaternary` → `#FEE2E2` / `#7F1D1D` | `icon.dangerButton.default` → `#B91C1C` / `#FCA5A5` |
| Outline | Default | *(transparent)* | `border.dangerButton.tertiary` → `#FECACA` / `#7F1D1D` | `icon.dangerButton.default` → `#B91C1C` / `#FCA5A5` |
| Outline | Hover | `surface.dangerButton.quaternary` → `#FFF0F0` / `#450A0A` | `border.dangerButton.secondary` → `#F87171` / `#DC2626` | `icon.dangerButton.default` → `#B91C1C` / `#FCA5A5` |
| Outline | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | `border.disabledButton.tertiary` → `#E2E4E6` / `#363B3F` | `icon.disabledButton.tertiary` → `#ADB2B7` / `#636B72` |
| Outline | Selected | `surface.dangerButton.quaternary` → `#FFF0F0` / `#450A0A` | `border.dangerButton.quaternary` → `#FEE2E2` / `#7F1D1D` | `icon.dangerButton.default` → `#B91C1C` / `#FCA5A5` |

---

#### 7f. Neutral Subdue Icon Button

Source: Figma node `2024-4516`. Styles: Fill, Outline. No Ghost style on Subdue Icon variants.

> ⚠️ Note: Outline Default and Outline Hover share the same border token (`border.neutralButton.tertiary` = `#E2E4E6`) — the hover visual difference comes only from the background fill appearing.

| Style | State | Surface Token | Border Token | Icon Token |
|---|---|---|---|---|
| Fill | Default | `surface.neutralButton.secondary` → `#E2E4E6` / `#363B3F` | — | `icon.neutralButton.default` → `#4D5358` / `#C9CDD0` |
| Fill | Hover | `surface.neutralButton.secondary-hover` → `#C9CDD0` / `#4D5358` | — | `icon.neutralButton.default` → `#4D5358` / `#C9CDD0` |
| Fill | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | — | `icon.disabledButton.tertiary` → `#ADB2B7` / `#636B72` |
| Fill | Selected | `surface.neutralButton.quaternary` → `#F9FBFB` / `#111314` | `border.neutralButton.quaternary` → `#F4F5F5` / `#222629` | `icon.neutralButton.default` → `#4D5358` / `#C9CDD0` |
| Outline | Default | *(transparent)* | `border.neutralButton.tertiary` → `#E2E4E6` / `#4D5358` | `icon.neutralButton.default` → `#4D5358` / `#C9CDD0` |
| Outline | Hover | `surface.neutralButton.quaternary` → `#F9FBFB` / `#111314` | `border.neutralButton.tertiary` → `#E2E4E6` / `#4D5358` | `icon.neutralButton.default` → `#4D5358` / `#C9CDD0` |
| Outline | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | `border.disabledButton.tertiary` → `#E2E4E6` / `#363B3F` | `icon.disabledButton.tertiary` → `#ADB2B7` / `#636B72` |
| Outline | Selected | `surface.neutralButton.quaternary` → `#F9FBFB` / `#111314` | `border.neutralButton.quaternary` → `#F4F5F5` / `#222629` | `icon.neutralButton.default` → `#4D5358` / `#C9CDD0` |

---

#### 7g. Brand Circle Icon Button

Source: Figma node `2103-548`. Border radius: `dimension.radius.600` = **24px** (full circle). Sizes: Small (36px), Medium (40px), Large (44px). Styles: Fill, Outline, Ghost.

> ⚠️ Key differences from Brand Square Icon Button (§7a):
> - **Fill Hover = Fill Default** — same `surface.brandPrimaryButton.default` token, no darkening on hover
> - **All Selected states** (Fill/Outline/Ghost) share `surface.brandPrimaryButton.quaternary` (`#F4FBF8`) + `border.brandPrimaryButton.quaternary` (`#E2F3EB`)
> - **Ghost Default** is fully transparent — no background token applied
> - **Outline Hover border** stays at `.default` token, does not darken to `.default-hover`
> - **Disabled** uses `.tertiary` surface (lighter) vs square icon button's `.default` surface for Fill

| Style | State | Surface Token | Border Token | Icon Token |
|---|---|---|---|---|
| Fill | Default | `surface.brandPrimaryButton.default` → `#007549` / `#84DBB4` | — | `icon.brandPrimaryButton.on-brand` → `#FFFFFF` / `#020617` |
| Fill | Hover | `surface.brandPrimaryButton.default` → `#007549` / `#84DBB4` | — | `icon.brandPrimaryButton.on-brand` → `#FFFFFF` / `#020617` |
| Fill | Selected | `surface.brandPrimaryButton.quaternary` → `#F4FBF8` / `#002318` | `border.brandPrimaryButton.quaternary` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Fill | Focus | `surface.brandPrimaryButton.default` → `#007549` / `#84DBB4` | `dimension.stroke.400` (`4px`) ring · `border.brandPrimaryButton.tertiary` → `#C5EDDA` / `#004C31` | `icon.brandPrimaryButton.on-brand` → `#FFFFFF` / `#020617` |
| Fill | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | — | `icon.disabledButton.tertiary` → `#ADB2B7` / `#636B72` |
| Outline | Default | *(transparent)* | `border.brandPrimaryButton.default` → `#007549` / `#84DBB4` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Hover | `surface.brandPrimaryButton.quaternary` → `#F4FBF8` / `#002318` | `border.brandPrimaryButton.default` → `#007549` / `#84DBB4` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Selected | `surface.brandPrimaryButton.quaternary` → `#F4FBF8` / `#002318` | `border.brandPrimaryButton.quaternary` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Focus | *(transparent)* | `dimension.stroke.400` (`4px`) ring · `border.brandPrimaryButton.tertiary` → `#C5EDDA` / `#004C31` *(replaces 1px outline)* | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | `border.disabledButton.tertiary` → `#E2E4E6` / `#363B3F` | `icon.disabledButton.tertiary` → `#ADB2B7` / `#636B72` |
| Ghost | Default | *(transparent — no bg token)* | — | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Ghost | Hover | `surface.brandPrimaryButton.quaternary` → `#F4FBF8` / `#002318` | — | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Ghost | Selected | `surface.brandPrimaryButton.quaternary` → `#F4FBF8` / `#002318` | `border.brandPrimaryButton.quaternary` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Ghost | Focus | `surface.brandPrimaryButton.quaternary` → `#F4FBF8` / `#002318` | `dimension.stroke.400` (`4px`) ring · `border.brandPrimaryButton.tertiary` → `#C5EDDA` / `#004C31` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Ghost | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | — | `icon.disabledButton.tertiary` → `#ADB2B7` / `#636B72` |

> **Drop shadow:** Fill Default and Fill Hover carry `Brand Drop Shadow Bottom/300` effect (`rgba(8,167,104,0.08–0.10)`, 4px offset) — absent on all other states and styles.
>
> **Focus ring:** Focus applies a 4px (`dimension.stroke.400`) `border.brandPrimaryButton.tertiary` ring on top of the state's existing surface, independent of `Selected`. The two states can stack — a Selected button that receives keyboard focus shows the brand-tertiary ring around the quaternary surface.

---

#### 7h. Brand Circle Subdue Icon Button

Source: Figma node `2103-645`. Border radius: `dimension.radius.600` = **24px** (full circle). Styles: Fill, Outline **only** (no Ghost). Sizes: **XSmall (32px)**, Small (36px), Medium (40px), Large (44px).

> ⚠️ Key differences from Brand Subdue Square Icon Button (§7b):
> - **Fill Default** uses **tertiary** surface (`#E2F3EB`) — lighter than Brand Subdue Icon's **secondary** (`#C5EDDA`)
> - **Fill Hover** → `surface.brandPrimaryButton.tertiary-hover` = `#C5EDDA` (still lighter than Brand Subdue Icon's `.secondary-hover` = `#84DBB4`)
> - **Outline Hover** → `surface.brandPrimaryButton.quaternary-hover` (`#E2F3EB`) + `border.brandPrimaryButton.quaternary-hover` (`#C5EDDA`)
> - **XSmall** (32px / `dimension.size.600`) size available — unique to this variant; icon size is 14px (`dimension.size.300`)

| Style | State | Surface Token | Border Token | Icon Token |
|---|---|---|---|---|
| Fill | Default | `surface.brandPrimaryButton.tertiary` → `#E2F3EB` / `#003322` | — | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Fill | Hover | `surface.brandPrimaryButton.tertiary-hover` → `#C5EDDA` / `#004C31` | — | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Fill | Active | `surface.brandPrimaryButton.quaternary` → `#F4FBF8` / `#002318` | `border.brandPrimaryButton.quaternary` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Fill | Focus | `surface.brandPrimaryButton.tertiary` → `#E2F3EB` / `#003322` | `dimension.stroke.400` (`4px`) ring · `border.brandPrimaryButton.tertiary` → `#C5EDDA` / `#004C31` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Fill | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | — | `icon.disabledButton.tertiary` → `#ADB2B7` / `#636B72` |
| Outline | Default | *(transparent)* | `border.brandPrimary.default` → `#007549` / `#84DBB4` (`1px`) | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Hover | `surface.brandPrimaryButton.tertiary` → `#E2F3EB` / `#003322` *(light tint on hover)* | `border.brandPrimary.default` → `#007549` / `#84DBB4` (`1px`) | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Active | `surface.brandPrimaryButton.quaternary` → `#F4FBF8` / `#002318` | `border.brandPrimaryButton.quaternary` → `#E2F3EB` / `#003322` | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Focus | *(transparent)* | `dimension.stroke.400` (`4px`) ring · `border.brandPrimaryButton.tertiary` → `#C5EDDA` / `#004C31` *(replaces 1px outline)* | `icon.brandPrimaryButton.default` → `#007549` / `#84DBB4` |
| Outline | Disabled | `surface.disabledButton.tertiary` → `#F4F5F5` / `#363B3F` | `border.disabledButton.tertiary` → `#E2E4E6` / `#363B3F` | `icon.disabledButton.tertiary` → `#ADB2B7` / `#636B72` |

> **Focus ring:** Same 4px tertiary-border pattern as Brand Circle Icon Button (§7g) — applied on top of the existing surface, with Active stacking visually when both states coincide.
>
> **Active vs Selected:** Per Figma node `2103-645`, this family uses the term **Active** (toggle-on metaphor) where the standard Circle Icon Button (§7g) uses **Selected**. The token mapping is the same; the prop in code is `active` (with `selected` accepted as a legacy alias).

---

### 5.8. Typography Tokens

All button labels use the UI font family. Font size and weight vary by button **Size**.

| Property | Token | Value |
|---|---|---|
| Font Family | `typography.family.ui` | `Sarabun` |
| Font Style (default) | `typography.style.medium` | `Medium` |
| Font Weight | `typography.weight.medium` | `500` |

#### Size-Specific Typography

| Button Size | Figma Height | Recommended Font Size Token | Resolved Value |
|---|---|---|---|
| Small | 36px | `typography.size.sm` | `14px` |
| Medium | 40px | `typography.size.base` | `16px` |
| Large | 44px | `typography.size.base` | `16px` |
| XSmall (Circle Subdue only) | 32px | `typography.size.xs` | `12px` |

> **Note:** The Figma data provides component dimensions rather than explicit typography variable bindings in the sparse section-level export. The size mappings above follow standard design system conventions for the observed component heights. Confirm per-component typography in the full node detail view if precise validation is required.

---

### 5.9. Spacing & Dimension Tokens

#### Button Container Heights

| Size Name | Height | Dimension Token | Resolved |
|---|---|---|---|
| XSmall | 32px | `dimension.size.600` | `32px` |
| Small | 36px | `dimension.size.700` | `36px` |
| Medium | 40px | `dimension.size.800` | `40px` |
| Large | 44px | `dimension.size.900` | `44px` |

#### Internal Padding (Horizontal)

Standard text buttons use horizontal padding to achieve their measured widths (111px Small, 137px Medium/Large). The following tokens apply:

| Size | Padding X Token | Resolved | Padding Y Token | Resolved |
|---|---|---|---|---|
| Small | `dimension.space.300` | `12px` | `dimension.space.200` | `8px` |
| Medium | `dimension.space.400` | `16px` | `dimension.space.200` | `8px` |
| Large | `dimension.space.400` | `16px` | `dimension.space.300` | `12px` |

#### Gap Between Icon and Label

| Token | Resolved |
|---|---|
| `dimension.space.200` | `8px` |

#### Border Radius

Buttons use the **full** pill radius by default in this design system:

| Token | Resolved | Notes |
|---|---|---|
| `dimension.radius.full` | `9999px` | Standard pill shape for all Brand, Neutral, Danger text & icon buttons |
| `dimension.radius.200` | `8px` | Alternative rounded rect (if applied to specific variants) |

#### Border Width (Outline Style)

| Token | Resolved |
|---|---|
| `dimension.stroke.100` | `1px` |

#### Icon Size

| Button Size | Icon Size Token | Resolved |
|---|---|---|
| XSmall | `dimension.size.300` | `16px` |
| Small | `dimension.size.400` | `20px` |
| Medium | `dimension.size.400` | `20px` |
| Large | `dimension.size.500` | `24px` |

---

### 5.10. Summary: Token Quick-Reference by Button Family

| Button Family | Surface (Default) | Text/Icon (on-fill) | Border (Outline) |
|---|---|---|---|
| Brand Fill | `surface.brandPrimaryButton.default` (`#007549`) | `text/icon.brandPrimaryButton.on-brand` (`#FFFFFF`) | — |
| Brand Outline | `surface.brandPrimaryButton.quinary` (`#FFFFFF`) | `text/icon.brandPrimaryButton.default` (`#007549`) | `border.brandPrimaryButton.default` (`#007549`) |
| Brand Ghost | `surface.brandPrimaryButton.quaternary` (`#F4FBF8`) | `text/icon.brandPrimaryButton.default` (`#007549`) | — |
| Brand Subdue Fill | `surface.brandPrimaryButton.secondary` (`#C5EDDA`) | `text/icon.brandPrimaryButton.default` (`#007549`) | — |
| Brand Subdue Outline | `surface.brandPrimaryButton.quinary` (`#FFFFFF`) | `text/icon.brandPrimaryButton.default` (`#007549`) | `border.brandPrimaryButton.tertiary` (`#C5EDDA`) |
| Danger Fill | `surface.dangerButton.default` (`#B91C1C`) | `text/icon.dangerButton.on-default` (`#FFFFFF`) | — |
| Danger Outline | `surface.dangerButton.quinary` (`#FFFFFF`) | `text/icon.dangerButton.default` (`#B91C1C`) | `border.dangerButton.default` (`#DC2626`) |
| Danger Ghost | `surface.dangerButton.quaternary` (`#FFF0F0`) | `text/icon.dangerButton.default` (`#B91C1C`) | — |
| Danger Subdue Fill | `surface.dangerButton.tertiary` (`#FEE2E2`) | `text/icon.dangerButton.default` (`#B91C1C`) | — |
| Danger Subdue Outline | *(transparent)* | `text/icon.dangerButton.default` (`#B91C1C`) | `border.dangerButton.tertiary` (`#FECACA`) |
| Neutral Fill | `surface.neutralButton.default` (`#4D5358`) | `text/icon.neutralButton.on-neutral` (`#FFFFFF`) | — |
| Neutral Outline | `surface.neutralButton.quinary` (`#FFFFFF`) | `text/icon.neutralButton.default` (`#4D5358`) | `border.neutralButton.default` (`#636B72`) |
| Neutral Ghost | `surface.neutralButton.quaternary` (`#F9FBFB`) | `text/icon.neutralButton.default` (`#4D5358`) | — |
| Neutral Subdue Fill | `surface.neutralButton.secondary` (`#E2E4E6`) | `text/icon.neutralButton.default` (`#4D5358`) | — |
| Neutral Subdue Outline | `surface.neutralButton.quinary` (`#FFFFFF`) | `text/icon.neutralButton.default` (`#4D5358`) | `border.neutralButton.secondary` (`#ADB2B7`) |
| Any Disabled | `surface.disabledButton.*` (fill/outline/ghost level) | `text/icon.disabledButton.on-disabled` or `.default` | `border.disabledButton.*` |

> **Icon-only buttons** use the same surface/border tokens as their text counterparts — replace `text.*` with `icon.*` only. See §5.7 for full state tables.

| Icon Button Family | Surface (Fill Default) | Icon Color | Border (Outline Default) |
|---|---|---|---|
| Brand Icon | `surface.brandPrimaryButton.default` (`#007549`) | `icon.brandPrimaryButton.on-brand` (`#FFFFFF`) | `border.brandPrimaryButton.default` (`#007549`) |
| Brand Subdue Icon | `surface.brandPrimaryButton.secondary` (`#C5EDDA`) | `icon.brandPrimaryButton.default` (`#007549`) | `border.brandPrimaryButton.tertiary` (`#C5EDDA`) |
| Danger Icon | `surface.dangerButton.default` (`#B91C1C`) | `icon.dangerButton.on-default` (`#FFFFFF`) | `border.dangerButton.default` (`#DC2626`) |
| Danger Subdue Icon | `surface.dangerButton.tertiary` (`#FEE2E2`) | `icon.dangerButton.default` (`#B91C1C`) | `border.dangerButton.tertiary` (`#FECACA`) |
| Neutral Icon | `surface.neutralButton.default` (`#4D5358`) | `icon.neutralButton.on-neutral` (`#FFFFFF`) | `border.neutralButton.default` (`#636B72`) |
| Neutral Subdue Icon | `surface.neutralButton.secondary` (`#E2E4E6`) | `icon.neutralButton.default` (`#4D5358`) | `border.neutralButton.tertiary` (`#E2E4E6`) |

---

#### 5.10.1. Button Usage Rules — Do & Don't

##### ✅ Do

| # | Rule | Rationale |
|---|------|-----------|
| 1 | **Use Fill for the primary CTA, Outline for secondary, Ghost for tertiary** | Creates clear visual hierarchy — the user always knows which action is most important |
| 2 | **Use Subdue Fill / Subdue Outline for low-emphasis actions inside cards, panels, or table rows** | Reduces visual noise in dense UI while preserving interactivity; avoids competing with the main CTA |
| 3 | **Use Neutral buttons for UI-level actions that are brand-agnostic** — filter, sort, expand, collapse | Reserve Brand green for business/conversion actions; Neutral signals utility, not commitment |
| 4 | **Apply Danger style only for destructive or irreversible actions** — delete, remove, reset | Danger red carries strong semantic weight; misusing it dilutes its warning value |
| 5 | **Use Danger Subdue Fill / Outline for soft-warning contexts** — archive, flag, unpublish (reversible) | Signals caution without the alarm of full Danger — appropriate when the action is recoverable |
| 6 | **Pair icon-only buttons with `aria-label` or `title`** — always | Icon-only buttons (`btn-icon`) are invisible to screen readers without an accessible label |
| 7 | **Keep button sizes consistent within the same action group** — Sm / Md / Lg must not be mixed | Mixing sizes in one row creates visual imbalance and breaks spatial alignment |
| 8 | **Use the `disabled` attribute — not `opacity: 0.5`** when an action is unavailable | The system provides explicit `surface/text/disabledButton.*` tokens; opacity overlay bypasses them and fails WCAG |

##### ❌ Don't

| # | Rule | Why |
|---|------|-----|
| 1 | **Don't place two Fill buttons side by side** | Two equal-weight CTAs confuse users about which action to take — one must always be demoted to Outline or Ghost |
| 2 | **Don't mix Brand and Danger Fill in the same action group** | Conflicting semantic colors (green + red) obscure visual hierarchy and alarm users unnecessarily |
| 3 | **Don't use Subdue Fill as the sole CTA in a standalone context** | Subdue variants have low visual weight by design — they will be missed as a primary action outside dense UI |
| 4 | **Don't use Ghost as a primary CTA** | Ghost (`quaternary` surface) is intentionally low-emphasis — it will go unnoticed when used as the main action |
| 5 | **Don't use Danger style for non-destructive actions** — e.g., "Save changes", "Submit form" | Users will hesitate or abandon a safe action if it appears dangerous |
| 6 | **Don't use icon-only buttons for complex or unfamiliar actions** | Icons require learned meaning — use a text label or icon + label when the action isn't universally understood |
| 7 | **Don't mix button sizes in the same action group** | Sm + Lg pairing in one row breaks visual rhythm and spatial consistency |
| 8 | **Don't style a `<div>` or `<a>` as a button** without `role="button"` and keyboard handling | Breaks keyboard navigation and assistive technology — always use `<button>` for interactive triggers |

##### Style → Hierarchy Quick Map

| Intent | Style | Key Token |
|--------|-------|-----------|
| Primary CTA (brand) | Brand Fill | `surface/brandPrimaryButton/default` |
| Secondary CTA (brand) | Brand Outline | `border/brandPrimaryButton/default` |
| Tertiary CTA (brand) | Brand Ghost | `surface/brandPrimaryButton/quaternary` |
| Passive / low-emphasis (brand) | Brand Subdue Fill · Subdue Outline | `surface/brandPrimaryButton/tertiary` |
| UI action (brand-agnostic) | Neutral Fill · Outline · Ghost | `surface/neutralButton/default` |
| Passive UI action | Neutral Subdue Fill · Subdue Outline | `surface/neutralButton/secondary` |
| Destructive action | Danger Fill · Outline · Ghost | `surface/dangerButton/default` |
| Soft warning / reversible risk | Danger Subdue Fill · Subdue Outline | `surface/dangerButton/tertiary` |
| Unavailable action | Disabled tokens | `surface/disabledButton/tertiary` (never `opacity`) |
| Icon-only (any family) | + `.btn-icon` class | square `36/40/44px` · requires `aria-label` |

---

### 5.11. Badge

Badges are non-interactive labels used to communicate status, category, or count. The MIH system has two main families: **Status Badge** and **Colorful Badge** (including Colorful Icon Badge/CIB).

---

#### 5.11.1. Status Badge

Status badges use semantic color tokens that convey meaning (info, success, warning, error, neutral). Each badge may contain a leading icon (semantic shape SVG) or dot (filled circle), and a text label.

**Sizes**

| Size | Height | Icon size | H-padding | Token |
|------|--------|-----------|-----------|-------|
| Medium (`bd-md`) | 24px | 14×14px | 8px | `dimension.height.badge.md` = `24` |
| Large (`bd-lg`) | 32px | 16×16px | 12px | `dimension.height.badge.lg` = `32` |

- Border-radius: `dimension.radius.full` (fully rounded pill)
- Gap (icon ↔ text): `dimension.gap.badge` = `4px`

**Token Mapping — Status Badge**

| Style | Surface Token | Icon Token | Border Token | Resolved Color |
|-------|--------------|------------|--------------|----------------|
| Neutral | `surface.statusBadge.neutral` | `icon.statuslBadge.neutral` | — | Surface `#E2E4E6` · Icon `#636B72` |
| Info | `surface.statusBadge.info` | `icon.statuslBadge.info` | — | Surface `#DBEAFE` · Icon `#1D4ED8` |
| Success | `surface.statusBadge.success` | `icon.statuslBadge.success` | — | Surface `#D1FAE5` · Icon `#065F46` |
| Warning | `surface.statusBadge.warning` | `icon.statuslBadge.warning` | — | Surface `#FEF3C7` · Icon `#92400E` |
| Error | `surface.statusBadge.error` | `icon.statuslBadge.error` | — | Surface `#FEE2E2` · Icon `#991B1B` |

> **Note:** Figma variable name for icon tokens uses the typo `statuslBadge` (lowercase "l" before "Badge"). This is intentional in the Figma token file — use `icon.statuslBadge.*` exactly as documented.

**Icon usage per status**

| Style | Icon element | SVG shape |
|-------|-------------|-----------|
| Neutral | Dot (`.badge-dot`) | Filled circle `r=4` |
| Info | Semantic icon (`.badge-semantic-icon`) | Circle outline + `i` glyph |
| Success | Semantic icon (`.badge-semantic-icon`) | Circle outline + checkmark |
| Warning | Semantic icon (`.badge-semantic-icon`) | Triangle outline + `!` |
| Error | Semantic icon (`.badge-semantic-icon`) | Circle outline + `×` |

---

#### 5.11.2. Colorful Badge — Subtle

Subtle colorful badges use a light tinted surface with a matching icon dot. No border token.

**Sizes (`ColorfulBadge` `size` prop)**

| Size | Height | Label | H-padding | V-padding | Dot | Icon | Source |
|------|--------|-------|-----------|-----------|-----|------|--------|
| `default` | 32px (`dimension.size.600`) | `typography.size.sm` 14 Medium | 12px (`space.300`) | 4px (`space.100`) | 8 | 16 | Figma DS |
| `small` | 24px (`dimension.size.500`) | `typography.size.xs` 12 Medium | 8px (`space.200`) | 4px (`space.100`) | 6 | 16 | Figma DS |
| `xsmall` | 20px (`dimension.size.400`) | `typography.size.xxs` 10 Medium | 6px (`space.150`) | 2px (`space.050`) | 4 | 12 | Code-side (not in Figma DS) — inline tag next to body text, e.g. the "ใหม่" new-patient tag after a worklist name (20 ก.ย. 69) |

**Token Mapping — Colorful Badge Subtle**

| Color | Surface Token | Icon Token | Resolved Surface | Resolved Icon |
|-------|--------------|------------|-----------------|---------------|
| Grey | `surface.colorfulBadge.grey` | `icon.colorfulBadge.grey` | `#E2E4E6` | `#636B72` |
| Red | `surface.colorfulBadge.red` | `icon.colorfulBadge.red` | `#FEE2E2` | `#991B1B` |
| Orange | `surface.colorfulBadge.orange` | `icon.colorfulBadge.orange` | `#FFEDD5` | `#9A3412` |
| Amber | `surface.colorfulBadge.amber` | `icon.colorfulBadge.amber` | `#FEF3C7` | `#92400E` |
| Yellow | `surface.colorfulBadge.yellow` | `icon.colorfulBadge.yellow` | `#FEF9C3` | `#854D0E` |
| Lime | `surface.colorfulBadge.lime` | `icon.colorfulBadge.lime` | `#ECFCCB` | `#3F6212` |
| Green | `surface.colorfulBadge.green` | `icon.colorfulBadge.green` | `#D1FAE5` | `#065F46` |
| Teal | `surface.colorfulBadge.teal` | `icon.colorfulBadge.teal` | `#CCFBF1` | `#134E4A` |
| Cyan | `surface.colorfulBadge.cyan` | `icon.colorfulBadge.cyan` | `#CFFAFE` | `#164E63` |
| Blue | `surface.colorfulBadge.blue` | `icon.colorfulBadge.blue` | `#DBEAFE` | `#1D4ED8` |
| Purple | `surface.colorfulBadge.purple` | `icon.colorfulBadge.purple` | `#EDE9FE` | `#5B21B6` |
| Pink | `surface.colorfulBadge.pink` | `icon.colorfulBadge.pink` | `#FCE7F3` | `#9D174D` |

All subtle badges use `.badge-dot` (filled circle SVG) as the leading icon element.

---

#### 5.11.3. Colorful Badge — Bold

Bold colorful badges use a saturated filled surface with a white icon dot.

**Token Mapping — Colorful Badge Bold**

| Color | Surface Token | Icon Token | Resolved Surface |
|-------|--------------|------------|-----------------|
| Grey Bold | `surface.colorfulBadge.grey-bold` | `icon.colorfulBadge.grey-bold` | `#636B72` |
| Red Bold | `surface.colorfulBadge.red-bold` | `icon.colorfulBadge.red-bold` | `#DC2626` |
| Orange Bold | `surface.colorfulBadge.orange-bold` | `icon.colorfulBadge.orange-bold` | `#EA580C` |
| Amber Bold | `surface.colorfulBadge.amber-bold` | `icon.colorfulBadge.amber-bold` | `#D97706` |
| Yellow Bold | `surface.colorfulBadge.yellow-bold` | `icon.colorfulBadge.yellow-bold` | `#CA8A04` |
| Lime Bold | `surface.colorfulBadge.lime-bold` | `icon.colorfulBadge.lime-bold` | `#65A30D` |
| Green Bold | `surface.colorfulBadge.green-bold` | `icon.colorfulBadge.green-bold` | `#059669` |
| Teal Bold | `surface.colorfulBadge.teal-bold` | `icon.colorfulBadge.teal-bold` | `#0D9488` |
| Cyan Bold | `surface.colorfulBadge.cyan-bold` | `icon.colorfulBadge.cyan-bold` | `#0891B2` |
| **Blue Bold** | `surface.colorfulBadge.blue-bold` ([`--surface-colorfulbadge-blue-bold`](src/index.css)) | white | **`#3B82F6`** (blue/500 — Figma 3396:1627 / 3401:4293 AISearch recent-history ICD code prefix) |
| **Indigo Bold** | `surface.colorfulBadge.indigo-bold` ([`--surface-colorfulbadge-indigo-bold`](src/index.css)) | white | **`#818CF8`** (indigo/400 — Figma 156:27354 Doctor AI ICD chip prefix) |
| Purple Bold | `surface.colorfulBadge.purple-bold` | `icon.colorfulBadge.purple-bold` | `#7C3AED` |
| Pink Bold | `surface.colorfulBadge.pink-bold` | `icon.colorfulBadge.pink-bold` | `#DB2777` |

Icon token resolves to `#FFFFFF` (white) for all bold variants.

**How `ColorfulBadge` selects the bold surface.** When `hierarchy='primary'`, the component renders `var(--surface-colorfulbadge-${style}-bold, var(--text-colorfulbadge-${style}))`. The CSS variable fallback chain means colors that have a published `*-bold` token (currently **blue** + **indigo**) pick up the dedicated mid-tone bg automatically — colors without one fall back to the saturated 700 accent. Adding more bold tokens to `src/index.css` (e.g. `--surface-colorfulbadge-teal-bold`) opts that style into the dedicated bold surface without any component edit. Outline + secondary hierarchies never touch the bold token.

```jsx
{/* Figma 156:27354 — Doctor-AI ICD code chip badge. */}
<ColorfulBadge style='indigo' hierarchy='primary'>A90</ColorfulBadge>
{/* Renders bg #818CF8 (indigo-bold), text #FFFFFF. */}
```

---

#### 5.11.4. Colorful Badge — Outline

Outline badges use a transparent/white surface with a colored border and matching icon.

**Token Mapping — Colorful Badge Outline**

| Color | Surface Token | Icon Token | Border Token | Resolved Border |
|-------|--------------|------------|--------------|-----------------|
| Grey Outline | `surface.colorfulBadge.grey-ol` | `icon.colorfulBadge.grey` | `border.colorfulBadge.grey` | `#ADB2B7` |
| Green Outline | `surface.colorfulBadge.green-ol` | `icon.colorfulBadge.green` | `border.colorfulBadge.green` | `#6EE7B7` |

- Surface resolves to `#FFFFFF` (white / transparent)
- Border width: `dimension.stroke.100` = `1px`
- Icon uses the same subtle-tier icon token as the matching subtle variant

---

#### 5.11.5. Colorful Icon Badge (CIB)

CIB is a circular badge container with a centered SVG icon. It is used standalone (no text label).

**Sizes**

| Size class | Container size | Icon size | Token |
|------------|---------------|-----------|-------|
| `cib-sm` | 24×24px | 12×12px | `dimension.size.cib.sm` = `24` |
| `cib-md` | 32×32px | 14×14px | `dimension.size.cib.md` = `32` |
| `cib-lg` | 40×40px | 16×16px | `dimension.size.cib.lg` = `40` |
| `cib-xl` | 48×48px | 20×20px | `dimension.size.cib.xl` = `48` |

- Border-radius: `dimension.radius.full` (circle)

**Token Mapping — CIB**

| Color | Surface Token | Icon Token | Resolved Surface |
|-------|--------------|------------|-----------------|
| Grey | `surface.colorfulBadge.grey` | `icon.colorfulBadge.grey` | `#E2E4E6` · `#636B72` |
| Red | `surface.colorfulBadge.red` | `icon.colorfulBadge.red` | `#FEE2E2` · `#991B1B` |
| Orange | `surface.colorfulBadge.orange` | `icon.colorfulBadge.orange` | `#FFEDD5` · `#9A3412` |
| Amber | `surface.colorfulBadge.amber` | `icon.colorfulBadge.amber` | `#FEF3C7` · `#92400E` |
| Yellow | `surface.colorfulBadge.yellow` | `icon.colorfulBadge.yellow` | `#FEF9C3` · `#854D0E` |
| Lime | `surface.colorfulBadge.lime` | `icon.colorfulBadge.lime` | `#ECFCCB` · `#3F6212` |
| Green | `surface.colorfulBadge.green` | `icon.colorfulBadge.green` | `#D1FAE5` · `#065F46` |
| Teal | `surface.colorfulBadge.teal` | `icon.colorfulBadge.teal` | `#CCFBF1` · `#134E4A` |
| Cyan | `surface.colorfulBadge.cyan` | `icon.colorfulBadge.cyan` | `#CFFAFE` · `#164E63` |
| Blue | `surface.colorfulBadge.blue` | `icon.colorfulBadge.blue` | `#DBEAFE` · `#1D4ED8` |
| Purple | `surface.colorfulBadge.purple` | `icon.colorfulBadge.purple` | `#EDE9FE` · `#5B21B6` |
| Pink | `surface.colorfulBadge.pink` | `icon.colorfulBadge.pink` | `#FCE7F3` · `#9D174D` |
| Grey Outline (`grey-ol`) | `surface.colorfulBadge.grey-ol` | `icon.colorfulBadge.grey` | `#FFFFFF` · `border.colorfulBadge.grey` `#ADB2B7` |
| Green Outline (`green-ol`) | `surface.colorfulBadge.green-ol` | `icon.colorfulBadge.green` | `#FFFFFF` · `border.colorfulBadge.green` `#6EE7B7` |

CIB uses the `.cib-icon` class on its inner `<svg>` element; toggling the Icon control in the preview hides/shows `.cib-icon` alongside `.badge-semantic-icon`.

---

#### 5.11.6. Badge Text Tokens

| Style Family | Text Token | Resolved Color |
|-------------|-----------|----------------|
| Status Badge (all) | `text.statuslBadge.default` | Inherits icon token color (same as `icon.statuslBadge.*`) |
| Colorful Badge Subtle (all) | `text.colorfulBadge.default` | Inherits icon token color (same as `icon.colorfulBadge.*`) |
| Colorful Badge Bold (all) | `text.colorfulBadge.on-bold` | `#FFFFFF` |

> Text tokens follow the same "statusl" typo convention as icon tokens for status badges.

---

#### 5.11.7. Badge Dimension Summary

| Token | Value | Description |
|-------|-------|-------------|
| `dimension.height.badge.md` | `24px` | Medium badge height |
| `dimension.height.badge.lg` | `32px` | Large badge height |
| `dimension.padding.badge.md` | `0 8px` | Medium horizontal padding |
| `dimension.padding.badge.lg` | `0 12px` | Large horizontal padding |
| `dimension.gap.badge` | `4px` | Icon-to-text gap |
| `dimension.radius.full` | `9999px` | Pill border-radius (all badges) |
| `dimension.stroke.100` | `1px` | Outline badge border width |
| `dimension.size.badge.icon.md` | `14px` | Icon size in md badge |
| `dimension.size.badge.icon.lg` | `16px` | Icon size in lg badge |
| `dimension.size.cib.sm` | `24px` | CIB small container |
| `dimension.size.cib.md` | `32px` | CIB medium container |
| `dimension.size.cib.lg` | `40px` | CIB large container |
| `dimension.size.cib.xl` | `48px` | CIB extra-large container |

---

#### 5.11.8. Usage Rules — Do & Don't

##### ✅ Do

| # | Rule | Example |
|---|------|---------|
| 1 | **ต้องมีอย่างน้อย 1 element** — icon, dot, หรือ text อย่างน้อยหนึ่งอย่าง | `<badge text-only>`, `<badge dot-only>`, `<CIB icon-only>` |
| 2 | **ใช้ semantic color ตรงความหมาย** — info/success/warning/error ต้องสื่อความหมายที่ถูกต้อง | Success badge สำหรับ "Approved", Error badge สำหรับ "Failed" |
| 3 | **ใช้ colorful badge สำหรับ category / tag** — ที่ไม่ใช่ status meaning | blue "Design", purple "UX", green "Frontend" |
| 4 | **ข้อความสั้นกระชับ 1–3 คำ** — badge ไม่ใช่ที่สำหรับ description | "Active", "Draft", "Pending", "In Review" |
| 5 | **ใช้ขนาดสม่ำเสมอในบริบทเดียวกัน** — ไม่ผสม md (32px) และ sm (24px) ในแถวเดียวกัน | ใช้ `bd-md` ทั้งหมดในตาราง หรือ default (24px) ทั้งหมด |

##### ❌ Don't

| # | Rule | Why |
|---|------|-----|
| 1 | **ห้ามใช้ badge เปล่า** — badge ที่ไม่มี icon, dot, หรือ text เลย | Empty pill ไม่สื่อความหมาย และ violates minimum content requirement |
| 2 | **ห้ามใช้ icon + dot พร้อมกันบน badge เดียว** — เลือกอย่างใดอย่างหนึ่ง | Redundant double indicator ทำให้ badge ดู cluttered และ inconsistent กับ Figma spec |
| 3 | **ห้ามใช้ badge เป็น interactive element** — ไม่มี click handler, cursor pointer, หรือ underline | Badge เป็น informational label ไม่ใช่ button หรือ link — ใช้ Button component แทน |
| 4 | **ห้ามใช้ text ที่ยาวเกินไป** — ไม่ truncate text ด้วย ellipsis | Badge ที่ truncate สูญเสีย readability — ถ้าข้อมูลยาว ใช้ chip หรือ tag component แทน |
| 5 | **ห้ามใช้สีผิด semantic** — เช่น ใช้ red bold badge สำหรับ non-error เพียงเพราะ aesthetic | ทำให้ users สับสนกับ error state — ใช้ colorful badge แทน status badge หากไม่ต้องการสื่อ status |

##### Minimum Content Rule (สรุป)

```
Badge ต้องมีอย่างน้อย:
  icon  (badge-semantic-icon สำหรับ status)
  OR
  dot   (badge-dot สำหรับ neutral status + colorful)
  OR
  text  (badge-text)

CIB (Colorful Icon Badge) = icon-only badge → ยกเว้นกฎ text requirement
Empty badge = ❌ INVALID
```

---

### 5.12. Notes for Developers

1. **Brand mode switching**: All tokens prefixed `Brand/primary/*` or `Brand/secondary/*` will automatically remap when the brand mode changes (e.g., switching from `default` to `sky-cyan`). No component code changes are required.

2. **Disabled opacity vs. disabled tokens**: This system uses **explicit disabled tokens** (separate color values) rather than opacity overlays. Apply the disabled token directly — do not add `opacity: 0.5`.

3. **Surface vs. Background tokens**: Both token groups exist. `surface.*` is preferred for interactive UI components. `background.*` is used in non-interactive or layout contexts.

4. **Selected state**: The "Selected" state represents a toggled-on condition (e.g., a filter button that is active). It uses a lighter surface level than the Default state to signal selection without the full visual weight of the filled button.

5. **Border width for Outline style**: Always `dimension.stroke.100` = `1px`. The stroke is inside the component bounds.

6. **Icon-only buttons**: Use the same surface/icon tokens as text buttons at the same style/state combination. Circle variants use `dimension.radius.full` for the container; square icon buttons use `dimension.radius.200` (8px) or `dimension.radius.full` depending on the specific frame.

---

### 5.13. Figma ↔ React Component Map

Central index of Figma component_sets / components in MIH Design System Foundation (`SBdh4TtY0KAa22s3dnyRNr`) paired with their React implementations. Use this as the **source of truth** for cross-tool navigation — open the Figma node in the design tool, then jump to the React file (and vice-versa).

**Why this exists (not Code Connect):** Figma's official Code Connect mapping API (`send_code_connect_mappings`, `@figma/code-connect` CLI) requires an **Organization or Enterprise plan with Developer seats**. Until the org upgrades, this table is the manual equivalent — pair-tested with `get_metadata` on the live DS file.

**Convention** — every React component file in `src/components/` SHOULD include a header docblock referencing its Figma node id, e.g.:

```jsx
/* MIH BrandButton — Figma node 231:29 (Brand Button ✅ · MIH Design System Foundation). */
```

The ✅ suffix marks the **approved** version of the component; some files reference older draft node ids historically (e.g. Alert.jsx had `325:2623` before the approved `297:59` set landed). When updating, supersede the old id and note it inline rather than deleting the history.

#### Verified mappings (Phase 1)

| Figma node | Figma name | React file | React export |
|---|---|---|---|
| `231:29`     | Brand Button ✅                 | [src/components/button/BrandButton.jsx](src/components/button/BrandButton.jsx) | `BrandButton` |
| `377:11596`  | Brand Icon Button ✅            | [src/components/button/BrandIconButton.jsx](src/components/button/BrandIconButton.jsx) | `BrandIconButton` |
| `302:73`     | Status Badge ✅                 | [src/components/badge/StatusBadge.jsx](src/components/badge/StatusBadge.jsx) | `StatusBadge` |
| `2076:8413`  | Colorful Badge ✅               | [src/components/badge/ColorfulBadge.jsx](src/components/badge/ColorfulBadge.jsx) | `ColorfulBadge` |
| `2307:11697` | Colorful icon Badge ✅          | [src/components/badge/ColorfulIconBadge.jsx](src/components/badge/ColorfulIconBadge.jsx) | `ColorfulIconBadge` |
| `297:59`     | Alert (Large) ✅                | [src/components/alert/Alert.jsx](src/components/alert/Alert.jsx) | `Alert` |
| `3355:3149`  | Small Alert ✅                  | [src/components/alert/SmallAlert.jsx](src/components/alert/SmallAlert.jsx) | `SmallAlert` |
| `2796:3168`  | Action List ✅                  | [src/components/list/ActionList.jsx](src/components/list/ActionList.jsx) | `ActionList` |
| `292:21`     | Accordion ✅                    | _(no React equivalent yet — TODO)_ | — |

#### Pending mappings (Phase 2 — to be filled)

To extend this table, run a one-off Figma scan (`use_figma` walking `figma.root.children` filtered to `COMPONENT_SET` nodes), then verify each React file exists. Add rows for at least: Neutral Button, Neutral Icon Button, Brand Subdue Button, Danger Button, Inputs (TextField, Select), Checkbox, Radio, Switch, Popup, Tabs, Stepper, Date Picker, Calendar.

The MIH DS Foundation file has **~1500 component nodes** total (most are Feather icon imports on the `icon ✍️` page — those are *not* the target. Focus on `Component ✅` / `Button ✅ 🌙 ✍️` / `Badge ✅ 🌙 ✍️` etc. — the ✅-flagged production versions).

#### Workflow when adding a new DS component

1. Build the variant set in MIH Design System Foundation, name it `Component/<Name>` with ✅ suffix when approved.
2. Capture its node id via `get_metadata` or inspecting the URL.
3. Create the React file in `src/components/<family>/<Name>.jsx`.
4. Add docblock header: `/* MIH <Name> — Figma node <id> (<Name> ✅ · MIH DS Foundation). */`.
5. Add a row to §5.13 above.
6. Add a Guidelines entry to [ComponentsLibraryPage.jsx](src/pages/components/ComponentsLibraryPage.jsx) with the demo + Do/Don't matrix.

---

## Appendix: How to Use This Reference

### Token Path Format

```
{layer}.{category}.{variant}.{state}
```

**Examples:**
```
background.brandPrimary.default         → Brand primary color (changes per brand mode)
background.brandPrimary.default-hover   → Hover state of the above
text.neutral.default                    → Neutral text color (same across brands)
border.brandPrimary.default             → Brand-colored border
```

### Implementing Multi-Brand Support

1. Use **Semantic tokens** in all components (never Primitive tokens directly)
2. Switch the **Brand mode** (default/B/C/D/E/F/G) to change `primary` + `secondary` colors globally
3. Switch the **light/dark mode** on the Semantic layer to change the full theme
4. Switch the **Platform mode** (desktop/mobile/tablet) to apply responsive sizing

### Quick Reference: Brand Mode Colors

| Mode | Primary (500) | Secondary (500) |
|------|--------------|-----------------|
| `default` | `#54CF97` | `#53954E` |
| `blue-serenity` | `#8EA0F8` | `#8EA0F8` |
| `cool-lavender` | `#6E45BC` | `#6E45BC` |
| `dusty-mauve` | `#977177` | `#977177` |
| `muted-teal` | `#6F9693` | `#6F9693` |
| `pink` | `#EC4899` | `#EF4444` |
| `rose` | `#E27B7F` | `#E27B7F` |
| `sky-cyan` | `#0E90E8` | `#0E90E8` |
| `soft-peach` | `#E05825` | `#E05825` |
| `warm-gray` | `#8A8078` | `#8A8078` |


---

## Appendix B: Icon Reference (MIH Design System)

> **Source:** Figma node `194:3530` · [MIH Design System Foundation](https://www.figma.com/design/SBdh4TtY0KAa22s3dnyRNr/MIH-Design-System-Foundation?node-id=194-3530)  
> **Libraries:** Feather Icons (standard) · Material Symbols Outlined (MIH custom)  
> **Total:** 332 icons  
> **Usage:** Set `fill="currentColor"` on the `<svg>` to inherit text color

---

### Standard Icons (Feather Icons)

| Preview | Name | Node ID | Source |
|---------|------|---------|--------|
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-activity"><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"></polyline></svg> | `Activity` | `319:2129` | `feather:activity` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-airplay"><path d="M5 17H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v10a2 2 0 0 1-2 2h-1"></path><polygon points="12 15 17 21 7 21 12 15"></polygon></svg> | `Airplay` | `319:2130` | `feather:airplay` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-alert-circle"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="8" x2="12" y2="12"></line><line x1="12" y1="16" x2="12.01" y2="16"></line></svg> | `Alert circle` | `319:2131` | `feather:alert-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-alert-octagon"><polygon points="7.86 2 16.14 2 22 7.86 22 16.14 16.14 22 7.86 22 2 16.14 2 7.86 7.86 2"></polygon><line x1="12" y1="8" x2="12" y2="12"></line><line x1="12" y1="16" x2="12.01" y2="16"></line></svg> | `Alert octagon` | `319:2132` | `feather:alert-octagon` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-alert-triangle"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path><line x1="12" y1="9" x2="12" y2="13"></line><line x1="12" y1="17" x2="12.01" y2="17"></line></svg> | `Alert triangle` | `319:2133` | `feather:alert-triangle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-align-center"><line x1="18" y1="10" x2="6" y2="10"></line><line x1="21" y1="6" x2="3" y2="6"></line><line x1="21" y1="14" x2="3" y2="14"></line><line x1="18" y1="18" x2="6" y2="18"></line></svg> | `Align center` | `319:2134` | `feather:align-center` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-align-justify"><line x1="21" y1="10" x2="3" y2="10"></line><line x1="21" y1="6" x2="3" y2="6"></line><line x1="21" y1="14" x2="3" y2="14"></line><line x1="21" y1="18" x2="3" y2="18"></line></svg> | `Align justify` | `319:2135` | `feather:align-justify` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-align-left"><line x1="17" y1="10" x2="3" y2="10"></line><line x1="21" y1="6" x2="3" y2="6"></line><line x1="21" y1="14" x2="3" y2="14"></line><line x1="17" y1="18" x2="3" y2="18"></line></svg> | `Align left` | `319:2136` | `feather:align-left` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-align-right"><line x1="21" y1="10" x2="7" y2="10"></line><line x1="21" y1="6" x2="3" y2="6"></line><line x1="21" y1="14" x2="3" y2="14"></line><line x1="21" y1="18" x2="7" y2="18"></line></svg> | `Align right` | `319:2137` | `feather:align-right` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-anchor"><circle cx="12" cy="5" r="3"></circle><line x1="12" y1="22" x2="12" y2="8"></line><path d="M5 12H2a10 10 0 0 0 20 0h-3"></path></svg> | `Anchor` | `319:2138` | `feather:anchor` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-aperture"><circle cx="12" cy="12" r="10"></circle><line x1="14.31" y1="8" x2="20.05" y2="17.94"></line><line x1="9.69" y1="8" x2="21.17" y2="8"></line><line x1="7.38" y1="12" x2="13.12" y2="2.06"></line><line x1="9.69" y1="16" x2="3.95" y2="6.06"></line><line x1="14.31" y1="16" x2="2.83" y2="16"></line><line x1="16.62" y1="12" x2="10.88" y2="21.94"></line></svg> | `Aperture` | `319:2128` | `feather:aperture` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-archive"><polyline points="21 8 21 21 3 21 3 8"></polyline><rect x="1" y="3" width="24" height="24"></rect><line x1="10" y1="12" x2="14" y2="12"></line></svg> | `Archive` | `319:2127` | `feather:archive` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-down-circle"><circle cx="12" cy="12" r="10"></circle><polyline points="8 12 12 16 16 12"></polyline><line x1="12" y1="8" x2="12" y2="16"></line></svg> | `Arrow down-circle` | `319:2126` | `feather:arrow-down-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-down-left"><line x1="17" y1="7" x2="7" y2="17"></line><polyline points="17 17 7 17 7 7"></polyline></svg> | `Arrow down-left` | `319:2125` | `feather:arrow-down-left` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-down-right"><line x1="7" y1="7" x2="17" y2="17"></line><polyline points="17 7 17 17 7 17"></polyline></svg> | `Arrow down-right` | `319:2124` | `feather:arrow-down-right` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-down"><line x1="12" y1="5" x2="12" y2="19"></line><polyline points="19 12 12 19 5 12"></polyline></svg> | `Arrow down` | `319:2123` | `feather:arrow-down` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-left-circle"><circle cx="12" cy="12" r="10"></circle><polyline points="12 8 8 12 12 16"></polyline><line x1="16" y1="12" x2="8" y2="12"></line></svg> | `Arrow left-circle` | `319:2122` | `feather:arrow-left-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-left"><line x1="19" y1="12" x2="5" y2="12"></line><polyline points="12 19 5 12 12 5"></polyline></svg> | `Arrow left` | `319:2121` | `feather:arrow-left` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-right-circle"><circle cx="12" cy="12" r="10"></circle><polyline points="12 16 16 12 12 8"></polyline><line x1="8" y1="12" x2="16" y2="12"></line></svg> | `Arrow right-circle` | `319:2120` | `feather:arrow-right-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-right"><line x1="5" y1="12" x2="19" y2="12"></line><polyline points="12 5 19 12 12 19"></polyline></svg> | `Arrow right` | `319:2119` | `feather:arrow-right` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-up-circle"><circle cx="12" cy="12" r="10"></circle><polyline points="16 12 12 8 8 12"></polyline><line x1="12" y1="16" x2="12" y2="8"></line></svg> | `Arrow up-circle` | `319:2109` | `feather:arrow-up-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-up-left"><line x1="17" y1="17" x2="7" y2="7"></line><polyline points="7 17 7 7 17 7"></polyline></svg> | `Arrow up-left` | `319:2110` | `feather:arrow-up-left` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-up-right"><line x1="7" y1="17" x2="17" y2="7"></line><polyline points="7 7 17 7 17 17"></polyline></svg> | `Arrow up-right` | `319:2111` | `feather:arrow-up-right` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-arrow-up"><line x1="12" y1="19" x2="12" y2="5"></line><polyline points="5 12 12 5 19 12"></polyline></svg> | `Arrow up` | `319:2112` | `feather:arrow-up` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-at-sign"><circle cx="12" cy="12" r="4"></circle><path d="M16 8v5a3 3 0 0 0 6 0v-1a10 10 0 1 0-3.92 7.94"></path></svg> | `At sign` | `319:2113` | `feather:at-sign` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-award"><circle cx="12" cy="8" r="7"></circle><polyline points="8.21 13.89 7 23 12 20 17 23 15.79 13.88"></polyline></svg> | `Award` | `319:2114` | `feather:award` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-bar-chart-2"><line x1="18" y1="20" x2="18" y2="10"></line><line x1="12" y1="20" x2="12" y2="4"></line><line x1="6" y1="20" x2="6" y2="14"></line></svg> | `Bar chart-2` | `319:2115` | `feather:bar-chart-2` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-bar-chart"><line x1="12" y1="20" x2="12" y2="10"></line><line x1="18" y1="20" x2="18" y2="4"></line><line x1="6" y1="20" x2="6" y2="16"></line></svg> | `Bar chart` | `319:2116` | `feather:bar-chart` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-battery-charging"><path d="M5 18H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h3.19M15 6h2a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2h-3.19"></path><line x1="23" y1="13" x2="23" y2="11"></line><polyline points="11 6 7 12 13 12 9 18"></polyline></svg> | `Battery charging` | `319:2117` | `feather:battery-charging` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-battery"><rect x="1" y="6" width="24" height="24" rx="2" ry="2"></rect><line x1="23" y1="13" x2="23" y2="11"></line></svg> | `Battery` | `319:2118` | `feather:battery` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-bell-off"><path d="M13.73 21a2 2 0 0 1-3.46 0"></path><path d="M18.63 13A17.89 17.89 0 0 1 18 8"></path><path d="M6.26 6.26A5.86 5.86 0 0 0 6 8c0 7-3 9-3 9h14"></path><path d="M18 8a6 6 0 0 0-9.33-5"></path><line x1="1" y1="1" x2="23" y2="23"></line></svg> | `Bell off` | `306:1700` | `feather:bell-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-bell"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path></svg> | `Bell` | `306:1701` | `feather:bell` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-bluetooth"><polyline points="6.5 6.5 17.5 17.5 12 23 12 1 17.5 6.5 6.5 17.5"></polyline></svg> | `Bluetooth` | `306:1702` | `feather:bluetooth` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-bold"><path d="M6 4h8a4 4 0 0 1 4 4 4 4 0 0 1-4 4H6z"></path><path d="M6 12h9a4 4 0 0 1 4 4 4 4 0 0 1-4 4H6z"></path></svg> | `Bold` | `306:1703` | `feather:bold` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-book-open"><path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"></path><path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"></path></svg> | `Book open` | `306:1704` | `feather:book-open` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-book"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path></svg> | `Book` | `306:1705` | `feather:book` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-bookmark"><path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z"></path></svg> | `Bookmark` | `306:1706` | `feather:bookmark` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-box"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"></path><polyline points="3.27 6.96 12 12.01 20.73 6.96"></polyline><line x1="12" y1="22.08" x2="12" y2="12"></line></svg> | `Box` | `306:1707` | `feather:box` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-briefcase"><rect x="2" y="7" width="24" height="24" rx="2" ry="2"></rect><path d="M16 21V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v16"></path></svg> | `Briefcase` | `306:1708` | `feather:briefcase` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-calendar"><rect x="3" y="4" width="24" height="24" rx="2" ry="2"></rect><line x1="16" y1="2" x2="16" y2="6"></line><line x1="8" y1="2" x2="8" y2="6"></line><line x1="3" y1="10" x2="21" y2="10"></line></svg> | `Calendar` | `306:1709` | `feather:calendar` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-camera-off"><line x1="1" y1="1" x2="23" y2="23"></line><path d="M21 21H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h3m3-3h6l2 3h4a2 2 0 0 1 2 2v9.34m-7.72-2.06a4 4 0 1 1-5.56-5.56"></path></svg> | `Camera off` | `294:8264` | `feather:camera-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-camera"><path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"></path><circle cx="12" cy="13" r="4"></circle></svg> | `Camera` | `294:8263` | `feather:camera` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-cast"><path d="M2 16.1A5 5 0 0 1 5.9 20M2 12.05A9 9 0 0 1 9.95 20M2 8V6a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v12a2 2 0 0 1-2 2h-6"></path><line x1="2" y1="20" x2="2.01" y2="20"></line></svg> | `Cast` | `294:8262` | `feather:cast` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-check-circle"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path><polyline points="22 4 12 14.01 9 11.01"></polyline></svg> | `Check circle` | `294:8261` | `feather:check-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-check-square"><polyline points="9 11 12 14 22 4"></polyline><path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"></path></svg> | `Check square` | `294:8260` | `feather:check-square` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-check"><polyline points="20 6 9 17 4 12"></polyline></svg> | `Check` | `294:8259` | `feather:check` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-chevron-down"><polyline points="6 9 12 15 18 9"></polyline></svg> | `Chevron down` | `294:7948` | `feather:chevron-down` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-chevron-left"><polyline points="15 18 9 12 15 6"></polyline></svg> | `Chevron left` | `294:8265` | `feather:chevron-left` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-chevron-right"><polyline points="9 18 15 12 9 6"></polyline></svg> | `Chevron right` | `294:8266` | `feather:chevron-right` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-chevron-up"><polyline points="18 15 12 9 6 15"></polyline></svg> | `Chevron up` | `294:8267` | `feather:chevron-up` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-chevrons-down"><polyline points="7 13 12 18 17 13"></polyline><polyline points="7 6 12 11 17 6"></polyline></svg> | `Chevrons down` | `306:925` | `feather:chevrons-down` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-chevrons-left"><polyline points="11 17 6 12 11 7"></polyline><polyline points="18 17 13 12 18 7"></polyline></svg> | `Chevrons left` | `306:926` | `feather:chevrons-left` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-chevrons-right"><polyline points="13 17 18 12 13 7"></polyline><polyline points="6 17 11 12 6 7"></polyline></svg> | `Chevrons right` | `306:933` | `feather:chevrons-right` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-chevrons-up"><polyline points="17 11 12 6 7 11"></polyline><polyline points="17 18 12 13 7 18"></polyline></svg> | `Chevrons up` | `306:934` | `feather:chevrons-up` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-chrome"><circle cx="12" cy="12" r="10"></circle><circle cx="12" cy="12" r="4"></circle><line x1="21.17" y1="8" x2="12" y2="8"></line><line x1="3.95" y1="6.06" x2="8.54" y2="14"></line><line x1="10.88" y1="21.94" x2="15.46" y2="14"></line></svg> | `Chrome` | `306:941` | `feather:chrome` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-circle"><circle cx="12" cy="12" r="10"></circle></svg> | `Circle` | `306:942` | `feather:circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-clipboard"><path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"></path><rect x="8" y="2" width="24" height="24" rx="1" ry="1"></rect></svg> | `Clipboard` | `306:949` | `feather:clipboard` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-clock"><circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline></svg> | `Clock` | `306:950` | `feather:clock` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-cloud-drizzle"><line x1="8" y1="19" x2="8" y2="21"></line><line x1="8" y1="13" x2="8" y2="15"></line><line x1="16" y1="19" x2="16" y2="21"></line><line x1="16" y1="13" x2="16" y2="15"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="12" y1="15" x2="12" y2="17"></line><path d="M20 16.58A5 5 0 0 0 18 7h-1.26A8 8 0 1 0 4 15.25"></path></svg> | `Cloud drizzle` | `306:957` | `feather:cloud-drizzle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-cloud-lightning"><path d="M19 16.9A5 5 0 0 0 18 7h-1.26a8 8 0 1 0-11.62 9"></path><polyline points="13 11 9 17 15 17 11 23"></polyline></svg> | `Cloud lightning` | `306:958` | `feather:cloud-lightning` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-cloud-off"><path d="M22.61 16.95A5 5 0 0 0 18 10h-1.26a8 8 0 0 0-7.05-6M5 5a8 8 0 0 0 4 15h9a5 5 0 0 0 1.7-.3"></path><line x1="1" y1="1" x2="23" y2="23"></line></svg> | `Cloud off` | `306:924` | `feather:cloud-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-cloud-rain"><line x1="16" y1="13" x2="16" y2="21"></line><line x1="8" y1="13" x2="8" y2="21"></line><line x1="12" y1="15" x2="12" y2="23"></line><path d="M20 16.58A5 5 0 0 0 18 7h-1.26A8 8 0 1 0 4 15.25"></path></svg> | `Cloud rain` | `306:927` | `feather:cloud-rain` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-cloud-snow"><path d="M20 17.58A5 5 0 0 0 18 8h-1.26A8 8 0 1 0 4 16.25"></path><line x1="8" y1="16" x2="8.01" y2="16"></line><line x1="8" y1="20" x2="8.01" y2="20"></line><line x1="12" y1="18" x2="12.01" y2="18"></line><line x1="12" y1="22" x2="12.01" y2="22"></line><line x1="16" y1="16" x2="16.01" y2="16"></line><line x1="16" y1="20" x2="16.01" y2="20"></line></svg> | `Cloud snow` | `306:932` | `feather:cloud-snow` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-cloud"><path d="M18 10h-1.26A8 8 0 1 0 9 20h9a5 5 0 0 0 0-10z"></path></svg> | `Cloud` | `306:935` | `feather:cloud` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-code"><polyline points="16 18 22 12 16 6"></polyline><polyline points="8 6 2 12 8 18"></polyline></svg> | `Code` | `306:940` | `feather:code` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-codepen"><polygon points="12 2 22 8.5 22 15.5 12 22 2 15.5 2 8.5 12 2"></polygon><line x1="12" y1="22" x2="12" y2="15.5"></line><polyline points="22 8.5 12 15.5 2 8.5"></polyline><polyline points="2 15.5 12 8.5 22 15.5"></polyline><line x1="12" y1="2" x2="12" y2="8.5"></line></svg> | `Codepen` | `306:943` | `feather:codepen` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-codesandbox"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"></path><polyline points="7.5 4.21 12 6.81 16.5 4.21"></polyline><polyline points="7.5 19.79 7.5 14.6 3 12"></polyline><polyline points="21 12 16.5 14.6 16.5 19.79"></polyline><polyline points="3.27 6.96 12 12.01 20.73 6.96"></polyline><line x1="12" y1="22.08" x2="12" y2="12"></line></svg> | `Codesandbox` | `306:948` | `feather:codesandbox` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-coffee"><path d="M18 8h1a4 4 0 0 1 0 8h-1"></path><path d="M2 8h16v9a4 4 0 0 1-4 4H6a4 4 0 0 1-4-4V8z"></path><line x1="6" y1="1" x2="6" y2="4"></line><line x1="10" y1="1" x2="10" y2="4"></line><line x1="14" y1="1" x2="14" y2="4"></line></svg> | `Coffee` | `306:951` | `feather:coffee` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-columns"><path d="M12 3h7a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2h-7m0-18H5a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h7m0-18v18"></path></svg> | `Columns` | `306:956` | `feather:columns` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-command"><path d="M18 3a3 3 0 0 0-3 3v12a3 3 0 0 0 3 3 3 3 0 0 0 3-3 3 3 0 0 0-3-3H6a3 3 0 0 0-3 3 3 3 0 0 0 3 3 3 3 0 0 0 3-3V6a3 3 0 0 0-3-3 3 3 0 0 0-3 3 3 3 0 0 0 3 3h12a3 3 0 0 0 3-3 3 3 0 0 0-3-3z"></path></svg> | `Command` | `306:959` | `feather:command` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-compass"><circle cx="12" cy="12" r="10"></circle><polygon points="16.24 7.76 14.12 14.12 7.76 16.24 9.88 9.88 16.24 7.76"></polygon></svg> | `Compass` | `306:923` | `feather:compass` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-copy"><rect x="9" y="9" width="24" height="24" rx="2" ry="2"></rect><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path></svg> | `Copy` | `306:928` | `feather:copy` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-corner-down-left"><polyline points="9 10 4 15 9 20"></polyline><path d="M20 4v7a4 4 0 0 1-4 4H4"></path></svg> | `Corner down-left` | `306:931` | `feather:corner-down-left` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-corner-down-right"><polyline points="15 10 20 15 15 20"></polyline><path d="M4 4v7a4 4 0 0 0 4 4h12"></path></svg> | `Corner down-right` | `306:936` | `feather:corner-down-right` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-corner-left-down"><polyline points="14 15 9 20 4 15"></polyline><path d="M20 4h-7a4 4 0 0 0-4 4v12"></path></svg> | `Corner left-down` | `306:939` | `feather:corner-left-down` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-corner-left-up"><polyline points="14 9 9 4 4 9"></polyline><path d="M20 20h-7a4 4 0 0 1-4-4V4"></path></svg> | `Corner left-up` | `306:944` | `feather:corner-left-up` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-corner-right-down"><polyline points="10 15 15 20 20 15"></polyline><path d="M4 4h7a4 4 0 0 1 4 4v12"></path></svg> | `Corner right-down` | `306:947` | `feather:corner-right-down` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-corner-right-up"><polyline points="10 9 15 4 20 9"></polyline><path d="M4 20h7a4 4 0 0 0 4-4V4"></path></svg> | `Corner right-up` | `306:952` | `feather:corner-right-up` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-corner-up-left"><polyline points="9 14 4 9 9 4"></polyline><path d="M20 20v-7a4 4 0 0 0-4-4H4"></path></svg> | `Corner up-left` | `306:955` | `feather:corner-up-left` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-corner-up-right"><polyline points="15 14 20 9 15 4"></polyline><path d="M4 20v-7a4 4 0 0 1 4-4h12"></path></svg> | `Corner up-right` | `306:960` | `feather:corner-up-right` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-cpu"><rect x="4" y="4" width="24" height="24" rx="2" ry="2"></rect><rect x="9" y="9" width="24" height="24"></rect><line x1="9" y1="1" x2="9" y2="4"></line><line x1="15" y1="1" x2="15" y2="4"></line><line x1="9" y1="20" x2="9" y2="23"></line><line x1="15" y1="20" x2="15" y2="23"></line><line x1="20" y1="9" x2="23" y2="9"></line><line x1="20" y1="14" x2="23" y2="14"></line><line x1="1" y1="9" x2="4" y2="9"></line><line x1="1" y1="14" x2="4" y2="14"></line></svg> | `Cpu` | `306:922` | `feather:cpu` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-credit-card"><rect x="1" y="4" width="24" height="24" rx="2" ry="2"></rect><line x1="1" y1="10" x2="23" y2="10"></line></svg> | `Credit card` | `306:929` | `feather:credit-card` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-crop"><path d="M6.13 1L6 16a2 2 0 0 0 2 2h15"></path><path d="M1 6.13L16 6a2 2 0 0 1 2 2v15"></path></svg> | `Crop` | `306:930` | `feather:crop` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-crosshair"><circle cx="12" cy="12" r="10"></circle><line x1="22" y1="12" x2="18" y2="12"></line><line x1="6" y1="12" x2="2" y2="12"></line><line x1="12" y1="6" x2="12" y2="2"></line><line x1="12" y1="22" x2="12" y2="18"></line></svg> | `Crosshair` | `306:937` | `feather:crosshair` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-database"><ellipse cx="12" cy="5" rx="9" ry="3"></ellipse><path d="M21 12c0 1.66-4 3-9 3s-9-1.34-9-3"></path><path d="M3 5v14c0 1.66 4 3 9 3s9-1.34 9-3V5"></path></svg> | `Database` | `306:938` | `feather:database` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-delete"><path d="M21 4H8l-7 8 7 8h13a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2z"></path><line x1="18" y1="9" x2="12" y2="15"></line><line x1="12" y1="9" x2="18" y2="15"></line></svg> | `Delete` | `306:945` | `feather:delete` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-disc"><circle cx="12" cy="12" r="10"></circle><circle cx="12" cy="12" r="3"></circle></svg> | `Disc` | `306:946` | `feather:disc` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-divide-circle"><line x1="8" y1="12" x2="16" y2="12"></line><line x1="12" y1="16" x2="12" y2="16"></line><line x1="12" y1="8" x2="12" y2="8"></line><circle cx="12" cy="12" r="10"></circle></svg> | `Divide circle` | `306:953` | `feather:divide-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-divide-square"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><line x1="8" y1="12" x2="16" y2="12"></line><line x1="12" y1="16" x2="12" y2="16"></line><line x1="12" y1="8" x2="12" y2="8"></line></svg> | `Divide square` | `306:954` | `feather:divide-square` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-divide"><circle cx="12" cy="6" r="2"></circle><line x1="5" y1="12" x2="19" y2="12"></line><circle cx="12" cy="18" r="2"></circle></svg> | `Divide` | `306:961` | `feather:divide` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-dollar-sign"><line x1="12" y1="1" x2="12" y2="23"></line><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"></path></svg> | `Dollar sign` | `306:921` | `feather:dollar-sign` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-download-cloud"><polyline points="8 17 12 21 16 17"></polyline><line x1="12" y1="12" x2="12" y2="21"></line><path d="M20.88 18.09A5 5 0 0 0 18 9h-1.26A8 8 0 1 0 3 16.29"></path></svg> | `Download cloud` | `306:920` | `feather:download-cloud` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-download"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="7 10 12 15 17 10"></polyline><line x1="12" y1="15" x2="12" y2="3"></line></svg> | `Download` | `306:919` | `feather:download` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-dribbble"><circle cx="12" cy="12" r="10"></circle><path d="M8.56 2.75c4.37 6.03 6.02 9.42 8.03 17.72m2.54-15.38c-3.72 4.35-8.94 5.66-16.88 5.85m19.5 1.9c-3.5-.93-6.63-.82-8.94 0-2.58.92-5.01 2.86-7.44 6.32"></path></svg> | `Dribbble` | `306:918` | `feather:dribbble` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-droplet"><path d="M12 2.69l5.66 5.66a8 8 0 1 1-11.31 0z"></path></svg> | `Droplet` | `306:917` | `feather:droplet` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-edit-2"><path d="M17 3a2.828 2.828 0 1 1 4 4L7.5 20.5 2 22l1.5-5.5L17 3z"></path></svg> | `Edit 2` | `306:916` | `feather:edit-2` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-edit-3"><path d="M12 20h9"></path><path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"></path></svg> | `Edit 3` | `306:915` | `feather:edit-3` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-edit"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path></svg> | `Edit` | `306:914` | `feather:edit` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg> | `External link` | `306:913` | `feather:external-link` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-eye-off"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"></path><line x1="1" y1="1" x2="23" y2="23"></line></svg> | `Eye off` | `306:912` | `feather:eye-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-eye"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg> | `Eye` | `306:902` | `feather:eye` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-facebook"><path d="M18 2h-3a5 5 0 0 0-5 5v3H7v4h3v8h4v-8h3l1-4h-4V7a1 1 0 0 1 1-1h3z"></path></svg> | `Facebook` | `306:903` | `feather:facebook` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-fast-forward"><polygon points="13 19 22 12 13 5 13 19"></polygon><polygon points="2 19 11 12 2 5 2 19"></polygon></svg> | `Fast forward` | `306:904` | `feather:fast-forward` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-feather"><path d="M20.24 12.24a6 6 0 0 0-8.49-8.49L5 10.5V19h8.5z"></path><line x1="16" y1="8" x2="2" y2="22"></line><line x1="17.5" y1="15" x2="9" y2="15"></line></svg> | `Feather` | `306:905` | `feather:feather` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-figma"><path d="M5 5.5A3.5 3.5 0 0 1 8.5 2H12v7H8.5A3.5 3.5 0 0 1 5 5.5z"></path><path d="M12 2h3.5a3.5 3.5 0 1 1 0 7H12V2z"></path><path d="M12 12.5a3.5 3.5 0 1 1 7 0 3.5 3.5 0 1 1-7 0z"></path><path d="M5 19.5A3.5 3.5 0 0 1 8.5 16H12v3.5a3.5 3.5 0 1 1-7 0z"></path><path d="M5 12.5A3.5 3.5 0 0 1 8.5 9H12v7H8.5A3.5 3.5 0 0 1 5 12.5z"></path></svg> | `Figma` | `306:906` | `feather:figma` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-minus"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="9" y1="15" x2="15" y2="15"></line></svg> | `File minus` | `306:907` | `feather:file-minus` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-plus"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="12" y1="18" x2="12" y2="12"></line><line x1="9" y1="15" x2="15" y2="15"></line></svg> | `File plus` | `306:908` | `feather:file-plus` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-text"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg> | `File text` | `306:909` | `feather:file-text` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file"><path d="M13 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9z"></path><polyline points="13 2 13 9 20 9"></polyline></svg> | `File` | `306:910` | `feather:file` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-film"><rect x="2" y="2" width="24" height="24" rx="2.18" ry="2.18"></rect><line x1="7" y1="2" x2="7" y2="22"></line><line x1="17" y1="2" x2="17" y2="22"></line><line x1="2" y1="12" x2="22" y2="12"></line><line x1="2" y1="7" x2="7" y2="7"></line><line x1="2" y1="17" x2="7" y2="17"></line><line x1="17" y1="17" x2="22" y2="17"></line><line x1="17" y1="7" x2="22" y2="7"></line></svg> | `Film` | `306:911` | `feather:film` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-filter"><polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"></polygon></svg> | `Filter` | `306:901` | `feather:filter` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-flag"><path d="M4 15s1-1 4-1 5 2 8 2 4-1 4-1V3s-1 1-4 1-5-2-8-2-4 1-4 1z"></path><line x1="4" y1="22" x2="4" y2="15"></line></svg> | `Flag` | `306:900` | `feather:flag` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-folder-minus"><path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"></path><line x1="9" y1="14" x2="15" y2="14"></line></svg> | `Folder minus` | `306:899` | `feather:folder-minus` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-folder-plus"><path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"></path><line x1="12" y1="11" x2="12" y2="17"></line><line x1="9" y1="14" x2="15" y2="14"></line></svg> | `Folder plus` | `306:898` | `feather:folder-plus` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-folder"><path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"></path></svg> | `Folder` | `306:897` | `feather:folder` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-framer"><path d="M5 16V9h14V2H5l14 14h-7m-7 0l7 7v-7m-7 0h7"></path></svg> | `Framer` | `306:896` | `feather:framer` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-frown"><circle cx="12" cy="12" r="10"></circle><path d="M16 16s-1.5-2-4-2-4 2-4 2"></path><line x1="9" y1="9" x2="9.01" y2="9"></line><line x1="15" y1="9" x2="15.01" y2="9"></line></svg> | `Frown` | `306:895` | `feather:frown` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-gift"><polyline points="20 12 20 22 4 22 4 12"></polyline><rect x="2" y="7" width="24" height="24"></rect><line x1="12" y1="22" x2="12" y2="7"></line><path d="M12 7H7.5a2.5 2.5 0 0 1 0-5C11 2 12 7 12 7z"></path><path d="M12 7h4.5a2.5 2.5 0 0 0 0-5C13 2 12 7 12 7z"></path></svg> | `Gift` | `306:894` | `feather:gift` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-git-branch"><line x1="6" y1="3" x2="6" y2="15"></line><circle cx="18" cy="6" r="3"></circle><circle cx="6" cy="18" r="3"></circle><path d="M18 9a9 9 0 0 1-9 9"></path></svg> | `Git branch` | `306:893` | `feather:git-branch` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-git-commit"><circle cx="12" cy="12" r="4"></circle><line x1="1.05" y1="12" x2="7" y2="12"></line><line x1="17.01" y1="12" x2="22.96" y2="12"></line></svg> | `Git commit` | `306:892` | `feather:git-commit` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-git-merge"><circle cx="18" cy="18" r="3"></circle><circle cx="6" cy="6" r="3"></circle><path d="M6 21V9a9 9 0 0 0 9 9"></path></svg> | `Git merge` | `306:882` | `feather:git-merge` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-git-pull-request"><circle cx="18" cy="18" r="3"></circle><circle cx="6" cy="6" r="3"></circle><path d="M13 6h3a2 2 0 0 1 2 2v7"></path><line x1="6" y1="9" x2="6" y2="21"></line></svg> | `Git pull-request` | `306:883` | `feather:git-pull-request` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-github"><path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path></svg> | `Github` | `306:884` | `feather:github` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-gitlab"><path d="M22.65 14.39L12 22.13 1.35 14.39a.84.84 0 0 1-.3-.94l1.22-3.78 2.44-7.51A.42.42 0 0 1 4.82 2a.43.43 0 0 1 .58 0 .42.42 0 0 1 .11.18l2.44 7.49h8.1l2.44-7.51A.42.42 0 0 1 18.6 2a.43.43 0 0 1 .58 0 .42.42 0 0 1 .11.18l2.44 7.51L23 13.45a.84.84 0 0 1-.35.94z"></path></svg> | `Gitlab` | `306:885` | `feather:gitlab` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-globe"><circle cx="12" cy="12" r="10"></circle><line x1="2" y1="12" x2="22" y2="12"></line><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"></path></svg> | `Globe` | `306:886` | `feather:globe` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-grid"><rect x="3" y="3" width="18" height="18"></rect><rect x="14" y="3" width="24" height="24"></rect><rect x="14" y="14" width="24" height="24"></rect><rect x="3" y="14" width="24" height="24"></rect></svg> | `Grid` | `306:887` | `feather:grid` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-hard-drive"><line x1="22" y1="12" x2="2" y2="12"></line><path d="M5.45 5.11L2 12v6a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-6l-3.45-6.89A2 2 0 0 0 16.76 4H7.24a2 2 0 0 0-1.79 1.11z"></path><line x1="6" y1="16" x2="6.01" y2="16"></line><line x1="10" y1="16" x2="10.01" y2="16"></line></svg> | `Hard drive` | `306:888` | `feather:hard-drive` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-hash"><line x1="4" y1="9" x2="20" y2="9"></line><line x1="4" y1="15" x2="20" y2="15"></line><line x1="10" y1="3" x2="8" y2="21"></line><line x1="16" y1="3" x2="14" y2="21"></line></svg> | `Hash` | `306:889` | `feather:hash` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-headphones"><path d="M3 18v-6a9 9 0 0 1 18 0v6"></path><path d="M21 19a2 2 0 0 1-2 2h-1a2 2 0 0 1-2-2v-3a2 2 0 0 1 2-2h3zM3 19a2 2 0 0 0 2 2h1a2 2 0 0 0 2-2v-3a2 2 0 0 0-2-2H3z"></path></svg> | `Headphones` | `306:890` | `feather:headphones` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-heart"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path></svg> | `Heart` | `306:891` | `feather:heart` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-help-circle"><circle cx="12" cy="12" r="10"></circle><path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path><line x1="12" y1="17" x2="12.01" y2="17"></line></svg> | `Help circle` | `306:880` | `feather:help-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-hexagon"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"></path></svg> | `Hexagon` | `306:881` | `feather:hexagon` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-home"><path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path><polyline points="9 22 9 12 15 12 15 22"></polyline></svg> | `Home` | `306:879` | `feather:home` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-image"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><circle cx="8.5" cy="8.5" r="1.5"></circle><polyline points="21 15 16 10 5 21"></polyline></svg> | `Image` | `306:878` | `feather:image` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-inbox"><polyline points="22 12 16 12 14 15 10 15 8 12 2 12"></polyline><path d="M5.45 5.11L2 12v6a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-6l-3.45-6.89A2 2 0 0 0 16.76 4H7.24a2 2 0 0 0-1.79 1.11z"></path></svg> | `Inbox` | `306:877` | `feather:inbox` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-info"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="16" x2="12" y2="12"></line><line x1="12" y1="8" x2="12.01" y2="8"></line></svg> | `Info` | `306:876` | `feather:info` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-instagram"><rect x="2" y="2" width="24" height="24" rx="5" ry="5"></rect><path d="M16 11.37A4 4 0 1 1 12.63 8 4 4 0 0 1 16 11.37z"></path><line x1="17.5" y1="6.5" x2="17.51" y2="6.5"></line></svg> | `Instagram` | `306:875` | `feather:instagram` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-italic"><line x1="19" y1="4" x2="10" y2="4"></line><line x1="14" y1="20" x2="5" y2="20"></line><line x1="15" y1="4" x2="9" y2="20"></line></svg> | `Italic` | `306:874` | `feather:italic` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-key"><path d="M21 2l-2 2m-7.61 7.61a5.5 5.5 0 1 1-7.778 7.778 5.5 5.5 0 0 1 7.777-7.777zm0 0L15.5 7.5m0 0l3 3L22 7l-3-3m-3.5 3.5L19 4"></path></svg> | `Key` | `306:873` | `feather:key` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-layers"><polygon points="12 2 2 7 12 12 22 7 12 2"></polygon><polyline points="2 17 12 22 22 17"></polyline><polyline points="2 12 12 17 22 12"></polyline></svg> | `Layers` | `306:872` | `feather:layers` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-layout"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><line x1="3" y1="9" x2="21" y2="9"></line><line x1="9" y1="21" x2="9" y2="9"></line></svg> | `Layout` | `306:862` | `feather:layout` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-life-buoy"><circle cx="12" cy="12" r="10"></circle><circle cx="12" cy="12" r="4"></circle><line x1="4.93" y1="4.93" x2="9.17" y2="9.17"></line><line x1="14.83" y1="14.83" x2="19.07" y2="19.07"></line><line x1="14.83" y1="9.17" x2="19.07" y2="4.93"></line><line x1="14.83" y1="9.17" x2="18.36" y2="5.64"></line><line x1="4.93" y1="19.07" x2="9.17" y2="14.83"></line></svg> | `Life buoy` | `306:863` | `feather:life-buoy` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-link-2"><path d="M15 7h3a5 5 0 0 1 5 5 5 5 0 0 1-5 5h-3m-6 0H6a5 5 0 0 1-5-5 5 5 0 0 1 5-5h3"></path><line x1="8" y1="12" x2="16" y2="12"></line></svg> | `Link 2` | `306:864` | `feather:link-2` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg> | `Link` | `306:865` | `feather:link` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-linkedin"><path d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"></path><rect x="2" y="9" width="24" height="24"></rect><circle cx="4" cy="4" r="2"></circle></svg> | `Linkedin` | `306:866` | `feather:linkedin` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-list"><line x1="8" y1="6" x2="21" y2="6"></line><line x1="8" y1="12" x2="21" y2="12"></line><line x1="8" y1="18" x2="21" y2="18"></line><line x1="3" y1="6" x2="3.01" y2="6"></line><line x1="3" y1="12" x2="3.01" y2="12"></line><line x1="3" y1="18" x2="3.01" y2="18"></line></svg> | `List` | `306:867` | `feather:list` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-loader"><line x1="12" y1="2" x2="12" y2="6"></line><line x1="12" y1="18" x2="12" y2="22"></line><line x1="4.93" y1="4.93" x2="7.76" y2="7.76"></line><line x1="16.24" y1="16.24" x2="19.07" y2="19.07"></line><line x1="2" y1="12" x2="6" y2="12"></line><line x1="18" y1="12" x2="22" y2="12"></line><line x1="4.93" y1="19.07" x2="7.76" y2="16.24"></line><line x1="16.24" y1="7.76" x2="19.07" y2="4.93"></line></svg> | `Loader` | `306:868` | `feather:loader` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-lock"><rect x="3" y="11" width="24" height="24" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg> | `Lock` | `306:869` | `feather:lock` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-log-in"><path d="M15 3h4a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2h-4"></path><polyline points="10 17 15 12 10 7"></polyline><line x1="15" y1="12" x2="3" y2="12"></line></svg> | `Log in` | `306:870` | `feather:log-in` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-log-out"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path><polyline points="16 17 21 12 16 7"></polyline><line x1="21" y1="12" x2="9" y2="12"></line></svg> | `Log out` | `306:871` | `feather:log-out` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-mail"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"></path><polyline points="22,6 12,13 2,6"></polyline></svg> | `Mail` | `306:861` | `feather:mail` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-map-pin"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"></path><circle cx="12" cy="10" r="3"></circle></svg> | `Map pin` | `306:860` | `feather:map-pin` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-map"><polygon points="1 6 1 22 8 18 16 22 23 18 23 2 16 6 8 2 1 6"></polygon><line x1="8" y1="2" x2="8" y2="18"></line><line x1="16" y1="6" x2="16" y2="22"></line></svg> | `Map` | `306:859` | `feather:map` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-maximize-2"><polyline points="15 3 21 3 21 9"></polyline><polyline points="9 21 3 21 3 15"></polyline><line x1="21" y1="3" x2="14" y2="10"></line><line x1="3" y1="21" x2="10" y2="14"></line></svg> | `Maximize 2` | `306:858` | `feather:maximize-2` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-maximize"><path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"></path></svg> | `Maximize` | `306:857` | `feather:maximize` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-meh"><circle cx="12" cy="12" r="10"></circle><line x1="8" y1="15" x2="16" y2="15"></line><line x1="9" y1="9" x2="9.01" y2="9"></line><line x1="15" y1="9" x2="15.01" y2="9"></line></svg> | `Meh` | `306:856` | `feather:meh` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-menu"><line x1="3" y1="12" x2="21" y2="12"></line><line x1="3" y1="6" x2="21" y2="6"></line><line x1="3" y1="18" x2="21" y2="18"></line></svg> | `Menu` | `306:855` | `feather:menu` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-message-circle"><path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8 8.5 8.5 0 0 1-7.6 4.7 8.38 8.38 0 0 1-3.8-.9L3 21l1.9-5.7a8.38 8.38 0 0 1-.9-3.8 8.5 8.5 0 0 1 4.7-7.6 8.38 8.38 0 0 1 3.8-.9h.5a8.48 8.48 0 0 1 8 8v.5z"></path></svg> | `Message circle` | `306:854` | `feather:message-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-message-square"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"></path></svg> | `Message square` | `306:853` | `feather:message-square` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-mic-off"><line x1="1" y1="1" x2="23" y2="23"></line><path d="M9 9v3a3 3 0 0 0 5.12 2.12M15 9.34V4a3 3 0 0 0-5.94-.6"></path><path d="M17 16.95A7 7 0 0 1 5 12v-2m14 0v2a7 7 0 0 1-.11 1.23"></path><line x1="12" y1="19" x2="12" y2="23"></line><line x1="8" y1="23" x2="16" y2="23"></line></svg> | `Mic off` | `306:852` | `feather:mic-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-mic"><path d="M12 1a3 3 0 0 0-3 3v8a3 3 0 0 0 6 0V4a3 3 0 0 0-3-3z"></path><path d="M19 10v2a7 7 0 0 1-14 0v-2"></path><line x1="12" y1="19" x2="12" y2="23"></line><line x1="8" y1="23" x2="16" y2="23"></line></svg> | `Mic` | `306:842` | `feather:mic` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-minimize-2"><polyline points="4 14 10 14 10 20"></polyline><polyline points="20 10 14 10 14 4"></polyline><line x1="14" y1="10" x2="21" y2="3"></line><line x1="3" y1="21" x2="10" y2="14"></line></svg> | `Minimize 2` | `306:843` | `feather:minimize-2` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-minimize"><path d="M8 3v3a2 2 0 0 1-2 2H3m18 0h-3a2 2 0 0 1-2-2V3m0 18v-3a2 2 0 0 1 2-2h3M3 16h3a2 2 0 0 1 2 2v3"></path></svg> | `Minimize` | `306:844` | `feather:minimize` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-minus-circle"><circle cx="12" cy="12" r="10"></circle><line x1="8" y1="12" x2="16" y2="12"></line></svg> | `Minus circle` | `306:845` | `feather:minus-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-minus-square"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><line x1="8" y1="12" x2="16" y2="12"></line></svg> | `Minus square` | `306:846` | `feather:minus-square` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-minus"><line x1="5" y1="12" x2="19" y2="12"></line></svg> | `Minus` | `306:847` | `feather:minus` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-monitor"><rect x="2" y="3" width="24" height="24" rx="2" ry="2"></rect><line x1="8" y1="21" x2="16" y2="21"></line><line x1="12" y1="17" x2="12" y2="21"></line></svg> | `Monitor` | `306:848` | `feather:monitor` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-moon"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg> | `Moon` | `306:849` | `feather:moon` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-more-horizontal"><circle cx="12" cy="12" r="1"></circle><circle cx="19" cy="12" r="1"></circle><circle cx="5" cy="12" r="1"></circle></svg> | `More horizontal` | `306:850` | `feather:more-horizontal` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-more-vertical"><circle cx="12" cy="12" r="1"></circle><circle cx="12" cy="5" r="1"></circle><circle cx="12" cy="19" r="1"></circle></svg> | `More vertical` | `306:851` | `feather:more-vertical` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-mouse-pointer"><path d="M3 3l7.07 16.97 2.51-7.39 7.39-2.51L3 3z"></path><path d="M13 13l6 6"></path></svg> | `Mouse pointer` | `306:841` | `feather:mouse-pointer` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-move"><polyline points="5 9 2 12 5 15"></polyline><polyline points="9 5 12 2 15 5"></polyline><polyline points="15 19 12 22 9 19"></polyline><polyline points="19 9 22 12 19 15"></polyline><line x1="2" y1="12" x2="22" y2="12"></line><line x1="12" y1="2" x2="12" y2="22"></line></svg> | `Move` | `306:840` | `feather:move` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-music"><path d="M9 18V5l12-2v13"></path><circle cx="6" cy="18" r="3"></circle><circle cx="18" cy="16" r="3"></circle></svg> | `Music` | `306:839` | `feather:music` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-navigation-2"><polygon points="12 2 19 21 12 17 5 21 12 2"></polygon></svg> | `Navigation 2` | `306:838` | `feather:navigation-2` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-navigation"><polygon points="3 11 22 2 13 21 11 13 3 11"></polygon></svg> | `Navigation` | `306:837` | `feather:navigation` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-octagon"><polygon points="7.86 2 16.14 2 22 7.86 22 16.14 16.14 22 7.86 22 2 16.14 2 7.86 7.86 2"></polygon></svg> | `Octagon` | `306:836` | `feather:octagon` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-package"><line x1="16.5" y1="9.4" x2="7.5" y2="4.21"></line><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"></path><polyline points="3.27 6.96 12 12.01 20.73 6.96"></polyline><line x1="12" y1="22.08" x2="12" y2="12"></line></svg> | `Package` | `306:835` | `feather:package` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-paperclip"><path d="M21.44 11.05l-9.19 9.19a6 6 0 0 1-8.49-8.49l9.19-9.19a4 4 0 0 1 5.66 5.66l-9.2 9.19a2 2 0 0 1-2.83-2.83l8.49-8.48"></path></svg> | `Paperclip` | `306:834` | `feather:paperclip` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-pause-circle"><circle cx="12" cy="12" r="10"></circle><line x1="10" y1="15" x2="10" y2="9"></line><line x1="14" y1="15" x2="14" y2="9"></line></svg> | `Pause circle` | `306:833` | `feather:pause-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-pause"><rect x="6" y="4" width="24" height="24"></rect><rect x="14" y="4" width="24" height="24"></rect></svg> | `Pause` | `306:832` | `feather:pause` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-pen-tool"><path d="M12 19l7-7 3 3-7 7-3-3z"></path><path d="M18 13l-1.5-7.5L2 2l3.5 14.5L13 18l5-5z"></path><path d="M2 2l7.586 7.586"></path><circle cx="11" cy="11" r="2"></circle></svg> | `Pen tool` | `306:822` | `feather:pen-tool` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-percent"><line x1="19" y1="5" x2="5" y2="19"></line><circle cx="6.5" cy="6.5" r="2.5"></circle><circle cx="17.5" cy="17.5" r="2.5"></circle></svg> | `Percent` | `306:823` | `feather:percent` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-phone-call"><path d="M15.05 5A5 5 0 0 1 19 8.95M15.05 1A9 9 0 0 1 23 8.94m-1 7.98v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"></path></svg> | `Phone call` | `306:824` | `feather:phone-call` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-phone-forwarded"><polyline points="19 1 23 5 19 9"></polyline><line x1="15" y1="5" x2="23" y2="5"></line><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"></path></svg> | `Phone forwarded` | `306:825` | `feather:phone-forwarded` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-phone-incoming"><polyline points="16 2 16 8 22 8"></polyline><line x1="23" y1="1" x2="16" y2="8"></line><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"></path></svg> | `Phone incoming` | `306:826` | `feather:phone-incoming` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-phone-missed"><line x1="23" y1="1" x2="17" y2="7"></line><line x1="17" y1="1" x2="23" y2="7"></line><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"></path></svg> | `Phone missed` | `306:827` | `feather:phone-missed` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-phone-off"><path d="M10.68 13.31a16 16 0 0 0 3.41 2.6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7 2 2 0 0 1 1.72 2v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.42 19.42 0 0 1-3.33-2.67m-2.67-3.34a19.79 19.79 0 0 1-3.07-8.63A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91"></path><line x1="23" y1="1" x2="1" y2="23"></line></svg> | `Phone off` | `306:828` | `feather:phone-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-phone-outgoing"><polyline points="23 7 23 1 17 1"></polyline><line x1="16" y1="8" x2="23" y2="1"></line><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"></path></svg> | `Phone outgoing` | `306:829` | `feather:phone-outgoing` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-phone"><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"></path></svg> | `Phone` | `306:830` | `feather:phone` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-pie-chart"><path d="M21.21 15.89A10 10 0 1 1 8 2.83"></path><path d="M22 12A10 10 0 0 0 12 2v10z"></path></svg> | `Pie chart` | `306:831` | `feather:pie-chart` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-play-circle"><circle cx="12" cy="12" r="10"></circle><polygon points="10 8 16 12 10 16 10 8"></polygon></svg> | `Play circle` | `306:821` | `feather:play-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-play"><polygon points="5 3 19 12 5 21 5 3"></polygon></svg> | `Play` | `306:820` | `feather:play` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-plus-circle"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="8" x2="12" y2="16"></line><line x1="8" y1="12" x2="16" y2="12"></line></svg> | `Plus circle` | `306:819` | `feather:plus-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-plus-square"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><line x1="12" y1="8" x2="12" y2="16"></line><line x1="8" y1="12" x2="16" y2="12"></line></svg> | `Plus square` | `306:818` | `feather:plus-square` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-plus"><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg> | `Plus` | `306:817` | `feather:plus` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-pocket"><path d="M4 3h16a2 2 0 0 1 2 2v6a10 10 0 0 1-10 10A10 10 0 0 1 2 11V5a2 2 0 0 1 2-2z"></path><polyline points="8 10 12 14 16 10"></polyline></svg> | `Pocket` | `306:816` | `feather:pocket` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-power"><path d="M18.36 6.64a9 9 0 1 1-12.73 0"></path><line x1="12" y1="2" x2="12" y2="12"></line></svg> | `Power` | `306:815` | `feather:power` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-printer"><polyline points="6 9 6 2 18 2 18 9"></polyline><path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"></path><rect x="6" y="14" width="24" height="24"></rect></svg> | `Printer` | `306:814` | `feather:printer` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-radio"><circle cx="12" cy="12" r="2"></circle><path d="M16.24 7.76a6 6 0 0 1 0 8.49m-8.48-.01a6 6 0 0 1 0-8.49m11.31-2.82a10 10 0 0 1 0 14.14m-14.14 0a10 10 0 0 1 0-14.14"></path></svg> | `Radio` | `306:813` | `feather:radio` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-refresh-ccw"><polyline points="1 4 1 10 7 10"></polyline><polyline points="23 20 23 14 17 14"></polyline><path d="M20.49 9A9 9 0 0 0 5.64 5.64L1 10m22 4l-4.64 4.36A9 9 0 0 1 3.51 15"></path></svg> | `Refresh ccw` | `306:812` | `feather:refresh-ccw` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-refresh-cw"><polyline points="23 4 23 10 17 10"></polyline><polyline points="1 20 1 14 7 14"></polyline><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"></path></svg> | `Refresh cw` | `306:802` | `feather:refresh-cw` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-repeat"><polyline points="17 1 21 5 17 9"></polyline><path d="M3 11V9a4 4 0 0 1 4-4h14"></path><polyline points="7 23 3 19 7 15"></polyline><path d="M21 13v2a4 4 0 0 1-4 4H3"></path></svg> | `Repeat` | `306:803` | `feather:repeat` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-rewind"><polygon points="11 19 2 12 11 5 11 19"></polygon><polygon points="22 19 13 12 22 5 22 19"></polygon></svg> | `Rewind` | `306:804` | `feather:rewind` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-rotate-ccw"><polyline points="1 4 1 10 7 10"></polyline><path d="M3.51 15a9 9 0 1 0 2.13-9.36L1 10"></path></svg> | `Rotate ccw` | `306:805` | `feather:rotate-ccw` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-rotate-cw"><polyline points="23 4 23 10 17 10"></polyline><path d="M20.49 15a9 9 0 1 1-2.12-9.36L23 10"></path></svg> | `Rotate cw` | `306:806` | `feather:rotate-cw` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-rss"><path d="M4 11a9 9 0 0 1 9 9"></path><path d="M4 4a16 16 0 0 1 16 16"></path><circle cx="5" cy="19" r="1"></circle></svg> | `Rss` | `306:807` | `feather:rss` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-save"><path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"></path><polyline points="17 21 17 13 7 13 7 21"></polyline><polyline points="7 3 7 8 15 8"></polyline></svg> | `Save` | `306:808` | `feather:save` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-scissors"><circle cx="6" cy="6" r="3"></circle><circle cx="6" cy="18" r="3"></circle><line x1="20" y1="4" x2="8.12" y2="15.88"></line><line x1="14.47" y1="14.48" x2="20" y2="20"></line><line x1="8.12" y1="8.12" x2="12" y2="12"></line></svg> | `Scissors` | `306:809` | `feather:scissors` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-search"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg> | `Search` | `306:810` | `feather:search` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-send"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg> | `Send` | `306:811` | `feather:send` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-server"><rect x="2" y="2" width="24" height="24" rx="2" ry="2"></rect><rect x="2" y="14" width="24" height="24" rx="2" ry="2"></rect><line x1="6" y1="6" x2="6.01" y2="6"></line><line x1="6" y1="18" x2="6.01" y2="18"></line></svg> | `Server` | `295:663` | `feather:server` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-settings"><circle cx="12" cy="12" r="3"></circle><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path></svg> | `Settings` | `295:664` | `feather:settings` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-share-2"><circle cx="18" cy="5" r="3"></circle><circle cx="6" cy="12" r="3"></circle><circle cx="18" cy="19" r="3"></circle><line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line><line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line></svg> | `Share 2` | `295:665` | `feather:share-2` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-share"><path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"></path><polyline points="16 6 12 2 8 6"></polyline><line x1="12" y1="2" x2="12" y2="15"></line></svg> | `Share` | `295:666` | `feather:share` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-shield-off"><path d="M19.69 14a6.9 6.9 0 0 0 .31-2V5l-8-3-3.16 1.18"></path><path d="M4.73 4.73L4 5v7c0 6 8 10 8 10a20.29 20.29 0 0 0 5.62-4.38"></path><line x1="1" y1="1" x2="23" y2="23"></line></svg> | `Shield off` | `295:667` | `feather:shield-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-shield"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"></path></svg> | `Shield` | `295:668` | `feather:shield` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-shopping-bag"><path d="M6 2L3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z"></path><line x1="3" y1="6" x2="21" y2="6"></line><path d="M16 10a4 4 0 0 1-8 0"></path></svg> | `Shopping bag` | `295:669` | `feather:shopping-bag` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-shopping-cart"><circle cx="9" cy="21" r="1"></circle><circle cx="20" cy="21" r="1"></circle><path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"></path></svg> | `Shopping cart` | `295:670` | `feather:shopping-cart` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-shuffle"><polyline points="16 3 21 3 21 8"></polyline><line x1="4" y1="20" x2="21" y2="3"></line><polyline points="21 16 21 21 16 21"></polyline><line x1="15" y1="15" x2="21" y2="21"></line><line x1="4" y1="4" x2="9" y2="9"></line></svg> | `Shuffle` | `295:671` | `feather:shuffle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-sidebar"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><line x1="9" y1="3" x2="9" y2="21"></line></svg> | `Sidebar` | `295:672` | `feather:sidebar` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-skip-back"><polygon points="19 20 9 12 19 4 19 20"></polygon><line x1="5" y1="19" x2="5" y2="5"></line></svg> | `Skip back` | `295:662` | `feather:skip-back` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-skip-forward"><polygon points="5 4 15 12 5 20 5 4"></polygon><line x1="19" y1="5" x2="19" y2="19"></line></svg> | `Skip forward` | `295:661` | `feather:skip-forward` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-slack"><path d="M14.5 10c-.83 0-1.5-.67-1.5-1.5v-5c0-.83.67-1.5 1.5-1.5s1.5.67 1.5 1.5v5c0 .83-.67 1.5-1.5 1.5z"></path><path d="M20.5 10H19V8.5c0-.83.67-1.5 1.5-1.5s1.5.67 1.5 1.5-.67 1.5-1.5 1.5z"></path><path d="M9.5 14c.83 0 1.5.67 1.5 1.5v5c0 .83-.67 1.5-1.5 1.5S8 21.33 8 20.5v-5c0-.83.67-1.5 1.5-1.5z"></path><path d="M3.5 14H5v1.5c0 .83-.67 1.5-1.5 1.5S2 16.33 2 15.5 2.67 14 3.5 14z"></path><path d="M14 14.5c0-.83.67-1.5 1.5-1.5h5c.83 0 1.5.67 1.5 1.5s-.67 1.5-1.5 1.5h-5c-.83 0-1.5-.67-1.5-1.5z"></path><path d="M15.5 19H14v1.5c0 .83.67 1.5 1.5 1.5s1.5-.67 1.5-1.5-.67-1.5-1.5-1.5z"></path><path d="M10 9.5C10 8.67 9.33 8 8.5 8h-5C2.67 8 2 8.67 2 9.5S2.67 11 3.5 11h5c.83 0 1.5-.67 1.5-1.5z"></path><path d="M8.5 5H10V3.5C10 2.67 9.33 2 8.5 2S7 2.67 7 3.5 7.67 5 8.5 5z"></path></svg> | `Slack` | `295:660` | `feather:slack` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-slash"><circle cx="12" cy="12" r="10"></circle><line x1="4.93" y1="4.93" x2="19.07" y2="19.07"></line></svg> | `Slash` | `295:659` | `feather:slash` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-sliders"><line x1="4" y1="21" x2="4" y2="14"></line><line x1="4" y1="10" x2="4" y2="3"></line><line x1="12" y1="21" x2="12" y2="12"></line><line x1="12" y1="8" x2="12" y2="3"></line><line x1="20" y1="21" x2="20" y2="16"></line><line x1="20" y1="12" x2="20" y2="3"></line><line x1="1" y1="14" x2="7" y2="14"></line><line x1="9" y1="8" x2="15" y2="8"></line><line x1="17" y1="16" x2="23" y2="16"></line></svg> | `Sliders` | `295:658` | `feather:sliders` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-smartphone"><rect x="5" y="2" width="24" height="24" rx="2" ry="2"></rect><line x1="12" y1="18" x2="12.01" y2="18"></line></svg> | `Smartphone` | `295:657` | `feather:smartphone` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-smile"><circle cx="12" cy="12" r="10"></circle><path d="M8 14s1.5 2 4 2 4-2 4-2"></path><line x1="9" y1="9" x2="9.01" y2="9"></line><line x1="15" y1="9" x2="15.01" y2="9"></line></svg> | `Smile` | `295:656` | `feather:smile` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-speaker"><rect x="4" y="2" width="24" height="24" rx="2" ry="2"></rect><circle cx="12" cy="14" r="4"></circle><line x1="12" y1="6" x2="12.01" y2="6"></line></svg> | `Speaker` | `295:655` | `feather:speaker` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-square"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect></svg> | `Square` | `295:654` | `feather:square` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-star"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon></svg> | `Star` | `295:653` | `feather:star` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-stop-circle"><circle cx="12" cy="12" r="10"></circle><rect x="9" y="9" width="24" height="24"></rect></svg> | `Stop circle` | `295:643` | `feather:stop-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-sun"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg> | `Sun` | `295:644` | `feather:sun` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-sunrise"><path d="M17 18a5 5 0 0 0-10 0"></path><line x1="12" y1="2" x2="12" y2="9"></line><line x1="4.22" y1="10.22" x2="5.64" y2="11.64"></line><line x1="1" y1="18" x2="3" y2="18"></line><line x1="21" y1="18" x2="23" y2="18"></line><line x1="18.36" y1="11.64" x2="19.78" y2="10.22"></line><line x1="23" y1="22" x2="1" y2="22"></line><polyline points="8 6 12 2 16 6"></polyline></svg> | `Sunrise` | `295:645` | `feather:sunrise` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-sunset"><path d="M17 18a5 5 0 0 0-10 0"></path><line x1="12" y1="9" x2="12" y2="2"></line><line x1="4.22" y1="10.22" x2="5.64" y2="11.64"></line><line x1="1" y1="18" x2="3" y2="18"></line><line x1="21" y1="18" x2="23" y2="18"></line><line x1="18.36" y1="11.64" x2="19.78" y2="10.22"></line><line x1="23" y1="22" x2="1" y2="22"></line><polyline points="16 5 12 9 8 5"></polyline></svg> | `Sunset` | `295:646` | `feather:sunset` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-table"><path d="M9 3H5a2 2 0 0 0-2 2v4m6-6h10a2 2 0 0 1 2 2v4M9 3v18m0 0h10a2 2 0 0 0 2-2V9M9 21H5a2 2 0 0 1-2-2V9m0 0h18"></path></svg> | `Table` | `295:647` | `feather:table` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-tablet"><rect x="4" y="2" width="24" height="24" rx="2" ry="2"></rect><line x1="12" y1="18" x2="12.01" y2="18"></line></svg> | `Tablet` | `295:648` | `feather:tablet` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-tag"><path d="M20.59 13.41l-7.17 7.17a2 2 0 0 1-2.83 0L2 12V2h10l8.59 8.59a2 2 0 0 1 0 2.82z"></path><line x1="7" y1="7" x2="7.01" y2="7"></line></svg> | `Tag` | `295:649` | `feather:tag` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-target"><circle cx="12" cy="12" r="10"></circle><circle cx="12" cy="12" r="6"></circle><circle cx="12" cy="12" r="2"></circle></svg> | `Target` | `295:650` | `feather:target` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-terminal"><polyline points="4 17 10 11 4 5"></polyline><line x1="12" y1="19" x2="20" y2="19"></line></svg> | `Terminal` | `295:651` | `feather:terminal` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-thermometer"><path d="M14 14.76V3.5a2.5 2.5 0 0 0-5 0v11.26a4.5 4.5 0 1 0 5 0z"></path></svg> | `Thermometer` | `295:652` | `feather:thermometer` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-thumbs-down"><path d="M10 15v4a3 3 0 0 0 3 3l4-9V2H5.72a2 2 0 0 0-2 1.7l-1.38 9a2 2 0 0 0 2 2.3zm7-13h2.67A2.31 2.31 0 0 1 22 4v7a2.31 2.31 0 0 1-2.33 2H17"></path></svg> | `Thumbs down` | `295:642` | `feather:thumbs-down` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-thumbs-up"><path d="M14 9V5a3 3 0 0 0-3-3l-4 9v11h11.28a2 2 0 0 0 2-1.7l1.38-9a2 2 0 0 0-2-2.3zM7 22H4a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h3"></path></svg> | `Thumbs up` | `295:641` | `feather:thumbs-up` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-toggle-left"><rect x="1" y="5" width="24" height="24" rx="7" ry="7"></rect><circle cx="8" cy="12" r="3"></circle></svg> | `Toggle left` | `295:640` | `feather:toggle-left` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-toggle-right"><rect x="1" y="5" width="24" height="24" rx="7" ry="7"></rect><circle cx="16" cy="12" r="3"></circle></svg> | `Toggle right` | `295:639` | `feather:toggle-right` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-tool"><path d="M14.7 6.3a1 1 0 0 0 0 1.4l1.6 1.6a1 1 0 0 0 1.4 0l3.77-3.77a6 6 0 0 1-7.94 7.94l-6.91 6.91a2.12 2.12 0 0 1-3-3l6.91-6.91a6 6 0 0 1 7.94-7.94l-3.76 3.76z"></path></svg> | `Tool` | `295:638` | `feather:tool` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-trash-2"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path><line x1="10" y1="11" x2="10" y2="17"></line><line x1="14" y1="11" x2="14" y2="17"></line></svg> | `Trash 2` | `295:637` | `feather:trash-2` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-trash"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg> | `Trash` | `295:636` | `feather:trash` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-trello"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><rect x="7" y="7" width="24" height="24"></rect><rect x="14" y="7" width="24" height="24"></rect></svg> | `Trello` | `295:635` | `feather:trello` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-trending-down"><polyline points="23 18 13.5 8.5 8.5 13.5 1 6"></polyline><polyline points="17 18 23 18 23 12"></polyline></svg> | `Trending down` | `295:634` | `feather:trending-down` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-trending-up"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"></polyline><polyline points="17 6 23 6 23 12"></polyline></svg> | `Trending up` | `295:633` | `feather:trending-up` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-triangle"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path></svg> | `Triangle` | `295:623` | `feather:triangle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-truck"><rect x="1" y="3" width="24" height="24"></rect><polygon points="16 8 20 8 23 11 23 16 16 16 16 8"></polygon><circle cx="5.5" cy="18.5" r="2.5"></circle><circle cx="18.5" cy="18.5" r="2.5"></circle></svg> | `Truck` | `295:624` | `feather:truck` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-tv"><rect x="2" y="7" width="24" height="24" rx="2" ry="2"></rect><polyline points="17 2 12 7 7 2"></polyline></svg> | `Tv` | `295:625` | `feather:tv` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-twitch"><path d="M21 2H3v16h5v4l4-4h5l4-4V2zm-10 9V7m5 4V7"></path></svg> | `Twitch` | `295:626` | `feather:twitch` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-twitter"><path d="M23 3a10.9 10.9 0 0 1-3.14 1.53 4.48 4.48 0 0 0-7.86 3v1A10.66 10.66 0 0 1 3 4s-4 9 5 13a11.64 11.64 0 0 1-7 2c9 5 20 0 20-11.5a4.5 4.5 0 0 0-.08-.83A7.72 7.72 0 0 0 23 3z"></path></svg> | `Twitter` | `295:627` | `feather:twitter` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-type"><polyline points="4 7 4 4 20 4 20 7"></polyline><line x1="9" y1="20" x2="15" y2="20"></line><line x1="12" y1="4" x2="12" y2="20"></line></svg> | `Type` | `295:628` | `feather:type` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-umbrella"><path d="M23 12a11.05 11.05 0 0 0-22 0zm-5 7a3 3 0 0 1-6 0v-7"></path></svg> | `Umbrella` | `295:629` | `feather:umbrella` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-underline"><path d="M6 3v7a6 6 0 0 0 6 6 6 6 0 0 0 6-6V3"></path><line x1="4" y1="21" x2="20" y2="21"></line></svg> | `Underline` | `295:630` | `feather:underline` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-unlock"><rect x="3" y="11" width="24" height="24" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 9.9-1"></path></svg> | `Unlock` | `295:631` | `feather:unlock` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-upload-cloud"><polyline points="16 16 12 12 8 16"></polyline><line x1="12" y1="12" x2="12" y2="21"></line><path d="M20.39 18.39A5 5 0 0 0 18 9h-1.26A8 8 0 1 0 3 16.3"></path><polyline points="16 16 12 12 8 16"></polyline></svg> | `Upload cloud` | `295:632` | `feather:upload-cloud` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-upload"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg> | `Upload` | `295:437` | `feather:upload` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-user-check"><path d="M16 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="8.5" cy="7" r="4"></circle><polyline points="17 11 19 13 23 9"></polyline></svg> | `User check` | `295:436` | `feather:user-check` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-user-minus"><path d="M16 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="8.5" cy="7" r="4"></circle><line x1="23" y1="11" x2="17" y2="11"></line></svg> | `User minus` | `295:435` | `feather:user-minus` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-user-plus"><path d="M16 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="8.5" cy="7" r="4"></circle><line x1="20" y1="8" x2="20" y2="14"></line><line x1="23" y1="11" x2="17" y2="11"></line></svg> | `User plus` | `295:434` | `feather:user-plus` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-user-x"><path d="M16 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="8.5" cy="7" r="4"></circle><line x1="18" y1="8" x2="23" y2="13"></line><line x1="23" y1="8" x2="18" y2="13"></line></svg> | `User x` | `295:433` | `feather:user-x` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-user"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg> | `User` | `295:432` | `feather:user` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-users"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="9" cy="7" r="4"></circle><path d="M23 21v-2a4 4 0 0 0-3-3.87"></path><path d="M16 3.13a4 4 0 0 1 0 7.75"></path></svg> | `Users` | `295:431` | `feather:users` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-video-off"><path d="M16 16v1a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V7a2 2 0 0 1 2-2h2m5.66 0H14a2 2 0 0 1 2 2v3.34l1 1L23 7v10"></path><line x1="1" y1="1" x2="23" y2="23"></line></svg> | `Video off` | `295:430` | `feather:video-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"><path d="M15 7C15 6.44772 14.5523 6 14 6H3C2.44772 6 2 6.44772 2 7V17C2 17.5523 2.44772 18 3 18H14C14.5523 18 15 17.5523 15 17V7ZM17.7197 12L22 15.0566V8.94238L17.7197 12ZM17 10.0566L22.4189 6.18652C22.7238 5.9688 23.1249 5.93895 23.458 6.11035C23.791 6.28178 24 6.62547 24 7V17C24 17.3745 23.791 17.7182 23.458 17.8896C23.1249 18.0611 22.7238 18.0312 22.4189 17.8135L17 13.9424V17C17 18.6569 15.6569 20 14 20H3C1.34315 20 0 18.6569 0 17V7C0 5.34315 1.34315 4 3 4H14C15.6569 4 17 5.34315 17 7V10.0566Z" fill="currentColor" stroke="none"/></svg> | `Video` | `295:429` | `feather:video` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-voicemail"><circle cx="5.5" cy="11.5" r="4.5"></circle><circle cx="18.5" cy="11.5" r="4.5"></circle><line x1="5.5" y1="16" x2="18.5" y2="16"></line></svg> | `Voicemail` | `295:428` | `feather:voicemail` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-volume-1"><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"></polygon><path d="M15.54 8.46a5 5 0 0 1 0 7.07"></path></svg> | `Volume 1` | `295:418` | `feather:volume-1` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-volume-2"><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"></polygon><path d="M19.07 4.93a10 10 0 0 1 0 14.14M15.54 8.46a5 5 0 0 1 0 7.07"></path></svg> | `Volume 2` | `295:419` | `feather:volume-2` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-volume-x"><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"></polygon><line x1="23" y1="9" x2="17" y2="15"></line><line x1="17" y1="9" x2="23" y2="15"></line></svg> | `Volume x` | `295:420` | `feather:volume-x` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-volume"><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"></polygon></svg> | `Volume` | `295:421` | `feather:volume` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-watch"><circle cx="12" cy="12" r="7"></circle><polyline points="12 9 12 12 13.5 13.5"></polyline><path d="M16.51 17.35l-.35 3.83a2 2 0 0 1-2 1.82H9.83a2 2 0 0 1-2-1.82l-.35-3.83m.01-10.7l.35-3.83A2 2 0 0 1 9.83 1h4.35a2 2 0 0 1 2 1.82l.35 3.83"></path></svg> | `Watch` | `295:422` | `feather:watch` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-wifi-off"><line x1="1" y1="1" x2="23" y2="23"></line><path d="M16.72 11.06A10.94 10.94 0 0 1 19 12.55"></path><path d="M5 12.55a10.94 10.94 0 0 1 5.17-2.39"></path><path d="M10.71 5.05A16 16 0 0 1 22.58 9"></path><path d="M1.42 9a15.91 15.91 0 0 1 4.7-2.88"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line></svg> | `Wifi off` | `295:423` | `feather:wifi-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-wifi"><path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line></svg> | `Wifi` | `295:424` | `feather:wifi` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-wind"><path d="M9.59 4.59A2 2 0 1 1 11 8H2m10.59 11.41A2 2 0 1 0 14 16H2m15.73-8.27A2.5 2.5 0 1 1 19.5 12H2"></path></svg> | `Wind` | `295:425` | `feather:wind` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x-circle"><circle cx="12" cy="12" r="10"></circle><line x1="15" y1="9" x2="9" y2="15"></line><line x1="9" y1="9" x2="15" y2="15"></line></svg> | `X circle` | `295:426` | `feather:x-circle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x-octagon"><polygon points="7.86 2 16.14 2 22 7.86 22 16.14 16.14 22 7.86 22 2 16.14 2 7.86 7.86 2"></polygon><line x1="15" y1="9" x2="9" y2="15"></line><line x1="9" y1="9" x2="15" y2="15"></line></svg> | `X octagon` | `295:427` | `feather:x-octagon` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg> | `X` | `295:411` | `feather:x` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-youtube"><path d="M22.54 6.42a2.78 2.78 0 0 0-1.94-2C18.88 4 12 4 12 4s-6.88 0-8.6.46a2.78 2.78 0 0 0-1.94 2A29 29 0 0 0 1 11.75a29 29 0 0 0 .46 5.33A2.78 2.78 0 0 0 3.4 19c1.72.46 8.6.46 8.6.46s6.88 0 8.6-.46a2.78 2.78 0 0 0 1.94-2 29 29 0 0 0 .46-5.25 29 29 0 0 0-.46-5.33z"></path><polygon points="9.75 15.02 15.5 11.75 9.75 8.48 9.75 15.02"></polygon></svg> | `Youtube` | `295:413` | `feather:youtube` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-zap-off"><polyline points="12.41 6.75 13 2 10.57 4.92"></polyline><polyline points="18.57 12.91 21 10 15.66 10"></polyline><polyline points="8 8 3 14 12 14 11 22 16 16"></polyline><line x1="1" y1="1" x2="23" y2="23"></line></svg> | `Zap off` | `295:414` | `feather:zap-off` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-zap"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"></polygon></svg> | `Zap` | `295:415` | `feather:zap` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-zoom-in"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line><line x1="11" y1="8" x2="11" y2="14"></line><line x1="8" y1="11" x2="14" y2="11"></line></svg> | `Zoom in` | `295:416` | `feather:zoom-in` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-zoom-out"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line><line x1="8" y1="11" x2="14" y2="11"></line></svg> | `Zoom out` | `295:417` | `feather:zoom-out` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-rotate-ccw"><polyline points="1 4 1 10 7 10"></polyline><path d="M3.51 15a9 9 0 1 0 2.13-9.36L1 10"></path></svg> | `Undo` | `2818:6316` | `feather:rotate-ccw` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-rotate-cw"><polyline points="23 4 23 10 17 10"></polyline><path d="M20.49 15a9 9 0 1 1-2.12-9.36L23 10"></path></svg> | `Redo` | `2818:6315` | `feather:rotate-cw` |

---

### Mobile Application · Home Feather (filled)

Figma [Mobile Application Home `2042:26021`](https://www.figma.com/design/6dCDQkwc8DhvyovBbDTyp5/Mobile-Application?node-id=2042-26021) uses filled Feather glyphs (not outline stroke, not Material). Rendered via `<Sym>` / `FigmaIcon` in [`src/components/mobile/figmaIcons.jsx`](src/components/mobile/figmaIcons.jsx).

| Figma name | Sym `name` | Aliases |
|------------|------------|---------|
| Calendar | `calendar` | `calendar_today`, `calendar_month`, `event` — purple Colorful icon Badge Archive [`194:153538`](https://www.figma.com/design/4WXeI8fgBLQGgQ9VheM2yD/-Archive--Mobile-Application?node-id=194-153538) |
| Clock | `clock` | `schedule`, `access_time` |
| Map pin | `map-pin` | `location_on`, `place`, `map_pin` — also clinic visit service badge (Archive `367:41054`) |
| Video | `video` | `videocam`, `video_cam` |
| File text | `file-text` | `description`, `file_text`, `article` |
| Chevron right | `chevron-right` | `chevron_right` |

Colorful icon Badge glyphs on the appointment card use `text/brandPrimary/secondary` (`#08A768`), not Colorful Badge label green (`#007549`).

---

### MIH Custom / Medical Icons (Material Symbols)

| Preview | Name | Node ID | Source |
|---------|------|---------|--------|
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960" fill="currentColor"><path d="m336-294 144-144 144 144 42-42-144-144 144-144-42-42-144 144-144-144-42 42 144 144-144 144 42 42ZM180-120q-24 0-42-18t-18-42v-600q0-24 18-42t42-18h600q24 0 42 18t18 42v600q0 24-18 42t-42 18H180Zm0-60h600v-600H180v600Zm0-600v600-600Z"/></svg> | `disabled_by_default` | `295:412` | `material:disabled_by_default` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M180-120q-24 0-42-18t-18-42v-600q0-24 18-42t42-18h299v60H180v600h299v60H180Zm486-185-43-43 102-102H360v-60h363L621-612l43-43 176 176-174 174Z"/></svg> | `leave` | `306:1588` | `material:logout` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M308-140h344v-127q0-72-50-121.5T480-438q-72 0-122 49.5T308-267v127Zm294-432q50-50 50-122v-126H308v126q0 72 50 122t122 50q72 0 122-50ZM160-80v-60h88v-127q0-71 40-129t106-84q-66-27-106-85t-40-129v-126h-88v-60h640v60h-88v126q0 71-40 129t-106 85q66 26 106 84t40 129v127h88v60H160Z"/></svg> | `wait` | `2036:2902` | `material:hourglass_empty` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M180-80q-24 0-42-18t-18-42v-620q0-24 18-42t42-18h65v-60h65v60h340v-60h65v60h65q24 0 42 18t18 42v620q0 24-18 42t-42 18H180Zm0-60h600v-430H180v430Zm0-490h600v-130H180v130Zm0 0v-130 130Z"/></svg> | `calendar_today` | `324:2528` | `material:calendar_today` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M250-160q-86 0-148-62T40-370q0-78 49.5-137.5T217-579q20-97 94-158.5T482-799q113 0 189.5 81.5T748-522v24q72-2 122 46.5T920-329q0 69-50 119t-119 50H510q-24 0-42-18t-18-42v-258l-83 83-43-43 156-156 156 156-43 43-83-83v258h241q45 0 77-32t32-77q0-45-32-77t-77-32h-63v-84q0-89-60.5-153T478-739q-89 0-150 64t-61 153h-19q-62 0-105 43.5T100-371q0 62 43.93 106.5T250-220h140v60H250Zm230-290Z"/></svg> | `cloud_upload` | `341:8156` | `material:cloud_upload` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M80-130v-60h800v60H80Zm40-120v-270h100v270H120Zm206 0v-470h100v470H326Zm207 0v-350h100v350H533Zm207 0v-590h100v590H740Z"/></svg> | `bar_chart_4_bars` | `376:10648` | `material:bar_chart_4_bars` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M180-120q-24.75 0-42.37-17.63Q120-155.25 120-180v-600q0-24.75 17.63-42.38Q155.25-840 180-840h205q5-35 32-57.5t63-22.5q36 0 63 22.5t32 57.5h205q24.75 0 42.38 17.62Q840-804.75 840-780v329q-14-8-29.5-13t-30.5-8v-308H180v600h309q4 16 9.02 31.17Q503.05-133.66 510-120H180Zm0-107v47-600 308-4 249Zm100-53h211q4-16 9-31t13-29H280v60Zm0-170h344q14-7 27-11.5t29-8.5v-40H280v60Zm0-170h400v-60H280v60Zm224.5-187.5Q515-818 515-832t-10.5-24.5Q494-867 480-867t-24.5 10.5Q445-846 445-832t10.5 24.5Q466-797 480-797t24.5-10.5ZM732.5-41Q655-41 600-96.5T545-228q0-78.43 54.99-133.72Q654.98-417 733-417q77 0 132.5 55.28Q921-306.43 921-228q0 76-55.5 131.5T732.5-41ZM718-101h33v-110h110v-33H751v-110h-33v110H608v33h110v110Z"/></svg> | `assignment_add` | `376:10732` | `material:assignment_add` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M315.5-398.73q-15.5-15.72-15.5-38.5 0-22.77 15.73-38.27 15.72-15.5 38.5-15.5 22.77 0 38.27 15.73 15.5 15.72 15.5 38.5 0 22.77-15.73 38.27-15.72 15.5-38.5 15.5-22.77 0-38.27-15.73Zm253 0q-15.5-15.72-15.5-38.5 0-22.77 15.73-38.27 15.72-15.5 38.5-15.5 22.77 0 38.27 15.73 15.5 15.72 15.5 38.5 0 22.77-15.73 38.27-15.72 15.5-38.5 15.5-22.77 0-38.27-15.73ZM480-140q142.38 0 241.19-98.95T820-480.47q0-25.53-4-50.53t-10-46q-20 5-43.26 7-23.26 2-48.74 2-97.11 0-183.56-40Q444-648 383-722q-34 81-97.5 141.5T140-487v7q0 142.37 98.81 241.19Q337.63-140 480-140Zm0 60q-83 0-156-31.5T197-197q-54-54-85.5-127T80-480q0-83 31.5-156T197-763q54-54 127-85.5T480-880q83 0 156 31.5T763-763q54 54 85.5 127T880-480q0 83-31.5 156T763-197q-54 54-127 85.5T480-80Zm-92-727q88 103 162.5 141T714-628q24 0 38-1t31-6q-45-81-122.5-133T480-820q-27 0-51 4t-41 9ZM149-558q48-18 109.5-81.5T346-793q-87 39-131.5 99.5T149-558Zm239-249Zm-42 14Z"/></svg> | `face` | `376:10749` | `material:face` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M180-120q-24.75 0-42.37-17.63Q120-155.25 120-180v-600q0-24.75 17.63-42.38Q155.25-840 180-840h205q5-35 32-57.5t63-22.5q36 0 63 22.5t32 57.5h205q24.75 0 42.38 17.62Q840-804.75 840-780v600q0 24.75-17.62 42.37Q804.75-120 780-120H180Zm0-60h600v-600H180v600Zm100-100h273v-60H280v60Zm0-170h400v-60H280v60Zm0-170h400v-60H280v60Zm224.5-187.5Q515-818 515-832t-10.5-24.5Q494-867 480-867t-24.5 10.5Q445-846 445-832t10.5 24.5Q466-797 480-797t24.5-10.5ZM180-180v-600 600Z"/></svg> | `clipboard-list` | `376:10772` | `material:assignment` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M540-81q-112 0-186-78.5T280-347v-35q-85-11-142.5-75.71T80-610v-230h120v-40h60v140h-60v-40h-60v169.68q0 71.32 49.5 120.82T310-440q71 0 120.5-49.5T480-610.32V-780h-60v40h-60v-140h60v40h120v230q0 87.58-57.5 152.29T340-382v35q0 85 56.5 145.5T540-141q81 0 140.5-60.15T740-347.23V-424q-35-10-57.5-39T660-530q0-45.83 32.12-77.92 32.12-32.08 78-32.08T848-607.92q32 32.09 32 77.92 0 38-22.5 67T800-424v77q0 111-76.5 188.5T540-81Zm265.5-413.32q14.5-14.33 14.5-35.5 0-21.18-14.32-35.68-14.33-14.5-35.5-14.5-21.18 0-35.68 14.32-14.5 14.33-14.5 35.5 0 21.18 14.32 35.68 14.33 14.5 35.5 14.5 21.18 0 35.68-14.32ZM770-530Z"/></svg> | `Stethoscope` | `376:10917` | `material:stethoscope` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M345-120q-94 0-159.5-65.5T120-345q0-45 17-86t49-73l270-270q32-32 73-49t86-17q94 0 159.5 65.5T840-615q0 45-17 86t-49 73L504-186q-32 32-73 49t-86 17Zm273-265 114-113q23-23 35.5-53.5T780-615q0-69-48-117t-117-48q-33 0-63.5 12.5T498-732L385-618l233 233ZM345-180q32 0 63-12.5t54-35.5l113-114-233-233-114 113q-23 23-35.5 53.5T180-345q0 69 48 117t117 48Z"/></svg> | `pill` | `377:11559` | `material:pill` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M480-120 300-300l44-44 136 136 136-136 44 44-180 180ZM344-612l-44-44 180-180 180 180-44 44-136-136-136 136Z"/></svg> | `unfold_more` | `380:12561` | `material:unfold_more` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M160-200v-60h80v-304q0-84 49.5-150.5T420-798v-22q0-25 17.5-42.5T480-880q25 0 42.5 17.5T540-820v22q81 17 130.5 83.5T720-564v304h80v60H160Zm320-302Zm0 422q-33 0-56.5-23.5T400-160h160q0 33-23.5 56.5T480-80ZM300-260h360v-304q0-75-52.5-127.5T480-744q-75 0-127.5 52.5T300-564v304Z"/></svg> | `Notification` | `376:11292` | `material:notifications` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M273-160 80-353l193-193 42 42-121 121h316v60H194l121 121-42 42Zm414-254-42-42 121-121H450v-60h316L645-758l42-42 193 193-193 193Z"/></svg> | `swap_horiz` | `2116:2875` | `material:swap_horiz` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="m833-41-39-39H160v-60h88v-127q0-70 40.5-128.5T394-480q-41-17-76.5-51T264-610L26-848l43-43L876-84l-43 43ZM566-480l-47-47q57-14 95-61t38-106v-126H308v82l-60-60v-22h-22l-60-60h634v60h-88v126q0 70-40.5 129T566-480ZM308-140h344v-82L441-433q-57 14-95 60.5T308-267v127Zm404 0h22l-22-22v22Z"/></svg> | `hourglass_disabled` | `2296:11175` | `material:hourglass_disabled` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M120-510v-330h330v330H120Zm0 390v-330h330v330H120Zm390-390v-330h330v330H510Zm0 390v-330h330v330H510ZM180-570h210v-210H180v210Zm390 0h210v-210H570v210Zm0 390h210v-210H570v210Zm-390 0h210v-210H180v210Zm390-390Zm0 180Zm-180 0Zm0-180Z"/></svg> | `grid_view` | `2188:4428` | `material:grid_view` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M477-120q-149 0-253-105.5T120-481h60q0 125 86 213t211 88q127 0 215-89t88-216q0-124-89-209.5T477-780q-68 0-127.5 31T246-667h105v60H142v-208h60v106q52-61 123.5-96T477-840q75 0 141 28t115.5 76.5Q783-687 811.5-622T840-482q0 75-28.5 141t-78 115Q684-177 618-148.5T477-120Zm128-197L451-469v-214h60v189l137 134-43 43Z"/></svg> | `history` | `2198:7104` | `material:history` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M349.91-160q-28.91 0-49.41-20.59-20.5-20.59-20.5-49.5t20.59-49.41q20.59-20.5 49.5-20.5t49.41 20.59q20.5 20.59 20.5 49.5t-20.59 49.41q-20.59 20.5-49.5 20.5Zm260 0q-28.91 0-49.41-20.59-20.5-20.59-20.5-49.5t20.59-49.41q20.59-20.5 49.5-20.5t49.41 20.59q20.5 20.59 20.5 49.5t-20.59 49.41q-20.59 20.5-49.5 20.5Zm-260-250q-28.91 0-49.41-20.59-20.5-20.59-20.5-49.5t20.59-49.41q20.59-20.5 49.5-20.5t49.41 20.59q20.5 20.59 20.5 49.5t-20.59 49.41q-20.59 20.5-49.5 20.5Zm260 0q-28.91 0-49.41-20.59-20.5-20.59-20.5-49.5t20.59-49.41q20.59-20.5 49.5-20.5t49.41 20.59q20.5 20.59 20.5 49.5t-20.59 49.41q-20.59 20.5-49.5 20.5Zm-260-250q-28.91 0-49.41-20.59-20.5-20.59-20.5-49.5t20.59-49.41q20.59-20.5 49.5-20.5t49.41 20.59q20.5 20.59 20.5 49.5t-20.59 49.41q-20.59 20.5-49.5 20.5Zm260 0q-28.91 0-49.41-20.59-20.5-20.59-20.5-49.5t20.59-49.41q20.59-20.5 49.5-20.5t49.41 20.59q20.5 20.59 20.5 49.5t-20.59 49.41q-20.59 20.5-49.5 20.5Z"/></svg> | `drag_indicator` | `2214:7896` | `material:drag_indicator` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M160-390v-60h640v60H160Zm0-120v-60h640v60H160Z"/></svg> | `drag_handle` | `2216:7975` | `material:drag_handle` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="m278-40 116-586-101 43v133h-61v-173l192-81q14-6 29.5-7.5T484-710q17 3 29.5 11t20.5 20l42 66q31 48 77.5 75.5T753-510v60q-70-2-123.5-30.5T533-568l-38 152 92 83v293h-60v-240l-108-98-79 338h-62Zm210.5-735.5Q467-797 467-827t21.5-51.5Q510-900 540-900t51.5 21.5Q613-857 613-827t-21.5 51.5Q570-754 540-754t-51.5-21.5Z"/></svg> | `directions_walk` | `2265:8943` | `material:directions_walk` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M180-630h600v-130H180v130Zm0 0v-130 130Zm0 550q-24 0-42-18t-18-42v-620q0-24 18-42t42-18h65v-60h65v60h340v-60h65v60h65q24 0 42 18t18 42v307q-14.17-7.29-29.08-12.14Q796-470 780-473v-97H180v430h319q6 17 14 31.5T532-80H180Zm417.5-15.5Q542-151 542-229t55.5-133.5Q653-418 731-418t133.5 55.5Q920-307 920-229T864.5-95.5Q809-40 731-40T597.5-95.5ZM789.24-128 817-156l-75-75v-112h-39v126l86.24 89Z"/></svg> | `calendar_clock` | `2265:8944` | `material:calendar_clock` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M80-160v-94q0-34 17-62.5t51-43.5q72-32 132-46t120-14q29 0 61.5 3.5T528-404l-49 49q-20-2-39.5-3.5T400-360q-58 0-105.5 10.5T172-306q-17 8-24.5 23t-7.5 29v34h319l60 60H80Zm545 16L484-285l42-42 99 99 213-213 42 42-255 255ZM292-524q-42-42-42-108t42-108q42-42 108-42t108 42q42 42 42 108t-42 108q-42 42-108 42t-108-42Zm167 304Zm5.5-347.5Q490-593 490-632t-25.5-64.5Q439-722 400-722t-64.5 25.5Q310-671 310-632t25.5 64.5Q361-542 400-542t64.5-25.5ZM400-632Z"/></svg> | `how_to_reg` | `2265:8945` | `material:how_to_reg` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M655-452v-60h145v60H655Zm33 292-119-88 34-47 119 88-34 47Zm-85-505-34-47 119-88 34 47-119 88ZM120-361v-240h160l200-200v640L280-361H120Zm300-288L307-541H180v120h127l113 109v-337Zm-94 168Z"/></svg> | `brand_awareness` | `2269:9387` | `material:brand_awareness` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M700-80v-120H580v-60h120v-120h60v120h120v60H760v120h-60Zm-520-80q-24 0-42-18t-18-42v-540q0-24 18-42t42-18h65v-60h65v60h260v-60h65v60h65q24 0 42 18t18 42v302q-15-2-30-2t-30 2v-112H180v350h320q0 15 3 30t8 30H180Zm0-470h520v-130H180v130Zm0 0v-130 130Z"/></svg> | `calendar_add_on` | `2295:11159` | `material:calendar_add_on` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M435-279h90v-156h156v-90H525v-156h-90v156H279v90h156v156ZM180-120q-24 0-42-18t-18-42v-600q0-24 18-42t42-18h600q24 0 42 18t18 42v600q0 24-18 42t-42 18H180Zm0-60h600v-600H180v600Zm0-600v600-600Z"/></svg> | `local_hospital` | `2296:11166` | `material:local_hospital` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M40-120v-720h560v720H371v-170H270v170H40Zm60-60h110v-170h221v170h109v-600H100v600Zm110-270h60v-60h-60v60Zm0-160h60v-60h-60v60Zm160 160h60v-60h-60v60Zm0-160h60v-60h-60v60Zm424 256-42-42 53-54H640v-60h165l-53-54 42-42 126 126-126 126ZM210-180v-170h221v170-170H210v170Z"/></svg> | `moving_ministry` | `2308:11875` | `material:moving_ministry` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M450-810v-150h60v150h-60Zm-187 53L152-869l42-43 112 112-43 43ZM150-40q-12.75 0-21.37-8.63Q120-57.25 120-70v-324l84-244.65q6-18.35 21.5-29.85T261-680h114v-75h153q-23 29-37.5 63T472-620H258l-60 176h323q13 17 29 32.5t34 27.5H180v200h600v-163q16-4 30.92-9.14Q825.83-361.29 840-369v299q0 12.75-8.62 21.37Q822.75-40 810-40h-21q-12.75 0-21.37-8.63Q759-57.25 759-70v-54H200v54q0 12.75-8.62 21.37Q182.75-40 170-40h-20Zm100-214h120q13 0 21.5-8.68 8.5-8.67 8.5-21.5 0-12.82-8.62-21.32-8.63-8.5-21.38-8.5H250v60Zm460 0v-60H590q-13 0-21.5 8.68-8.5 8.67-8.5 21.5 0 12.82 8.63 21.32 8.62 8.5 21.37 8.5h120ZM180-384v200-200Zm518-126 141-142-28-28-113 114-59-60-28 29 87 87Zm164.5-222.5Q919-676 919-595t-56.5 137.5Q806-401 725-401t-137.5-56.5Q531-514 531-595t56.5-137.5Q644-789 725-789t137.5 56.5Z"/></svg> | `ambulance` | `2334:1882` | `material:ambulance` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M452-202h60v-201l82 82 42-42-156-152-154 154 42 42 84-84v201ZM220-80q-24 0-42-18t-18-42v-680q0-24 18-42t42-18h361l219 219v521q0 24-18 42t-42 18H220Zm331-554v-186H220v680h520v-494H551ZM220-820v186-186 680-680Z"/></svg> | `file-up` | `2335:2007` | `material:upload_file` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M180-80q-24 0-42-18t-18-42v-620q0-24 18-42t42-18h65v-60h65v60h340v-60h65v60h65q24 0 42 18t18 42v620q0 24-18 42t-42 18H180Zm0-60h600v-620H180v620Zm180 0v-92l-72-84q-11-11-19.5-30t-8.5-44q0-13 2.5-25.5T271-440q-5-11-8-23.5t-3-26.5q0-25 8.5-44t19.5-30l72-84v-112h60v123q0 5-7 19l-80 94q-7 8-10 16.5t-3 17.5q0 20 13 34.5t33 14.5q9 0 17-3t14-10q17-17 38.5-26t44.5-9q23 0 44.5 9t38.5 26q7 7 15 10t16 3q20 0 33-14.5t13-33.5q0-9-3.5-17.5T627-523l-80-95q-4-4-5.5-9t-1.5-10v-123h60v112l73 86q14 16 20.5 34.5T700-489q0 13-3.5 25.5T688-440q6 12 9 24.5t3 25.5q0 25-8.5 44T672-316l-72 84v92h-60v-103q0-6 7-19l80-94q7-8 10-17t3-18q-11 5-22 7.5t-23 2.5q-20 0-40-8t-35-24q-7-8-17.5-12t-22.5-4q-11 0-21.5 4T440-413q-15 16-34.5 24t-39.5 8q-12 0-23.5-2.5T320-391q0 9 3 18t10 17l80 94q3 5 5 9.5t2 9.5v103h-60Zm-180 0v-620 620Z"/></svg> | `radiology` | `2340:3840` | `material:radiology` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M200-120v-60h208v-104h-15q-81 0-137-56t-56-137q0-61 35-111t92-70q4-40 35-65t72-22l-21-59 41-14.56L440-856l66-24 14 37 40-14 113 295-43 15 14 37-64 23-14-37-43 16-25-68q-15 17-35.5 24.5t-43.83 6.5Q393-546 371-561t-35-38q-35 17-55.5 49.97Q260-516.07 260-477q0 55.42 38.79 94.21Q337.58-344 393-344h347v60H508v104h252v60H200Zm356-452 53-19-80-206-53 19 80 206Zm-94.5-37.32q14.5-14.33 14.5-35.5 0-21.18-14.32-35.68-14.33-14.5-35.5-14.5-21.18 0-35.68 14.32-14.5 14.33-14.5 35.5 0 21.18 14.32 35.68 14.33 14.5 35.5 14.5 21.18 0 35.68-14.32ZM556-572Zm-130-75Zm2 0Z"/></svg> | `biotech` | `2340:3839` | `material:biotech` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M626-533q23 0 38.5-15.5T680-587q0-23-15.5-38.5T626-641q-23 0-38.5 15.5T572-587q0 23 15.5 38.5T626-533Zm-292 0q23 0 38.5-15.5T388-587q0-23-15.5-38.5T334-641q-23 0-38.5 15.5T280-587q0 23 15.5 38.5T334-533Zm267.5 236.5Q657-332 682-393H278q26 61 81 96.5T480-261q66 0 121.5-35.5ZM324-111.5Q251-143 197-197t-85.5-127Q80-397 80-480t31.5-156Q143-709 197-763t127-85.5Q397-880 480-880t156 31.5Q709-817 763-763t85.5 127Q880-563 880-480t-31.5 156Q817-251 763-197t-127 85.5Q563-80 480-80t-156-31.5ZM480-480Zm241 241q99-99 99-241t-99-241q-99-99-241-99t-241 99q-99 99-99 241t99 241q99 99 241 99t241-99Z"/></svg> | `mood_heart` | `2362:5568` | `material:mood` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M85-545q23-143 134.5-239T480-880q75 0 141 25.5T740-784q-10 17-16.5 30.5T713-728q-46-43-105.5-67.5T480-820q-112 0-199 64T159-592q-22 5-41 16.5T85-545ZM480-80q-149 0-260.5-96.5T85-416q13 20 32 32t42 17q35 100 122.5 163.5T480-140q142 0 241-99.5T820-480q0-17-2-34t-5-34q7 2 13.5 2.5t13.5.5q9 0 17.5-1t16.5-3q3 17 4.5 34t1.5 35q0 82-31.5 155T763-197.5q-54 54.5-127 86T480-80ZM336-503l77-77-78-78-35 35 43 42-43 43 36 35Zm451-124q-22-22-22-53 0-26 13.5-54t61.5-97q48 69 61.5 97t13.5 54q0 31-22 53t-53 22q-31 0-53-22ZM625-502l36-36-43-43 42-42-35-35-78 78 78 78Zm-145 85q-26 0-51 6t-48 18l-146-84q0-16-7-29.5T208-528q-20-11-42-5.5T132-508q-11 20-5 42t26 34q14 8 28.5 7t28.5-9l125 73q-18 17-32.5 37T278-280h53q22-42 62.5-65t87.5-23q47 0 86.5 23t62.5 65h52q-25-62-80-99.5T480-417Zm0-63Z"/></svg> | `sick` | `2362:5573` | `material:sick` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M480-581 324-425l40 40 86-86v201h60v-201l86 86 40-40-156-156Zm-300-93v494h600v-494H180Zm0 554q-24.75 0-42.37-17.63Q120-155.25 120-180v-529q0-9.88 3-19.06 3-9.18 9-16.94l52-71q8-11 20.94-17.5Q217.88-840 232-840h495q14.12 0 27.06 6.5T775-816l53 71q6 7.76 9 16.94 3 9.18 3 19.06v529q0 24.75-17.62 42.37Q804.75-120 780-120H180Zm17-614h565l-36.41-46H233l-36 46Zm283 307Z"/></svg> | `Archive restore` | `2373:6338` | `material:unarchive` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M280-453h400v-60H280v60ZM480-80q-82 0-155-31.5t-127.5-86Q143-252 111.5-325T80-480q0-83 31.5-156t86-127Q252-817 325-848.5T480-880q83 0 156 31.5T763-763q54 54 85.5 127T880-480q0 82-31.5 155T763-197.5q-54 54.5-127 86T480-80Zm0-60q142 0 241-99.5T820-480q0-142-99-241t-241-99q-141 0-240.5 99T140-480q0 141 99.5 240.5T480-140Zm0-340Z"/></svg> | `Circle minus` | `2373:6361` | `material:do_not_disturb_on` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M222-80q-43.75 0-74.37-30.63Q117-141.25 117-185v-125h127v-570l59.8 60 59.8-60 59.8 60 59.8-60 59.8 60 60-60 60 60 60-60 60 60 60-60v695q0 43.75-30.62 74.37Q781.75-80 738-80H222Zm516-60q20 0 32.5-12.5T783-185v-595H304v470h389v125q0 20 12.5 32.5T738-140ZM357-622v-60h240v60H357Zm0 134v-60h240v60H357Zm333-134q-12 0-21-9t-9-21q0-12 9-21t21-9q12 0 21 9t9 21q0 12-9 21t-21 9Zm0 129q-12 0-21-9t-9-21q0-12 9-21t21-9q12 0 21 9t9 21q0 12-9 21t-21 9ZM221-140h412v-110H177v65q0 20 12.65 32.5T221-140Zm-44 0v-110 110Z"/></svg> | `receipt_long` | `2374:6409` | `material:receipt_long` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M479-120 189-279v-240L40-600l439-240 441 240v317h-60v-282l-91 46v240L479-120Zm0-308 315-172-315-169-313 169 313 172Zm0 240 230-127v-168L479-360 249-485v170l230 127Zm1-240Zm-1 74Zm0 0Z"/></svg> | `School` | `2485:3139` | `material:school` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M377-198v-60h463v60H377Zm0-252v-60h463v60H377Zm0-253v-60h463v60H377ZM189-161q-28.05 0-48.02-19Q121-199 121-227.5t19.5-48q19.5-19.5 48-19.5t47.5 19.98q19 19.97 19 48.02 0 27.23-19.39 46.61Q216.23-161 189-161Zm0-252q-28.05 0-48.02-19.5Q121-452 121-480t19.98-47.5Q160.95-547 189-547q27.23 0 46.61 19.5Q255-508 255-480t-19.39 47.5Q216.23-413 189-413Zm-48.5-272.5Q121-705 121-733t19.5-47.5Q160-800 188-800t47.5 19.5Q255-761 255-733t-19.5 47.5Q216-666 188-666t-47.5-19.5Z"/></svg> | `format_list_bulleted` | `2520:8066` | `material:format_list_bulleted` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M179-179q-19-19-19-47t19-47q19-19 47-19t47 19q19 19 19 47t-19 47q-19 19-47 19t-47-19Zm254 0q-19-19-19-47t19-47q19-19 47-19t47 19q19 19 19 47t-19 47q-19 19-47 19t-47-19Zm254 0q-19-19-19-47t19-47q19-19 47-19t47 19q19 19 19 47t-19 47q-19 19-47 19t-47-19ZM179-433q-19-19-19-47t19-47q19-19 47-19t47 19q19 19 19 47t-19 47q-19 19-47 19t-47-19Zm254 0q-19-19-19-47t19-47q19-19 47-19t47 19q19 19 19 47t-19 47q-19 19-47 19t-47-19Zm254 0q-19-19-19-47t19-47q19-19 47-19t47 19q19 19 19 47t-19 47q-19 19-47 19t-47-19ZM179-687q-19-19-19-47t19-47q19-19 47-19t47 19q19 19 19 47t-19 47q-19 19-47 19t-47-19Zm254 0q-19-19-19-47t19-47q19-19 47-19t47 19q19 19 19 47t-19 47q-19 19-47 19t-47-19Zm254 0q-19-19-19-47t19-47q19-19 47-19t47 19q19 19 19 47t-19 47q-19 19-47 19t-47-19Z"/></svg> | `apps` | `2566:10945` | `material:apps` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M323-111.5Q250-143 196-197t-85-127.5Q80-398 80-482t31-156.5Q142-711 196-765t127-84.5Q396-880 480-880t157 30.5Q710-819 764-765t85 126.5Q880-566 880-482t-31 157.5Q818-251 764-197t-127 85.5Q564-80 480-80t-157-31.5ZM480-138q35-36 58.5-82.5T577-331H384q14 60 37.5 108t58.5 85Zm-85-12q-25-38-43-82t-30-99H172q38 71 88 111.5T395-150Zm171-1q72-23 129.5-69T788-331H639q-13 54-30.5 98T566-151ZM152-391h159q-3-27-3.5-48.5T307-482q0-25 1-44.5t4-43.5H152q-7 24-9.5 43t-2.5 45q0 26 2.5 46.5T152-391Zm221 0h215q4-31 5-50.5t1-40.5q0-20-1-38.5t-5-49.5H373q-4 31-5 49.5t-1 38.5q0 21 1 40.5t5 50.5Zm275 0h160q7-24 9.5-44.5T820-482q0-26-2.5-45t-9.5-43H649q3 35 4 53.5t1 34.5q0 22-1.5 41.5T648-391Zm-10-239h150q-33-69-90.5-115T565-810q25 37 42.5 80T638-630Zm-254 0h194q-11-53-37-102.5T480-820q-32 27-54 71t-42 119Zm-212 0h151q11-54 28-96.5t43-82.5q-75 19-131 64t-91 115Z"/></svg> | `Language` | `2657:5282` | `material:language` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M480-80q-82 0-155-31.5t-127.5-86Q143-252 111.5-325T80-480q0-85 32-158t87.5-127q55.5-54 130-84.5T489-880q79 0 150 26.5T763.5-780q53.5 47 85 111.5T880-527q0 108-63 170.5T650-294h-75q-18 0-31 14t-13 31q0 27 14.5 46t14.5 44q0 38-21 58.5T480-80Zm0-400Zm-198 11q15-15 15-35t-15-35q-15-15-35-15t-35 15q-15 15-15 35t15 35q15 15 35 15t35-15Zm126-170q15-15 15-35t-15-35q-15-15-35-15t-35 15q-15 15-15 35t15 35q15 15 35 15t35-15Zm214 0q15-15 15-35t-15-35q-15-15-35-15t-35 15q-15 15-15 35t15 35q15 15 35 15t35-15Zm131 170q15-15 15-35t-15-35q-15-15-35-15t-35 15q-15 15-15 35t15 35q15 15 35 15t35-15ZM480-140q11 0 15.5-4.5T500-159q0-14-14.5-26T471-238q0-46 30-81t76-35h73q76 0 123-44.5T820-527q0-132-100-212.5T489-820q-146 0-247.5 98.5T140-480q0 141 99.5 240.5T480-140Z"/></svg> | `Palette` | `2657:5327` | `material:palette` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M557.5-135.5Q502-191 502-269t55.5-133.5Q613-458 691-458t133.5 55.5Q880-347 880-269t-55.5 133.5Q769-80 691-80t-133.5-55.5ZM749.24-168 777-196l-75-75v-112h-39v126l86.24 89ZM180-120q-24.75 0-42.37-17.63Q120-155.25 120-180v-600q0-26 17-43t43-17h202q7-35 34.5-57.5T480-920q36 0 63.5 22.5T578-840h202q26 0 43 17t17 43v308q-15-9-29.52-15.48Q795.97-493.96 780-499v-281h-60v90H240v-90h-60v600h280q5 15 12 29.5t17 30.5H180Zm328.5-671.5Q520-803 520-820t-11.5-28.5Q497-860 480-860t-28.5 11.5Q440-837 440-820t11.5 28.5Q463-780 480-780t28.5-11.5Z"/></svg> | `pending_actions` | `2889:4827` | `material:pending_actions` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M430.5-755.5Q409-777 409-807t21.5-51.5Q452-880 482-880t51.5 21.5Q555-837 555-807t-21.5 51.5Q512-734 482-734t-51.5-21.5ZM696-80v-209H482q-30 0-51-21t-21-51v-247q0-30 21-51t51-21q23 0 39 9t38 35q42 49 92 82t109 35v60q-51 0-105-25t-104-67v183h133q30 0 51 21t21 51v216h-60Zm-300 0q-83 0-139.5-56.5T200-276q0-68 49.5-125.5T380-468v61q-54 5-86.5 44.5T261-276q0 58 38.5 97t96.5 39q47 0 87-32.5t44-86.5h61q-8 80-66 129.5T396-80Z"/></svg> | `accessible` | `2894:4866` | `material:accessible` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="M80-707v-173h173v60H140v113H80Zm0 627v-173h60v113h113v60H80Zm627 0v-60h113v-113h60v173H707Zm113-627v-113H707v-60h173v173h-60ZM708-251h63v63h-63v-63Zm0-126h63v63h-63v-63Zm-63 63h63v63h-63v-63Zm-63 63h63v63h-63v-63Zm-63-63h63v63h-63v-63Zm126-126h63v63h-63v-63Zm-63 63h63v63h-63v-63Zm-63-63h63v63h-63v-63Zm252-332v252H519v-252h252ZM440-440v252H188v-252h252Zm0-332v252H188v-252h252Zm-50 534v-152H238v152h152Zm0-332v-152H238v152h152Zm331 0v-152H569v152h152Z"/></svg> | `qr_code_scanner` | `2902:5052` | `material:qr_code_scanner` |
| <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"><path d="m310-60-60-45v-185h-60q-24 0-42-18t-18-42v-320h-10q-13 0-21.5-8.5T90-700q0-13 8.5-21.5T120-730h130v-90h-30q-13 0-21.5-8.5T190-850q0-13 8.5-21.5T220-880h120q13 0 21.5 8.5T370-850q0 13-8.5 21.5T340-820h-30v90h130q13 0 21.5 8.5T470-700q0 13-8.5 21.5T440-670h-10v320q0 24-18 42t-42 18h-60v230ZM190-350h180v-80h-80q-8 0-14-6t-6-14q0-8 6-14t14-6h80v-80h-80q-8 0-14-6t-6-14q0-8 6-14t14-6h80v-80H190v320ZM600-80q-24 0-42-18t-18-42v-266q0-32 8-48.5t21-30.5q19-20 25-31t6-24v-30h-10q-13 0-21.5-8.5T560-600q0-13 9-21.5t21-8.5h200q13 0 21.5 8.5T820-600q0 13-8.5 21.5T790-570h-10v30q0 12 7.5 25t26.5 33q13 14 19.5 29t6.5 47v266q0 24-18 42t-42 18H600Zm0-300h180v-26q0-18-10-32t-22-29q-15-19-21.5-35t-6.5-38v-30h-60v30q0 21-6 37t-21 35q-12 15-22.5 29.5T600-406v26Zm0 120h180v-80H600v80Zm0 120h180v-80H600v80Zm0-120h180-180Z"/></svg> | `Vaccine` | `2664:38` | `material:vaccines` |

---

### Usage

```html
<!-- Copy the SVG inline and set fill to inherit color -->
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
     fill="none" stroke="currentColor" stroke-width="2"
     stroke-linecap="round" stroke-linejoin="round">
  <!-- paste path(s) here -->
</svg>

<!-- For Material Symbols (fill-based): -->
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 -960 960 960"
     fill="currentColor">
  <!-- paste path here -->
</svg>
```

| Button Size | Icon Size |
|-------------|-----------|
| Small (S) | 16 × 16 px |
| Medium (M) | 20 × 20 px |
| Large (L) | 24 × 24 px |

---

## 6. Component Tokens

Component tokens are defined in the **MIH Design System Foundation** Semantic variable collection. They follow the Figma naming convention `{property}/{component}/{state}` and map to Primitive tokens. Only tokens that appear in the Figma variable defs for each component are listed here.

> **Naming rule:** Use the exact Figma Semantic token name (e.g. `icon/accordion/collapsed`) as the CSS variable alias in code. Append the hex value from the **default** (light) mode.

---

### 6.1 Button

See **Section 5** (Component Token Mapping) for the complete Button token set.

| Figma Semantic Token | Hex | Primitive |
|----------------------|-----|-----------|
| `icon/brandPrimaryButton/default` | `#007549` | emerald-700 |
| `icon/brandPrimary/on-brand` | `#ffffff` | white |
| `icon/neutral/default` | `#4d5358` | neutral-700 |
| `icon/disabled/default` | `#adb2b7` | neutral-400 |

#### 6.1a Forms library card (`FormCard` in `FormsLibrary.jsx`)

**Figma:** [MIH Design System — Card · FormCard](https://www.figma.com/design/KGU3Sk3s04cgRNSbE2M78w/MIH-Design-System---Card?node-id=150-1223) (node `150:1223`).

Thumbnail + **two pills** (row, `gap` 8px) + title/code, then a **two-button action row** (equal flex) and an optional **“ดูทั้งหมด”** control.

| Pill | Role | Component | Notes |
|---|---|---|---|
| 1 | **ประเภทฟอร์ม** (category) | `<ColorfulBadge>` | `size='small'`, `hierarchy='secondary'`, `showDot`, material `icon` + label from `FORM_CATEGORIES` — tokens `--surface-colorfulbadge-*` / `--text-colorfulbadge-*` |
| 2 | **สถานะ** (e.g. ประเมินแล้ว, รอกรอกข้อมูล) | `<StatusBadge>` | `size='small'`, `showDot`, material `icon` + label from `FORM_STATUSES` — tokens `--surface-statusbadge-*` / `--text-statusbadge-*` |

**Card frame (per Figma states Default · Hover · Selected):** `box-shadow` simulates border width without layout shift — `0 0 0 1px` default (`--border-card-default`), `0 0 0 4px` + `var(--shadow-card-hover)` on hover (`--border-brandPrimaryButton-tertiary`), `0 0 0 4px` + `var(--shadow-card)` when `selected={true}` (explicit list selection / Figma Selected) using `--border-brandPrimaryButton-default`. Cards that are only “กำลังเปิดอยู่” in workflow sense stay **default** frame unless the parent passes `selected`.

All actions use canonical button components from **§5.2** (Brand Subdue) and **§5.6** (Neutral Subdue); icons use **Material Symbols** names passed through those components’ `leadingIcon` / `trailingIcon`.

| UI | Component | `variant` | `size` | Icon prop | Token family (see §5.10) |
|---|---|---|---|---|---|
| ดู (preview) | `<BrandSubdueButton>` | `fill` | `sm` | `leadingIcon='visibility'` | Brand Subdue Fill |
| แก้ไข (form open) | `<BrandSubdueButton>` | `outline` | `sm` | `leadingIcon='edit'` | Brand Subdue Outline |
| กรอกข้อมูล (form not open) | `<NeutralSubdueButton>` | `outline` | `sm` | `leadingIcon='edit'` | Neutral Subdue Outline |
| ดูทั้งหมด | `<NeutralSubdueButton>` | `outline` | `sm` | `trailingIcon='chevron_right'` | Neutral Subdue Outline |

Row buttons set `className='flex-1 min-w-0'` and `style={{ minWidth: 0, maxWidth: 'none' }}` so they share the card width without the component default `minWidth` / `maxWidth` fighting the grid.

---

### 6.2 Calendar

**Nodes:** `325-3609` (day cell states) · `308-3` (Default calendar) · Figma library: **MIH Design System Foundation / Semantic**

#### 6.2.1 Color Tokens

| CSS Variable | Figma Semantic Token | Primitive | Hex | Usage |
|---|---|---|---|---|
| `--surface-calendar-bg` | `surface/calendar/default` | neutral/white | `#FFFFFF` | Calendar container background |
| `--surface-calendar-hover` | `surface/calendar/hover` | emerald-100 | `#E2F3EB` | Day hover bg · Range strip bg |
| `--text-calendar-hover` | `text/calendar/hover` | emerald-700 | `#007549` | Day number on hover |
| `--surface-calendar-current` | `surface/calendar/current` | emerald-100 | `#E2F3EB` | Today/Current day bg — **no border** |
| `--text-calendar-current` | `text/calendar/current` | emerald-700 | `#007549` | Today/Current day number |
| `--surface-calendar-selected` | `surface/calendar/selected` | emerald-700 | `#007549` | Selected day · Range endpoints bg (web deep-green) |
| `--surface-calendar-selected-mobile` | `surface/calendar/selected` | emerald-600 | `#08A768` | Selected day bg on **mobile** (brighter emerald — §12889 · Figma 173:82996) |
| `--text-calendar-selected` | `text/calendar/selected` | neutral/white | `#FFFFFF` | Selected day · Range endpoints text |
| `--text-calendar-disabled` | `text/calendar/disabled` | slate-200 | `#CBD5E1` | Disabled day number |
| `--text-calendar-other` | `text/neutral/tertiary` | neutral-400 | `#ADB2B7` | Other-month day numbers |
| `--text-calendar-default` | `text/neutral/default` | neutral-900 | `#1A1E23` | Default day number |
| `--text-calendar-month` | `text/calendar/month` | neutral-700 | `#4D5358` | Month/year header label |
| `--text-calendar-dow` | `text/neutral/secondary` | neutral-500 | `#636B72` | Day-of-week header (Su Mo Tu…) |
| `--icon-calendar-default` | `icon/calendar/default` | neutral-400 | `#ADB2B7` | Chevron nav icons |
| `--border-calendar` | `border/dropdown/default` | neutral-100 | `#EBEBEB` | Calendar container border |

#### 6.2.2 Dimension Tokens

| Property | Figma Value | CSS | Role |
|---|---|---|---|
| Container width | 320 px | `width: 320px` | `.cal-wrap` |
| Container border-radius | 16 px | `border-radius: 16px` | Outer card |
| Body padding | 16 px H, 8 px top, 16 px bottom | `.cal-body` | Inner padding |
| Day cell size | 40 × 40 px | `width/height: 40px` | `.cal-day` |
| Day cell border-radius | 9999 px | `border-radius: 9999px` | Fully round |
| Day grid gap | 2 px | `gap: 2px` | `.cal-grid` |
| DOW row height | 28 px | `height: 28px` | `.cal-dow-cell` |
| Header height | 32 px | `height: 32px` | `.cal-header` |

#### 6.2.3 Typography

| Element | Font | Size | Weight | Color Token |
|---|---|---|---|---|
| Month/year label | Sarabun | 16 px | 500 | `--text-calendar-month` |
| Day-of-week (DOW) | Sarabun | 12 px | 500 | `--text-calendar-dow` |
| Day number (default) | Sarabun | 14 px | 400 | `--text-calendar-default` |
| Day number (today / selected) | Sarabun | 14 px | 500 | — |

#### 6.2.4 Day Cell States

| State | Surface | Text | Notes |
|---|---|---|---|
| Default | transparent | `--text-calendar-default` | White bg from container |
| Hover | `--surface-calendar-hover` | `--text-calendar-hover` | CSS `:hover` only — no JS |
| Today / Current | `--surface-calendar-current` | `--text-calendar-current` | **No border** (old impl had border — wrong) |
| Selected | `--surface-calendar-selected` | `--text-calendar-selected` | Single-select; fully round (mobile → `--surface-calendar-selected-mobile` `#08A768`) |
| Full (เต็ม) | transparent | `--text-calendar-disabled` | Fully-booked day — renders label **"เต็ม"** (12px) instead of the number; `cursor: not-allowed`. Figma 140:9320 |
| Range start | `--surface-calendar-selected` | `--text-calendar-selected` | `border-radius: 9999px 0 0 9999px` |
| Range (middle) | `--surface-calendar-hover` | `--text-calendar-default` | `border-radius: 0` |
| Range end | `--surface-calendar-selected` | `--text-calendar-selected` | `border-radius: 0 9999px 9999px 0` |
| Disabled | transparent | `--text-calendar-disabled` | `cursor: not-allowed` |
| Other-month | transparent | `--text-calendar-other` | Non-clickable |

#### 6.2.5 Icon Token

| Icon | Figma Node | Token | CSS Var | Value |
|---|---|---|---|---|
| Chevron left (prev) | `306:1059` | `icon/calendar/default` | `--icon-calendar-default` | `#ADB2B7` |
| Chevron right (next) | `306:1059` rotated | `icon/calendar/default` | `--icon-calendar-default` | `#ADB2B7` |

SVG path (Figma exact): `M9.52864 3.52864 ... L5.52864 7.52864Z` — right chevron = same path rotated 180°.

#### 6.2.6 CSS Implementation

```css
/* :root tokens */
--surface-calendar-bg:       #FFFFFF;
--surface-calendar-hover:    #E2F3EB;   /* surface/calendar/hover */
--text-calendar-hover:       #007549;   /* text/calendar/hover */
--surface-calendar-current:  #E2F3EB;   /* surface/calendar/current */
--text-calendar-current:     #007549;   /* text/calendar/current */
--surface-calendar-selected: #007549;   /* surface/calendar/selected (web) */
--surface-calendar-selected-mobile: #08A768;  /* surface/calendar/selected on mobile (emerald-600) */
--text-calendar-selected:    #FFFFFF;   /* text/calendar/selected */
--text-calendar-disabled:    #CBD5E1;   /* text/calendar/disabled */
--text-calendar-other:       #ADB2B7;   /* text/neutral/tertiary */
--text-calendar-default:     #1A1E23;   /* text/neutral/default */
--text-calendar-month:       #4D5358;   /* text/calendar/month */
--text-calendar-dow:         #636B72;   /* text/neutral/secondary */
--icon-calendar-default:     #ADB2B7;   /* icon/calendar/default */
--border-calendar:           #EBEBEB;   /* border/dropdown/default */

/* Component classes — Type=Single */
.cal-wrap   { border:1px solid var(--border-calendar); border-radius:16px;
              background:var(--surface-calendar-bg); width:320px; }

/* Type=Range: two 320px panels, NO center divider (Figma node 2074-6195) */
.cal-range-outer { border:1px solid var(--border-calendar); border-radius:16px;
                   background:var(--surface-calendar-bg); width:640px;
                   display:flex; overflow:hidden; }
.cal-panel-left  { width:320px; padding:16px 8px 16px 16px;
                   display:flex; flex-direction:column; gap:4px; }
.cal-panel-right { width:320px; padding:16px 16px 16px 8px;
                   display:flex; flex-direction:column; gap:4px; }
/* ⚠ No border-right / center divider — panels share only the outer border */

.cal-header { padding:8px 16px; height:32px; display:flex;
              align-items:center; justify-content:space-between; }
.cal-month-label { font-size:16px; font-weight:500;
                   color:var(--text-calendar-month); }
.cal-body   { padding:8px 16px 16px; }
.cal-dow-cell { height:28px; font-size:12px; font-weight:500;
                color:var(--text-calendar-dow); }
.cal-grid   { display:grid; grid-template-columns:repeat(7,1fr); gap:2px; }
.cal-day    { width:40px; height:40px; border-radius:9999px;
              font-size:14px; font-weight:400;
              color:var(--text-calendar-default); }
/* Hover — CSS only */
.cal-day:not(.disabled):not(.other-month):not(.selected):not(.range-start):not(.range-end):hover {
  background:var(--surface-calendar-hover);
  color:var(--text-calendar-hover);
}
/* Today — NO border */
.cal-day.today    { background:var(--surface-calendar-current);
                    color:var(--text-calendar-current); }
.cal-day.selected { background:var(--surface-calendar-selected);
                    color:var(--text-calendar-selected); }
.cal-day.range         { background:var(--surface-calendar-hover); border-radius:0; }
.cal-day.range-start   { background:var(--surface-calendar-selected);
                         color:var(--text-calendar-selected);
                         border-radius:9999px 0 0 9999px; }
.cal-day.range-end     { background:var(--surface-calendar-selected);
                         color:var(--text-calendar-selected);
                         border-radius:0 9999px 9999px 0; }
.cal-day.disabled      { color:var(--text-calendar-disabled); cursor:not-allowed; }
.cal-day.other-month   { color:var(--text-calendar-other); cursor:default; }
```

#### 6.2.7 Interaction — Type=Default & Type=Range

**Type=Default** — Single-select: click any non-disabled, non-other-month day to select it. Clicking the same day again deselects it.

**Type=Range** — Two 320 px panels side-by-side inside a single 640 px border, **no center divider** (Figma node 2074-6195). Two-click selection:
1. Click 1 → sets range start (shown as `range-start`, fully round emerald circle)
2. Click 2 → sets range end; days between get `range` class (emerald-100 strip, no radius); endpoints get `range-start`/`range-end`
3. If click 2 date < click 1 date, the two are automatically swapped.
4. Clicking the same date as start clears the selection.

**Hover** is handled purely by CSS (no JavaScript), using the `:not()` selector chain above.

#### 6.2.8 Token Alias Chain

```
surface/calendar/current  →  Brand/primary/100  →  emerald-100  →  #E2F3EB
surface/calendar/selected →  Brand/primary/700  →  emerald-700  →  #007549
icon/calendar/default     →  neutral-400         →               →  #ADB2B7
border/dropdown/default   →  neutral-100                         →  #EBEBEB
text/calendar/month       →  neutral-700         →               →  #4D5358
```

> **Key correction from v1:** Day cell height is **40 px** (not 36 px). Today/Current state has **no border** (old CSS used `border: 1.5px solid`). Range strip reuses `surface/calendar/hover` — there is no separate `surface/calendar/range` token in Figma.

---

### 6.2.5b Chevron Button

**Node:** `3070-4061` (component set) · `3070-4062` (Default) · `3070-4070` (Hover)  
**Figma library:** MIH Design System Foundation / Semantic  
**Aliased by:** Calendar nav buttons (node 2074-6402)

#### Overview

Chevron Button เป็น utility button ขนาด 24×24px สำหรับ navigation ใน Calendar และ component อื่น ๆ ที่ต้องการปุ่มลูกศร มี 2 states: Default และ Hover

#### Dimension Tokens

| Property | Figma Token | Value | CSS |
|---|---|---|---|
| Width | dimension/size/600 | 24 px | `width: 24px` |
| Height | dimension/size/600 | 24 px | `height: 24px` |
| Border Radius | border-radius/200 | 8 px | `border-radius: 8px` |
| Icon Size | dimension/size/400 | 16 px | SVG 16×16 |

#### Color Tokens

| CSS Variable | Semantic Token | Alias Chain | Value | State |
|---|---|---|---|---|
| `--icon-calendar-default` | `icon/calendar/default` | neutral-400 | `#ADB2B7` | Default icon fill |
| `--icon-calendar-hover` | `icon/calendar/hover` | primary/600 | `#08A768` | Hover icon fill |
| `--surface-brandprimary-tertiary` | `surface/brandPrimary/tertiary` | primary/100 | `#E2F3EB` | Hover background |

#### States

| State | Background | Icon Fill | Figma Node |
|---|---|---|---|
| Default | transparent | `#ADB2B7` | 3070:4062 |
| Hover | `#E2F3EB` | `#08A768` | 3070:4070 |

#### CSS Implementation

```css
/* Chevron Button (node 3070:4061) */
.chev-btn {
  width: 24px;
  height: 24px;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  padding: 0;
  background: transparent;
  transition: background 0.15s;
}
.chev-btn svg path {
  fill: var(--icon-calendar-default);  /* #ADB2B7 */
  transition: fill 0.15s;
}
.chev-btn:hover {
  background: var(--surface-brandprimary-tertiary);  /* #E2F3EB */
}
.chev-btn:hover svg path {
  fill: var(--icon-calendar-hover);  /* #08A768 */
}
```

#### Token Alias Chain

```
icon fill (Default) : icon/calendar/default → color/neutral/400 → #ADB2B7
icon fill (Hover)   : icon/calendar/hover   → primary/600       → #08A768
bg (Hover)          : surface/brandPrimary/tertiary → primary/100 → #E2F3EB
```

#### Calendar Integration

Calendar nav buttons (node 2074-6402) alias ChevronButton tokens:

```css
/* Calendar inherits ChevronButton spec */
.cal-nav-btn { width: 24px; height: 24px; border-radius: 8px; }
.cal-nav-btn:hover { background: var(--surface-brandprimary-tertiary); }
.cal-nav-btn:hover svg path { fill: var(--icon-calendar-hover); }
```


---

### 6.3 Checkbox

**Nodes:** `321-2329` (component set) · Checkbox box `269:7–269:27` · Checkbox Button `2254:8344` · Checkbox Group `328:5438`  
**Icon position frame:** `328:5706` (Frame 1 — wrapper sizing) · **Selected-Add card:** `2498:6240`  
**Figma library:** MIH Design System Foundation / Semantic

#### 6.3.1 Component Properties

| Property | Values | Applies to |
|---|---|---|
| **State** | Default · Hover · Selected · Disabled-Unchecked · Disabled-Checked | Checkbox, Checkbox Button, Checkbox Group |
| **Type** | Default · Other (=Selected-Add) | Checkbox |
| **showBadge** | Yes · No | Checkbox |
| **showSubtext** | Yes · No | Checkbox |
| **showGroupInput1** | Yes · No | Checkbox (type=Other) |
| **showGroupInput2** | Yes · No | Checkbox (type=Other) |
| **Show border** | Yes · No | Checkbox Button only |

> **State=Selected, Type=Other** (node `2498:6240`): When checked, reveals an expandable card below the row with input fields and an "เพิ่ม" action button. Unique to Checkbox rows that trigger add-item flows.

#### 6.3.2 Color Tokens

| CSS Variable | Semantic Token | Alias Chain | Value | State |
|---|---|---|---|---|
| `#FFFFFF` (white) | `surface/switch/default` | grey/50 | `#FFFFFF` | Default fill |
| `--border-switch-default` | `border/switch/default` | neutral/300 | `#C9CDD0` | Default stroke 1.5px |
| `--surface-switch-on` | `surface/switch/on` | teal/600 | `#1B867D` | Selected + Selected-Add fill |
| `--surface-switch-hover` | `surface/switch/hover` | teal/300 | `#3EC3B2` | Hover (unchecked box hovered) fill |
| `--surface-switch-on-hover` | `surface/switch/on-hover` | teal/700 | `#196C65` | Selected-Hover (checked box hovered) fill |
| `--border-switch-disabled` | `border/switch/disabled` | neutral/200 | `#E2E4E6` | Disabled-Unchecked stroke |
| `--surface-disabled-secondary` | `surface/disabled/secondary-hover` | neutral/300 | `#C9CDD0` | Disabled-Checked fill |
| `#FFFFFF` (white) | `text/brandSecondary/on-default` | grey/50 | `#FFFFFF` | Checkmark stroke |
| `--text-content-default` | `text/content/default` | neutral/800 | `#363B3F` | Label text |
| `--text-content-tertiary` | `text/content/tertiary` | neutral/500 | `#858C92` | Subtext |
| `--text-content-quaternary` | `text/content/quaternary` | neutral/400 | `#ADB2B7` | Disabled label/subtext |
| `--surface-neutralbtn-quinary` | `surface/neutralButton/quinary` | grey/50 | `#FFFFFF` | Button bg (Default) |
| `--border-neutralbtn-tertiary` | `border/neutralButton/tertiary` | neutral/200 | `#E2E4E6` | Button border (Show border=Yes) |
| `--surface-neutralbtn-quaternary` | `surface/neutralButton/quaternary` | neutral/50 | `#F9FBFB` | Button bg (Hover) |
| `--surface-card-brand100` | `surface/card/brand-100` | **primary/50** (#f4fbf8) ★ | `#F4FBF8` | Selected-Add card background (Figma `2498:6269`) |
| `--border-brandprimary-tertiary` | `border/brandPrimaryButton/tertiary` | primary/200 → emerald/200 | `#C5EDDA` | Selected-Add button border |
| `--text-brandprimary-default` | `text/brandPrimaryButton/default` | primary/700 → emerald/700 | `#007549` | Selected-Add "เพิ่ม" button text + icon |
| `--surface-brandprimarybutton-tertiary` | `surface/brandPrimaryButton/tertiary` | primary/100 → emerald/100 | `#E2F3EB` | Brand Subdue Icon Button bg (trash btn in card) |
| `--text-topic-tertiary-title` | `text/topic/tertiary-title` | neutral/800 | `#363B3F` | Group card section title (Sarabun Bold 14px) |
| `--surface-colorfulbadge-grey` | `surface/colorfulbadge/grey` | neutral/50 | `#F5F7F9` | Colorful Badge background (Grey) |
| `--text-colorfulbadge-grey` | `text/colorfulbadge/grey` | neutral/700 | `#4D5358` | Colorful Badge text (Grey) |
| `--surface-input-default` | `surface/input/default` | white | `#FFFFFF` | InputField background |
| `--border-input-default` | `border/input/default` | neutral/200 | `#E2E4E6` | InputField border (Default) |
| `--text-input-default` | `text/input/default` | neutral/400 | `#ADB2B7` | InputField placeholder |
| `--text-input-required` | `text/input/require` | red/600 | `#DC2626` | InputField required asterisk (*) |
| `--icon-input-default` | `icon/input/default` | neutral/400 | `#ADB2B7` | InputField leading icon (calendar) |

> **Correction from v1:** Checked fill was `#007549` (emerald-700). Figma uses `surface/switch/on` → `teal/600` → **`#1B867D`**. No "indeterminate" state exists in Figma spec.  
> ★ **`surface/card/brand-100`** resolves to **`primary/50`** (`#f4fbf8`) — despite the name containing "100", the actual Figma primitive is `primary/50`. `setBrandTheme()` sets this to `p[50]`. Verified via `get_variable_defs` on node `2498:6269`.

#### 6.3.3 Dimension Tokens

| Property | Figma Token | Value | Component |
|---|---|---|---|
| Box size | `dimension/size/400` | **16×16 px** | Checkbox box |
| Box border-radius | `border-radius/100` | **4 px** | Checkbox box |
| Stroke weight | — | **1.5 px** | Default / Disabled stroke |
| **Wrapper (Frame 1)** | node `328:5706` | **16×20 px, padding-top: 4px** | `.chk-box-wrap` — positions box at y:4 to align with label baseline |
| Row gap | — | **8 px** | Checkbox Row horizontal gap (between wrapper and label) |
| Group gap | — | **12 px** | Checkbox Group vertical gap |
| Button size | — | **286×79 px** | Checkbox Button |
| Button padding | — | **l16 r16 t8 b8** | Checkbox Button |
| Button border-radius | `border-radius/200` | **8 px** | Checkbox Button |
| Button card stroke | `dimension/stroke/200` | **2 px inset ring** | Checkbox Button + RadioButton (anti-jump) |
| Button lift shadow | `Brand Drop Shadow Top/200` | `--shadow-brand-top-200` | Checkbox Button Hover + Selected |
| Checkmark icon | — | **7.6×6 px** (SVG Vector) | Centered in 16×16 box |

#### 6.3.3b CheckboxButton Card — States (Figma [2254:8344](https://www.figma.com/design/SBdh4TtY0KAa22s3dnyRNr/MIH-Design-System-Foundation?node-id=2254-8344), `showBorder=true`)

> **Not the same chrome as RadioButton (2860:3497).** Checkbox Button uses a **1px** border on Default/Hover/Disabled and **2px** on Selected; Hover casts **Brand Drop Shadow Bottom/200** (`--shadow-brand-drop-bottom-200`), Selected casts **Brand Drop Shadow Top/200** (`--shadow-brand-top-200`).

| State | React `state` | Card border | Drop shadow | Checkbox atom |
|---|---|---|---|---|
| Default | `default` | 1px `--border-neutralbutton-tertiary` | — | Empty (`surface/switch/default`) |
| Hover | `hover` | 1px `--border-brandprimarybutton-tertiary` | `--shadow-brand-drop-bottom-200` | Preview `--surface-switch-on-hover` + tick (unchecked row) |
| Selected | `selected` | 2px `--border-brandprimarybutton-default` | `--shadow-brand-top-200` | `--surface-switch-on` + tick |
| Disabled-Unchecked | `disabled-unchecked` | 1px neutral tertiary | — | Disabled stroke, label `--text-content-quaternary` |
| Disabled-Checked | `disabled-checked` | 1px neutral tertiary | — | `--surface-disabled-secondary-hover` + tick, label `--text-content-secondary` |

**`showBorder=false`:** 40px min-height, no border/shadow (Figma nodes `2623:2268` …).

**Implementation:** `src/components/checkbox/CheckboxButton.jsx`. PHCIS Registration “เหมือนที่อยู่ตามทะเบียนบ้าน” (`207:19757`): `border` + label-only slots. Toggle stays checkbox (`onChange(boolean)`).

| Selected-Add card radius | — | **12 px** | `.chk-addon` expandable card |
| Selected-Add card padding | — | **16 px** | `.chk-addon` inner padding |
| Selected-Add item gap | — | **12 px** | Gap between inputs inside `.chk-addon` |
| InputField height | `dimension/size/1000` | **40 px** | Input box (Large size) |
| InputField padding | `dimension/space/200` × `dimension/space/400` | **t8 b8 l16 r16** | Input box inner padding |
| InputField radius | `dimension/radius/200` | **8 px** | Input box corners |
| InputField border | `dimension/stroke/100` | **1 px** | Input border width |
| InputField icon | `icon-size/input/large` | **20×20 px** | Leading icon in input |
| Brand Subdue Icon Btn | — | **40×40 px** | Trash icon button in group card 2 |
| Icon Btn radius | `dimension/radius/200` | **8 px** | Icon button corners |
| Colorful Badge height | `dimension/size/500` | **24 px** | Small size (padding: 4px 8px) |
| Colorful Badge radius | pill | **999 px** | Capsule shape |
| Colorful Badge width (`width="hug"`) | hug content | label + horizontal padding | Treatment history / `Information` inline badges (Figma 176:38684) — no `min-width` floor on text pills |
| Colorful Badge width (`width="cap"`, default) | min **40 px** · max **224 px** | ellipsis when label overflows | Select popover rows, tables, fixed slots |

#### Clinical badge palettes (CPOE · system-wide)

Source: `src/components/badge/clinicalBadgeStyles.js` — used by `BadgeSelectForm`, `HistoryPillBadge`, `Select` popover rows, and treatment read-only panels.

| Domain | Figma | Resolver | ColorfulBadge `style` per label |
|---|---|---|---|
| ระดับความรุนแรง | [245:24395](https://www.figma.com/design/cOjwwHvgGURA3IEuls8RT5/PHCIS-%7C-CPOE?node-id=245-24395) | `resolveSeverityBadgeStyle` | วิกฤต `red` · รุนแรง `orange` · ปานกลาง `blue` · เล็กน้อย `teal` · น้อย `yellow` |
| ความน่าจะเป็น | [245:24530](https://www.figma.com/design/cOjwwHvgGURA3IEuls8RT5/PHCIS-%7C-CPOE?node-id=245-24530) | `resolveLikelihoodBadgeStyle` | แน่นอน `teal` · น่าจะใช่ `blue` · เป็นไปได้ `indigo` · สงสัย `purple` |
| สถานะการรักษา | [245:24603](https://www.figma.com/design/cOjwwHvgGURA3IEuls8RT5/PHCIS-%7C-CPOE?node-id=245-24603) | `resolveTreatmentStatusBadgeStyle` | ไม่ได้รักษา `grey` · เรื้อรังไม่หาย `red` · อยู่ระหว่างการรักษา `orange` · ควบคุมได้ `indigo` · สงบ/หายดีชั่วคราว `blue` · หายแล้ว `teal` |

`HistoryListForm` schema: `statusBadgeKind` = `likelihood` (allergy) or `treatmentStatus` (disease).

#### 6.3.3a Icon Position — Frame 1 (node 328:5706)

The Figma wrapper **Frame 1** (`328:5706`) is **16 wide × 20 tall** with a **vertical auto-layout** and **padding-top: 4 px**. This places the 16×16 checkbox box at `y = 4` within the frame — aligning the box's top edge with the first line of label text rather than centering it in the row.

```
Frame 1 (16×20)
├── padding-top: 4px
└── chk-box (16×16)    ← sits at y:4, top-aligned to label text
```

Implementation:
```css
.chk-box-wrap {
  width: 16px;
  flex-shrink: 0;
  padding-top: 4px;   /* matches Figma Frame 1 node 328:5706 */
}
```

HTML pattern (every checkbox row must use the wrapper):
```html
<div class="chk-row" onclick="chkToggleRow(this)">
  <div class="chk-box-wrap">
    <div class="chk-box">
      <svg class="chk-icon" ...><!-- checkmark SVG --></svg>
    </div>
  </div>
  <div class="chk-content">
    <span class="chk-label">Label text</span>
    <span class="chk-sub">Subtext</span>
  </div>
</div>
```

#### 6.3.4 States

| State | Box Fill | Box Stroke | Checkmark | Label Color | Add Card | Node |
|---|---|---|---|---|---|---|
| Default | `#FFFFFF` | 1.5px `#C9CDD0` | hidden | `#363B3F` | — | 269:7 |
| Hover | `#3EC3B2` | none | visible (white) | `#363B3F` | — | 2254:8411 |
| Selected | `#1B867D` | none | visible (white) | `#363B3F` | — | 269:14 |
| Selected-Hover | `#196C65` | none | visible (white) | `#363B3F` | — | — |
| **Selected-Add** | `#1B867D` | none | visible (white) | `#363B3F` | **visible** (`#F4FBF8` card) | 2498:6240 |
| Disabled-Unchecked | `#FFFFFF` | 1.5px `#E2E4E6` | hidden | `#ADB2B7` | — | 269:20 |
| Disabled-Checked | `#C9CDD0` | none | visible (white) | `#ADB2B7` | — | 269:27 |

> **Toggle behavior:** Click → Selected (checked, addon opens). Click again → Default (unchecked, addon closes). Hover (unchecked): `#3EC3B2`. Selected/Selected-Add: `#1B867D`. Selected-Hover: `#196C65`.

#### 6.3.5 CSS Implementation

```css
/* Checkbox wrapper — icon position fix per Figma Frame 1 (node 328:5706) */
.chk-box-wrap {
  width: 16px;
  flex-shrink: 0;
  padding-top: 4px;     /* positions box at y:4 — aligns with label text baseline */
}

/* Checkbox box */
.chk-box {
  width: 16px; height: 16px; border-radius: 4px;
  background: #FFFFFF;
  border: 1.5px solid var(--border-switch-default);   /* #C9CDD0 */
  display: flex; align-items: center; justify-content: center;
  transition: background .15s, border-color .15s; box-sizing: border-box;
}
.chk-icon { opacity: 0; pointer-events: none; flex-shrink: 0; transition: opacity .1s; }

/* Hover — unchecked box hovered: surface/switch/hover → teal/300 → #3EC3B2 */
.chk-box:not(.disabled):hover,
.chk-row:not(.disabled):hover .chk-box:not(.checked) {
  background: var(--surface-switch-hover);   /* #3EC3B2 — surface/switch/hover → teal/300 */
  border-color: transparent;
}
.chk-box:not(.disabled):hover .chk-icon,
.chk-row:not(.disabled):hover .chk-box:not(.checked) .chk-icon { opacity: 1; }

/* Selected-Hover — checked box hovered: surface/switch/on-hover → teal/700 → #196C65 */
.chk-box.checked:not(.disabled):hover,
.chk-row:not(.disabled):hover .chk-box.checked:not(.disabled) {
  background: var(--surface-switch-on-hover);   /* #196C65 — surface/switch/on-hover → teal/700 */
}

/* Selected */
.chk-box.checked { background: var(--surface-switch-on); border-color: transparent; }
.chk-box.checked .chk-icon { opacity: 1; }

/* Disabled-Unchecked */
.chk-box.disabled { background: #FFFFFF; border-color: var(--border-switch-disabled); cursor: not-allowed; }

/* Disabled-Checked */
.chk-box.disabled.checked {
  background: var(--surface-disabled-secondary);   /* #C9CDD0 */
  border-color: transparent;
}
.chk-box.disabled.checked .chk-icon { opacity: 1; }

/* Row layout */
.chk-row { display: flex; align-items: flex-start; gap: 8px; cursor: pointer; }
.chk-content { display: flex; flex-direction: column; flex: 1; }

/* Checkbox Button — Figma 2254:8344 (React: CheckboxButton.jsx) */
.chk-btn {
  padding: 8px 16px; border-radius: 8px; min-height: 56px;
  background: var(--surface-neutralbutton-quinary);
  border: var(--dim-stroke-100) solid var(--border-neutralbutton-tertiary);
  transition: border-color 120ms ease, box-shadow 120ms ease;
}
.chk-btn:not(.disabled):hover {
  border: var(--dim-stroke-100) solid var(--border-brandprimarybutton-tertiary);
  box-shadow: var(--shadow-brand-drop-bottom-200);
}
.chk-btn.checked:not(.disabled) {
  border: var(--dim-stroke-200) solid var(--border-brandprimarybutton-default);
  box-shadow: var(--shadow-brand-top-200);
}
.chk-btn.no-border { min-height: 40px; border: none; box-shadow: none; }

/* Selected-Add expandable card (node 2498:6240) */
/* --surface-card-brand100 → surface/card/brand-100 → primary/50 = #f4fbf8 (Figma 2498:6269) */
/* ⚠️ token name "brand-100" resolves to primary/50 (not primary/100) — verified via get_variable_defs */
.chk-addon {
  display: none; margin-top: 12px;
  background: var(--surface-card-brand100);   /* surface/card/brand-100 → primary/50 → #f4fbf8 */
  border-radius: 12px; padding: 16px;
  gap: 12px; flex-direction: column;
}
.chk-addon.open { display: flex; }

/* Selected-Add "เพิ่ม" button */
.chk-addon-btn {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 6px 12px; border-radius: 8px; cursor: pointer;
  border: 1px solid var(--border-brandprimary-tertiary);   /* #C5EDDA */
  background: transparent; font-size: 13px;
  color: var(--text-brandprimary-default);   /* #007549 */
}
```

JavaScript for toggling states:
```javascript
function chkToggleRow(row) {
  if (row.classList.contains("disabled")) return;
  var box = row.querySelector(".chk-box");
  if (!box || box.classList.contains("disabled")) return;
  box.classList.toggle("checked");
}

// State=Selected, Type=Other: toggle → Selected (checked + addon open) / Default (unchecked + addon closed)
function chkToggleAddon(row) {
  if (row.classList.contains("disabled")) return;
  var box = row.querySelector(".chk-box");
  if (!box || box.classList.contains("disabled")) return;
  // Toggle: click → Selected (checked + addon open); click again → Default (unchecked + addon closed)
  box.classList.toggle("checked");
  var addon = row.querySelector(".chk-addon");
  if (addon) addon.classList.toggle("open", box.classList.contains("checked"));
}
```

#### 6.3.6 Token Alias Chain

```
surface/switch/default              → grey/50              → #FFFFFF
border/switch/default               → neutral/300          → #C9CDD0
surface/switch/on                   → teal/600             → #1B867D    ← Selected + Selected-Add
surface/switch/hover                → teal/300             → #3EC3B2    ← Hover (unchecked hovered)
surface/switch/on-hover             → teal/700             → #196C65    ← Selected-Hover
border/switch/disabled              → neutral/200           → #E2E4E6
surface/disabled/secondary-hover    → neutral/300          → #C9CDD0    ← Disabled-Checked
text/brandSecondary/on-default      → grey/50              → #FFFFFF    ← checkmark stroke
text/content/default                → neutral/800          → #363B3F
text/content/tertiary               → neutral/500          → #858C92
text/content/quaternary             → neutral/400          → #ADB2B7
surface/neutralButton/quinary       → grey/50              → #FFFFFF    ← Button Default bg
border/neutralButton/tertiary       → neutral/200           → #E2E4E6   ← Button border
surface/neutralButton/quaternary    → neutral/50            → #F9FBFB   ← Button Hover bg
surface/card/brand-100              → primary/50 → emerald/50  → #F4FBF8  ← Selected-Add card bg
border/brandPrimaryButton/tertiary  → primary/200 → emerald/200 → #C5EDDA  ← Add button border
text/brandPrimaryButton/default     → primary/700 → emerald/700 → #007549  ← Add button text/icon
surface/brandPrimaryButton/tertiary → primary/100 → emerald/100 → #E2F3EB  ← Icon button bg (trash)
text/topic/tertiary-title           → neutral/800               → #363B3F  ← Group card title
surface/colorfulbadge/grey          → neutral/50               → #F5F7F9  ← Badge bg (Grey)
text/colorfulbadge/grey             → neutral/700               → #4D5358  ← Badge text (Grey)
surface/input/default               → white                     → #FFFFFF  ← InputField bg
border/input/default                → neutral/200               → #E2E4E6  ← InputField border
text/input/default                  → neutral/400               → #ADB2B7  ← InputField placeholder
text/input/require                  → red/600                   → #DC2626  ← InputField required *
icon/input/default                  → neutral/400               → #ADB2B7  ← InputField leading icon
```

---

### 6.4 Date Picker

**Node:** `341-6345` · Figma library: **MIH Design System Foundation / Semantic**

The Date Picker reuses `input` tokens for the field and `calendar` tokens for the popup.

#### 6.4.1 Component Properties (Figma node 341-6345)

| Property | Values |
|----------|--------|
| Size | `Large` (40px) · `Medium` (36px) |
| State | `Default` · `Focus` · `Open` · `Filled` · `Error` · `Disabled` |

#### 6.4.2 Input Field — Color Tokens

| Figma Semantic Token | CSS Variable | Hex | Primitive | State |
|----------------------|-------------|-----|-----------|-------|
| `surface/input/default` | `--surface-input-default` | `#FFFFFF` | white | Default bg |
| `border/input/default` | `--border-input-default` | `#E2E4E6` | neutral-200 | Default border |
| `text/input/default` | `--text-input-default` | `#ADB2B7` | neutral-400 | Placeholder |
| `border/input/hover` | `--border-input-hover` | `#08A768` | primary/600 | Focus border |
| `focus-ring/input/hover` | `--focus-ring-input-hover` | `rgba(8,167,104,.14)` | primary/600 14% | Focus ring |
| `border/input/typing` | `--border-input-typing` | `#08A768` | primary/600 | Open border |
| `focus-ring/input/typing` | `--focus-ring-input-typing` | `rgba(8,167,104,.14)` | primary/600 14% | Open ring |
| `border/input/filled` | `--border-input-filled` | `#E2E4E6` | neutral-200 | Filled border |
| `text/input/filled` | `--text-input-filled` | `#4D5358` | neutral-700 | Filled value text |
| `border/input/error` | `--border-input-error` | `#DC2626` | red-600 | Error border |
| `border/input/disabled` | `--border-input-disabled` | `#E2E4E6` | neutral-200 | Disabled border |
| `surface/input/disabled` | `--surface-input-disabled` | `#F4F5F5` | neutral-100 | Disabled bg |
| `text/input/disabled` | `--text-input-disabled` | `#ADB2B7` | neutral-400 | Disabled text — **empty fields only** (see rule below) |
| `border/input/hover` | `--border-input-hover` | `#08A768` | primary/600 | Hover border **(2px)** + Focus/Open border (2px) |
| `focus-ring/input/hover` | `--focus-ring-input-hover` | `rgba(8,167,104,.14)` | primary/600 14% | Hover ring + Focus ring (4px) |
| `icon/input/hover` | `--icon-input-hover` | `#08A768` | primary/600 | Calendar + chevron icon on hover |
| `border/input/error` | `--border-input-error` | `#DC2626` | red-600 | Error border **(2px)** |
| `focus-ring/input/error` | `--focus-ring-input-error` | `rgba(220,38,38,.12)` | red-600 12% | Error ring (4px) |

**Disabled + filled → use the filled text colour.** A control that is disabled but already
carries a value must render that value in `--text-input-filled` (`#4D5358`), not
`--text-input-disabled` (`#ADB2B7`). Disabled means "you cannot change this", not "this is
unimportant" — a read-only field such as *วันเวลาที่สั่งนัด* or a locked *หน่วยงานที่สั่งนัด* is
often the field the user most needs to read, and it must match the weight of the editable
fields beside it. `--text-input-disabled` stays reserved for a disabled field that is *empty*
(its placeholder). This is resolved centrally — `resolveBox` in
[`src/components/forms/_inputBox.jsx`](src/components/forms/_inputBox.jsx) (InputField ·
Textarea · VoiceTextarea) and `resolveTrigger` in
[`src/components/forms/Select.jsx`](src/components/forms/Select.jsx) — so every control picks
it up without a per-page override. Borders, background and icons keep their disabled tokens.

#### 6.4.3 Calendar Popup — Color Tokens

| Figma Semantic Token | CSS Variable | Hex | Primitive | Usage |
|----------------------|-------------|-----|-----------|-------|
| `surface/calendar/default` | `--surface-calendar-bg` | `#FFFFFF` | white | Popup bg |
| `border/dropdown/default` | `--border-calendar` | `#EBEBEB` | neutral-100 | Popup border |
| `text/calendar/month` | `--text-calendar-month` | `#4D5358` | neutral-700 | Month/year label |
| `text/neutral/secondary` | `--text-calendar-dow` | `#636B72` | neutral-500 | Day-of-week headers |
| `text/neutral/default` | `--text-calendar-default` | `#1A1E23` | neutral-900 | Regular day numbers |
| `surface/calendar/hover` | `--surface-calendar-hover` | `#E2F3EB` | emerald-100 | Day hover bg |
| `text/calendar/hover` | `--text-calendar-hover` | `#007549` | emerald-700 | Day hover text |
| `surface/calendar/current` | `--surface-calendar-current` | `#E2F3EB` | emerald-100 | Today bg |
| `text/calendar/current` | `--text-calendar-current` | `#007549` | emerald-700 | Today text |
| `surface/calendar/selected` | `--surface-calendar-selected` | `#007549` | emerald-700 | Selected day bg |
| `text/calendar/selected` | `--text-calendar-selected` | `#FFFFFF` | white | Selected day text |
| `text/calendar/disabled` | `--text-calendar-disabled` | `#CBD5E1` | slate-200 | Out-of-month day text |

#### 6.4.4 State Behavior

| State | Border | Background | Text | Ring |
|-------|--------|------------|------|------|
| Default | `#E2E4E6` 1px | `#FFFFFF` | `#ADB2B7` (placeholder) | — |
| Hover | `#08A768` **2px** | `#FFFFFF` | `#ADB2B7` · icon → `#08A768` | `rgba(8,167,104,.14)` 4px |
| Focus (hover) | `#08A768` 2px | `#FFFFFF` | `#ADB2B7` | `rgba(8,167,104,.14)` 4px |
| Open (calendar visible) | `#08A768` 2px | `#FFFFFF` | `#ADB2B7` | `rgba(8,167,104,.14)` 4px |
| Filled (date selected) | `#E2E4E6` 1px | `#FFFFFF` | `#4D5358` (value) | — |
| Error | `#DC2626` **2px** | `#FFFFFF` | `#ADB2B7` | `rgba(220,38,38,.12)` 4px |
| Disabled | `#E2E4E6` 1px | `#F4F5F5` | `#ADB2B7` | — |

> **Behavior:** Mouse-over → Hover (border + icons turn `#08A768`, no ring). Click input → Open state, calendar popup appears. Click a day → Filled state, popup closes. Chevron icon rotates 180° when open. Click outside → closes popup, returns to Filled (if date selected) or Default.

#### 6.4.5 CSS Implementation

```css
/* Input field */
.dp2-inp {
  display: flex; align-items: center; gap: 8px; height: 40px; padding: 8px 16px;
  border-radius: 8px; border: 1px solid var(--border-input-default);
  background: var(--surface-input-default); cursor: pointer;
}
.dp2-inp.md { height: 36px; padding: 6px 12px; }

/* States */
/* Hover — 2px green border + ring + green icons (Figma 3070-4595) */
.dp2-inp:hover:not(.dp2-open):not(.dp2-focus):not(.dp2-error):not(.dp2-disabled) {
  border: 2px solid var(--border-input-hover);              /* #08A768, 2px */
  box-shadow: 0 0 0 4px var(--focus-ring-input-hover);      /* rgba(8,167,104,.14) */
}
.dp2-inp:hover:not(.dp2-open):not(.dp2-focus):not(.dp2-error):not(.dp2-disabled) .dp2-cal-icon,
.dp2-inp:hover:not(.dp2-open):not(.dp2-focus):not(.dp2-error):not(.dp2-disabled) .dp2-chev {
  color: var(--icon-input-hover);   /* #08A768 */
}
.dp2-inp.dp2-focus  { border: 2px solid var(--border-input-hover);  box-shadow: 0 0 0 4px var(--focus-ring-input-hover);  }
.dp2-inp.dp2-open   { border: 2px solid var(--border-input-typing); box-shadow: 0 0 0 4px var(--focus-ring-input-typing); }
.dp2-inp.dp2-filled .dp2-ph { color: var(--text-input-filled); }
/* Error — 2px red border + red ring (Figma 341-6364) */
.dp2-inp.dp2-error  { border: 2px solid var(--border-input-error);      /* #DC2626 */
  box-shadow: 0 0 0 4px var(--focus-ring-input-error); }                 /* rgba(220,38,38,.12) */
.dp2-inp.dp2-disabled { background: var(--surface-input-disabled); border-color: var(--border-input-disabled); pointer-events: none; }

/* Calendar popup */
.dp2-popup { display: none; position: absolute; top: calc(100% + 6px); z-index: 300;
  background: var(--surface-calendar-bg); border: 1px solid var(--border-calendar);
  border-radius: 16px; width: 320px; padding: 16px; flex-direction: column; gap: 4px; }
.dp2-popup.open { display: flex; }

/* Day cells */
.dp2-day { display: flex; align-items: center; justify-content: center;
  height: 40px; border-radius: 9999px; cursor: pointer;
  background: var(--surface-calendar-bg); color: var(--text-calendar-default); }
.dp2-day:hover:not(.dp2-other) { background: var(--surface-calendar-hover); color: var(--text-calendar-hover); }
.dp2-day.dp2-cur  { background: var(--surface-calendar-current);  color: var(--text-calendar-current);  }
.dp2-day.dp2-sel  { background: var(--surface-calendar-selected); color: var(--text-calendar-selected); }
.dp2-day.dp2-other { color: var(--text-calendar-disabled); cursor: default; }
```

#### 6.4.6 Dimension Tokens (Figma node 341-6345)

| CSS Variable | Design Token | Value | Role |
|---|---|---|---|
| `--dp-height-lg` | `dimension/size/1000` | **40px** | Height — Large |
| `--dp-height-md` | `dimension/size/900` | **36px** | Height — Medium |
| `--dp-pad-x` | `dimension/space/400` | **16px** | Padding X (Large) |
| `--dp-pad-x-md` | `dimension/space/300` | **12px** | Padding X (Medium) |
| `--dp-gap` | `dimension/space/200` | **8px** | Gap (icon ↔ text) |
| `--dp-radius` | `dimension/radius/200` | **8px** | Input border-radius |
| `--dp-popup-radius` | `dimension/radius/400` | **16px** | Calendar popup radius |
| `--dp-popup-width` | — | **320px** | Calendar popup width |
| `--dp-day-size` | — | **40px** | Day cell height (border-radius: 9999px) |

#### 6.4.7 Token Alias Chain

```
--border-input-hover      ← border/input/hover      ← primary/600   ← #08A768   (Hover 2px · Focus 2px · Open 2px)
--focus-ring-input-hover  ← focus-ring/input/hover  ← primary/600 14% ← rgba(8,167,104,.14)  (Hover + Focus ring)
--border-input-error      ← border/input/error      ← red-600       ← #DC2626   (2px)
--focus-ring-input-error  ← focus-ring/input/error  ← red-600 12%   ← rgba(220,38,38,.12)    (Error ring)
--icon-input-hover     ← icon/input/hover     ← primary/600    ← #08A768  (icons on Hover)
--border-input-typing  ← border/input/typing  ← primary/600    ← #08A768
--surface-input-disabled ← surface/input/disabled ← neutral-100 ← #F4F5F5
--surface-calendar-selected ← surface/calendar/selected ← emerald-700 ← #007549
--surface-calendar-current  ← surface/calendar/current  ← emerald-100 ← #E2F3EB
```

---

### 6.5 Stepper

**Node:** `2709-6338` · btn− `3070-4643` · btn+ `3070-4644` · Figma library: **MIH Design System Foundation / Semantic**

#### 6.5.1 Component Properties (Figma node 2709-6338)

| Property | Values |
|----------|--------|
| Size | `Large` (40px) · `Small` (36px) |
| State | `Default` · `Hover` · `Focus` · `Active` · `Min` · `Max` · `Disabled` |
| Behavior | Click ± buttons **or** type directly in the number field |

#### 6.5.2 Color Tokens

| Role | CSS Variable | Figma Token | Hex | Primitive |
|------|-------------|-------------|-----|-----------|
| Container bg | `--surface-stepper-default` | `surface/input/default` | `#FFFFFF` | white |
| Border — Default | `--border-stepper-default` | `border/input/default` | `#E2E4E6` | neutral-200 |
| Border — Hover | `--border-stepper-hover` | `border/input/hover` | `#08A768` | primary/600 |
| Border — Focus | `--border-stepper-focus` | `border/input/typing` | `#08A768` | primary/600 |
| Focus ring | `--focus-ring-stepper` | `focus-ring/input` | `rgba(8,167,104,.14)` | primary/600 14% |
| Button bg | `--surface-stepper-btn` | `surface/brandprimarybutton/quinary` | `#FFFFFF` | white |
| Button bg — Hover | `--surface-stepper-btn-hover` | `surface/brandprimarybutton/quaternary-hover` | `#E2F3EB` | emerald-100 |
| Button bg — Active/Pressed | `--surface-stepper-btn-active` | `surface/brandprimarybutton/quaternary-hover` | `#E2F3EB` | emerald-100 |
| Button icon | `--icon-stepper-btn` | `icon/neutral/secondary` | `#636B72` | neutral-500 |
| Button icon — Hover | `--icon-stepper-btn-hover` | `icon/neutral/default` | `#007549` | emerald-700 |
| Button icon — Active/Pressed | `--icon-stepper-btn-active` | `icon/stepper/btn-active` | `#007549` | emerald-700 |
| Button icon — Disabled | `--icon-stepper-btn-dis` | `icon/input/disabled` | `#ADB2B7` | neutral-400 |
| Disabled bg | `--surface-stepper-disabled` | `surface/input/disabled` | `#F4F5F5` | neutral-100 |
| Disabled border | `--border-stepper-disabled` | `border/input/disabled` | `#E2E4E6` | neutral-200 |
| Disabled text/icon | `--text-stepper-disabled` | `text/input/disabled` | `#ADB2B7` | neutral-400 |

#### 6.5.3 Button States (nodes 3070-4643 · 3070-4644)

| State | Button bg | Button icon | Description |
|-------|-----------|-------------|-------------|
| Default | `#F9FBFB` | `#636B72` | Resting state |
| Hover | `#F4FBF8` | `#007549` | Mouse over button |
| Active / Pressed | `#E2F3EB` | `#007549` | Mouse held down |
| Disabled | transparent | `#ADB2B7` | At min (−) or max (+) boundary |

#### 6.5.4 State Behavior

| State | Container border | Width token | Focus ring |
|-------|-----------------|-------------|------------|
| Default | `border/input/default` = `#E2E4E6` | `dimension/stroke/150` = **1.5 px** | — |
| **Hover** | `border/input/hover` = `#08A768` (brand) | **`dimension/stroke/200` = 2 px** `--fi-stroke-active` | — |
| **Focus** | `border/input/hover` = `#08A768` (brand) | **`dimension/stroke/200` = 2 px** `--fi-stroke-active` | `rgba(primary/600, .14)` 4px |
| Active / Pressed | — | — | — |
| Disabled | `border/input/disabled` = `#E2E4E6` · bg `#F4F5F5` | 1.5 px | — |

> **Behavior:** User can click `−`/`+` buttons to decrement/increment, or click the number field and type directly. Value is clamped to `[min, max]` on blur. The `−` button disables automatically at `min`, `+` at `max`.

#### 6.5.5 CSS Implementation

```css
.st2 {
  display: inline-flex; align-items: stretch;
  border: 1.5px solid var(--border-stepper-default);
  border-radius: 8px; overflow: hidden;
  background: var(--surface-stepper-default);
  height: 40px; min-width: 160px;
}
.st2.sm { height: 36px; min-width: 140px; }
/* Hover: border/input/hover + dimension/stroke/200 = 2px (--fi-stroke-active) */
.st2:hover:not(.st2-dis)  { border-color: var(--border-stepper-hover);
                             border-width: var(--fi-stroke-active, 2px); }
.st2.st2-focus            { border-color: var(--border-stepper-focus);
                             border-width: var(--fi-stroke-active, 2px);
                             box-shadow: 0 0 0 4px var(--focus-ring-stepper); }
.st2.st2-dis              { background: var(--surface-stepper-disabled);
                             border-color: var(--border-stepper-disabled); }
/* ± Buttons */
.st2-btn {
  display: inline-flex; align-items: center; justify-content: center;
  width: 40px; flex-shrink: 0; border: none; cursor: pointer;
  background: var(--surface-stepper-btn); color: var(--icon-stepper-btn);
}
.st2-btn:hover:not(:disabled)  { background: var(--surface-stepper-btn-hover);
                                   color: var(--icon-stepper-btn-hover); }
.st2-btn:active:not(:disabled) { background: var(--surface-stepper-btn-active);
                                   color: var(--icon-stepper-btn-active); }
.st2-btn:disabled { cursor: not-allowed; color: var(--icon-stepper-btn-dis); }
/* Divider lines */
.st2-div { width: 1px; background: var(--border-stepper-default); align-self: stretch; }
/* Input field */
.st2-inp {
  flex: 1; border: none; background: transparent; outline: none;
  text-align: center; font-size: 16px; font-weight: 500; color: var(--n900);
  -moz-appearance: textfield; appearance: textfield;
}
.st2-inp::-webkit-inner-spin-button,
.st2-inp::-webkit-outer-spin-button { -webkit-appearance: none; }
```

#### 6.5.6 Dimension Tokens (Figma node 2709-6338)

| Property | Design Token | Value | Role |
|----------|-------------|-------|------|
| Height — Large | `dimension/size/1000` | **40px** | Large variant |
| Height — Small | `dimension/size/900` | **36px** | Small variant |
| Button width | — | **40px** | ± button hit area |
| Border radius | `dimension/radius/200` | **8px** | Container corner |
| Border default | `dimension/stroke/150` | **1.5px** | Outer border |

#### 6.5.7 Token Alias Chain

```
--border-stepper-default      ← border/input/default       ← neutral-200       ← #E2E4E6
--border-stepper-hover        ← border/input/hover         ← primary/600       ← #08A768
--border-stepper-focus        ← border/input/typing        ← primary/600       ← #08A768
--focus-ring-stepper          ← focus-ring/input           ← primary/600 14%   ← rgba(8,167,104,.14)
--surface-stepper-btn         ← surface/brandprimarybutton/quinary           ← white       ← #FFFFFF
--surface-stepper-btn-hover   ← surface/brandprimarybutton/quaternary-hover   ← emerald-100 ← #E2F3EB
--surface-stepper-btn-active  ← surface/brandprimarybutton/quaternary-hover    ← emerald-100 ← #E2F3EB
--icon-stepper-btn-hover      ← icon/neutral/default       ← emerald-700       ← #007549
--icon-stepper-btn-active     ← icon/stepper/btn-active    ← emerald-700       ← #007549
--icon-stepper-btn-dis        ← icon/input/disabled        ← neutral-400       ← #ADB2B7
--surface-stepper-disabled    ← surface/input/disabled     ← neutral-100       ← #F4F5F5
```

---

### 6.6 Switch

**Node:** `368-31` · Figma library: **MIH Design System Foundation / Semantic**

> **Important:** Switch track uses the **muted-teal** palette (not emerald) for the On states. All `surface/switch/*` tokens are in the MIH Semantic collection.

#### 6.6.1 Component Properties (Figma node 368-31)

| Property | Values |
|----------|--------|
| State | `Default` · `Hover` · `Disabled` |
| Checked | `true` · `false` |
| Behavior | Click to toggle · CSS `:hover` applies track color change automatically |

#### 6.6.2 Color Tokens

| Role | CSS Variable | Figma Token | Hex | Primitive |
|------|-------------|-------------|-----|-----------|
| Track Off | `--surface-switch-off` | `surface/switch/off` | `#E2E4E6` | neutral-200 |
| Track Off Hover | `--surface-switch-off-hover` | `surface/switch/off-hover` | `#C9CDD0` | neutral-300 |
| Track On | `--surface-switch-on` | `surface/switch/on` | `#1B867D` | muted-teal/600 |
| Track On Hover | `--surface-switch-on-hover` | `surface/switch/on-hover` | `#196C65` | muted-teal/700 |
| Track Disabled | `--surface-switch-disabled` | `surface/switch/disabled` | `#E2E4E6` | neutral-200 |
| Thumb Default | `--surface-switch-thumb` | `surface/switch/icon` | `#FFFFFF` | white · box-shadow |
| Thumb Disabled | `--surface-switch-thumb-disabled` | `surface/switch/icon-disabled` | `#F4F5F5` | neutral-100 · no shadow |

#### 6.6.3 State Behavior

| State | checked | Track color | Thumb bg | Thumb shadow |
|-------|---------|-------------|----------|--------------|
| Default Off | `false` | `#E2E4E6` | `#FFFFFF` | `0 2px 4px rgba(0,0,0,.20)` |
| Hover Off | `false` | `#C9CDD0` | `#FFFFFF` | `0 2px 4px rgba(0,0,0,.20)` |
| Default On | `true` | `#1B867D` | `#FFFFFF` | `0 2px 4px rgba(0,0,0,.20)` |
| Hover On | `true` | `#196C65` | `#FFFFFF` | `0 2px 4px rgba(0,0,0,.20)` |
| Disabled Off | `false` | `#E2E4E6` | `#F4F5F5` | none |
| Disabled On | `true` | `#E2E4E6` | `#F4F5F5` | none |

> **Behavior:** Click toggles `.on` class on the track. Thumb slides with CSS `transition: left .18s ease`. Hover state is handled entirely by CSS `:hover` pseudo-class — no JS needed.

#### 6.6.4 CSS Implementation

```css
.sw-track {
  width: 44px; height: 24px; border-radius: 12px;
  background: var(--surface-switch-off);
  position: relative; cursor: pointer;
  transition: background .18s ease;
}
.sw-track.on                              { background: var(--surface-switch-on); }
.sw-track:not(.disabled):not(.on):hover  { background: var(--surface-switch-off-hover); }
.sw-track:not(.disabled).on:hover        { background: var(--surface-switch-on-hover); }
.sw-track.disabled { background: var(--surface-switch-disabled); cursor: not-allowed; pointer-events: none; }

.sw-thumb {
  position: absolute; top: 2px; left: 2px;
  width: 20px; height: 20px; border-radius: 50%;
  background: var(--surface-switch-thumb);
  box-shadow: 0 2px 4px rgba(0,0,0,.20), 0 1px 2px rgba(0,0,0,.12);
  transition: left .18s ease;
}
.sw-track.on .sw-thumb       { left: 22px; }
.sw-track.disabled .sw-thumb { background: var(--surface-switch-thumb-disabled); box-shadow: none; }
```

```javascript
function toggleSwitch(el) {
  if (el.classList.contains('disabled')) return;
  el.classList.toggle('on');
}
```

#### 6.6.5 Dimension Tokens (Figma node 368-31)

| CSS Variable | Dimension Token | Value | Role |
|-------------|----------------|-------|------|
| `--sw-track-w` | — | **44 px** | Track width |
| `--sw-track-h` | — | **24 px** | Track height |
| `--sw-thumb-size` | `dimension/size/500` | **20 px** | Thumb diameter |
| `--sw-radius` | `dimension/radius/pill` | **12 px** | Track border-radius (fully rounded) |

#### 6.6.6 Token Alias Chain

```
--surface-switch-off            ← surface/switch/off           ← neutral-200    ← #E2E4E6
--surface-switch-off-hover      ← surface/switch/off-hover     ← neutral-300    ← #C9CDD0
--surface-switch-on             ← surface/switch/on            ← muted-teal/600 ← #1B867D
--surface-switch-on-hover       ← surface/switch/on-hover      ← muted-teal/700 ← #196C65
--surface-switch-disabled       ← surface/switch/disabled      ← neutral-200    ← #E2E4E6
--surface-switch-thumb          ← surface/switch/icon          ← white          ← #FFFFFF
--surface-switch-thumb-disabled ← surface/switch/icon-disabled ← neutral-100    ← #F4F5F5
```

---

### 6.7 Timeslot

**Nodes:** `2302-11352` (section) · `2302-11294` (Default) · `2402-4568` (Hover) · `2302-11302` (Selected) · `2302-11298` (Disabled) · `2302-11272` (3-col box) · `2643-4883` (5-col box)

> **Figma-verified** via `get_design_context` per state. All tokens are dedicated `timeslot` semantic tokens.

#### 6.7.1 Color Tokens

| State | CSS Variable | Semantic Alias | Primitive / Brand | Emerald Hex | Brand-responsive |
|-------|-------------|---------------|-------------------|-------------|-----------------|
| Default — bg | `--surface-timeslot-default` | `surface/timeslot/default` | `grey/50` (fixed white) | `#FFFFFF` | — |
| Default — border | `--border-timeslot-slot` | `border/brandprimary/quaternary` | `primary/100` = `var(--em100)` | `#E2F3EB` | ✓ |
| Default — text | `--text-timeslot-default` | `text/timeslot/default` | `neutral/700` (fixed) | `#4D5358` | — |
| Default — shadow | `--shadow-timeslot-default` | `primaryshadow/600` | `p[600] @ 8%` | `rgba(8,167,104,.08)` | ✓ |
| Hover — bg | `--surface-timeslot-hover` | `surface/timeslot/hover` | `primary/100` = `var(--em100)` | `#E2F3EB` | ✓ |
| Hover — text | `--text-timeslot-hover` | `text/timeslot/hover` | `primary/700` = `var(--em700)` | `#007549` | ✓ |
| Selected — bg | `--surface-timeslot-selected` | `surface/timeslot/selected` | `primary/700` = `var(--em700)` | `#007549` | ✓ |
| Selected — text | `--text-timeslot-selected` | `text/timeslot/selected` | `grey/50` (fixed white) | `#FFFFFF` | — |
| Disabled — bg | `--surface-timeslot-disabled` | `surface/timeslot/disabled` | `primary/50` = `var(--em50)` | `#F4FBF8` | ✓ |
| Disabled — text | `--text-timeslot-disabled` | `text/timeslot/disabled` | `primary/100` = `var(--em100)` | `#E2F3EB` ⚠️ | ✓ |
| Box — bg | `--surface-timeslot-box` | `surface/timeslot/box` | `primary/50` = `var(--em50)` | `#F4FBF8` | ✓ |
| Box — border | `--border-timeslot-box` | `border/timeslot/default` | `primary/100` = `var(--em100)` | `#E2F3EB` | ✓ |

> ⚠️ **Disabled text is `primary/100`** — intentionally near-invisible on the `primary/50` background to indicate unavailability (Figma node `2302-11298`). Both follow the brand, so the contrast ratio is consistent across all brand modes.
>
> **Implementation note:** `--surface-timeslot-box` and `--surface-timeslot-disabled` **must** be in `setBrandTheme()` as `p[50]` — previously they were hardcoded to `#FAFCFA` (emerald-only) and never updated on brand switch.

#### 6.7.2 Border & Shadow Rules

| State | Border | Box Shadow |
|-------|--------|------------|
| Default | `1px solid #E2F3EB` | `0 1px 2px rgba(8,167,104,.08)` |
| Hover | none (`transparent`) | none |
| Selected | none (`transparent`) | none |
| Disabled | none (`transparent`) | none |

#### 6.7.3 Dimension Tokens

> Figma: h=40px · radius=8px · border=1px (Default only) · min-w=80px · max-w=256px · px=16px · py=8px

| Property | Dimension Token | Value |
|----------|----------------|-------|
| Height | `dimension/size/800` | **40px** |
| Min Width | — | **80px** |
| Max Width | — | **256px** |
| Border Radius | `dimension/radius/200` | **8px** |
| Padding X | `dimension/space/400` | **16px** |
| Padding Y | `dimension/space/200` | **8px** |
| Grid Gap | `dimension/space/300` | **12px** |
| Box Border Radius | `dimension/radius/400` | **12px** |
| Box Padding | `dimension/space/600` | **24px** |

#### 6.7.4 Typography

| Property | Token | Value |
|----------|-------|-------|
| Font Family | `typograpphy/family/ui` | Sarabun |
| Font Weight | `typograpphy/weight/medium` | 500 (Medium) |
| Font Size | `typograpphy/size/base` | 16px |
| Line Height | — | 1.5 |

#### 6.7.5 CSS Classes

```css
/* Base slot — Default state only has border + drop shadow */
.ts-slot {
  display: flex; align-items: center; justify-content: center;
  height: 40px; min-width: 80px; max-width: 256px; border-radius: 8px;
  border: 1px solid var(--border-timeslot-slot);
  background: var(--surface-timeslot-default);
  box-shadow: 0 1px 2px var(--shadow-timeslot-default);
  font-size: 16px; font-weight: 500; color: var(--text-timeslot-default);
  cursor: pointer; padding: 8px 16px;
  transition: background .12s, border-color .12s, box-shadow .12s, color .12s;
}
/* Hover — native CSS, no JS needed */
.ts-slot:hover:not(.selected):not(.disabled) {
  background: var(--surface-timeslot-hover);
  border-color: transparent; box-shadow: none;
  color: var(--text-timeslot-hover);
}
.ts-slot.selected {
  background: var(--surface-timeslot-selected);
  border-color: transparent; box-shadow: none;
  color: var(--text-timeslot-selected);
}
.ts-slot.disabled {
  background: var(--surface-timeslot-disabled);
  border-color: transparent; box-shadow: none;
  color: var(--text-timeslot-disabled); cursor: not-allowed;
}
/* Grid layouts */
.ts-grid { display: grid; gap: 12px; }
.ts-grid.col3 { grid-template-columns: repeat(3, 1fr); }
.ts-grid.col5 { grid-template-columns: repeat(5, 1fr); }
/* Container box */
.ts-box {
  background: var(--surface-timeslot-box);
  border: 1px solid var(--border-timeslot-box);
  border-radius: 12px; padding: 24px;
  display: flex; flex-direction: column; gap: 12px;
}
```

#### 6.7.6 HTML Structure

```html
<!-- Container box -->
<div class="ts-box">
  <!-- 3-column grid -->
  <div class="ts-grid col3">
    <div class="ts-slot" onclick="selectSlot(this)">08:00–08:30</div>
    <div class="ts-slot selected" onclick="selectSlot(this)">09:30–10:00</div>
    <div class="ts-slot disabled">11:00–11:30 (เต็ม)</div>
  </div>
</div>
```

#### 6.7.7 Alias Chain

```
Default slot border (brand-responsive):
  --border-timeslot-slot → border/brandprimary/quaternary → primary/100 → var(--em100)

Default drop shadow (brand-responsive):
  --shadow-timeslot-default → primaryshadow/600 → rgba(p[600], 0.08)

Hover bg + text (brand-responsive):
  --surface-timeslot-hover → surface/timeslot/hover → primary/100 → var(--em100)
  --text-timeslot-hover    → text/timeslot/hover    → primary/700 → var(--em700)

Selected bg (brand-responsive):
  --surface-timeslot-selected → surface/timeslot/selected → primary/700 → var(--em700)

Disabled bg (brand-responsive ← previously missing):
  --surface-timeslot-disabled → surface/timeslot/disabled → primary/50 → var(--em50)

Disabled text (brand-responsive, ⚠️ near-invisible by design):
  --text-timeslot-disabled → text/timeslot/disabled → primary/100 → var(--em100)

Box container bg (brand-responsive ← previously missing):
  --surface-timeslot-box → surface/timeslot/box → primary/50 → var(--em50)

Box container border (brand-responsive):
  --border-timeslot-box → border/timeslot/default → primary/100 → var(--em100)
```

---

### 6.8 Tooltip

**Nodes:** `365-35` (section) · `341-8800` (Top) · `341-8783` (Bottom) · `341-8794` (Left) · `341-8788` (Right)

> **Figma-verified** via `get_design_context` node 365-35. Uses dedicated `surface/tooltip/default` semantic token — **not** neutral as previously documented.

#### 6.8.1 Color Tokens

| CSS Variable | Semantic Alias | Hex | Primitive |
|-------------|---------------|-----|-----------|
| `--surface-tooltip` | `surface/tooltip/default` | `#007549` | emerald-700 |
| `--text-tooltip` | `color/neutral/50` | `#F9FBFB` | neutral-50 |
| `--shadow-tooltip` | `color/shadow/100` + `color/shadow/050` | — | Drop Shadow Bottom/400 |

#### 6.8.2 Shadow — Drop Shadow Bottom/400

| Layer | Token | CSS Value |
|-------|-------|-----------|
| Ambient | `color/shadow/100` | `0 16px 32px rgba(100,116,139,.15)` |
| Key | `color/shadow/050` | `0 4px 4px rgba(100,116,139,.10)` |

> **Brand variant — `Brand Drop Shadow Bottom/400`** (`--shadow-brand-drop-bottom-400`): same two-layer geometry but tinted with `PrimaryShadow/600 + /601` and a −4px spread — `0 4px 4px -4px var(--PrimaryShadow-600), 0 16px 32px -4px var(--PrimaryShadow-601)`. Used by the floating auth card on PHCIS-Signin "select" (node 42:8722). Brand-aware via `--PrimaryShadow-*`.

#### 6.8.3 Dimension Tokens

> Figma: w=160px · radius=8px · padding=8px · arrow=10×10px rotated 45° (14.14px diagonal) · trigger=36×36px

| Property | Token | Value |
|----------|-------|-------|
| Bubble Width | — | 160px |
| Bubble Radius | `dimension/space/200` | 8px |
| Bubble Padding | `dimension/space/200` | 8px (all sides) |
| Trigger Width/Height | `dimension/size/900` | 36px |
| Trigger Radius | `dimension/radius/200` | 8px |
| Arrow tip → trigger gap | `dimension/space/100` | **4px** (node 3070-4743) |
| CSS bubble offset | — | **11px** = 4px gap + 7px CSS arrow |
| Arrow (Figma) | — | 10×10px square rotated 45° |
| Arrow (CSS impl) | — | `border: 7px solid transparent` |

#### 6.8.4 Typography

| Property | Token | Value |
|----------|-------|-------|
| Font Family | `typograpphy/family/body` | Sarabun |
| Font Weight | `typograpphy/weight/regular` | 400 (Regular) |
| Font Size | `typograpphy/size/sm` | 14px |
| Line Height | — | 1.5 |
| Text Align | — | center |

#### 6.8.5 Positions & Arrow Rules

> **Gap:** `dimension/space/100` = **4px** จาก arrow tip ถึง trigger button (node `3070-4743`)  
> CSS offset = 4px gap + 7px CSS arrow height = **11px** (`calc(100% + 11px)`)  
> ⚠️ Demo containers ต้องใช้ `margin` (ไม่ใช่ `padding`) บน `.tt-host` — padding ทำให้ `100%` ขยายเกิน และ bubble ลอยไกลผิดปกติ

| Position | Bubble placement | Arrow direction | `::after` border rule |
|----------|-----------------|-----------------|----------------------|
| Top | `bottom: calc(100% + 11px)` | ▼ points down | `border-top-color: var(--surface-tooltip)` |
| Bottom | `top: calc(100% + 11px)` | ▲ points up | `border-bottom-color: var(--surface-tooltip)` |
| Left | `right: calc(100% + 11px)` | ▶ points right | `border-left-color: var(--surface-tooltip)` |
| Right | `left: calc(100% + 11px)` | ◀ points left | `border-right-color: var(--surface-tooltip)` |

#### 6.8.6 CSS Classes

```css
.tt-host { position: relative; display: inline-flex; align-items: center; justify-content: center; }
/* ⚠️ Demo spacing: use margin (not padding) on .tt-host to avoid inflating 100% */

.tt-bubble {
  position: absolute;
  background: var(--surface-tooltip);     /* surface/tooltip/default */
  color: var(--text-tooltip);             /* color/neutral/50 */
  font-size: 14px;                        /* typography/size/sm */
  font-weight: 400;                       /* typography/weight/regular */
  line-height: 1.5;
  padding: 8px;                           /* dimension/space/200 */
  border-radius: 8px;                     /* dimension/space/200 */
  width: 160px; text-align: center;
  pointer-events: none; z-index: 10;
  box-shadow: var(--shadow-tooltip);      /* Drop Shadow Bottom/400 */
  opacity: 0; transition: opacity .15s;
}
.tt-host:hover .tt-bubble { opacity: 1; }

/* Arrow — 7px CSS border-trick triangle */
.tt-bubble::after { content: ''; position: absolute; border: 7px solid transparent; }

/* Positions — offset = 4px visual gap + 7px arrow = 11px (node 3070-4743) */
.tt-bubble.top    { bottom: calc(100% + 11px); left: 50%; transform: translateX(-50%); }
.tt-bubble.top::after    { top: 100%;    left: 50%; transform: translateX(-50%);  border-top-color:    var(--surface-tooltip); }

.tt-bubble.bottom { top:    calc(100% + 11px); left: 50%; transform: translateX(-50%); }
.tt-bubble.bottom::after { bottom: 100%; left: 50%; transform: translateX(-50%); border-bottom-color: var(--surface-tooltip); }

.tt-bubble.left   { right: calc(100% + 11px); top: 50%; transform: translateY(-50%); }
.tt-bubble.left::after   { left: 100%;   top: 50%;  transform: translateY(-50%); border-left-color:   var(--surface-tooltip); }

.tt-bubble.right  { left:  calc(100% + 11px); top: 50%; transform: translateY(-50%); }
.tt-bubble.right::after  { right: 100%;  top: 50%;  transform: translateY(-50%); border-right-color:  var(--surface-tooltip); }
```

#### 6.8.7 Alias Chain

```
Background:
  --surface-tooltip
    └─ surface/tooltip/default
         └─ emerald-700 (#007549)   ← GREEN, not neutral-900

Text:
  --text-tooltip
    └─ color/neutral/50
         └─ #F9FBFB

Shadow:
  --shadow-tooltip
    ├─ color/shadow/100 → rgba(100,116,139,.15)  [0 16px 32px]
    └─ color/shadow/050 → rgba(100,116,139,.10)  [0 4px 4px]
```

---

### 6.9 Accordion

**Figma node:** `292-21` · Library: **MIH Design System Foundation**  
**CSS prefix:** `acc-` · **Behavior reference:** Tailwind UI Disclosure (click-to-toggle, single-open per group)

---

#### 6.9.1 Color Tokens

| CSS Variable | Figma Semantic Token | Hex | Primitive | Usage |
|---|---|---|---|---|
| `--surface-accordion-default` | `surface/accordion/collapsed` | `#FFFFFF` | white | Header bg — collapsed & expanded |
| `--surface-accordion-expanded` | `surface/accordion/expanded` | `#FFFFFF` | white | Header bg — open state |
| `--surface-accordion-disabled` | `surface/accordion/disabled` | `#F4F5F5` | neutral-100 | Header bg — disabled |
| `--border-accordion` | `border/accordion/default` | `#C9CDD0` | neutral-300 | Outer border |
| `--border-accordion-line` | `border/accordion/line` | `#E2E4E6` | neutral-200 | Divider (header ↔ body) |
| `--text-accordion-title` | `text/accordion/collapsed` | `#4D5358` | neutral-700 | Title — default & expanded |
| `--text-accordion-title-disabled` | `text/accordion/disabled` | `#ADB2B7` | neutral-400 | Title — disabled |
| `--text-accordion-content` | `text/accordion/content` | `#636B72` | neutral-600 | Body content text |
| `--icon-accordion-chevron` | `icon/accordion/collapsed` | `#ADB2B7` | neutral-400 | Chevron — all active states |
| `--icon-accordion-chevron-disabled` | `icon/accordion/disabled` | `#ADB2B7` | neutral-400 | Chevron — disabled |

> **Note:** Chevron is neutral-400 in all states (collapsed, expanded, disabled). Expanded header background stays `#FFFFFF` — no tint on open.

---

#### 6.9.2 Dimension Tokens

> Figma node 292-21: header padding 16px · gap 8px · content padding 12px 16px 16px · radius 8px · border 1px · chevron 24×24 px

| CSS Variable | Dimension Token | Value | Role |
|---|---|---|---|
| `--acc-radius` | `dimension/radius/200` | **8 px** | Border radius (item & group corners) |
| `--acc-stroke` | `dimension/stroke/100` | **1 px** | Outer border & divider width |
| `--acc-header-pad` | `dimension/space/400` | **16 px** | Header padding (all sides) |
| `--acc-header-gap` | `dimension/space/200` | **8 px** | Gap between chevron and title |
| `--acc-content-pt` | `dimension/space/300` | **12 px** | Content top padding |
| `--acc-content-px` | `dimension/space/400` | **16 px** | Content horizontal padding |
| `--acc-content-pb` | `dimension/space/400` | **16 px** | Content bottom padding |
| `--acc-chevron-size` | `dimension/icon/lg` | **24 px** | Chevron icon width & height |

---

#### 6.9.3 Typography

| Element | Font Size | Weight | Line-height | Token |
|---|---|---|---|---|
| Title | 16 px | Regular (400) | 1.5 (24 px) | `typography/body/md` |
| Body content | 14 px | Regular (400) | 1.5 (21 px) | `typography/body/sm` |

---

#### 6.9.4 CSS Classes

```css
/* ── Item (standalone) ─────────────────────────────── */
.acc-item {
  border: 1px solid var(--border-accordion);
  border-radius: 8px;
  overflow: hidden;
  background: var(--surface-accordion-default);
  width: 100%;
  max-width: 480px;
}

/* ── Group (stacked, shared borders) ───────────────── */
.acc-group { display: flex; flex-direction: column; max-width: 480px; }
.acc-group .acc-item { border-radius: 0; border-bottom: none; max-width: none; }
.acc-group .acc-item:first-child { border-radius: 8px 8px 0 0; }
.acc-group .acc-item:last-child  { border-radius: 0 0 8px 8px; border-bottom: 1px solid var(--border-accordion); }

/* ── Header ─────────────────────────────────────────── */
.acc-header {
  display: flex; align-items: center; gap: 8px;
  padding: 16px; cursor: pointer;
  background: var(--surface-accordion-default);
}
.acc-item.disabled .acc-header {
  background: var(--surface-accordion-disabled);
  cursor: not-allowed;
}

/* ── Title ──────────────────────────────────────────── */
.acc-title {
  flex: 1; min-width: 0;
  font-size: 16px; font-weight: 400; line-height: 1.5;
  color: var(--text-accordion-title);
}
.acc-item.disabled .acc-title { color: var(--text-accordion-title-disabled); }

/* ── Chevron ────────────────────────────────────────── */
.acc-chevron {
  width: 24px; height: 24px;
  color: var(--icon-accordion-chevron);
  flex-shrink: 0;
  transition: transform .22s cubic-bezier(.4,0,.2,1);
}
.acc-item.open .acc-chevron { transform: rotate(180deg); }

/* ── Divider (header ↔ body) ────────────────────────── */
.acc-divider { height: 1px; background: var(--border-accordion-line); }

/* ── Body (smooth max-height animation) ─────────────── */
.acc-body {
  max-height: 0; overflow: hidden;
  transition: max-height .25s cubic-bezier(.4,0,.2,1);
}
.acc-item.open .acc-body { max-height: 800px; }

/* ── Body inner (padding on wrapper, not body) ──────── */
.acc-body-inner {
  padding: 12px 16px 16px;
  font-size: 14px; font-weight: 400; line-height: 1.5;
  color: var(--text-accordion-content);
}
```

> **Why `max-height` animation?** CSS cannot animate `height: auto`. Setting `max-height: 0 → 800px` with `overflow: hidden` gives a smooth collapse/expand. Padding lives on `.acc-body-inner` (not `.acc-body`) so the padding itself does not animate.

---

#### 6.9.5 HTML Structure

```html
<!-- ── Standalone item ──────────────────────────────── -->
<div class="acc-item">
  <div class="acc-header" onclick="toggleAcc(this)">
    <span class="acc-title">ชื่อหัวข้อ</span>
    <svg class="acc-chevron" viewBox="0 0 24 24" fill="none"
         stroke="currentColor" stroke-width="2"
         stroke-linecap="round" stroke-linejoin="round">
      <polyline points="6 9 12 15 18 9"/>
    </svg>
  </div>
  <div class="acc-body">
    <div class="acc-divider"></div>
    <div class="acc-body-inner">เนื้อหา</div>
  </div>
</div>

<!-- ── Stacked group (single-open) ─────────────────── -->
<div class="acc-group" id="my-group">
  <div class="acc-item">
    <div class="acc-header" onclick="toggleAcc(this,'my-group')">
      <span class="acc-title">รายการที่ 1</span>
      <svg class="acc-chevron" viewBox="0 0 24 24" fill="none"
           stroke="currentColor" stroke-width="2"
           stroke-linecap="round" stroke-linejoin="round">
        <polyline points="6 9 12 15 18 9"/>
      </svg>
    </div>
    <div class="acc-body">
      <div class="acc-divider"></div>
      <div class="acc-body-inner">เนื้อหา</div>
    </div>
  </div>
  <!-- ...more .acc-item... -->
</div>

<!-- ── Disabled item ────────────────────────────────── -->
<div class="acc-item disabled">
  <div class="acc-header" onclick="toggleAcc(this)">
    <span class="acc-title">ปิดใช้งาน</span>
    <svg class="acc-chevron" viewBox="0 0 24 24" fill="none"
         stroke="currentColor" stroke-width="2"
         stroke-linecap="round" stroke-linejoin="round">
      <polyline points="6 9 12 15 18 9"/>
    </svg>
  </div>
</div>
```

---

#### 6.9.6 JavaScript (toggleAcc)

```javascript
function toggleAcc(headerEl, groupId) {
  const item = headerEl.closest('.acc-item');
  if (!item || item.classList.contains('disabled')) return;
  const wasOpen = item.classList.contains('open');
  // Close all siblings in group
  const group = groupId
    ? document.getElementById(groupId)
    : (item.closest('.acc-group') || item.parentElement);
  if (group) group.querySelectorAll('.acc-item.open').forEach(i => i.classList.remove('open'));
  // Toggle this item
  if (!wasOpen) item.classList.add('open');
}
```

---

#### 6.9.7 Alias Chain

```
surface/accordion/collapsed  →  neutral/0   → #FFFFFF
surface/accordion/disabled   →  neutral/100 → #F4F5F5
border/accordion/default     →  neutral/300 → #C9CDD0
border/accordion/line        →  neutral/200 → #E2E4E6
text/accordion/collapsed     →  neutral/700 → #4D5358
text/accordion/disabled      →  neutral/400 → #ADB2B7
text/accordion/content       →  neutral/600 → #636B72
icon/accordion/collapsed     →  neutral/400 → #ADB2B7
```

---

### 6.10 Action List

**Figma nodes:** `2796:3166` (Action List container) · **`382:14360`** (List row · canonical property panel) · Library: **MIH Design System Foundation**  
**Component:** [`src/components/list/ActionList.jsx`](src/components/list/ActionList.jsx) · **CSS prefix:** `al-` · **Behavior reference:** Tailwind Headless UI Menu (click-to-select, single-active per group)

---

#### 6.10.1 Color Tokens

| CSS Variable | Figma Semantic Token | Hex | Primitive | Usage |
|---|---|---|---|---|
| `--surface-actionlist-default` | `surface/actionList/default` | `#FFFFFF` | white | Container background |
| `--surface-actionlist-item` | `surface/list/default` | `#FFFFFF` | white | Item row background — default |
| `--surface-actionlist-hover` | `surface/list/hover` | `#F4FBF8` | emerald-50 | Item row background — hover & active |
| `--surface-brandprimary-quaternary` | `surface/brandprimary/quaternary` | `#F4FBF8` | emerald-50 | Circle icon background |
| `--border-actionlist` | `border/actionList/default` | `#EBEBEB` | — | Container border |
| `--text-actionlist-primary` | `text/list/default` | `#4D5358` | neutral-700 | Label — default |
| `--text-actionlist-primary-hover` | `text/list/hover` | `#007549` | emerald-700 | Label — hover & active |
| `--text-list-selected` | `text/list/selected` | `#4D5358` | neutral-700 | Label — `state="selected"` |
| `--text-list-selected-hover` | `text/list/selected-hover` | `#007549` | emerald-700 | Label — `state="selected-hover"` |
| `--text-list-subtext` | `text/list/subtext` | `#858C92` | neutral-500 | Subtext row — Default / Selected |
| `--text-list-subtext-hover` | `text/list/subtext-hover` | `#54CF97` | emerald-500 | Subtext row — Hover / Selected-Hover (brand-aware) |
| `--text-actionlist-topic-title` | `text/topic/secondary-title` | `#007549` | emerald-700 | Topic title text |
| `--text-actionlist-topic-sub` | `text/topic/tertiary-subtext` | `#ADB2B7` | neutral-400 | Topic subtext |
| `--icon-actionlist-primary` | `icon/list/primary-default` | `#4D5358` | neutral-700 | Leading icon — default |
| `--icon-actionlist-primary-hover` | `icon/list/primary-hover` | `#007549` | emerald-700 | Leading icon — hover & active |
| `--icon-actionlist-secondary` | `icon/list/secondary-default` | `#ADB2B7` | neutral-400 | Trailing chevron — default |
| `--icon-actionlist-secondary-hover` | `icon/list/secondary-hover` | `#007549` | emerald-700 | Trailing chevron — hover & active |
| `--icon-positive-success` ★ | `icon/positive/success` | `#24a899` | teal-500 | SuccessIcon (TickOnCircle) — **fixed, non-brand** |

★ `icon/positive/success` is a **fixed** token — always teal-500 `#24a899`, not brand-responsive. Figma node `2798:5280`. Size via `dimension/size/600` = `--dim-size-600 = 24px`.

> **Correction from previous draft:** `surface/list/hover` = `#F4FBF8` (emerald-50), NOT `#E2F3EB` (emerald-100). Secondary icon/text = neutral-400 (`#ADB2B7`), not neutral-500. Item gap = 8px (`dimension/space/200`), not 12px.

---

#### 6.10.2 Dimension Tokens

> Figma node 2796-3166: container radius 12px · py 12px · px 4px · item-gap 4px · item min-h 48px · item px 12px · item py 8px · icon-label gap 8px · icon 16px · circle 36px

| CSS Variable | Dimension Token | Value | Role |
|---|---|---|---|
| `--al-container-radius` | `dimension/radius/400` | **12 px** | Container border-radius |
| `--al-container-py` | `dimension/space/300` | **12 px** | Container top/bottom padding |
| `--al-container-px` | `dimension/space/100` | **4 px** | Container left/right padding |
| `--al-item-gap` | `dimension/space/100` | **4 px** | Gap between item rows |
| `--al-item-px` | `dimension/space/300` | **12 px** | Item horizontal padding |
| `--al-item-py` | `dimension/space/200` | **8 px** | Item vertical padding |
| `--al-icon-gap` | `dimension/space/200` | **8 px** | Icon ↔ label gap inside item |
| `--al-item-min-h` | — | **48 px** | Minimum item height |
| `--al-item-radius` | — | **6 px** | Item border-radius |
| `--al-icon-sm` | `dimension/icon/sm` | **16 px** | Leading / trailing icon size |
| `--al-circle-size` | `dimension/size/700` | **36 px** | Circle icon container size |
| `--al-circle-radius` | — | **18 px** | Circle icon border-radius |
| `--al-topic-px` | `dimension/space/400` | **16 px** | Topic section horizontal padding |
| `--al-topic-py` | `dimension/space/200` | **8 px** | Topic section vertical padding |
| `--al-topic-gap` | `dimension/space/300` | **12 px** | Topic icon ↔ text gap |

---

#### 6.10.3 Typography

| Element | Font Size | Weight | Line-height | Token |
|---|---|---|---|---|
| Item label | 14 px | Regular (400) | 1.5 | `typography/body/sm` |
| Topic title | 16 px | Bold (700) | 1.5 | `typography/heading-content/base` |
| Topic subtext | 12 px | Medium (500) | 1.5 | `typography/body/xs` |
| Badge text | 12 px | Medium (500) | 1 | `typography/label/xs` |

---

#### 6.10.4 Figma Component Properties (node 382:14360 / 382:14361 — canonical "List" row)

| Property | Type | Values | Default |
|---|---|---|---|
| `state` | enum | `Default` · `Hover` · **`Selected`** · **`Selected-Hover`** | `Default` |
| `icon` (iconWeight) | enum | `Default` (font-weight 400) · `Darker` (font-weight 700) | `Default` |
| `showLeadingIcon` | bool | true / false | **`true`** — 16×16 leading icon |
| `showSubtext` | bool | true / false | **`false`** — secondary line under the label (`text/list/subtext`) |
| `showSwitch` | bool | true / false | **`true`** — 44×24 Switch (`surface/switch/*`) |
| `showBadge` | bool | true / false | **`true`** — Colorful badge pill (trailing position) |
| `showCheck` | bool | true / false | **`true`** — 16×16 check mark |
| `showChevronRight` | bool | true / false | **`true`** — 16×16 chevron right |
| `showLeadingBadge` | bool | true / false | **`false`** — first leading Colorful pill (e.g. queue number) |
| `showLeadingBadge2` | bool | true / false | **`false`** — second leading Colorful pill (e.g. HN) |
| `showLeadingBadge3` | bool | true / false | **`false`** — leading Colorful pill in a **fixed 64 px column** (Figma 382:14361 / 3401:4292). Primary-hierarchy bold-fill pill (e.g. ICD code `I10`) wrapped in a `width: 64px` flex container so codes of varying length (`I10`, `M54.9`, `G43.9`) keep label text aligned across rows. Used by the AISearch recent-history dropdown (Figma 3396:1627). |

> **Row anatomy (left-to-right):** `leadingIcon` (Material Symbol) → `leadingBadge` (Colorful pill) → `leadingBadge2` (Colorful pill) → `leadingBadge3` (Colorful pill, fixed 64 px column) → `circleIcon` (brand tile) → label + optional subtext → trailing `badge` (StatusBadge) → SuccessTick → Switch → check → chevron-right. Each slot is independently togglable; show* flags AND-gate with the corresponding content prop, so omitting `leadingBadge` keeps the slot empty even when `showLeadingBadge={true}`.

> **`leadingBadge3` spec (Figma 382:14361 / 3401:4292 / 3401:4293):** A primary-hierarchy ColorfulBadge (`hierarchy="primary"`, `fill="default"`) wrapped in a 64 px fixed-width column (`<div style={{ width: 64 }}>`). The badge inherits its own `min-width: 40 px` from `width="cap"`, so single-character codes don't collapse. The 64 px column is wider than the 40 px badge, leaving consistent breathing room before the label regardless of the code's character count. AISearch's `HistoryRow` (see `src/components/search/AISearch.jsx`) consumes the exact same slot pattern.

> **Leading badges spec:** Aliases `<ColorfulBadge style={tone} size="small" fill="default" hierarchy="secondary">` (Figma node 2076:8413). Resolves to a 24 px tall pill, radius 9999, `surface/colorfulBadge/{tone}` + `text/colorfulBadge/{tone}` (label2 / Sarabun Medium 12). The pair is brand-neutral (Colorful Badge palette is fixed semantic colors, not brand-aware) and matches the queue (teal) + HN (indigo) pills used in the PrimarySearch recent / typing dropdowns (Figma 3345:2981 + 3346:8231).

> **State surface + text matrix** (all brand-aware unless noted):
>
> | State | Surface | Label | Subtext |
> |---|---|---|---|
> | Default | `surface/list/default` (`#FFFFFF`) | `text/list/default` (`#4D5358`) | `text/list/subtext` (`#858C92`) |
> | Hover | `surface/list/hover` (`primary/50`) | `text/list/hover` (`primary/700`) | `text/list/subtext-hover` (`primary/500`) |
> | Selected | `surface/list/selected` (`#FFFFFF`) | `text/list/selected` (`#4D5358`) | `text/list/subtext` (`#858C92`) |
> | Selected-Hover | `surface/list/selected-hover` (`primary/50`) | `text/list/selected-hover` (`primary/700`) | `text/list/subtext-hover` (`primary/500`) |

> **Border-radius:** All four states use `dimension/radius/100` = **4 px** in the current Figma. (Earlier drafts ran Default at 6 px and Hover at 4 px; the canonical spec normalised them to 4 px for visual continuity across state transitions.)

> **SuccessIcon (TickOnCircle):** Filled circle `fill: var(--icon-positive-success, #24a899)` + white checkmark. Size 24×24 via `dimension/size/600` = `var(--dim-size-600, 24px)`. Token: `icon/positive/success` → teal-500 `#24a899` (**fixed**, non-brand-responsive). Figma node `2798:5280`.

> **SuccessIcon (TickOnCircle):** Filled circle `fill: var(--icon-positive-success, #24a899)` + white checkmark. Size 24×24 via `dimension/size/600` = `var(--dim-size-600, 24px)`. Token: `icon/positive/success` → teal-500 `#24a899` (**fixed**, non-brand-responsive). Figma node `2798:5280`.

> **Switch (Toggle):** `44×24px`. Aliases MIH DS Switch tokens — NOT brand-responsive.
> - Track OFF: `surface/switch/off` = `--surface-switch-off` → `#E2E4E6` (neutral-200)
> - Track ON: `surface/switch/on` = `--surface-switch-on` → `#1B867D` (teal/600, **fixed**)
> - Track OFF hover: `surface/switch/off-hover` = `--surface-switch-off-hover` → `#C9CDD0` (neutral-300)
> - Track ON hover: `surface/switch/on-hover` = `--surface-switch-on-hover` → `#196C65` (teal/700, **fixed**)
> - Thumb: `surface/switch/icon` = `--surface-switch-thumb` → `#FFFFFF`

---

#### 6.10.5 CSS Classes

```css
/* ── Container ─────────────────────────────────────── */
.al-wrap {
  background: var(--surface-actionlist-default);
  border: 1px solid var(--border-actionlist);
  border-radius: var(--al-container-radius, 12px);
  padding: var(--al-container-py, 12px) var(--al-container-px, 4px);
  display: flex; flex-direction: column;
  gap: var(--al-item-gap, 4px);
  min-width: 160px;
}

/* ── Topic header ───────────────────────────────────── */
.al-topic {
  display: flex; align-items: flex-start;
  gap: var(--al-topic-gap, 12px);
  padding: var(--al-topic-py, 8px) var(--al-topic-px, 16px);
}
.al-topic-content { flex: 1; display: flex; flex-direction: column; gap: 2px; }
.al-topic-title { font-size: 16px; font-weight: 700; line-height: 1.5; color: var(--text-actionlist-topic-title); }
.al-topic-sub   { font-size: 12px; font-weight: 500; line-height: 1.5; color: var(--text-actionlist-topic-sub); }

/* ── Item row ───────────────────────────────────────── */
.al-item {
  display: flex; align-items: center;
  gap: var(--al-icon-gap, 8px);
  min-height: var(--al-item-min-h, 48px);
  padding: var(--al-item-py, 8px) var(--al-item-px, 12px);
  border-radius: var(--al-item-radius, 6px);
  background: var(--surface-actionlist-item, #FFFFFF);
  cursor: pointer; transition: background .12s;
}
.al-item:hover, .al-item.hover, .al-item.active {
  background: var(--surface-actionlist-hover);
}

/* ── Leading icon (16×16) ───────────────────────────── */
.al-icon {
  width: var(--al-icon-sm, 16px); height: var(--al-icon-sm, 16px);
  color: var(--icon-actionlist-primary); flex-shrink: 0; transition: color .12s;
}
.al-item:hover .al-icon, .al-item.hover .al-icon,
.al-item.active .al-icon { color: var(--icon-actionlist-primary-hover); }

/* ── Circle icon (36×36) ────────────────────────────── */
.al-circle-icon {
  width: var(--al-circle-size, 36px); height: var(--al-circle-size, 36px);
  border-radius: var(--al-circle-radius, 18px);
  background: var(--surface-brandprimary-quaternary, #F4FBF8);
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0; color: var(--icon-actionlist-primary-hover);
}

/* ── Label ─────────────────────────────────────────── */
.al-label {
  flex: 1; min-width: 0;
  font-size: 14px; font-weight: 400; line-height: 1.5;
  color: var(--text-actionlist-primary); transition: color .12s;
}
.al-item:hover .al-label, .al-item.hover .al-label,
.al-item.active .al-label { color: var(--text-actionlist-primary-hover); }

/* ── Trailing icon (16×16) ─────────────────────────── */
.al-trailing {
  width: var(--al-icon-sm, 16px); height: var(--al-icon-sm, 16px);
  color: var(--icon-actionlist-secondary); flex-shrink: 0; transition: color .12s;
}
.al-item:hover .al-trailing, .al-item.hover .al-trailing,
.al-item.active .al-trailing { color: var(--icon-actionlist-secondary-hover); }

/* ── Badge pill ─────────────────────────────────────── */
.al-badge-pill {
  display: inline-flex; align-items: center; justify-content: center;
  height: 24px; padding: 4px 8px; border-radius: 999px;
  font-size: 12px; font-weight: 500; flex-shrink: 0;
}
.al-badge-pill.warning { background: #FFEDD5; color: #C2410C; }
.al-badge-pill.success { background: #D1FAE5; color: #007549; }
.al-badge-pill.info    { background: #DBEAFE; color: #1D4ED8; }
/* ── SuccessIcon — TickOnCircle (Figma node 2798:5280) ── */
/* icon/positive/success → teal-500 #24a899 (fixed, non-brand) */
/* dimension/size/600 = 24px */
.al-success-icon {
  width: var(--dim-size-600, 24px);
  height: var(--dim-size-600, 24px);
  flex-shrink: 0;
  color: var(--icon-positive-success, #24a899);
}
/* ── Switch (Figma node 368:3 · 44×24 · surface/switch/off) ── */
.al-switch {
  width: 44px; height: 24px; border-radius: 12px;
  background: var(--surface-switch-off, #E2E4E6);
  position: relative; cursor: pointer; flex-shrink: 0;
  transition: background .2s;
}
/* ON: surface/switch/on → teal/600 (fixed, not brand) */
.al-switch.on              { background: var(--surface-switch-on, #1B867D); }
/* OFF hover: surface/switch/off-hover → neutral-300 */
.al-switch:not(.on):hover  { background: var(--surface-switch-off-hover, #C9CDD0); }
/* ON hover: surface/switch/on-hover → teal/700 */
.al-switch.on:hover        { background: var(--surface-switch-on-hover, #196C65); }
.al-switch-thumb {
  position: absolute; width: 20px; height: 20px; border-radius: 50%;
  background: var(--surface-switch-thumb, #FFFFFF); top: 2px; left: 2px;
  box-shadow: 0 1px 3px rgba(0,0,0,.2);
  transition: transform .2s;
}
.al-switch.on .al-switch-thumb { transform: translateX(20px); }
/* ── Hover radius override (dimension/radius/100 = 4px) ── */
.al-item:hover, .al-item.hover, .al-item.active { border-radius: 4px; }
```

---

#### 6.10.6 HTML Structure

```html
<!-- ── Container ────────────────────────────────────── -->
<div class="al-wrap" id="my-list">

  <!-- Optional topic header -->
  <div class="al-topic">
    <div class="al-topic-content">
      <div class="al-topic-title">หัวข้อ</div>
      <div class="al-topic-sub">คำอธิบายเพิ่มเติม</div>
    </div>
  </div>

  <!-- Item: leading icon + label + chevron -->
  <div class="al-item" onclick="selectAlItem(this,'my-list')">
    <svg class="al-icon" viewBox="0 0 24 24" ...>...</svg>
    <span class="al-label">ชื่อรายการ</span>
    <svg class="al-trailing" viewBox="0 0 24 24" ...><polyline points="9 18 15 12 9 6"/></svg>
  </div>

  <!-- Item: circle icon + label + badge -->
  <div class="al-item" onclick="selectAlItem(this,'my-list')">
    <div class="al-circle-icon">
      <svg style="width:20px;height:20px;" viewBox="0 0 24 24" ...>...</svg>
    </div>
    <span class="al-label">ชื่อรายการ</span>
    <span class="al-badge-pill warning">ใหม่</span>
  </div>

  <!-- Item: active state -->
  <div class="al-item active" onclick="selectAlItem(this,'my-list')">
    <svg class="al-icon" viewBox="0 0 24 24" ...>...</svg>
    <span class="al-label">ถูกเลือก</span>
    <svg class="al-trailing" viewBox="0 0 24 24" ...><polyline points="9 18 15 12 9 6"/></svg>
  </div>

</div>
```

---

#### 6.10.7 JavaScript (selectAlItem)

```javascript
/* Tailwind Headless UI Menu — single-active per group */
function selectAlItem(el, groupId) {
  var wrap = groupId ? document.getElementById(groupId) : el.closest('.al-wrap');
  if (wrap) wrap.querySelectorAll('.al-item.active').forEach(function(i) {
    i.classList.remove('active');
  });
  el.classList.add('active');
}
```

---

#### 6.10.8 Alias Chain

```
surface/actionList/default       →  neutral/0    → #FFFFFF           (container bg — fixed neutral)
surface/list/default             →  neutral/0    → #FFFFFF           (item bg — fixed neutral)
surface/list/hover               →  primary/50   → var(--em50)       (item hover/active bg — BRAND)
surface/brandprimary/quaternary  →  primary/50   → var(--em50)       (circle icon bg — BRAND)
border/actionList/default        →  —            → #EBEBEB           (container border — fixed neutral)
text/list/default                →  neutral/700  → #4D5358           (label default — fixed neutral)
text/list/hover                  →  primary/700  → var(--em700)      (label hover/active — BRAND)
text/topic/secondary-title       →  primary/700  → var(--em700)      (topic title — BRAND)
text/topic/tertiary-subtext      →  neutral/400  → #ADB2B7           (topic sub — fixed neutral)
icon/list/primary-default        →  neutral/700  → #4D5358           (leading icon — fixed neutral)
icon/list/primary-hover          →  primary/700  → var(--em700)      (leading icon hover — BRAND)
icon/list/secondary-default      →  neutral/400  → #ADB2B7           (trailing icon — fixed neutral)
icon/list/secondary-hover        →  primary/700  → var(--em700)      (trailing icon hover — BRAND)

/* Badge pill (in item row) */
text/status/success (badge text) →  primary/700  → var(--em700)      (success badge text — BRAND)
surface/status/success (badge)   →  fixed green  → #D1FAE5           (success badge bg — fixed neutral)
```

---

### 6.11 Avatar

**Node:** `2303-11443` · Figma library: **MIH Design System Foundation / Semantic**

Two display variants: **Initial** (letter monogram on color background) and **Photo** (circular image, `object-fit: cover`). Both share the same size scale, status indicator, and group overlap patterns.

---

#### 6.11.1 Surface & Border Tokens

| Figma Semantic Token | Hex | Primitive | Usage |
|----------------------|-----|-----------|-------|
| `surface/avatar/default` | `#E2E4E6` | neutral-200 | Default background (no color class) |
| `border/avatar` | `#FFFFFF` | white | 2px ring around every avatar |
| `border/neutral/quaternary` | `#F4F5F5` | neutral-100 | Separator ring between stacked avatars |

---

#### 6.11.2 Initials Color Variants (Primitive references)

| Variant | CSS class | Background | Text |
|---------|-----------|-----------|------|
| Doctor / Physician | `.color-em` | `emerald-100` `#E2F3EB` | `emerald-700` `#007549` |
| Nurse | `.color-bl` | `blue-100` `#DBEAFE` | `blue-700` `#1D4ED8` |
| Patient | `.color-rd` | `red-100` `#FEE2E2` | `red-700` `#B91C1C` |
| Drug / Rx | `.color-or` | `orange-100` `#FFEDD5` | `orange-700` `#C2410C` |
| Generic User | `.color-tl` | `teal-100` `#CCEFED` | `teal-600` `#24A899` |
| Default / Fallback | _(none)_ | `neutral-200` `#E2E4E6` | `neutral-600` `#636B72` |

---

#### 6.11.3 Photo Variant

Add `.photo` to `.av` — renders a circular `<img>` (object-fit: cover) from Figma assets. Background is the per-person frame fill from Figma set via inline `style`. The `img` is absolute-positioned to fill the full circle.

```html
<div class="av photo md" style="background:#E7F1F8;">
  <img src="https://www.figma.com/api/mcp/asset/{assetId}" alt="Doctor A">
</div>
```

**Figma photo assets — node 2303-11443**

| Person | Figma Node | Asset UUID | Bg |
|--------|-----------|-----------|-----|
| Doctor A | `376:11301` | `1c68d7a3-929d-4f0c-8478-61cc7e6df9c6` | `#E7F1F8` |
| Doctor B | `3070:4443` | `f55e31bb-fc36-4a1f-9e08-9666a27b9364` | `#E7F1F8` |
| Doctor C | `2887:4821` | `eb37c3d9-19f4-4395-8013-7d2ac316457d` | `#CFE6F7` |
| Nurse A | `2989:4325` | `8c5c52fb-cbf1-477e-b4fc-47432e479335` | `#CFE6F7` |
| Doctor D | `3070:4446` | `885864f2-52af-4bae-b3d1-8d93ed64c52a` | `#ECEAEA` |
| User A | `2303:11442` | `3ebb4e7d-829a-47ac-ba41-c6346bcba556` | `#6EBBD0` |

> **Note:** Figma MCP asset URLs expire after ~7 days. Re-fetch via `get-assets` on node `2303-11443` when URLs rotate.

**CSS:**
```css
.av.photo { padding: 0; }
.av.photo img {
  position: absolute; inset: 0;
  width: 100%; height: 100%;
  object-fit: cover; border-radius: 50%;
  pointer-events: none;
}
```

---

#### 6.11.4 Status Indicator

| Status | Token | Hex | CSS class |
|--------|-------|-----|-----------|
| Online | `surface/avatar/online` → `emerald-600` | `#08A768` | `.av-status.online` |
| Offline | `surface/avatar/offline` → `neutral-400` | `#ADB2B7` | `.av-status.offline` |
| Busy | `surface/avatar/busy` → `red-600` | `#DC2626` | `.av-status.busy` |

Status dot: `10×10px`, `border-radius: 50%`, `border: 2px solid #FFFFFF`, positioned `bottom:0; right:0` absolute inside `.av` (`position: relative`).

---

#### 6.11.5 Size Scale

| Size | Modifier | W × H | Font | Border |
|------|----------|-------|------|--------|
| xs | `.av.xs` | 24 × 24 px | 9px | 1.5px |
| sm | `.av.sm` | 32 × 32 px | 11px | 2px |
| md _(default)_ | `.av.md` | 40 × 40 px | 14px | 2px |
| lg | `.av.lg` | 48 × 48 px | 16px | 2px |
| xl | `.av.xl` | 56 × 56 px | 18px | 2px |

---

#### 6.11.6 Avatar Group

```css
.av-group { display: flex; }
.av-group .av { margin-right: -10px; box-shadow: 0 0 0 2px #FFFFFF; }
```

Add a `+N` overflow pill as the last `.av` with `neutral-200` bg and `neutral-600` text.

---

#### 6.11.7 Dimension Tokens

| CSS Variable | Dimension Token | Value | Role |
|---|---|---|---|
| `--av-size` | `dimension/size/1000` | **40 px** | Avatar base size (md) |
| `--av-stroke` | `dimension/stroke/200` | **2 px** | White border ring |

---

### 6.12 Form Input

**Figma library:** MIH Design System Foundation / Semantic  
**Variants:** Large (40px) · Medium (36px) · Textarea · Input Slash · Voice Input  
**Token namespace:** `surface/input/*` · `border/input/*` · `text/input/*` · `icon/input/*`

> `border/input/hover` aliases `Brand/primary/600` (#08a768 light / #6ad6a5 dark), same level as `border/input/typing`.

#### Slots (optional parts — togglable in preview)

| Slot | Figma property | Token | Always shown? |
|------|---------------|-------|---------------|
| **Leading icon** | `Icon=True/False` | `icon/input/default` → changes per state | Optional |
| **Trailing icon** | `Trailing=True/False` | — clear ×, eye 👁, status ✓/✗ | Optional |
| **Optional text** | `Optional=True/False` | `text/content/tertiary` (`#858c92`) | Only for non-required fields |

- **Leading icon** is the SVG prefix inside the input box. Uses `icon/input/*` tokens — color shifts with state (default → neutral-400, hover/typing → primary/600, error → red-600).
- **Trailing icon** covers three sub-types: **clear ×** button (disappears when field is empty), **eye toggle** (password), and **status icon** ✓/✗ (shows after blur validation).
- **Optional text** appears right-aligned in the label row for non-required fields. Uses `text/content/tertiary` (#858c92, neutral-500).

---

#### 6.12.1 Surface Tokens

| Figma Semantic Token | Light Hex | Dark Hex | Primitive Alias | Usage |
|----------------------|-----------|----------|-----------------|-------|
| `surface/input/default` | `#ffffff` | `#111314` | white | Input background — all interactive states |
| `surface/input/disabled` | `#f4f5f5` | `#1a1d1f` | neutral-100 | Disabled input background |

---

#### 6.12.2 Border Tokens

| Figma Semantic Token | Light Hex | Dark Hex | Primitive Alias | State | Note |
|----------------------|-----------|----------|-----------------|-------|------|
| `border/input/default` | `#e2e4e6` | `#4d5358` | neutral-200 | Default | 1px stroke |
| `border/input/hover` | `#08a768` | `#6ad6a5` | Brand/primary/600 | Hover (cursor over field, not focused) | 2px stroke + glow ring |
| `border/input/typing` | `#08a768` | `#6ad6a5` | Brand/primary/600 | Typing / Active focus | 2px stroke + glow ring |
| `border/input/filled` | `#e2e4e6` | `#363b3f` | neutral-200 | Filled (has value, unfocused) | 1px stroke |
| `border/input/error` | `#dc2626` | `#f87171` | Primitive/red-600 | Error | 2px stroke + red glow ring |
| `border/input/disabled` | `#e2e4e6` | `#363b3f` | neutral-200 | Disabled | 1px stroke |

**Focus glow rings (box-shadow):**

| State | Shadow Value | Primitive |
|-------|-------------|-----------|
| Hover | `0 0 0 4px rgba(8,167,104,.14)` | emerald-600 / primary/600 at 14% |
| Typing | `0 0 0 4px rgba(8,167,104,.14)` | emerald-600 / primary/600 at 14% |
| Error | `0 0 0 3px rgba(220,38,38,.12)` | red-600 at 12% |

---

#### 6.12.3 Text Tokens

| Figma Semantic Token | Light Hex | Dark Hex | Primitive Alias | Usage |
|----------------------|-----------|----------|-----------------|-------|
| `text/input/default` | `#adb2b7` | `#636b72` | neutral-400 | Placeholder text |
| `text/input/typing` | `#4d5358` | `#c9cdd0` | neutral-700 | Active typing value |
| `text/input/filled` | `#4d5358` | `#c9cdd0` | neutral-700 | Filled value (unfocused) |
| `text/input/disabled` | `#adb2b7` | `#636b72` | neutral-400 | Disabled placeholder / value |
| `text/content/default` | `#363b3f` | `#f4f5f5` | neutral-900 | Label text |
| `text/content/disabled` | `#636b72` | `#858c92` | neutral-600 | Disabled label |
| `text/content/tertiary` | `#858c92` | `#636b72` | neutral-500 | Helper / hint text |
| `text/input/required` | `#dc2626` | `#f87171` | red-600 | Required asterisk (*) |
| `text/input/helper-error` | `#dc2626` | `#f87171` | red-600 | Error helper text |
| `text/input/helper-disabled` | `#adb2b7` | `#636b72` | neutral-400 | Disabled helper text |

---

#### 6.12.4 Icon Tokens

| Figma Semantic Token | Light Hex | Dark Hex | Primitive Alias | State |
|----------------------|-----------|----------|-----------------|-------|
| `icon/input/default` | `#adb2b7` | `#636b72` | neutral-400 | Leading icon — default / hover |
| `icon/input/hover` | `#08a768` | `#6ad6a5` | Brand/primary/600 | Leading icon — hover / focus active |
| `icon/input/typing` | `#4d5358` | `#c9cdd0` | neutral-700 | Leading icon — typing |
| `icon/input/filled` | `#adb2b7` | `#636b72` | neutral-400 | Leading icon — filled (unfocused) |
| `icon/input/error` | `#dc2626` | `#f87171` | red-600 | Leading icon — error |
| `icon/input/disabled` | `#adb2b7` | `#636b72` | neutral-400 | Leading icon — disabled |

---

#### 6.12.5 Dimension Tokens

##### Size & Shape

| Property | Large (default) | Medium | CSS Variable | Source |
|----------|----------------|--------|--------------|--------|
| **Height** | `40px` | `36px` | `--fi-height-lg` / `--fi-height-md` | Figma frame height |
| **Textarea** min-height | `96px` | — | `--fi-min-height-ta` | Figma frame |
| **Border radius** | `8px` | `8px` | `--fi-radius` | Figma corner radius |

##### Spacing

| Property | Large (default) | Medium | CSS Variable | Source |
|----------|----------------|--------|--------------|--------|
| **Padding** horizontal | `16px` | `12px` | `--fi-padding-h-lg` / `--fi-padding-h-md` | Figma auto-layout |
| **Padding** vertical | `8px` | `6px` | `--fi-padding-v-lg` / `--fi-padding-v-md` | Figma auto-layout |
| **Textarea** padding | `8px 12px` | — | `--fi-padding-ta-v` / `--fi-padding-ta-h` | Figma auto-layout |
| **Gap** (icon ↔ text) | `8px` | `6px` | `--fi-gap-lg` / `--fi-gap-md` | Figma auto-layout |

##### Stroke

| State | Width | CSS Variable | Source |
|-------|-------|--------------|--------|
| Default / Filled / Disabled | `1px` | `--fi-stroke-default` | Figma stroke |
| Hover / Typing / Error | `2px` | `--fi-stroke-active` | Figma stroke |

##### Icon

| Size | Value | CSS Variable | Token | Figma node |
|------|-------|--------------|-------|------------|
| Leading icon — Large | `20 × 20 px` | `--fi-icon-size-lg` | `icon-size/input/large` | 2496-5341 |
| Leading icon — Medium | `16 × 16 px` | `--fi-icon-size-md` | `icon-size/input/medium` | — |

> **Slash variant** uses the Large icon size (`20 × 20 px`) for both the `calendar_today` (`#ic-cal`) and clock (`#ic-clock`) leading icons. All `.fi-lead-ico` elements are sized via `width/height: var(--fi-icon-size-lg)`. The icon toggle (`toggleSlashIcon`) in the preview toggles `.fi-no-lead` on `#slash-section` to show/hide leading icons across both the interactive demo and the States Reference.

---

#### 6.12.6 Typography Tokens

| Element | Font Size | Weight | Line Height | CSS Variable | Token / Primitive |
|---------|-----------|--------|-------------|--------------|-------------------|
| Label (Large) | `14px` | `500` | `1.5` | `--fi-font-size-label-lg` / `--fi-font-weight-label` / `--fi-line-height` | `text/content/default` |
| Label (Medium) | `12px` | `500` | `1.5` | `--fi-font-size-label-md` | `text/content/default` |
| Placeholder (Large) | `16px` | `400` | `1.5` | `--fi-font-size-value-lg` | `text/input/default` |
| Placeholder (Medium) | `14px` | `400` | `1.5` | `--fi-font-size-value-md` | `text/input/default` |
| Value / Typing | same as placeholder | `400` | `1.5` | `--fi-font-size-value-lg` | `text/input/filled` |
| Helper text | `14px` | `400` | `1.5` | `--fi-font-size-helper` | `text/content/tertiary` |
| Required asterisk | `14px` | `400` | `1.5` | `--fi-font-size-label-lg` | `text/input/required` |

Font family: `Sarabun` (all elements)

---

#### 6.12.7 State Summary

| State | Border token | Border Hex | Glow ring | Background | Text (value) |
|-------|-------------|------------|-----------|------------|--------------|
| **Default** | `border/input/default` | `#e2e4e6` | none | `#ffffff` | `#adb2b7` (placeholder) |
| **Hover** | `border/input/hover` | `#08a768` | `rgba(8,167,104,.14)` | `#ffffff` | `#adb2b7` (placeholder) |
| **Typing / Focus** | `border/input/typing` | `#08a768` | `rgba(8,167,104,.14)` | `#ffffff` | `#4d5358` (value) |
| **Filled** | `border/input/filled` | `#e2e4e6` | none | `#ffffff` | `#4d5358` (value) |
| **Error** | `border/input/error` | `#dc2626` | `rgba(220,38,38,.12)` | `#ffffff` | `#4d5358` (value) |
| **Disabled** | `border/input/disabled` | `#e2e4e6` | none | `#f4f5f5` | `#adb2b7` (placeholder) |

---

#### 6.12.8 Variant Notes

**Input Field Slash** — two `<input>` fields side-by-side, separated by a `/` divider character (`color: neutral-400 #adb2b7`). Each field shares the same border/input/* tokens. Only the focused half shows the typing border. See §6.12.11 for the full interactive demo spec (date range DD/MM/YYYY + optional time range HH:MM, auto-format, Tab auto-jump, day-count calculation).

**Voice Input** — identical token set; the leading icon is a microphone (`icon/input/default` neutral-400 → `icon/input/hover` primary/600 on focus).

**Textarea** — same border/input/* and text/input/* tokens; height minimum 96px, `resize: vertical`, padding `8px 12px`. See §6.12.10 for the full interactive demo spec (char counter with `.near`/`.full` states, min-length validation, submit chip).

**Password field** — uses `border/input/typing` on focus, with:
- Eye-toggle trailing icon (`icon/input/default` → neutral-400)
- Strength bar: weak `#dc2626` (red-600) → fair `#ca8a04` (yellow-600) → good `#08a768` (primary/600) → strong `#007549` (primary/700)

---

#### 6.12.9 CSS Variable Quick Reference

```css
/* Semantic Input Tokens — MIH Design System */

/* Surface */
--surface-input-default:        #ffffff;   /* surface/input/default   → white */
--surface-input-disabled:       #f4f5f5;   /* surface/input/disabled  → neutral-100 */

/* Border (1px default/filled/disabled · 2px hover/typing/error) */
--border-input-default:         #e2e4e6;   /* border/input/default    → neutral-200 */
--border-input-hover:           #08a768;   /* border/input/hover      → Brand/primary/600 */
--border-input-typing:          #08a768;   /* border/input/typing     → Brand/primary/600 */
--border-input-filled:          #e2e4e6;   /* border/input/filled     → neutral-200 */
--border-input-error:           #dc2626;   /* border/input/error      → red-600 */
--border-input-disabled:        #e2e4e6;   /* border/input/disabled   → neutral-200 */

/* Focus glow rings (box-shadow) */
--focus-ring-input-hover:       rgba(8,167,104,.14);   /* hover  glow — primary/600 14% · 4px spread */
--focus-ring-input-typing:      rgba(8,167,104,.14);   /* typing glow — primary/600 14% · 4px spread */
--focus-ring-input-error:       rgba(220,38,38,.12);   /* error  glow — red-600 12%     · 3px spread */

/* Input text */
--text-input-default:           #adb2b7;   /* text/input/default      → neutral-400 (placeholder) */
--text-input-filled:            #4d5358;   /* text/input/filled       → neutral-700 (typed value) */
--text-input-disabled:          #adb2b7;   /* text/input/disabled     → neutral-400 */
--text-input-required:          #dc2626;   /* text/input/required     → red-600 (asterisk *) */
--text-input-helper-error:      #dc2626;   /* text/input/helper-error → red-600 */
--text-input-helper-dis:        #adb2b7;   /* text/input/helper-disabled → neutral-400 */

/* Content / Label / Helper */
--text-content-default:         #363b3f;   /* text/content/default    → neutral-900 (labels) */
--text-content-tertiary:        #858c92;   /* text/content/tertiary   → neutral-500 (helper / optional / counter) */
--text-content-disabled:        #636b72;   /* text/content/disabled   → neutral-600 */

/* Icon */
--icon-input-default:           #adb2b7;   /* icon/input/default      → neutral-400 */
--icon-input-hover:             #08a768;   /* icon/input/hover        → Brand/primary/600 */
--icon-input-typing:            #08a768;   /* icon/input/typing       → Brand/primary/600 */
--icon-input-filled:            #adb2b7;   /* icon/input/filled       → neutral-400 */
--icon-input-error:             #dc2626;   /* icon/input/error        → red-600 */
--icon-input-disabled:          #adb2b7;   /* icon/input/disabled     → neutral-400 */

/* Input Field Slash separator "/" */
--text-slash-sep:               #adb2b7;   /* neutral-400 — subdued divider between paired fields */

/* ── Dimension Tokens ── */

/* Size */
--fi-height-lg:          40px;    /* height/input/large  */
--fi-height-md:          36px;    /* height/input/medium */
--fi-min-height-ta:      96px;    /* min-height/input/textarea */

/* Padding */
--fi-padding-h-lg:       16px;    /* padding-horizontal/input/large  */
--fi-padding-h-md:       12px;    /* padding-horizontal/input/medium */
--fi-padding-v-lg:       8px;     /* padding-vertical/input/large    */
--fi-padding-v-md:       6px;     /* padding-vertical/input/medium   */
--fi-padding-ta-v:       8px;     /* padding-vertical/input/textarea */
--fi-padding-ta-h:       12px;    /* padding-horizontal/input/textarea */

/* Gap (icon ↔ text, managed via flex gap) */
--fi-gap-lg:             8px;     /* gap/input/large  */
--fi-gap-md:             6px;     /* gap/input/medium */

/* Shape */
--fi-radius:             8px;     /* radius/input */

/* Stroke width */
--fi-stroke-default:     1px;     /* stroke/input/default — default, filled, disabled */
--fi-stroke-active:      2px;     /* stroke/input/active  — hover, typing, error */

/* Icon size */
--fi-icon-size-lg:       20px;    /* icon-size/input/large  */
--fi-icon-size-md:       16px;    /* icon-size/input/medium */

/* Typography */
--fi-font-size-value-lg: 16px;    /* font-size/input/value/large  */
--fi-font-size-value-md: 14px;    /* font-size/input/value/medium */
--fi-font-size-label-lg: 14px;    /* font-size/input/label/large  */
--fi-font-size-label-md: 12px;    /* font-size/input/label/medium */
--fi-font-size-helper:   14px;    /* font-size/input/helper */
--fi-font-weight-label:  500;     /* font-weight/input/label */
--fi-line-height:        1.5;     /* line-height/input */
```

---

#### 6.12.10 Interactive Demo — §2 Textarea

The preview includes a live Textarea demo card (`#fiw-ta-notes`, `#fiw-ta-remark`) demonstrating the full state lifecycle with real validation and character counting.

**Fields**

| Field | ID | Required | Min | Max chars | Rows |
|-------|----|----------|-----|-----------|------|
| Clinical Notes | `fi-ta-notes` | ✓ (asterisk) | 20 chars | 500 | 4 |
| Additional Remarks | `fi-ta-remark` | — (Optional) | — | 200 | 2 |

**Character counter** (`.fi-counter`)

The counter `N / MAX` sits below-right of each textarea and updates on every `input` event.

| Counter state | Class added | Trigger |
|--------------|-------------|---------|
| Default | _(none)_ | `len < 80 % of max` |
| Near-limit | `.near` | `len ≥ 80 % of max` — amber colour |
| At limit | `.full` | `len === max` — red colour |

Token used: `text/input/helper-error` (`#dc2626`) for `.full`; `text/content/tertiary` (`#858c92`) for default counter.

**Submit validation (`taSubmit()`)**

1. Trims whitespace from Clinical Notes value.
2. If `len < 20`: applies `.is-error` to `#fib-ta-notes`, triggers `.do-shake` CSS animation (400 ms), sets helper message to `"กรุณากรอกอย่างน้อย 20 ตัวอักษร (ขณะนี้ N ตัว)"` with `text/input/helper-error` colour, and returns focus to the field.
3. On success: removes `.is-error`, sets helper to `"✓ บันทึกสำเร็จ"` (green, `border/input/typing` `#08a768`), and appends a success chip to `#ta-chip-row` with a `fadeUp` entrance animation.

**Reset (`taReset()`)**

Clears both textarea values, resets both counters to `0 / MAX`, removes `.is-error` from both boxes, resets helper messages to their default hint text, removes all chips from `#ta-chip-row`, and focuses `fi-ta-notes`.

**CSS classes involved**

| Class | Applied to | Effect |
|-------|-----------|--------|
| `.fi-box-ta` | `.fi-box` | Sets `height: auto`, enables `resize: vertical`, padding `8px 12px` |
| `.is-error` | `.fi-box` | `border-color: border/input/error` + red glow ring |
| `.do-shake` | `.fi-box` | 3-cycle horizontal shake keyframe (400 ms, auto-removed) |
| `.fi-counter` | `<p>` below textarea | Counter base style (right-aligned, small) |
| `.fi-counter.near` | same | Amber colour when ≥ 80 % full |
| `.fi-counter.full` | same | Red colour at max capacity |

---

#### 6.12.10b SuggestibleTextarea (autocomplete popover)

> **Component:** [`SuggestibleTextarea`](src/components/forms/SuggestibleTextarea.jsx) — wraps `Textarea` (or `VoiceTextarea` when `voice` is set) with the shared `useTextareaSuggestions` hook ([`textareaSuggestions.jsx`](src/components/forms/textareaSuggestions.jsx)).
> **Figma:** PHCIS · OPD `162:24221` (popover row spec) · PHCIS-ALL `3185:144422` (popover position).
> **Library demo:** [/components/suggestible-textarea](src/pages/components/ComponentsLibraryPage.jsx).

**When it opens.** As soon as the textarea value's trimmed length reaches `minChars` (default `3`; `DURATION_SUGGESTIONS` uses `1` because durations often start with a single digit). The hook keeps the popover open while the value stays at-or-above threshold and closes it on `Escape`, outside click, or when a suggestion is committed.

**Position rule (Figma PHCIS-ALL 3185:144422).** The popover anchors **4 px below the baseline of the last typed glyph row** — not below the textarea's bottom inner padding. For an anchor whose `tagName === 'TEXTAREA'` the hook computes:

```
contentBottom = rect.top + min(scrollHeight, clientHeight) − paddingBottom
top           = max(8, contentBottom + 4)
```

`paddingBottom` is read from `getComputedStyle(anchor).paddingBottom` (the canonical Textarea uses `var(--dim-space-200)` = 8 px). Subtracting it eliminates the dead space below the last line so the popover hugs the text. Inputs / non-textarea anchors fall back to `rect.bottom`.

**Z-index.** Sits at `var(--sm-z-modal-popover, 1450)` so it overlays modals — required when the textarea lives inside `BodyRecordPopup` / `DrawingCanvasPopup` / `MarkNoteEditor`.

**Row layout.** Each suggestion row is `min-h-48` · `px 12` · `py 8` · `gap 8`, with the matched substring rendered via [`highlightMatch`](src/components/forms/textareaSuggestions.jsx) in `var(--text-brandPrimary-secondary, #08A768)` bold. A trailing 16 px `add_circle` icon swaps from `--icon-brandPrimary-quaternary` to `--icon-brandPrimary-secondary` on hover. Clicking (or pressing the row's icon) commits the entire phrase via `onChange`.

**Props**

| Prop | Type | Default | Notes |
|---|---|---|---|
| `value` / `onChange` | `string` / `(e) => void` | required | `onChange` receives a DOM-event-shaped object (`{ target: { value } }`) — same as a native `<textarea onChange>`. |
| `suggestions` | `string[]` | required | Plain strings. Filtered by case-insensitive `includes(query)`; top 10 are rendered. |
| `minChars` | `number` | `3` | Use `1` for digit-led phrase pools (`DURATION_SUGGESTIONS`). |
| `rows` | `number` | `3` | Forwarded to the underlying `Textarea`. |
| `voice` | `boolean` | `false` | Swap inner field for `VoiceTextarea`; `height` / `minHeight` become forwarded. |
| `popoverWidth` | `number` | `591` | Clamped to viewport; widens with the screen. |
| `popoverAriaLabel` | `string` | `'คำแนะนำ'` | Used as the listbox label for screen readers. |

**Where it's used**

- **OPD Screening** (`src/pages/opd/ScreeningPage.jsx`) — Chief Complaint · ระยะเวลาที่มีอาการ · ประวัติปัจจุบัน.
- **TreatmentSteps Step 3 — บันทึกของแพทย์** (`src/components/examination/ExamNotesPanel.jsx`) — `cc` · `pi` · `pe` · `duration`.
- **TreatmentSteps Step 4 — บันทึกการรักษา** (`src/components/forms/TreatmentSteps.jsx`) — Chief complaint · Present illness.
- **Drawing canvas mark notes** — `MarkNoteEditor` textarea (via [`useTextareaSuggestions`](src/components/forms/textareaSuggestions.jsx) with region-aware `markerSuggestions.js`).

**Suggestion pools** (exported from `src/pages/opd/ScreeningPage.jsx`)

| Export | minChars | Use |
|---|---|---|
| `CHIEF_COMPLAINT_SUGGESTIONS` | 3 | One-line Thai presentations across body systems. |
| `DURATION_SUGGESTIONS` | 1 | Time phrases (`30 นาที` … `เรื้อรังมาหลายปี`). |
| `PRESENT_ILLNESS_SUGGESTIONS` | 3 | Full HPI narrative sentences. |
| `PHYSICAL_EXAM_SUGGESTIONS` | 3 | Head-to-toe exam findings (HEENT · chest · abdomen · neuro · skin · MS). |

---

#### 6.12.11 Interactive Demo — §3 Input Field Slash

The preview includes a live date-range + time-range demo card demonstrating the Slash variant — two `<input>` elements joined by a `/` separator, each styled with `.fi-box` tokens.

**Fields**

| Field | ID | Box ID | Format | Max length | Required |
|-------|----|--------|--------|------------|---------|
| Start date | `fi-date-s` | `fib-date-s` | `DD/MM/YYYY` | 10 | ✓ |
| End date | `fi-date-e` | `fib-date-e` | `DD/MM/YYYY` | 10 | ✓ |
| Start time | `fi-time-s` | `fib-time-s` | `HH:MM` | 5 | — (Optional) |
| End time | `fi-time-e` | `fib-time-e` | `HH:MM` | 5 | — (Optional) |

> **Note:** Each `.fi-wrap` row contains **two** `.fi-box` elements. The standard `_fiSetError` helper (which finds the first `.fi-box` in the wrap) cannot target individual boxes. The Slash demo uses dedicated helpers `_slashSetErr(boxId, msgId, text)` and `_slashClrBox(boxId)` that address boxes by id directly.

**Auto-format on input**

`_fmtDate(inp)` — strips all non-digits, then inserts `/` after position 2 (day) and position 5 (month), and caps at 10 characters. Runs on every `input` event.

`_fmtTime(inp)` — strips all non-digits, inserts `:` after position 2 (hour), caps at 5 characters.

Both fields use `inputmode="numeric"` so mobile devices show the numeric keypad.

**Tab auto-jump**

When the start-date field (`fi-date-s`) has exactly 10 characters and the user presses <kbd>Tab</kbd> (without Shift), the default tab behaviour is suppressed and focus jumps directly to `fi-date-e`. This mirrors the Figma Slash component UX where filling the first slot automatically moves the cursor to the second.

**Blur validation**

On `blur`, each date/time field runs a format check:
- Date: must match `/^(\d{2})\/(\d{2})\/(\d{4})$/`, month 1–12, day 1–31, and pass `Date` object back-check (catches invalid days like 31/02).
- Time: must match `/^(\d{2}):(\d{2})$/`, hour ≤ 23, minute ≤ 59.

If the field has content but fails validation, the corresponding `.fi-box` receives `.is-error` and the shared helper message shows the format hint (`"รูปแบบ DD/MM/YYYY"` / `"รูปแบบ HH:MM"`).

**Confirm validation (`slashConfirm()`)**

Order of checks:

1. **Date required** — both `fi-date-s` and `fi-date-e` must be non-empty and valid dates; shake + error on first failure.
2. **Date range** — `startDate ≤ endDate`; if reversed, shakes `fib-date-e` and shows `"วันสิ้นสุดต้องไม่ก่อนวันเริ่มต้น"`.
3. **Time (if either field filled)** — both time fields must be valid; if either is present but one is missing, flags the empty one.
4. **Time range** — `startTime ≤ endTime`; if reversed, shakes `fib-time-e` and shows `"เวลาสิ้นสุดต้องไม่ก่อนเวลาเริ่มต้น"`.
5. **Success** — calculates day count (`Math.round((endDate − startDate) / 86_400_000) + 1`), sets `fim-slash-date` helper to `"✓ N วัน"` (green), appends a chip to `#slash-chip-row` showing the date range and optional time range.

**Reset (`slashReset()`)**

Clears all four input values, removes `.is-error` from all four `.fi-box` elements, resets both helper messages to placeholder text, removes all chips from `#slash-chip-row`, and focuses `fi-date-s`.

**Token behaviour specific to Slash variant**

| Detail | Value |
|--------|-------|
| `/` separator colour | `neutral-400` `#adb2b7` — CSS var `--text-slash-sep` |
| Each half border | Shared `border/input/*` — only the *focused* half shows typing state |
| Leading icon — size | **20 × 20 px** — `--fi-icon-size-lg` (`icon-size/input/large`) · Figma node 2496-5341 |
| Leading icon — colour | `icon/input/default` neutral-400 (idle); hover → `icon/input/hover` primary/600; error → `icon/input/error` |
| Date icon element | `<svg class="fi-lead-ico"><use href="#ic-cal"/></svg>` — `calendar_today` symbol, `viewBox="0 0 20 20"` |
| Time icon element | `<svg class="fi-lead-ico"><use href="#ic-clock"/></svg>` — clock symbol, `viewBox="0 0 20 20"` |
| Both halves are in the same `.fi-wrap` | Shared helper text (`fim-slash-date`, `fim-slash-time`) below the pair |

**Icon visibility toggle**

The preview includes a **Leading icon** toggle button (`id="tog-slash-lead"`) above the §3 section. Clicking it calls `toggleSlashIcon(btn)` which:

1. Checks whether `btn` has the `active` class (icon currently visible).
2. Adds / removes the `.fi-no-lead` class on `#slash-section` — the wrapper that encloses both the interactive demo cards and the States Reference card.
3. Toggles the `active` class on the button to reflect the current state.

CSS rules triggered by `.fi-no-lead` on the wrapper:

| CSS rule | Effect |
|----------|--------|
| `.fi-no-lead .fi-lead-ico { display: none !important }` | Hides `.fi-lead-ico` SVG in interactive demo |
| `.fi-no-lead .inp svg.ico { display: none !important }` | Hides `.ico` SVG in States Reference cells |
| `.fi-no-lead .inp.lg-ico { padding-left: 16px }` | Removes icon indent in States Reference cells |
| `.fi-no-lead .fi-box .fi-inp { padding-left: 12px !important }` | Removes icon indent in interactive demo inputs |

---

---

### 6.13 Alert

> **Figma node:** `297:59` · MIH Design System Foundation  
> Component name: **LargeAlert**  
> Types: **Info · Success · Warning · Destructive** (Figma uses "Destructive" for Error)

The Alert component communicates system feedback. All color, text, and icon tokens share the same hex value per type — `icon/alert/*` and `text/alert/*` resolve to identical colors within each severity.

---

#### Figma Component Properties (all toggleable)

| Property | Type | Default | Description |
|---|---|---|---|
| `type` | `Info \| Success \| Warning \| Destructive` | Info | Severity variant — changes all tokens |
| `showIcon` | boolean | true | 20 px leading icon (Info circle / Check circle / Alert triangle / X octagon) |
| `showSubtext` | boolean | true | Subtitle text below the heading |
| `showIndicator` | boolean | false | Row of dot + "ระดับ" bullet indicators |
| `showBadge` | boolean | false | Row of StatusBadge chips matching the type |
| `showDetail` | boolean | true | White detail box (`surface/alertDialog/default`) with explain text |
| `showScrollBar` | boolean | true | 4 px vertical scroll indicator on the right |
| `showX` | boolean | true | 16 px close (×) button |

---

#### Color Tokens (from Figma variable_defs)

| Figma Alias Token | Info | Success | Warning | Destructive |
|---|---|---|---|---|
| `surface/alert/*` | `#f0f6ff` | `#f4fbfa` | `#fff9f0` | `#fff0f0` |
| `border/alert/*` | `#dbeafe` | `#d1f6ee` | `#ffedd5` | `#fee2e2` |
| `icon/alert/*` | `#1d4ed8` | `#196c65` | `#c2410c` | `#b91c1c` |
| `text/alert/*` | `#1d4ed8` | `#196c65` | `#c2410c` | `#b91c1c` |
| `surface/scrollBar/*-bar` | `#dbeafe` | `#d1f6ee` | `#ffedd5` | `#fee2e2` |
| `surface/scrollBar/*-pill` | `#93c5fd` | `#6cdcca` | `#fdba74` | `#fca5a5` |
| `surface/statusBadge/*` | `#dbeafe` | `#d1f6ee` | `#ffedd5` | `#fee2e2` |

**Shared tokens (type-independent):**
| Token | Hex | Usage |
|---|---|---|
| `surface/alertDialog/default` | `#ffffff` | Detail box background |
| `text/content/secondary` | `#636b72` | Detail box text |
| `text/content/default` | `#363b3f` | Indicator "ระดับ" label text |

---

#### Dimension Tokens

| Property | Token | Value | Role |
|---|---|---|---|
| Icon size | `dimension/size/400` | **20 px** | Leading severity icon |
| Icon top offset | `dimension/space/050` | **2 px** | Top padding on icon column |
| Close icon size | — | **16 px** | × button |
| ScrollBar width | `dimension/size/50` | **4 px** | Scrollbar track width |
| ScrollBar height | — | **150 px** | Scrollbar track height |
| ScrollBar pill height | — | **33 px** | Thumb height |
| ScrollBar radius | `dimension/radius/500` | **16 px** | Track & thumb radius |
| Container padding | `dimension/space/400` | **16 px** | All sides |
| Content gap | `dimension/space/200` | **8 px** | Between text block and detail box |
| Text gap | `dimension/space/100` | **4 px** | Between title and subtext |
| Icon ↔ content gap | `dimension/space/300` | **12 px** | Horizontal gap |
| Detail box padding | `dimension/space/300` | **12 px** | Internal padding |
| Container border radius | `dimension/radius/600` | **24 px** | Large Alert outer corners |
| Detail box border radius | `dimension/radius/200` | **8 px** | Detail box corners |
| Border width | `dimension/stroke/100` | **1 px** | Container and detail box border |

---

#### Icons (per type)

| Type | Figma Icon Name | SVG Shape |
|---|---|---|
| Info | Info | Circle with center dot + vertical bar |
| Success | Check circle | Circle with checkmark path |
| Warning | Alert triangle | Triangle with vertical bar |
| Destructive | X octagon | Octagon with vertical bar (shown as circle in preview) |

---

#### Typography

| Role | Token | Resolved | Usage |
|---|---|---|---|
| Title | `typograpphy/family/heading-content` · Bold · `typograpphy/size/base` | Sarabun 700 · 16px · lh 1.5 | Alert heading |
| Subtext | `typograpphy/family/body` · Regular · `typograpphy/size/sm` | Sarabun 400 · 14px · lh 1.5 | Subtitle / message |
| Detail text | `typograpphy/family/body` · Regular · `typograpphy/size/sm` | Sarabun 400 · 14px · lh 1.5 | Explain text in detail box |
| Badge label | `typograpphy/family/body` · Medium · `typograpphy/size/xs` | Sarabun 500 · 12px · lh 1.5 | StatusBadge chip text |
| Indicator | `typograpphy/family/body` · Regular · `typograpphy/size/xs` | Sarabun 400 · 12px · lh 1.5 | "ระดับ" bullet label |

---

#### X Button States (node 3064:4984)

> **Props:** `state?: "Default" | "Hover"` · `type?: "info" | "success" | "warning" | "error"`  
> **Source:** Figma node `3064:4984` — component set with 4 types × 2 states = 8 variants

The X button is a standalone component placed inside the Alert. Its hover background changes per alert type using `background/*/tertiary-hover` tokens. Note: **Success** uses the `positive` token namespace, not `success`.

##### Dimension Tokens

| Property | Token | Value | Role |
|---|---|---|---|
| Container size | `dimension/size/500` | **24 × 24 px** | Click target |
| Border radius | `dimension/radius/200` | **8 px** | Rounded corners |
| Icon size | — | **16 px** | X close icon |

##### Color Tokens — Default State

| Type | Token | Hex | Role |
|---|---|---|---|
| Info | `icon/alert/info` | `#1d4ed8` | Icon color |
| Success | `icon/alert/success` | `#196c65` | Icon color |
| Warning | `icon/alert/warning` | `#c2410c` | Icon color |
| Destructive | `icon/alert/error` | `#b91c1c` | Icon color |
| All | — | transparent | Background |

##### Color Tokens — Hover State

| Type | Token | Hover Bg | Icon color |
|---|---|---|---|
| Info | `background/info/tertiary-hover` | `#bfdbfe` | `#1d4ed8` |
| Success | `background/positive/tertiary-hover` | `#c5edda` | `#196c65` |
| Warning | `background/warning/tertiary-hover` | `#fed7aa` | `#c2410c` |
| Destructive | `background/danger/tertiary-hover` | `#fecaca` | `#b91c1c` |

> ⚠️ **Token namespace note:** Success uses `background/positive/*` (not `background/success/*`). Danger/Destructive uses `background/danger/*` (not `background/error/*`).

#---

#### Icon Reference

Icons in Form Builder are aliased **directly from the MIH Design System icon library**. Do not substitute arbitrary icons.

| Variant | Icon Name | Figma Node | Library | SVG Shape | Color Alias | Hex |
|---|---|---|---|---|---|---|
| Content card | **Type** | `2210:7778` | Simple Design System (Community) | T — horizontal bar top + vertical stem | `icon/formBuilder/content` | `#007549` |
| Signature area | **Edit 3** | `2286:10005` | Simple Design System (Community) | Diagonal pen stroke + horizontal baseline | `icon/formBuilder/default` | `#007549` |

**Type icon SVG** (Feather `type`, 20×20, viewBox 0 0 24 24):
```html
<svg width="20" height="20" viewBox="0 0 24 24" fill="none"
     stroke="#007549" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <!-- alias: icon/formBuilder/content -->
  <path d="M4 7V4h16v3"/>
  <line x1="9" y1="20" x2="15" y2="20"/>
  <line x1="12" y1="4" x2="12" y2="20"/>
</svg>
```

**Edit 3 icon SVG** (Feather `edit-3`, 20×20, viewBox 0 0 24 24):
```html
<svg width="20" height="20" viewBox="0 0 24 24" fill="none"
     stroke="#007549" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <!-- alias: icon/formBuilder/default -->
  <path d="M12 20h9"/>
  <path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/>
</svg>
```

> **Icon sizing:** both icons render at `dimension/size/400` = **20 px**. The icon box container (Content only) is `dimension/size/700` = **36 px** with background `icon/formBuilder/iconBox-default` = `#e2f3eb`.

#### States Summary

| Type | Default bg / icon | Hover bg / icon |
|---|---|---|
| Info | transparent / `#1d4ed8` | `#bfdbfe` / `#1d4ed8` |
| Success | transparent / `#196c65` | `#c5edda` / `#196c65` |
| Warning | transparent / `#c2410c` | `#fed7aa` / `#c2410c` |
| Destructive | transparent / `#b91c1c` | `#fecaca` / `#b91c1c` |

---

#### 6.13.1 SmallAlert

> **Figma node:** [`3355:3149`](https://www.figma.com/design/SBdh4TtY0KAa22s3dnyRNr/MIH-Design-System-Foundation?node-id=3355-3149) · MIH Design System Foundation  
> **Code:** [`src/components/alert/SmallAlert.jsx`](src/components/alert/SmallAlert.jsx)

Compact pill for **transient inline feedback** (e.g. copy to clipboard). Two states — **Success** · **Error** — each uses a **solid chromatic fill** (`surface/smallalert/*` → `Primitive/color/teal/500` · `Primitive/color/red/500`) with **white** label and leading icon (`text/smallAlert/on*` → `Primitive/color/grey/50`, matching Figma `text/positive/on-positive` and `text/danger/on-default`). This is intentionally **not** the same visual language as **LargeAlert** (`surface/alert/*` uses pastel 50-tier fills + toned body text).

| Property | Token / value | Role |
|---|---|---|
| State `Success` | `surface.smallAlert.success` | Pill background `#24A899` |
| State `Error` | `surface.smallAlert.error` | Pill background `#EF4444` |
| Label + icon color | `text.smallAlert.onSuccess` · `text.smallAlert.onError` | `#FFFFFF` (both modes in current spec) |
| Leading graphic | **24 × 24 px** | Figma Tick-on-circle (success) · X (error) — code uses Material Symbols `check_circle` / `cancel`, filled, same color as label |
| Label typography | `typograpphy/family/body` · Regular · `typograpphy/size/base` | Sarabun 400 · 16px · lh 1.5 |
| Horizontal padding | `dimension/space/400` · `dimension/space/600` | **16 px** leading · **24 px** trailing |
| Vertical padding | `dimension/space/200` | **8 px** top / bottom |
| Icon ↔ label gap | `dimension/space/100` | **4 px** |
| Corner radius | `dimension/radius/full` | Pill |
| Success shadow | `--shadow-smallalert-success` | Brand-tinted double drop shadow (Figma **Brand Drop Shadow Bottom/200**) |
| Error shadow | `--shadow-smallalert-error` | Neutral slate-tinted double drop shadow (Figma **Drop Shadow Bottom/200**) |

**React props:** `variant`: `'success'` \| `'error'` · optional `children` for custom copy (defaults: Thai strings above).

---

#### 6.13.2 AlertDialog

> **Figma node:** `1:1159` (MIH Design System · Popup file)
> Component: [`src/components/popup/AlertDialog.jsx`](src/components/popup/AlertDialog.jsx)

Compact confirmation modal for critical actions — distinct from `<Popup>` which has a full header/body/footer chrome. AlertDialog is a borderless centered card with a tone icon, title, supporting text, and a 2-button action row.

##### Property panel

| Property | Values | Default |
|---|---|---|
| `type` | `info` · `success` · `warning` · `destructive` | `info` |
| `alignment` | `center` · `left` | `center` |
| `title` / `description` | string | — |
| `cancelLabel` / `confirmLabel` | string | `ยกเลิก` / `ตกลง` |
| `showCancel` | boolean | `true` |

##### Dimension Tokens

| Property | Token | Value | Role |
|---|---|---|---|
| Width — Center | `dimension/size/2700` | **448 px** | Card max-width on Center alignment |
| Width — Left | fixed | **320 px** | Card max-width on Left alignment |
| Padding | `dimension/space/800` | **32 px** | All sides |
| Gap (content ↔ actions) | `dimension/space/600` | **24 px** | Outer flex gap |
| Gap (icon ↔ text — Center) | `dimension/space/300` | **12 px** | Column gap |
| Gap (icon ↔ title — Left) | `dimension/space/300` | **12 px** | Inline gap |
| Gap (title ↔ description) | `dimension/space/200` | **8 px** | Text-content gap |
| Gap (Cancel ↔ Confirm) | `dimension/space/300` | **12 px** | Actions row gap |
| Radius | `dimension/radius/500` | **16 px** | Card corner |
| Icon size — Center | `dimension/size/700` | **32 px** | Tone icon |
| Icon size — Left | `dimension/size/600` | **24 px** | Tone icon |
| Button height | `button.size.lg` | **44 px** | Both actions |

##### Surface / Text Tokens

| CSS var | Figma token | Hex (light) | Role |
|---|---|---|---|
| `--surface-alertdialog-default` | `surface/alertdialog/default` | `#FFFFFF` | Card bg |
| `--border-alertdialog-default` | `border/alertdialog/default` | `#E2E4E6` | 1 px card border |
| `--text-alertdialog-text` | `text/alertdialog/text` | `#4D5358` | Title color (Bold) |
| `--text-alertdialog-subtext` | `text/alertdialog/subtext` | `#636B72` | Description color |
| `--icon-alertdialog-info` | `icon/alertDialog/info` | `#007549` ★ | Info icon — **brand-aware** (primary/700) |
| `--icon-alertdialog-success` | `icon/alert/success` | `#196C65` | Success icon — teal/700 (fixed) |
| `--icon-alertdialog-warning` | `icon/alert/warning` | `#C2410C` | Warning icon — orange/700 (fixed) |
| `--icon-alertdialog-destructive` | `icon/alert/error` | `#B91C1C` | Destructive icon — red/700 (fixed) |

> ★ `icon/alertDialog/info` is the only icon token in the `alertDialog` namespace — it resolves to `primary/700` so it retints with `setBrandTheme()`. The other 3 tones reuse the banner-Alert tokens (`icon/alert/{success,warning,error}`), which are fixed semantic colors that should not shift across brands.

##### Type matrix

| Type | Icon | Tone token | Primary button | Secondary button |
|---|---|---|---|---|
| Information | `info` | `--icon-alertdialog-info` → primary/700 ★ | `<BrandButton>` (green) | `<BrandSubdueButton variant='outline'>` |
| Success | `check_circle` | `--icon-alertdialog-success` → teal/700 | `<BrandButton>` | `<BrandSubdueButton variant='outline'>` |
| Warning | `warning` | `--icon-alertdialog-warning` → orange/700 | `<DangerButton>` (red) | `<DangerSubdueButton variant='outline'>` |
| Destructive | `cancel` | `--icon-alertdialog-destructive` → red/700 | `<DangerButton>` | `<DangerSubdueButton variant='outline'>` |

> ★ **Warning button family note:** Figma spec calls for orange action buttons on the Left + Warning combination. The DS button library currently has no Warning sibling, so AlertDialog collapses Warning to the Danger button family (red) for both alignments. This keeps DS button aliasing clean — if a true orange Warning button is later added, AlertDialog will pick it up automatically via the type→button-family map in [`AlertDialog.jsx`](src/components/popup/AlertDialog.jsx).

##### Behaviour

- `role="alertdialog"` + `aria-modal="true"`, with `aria-labelledby` and `aria-describedby` wired to the title/description.
- Esc and backdrop click both fire `onClose`; the primary button fires `onConfirm`.
- Locks body scroll while open (stack-aware — composes with `<Popup>` and other dialogs).
- Renders via `createPortal` to `document.body` so the dialog escapes any `overflow: hidden` ancestor.
- **Enter / exit motion** — same contract as [`Popup`](#6133-popup) § Modal enter / exit motion (`useModalMotion` + `.mih-popup-overlay` / `.mih-popup-dialog`).

##### When to use

- One-shot reversible-with-effort actions: delete, sign out, discard draft, revert change.
- Soft confirmations where a single decision is enough — no form, no multi-step.
- **Don't** use AlertDialog for inputs or wizard flows — reach for `<Popup size='large'>` or `<Popup size='extraLarge'>` instead.

#### 6.13.3 Popup

> **Figma nodes:** `1:424` (Default) · `1:564` (Large) · `1:639` / `75:35675` (Extra Large — Popup Very Large)  
> **Component:** [`src/components/popup/Popup.jsx`](src/components/popup/Popup.jsx)

Full chrome modal — header (title · subtitle · close) · scrollable body · optional footer (tertiary → secondary → primary). Distinct from [`AlertDialog`](src/components/popup/AlertDialog.jsx) (compact confirmation card).

##### Size variants

| `size` | Base width | Max behaviour | Typical use |
|---|---|---|---|
| `default` | **448 px** | `min(448px, 100vw − 48px)` | Confirmations, short forms |
| `large` | **640 px** | `min(640px, 100vw − 48px)` | Medium forms, pickers |
| `extraLarge` | **1440 px** | Width **and** height fill viewport up to caps below | Body map, vitals graph, settings |

##### `extraLarge` — fluid viewport box

When `size='extraLarge'` (do **not** pass a fixed `width` — let the size token drive layout):

| Property | Value | Token / note |
|---|---|---|
| Max width | **1440 px** | `VIEWPORT_MAX_WIDTH` |
| Max height | **1024 px** | `VIEWPORT_MAX_HEIGHT` |
| Viewport inset | **24 px** each edge | `--dim-space-600` (`VIEWPORT_EDGE_INSET`) |
| Resolved width | `min(1440px, calc(100vw − 48px))` | Grows with screen below cap |
| Resolved height | `min(1024px, calc(100vh − 48px))` | Dialog stretches vertically on short viewports |
| Overlay padding | 24 px all sides | Centers panel inside dimmed backdrop |

On narrow viewports (e.g. tablet preview) the dialog shrinks with the screen; on wide desktops it approaches **1440×1024** before hitting the cap.

##### Domain wrappers (all use `extraLarge`)

| Wrapper | Figma | File | Notes |
|---|---|---|---|
| `DrawingCanvasPopup` | PHCIS Medical-examination `75:35675` | [`DrawingCanvasPopup.jsx`](src/components/popup/DrawingCanvasPopup.jsx) | **Reusable shell** — hosts `BodyDrawingCanvas` fillHeight; built-in dirty tracking (Save disabled until canvas reports any user edit via `onDirtyChange`); mirrors `markers` + `drawings` + `viewKey` between parent + canvas; resets only on the `open ↑` transition (so parent reflections don't clobber dirty). Props: `title`, `subtitle?`, `viewKey?`, `markers?`, `onMarkersChange?`, `drawings?`, `onDrawingsChange?`, `onSave(markers, drawings)`, `onClose`, `saveLabel?`, `cancelLabel?`, `minBodyHeight?`. Domain wrappers compose this. |
| `BodyRecordPopup` | PHCIS Medical-examination `75:35675` | [`BodyRecordPopup.jsx`](src/components/examination/BodyRecordPopup.jsx) | Thin wrapper over `DrawingCanvasPopup` — resolves `title` from `systemKey` via `EXAM_SYSTEM_TABS` (HEENT / Heart / Lungs / Abdomen / Extremities). All other props forward unchanged. |
| `VitalsGraphPopup` | PHCIS-ALL `3185:144347` / `2745:104642` | [`VitalsGraphPopup.jsx`](src/components/popup/VitalsGraphPopup.jsx) | Wraps `VitalsSignGraph` with `fillHeight` — chart area grows to fill popup body (min 508px); footer ยกเลิก · ปิด. Prefer [`VitalsSignGraphDialog`](src/components/vitals/VitalsSignGraphDialog.jsx) for trigger + popup |

**Preview:** `/components` → **Popup** · **DrawingCanvasPopup** · **BodyRecordPopup** · **VitalsGraphPopup** · **VitalsSignGraphDialog**.

##### Behaviour

- `role="dialog"` · `aria-modal="true"` · portal to `document.body`.
- Esc and backdrop click call `onClose` (clicks inside the panel do not close).
- Stack-aware body scroll lock (composes with `AlertDialog` and nested modals).
- **Enter / exit motion** — see § Modal enter / exit motion below.

##### Modal enter / exit motion (Popup family)

Every **centered modal** in the app — `<Popup>`, `<AlertDialog>`, and all domain wrappers that compose them (`DrawingCanvasPopup`, `AppointmentHistoryPopup`, `MedicalHistoryPopup`, `VitalsGraphPopup`, `AddMedicationPopup`, …) — shares one motion contract:

| Layer | CSS class | Enter | Exit | Duration / easing |
|---|---|---|---|---|
| Backdrop | `.mih-popup-overlay` | opacity **0 → 1** | **1 → 0** | `--motion-duration-modal` · `--motion-easing-standard` |
| Panel | `.mih-popup-dialog` | opacity **0 → 1** · `translateY(--motion-panel-offset-y) → 0` · `scale(--motion-panel-scale-from) → 1` | reverse | same |

**Implementation**

| Piece | File | Notes |
|---|---|---|
| Hook | [`useModalMotion.js`](src/components/popup/useModalMotion.js) | `mounted` keeps the portal in the DOM through exit; `motionState` drives `data-state="visible"\|"hidden"` |
| Styles | [`src/index.css`](src/index.css) | `.mih-popup-overlay` · `.mih-popup-dialog` |
| Constants | `MODAL_MOTION_MS` (= `--motion-duration-modal`) | Unmount delay after `open` → `false` |

**Caller rule:** always render `<Popup open={open} />` / `<AlertDialog open={open} />` and toggle `open` — **never** `{open && <Popup … />}` or the exit animation is skipped.

**Anchored popovers** (`Popover`, `ProfilePopover`, search panels) are **not** centered modals — they use `.mih-popover-in` with `--motion-duration-popover` instead.

##### Popup-family chrome tokens (2026-05-25 audit)

Every popup container — `<Popup>`, `<AlertDialog>`, `<ProfilePopover>`, plus anything composing them (`ScreeningConfirmPopup`, `DrawingCanvasPopup`, `VitalsGraphPopup`, etc.) — uses the **same** chrome:

| Slot | Token | Resolves to | Figma origin |
|---|---|---|---|
| Container shadow | `--shadow-drop-bottom-400` | `0 4px 4px rgba(100,116,139,.10), 0 16px 32px rgba(100,116,139,.15)` | `effect.drop-shadow-bottom.400` |
| Container radius | `--dim-radius-500` (16 px) | 16 px | `dimension.radius.500` |
| Backdrop overlay | `--surface-overlay-default` | `rgba(100,116,139,.40)` (slate-500 @ 40%) | aligned with drop-shadow-bottom slate family |

Pre-audit drift that was fixed:
- `<Popup>` and `<AlertDialog>` referenced `--shadow-dropdown` (functionally identical but un-spec'd name) — now reference `--shadow-drop-bottom-400` directly. `--shadow-dropdown` remains as a back-compat alias.
- Three popups hardcoded their backdrop (`rgba(15,23,42,.4)` slate-900 / `rgba(0,0,0,.55)` pure-black) — now standardized on `--surface-overlay-default` so the overlay tint matches the slate-500 family used by the container shadow.
- `<ScreeningConfirmPopup>` and `<CameraCapturePopup>` used non-spec radii (`--shadow-card`, `--dim-radius-300`) — migrated.

**Rule:** any new popup-shaped component MUST use these three tokens. Don't reach for `--shadow-card`, `--shadow-card-hover`, or hardcoded `rgba(...)` backdrops — those belong to the card family.

---

### 6.14 Dropdown Menu

> **Figma nodes:** `328:4440` (Default panel) · `311:3` (List Item) · `382:14565` (Menu variant)  
> **MIH Design System Foundation** · Used in: filter chips, context menus, user-profile menus

The Dropdown renders a floating panel containing a list of selectable items. Each row may carry a leading icon, label, optional switch, badge, checkmark, or chevron-right. A **Menu** variant adds a user-profile header with avatar, name, and role.

> **Selected vs Default visual note:** `surface/list/selected` and `text/list/selected` resolve to the **same hex values** as their Default counterparts. The only visual cue for a selected row is the checkmark icon (`showCheck = true`, color `#08A768`).

---

#### Panel Container Tokens

| Figma Alias Token | Light Hex | Primitive | Usage |
|---|---|---|---|
| `surface/dropdown/default` | `#ffffff` | white | Panel background |
| `border/dropdown/default` | `#ebebeb` | neutral-150 | Panel border (1 px) |
| `color/shadow/100` | `rgba(100,116,139,.15)` | slate-500/15% | Outer shadow layer: `0 16px 32px` |
| `color/shadow/050` | `rgba(100,116,139,.10)` | slate-500/10% | Inner shadow layer: `0 4px 4px` |

---

#### Panel Dimension Tokens

| Property | Token | Value | Role |
|---|---|---|---|
| Panel width | _(fixed)_ | **264 px** | Container width |
| Panel border | `dimension/stroke/100` | **1 px** | Border width |
| Panel radius | `dimension/radius/500` | **16 px** | Panel corner radius |
| Inner padding | `dimension/space/100` | **4 px** | Padding around list |
| Item width | `dimension/size/2400` | **256 px** | List-item width (264 − 4×2) |
| Item min-height | _(min-height)_ | **48 px** | Single-line row minimum |
| Item radius | `dimension/radius/100` | **6 px** | List-item corner radius |
| Padding H | `dimension/space/300` | **12 px** | Row horizontal padding |
| Padding V / Gap | `dimension/space/200` | **8 px** | Row vertical padding & icon gap |
| Icon size | `dimension/size/300` | **16 px** | Leading & trailing icons |

---

#### List Item Color Tokens by State

| State | Surface token | CSS var | Text token | CSS var | Icon token | CSS var | Brand? |
|---|---|---|---|---|---|---|---|
| **Default** | `surface/list/default` | `--surface-list-default` | `text/list/default` | `--text-list-default` | `icon/list/secondary-default` | `--icon-list-secondary-default` | — |
| **Hover** | `surface/list/hover` | `--surface-list-hover` | `text/list/hover` | `--text-list-hover` | `icon/list/primary-selected-hover` | `--icon-list-primary-selected-hover` | ✓ |
| **Selected** | `surface/list/selected` | `--surface-list-default` ¹ | `text/list/selected` | `--text-list-default` ¹ | `icon/list/secondary-default` | `--icon-list-secondary-default` | — |
| **Selected-Hover** | `surface/list/selected-hover` | `--surface-list-selected-hover` | `text/list/selected-hover` | `--text-list-hover` | `icon/list/primary-selected-hover` | `--icon-list-primary-selected-hover` | ✓ |

> ¹ `surface/list/selected` and `text/list/selected` resolve to identical hex as Default state — the checkmark is the sole visual differentiator.

**Resolved hex values (Emerald / default brand):**

| CSS var | Primitive | Light hex | Brand-responsive |
|---|---|---|---|
| `--surface-list-default` | fixed white | `#ffffff` | — |
| `--surface-list-hover` | `primary/50` → `var(--em50)` | `#f4fbf8` | ✓ `setBrandTheme()` |
| `--surface-list-selected-hover` | `primary/50` → `var(--em50)` | `#f4fbf8` | ✓ `setBrandTheme()` |
| `--text-list-default` | `neutral/700` (fixed) | `#4d5358` | — |
| `--text-list-hover` | `primary/700` → `var(--em700)` | `#007549` | ✓ `setBrandTheme()` |
| `--icon-list-secondary-default` | `neutral/500` (fixed) | `#858c92` | — |
| `--icon-list-primary-selected-hover` | `primary/700` → `var(--em700)` | `#007549` | ✓ `setBrandTheme()` |

> **Note on icon alias name:** The correct Figma token for icon hover/selected-hover is `icon/list/primary-selected-hover`, **not** `icon/list/hover`. The CSS var `--icon-list-primary-selected-hover` is updated by `setBrandTheme()` alongside `--text-list-hover`.

---

#### Optional Element Tokens

| Element | Prop | Alias Token | Hex | Notes |
|---|---|---|---|---|
| Checkmark icon | `showCheck` | _(color hardcoded)_ | `#08A768` | Shown only when `selected` or `selected-hover` |
| Switch (off) | `showSwitch` | `surface/switch/off` | `#e2e4e6` | Track background when toggle is OFF |
| Badge background | `showBadge` | `surface/colorfulbadge/blue` | `#dbeafe` | Inline metadata badge |
| Badge text | `showBadge` | `text/colorfulbadge/blue` | `#1d4ed8` | Badge label color |
| Chevron-right | `showChevronRight` | `icon/list/trailing` _(ref)_ | `#adb2b7` | Sub-menu indicator |
| Divider | — | `border/neutral/quaternary` | `#f4f5f5` | Section separator line |

---

#### Dropdown/Menu Variant Tokens

The **Menu** variant prepends a user-profile header above the list items.

| Element | Alias Token | Hex | Notes |
|---|---|---|---|
| User name | `text/modal/text` | `#4d5358` | Full name label |
| User role/subtitle | `text/modal/subtext` | `#adb2b7` | Role or department line |
| Header divider | `border/neutral/quaternary` | `#f4f5f5` | Line below header |
| Avatar background | `surface/tag/brand/default` _(ref)_ | `#e2f3eb` | Avatar placeholder fill |

---

#### States Summary

| State | Trigger | Visual change |
|---|---|---|
| **Default** | Panel opens | `var(--surface-list-default)` bg · `var(--text-list-default)` text · `var(--icon-list-secondary-default)` icon |
| **Hover** | Mouse over row | `var(--surface-list-hover)` bg · `var(--text-list-hover)` text · `var(--icon-list-primary-selected-hover)` icon — all brand-responsive |
| **Selected** | Row clicked | Checkmark appears; bg/text/icon unchanged from Default |
| **Selected-Hover** | Mouse over selected row | `var(--surface-list-selected-hover)` bg · `var(--text-list-hover)` text · `var(--icon-list-primary-selected-hover)` icon — all brand-responsive |

> **Implementation note:** Hover styles are applied via CSS `.ddm-item:hover` and `.ddm-item.selected-hover`. Static reference card rows use inline `style="background:var(--surface-list-hover)"` + `style="color:var(--text-list-hover)"` + `stroke="currentColor"` on SVGs. Never hardcode `#f4fbf8` or `#007549` — always alias through the CSS vars so brand switching works correctly.

---

#### Interactive Behavior

- Panel floats 4 px below trigger (gap = `dimension/space/100`)
- Clicking outside the panel closes it
- Only one item can be selected at a time in single-select mode
- Switch toggle is independent of row selection state
- Menu variant: clicking logout/settings items triggers navigation; no checkmark shown

---

### 6.15 Tab

> Figma node 367-9620 · 3 variants: **Tab Underline** · **Tab Pill** · **Tab Capsule**

---

#### Interactive Demo

The Tab component preview exposes three live variants simultaneously:

| Variant | States shown | Interaction |
|---|---|---|
| **Tab Underline** | Default · Hover · Active · Active Hover · Disabled | Click tab → see text/border tokens; hover over active tab → `surface/tabcapsule/hover` bg |
| **Tab Pill** | Default · Hover · Selected · Disabled | Click to select; pill lifts with shadow |
| **Tab Capsule** | Default · Hover · Selected · Disabled · In Progress · Success | Click any capsule to inspect its token |

Below the interactive demo, a **state grid** (same format as Button) presents all states × sizes side-by-side for design reference.

---

#### 6.15.1 Tab Underline

> Sizes: Large 44px / Medium 40px / Small 36px · States: Default · Hover · Active · Active Hover · Disabled

##### Alias Tokens

| CSS var | Figma token | → Brand | Primitive | Usage |
|---|---|---|---|---|
| `--text-tu-default` | `text/tabunderline/default` | neutral-700 | `var(--n700)` = #4D5358 | Default + Hover label text |
| `--text-tu-selected` | `text/tabUnderline/selected` | **primary/700** | `var(--em700)` | Active + Active Hover label text |
| `--text-tu-disabled` | `text/tabunderline/disabled` | neutral-300 | `var(--n300)` = #C9CDD0 | Disabled label text |
| `--border-tab-selected` | `border/tab/selected` | **primary/700** | `var(--em700)` | Active underline bar — Selected & Active Hover |
| `--surface-brandprimary-quaternary` | `surface/brandprimary/quaternary` | **primary/50** | `var(--em50)` | Hover + Active Hover background |

> ⚠️ **CSS var naming:** CSS custom properties ไม่รองรับ `/` ในชื่อตัวแปร ต้องใช้ชื่อย่อ (`--border-tab-selected`, `--text-tu-selected`) แทน Figma token path (`border/tab/selected`) เสมอ

> **Figma variable IDs (Tab Underline active state — node 2207:7602):**
> - Stroke (underline bar): `VariableID:2396:1888` → `border/tab/selected` → `primary/700` → `--border-tab-selected`
> - Text fill: `VariableID:2396:1880` → `text/tabUnderline/selected` → `primary/700` → `--text-tu-selected`
> - Icon fill: `VariableID:2396:1876` → `icon/tabUnderline/selected` → `primary/700` → `--text-tu-selected`
>
> ทั้งสามตัว resolve เป็น **`primary/700`** → `--em700` — `setBrandTheme()` อัปเดตอัตโนมัติเมื่อเปลี่ยน brand

##### Dimension Tokens

| Property | Dimension Token | Value | Role |
|---|---|---|---|
| Height — Large | `dimension/size/900` | **44 px** | `.tu-lg` |
| Height — Medium | `dimension/size/800` | **40 px** | `.tu-md` |
| Height — Small | `dimension/size/700` | **36 px** | `.tu-sm` |
| Padding X | `dimension/space/600` | **24 px** | Left/right inner spacing |
| Active border — Large/Medium | `dimension/stroke/400` | **4 px** | Bottom underline bar |
| Active border — Small | `dimension/stroke/200` | **2 px** | Bottom underline bar |
| Icon size — Large/Medium | `dimension/size/400` | **16 px** | Leading icon |
| Icon size — Small | `dimension/size/350` | **14 px** | Leading icon |
| Font size — Large/Medium | `typograpphy/size/base` | **16 px** | Label |
| Font size — Small | `typograpphy/size/xs` | **12 px** | Label |

##### States

| State | Text Token | Background | Border bottom |
|---|---|---|---|
| Default | `text/tabunderline/default` → neutral-700 | — | — |
| Hover | `text/tabunderline/default` → neutral-700 | `surface/brandprimary/quaternary` → **primary/50** | — |
| Active (Selected) | `text/tabUnderline/selected` → **primary/700** · Bold | — | `border/tab/selected` → **primary/700** · 4px (Lg/Md) / 2px (Sm) |
| Active Hover | `text/tabUnderline/selected` → **primary/700** · Bold | `surface/brandprimary/quaternary` → **primary/50** | `border/tab/selected` → **primary/700** · 4px (Lg/Md) / 2px (Sm) |
| Disabled | `text/tabunderline/disabled` → neutral-300 | — | — |

> **Theme note:** Active and Active Hover states alias `border/tab/selected` → `primary/700` → `--em700`. Changing the brand theme updates the underline bar and text color automatically.

##### Underline width (animated + static)

The active underline — the **shared sliding ink** when `animated` is on, or the **per-tab** `scaleX` bar when `animated` is off — spans **only the label text box**, not the full tab button width. The **leading icon** and optional **count** chip sit outside that span so the bar tracks **word length** (e.g. short English vs long Thai copy). In code the label is wrapped in `[data-tabunderline-ink]` and layout measures that node for indicator `x` / `width`; if it is missing, measurement falls back to the tab button with horizontal inset (`dimension/space/300`).

##### Overflow Condition

> Figma node **2867:4170** — "Neutral Button" (more_horiz + chevron_down). Shown only when the tab strip cannot render every tab within the container width.

When the available width cannot fit all tabs, `<TabUnderline>` auto-hides the trailing tabs and renders a single **More** trigger on the right. Clicking it opens a `surface/dropdown/default` popover listing the hidden tabs. Selecting a hidden tab calls the same `onChange(key)` and closes the popover.

| Property | Token / Value | Role |
|---|---|---|
| Trigger gap | `dimension/space/150` = **6 px** | between `more_horiz` and `keyboard_arrow_down` |
| Trigger padding X | `dimension/space/300` = **12 px** | left / right inner spacing |
| Trigger padding Y | `dimension/space/150` = **6 px** | top / bottom inner spacing |
| Trigger radius | `dimension/radius/200` = **8 px** | rounded corners |
| Trigger icon size | `dimension/size/400` = **16 px** | both icons |
| Trigger hover bg | `surface/list/hover` (brand-responsive) | only on hover / when popover open |
| Trigger icon — default | `icon/tabunderline/default` → neutral-700 | resting tone |
| Trigger icon — hover | `icon/tabunderline/hover` → primary/600 | mouse over the trigger |
| Trigger icon — selected-in-overflow | `icon/tabunderline/selected` → primary/700 | when the currently-active tab is hidden in the dropdown |
| Popover surface | `surface/dropdown/default` · `border/dropdown/default` · `shadow/dropdown` | matches dropdown menus elsewhere |
| Popover position | fixed, anchored to trigger's bottom-right, `z = --sm-z-popover (1199)` | unaffected by `overflow:hidden` ancestors |
| Item row | `min-h 40` · `px 12` · `py 8` · `gap 8` · `radius 6` | identical to `<SuggestionList>` rows |

**Measurement rule.** Visible tabs are determined greedily: sum tab widths left-to-right, stop at the first tab whose right edge would exceed `container.clientWidth − triggerWidth`. The last tab needs no trigger reservation, so it gets the full container width. Re-measure on `ResizeObserver` ticks and on `tabs` prop changes; hidden tabs use cached widths so the comparison stays stable.

**Selected tab in overflow.** The sliding indicator hides (it has no anchor), the More trigger paints the selected tone, and the dropdown row for that tab gets `surface/list/selected` background. Selecting any other tab via the dropdown brings the indicator back and resets the trigger tone.

**Close behaviour.** Outside `mousedown`, `Escape`, and `window.resize` all dismiss the popover. The trigger's `onClick` toggles open ↔ close.

##### Filter toolbar overflow (queue pages)

> Reuses Figma **2867:4170** — the same **More** trigger (`more_horiz` + `keyboard_arrow_down`) and dropdown chrome as `TabUnderline` overflow (§6.15.1). Implemented as `<OverflowFlexRow>` in code.

When the horizontal space in the list card cannot fit every `FilterTag` / `FilterDropdown` / `TagDivider` / `FilterDateTag` in the toolbar, trailing items are **hidden in the main row** (`display: none` on measured wrappers) and duplicated in a **fixed** `surface/dropdown/default` popover opened from the More trigger. **Measurement rule** matches tabs: greedy left-to-right width sum, reserve trigger width while any items remain to the right; **re-measure** on `ResizeObserver` on the row container.

**Popover interaction.** Clicks inside portalled filter UI (`.mih-search-popover`, modal `role="dialog"`) do **not** dismiss the overflow menu so nested dropdowns and `DateFilterPopup` stay usable.

**Implementation.** OPD Queue (`QueuePage.jsx`), MR registration queue (`RegistrationQueuePage.jsx`), and Treatment queue (`TreatmentQueuePage.jsx`) wrap the chip row with `<OverflowFlexRow>`.

---

#### 6.15.2 Tab Pill

> Sizes: Default 32px / Large 48px · Styles: Text · Icon · States: Default · Hover · Selected · Disabled

##### Alias Tokens

| CSS var | Figma token | Primitive | Hex (light) | Hex (dark) | Usage |
|---|---|---|---|---|---|
| `--text-tabpill-default` | `text/tabpill/default` | **primary/700** ↔ **primary/300** | `#007549` | `#91DFBB` | Default + Hover + Selected label (brand-aware) |
| `--text-tabpill-hover` | `text/tabpill/hover` | **primary/700** ↔ **primary/300** | `#007549` | `#91DFBB` | Hover label (brand-aware) |
| `--text-tabpill-selected` | `text/tabpill/selected` | **primary/700** ↔ **primary/300** | `#007549` | `#91DFBB` | Selected label (brand-aware) |
| `--text-tabpill-disabled` | `text/tabpill/disabled` | **primary/500** | `#54CF97` | `#54CF97` | Disabled label (brand-aware, both modes share) |
| `--icon-tabpill-default` | `icon/tabpill/default` | **primary/700** ↔ **primary/300** | `#007549` | `#91DFBB` | Default + Hover + Selected icon |
| `--icon-tabpill-hover` | `icon/tabpill/hover` | **primary/700** ↔ **primary/300** | `#007549` | `#91DFBB` | Hover icon |
| `--icon-tabpill-selected` | `icon/tabpill/selected` | **primary/700** ↔ **primary/300** | `#007549` | `#91DFBB` | Selected icon |
| `--icon-tabpill-disabled` | `icon/tabpill/disabled` | **primary/500** | `#54CF97` | `#54CF97` | Disabled icon |
| `--surface-tabpill-background` | `surface/tabpill/background` | `emerald/100` ↔ `emerald/900` | `#E2F3EB` | `#003322` | TabBar parent container |
| `--surface-tabpill-default` | `surface/tabpill/default` | `grey/50` ↔ `neutral/800` | `#FFFFFF` | `#363B3F` | Default + Selected + Disabled pill bg |
| `--surface-tabpill-hover` | `surface/tabpill/hover` | **primary/200** ↔ **primary/800** | `#C5EDDA` | `#004C31` | Hover pill bg (brand-aware) |
| `--surface-tabpill-selected` | `surface/tabpill/selected` | `grey/50` ↔ `neutral/800` | `#FFFFFF` | `#363B3F` | Selected pill bg (alias of default) |
| `--surface-tabpill-disabled` | `surface/tabpill/disabled` | `grey/50` ↔ `neutral/800` | `#FFFFFF` | `#363B3F` | Disabled pill bg (alias of default) |

##### Dimension Tokens

| Property | Dimension Token | Value | Role |
|---|---|---|---|
| Height — Default | `dimension/size/600` | **32 px** | `.tp-item` default |
| Height — Large | — | **48 px** | `.tp-item.tp-lg` (via padding) |
| Border radius | `dimension/radius/full` | **9999 px** | Pill shape |
| Padding — Default | — | `6px 12px` | Inner spacing |
| Padding — Large | `dimension/space/400` / `dimension/space/300` | `12px 16px` | Inner spacing |
| Gap | `dimension/space/150` | **6 px** | Icon ↔ label |
| Icon — Default | `dimension/size/350` | **14 px** | Leading icon |
| Icon — Large | `dimension/size/400` | **16 px** | Leading icon |
| Font — Default | `typograpphy/size/sm` | **14 px / Medium** | Label |
| Font — Large | `typograpphy/size/base` | **16 px / Medium** | Label |
| Shadow (hover/selected) | `Drop Shadow Bottom/200` | `0 1px 4px rgba(100,116,139,.15)` | Elevated appearance |

---

#### 6.15.3 Tab Capsule

> Size: Default 40px · Style: Brand · Variants: Default (pill icon) / In Progress (orange dot) / Success (tick icon) · States per variant: Default · Hover · Active · Disabled · Figma node: 2612-7791

##### Dimension Tokens

| Property | Token / value | Role |
|---|---|---|
| Height | `dimension/size/800` · **40 px** | Fixed chip height |
| Padding | `dimension/space/300` × `dimension/space/100` · **12 × 4 px** | Horizontal / vertical inset |
| Gap | `dimension/space/150` · **6 px** | Leading icon ↔ label ↔ trailing icon |
| Border radius | pill · **9999 px** | Capsule shape |
| **Width (filter tag)** | **hug content** | `variant="filter"` — label-only chips; width follows copy (CPOE [`176:38632`](https://www.figma.com/design/cOjwwHvgGURA3IEuls8RT5/PHCIS-%7C-CPOE?node-id=176-38632)). No default `min-width` / `max-width`. |
| Filter row gap | `dimension/space/200` · **8 px** | Between capsules in `Tag group` (`176:38632`). |
| Width cap (optional) | `dimension/size/2200` · **192 px** | Pass `maxWidth` on `TabCapsule` only when truncation is required (DS variant default; not used in Step 1 filter row). |

##### Alias Tokens

| CSS var | Figma token | Primitive | Value | Usage |
|---|---|---|---|---|
| `--surface-tabcapsule-default` | `surface/tabcapsule/default` | white | `#FFFFFF` | Default background (fixed) |
| `--surface-tabcapsule-hover` | `surface/tabcapsule/hover` | **primary/100** | `var(--em100)` | Hover background — brand |
| `--surface-tabcapsule-selected` | `surface/tabcapsule/selected` | **primary/100** | `var(--em100)` | Selected background — brand |
| `--surface-tabcapsule-disabled` | `surface/tabcapsule/disabled` | white | `#FFFFFF` | Disabled background (fixed) |
| `--surface-tabcapsule-success` | `surface/tabcapsule/success` | white | `#FFFFFF` | Success + InProgress background (fixed) |
| `--border-tabcapsule-default` | `border/tabcapsule/default` | neutral-200 | `#E2E4E6` | Default border (fixed) |
| `--border-tabcapsule-hover` | `border/tabcapsule/hover` | **primary/200** | `var(--em200)` | Active-state border — brand *(Figma naming swapped)* |
| `--border-tabcapsule-selected` | `border/tabcapsule/selected` | **primary/200** | `var(--em200)` | Hover-state border — brand *(Figma naming swapped)* |
| `--border-tabcapsule-disabled` | `border/tabcapsule/disabled` | neutral-200 | `#E2E4E6` | Disabled border (fixed) |
| `--text-tabcapsule-default` | `text/tabcapsule/default` | neutral-700 | `#4D5358` | Default label (fixed) |
| `--text-tabcapsule-hover` | `text/tabcapsule/hover` | **primary/700** | `var(--em700)` | Hover label — brand |
| `--text-tabcapsule-selected` | `text/tabcapsule/selected` | **primary/700** | `var(--em700)` | Selected label — brand |
| `--text-tabcapsule-disabled` | `text/tabcapsule/disabled` | neutral-300 | `#C9CDD0` | Disabled label (fixed) |
| `--text-tabcapsule-inprogress` | `text/tabcapsule/inProgress` | neutral-700 | `#4D5358` | In Progress label (fixed) |
| `--text-tabcapsule-success` | `text/tabcapsule/success` | neutral-700 | `#4D5358` | Success label (fixed) |

> ⚠ **Figma token-name swap (Default variant only):** `border/tabcapsule/hover` and `border/tabcapsule/selected` are named opposite to their state. Hover state receives `border/tabcapsule/selected`; Active state receives `border/tabcapsule/hover`. Both resolve to `primary/200`. In Progress and Success variants follow standard (non-swapped) naming.
>
> **Theme note:** All brand-derived Tab Capsule tokens alias `primary/*` → `--em*` in CSS. The border, background, and text of Hover/Active states update automatically when the brand theme changes via `setBrandTheme()`.

##### States × Variants

**Default variant** · Pill icon · `<svg>` list icon

| State | Background | Border | Text |
|---|---|---|---|
| Default | `surface/tabcapsule/default` `#FFFFFF` | `border/tabcapsule/default` `#E2E4E6` | `text/tabcapsule/default` `#4D5358` |
| Hover | `surface/tabcapsule/hover` → **primary/100** | `border/tabcapsule/selected` ⚠ → **primary/200** | `text/tabcapsule/hover` → **primary/700** |
| Active | `surface/tabcapsule/selected` → **primary/100** | `border/tabcapsule/hover` ⚠ → **primary/200** | `text/tabcapsule/selected` → **primary/700** |
| Disabled | `surface/tabcapsule/disabled` `#FFFFFF` | `border/tabcapsule/disabled` `#E2E4E6` | `text/tabcapsule/disabled` `#C9CDD0` |

**In Progress variant** · Orange (amber) 6 px dot · shares `surface/tabcapsule/success` (#FFF) for default/disabled bg

| State | Background | Border | Text |
|---|---|---|---|
| Default | `surface/tabcapsule/success` `#FFFFFF` | `border/tabcapsule/inprogress` `#E2E4E6` | `text/tabcapsule/inprogress` `#4D5358` |
| Hover | `surface/tabcapsule/hover` → **primary/100** | `border/tabcapsule/hover` → **primary/200** | `text/tabcapsule/inprogress` `#4D5358` |
| Active | `surface/tabcapsule/selected` → **primary/100** | `border/tabcapsule/selected` → **primary/200** | `text/tabcapsule/inprogress` `#4D5358` |
| Disabled | `surface/tabcapsule/success` `#FFFFFF` | `border/tabcapsule/inprogress` `#E2E4E6` | `text/tabcapsule/inprogress` `#C9CDD0` |

**Success variant** · TickOnCircle icon (#D1FAE5 bg, #007549 tick) · shares `surface/tabcapsule/success` (#FFF) for default/disabled bg

| State | Background | Border | Text |
|---|---|---|---|
| Default | `surface/tabcapsule/success` `#FFFFFF` | `border/tabcapsule/success` `#E2E4E6` | `text/tabcapsule/success` `#4D5358` |
| Hover | `surface/tabcapsule/hover` → **primary/100** | `border/tabcapsule/hover` → **primary/200** | `text/tabcapsule/success` `#4D5358` |
| Active | `surface/tabcapsule/selected` → **primary/100** | `border/tabcapsule/selected` → **primary/200** | `text/tabcapsule/success` `#4D5358` |
| Disabled | `surface/tabcapsule/success` `#FFFFFF` | `border/tabcapsule/success` `#E2E4E6` | `text/tabcapsule/success` `#C9CDD0` |

#### Interactive Behavior

- **Variant selector** — 3 buttons (Default / In Progress / Success); clicking `capSetVariant(v, btn)` switches the active variant class and re-renders the capsule bar
- **Live capsule bar** — 4 tabs rendered by JS (`capInit → render`); Tab 4 is always disabled
- **Hover**: mouseover → apply hover tokens + display hover chips; mouseleave → restore default tokens
- **Click**: sets active tab index; applies active bg/border/text tokens with `font-weight:700`
- **Token display** (`#icap-info`) — updates on every hover/click to show the 3 token chips (bg · bd · tx) with hex swatches for the current state × variant combination

##### 6.15.3a Optional slots — `badge` + `showAddIcon` (Figma 2612:7791)

Two opt-in slots cover the design-system property panel toggles `Show Badge` and `Show Add`. Both are off by default — the existing filter / step-progress rows keep their original chrome.

| Prop | Type | Default | What it does |
|---|---|---|---|
| `badge` | `string \| { label, style?, hierarchy? } \| false` | `false` | Renders a leading `ColorfulBadge` before the label. Pass a string for label-only (uses defaults `style='indigo'`, `hierarchy='primary'`), or an object to override the badge style. Automatically suppresses the type's leading icon. |
| `badgeStyle` | ColorfulBadge `style` | `'indigo'` | Badge style when `badge` is a plain string. Maps to any `--surface-colorfulbadge-*` token (indigo, magenta, teal, blue, red, …). |
| `badgeHierarchy` | `'primary' \| 'secondary'` | `'primary'` | Forwarded to `ColorfulBadge` — `primary` = solid badge with white text (matches Figma 3394:1134 "indigo-bold"). |
| `showAddIcon` | `boolean` | `false` | Swaps the trailing chevron for an `add_circle` action glyph (20 px — Figma 3394:1154). Color is state-aware: **default** → `--icon-neutral-tertiary` (#ADB2B7), **hover** → `--icon-tabcapsule-hover`, **active** → `--icon-tabcapsule-selected`, **disabled** → `--icon-tabcapsule-disabled`. So the glyph reads as a neutral "add" affordance at rest and retints with the brand when the chip is interacted with. Trailing chevron and add-icon are mutually exclusive — passing `showAddIcon` implicitly turns off `showTrailingIcon`. |

**Canonical usage — Doctor-AI ICD suggestion chips** (Figma 156:27354 / 331:44658):

```jsx
<TabCapsule
  label='Dengue fever (ไข้แดงกี)'
  badge='A90'           // indigo-bold ColorfulBadge prefix
  showAddIcon            // trailing add_circle glyph
  showLeadingIcon={false}
  showTrailingIcon={false}
  onClick={() => addDxFromSuggestion(sug)}
/>
```

Pass an object to override the badge tone — e.g. `badge={{ label: 'G43.9', style: 'magenta', hierarchy: 'primary' }}` for diagnosis-class chips that differ from the default indigo.

---


### 6.16 Tag

> **Figma node:** `2056:4263` · Neutral variant · Brand variant · SM 32px · MD 36px · LG 40px

The Tag component is a **pill-shaped dropdown filter chip** used for filtering and faceted search. Clicking a tag opens a dropdown list of options below it. It ships in two style variants (Neutral / Brand) and three sizes, each with **five** interaction states (Default, Hover, Selected, Selected-Hover, Open).

---

#### Figma Properties

| Property | Default | Notes |
|---|---|---|
| `showLeadingIcon` | `true` | 16px link/chain icon (named `"pill"` in Figma) on the left; color follows `icon/*` token |
| `showTailingIcon` | `true` | 16px **Chevron Down** in non-open states (token `icon/tag/tailing` = `#c9cdd0`); **Chevron Up** in Open state (color follows text token) |
| `showTitle` | `false` | Brand + Large only; renders a 12px subtitle below the label |

> **Icon semantics:** Tag is a **dropdown chip**, not a removable chip. The trailing icon is a Chevron (↓ / ↑), never ✕. The `showTailingIcon` prop hides the Chevron Down in non-open states; Open state always shows Chevron Up regardless of this prop.

---

#### Icon Reference

All icons **must alias from the MIH Design System Foundation** library. Do not use arbitrary SVG paths.

| Role | Icon Name | Figma Node | Token | Value |
|---|---|---|---|---|
| Leading icon | **Link 2** | `377:11558` | `icon/tag/default` · `icon/brandTag/default` | `#4d5358` (neutral) · `#007549` (brand hover/selected) |
| Trailing — closed | **Chevron down** | `328:4566` | `icon/tag/tailing` · `icon/brandTag/tailing` | `#c9cdd0` (all states) |
| Trailing — open | **Chevron up** | `328:4637` | `text/neutralTag/hover` · `text/brandTag/hover` | `#4d5358` (neutral open) · `#007549` (brand open) |

**Link 2 SVG** (16 × 16 viewBox, `stroke="currentColor"`, `stroke-width="1.35"`, `stroke-linecap="round"`, `stroke-linejoin="round"`):
```
<!-- Link 2 · node 377:11558 · icon/tag/default=#4d5358 | icon/brandTag/default=#007549 -->
<path d="M6.5 9.5l-1 1A2.475 2.475 0 012 8a2.475 2.475 0 012.5-2.5l2-2a2.475 2.475 0 013.5 3.5"/>
<path d="M9.5 6.5l1-1A2.475 2.475 0 0114 8a2.475 2.475 0 01-2.5 2.5l-2 2A2.475 2.475 0 016 9"/>
```

**Chevron Down SVG** (16 × 16 viewBox, `stroke="currentColor"`, `stroke-width="1.4"`, `stroke-linecap="round"`, `stroke-linejoin="round"`):
```
<!-- Chevron down · node 328:4566 · icon/tag/tailing=#c9cdd0 -->
<path d="M4.5 6.5L8 10l3.5-3.5"/>
```

**Chevron Up SVG** (16 × 16 viewBox, same stroke attributes — color = text token of current state):
```
<!-- Chevron up · node 328:4637 -->
<path d="M4.5 9.5L8 6l3.5 3.5"/>
```

---

#### Alias Tokens — Neutral Variant

> ⚠️ **Figma typo in token name:** The Selected-Hover background token is literally named `surface/neutralTag/selectedy-hover` (extra "y") in the Figma file. Use this exact name when referencing the Figma Semantic collection.

| State | Token | Value | Role |
|---|---|---|---|
| **Default** | `surface/neutralTag/default` | `#ffffff` | Background |
| | `border/neutralTag/default` | `#e2e4e6` | Border |
| | `text/neutralTag/default` | `#4d5358` | Label text |
| | `icon/tag/default` | `#4d5358` | Leading icon |
| | `icon/tag/tailing` | `#c9cdd0` | Trailing Chevron Down |
| **Hover** | `surface/neutralTag/default-hover` | `#f9fbfb` | Background |
| | `border/neutralTag/default-hover` | `#e2e4e6` | Border |
| | `text/neutralTag/hover` | `#4d5358` | Label text |
| | `icon/tag/tailing` | `#c9cdd0` | Trailing Chevron Down |
| **Selected** | `surface/neutralTag/selected` | `#f4f5f5` | Background |
| | `border/neutralTag/selected` | `#e2e4e6` | Border |
| | `text/neutralTag/selected` | `#4d5358` | Label text |
| | `icon/tag/tailing` | `#c9cdd0` | Trailing Chevron Down |
| **Selected-Hover** | `surface/neutralTag/selectedy-hover` ⚠️ | `#e2e4e6` | Background (Figma typo in name) |
| | `border/neutralTag/quaternary` | `#e2e4e6` | Border |
| | `text/neutralTag/selected-hover` | `#4d5358` | Label text |
| | `icon/tag/tailing` | `#c9cdd0` | Trailing Chevron Down |
| **Open** | `surface/neutralTag/default-hover` | `#f9fbfb` | Chip background (same as Hover) |
| | `border/neutralTag/default-hover` | `#e2e4e6` | Chip border |
| | `text/neutralTag/hover` | `#4d5358` | Label text + Chevron Up color |
| | `surface/dropdown/default` | `#ffffff` | Dropdown panel background |
| | `border/dropdown/default` | `#ebebeb` | Dropdown panel border |
| | `icon/list/secondary-default` | `#858c92` | List item icon (User icon) |
| | `text/list/default` | `#4d5358` | List item text |

---

#### Alias Tokens — Brand Variant

> 🎨 **Theme-aware:** All brand tokens alias `primary/*` which maps to the active brand primitive (`--em50` … `--em700`). Switching brand theme via `setBrandTheme()` cascades through all brand tag states automatically.

| State | Token | Alias Chain | Default (Emerald) | CSS Variable | Role |
|---|---|---|---|---|---|
| **Default** | `surface/brandTag/default` | `color/grey/50` (fixed) | `#ffffff` | `--surface-brandtag-default` | Background |
| | `border/brandTag/default` | `color/neutral/200` (fixed) | `#e2e4e6` | `--border-brandtag-default` | Border |
| | `text/brandTag/default` | `color/neutral/700` (fixed) | `#4d5358` | `--text-brandtag-default` | Label text |
| | `icon/brandTag/default` | `primary/700` → `--em700` | `#007549` | `--icon-brandtag-default` | Leading icon |
| | `icon/tag/tailing` | `color/neutral/300` (fixed) | `#c9cdd0` | — | Trailing Chevron Down |
| **Hover** | `surface/brandTag/default-hover` | `primary/50` → `--em50` | `#f4fbf8` | `--surface-brandtag-hover` | Background |
| | `border/brandTag/default-hover` | `primary/200` → `--em200` | `#c5edda` | `--border-brandtag-hover` | Border |
| | `text/brandTag/hover` | `primary/700` → `--em700` | `#007549` | `--text-brandtag-hover` | Label text |
| | `icon/brandTag/hover` | `primary/700` → `--em700` | `#007549` | `--icon-brandtag-hover` | Leading icon |
| | `icon/tag/tailing` | `color/neutral/300` (fixed) | `#c9cdd0` | — | Trailing Chevron Down |
| **Selected** | `surface/brandTag/selected` | `primary/50` → `--em50` | `#f4fbf8` | `--surface-brandtag-selected` | Background |
| | `border/brandTag/selected` | `primary/200` → `--em200` | `#c5edda` | `--border-brandtag-selected` | Border |
| | `text/brandTag/selected` | `primary/700` → `--em700` | `#007549` | `--text-brandtag-selected` | Label text |
| | `icon/brandTag/selected` | `primary/700` → `--em700` | `#007549` | `--icon-brandtag-selected` | Leading icon |
| | `icon/tag/tailing` | `color/neutral/300` (fixed) | `#c9cdd0` | — | Trailing Chevron Down |
| **Selected-Hover** | `surface/brandTag/selected-hover` | `primary/100` → `--em100` | `#e2f3eb` | `--surface-brandtag-selected-hover` | Background |
| | `border/brandTag/quaternary` | `primary/200` → `--em200` | `#c5edda` | `--border-brandtag-selected-hover` | Border |
| | `text/brandTag/selected-hover` | `primary/700` → `--em700` | `#007549` | `--text-brandtag-selected-hover` | Label text |
| | `icon/brandTag/selected-hover` | `primary/700` → `--em700` | `#007549` | `--icon-brandtag-selected-hover` | Leading icon |
| | `icon/tag/tailing` | `color/neutral/300` (fixed) | `#c9cdd0` | — | Trailing Chevron Down |
| **Open** | `surface/brandTag/default-hover` | `primary/50` → `--em50` | `#f4fbf8` | `--surface-brandtag-hover` | Chip background (same as Hover) |
| | `border/brandTag/default-hover` | `primary/200` → `--em200` | `#c5edda` | `--border-brandtag-hover` | Chip border |
| | `text/brandTag/hover` | `primary/700` → `--em700` | `#007549` | `--text-brandtag-hover` | Label text + Chevron Up color |
| | `surface/dropdown/default` | fixed | `#ffffff` | — | Dropdown panel background |
| | `border/dropdown/default` | fixed | `#ebebeb` | — | Dropdown panel border |
| | `icon/list/secondary-default` | fixed | `#858c92` | — | List item icon |
| | `text/list/default` | fixed | `#4d5358` | — | List item text |
| **Brand LG title** | `text/brandTag/title` | `color/neutral/500` (fixed) | `#858c92` | — | Subtitle — 12px, `showTitle` only |

---

#### Dropdown Panel Tokens (Open State)

| Token | Value | Role |
|---|---|---|
| `surface/dropdown/default` | `#ffffff` | Panel background |
| `border/dropdown/default` | `#ebebeb` | Panel border (1px) |
| `dimension/radius/500` | **16 px** | Panel border radius |
| `Drop Shadow Bottom/400` | `rgba(100,116,139,.15)` @ 32px, `rgba(100,116,139,.10)` @ 4px | Elevation shadow |
| `icon/list/secondary-default` | `#858c92` | Icon per list row |
| `text/list/default` | `#4d5358` | Text per list row |
| Min-height per row | — | **48 px** |
| Row padding | `dimension/space/300` / `dimension/space/200` | 12px horizontal / 8px vertical |

---

#### States Summary

> 🎨 **Theme note:** Brand column values shown for Emerald (default). When brand theme changes, all brand values update automatically via `primary/*` → `--em*` alias cascade.

| State | Neutral bg / border / text | Brand bg / border / text | Brand alias chain | Trailing icon |
|---|---|---|---|---|
| Default | `#ffffff` / `#e2e4e6` / `#4d5358` | `#ffffff` / `#e2e4e6` / `#4d5358` | fixed (grey/neutral) | ↓ Chevron (`#c9cdd0`) |
| Hover | `#f9fbfb` / `#e2e4e6` / `#4d5358` | `var(--em50)` / `var(--em200)` / `var(--em700)` | `primary/50` · `primary/200` · `primary/700` | ↓ Chevron (`#c9cdd0`) |
| Selected | `#f4f5f5` / `#e2e4e6` / `#4d5358` | `var(--em50)` / `var(--em200)` / `var(--em700)` | `primary/50` · `primary/200` · `primary/700` | ↓ Chevron (`#c9cdd0`) |
| Selected-Hover | `#e2e4e6` / `#e2e4e6` / `#4d5358` | `var(--em100)` / `var(--em200)` / `var(--em700)` | `primary/100` · `primary/200` · `primary/700` | ↓ Chevron (`#c9cdd0`) |
| **Open** | `#f9fbfb` / `#e2e4e6` / `#4d5358` | `var(--em50)` / `var(--em200)` / `var(--em700)` | same as Hover | ↑ Chevron (text color) + dropdown panel |

---

#### Dimension Tokens

| Role | Dimension Token | Value |
|---|---|---|
| Height — Small | `dimension/size/600` | **32 px** |
| Height — Medium | `dimension/size/700` | **36 px** |
| Height — Large | `dimension/size/800` | **40 px** |
| Border radius | — | **999 px** (pill shape) |
| Border width | `dimension/stroke/100` | **1 px** |
| Gap (icon ↔ label) | `dimension/space/150` | **6 px** |
| Padding X | `dimension/space/300` | **12 px** |
| Padding Y | `dimension/space/100` | **4 px** |
| Icon size | `dimension/size/300` | **16 px** |
| Max-width | `dimension/size/2200` | **192 px** |

---

#### Typography

| Prop | Value |
|---|---|
| Font size (SM) | 13px |
| Font size (MD / LG) | 14px |
| Font weight | 500 (medium) |
| Title (Brand LG only) | 11–12px, weight 400, color `text/brandTag/title` = `#858c92` |

---

#### Interactive Behavior

- Clicking a Tag chip **opens / closes a dropdown panel** below it (Default/Hover/Selected → Open → back)
- **Trailing icon** is always a Chevron: ↓ (Chevron Down, `#c9cdd0`) in closed states; ↑ (Chevron Up, text-color) in Open state
- `showTailingIcon = false` hides Chevron Down in non-open states; Chevron Up in Open state is always visible
- **Leading icon** is the Design System "pill" (link/chain) icon — color follows `icon/tag/default` (neutral) or `icon/brandTag/default` (brand, `#007549`)
- **Open state tokens** reuse `default-hover` chip tokens, plus a separate `surface/dropdown/default` panel namespace
- `showTitle` only applies to **Brand + Large** — adds a subtitle line below the label text
- Brand tags show brand-colored icon and text on hover/selected/open — value is `var(--em700)` (Emerald default `#007549`), updates automatically when brand theme changes

#### 6.16.1 FilterDropdown (`src/components/forms/FilterTag.jsx`)

`FilterDropdown` composes **`FilterTag`** (trigger chip) + portalled **`Popover`** + **`ActionList`** rows. It implements the Tag **Open** interaction from §6.16: click toggles the panel; pick a row to set `value` and close.

| Prop | Default | Notes |
|---|---|---|
| `options` | — | `{ value, label }[]` |
| `value` / `onChange` | — | Controlled selection |
| `prefix` | — | Optional gray title before label (filter toolbars) |
| `leadingIcon` | — | Material icon name on chip (e.g. `language` in VoiceTextarea) |
| `menuZIndex` | `--sm-z-popover-nested` | Use `--sm-z-modal-popover` (1450) when the chip lives inside a modal or `MarkNoteEditor` |
| `tagStyle` | — | Override chip dimensions (VoiceTextarea: **36px** height, max-width **192px**) |

**Panel styling** aliases §6.14 / Open-state dropdown tokens: `surface/dropdown/default`, `border/dropdown/default`, `dim-radius-500`, `shadow-dropdown`, `mih-search-popover` entrance class.

**VoiceTextarea language picker** (Figma `3280:3807`, toolbar): `leadingIcon="language"`, no `prefix`, label shows the active locale. Options:

| Label | `value` (BCP-47) |
|---|---|
| ภาษาไทย | `th-TH` |
| ภาษาอังกฤษ | `en-US` |

Export: `VOICE_TEXTAREA_LANGUAGE_OPTIONS` from `VoiceTextarea.jsx`. Pass `menuZIndex="var(--sm-z-modal-popover)"` so the list stacks above `Popup` / body-mark editor (`z-index` 1450).

**Nested dismiss rules.** Portalled menus use class `mih-search-popover`. Clicks inside that panel must **not** close parent overlays (filter overflow §6.15.1, `Popup` backdrop handler, body-mark `pointerdown` dismiss). Body drawing excludes `.mih-search-popover` alongside `[data-mark-note-editor]` and `[data-drawing-tool-sidebar]`.


### 6.17 Search

> **Figma:** [MIH Design System Foundation — Primary Search](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2056-4246)
> Primary Search node: `2056:4246` · Secondary Search node: `305:43`

The Search component ships in **two variants**:

| Variant | Description | Height | Radius | Figma Node |
|---------|-------------|--------|--------|------------|
| **Primary Search** | Full-width pill with brand border. **5 states** — Default · Hover · Focus · Typing · Filled. Typing adds X + submit controls. | `56px` | `9999px` | `2056:4246` |
| **Secondary Search** | Compact inline input with neutral border, focus ring, clear button | `40px` / `36px` (SM) | `8px` | `305:43` |

---

#### Token Alias Chain

All tokens follow the standard 2-layer alias chain:

```
Semantic token                      →  Primitive / Brand     →  Resolved value
────────────────────────────────────────────────────────────────────────────────
border/primarySearch/default        →  primary/600           →  var(--border-primarySearch-default)  #08A768
border/primarySearch/hover          →  primary/700           →  var(--border-primarySearch-hover)    #007549
border/primarySearch/typing         →  primary/700           →  var(--border-primarySearch-typing)   #007549
border/primarySearch/filled         →  primary/600           →  var(--border-primarySearch-filled)   #08A768

border/secondarySearch/default      →  neutral/300           →  #C9CDD0  (fixed)
border/secondarySearch/hover        →  primary/600           →  var(--em600)  #08A768  ← not 700
border/secondarySearch/typing       →  primary/600           →  var(--em600)  #08A768  ← not 700
border/secondarySearch/filled       →  neutral/300           →  #C9CDD0  (fixed)

text/primarySearch/default          →  neutral/400           →  #ADB2B7  (placeholder)
text/primarySearch/hover            →  neutral/700           →  #4D5358  (placeholder, hover)
text/primarySearch/typing           →  neutral/700           →  #4D5358  (active typed text)
text/primarySearch/filled           →  neutral/700           →  #4D5358  (value text)

icon/primarySearch/default          →  primary/600           →  #08A768
icon/primarySearch/hover            →  primary/700           →  #007549
icon/secondarySearch/default        →  neutral/400           →  #ADB2B7
icon/secondarySearch/typing         →  neutral/700           →  #4D5358

surface/search/default              →  grey/50               →  #FFFFFF  (shared)
surface/brandPrimaryButton/default  →  emerald/700           →  #007549  (typing button)
PrimaryShadow/600                   →  brand primitive        →  rgba(8,167,104,0.08)
PrimaryShadow/601                   →  brand primitive        →  rgba(8,167,104,0.10)
primaryBorder/600                   →  brand primitive        →  rgba(8,167,104,0.15)  (focus ring)
```

---

#### 6.17.1 Primary Search — Color Tokens

> **Property panel — Figma 2056:4246:** the canonical `State` enum exposes **five** values: `Default` · `Hover` · `Focus` · `Typing` · `Filled`. `Focus` and `Typing` both have the input focused but split on whether a value is present. Only Typing adds the XButton + Brand Circle Icon Button. The code component preserves the existing recent/suggestion dropdown API below the Figma shell for production queue lookup.

| Property | Semantic Token | Primitive / Brand | CSS Variable | Brand-responsive |
|----------|---------------|-------------------|-------------|-----------------|
| Background | `surface/search/default` | `grey/50` | `#FFFFFF` | — |
| Border — Default | `border/primarySearch/default` | `primary/600` | `var(--border-primarySearch-default)` | ✓ |
| Border — Hover | `border/primarySearch/hover` | `primary/700` | `var(--border-primarySearch-hover)` | ✓ |
| Border — Focus | `border/primarySearch/focus` (aliases `default`) | `primary/600` | `var(--border-primarySearch-focus)` | ✓ |
| Border — Typing | `border/primarySearch/typing` | `primary/700` | `var(--border-primarySearch-typing)` | ✓ |
| Border — Filled | `border/primarySearch/filled` | `primary/600` | `var(--border-primarySearch-filled)` | ✓ |
| Shadow layer 1 | `PrimaryShadow/600` | `primary/600` @ 8% | `var(--PrimaryShadow-600)` | ✓ |
| Shadow layer 2 | `PrimaryShadow/601` | `primary/600` @ 10% | `var(--PrimaryShadow-601)` | ✓ |

**Text tokens (per state):**

| State | Layer | Semantic Token | Primitive | Value |
|-------|-------|---------------|-----------|-------|
| Default | Placeholder hint | `text/primarySearch/default` | `neutral/400` | `#ADB2B7` |
| Hover | Placeholder hint | `text/primarySearch/hover` | `neutral/700` | `#4D5358` |
| Focus | Placeholder hint | `text/primarySearch/focus` (aliases `default`) | `neutral/400` | `#ADB2B7` |
| Typing | Active typed text | `text/primarySearch/typing` | `neutral/700` | `#4D5358` |
| Filled | Value text | `text/primarySearch/filled` | `neutral/700` | `#4D5358` |

> **Note — Placeholder alias (node `2056:4105`):** `text/primarySearch/hover` is applied directly to the placeholder text layer in Hover state; `text/primarySearch/focus` keeps the placeholder grey because the input is empty until the user actually starts typing. `text/primarySearch/typing` and `text/primarySearch/filled` apply to the active input and value layers respectively.

**Right cluster (per state):**

| State | XButton (40 px) | SubmitButton (40 px) | Dropdown |
|---|---|---|---|
| Default | — | — | — |
| Hover | — | — | — |
| Focus | — | — | `Dropdown/Recent` (history rows) |
| Typing | ✓ (brand-quaternary chip) | ✓ (brand-primary circle, `arrow_upward`) | `Dropdown/Recent` (suggestion rows) |
| Filled | — | — | — |

> The XButton appears **only** in `Typing` — it wipes the value back to empty, collapsing the state to `Focus` and restoring the recent dropdown.

**Icon tokens:**

| State | Semantic Token | Primitive | Value |
|-------|---------------|-----------|-------|
| Default | `icon/primarySearch/default` | `primary/600` = `emerald/600` | `#08A768` |
| Hover | `icon/primarySearch/hover` | `primary/700` = `emerald/700` | `#007549` |
| Typing | `icon/primarySearch/typing` | `primary/700` = `emerald/700` | `#007549` |
| Filled | `icon/primarySearch/filled` | `primary/600` = `emerald/600` | `#08A768` |

---

#### 6.17.2 Primary Search — Dimension Tokens

| Property | Semantic Token | Resolved Value |
|----------|---------------|----------------|
| Height | `dimension/size/1200` | `56px` |
| Border-width (all states) | `dimension/stroke/200` | `2px` |
| Border-radius | `dimension/radius/full` | `9999px` |
| Padding X | `dimension/size/500` | `24px` |
| Padding Y | `dimension/space/400` | `16px` |
| Gap | `dimension/space/600` | `24px` |
| Icon size | `dimension/size/400` | `20px` |
| Min-width — Default / Typing | `dimension/size/2500` | `320px` |
| Min-width — Hover / Focus / Filled | `dimension/size/2100` | `160px` |
| Max-width | `dimension/size/2900` | `640px` |

---

#### 6.17.3 Primary Search — Shadow Tokens

**`Brand Drop Shadow Bottom/200`** — applied to the search bar in all states:

```
layer 1: DROP_SHADOW
         color  = PrimaryShadow/600  →  rgba(8,167,104,0.08)
         offset = (0, 1px) · radius = 4px · spread = 0

layer 2: DROP_SHADOW
         color  = PrimaryShadow/601  →  rgba(8,167,104,0.10)
         offset = (0, 1px) · radius = 8px · spread = 0
```

CSS: `box-shadow: 0 1px 4px rgba(8,167,104,.08), 0 1px 8px rgba(8,167,104,.10);`

---

#### 6.17.4 Primary Search — Typing State

**Search button (Brand Circle Icon Button):**

| Property | Semantic Token | Primitive | Value |
|----------|---------------|-----------|-------|
| Background | `surface/brandPrimaryButton/default` | `emerald/700` | `#007549` |
| Icon color | `icon/brandPrimary/on-brand` | `grey/50` | `#FFFFFF` |
| Size | `dimension/size/800` | — | `40px × 40px` |
| Border-radius | `dimension/radius/full` | — | `9999px` |

**Dropdown panel:**

| Property | Semantic Token | Primitive | Value |
|----------|---------------|-----------|-------|
| Background | `surface/dropdown/default` | `grey/50` | `#FFFFFF` |
| Border color | `border/dropdown/default` | `grey/600` | `#EBEBEB` |
| Border-width | `dimension/stroke/100` | — | `1px` |
| Border-radius | `dimension/radius/500` | — | `16px` |
| Padding | `dimension/space/100` | — | `4px` |

Dropdown shadow (`Drop Shadow Bottom/400`):
```
layer 1: color = color/shadow/050  →  rgba(100,116,139,0.10)
         offset = (0, 4px) · radius = 4px · spread = -4px

layer 2: color = color/shadow/100  →  rgba(100,116,139,0.15)
         offset = (0, 16px) · radius = 32px · spread = -4px
```

**Section header** (Figma node I2692:5995;3345:2219 — "Title" row of `Dropdown/Recent`):

| Property | Semantic Token | CSS Variable | Value |
|---|---|---|---|
| Title color | `text/content/tertiary` | `--text-content-tertiary` | `#858C92` |
| Title font | `body4` | — | Sarabun Regular 14 / 1.5 (no uppercase / no letter-spacing) |
| Padding — top | `dimension/space/400` | — | `16px` |
| Padding — bottom | `dimension/space/200` | — | `8px` |
| Padding — left / right | `dimension/space/400` | — | `16px` |
| Gap (title ↔ trailing) | — | — | `10px` |
| Trailing slot | — | — | `<NeutralSubdueButton variant='outline' size='sm'>` (e.g. "ล้างประวัติ") |

**List items inside dropdown** (Figma node I2692:5995;3345:2068 — list rows of `Dropdown/Recent`):

| Property | Semantic Token | CSS Variable | Value | Brand-responsive |
|----------|---------------|-------------|-------|-----------------|
| Background (default) | `surface/list/default` | `--surface-list-default` | `#FFFFFF` | — |
| Background (hover) | `surface/list/hover` | `--surface-list-hover` | `var(--em50)` | ✓ |
| Label color (default) | `text/list/default` | `--text-list-default` | `#4D5358` | — |
| Label color (hover) | `text/list/hover` | `--text-list-hover` | `var(--em700)` | ✓ |
| Sublabel color (default) | `text/content/tertiary` | `--text-content-tertiary` | `#858C92` | — |
| Sublabel color (hover) | `text/list/hover` | `--text-list-hover` | `var(--em700)` | ✓ |
| Leading icon (default, optional) | `text/content/tertiary` | `--text-content-tertiary` | `#858C92` | — |
| Leading icon (hover) | `text/list/hover` | `--text-list-hover` | `var(--em700)` | ✓ |
| Label font | `body4` | — | Sarabun Regular `14px` / 1.5 | — |
| Sublabel font size | `typograpphy/size/xs` | — | `12px` | — |
| Min-height | `dimension/size/1000` | — | `48px` | — |
| Padding X | `dimension/space/300` | — | `12px` | — |
| Padding Y | `dimension/space/200` | — | `8px` | — |
| Gap (icon ↔ label) | `dimension/space/200` | — | `8px` | — |
| Border-radius | `dimension/radius/100` | — | `4px` | — |
| Transition | — | — | `colors 150ms ease-out` | — |

> **Leading icon is optional** per Figma 2692:5995 — every row in the reference renders text-only. The current implementation renders the icon **only when `item.icon` is supplied** by the caller, so the default `<PrimarySearch>` dropdown matches the Figma spec. Callers that want a consistent icon on every row (e.g. a clock for history) can pass `leadingIcon` to `<SuggestionRow>` directly.

> **Implementation note (hover specificity):** Tailwind v2.1 + JIT cannot reliably apply arbitrary-value classes like `hover:bg-[color:var(--surface-list-hover)]` through inline-styled rows. The component uses a dedicated CSS class `.mih-search-row` in `src/index.css` whose `:hover` selector and `:hover .mih-search-row__icon`/`__sublabel` descendants drive every tone change. This guarantees the hover state works regardless of which framework version is in play and bypasses the inline-style → pseudo-class specificity trap.

##### Rich row variant — Figma 3345:2981 (Recent) / 3346:8231 (Typing)

The same `<SuggestionRow>` supports a richer layout used in the **recent-search** and **typing** dropdowns of PrimarySearch. Leading content becomes a stack of colorful pills (queue + HN) instead of a generic icon, and a trailing chevron-right fades in on hover to telegraph the row's clickability.

| Slot | Source | Token / Spec |
|---|---|---|
| Leading pills | `item.badges: Array<{ label, tone }>` | Aliases `<ColorfulBadge style={tone} size='small' fill='default' hierarchy='secondary'>` (Figma node 2076:8413). Per Figma 3345:2981 queue = `teal`, HN = `indigo`. Resolves to h 24 · px 8 · py 4 · radius 9999 · `surface/colorfulBadge/{tone}` + `text/colorfulBadge/{tone}`. |
| Label | `item.label` | `body4` — `text/list/default` → `text/list/hover` |
| Sublabel | `item.sublabel` | `caption1` — `text/list/subtext` (`#858C92`) → `text/list/subtext-hover` (`primary/500`, brand-aware) |
| Trailing chevron | always rendered, fades in on `:hover` only | `chevron_right` 16 px · `icon/list/secondary-default` (`#858C92`) → `icon/list/secondary-hover` (`primary/700`, brand-aware) |

**Precedence:** If `item.badges` is set the row renders the pill stack; otherwise it falls back to the `item.icon` slot (icon-only or text-only per Figma 2692:5995). All three layouts share the same surface/text/chevron hover tokens.

**Chevron behaviour:** The chevron sits in the DOM at all times so the row's layout doesn't shift when the user hovers. `.mih-search-row__chevron` starts at `opacity: 0` and fades to `1` on `.mih-search-row:hover` (CSS only — no JS state). Resting rows look like the recent-search example with no trailing affordance; hovered rows reveal the navigation hint.

---

#### 6.17.5 Secondary Search — Color Tokens

| Property | Semantic Token | Primitive / Brand | Light Mode |
|----------|---------------|-------------------|------------|
| Background | `surface/search/default` | `grey/50` | `#FFFFFF` |
| Border — Default | `border/secondarySearch/default` | `neutral/300` | `#C9CDD0` |
| Border — Hover | `border/secondarySearch/hover` | `primary/600` | `var(--em600)` |
| Border — Typing | `border/secondarySearch/typing` | `primary/600` | `var(--em600)` |
| Border — Filled | `border/secondarySearch/filled` | `neutral/300` | `#C9CDD0` |
| Focus ring (Hover/Typing) | `primaryBorder/600` | brand primitive | `rgba(8,167,104,0.15)` |

**Focus ring detail — effect token `input/100`:**
```
DROP_SHADOW · color: primaryBorder/600  →  rgba(8,167,104,0.15)
             offset: (0, 0) · radius: 0 · spread: 4px
```
CSS: `box-shadow: 0 0 0 4px rgba(8,167,104,0.15);`

**Text tokens:**

| State | Semantic Token | Primitive | Value |
|-------|---------------|-----------|-------|
| Default | `text/secondarySearch/default` | `neutral/400` | `#ADB2B7` |
| Hover | `text/secondarySearch/hover` | `neutral/700` | `#4D5358` |
| Typing | `text/secondarySearch/typing` | `neutral/700` | `#4D5358` |
| Filled | `text/secondarySearch/filled` | `neutral/700` | `#4D5358` |

**Icon tokens:**

| State | Semantic Token | Primitive | Value |
|-------|---------------|-----------|-------|
| Default | `icon/secondarySearch/default` | `neutral/400` | `#ADB2B7` |
| Typing | `icon/secondarySearch/typing` | `neutral/700` | `#4D5358` |
| Icon size | *(fixed)* | — | `16px` |

---

#### 6.17.6 Secondary Search — Dimension Tokens

| Property | Semantic Token | Resolved Value |
|----------|---------------|----------------|
| Height — Default size | `dimension/size/800` | `40px` |
| Height — Small size | `dimension/size/700` | `36px` |
| Border-width — Default / Filled | `dimension/stroke/100` | `1px` |
| Border-width — Hover / Typing | `dimension/stroke/200` | `2px` |
| Border-radius | `dimension/radius/200` | `8px` |
| Padding | `dimension/space/300` | `12px` |
| Gap | `dimension/space/200` | `8px` |
| Min-width | `dimension/size/2300` | `224px` |
| Max-width | `dimension/size/2700` | `448px` |

> **Border-width toggle:** Secondary Search switches between `1px` (Default/Filled) and `2px` (Hover/Typing) using `dimension/stroke/100` and `dimension/stroke/200` respectively. This is the only component where border-width changes per state.

##### Clear (X) Button — Figma 3345:2713 / 3345:2721

> Trailing "close" affordance that wipes the search value when the field is `filled`. Implemented inline in [`SecondarySearch.jsx`](src/components/search/SecondarySearch.jsx) as `<XButton>` so the morph-on-hover (square → circle) stays scoped to the search component.

| Property | Semantic Token | CSS Variable | Value | Brand-responsive |
|---|---|---|---|---|
| Width × Height | `dimension/size/600` | `--dim-size-600` | `32 × 32 px` | — |
| Background — Default | `surface/brandprimary/quaternary` | `--surface-brandPrimary-quaternary` | `var(--brand-p50)` `#F4FBF8` | ✓ |
| Background — Hover | `surface/brandprimary/quaternary-hover` | `--surface-brandPrimary-quaternary-hover` | `var(--brand-p100)` `#E2F3EB` | ✓ |
| Border-radius — Default | `dimension/radius/600` | `--dim-radius-600` | `24 px` | — |
| Border-radius — Hover | `dimension/radius/full` | `--dim-radius-full` | `9999 px` (perfect circle) | — |
| Icon | Material Symbols `close` | — | `16 px` | — |
| Icon color | `text/brandPrimary/default` | `--text-brandPrimary-default` | `var(--brand-p700)` `#007549` | ✓ |
| Transition | — | — | `background-color 150 / border-radius 200 / color 150 ms ease-out` | — |

> **Why the radius morphs:** the Figma `X Button · info / Hover` variant swaps to a fully-circular shape — visually this signals "press me to clear" by lifting the square chip off the input. The transition stack above keeps the round-out feeling tactile rather than jumpy.

---

#### 6.17.6b AI Search — Color Tokens

> The Doctor-AI panel (Figma 156:27359 / Variables panel `border/aiSearch` + `icon/aiSearch`) introduces an **indigo-themed** search variant that opts out of the brand-aware primary/secondary palettes. The four states reuse the indigo color primitives in light mode and flip to lighter indigo stops in dark mode so contrast holds on dark surfaces.

| Property | Semantic Token | Light primitive | Dark primitive | CSS variable |
|----------|---------------|-----------------|----------------|--------------|
| Border — Default | `border/aiSearch/default` | `indigo/500` `#6F82FF` | `indigo/500` `#6F82FF` | [`--border-aiSearch-default`](src/index.css) |
| Border — Hover | `border/aiSearch/hover` | `indigo/700` `#4338CA` | `indigo/300` `#A5B4FC` | [`--border-aiSearch-hover`](src/index.css) |
| Border — Typing | `border/aiSearch/typing` | `indigo/700` `#4338CA` | `indigo/300` `#A5B4FC` | [`--border-aiSearch-typing`](src/index.css) |
| Border — Filled | `border/aiSearch/filled` | `indigo/600` `#4F46E5` | `indigo/400` `#818CF8` | [`--border-aiSearch-filled`](src/index.css) |
| Icon — Default | `icon/aiSearch/default` | `indigo/600` `#4F46E5` | `indigo/400` `#818CF8` | [`--icon-aiSearch-default`](src/index.css) |
| Icon — Hover | `icon/aiSearch/hover` | `indigo/700` `#4338CA` | `indigo/300` `#A5B4FC` | [`--icon-aiSearch-hover`](src/index.css) |
| Icon — Typing | `icon/aiSearch/typing` | `indigo/700` `#4338CA` | `indigo/300` `#A5B4FC` | [`--icon-aiSearch-typing`](src/index.css) |
| Icon — Filled | `icon/aiSearch/filled` | `indigo/600` `#4F46E5` | `indigo/400` `#818CF8` | [`--icon-aiSearch-filled`](src/index.css) |

**Underlying indigo primitives** (`Primitive/color/indigo/*`):

| CSS variable | Hex | Used by |
|---|---|---|
| [`--color-indigo-300`](src/index.css) | `#A5B4FC` | Dark-mode hover/typing |
| [`--color-indigo-400`](src/index.css) | `#818CF8` | Dark-mode default/filled · `--surface-colorfulbadge-indigo-bold` · indigo shadow primitives |
| [`--color-indigo-600`](src/index.css) | `#4F46E5` | Light-mode default/filled · `--text-colorfulbadge-indigo` |
| [`--color-indigo-700`](src/index.css) | `#4338CA` | Light-mode hover/typing |

**Dark-mode trigger.** The dark values activate on either:
1. `<html data-theme="dark">` — explicit override (palette switcher, "dark" toggle), or
2. `@media (prefers-color-scheme: dark)` when `[data-theme]` is unset or explicitly `light`

The override block lives in [src/index.css](src/index.css) right after the Brand Palette `:root` close, so it cascades over every brand mode. Other tokens stay in their light-mode default — dark-mode coverage will roll out incrementally as Figma publishes additional dark variables.

---

#### 6.17.6c AI Search — Component spec (Figma 3396:1598)

> **Component:** [`AISearch`](src/components/search/AISearch.jsx) · **Demo:** [/components/ai-search](src/pages/components/ComponentsLibraryPage.jsx) · **Production use:** Step 4 Doctor-AI ICD search (Figma PHCIS · Medical-examination 156:27359).

The AI Search shares PrimarySearch's 64 px Large shell + 5-state matrix but swaps every brand-green hook for the indigo `aiSearch` semantic group. Visually it's the canonical "AI assist" surface in the system — the indigo border + violet drop shadow + linear-gradient submit button add up to one recognisable lockup.

**Dimension parity with PrimarySearch.** Outer height, padding, radius, dropdown chrome, list-row hover tokens, history persistence, and the 5-state auto-resolver (`focused + empty → focus`, `focused + value → typing`, `blurred + value → filled`) are **identical**. Only the indigo accent + gradient submit differ. The component is therefore safe to swap-in wherever PrimarySearch already lives — the API surface is the same.

**Visual signature (vs PrimarySearch):**

| Element | PrimarySearch | AISearch |
|---|---|---|
| Border tokens | `border/primarySearch/*` (emerald) | `border/aiSearch/*` (indigo) |
| Icon tokens | `icon/primarySearch/*` (emerald) | `icon/aiSearch/*` (indigo) |
| Drop shadow | `Brand Drop Shadow Bottom/200` (brand-aware emerald) | **Indigo Drop Shadow Bottom/200** `--shadow-indigo-drop-bottom-200` (two-layer: indigo-050 `0 1px 4px` + indigo-100 `0 1px 8px`). The legacy single-layer alias `--shadow-indigo-drop-bottom` now points at /200 so older callers auto-upgrade. |
| Submit button | Solid brand-green 36 px circle | **Linear-gradient `#6F82FF → #C079EA`** 36 px circle |
| Clear (X) | None | 32 px circle, `--color-indigo-100` (#EEF2FF) bg |
| Dropdown header | `ค้นหาล่าสุด` + page-specific filter | `ค้นหาล่าสุด` + optional `ล้างประวัติ` Neutral Subdue Button |
| Right cluster (default/hover/filled) | Avatar pile + "ค้นหาล่าสุด" caption | Plain — no avatars |

**Indigo Drop Shadow Bottom/200 — token spec (Figma "Indigo Drop Shadow Bottom/200"):**

Two-layer drop stacked the same way as Brand Drop Shadow Bottom/200, but with the indigo shadow primitives so the violet glow stays consistent across every brand mode.

| Layer | Offset | Blur | Spread | Color | Token |
|---|---|---|---|---|---|
| Inner | X 0 · Y 1 | 4 | 0 | `#818CF8 @ 10%` | [`--color-shadow-indigo-050`](src/index.css) |
| Outer | X 0 · Y 1 | 8 | 0 | `#818CF8 @ 15%` | [`--color-shadow-indigo-100`](src/index.css) |

| CSS variable | Composite |
|---|---|
| `--shadow-indigo-drop-bottom-200` | `0 1px 4px 0 var(--color-shadow-indigo-050), 0 1px 8px 0 var(--color-shadow-indigo-100)` |
| `--filter-indigo-drop-bottom-200` | `drop-shadow(0 1px 4px var(--color-shadow-indigo-050)) drop-shadow(0 1px 8px var(--color-shadow-indigo-100))` |
| `--shadow-indigo-drop-bottom` *(legacy alias)* | → `var(--shadow-indigo-drop-bottom-200)` — auto-upgrades callers that landed before the /200 token was published |
| `--filter-indigo-drop-bottom` *(legacy alias)* | → `var(--filter-indigo-drop-bottom-200)` |

**Doctor AI panel container (Figma 156:27359):** outer 2 px border + indigo-tinted surface around the AISearch shell. Aliased via:

| Token | CSS variable | Value | Used by |
|---|---|---|---|
| `surface/card/indigo-100` | [`--surface-card-indigo-100`](src/index.css) | `#F3F5FC` | DoctorAiPanel container fill |
| `border/doctorAI/panel` | [`--border-doctor-ai-panel`](src/index.css) → `--color-indigo-500` | `#6F82FF` | DoctorAiPanel 2 px border |
| `Primitive/color/indigo/500` | [`--color-indigo-500`](src/index.css) | `#6F82FF` | base primitive (Tailwind indigo-500) |

**Component props (canonical):**

| Prop | Type | Default | Notes |
|---|---|---|---|
| `value` / `onChange` | `string` / `(next) => void` | required | DOM-event-shaped — `(next)` is the new value, not the event. |
| `onSubmit` | `(value, pickedItem?) => void` | — | Fires on Enter, on gradient-submit click, and on history-row click. |
| `onClear` | `() => void` | — | Fires when the X chip is clicked or `Escape` is pressed. |
| `onClearHistory` | `() => void` | — | Optional — shows the "ล้างประวัติ" outline button in the focus-state header. |
| `placeholder` | `string` | `'ค้นหารหัสโรค เช่น ไข้เลือดออก'` | — |
| `history` | `Array<string \| { id?, label, value?, badge?, badgeStyle? }>` | `[]` | Rich rows render the ColorfulBadge prefix. |
| `historyTitle` | `string` | `'ค้นหาล่าสุด'` | Header copy in the focus-state dropdown. |
| `filledBadge` | `string \| { label, style }` | — | Optional commit-state badge (e.g. ICD code prefix). |
| `state` | `'default' \| 'hover' \| 'focus' \| 'typing' \| 'filled'` | _auto_ | Forces visual state for design-system grids. |
| `width` · `minWidth` · `maxWidth` | px | `—` · `320` · `640` | Matches Figma Large sizing. |
| `showShadow` | `boolean` | `true` | Toggle the indigo drop. |

**Where it ships today.** The Step 4 Doctor-AI panel ([src/components/forms/TreatmentSteps.jsx](src/components/forms/TreatmentSteps.jsx) → `DoctorAiPanel`) renders `<AISearch>` inside a gradient-bordered card carrying the "Doctor AI" Lottie wordmark + the `ค้นหารหัสโรค` caption + the indigo-bold suggestion `TabCapsule` chips below the bar. The pattern: **AISearch handles the search input + dropdown**, the page surrounds it with the panel chrome.

---

#### 6.17.7 State Summary

**Primary Search:**

| State | Border Token | Primitive | Border-width | Icon Token | Icon Color |
|-------|-------------|-----------|-------------|------------|------------|
| Default | `border/primarySearch/default` | `emerald/600` | `2px` | `icon/primarySearch/default` | `#08A768` |
| Hover | `border/primarySearch/hover` | `emerald/700` | `2px` | `icon/primarySearch/hover` | `#007549` |
| Typing | `border/primarySearch/typing` | `emerald/700` | `2px` | `icon/primarySearch/typing` | `#007549` |
| Filled | `border/primarySearch/filled` | `emerald/600` | `2px` | `icon/primarySearch/filled` | `#08A768` |

**Secondary Search:**

| State | Border Token | Primitive | Border-width | Focus Ring |
|-------|-------------|-----------|-------------|------------|
| Default | `border/secondarySearch/default` | `neutral/300` | `1px` | — |
| Hover | `border/secondarySearch/hover` | `emerald/700` | `2px` | `primaryBorder/600` · 4px |
| Typing | `border/secondarySearch/typing` | `emerald/700` | `2px` | `primaryBorder/600` · 4px |
| Filled | `border/secondarySearch/filled` | `neutral/300` | `1px` | — |

**AI Search** (indigo-themed Doctor AI panel — Figma 156:27359):

| State | Border Token | Light → Dark | Icon Token | Light → Dark |
|-------|-------------|--------------|------------|--------------|
| Default | `border/aiSearch/default` | `indigo/500` (no swap) | `icon/aiSearch/default` | `indigo/600` → `indigo/400` |
| Hover | `border/aiSearch/hover` | `indigo/700` → `indigo/300` | `icon/aiSearch/hover` | `indigo/700` → `indigo/300` |
| Typing | `border/aiSearch/typing` | `indigo/700` → `indigo/300` | `icon/aiSearch/typing` | `indigo/700` → `indigo/300` |
| Filled | `border/aiSearch/filled` | `indigo/600` → `indigo/400` | `icon/aiSearch/filled` | `indigo/600` → `indigo/400` |

---

#### 6.17.8 All Color Tokens Reference

| Semantic Token | Category | Primitive / Brand | Light Mode |
|---------------|----------|-------------------|------------|
| `surface/search/default` | Surface | `grey/50` | `#FFFFFF` |
| `surface/list/default` | Surface | `grey/50` | `#FFFFFF` |
| `surface/list/hover` | Surface | `primary/50` = `var(--em50)` | brand-responsive ✓ |
| `surface/dropdown/default` | Surface | `grey/50` | `#FFFFFF` |
| `surface/brandPrimaryButton/default` | Surface | `emerald/700` | `#007549` |
| `border/primarySearch/default` | Border | `emerald/600` | `#08A768` |
| `border/primarySearch/hover` | Border | `emerald/700` | `#007549` |
| `border/primarySearch/typing` | Border | `emerald/700` | `#007549` |
| `border/primarySearch/filled` | Border | `emerald/600` | `#08A768` |
| `border/secondarySearch/default` | Border | `neutral/300` | `#C9CDD0` |
| `border/secondarySearch/hover` | Border | `emerald/700` | `#007549` |
| `border/secondarySearch/typing` | Border | `emerald/700` | `#007549` |
| `border/secondarySearch/filled` | Border | `neutral/300` | `#C9CDD0` |
| `border/dropdown/default` | Border | `grey/600` | `#EBEBEB` |
| `text/primarySearch/default` | Text | `neutral/400` | `#ADB2B7` |
| `text/primarySearch/hover` | Text | `neutral/700` | `#4D5358` |
| `text/primarySearch/typing` | Text | `neutral/700` | `#4D5358` |
| `text/primarySearch/filled` | Text | `neutral/700` | `#4D5358` |
| `text/secondarySearch/default` | Text | `neutral/400` | `#ADB2B7` |
| `text/secondarySearch/hover` | Text | `neutral/700` | `#4D5358` |
| `text/secondarySearch/typing` | Text | `neutral/700` | `#4D5358` |
| `text/secondarySearch/filled` | Text | `neutral/700` | `#4D5358` |
| `text/list/default` | Text | `neutral/700` | `#4D5358` |
| `icon/primarySearch/default` | Icon | `primary/600` = `emerald/600` | `#08A768` |
| `icon/primarySearch/hover` | Icon | `primary/700` = `emerald/700` | `#007549` |
| `icon/primarySearch/typing` | Icon | `primary/700` = `emerald/700` | `#007549` |
| `icon/primarySearch/filled` | Icon | `primary/600` = `emerald/600` | `#08A768` |
| `icon/brandPrimary/on-brand` | Icon | `grey/50` | `#FFFFFF` |
| `icon/secondarySearch/default` | Icon | `neutral/400` | `#ADB2B7` |
| `icon/secondarySearch/typing` | Icon | `neutral/700` | `#4D5358` |
| `PrimaryShadow/600` | Shadow primitive | — | `rgba(8,167,104,0.08)` |
| `PrimaryShadow/601` | Shadow primitive | — | `rgba(8,167,104,0.10)` |
| `primaryBorder/600` | Focus ring primitive | — | `rgba(8,167,104,0.15)` |
| `color/shadow/050` | Shadow primitive | — | `rgba(100,116,139,0.10)` |
| `color/shadow/100` | Shadow primitive | — | `rgba(100,116,139,0.15)` |

---

#### 6.17.9 All Number Tokens Reference

| Token | Resolved Value | Used In |
|-------|----------------|---------|
| `dimension/stroke/100` | `1px` | Secondary Default/Filled · Dropdown border |
| `dimension/stroke/200` | `2px` | Primary (all states) · Secondary Hover/Typing |
| `dimension/radius/100` | `4px` | Dropdown list item |
| `dimension/radius/200` | `8px` | Secondary Search container |
| `dimension/radius/500` | `16px` | Primary Search · Dropdown panel |
| `dimension/radius/600` | `24px` | Typing search button |
| `dimension/size/400` | `20px` | Primary Search icon |
| `dimension/size/500` | `24px` | Primary Search padding X |
| `dimension/size/700` | `36px` | Secondary Small height · Typing button size |
| `dimension/size/800` | `40px` | Secondary Default height |
| `dimension/size/1000` | `48px` | Dropdown list item min-height |
| `dimension/size/2300` | `224px` | Secondary min-width |
| `dimension/size/2450` | `280px` | OTP channel select card width (PHCIS-Signin 42:8722) |
| `dimension/size/2470` | `298px` | Auth primary button width (PHCIS-Signin 112:10776) |
| `dimension/size/2500` | `320px` | Primary min-width |
| `dimension/size/2700` | `448px` | Secondary max-width |
| `dimension/size/2900` | `640px` | Primary max-width |
| `dimension/space/100` | `4px` | Dropdown panel padding |
| `dimension/space/200` | `8px` | Gap · List item padding Y |
| `dimension/space/300` | `12px` | Secondary padding · List item padding X |
| `dimension/space/400` | `16px` | Primary padding Y |
| `dimension/space/600` | `24px` | Primary gap |
| `dimension/space/negative-300` | `-12px` | Recent avatar overlap |
| `typograpphy/size/xs` | `12px` | Caption ("ค้นหาล่าสุด") |
| `typograpphy/size/sm` | `14px` | Primary input text · List item text |
| `typograpphy/size/base` | `16px` | Secondary input text |
| `typograpphy/weight/regular` | `400` | All text |
| `typograpphy/family/body` | `Sarabun` | All text |

---

#### 6.17.10 Typography

| Variant | Element | Composite Token | Resolved |
|---------|---------|-----------------|---------|
| Primary | Input text | `body4` | Sarabun · Regular · 14px · lh 1.5 |
| Primary | Caption ("ค้นหาล่าสุด") | `caption1` | Sarabun · Regular · 12px · lh 1.5 |
| Secondary | Input text | `body3` | Sarabun · Regular · 16px · lh 1.5 |

---

#### Interactive Behavior

The preview panel (`id="panel-search"`) contains two independent IIFE-scoped demos:

**Primary Search (`psSetState`):**  
- **Default / Hover / Filled** — shows the search bar with a right-side avatar cluster (three overlapping 32px circular avatars, `-12px` overlap)  
- **Typing** — replaces the avatar cluster with a 36px brand circle button (dark green, white search icon) and renders a dropdown panel below the bar containing three mock list items  
- Token chip row below each state bar shows the active border, text, and icon tokens for that state  

**Secondary Search (`ssSetState` + `ssSzSet`):**  
- State selector cycles through Default / Hover / Typing / Filled  
- Height selector toggles between Default (40px) and Small (36px)  
- Hover and Typing states render the emerald 2px border and `box-shadow: 0 0 0 4px rgba(8,167,104,.15)` focus ring  
- Typing state additionally shows a `✕` clear button on the right  
- Border-width switches between `1px` and `2px` via inline style (not CSS class), matching `dimension/stroke/100` ↔ `dimension/stroke/200`


---

### 6.18 Step (Step Indicator)

> **Figma node:** `2709:6562` · MIH Design System Foundation  
> A horizontal multi-step progress indicator. Each step item is a **160 px** column containing  
> an indicator row (28 px circle + optional 1 px connector line) and a content block (title + optional subtitle).  
> **Figma props:** `type` (Default | Current | Success) · `state` (Default | Hover) · `showLine` (bool) · `showSubtext` (bool)  
> **Check icon:** Figma node `2857:2369` — 16×16 filled SVG, exported via `exportAsync({ format: 'SVG_STRING' })`

---

#### 6.18.1 Color Tokens

| CSS Variable | Figma Alias Token | Primitive | Hex | Usage |
|---|---|---|---|---|
| `--surface-steps-default` | `surface/steps/default` | emerald-100 | `#E2F3EB` | Circle bg — type=Default, state=Default |
| `--surface-steps-hover` | `surface/steps/hover` | emerald-200 | `#C5EDDA` | Circle bg — type=Default, state=Hover |
| `--surface-steps-selected` | `surface/steps/selected` | emerald-700 | `#007549` | Circle bg — type=Current |
| `--surface-steps-success` | `surface/steps/success` | teal-500 | `#24A899` | Circle bg — type=Success, state=Default |
| `--surface-steps-success-hover` | `surface/steps/success-hover` | teal-600 | `#1B867D` | Circle bg — type=Success, state=Hover |
| `--primaryborder-600` | `primaryBorder/600` | rgba(8,167,104,0.15) | — | 4px glow ring — type=Current (`box-shadow`) |
| `--border-steps-default` | `border/steps/default` | emerald-200 | `#C5EDDA` | Connector line — all states |
| `--text-steps-default` | `text/steps/default` | emerald-700 | `#007549` | Circle number — type=Default |
| `--text-steps-select` | `text/steps/select` | white | `#FFFFFF` | Circle number — type=Current |
| *(inline `#ffffff`)* | `icon/step/success-on` | white | `#FFFFFF` | Checkmark fill — type=Success |
| `--text-steps-title` | `text/steps/title` | primary-700 @50% | `rgba(0,117,73,0.5)` | Title — type=Default, state=Default |
| `--text-steps-title-hover` | `text/steps/title-hover` | primary-700 | `var(--em700)` | Title — type=Default, state=Hover |
| `--text-steps-title-select` | `text/steps/title-select` | primary-700 | `var(--em700)` | Title — type=Current (bold) |
| `--text-steps-title-success` | `text/steps/title-success` | teal-700 (fixed) | `#196c65` | Title — type=Success, **both** Default **and** Hover (not brand-responsive) |
| `--text-steps-subtitle` | `text/steps/subtitle` | neutral-500 (fixed) | `#858C92` | Subtitle — all states |

---

#### 6.18.2 Dimension Tokens

| Property | Figma Token | Value | Role |
|---|---|---|---|
| Step item width | `dimension/size/2100` | **160 px** | `.st` container width |
| Circle diameter | `dimension/size/700` | **28 px** | `.st-circle` width & height |
| Connector line height | `dimension/size/1` | **1 px** | `.st-line` height |
| Current ring | `dimension/stroke/400` | **4 px** | `box-shadow: 0 0 0 4px` via `--primaryborder-600` |
| Indicator row gap | `dimension/space/150` | **6 px** | gap: circle ↔ line |
| Column gap | `dimension/space/200` | **8 px** | gap: indicator ↓ content |
| Check icon size | `dimension/size/300` | **16 px** | `.st-check` SVG (Figma 2857:2369) |

**PHCIS product density override (2026-08-24):** the Foundation dimensions
above remain the source reference, while `StepProgress` aliases the product
surface to the nearest smaller existing tokens. No new token is introduced.

| Runtime property | Default | `compact` |
|---|---:|---:|
| Step item width | `dimension/size/1900` · 112 px | `dimension/size/1700` · 80 px |
| Visual circle | `dimension/size/400` · 20 px | `dimension/size/300` · 16 px |
| Current ring | `dimension/stroke/200` · 2 px | `dimension/stroke/200` · 2 px |
| Circle number / title | `typography/size/xs` · 12 px | `typography/size/xxs` · 10 px |
| Subtitle | `typography/size/xxs` · 10 px | hidden |
| Indicator / row gap | `dimension/space/050`–`100` · 2–4 px | `dimension/space/0`–`100` · 0–4 px |
| Interactive hit target | absolute `dimension/size/900` · **44 px** (does not affect row height) | same |

When global header morph passes 35%, `data-mih-step-inline="1"` switches the
sticky step band to an inline compact layout: number/check circle on the left,
Thai step title on the right, subtitle and connector hidden. Hysteresis returns
to the expanded stacked layout below 15%, preventing flicker near the threshold.

---

#### 6.18.3 Typography

| Element | Figma Token | Value |
|---|---|---|
| Circle number — Default/Current | `typograpphy/family/heading-content` · Bold | Sarabun Bold 14px |
| Title — Current | `typograpphy/family/heading-content` · Bold | Sarabun Bold 14px |
| Title — Default/Success | `typograpphy/family/body` · Regular | Sarabun Regular 14px |
| Subtitle | `typograpphy/family/body` · Regular | Sarabun Regular 14px, `#858C92` |

---

#### 6.18.4 States Summary

| type | state | Circle CSS class | Circle bg token | Ring | Number/Icon | Title token | Title hex |
|---|---|---|---|---|---|---|---|
| **Default** | Default | `.st-circle--default` | `surface/steps/default` | — | `text/steps/default` | `text/steps/title` | `rgba(0,117,73,.5)` regular |
| **Default** | Hover | `.st-circle--hover` | `surface/steps/hover` | — | `text/steps/default` | `text/steps/title-hover` | `var(--em700)` regular |
| **Current** | Default | `.st-circle--current` | `surface/steps/selected` | 4px `rgba(8,167,104,0.15)` | `text/steps/select` (#FFF) | `text/steps/title-select` | `var(--em700)` **bold** |
| **Success** | Default | `.st-circle--success` | `surface/steps/success` | — | ✓ SVG `#ffffff` | `text/steps/title-success` | `#196c65` (teal-700, fixed) |
| **Success** | Hover | `.st-circle--success-hover` | `surface/steps/success-hover` | — | ✓ SVG `#ffffff` | `text/steps/title-success` | `#196c65` (teal-707, fixed) |

---

#### 6.18.5 CSS Implementation

```css
/* ── Step tokens → :root ── */
:root {
  --surface-steps-default:       #E2F3EB;   /* surface/steps/default → emerald-100 */
  --surface-steps-hover:         #C5EDDA;   /* surface/steps/hover → emerald-200 */
  --surface-steps-selected:      #007549;   /* surface/steps/selected → emerald-700 */
  --surface-steps-success:       #24A899;   /* surface/steps/success → teal-500 */
  --surface-steps-success-hover: #1B867D;   /* surface/steps/success-hover → teal-600 */
  --border-steps-default:        #C5EDDA;   /* border/steps/default → emerald-200 */
  --primaryborder-600:           rgba(8,167,104,0.15); /* primaryBorder/600 */
  --text-steps-default:          #007549;   /* text/steps/default → emerald-700 */
  --text-steps-select:           #FFFFFF;   /* text/steps/select → white */
  --text-steps-title:            rgba(0,117,73,0.5); /* text/steps/title */
  --text-steps-title-hover:      var(--em700);  /* text/steps/title-hover → primary/700 */
  --text-steps-title-select:     var(--em700);  /* text/steps/title-select → primary/700 */
  --text-steps-title-success:    #196c65;        /* text/steps/title-success → teal/700 (fixed) */
  --text-steps-subtitle:         #858C92;        /* text/steps/subtitle → neutral-500 (fixed) */
}

/* ── Layout ── */
.st-row      { display:flex; align-items:flex-start; }
.st          { width:160px; display:flex; flex-direction:column; gap:8px; flex-shrink:0; }
.st-indicator{ display:flex; align-items:center; gap:6px; width:100%; }

/* Circle 28×28 */
.st-circle { width:28px; height:28px; border-radius:9999px; display:flex; align-items:center; justify-content:center; flex-shrink:0; font-family:'Sarabun',sans-serif; font-size:14px; font-weight:700; }
.st-circle--default       { background:var(--surface-steps-default); color:var(--text-steps-default); }
.st-circle--hover         { background:var(--surface-steps-hover); color:var(--text-steps-default); }
.st-circle--current       { background:var(--surface-steps-selected); color:var(--text-steps-select); box-shadow:0 0 0 4px var(--primaryborder-600); }
.st-circle--success       { background:var(--surface-steps-success); }
.st-circle--success-hover { background:var(--surface-steps-success-hover); }

/* Connector line — showLine=true (omit element for showLine=false / last step) */
.st-line { flex:1; min-width:1px; height:1px; background:var(--border-steps-default); }

/* Content */
.st-content  { display:flex; flex-direction:column; gap:2px; }
.st-title    { font-family:'Sarabun',sans-serif; font-size:14px; line-height:1.5; }
.st-title--default       { font-weight:400; color:var(--text-steps-title); }
.st-title--hover         { font-weight:400; color:var(--text-steps-title-hover); }
.st-title--current       { font-weight:700; color:var(--text-steps-title-select); }
.st-title--success       { font-weight:400; color:var(--text-steps-title-success); }  /* teal/700 fixed */
.st-title--success-hover { font-weight:400; color:var(--text-steps-title-select); }   /* primary/700 */
.st-subtitle { font-family:'Sarabun',sans-serif; font-size:14px; font-weight:400; color:var(--text-steps-subtitle); }

/* Check icon container */
.st-check { width:16px; height:16px; display:flex; align-items:center; justify-content:center; }
```

---

#### 6.18.6 Interaction — Hover & Click

> Step items are interactive. **Hover** applies the Figma Hover state via CSS. **Click** selects that step as Current, advancing/reversing the flow.

**Type modifier classes** (added dynamically by JS, or statically in HTML):

| Class | Cursor | Hover behavior |
|---|---|---|
| `.st--default` | `pointer` | Circle → `surface/steps/hover` · Title → `text/steps/title-hover` |
| `.st--success` | `pointer` | Circle → `surface/steps/success-hover` · Title → `text/steps/title-select` |
| `.st--current` | `default` | No hover change (already at Current state) |

```css
/* Hover interaction — CSS-driven, tokens from :root */
.st--default { cursor:pointer; }
.st--success { cursor:pointer; }
.st--current { cursor:default; }

/* Smooth transitions */
.st--default .st-circle--default,
.st--success .st-circle--success { transition:background .18s ease; }
.st--default .st-title--default,
.st--success .st-title--success  { transition:color .18s ease; }

/* Hover: type=Default → surface/steps/hover + text/steps/title-hover */
.st--default:hover .st-circle--default { background:var(--surface-steps-hover); }
.st--default:hover .st-title--default  { color:var(--text-steps-title-hover); }

/* Hover: type=Success → surface/steps/success-hover + text/steps/title-select */
.st--success:hover .st-circle--success { background:var(--surface-steps-success-hover); }
.st--success:hover .st-title--success  { color:var(--text-steps-title-select); }
```

**Click JS pattern:**

```js
/* Each .st in interactive demo gets onclick="_stClick(i)" */
function _stClick(idx) {
  _stCur = idx;   // jump to clicked step
  _renderSteps(); // re-render all steps
}

function _renderSteps() {
  // ...for each step i:
  var typeClass = i < _stCur ? 'st--success'
                : i === _stCur ? 'st--current'
                : 'st--default';
  // Emit: <div class="st {typeClass}" onclick="_stClick(i)" role="button" ...>
}
```

---

#### 6.18.6 Check Icon (Figma 2857:2369 — exact export)

```html
<!-- fill="white" = icon/step/success-on (#ffffff) -->
<svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
  <path d="M12.862 3.52864C13.1223 3.26829 13.5443 3.26829 13.8047 3.52864C14.065 3.78899 14.065 4.21099 13.8047 4.47134L6.47135 11.8047C6.211 12.065 5.78899 12.065 5.52865 11.8047L2.19531 8.47134C1.93496 8.21099 1.93496 7.78898 2.19531 7.52864C2.45566 7.26829 2.87767 7.26829 3.13802 7.52864L6 10.3906L12.862 3.52864Z" fill="white"/>
</svg>
```

---

#### 6.18.7 Token Alias Chain

```
Token                       Brand alias          CSS variable              Note
──────────────────────────────────────────────────────────────────────────────────
surface/steps/default       primary/100        → var(--surface-steps-default)   BRAND
surface/steps/hover         primary/200        → var(--surface-steps-hover)     BRAND
surface/steps/selected      primary/700        → var(--surface-steps-selected)  BRAND
surface/steps/success       teal/500           → #24A899  (fixed neutral)
surface/steps/success-hover teal/600           → #1B867D  (fixed neutral)
border/steps/default        primary/200        → var(--border-steps-default)    BRAND
text/steps/default          primary/700        → var(--text-steps-default)      BRAND
text/steps/select           white              → #FFFFFF  (fixed)
text/steps/title            primary/700 @50%   → var(--text-steps-title)        BRAND (opacity)
text/steps/title-hover      primary/700        → var(--text-steps-title-hover)  BRAND
text/steps/title-select     primary/700        → var(--text-steps-title-select) BRAND
text/steps/subtitle         neutral/500        → #858C92  (fixed neutral)
```

---

### 6.19 Pagination

> **Figma node:** `312:52` · `306:1814` · MIH Design System Foundation  
> `328:4734` (section) · Implemented in `mih-components-all-v2.html` · `#panel-pagination`  
> Page navigation with prev/next chevron buttons and numbered page buttons.

---

#### 6.19.1 Color Tokens

| State | Property | Semantic Token | CSS Variable | Primitive | Hex |
|---|---|---|---|---|---|
| Inactive | Surface | `surface/pagination/default` | `--surface-pagination-default` | white | `#FFFFFF` |
| Inactive | Text | `text/pagination/default` | `--text-pagination-default` | neutral/700 | `#4D5358` |
| Hover | Surface | `surface/pagination/hover` | `--surface-pagination-hover` | brand/p50 (light) · brand/p950 (dark) | `#F4FBF8` / `#002318` |
| Hover | Text | `text/pagination/hover` | `--text-pagination-hover` | brand/p700 (light) · brand/p300 (dark) | `#007549` / `#91DFBB` |
| Current | Surface | `surface/pagination/selected` | `--surface-pagination-selected` | brand/light | `#E2F3EB` |
| Current | Text | `text/pagination/selected` | `--text-pagination-selected` | brand/p700 | `#007549` |
| Ellipsis | Text | `text/neutral/tertiary` | `--text-neutral-tertiary` | neutral/400 | `#ADB2B7` |

> **Correction from previous:** `surface/pagination/selected` was previously called "Hover" in Figma state naming (node 306:1813). The correct semantic meaning is the **selected/current page** state. There is **no border** on buttons (previous implementation had `border: 1px solid #E2E4E6` which is not in Figma).

#### 6.19.2 Dimension Tokens

| Property | Design Token | Value | CSS |
|---|---|---|---|
| Button height / min-width | `dimension/size/700` | 36 px | `height:36px; min-width:36px` |
| Border radius | `dimension/radius/200` | 8 px | `--pg-radius: 8px` |
| Gap between buttons | `dimension/space/100` | 4 px | `gap:4px` (container) |
| Chevron icon size | `dimension/size/400` | 16 px | SVG `width="16" height="16"` |

> **Correction:** Button size token is `dimension/size/700` = 36px. Previous doc used `dimension/size/900` (wrong).

#### 6.19.3 Typography

| Property | Token | Value |
|---|---|---|
| Font family | `typograpphy/family/body` | `'Sarabun', sans-serif` |
| Font size | `typograpphy/size/sm` | 14 px |
| Font weight | `typograpphy/weight/regular` | 400 Regular |
| Line height | — | 1.5 |

> **Correction:** Font weight is **400 Regular** (not 500 Medium as previously implemented).

#### 6.19.4 CSS Implementation

```css
/* :root — light mode */
--surface-pagination-default:  #FFFFFF;
--surface-pagination-hover:    var(--brand-p50);    /* #F4FBF8 — alias Brand/primary/50  */
--surface-pagination-selected: var(--brand-p100);   /* #E2F3EB — alias Brand/primary/100 */
--text-pagination-default:     #4D5358;
--text-pagination-hover:       var(--brand-p700);   /* #007549 — alias Brand/primary/700 */
--text-pagination-selected:    var(--brand-p700);   /* #007549 — alias Brand/primary/700 */
--text-neutral-tertiary:       #ADB2B7;

/* Dark mode override (brand-p* primitives flip per :root[data-theme=dark]) */
/* --surface-pagination-hover  → brand/p950 (#002318) */
/* --text-pagination-hover     → brand/p300 (#91DFBB) */

/* Component */
.pagination { display: flex; gap: 4px; align-items: center; }  /* dimension/space/100 */

.page-btn {
  min-width: 36px; height: 36px;                 /* dimension/size/700 */
  border-radius: 8px;                            /* dimension/radius/200 */
  border: none;                                  /* Figma: no border */
  font-size: 14px; font-weight: 400;             /* typograpphy/size/sm · weight/regular */
  font-family: 'Sarabun', sans-serif;            /* typograpphy/family/body */
  color: var(--text-pagination-default);         /* #4D5358 */
  background: var(--surface-pagination-default); /* #FFFFFF */
}
.page-btn:hover:not(.ellipsis):not([disabled]) {
  background: var(--surface-pagination-hover);   /* brand/p50  → #F4FBF8 */
  color: var(--text-pagination-hover);           /* brand/p700 → #007549 */
}
.page-btn.active {
  background: var(--surface-pagination-selected); /* #E2F3EB */
  color: var(--text-pagination-selected);         /* #007549 */
}
.page-btn.ellipsis {
  color: var(--text-neutral-tertiary);            /* #ADB2B7 */
  background: transparent;
}
```

#### 6.19.5 Chevron Icon

Prev/Next use inline SVG (replaces Figma asset URL which expires in 7 days):

```html
<!-- Prev (chevron-left) -->
<svg width="16" height="16" viewBox="0 0 24 24" fill="none"
     stroke="currentColor" stroke-width="1.8"
     stroke-linecap="round" stroke-linejoin="round">
  <polyline points="15 18 9 12 15 6"/>
</svg>

<!-- Next (chevron-right) -->
<svg width="16" height="16" viewBox="0 0 24 24" fill="none"
     stroke="currentColor" stroke-width="1.8"
     stroke-linecap="round" stroke-linejoin="round">
  <polyline points="9 18 15 12 9 6"/>
</svg>
```

Icon inherits `currentColor` from `.page-btn` → `text/pagination/default` (#4D5358).

---


### 6.21 Radio

> **Figma nodes:** `328-5812` (Radio Dot — States) · `2860-3497` (RadioButton Card)
> **Source file:** MIH Design System Foundation
> **v2 classes:** `.rd`, `.rd--*`, `.rb`, `.rb--*`, `.rb-content`, `.rb-title`, `.rb-desc`, `.rb-badge`

---

#### 6.21.1 Color Tokens

##### Radio Dot — CSS Variables (`:root`)

| CSS Variable | MIH DS Token | Primitive | Value | Usage |
|---|---|---|---|---|
| `--surface-radio-default` | `surface/input/default` | white | `#FFFFFF` | Default + Disabled-Unchecked background |
| `--border-radio-default` | `border/switch/default` | neutral-300 | `#C9CDD0` | Default border |
| `--border-radio-hover` | `surface/switch/on-hover` | teal-700 | `#196C65` | Hover border |
| `--dot-radio-hover` | `surface/switch/on-hover` | teal-700 | `#196C65` | Hover inner dot fill |
| `--border-radio-selected` | `surface/switch/on` | teal-600 | `#1B867D` | Selected border |
| `--dot-radio-selected` | `surface/switch/on` | teal-600 | `#1B867D` | Selected inner dot fill |
| `--surface-radio-disabled` | `surface/disabled/secondary` | neutral-200 | `#E2E4E6` | Disabled-Checked bg + border |
| `--border-radio-disabled` | `border/switch/disabled` | neutral-200 | `#E2E4E6` | Disabled-Unchecked border |

##### RadioButton Card — CSS Variables (`:root`)

| CSS Variable | MIH DS Token | Primitive | Value | Usage |
|---|---|---|---|---|
| `--surface-rb-default` | `surface/neutralbutton/quinary` | white | `#FFFFFF` | Default / Hover / Selected / Disabled background |
| `--border-rb-default` | `border/neutralbutton/tertiary` | neutral-200 | `#E2E4E6` | Default / Disabled border (1px) |
| `--border-rb-hover` | `border/brandPrimaryButton/tertiary` | brand-p200 | `#C5EDDA` (default brand) | Hover border (2px) — brand-aware |
| `--border-rb-selected` | `border/brandPrimaryButton/default` | brand-p700 | `#007549` (default brand) | Selected border (2px) — brand-aware, retints on `<html data-brand>` |
| `--shadow-rb-elevated` | `Brand Drop Shadow Top/200` (`--shadow-brand-top-200`) | brand-p600 @ 8/10% | `0 -1px 4px / 0 -1px 8px` rgba(brand-p600 …) | Lift cue for Hover + Selected (PrimaryShadow/600 + 601 stack) |
| `--text-rb-title` | `text/content/default` | neutral-900 | `#363B3F` | Title — active states |
| `--text-rb-title-disabled` | `text/content/quaternary` | neutral-400 | `#ADB2B7` | Title — disabled |
| `--text-rb-desc` | `text/content/tertiary` | neutral-500 | `#858C92` | Description — active states |
| `--text-rb-desc-disabled` | `text/content/quaternary` | neutral-400 | `#ADB2B7` | Description — disabled |

##### RadioButton Icon Slots — CSS Variables (`:root`)

| CSS Variable | MIH DS Token | Primitive | Value | Usage |
|---|---|---|---|---|
| `--surface-rb-circle-icon` | `surface/brandprimary/quaternary` | emerald-50 | `#F4FBF8` | Circle icon container background |
| `--icon-rb-primary` | `icon/brandPrimary/default` | emerald-700 | `#007549` | Stethoscope icon stroke |
| `--icon-rb-default` | `icon/content/secondary` | neutral-700 | `#4D5358` | Emoji icon fill — MIH DS `sentiment_satisfied` (2831-6734) |

---

#### 6.21.2 Radio Dot — States (Figma 328-5812)

| State | CSS Class | Border | Background | Inner Dot |
|---|---|---|---|---|
| Default | `.rd` | `--border-radio-default` #C9CDD0 | `--surface-radio-default` #FFFFFF | none |
| Hover | `.rd.rd--hover` | `--border-radio-hover` #196C65 | `--surface-radio-default` #FFFFFF | `--dot-radio-hover` #196C65 |
| Selected | `.rd.rd--selected` | `--border-radio-selected` #1B867D | `--surface-radio-default` #FFFFFF | `--dot-radio-selected` #1B867D |
| Disabled-Checked | `.rd.rd--dis-checked` | `--surface-radio-disabled` #E2E4E6 | `--surface-radio-disabled` #E2E4E6 | `#FFFFFF` |
| Disabled-Unchecked | `.rd.rd--dis-unchecked` | `--border-radio-disabled` #E2E4E6 | `--surface-radio-default` #FFFFFF | none |

---

#### 6.21.3 RadioButton Card — States (Figma 2860-3497, border=true)

| State | CSS Classes | Background | Border | Border-width | Drop shadow |
|---|---|---|---|---|---|
| Default | `.rb` | `--surface-rb-default` #FFFFFF | `--border-rb-default` #E2E4E6 | 1px | — |
| Hover | `.rb` (`:hover`) | `--surface-rb-default` #FFFFFF | `--border-rb-hover` #C5EDDA | 2px | `--shadow-rb-elevated` (Brand Drop Shadow Top/200) |
| Selected | `.rb.rb--selected` | `--surface-rb-default` #FFFFFF | `--border-rb-selected` #007549 | 2px | `--shadow-rb-elevated` (Brand Drop Shadow Top/200) |
| Disabled-Checked | `.rb.rb--disabled` | `--surface-rb-default` #FFFFFF | `--border-rb-default` #E2E4E6 | 1px | — |
| Disabled-Unchecked | `.rb.rb--disabled` | `--surface-rb-default` #FFFFFF | `--border-rb-default` #E2E4E6 | 1px | — |

**Notes:**
- **Hover** and **Selected** share the same brand drop shadow (`--shadow-brand-top-200` — green-tinted, -Y offset) so engaged cards read as one elevation family. Only the border colour distinguishes them: Hover stays on the soft brand-p200 tint, Selected steps up to the solid brand-p700 outline.
- **border=false:** Add `.rb--no-border` → removes border + shadow, min-height drops to 40px.
- **Anti-jump:** React `RadioButton` uses a permanent 2px transparent CSS border and draws the visible stroke through an INSET box-shadow so the 1px ↔ 2px swap on hover/selected does not nudge sibling cards. The same shadow stack carries `--shadow-brand-top-200` as a second layer for the lift cue.
- **Brand-aware:** `--border-rb-hover` aliases `border/brandPrimaryButton/tertiary` (brand-p200) and `--border-rb-selected` aliases `border/brandPrimaryButton/default` (brand-p700). Both retint automatically when `<html data-brand="…">` switches palette.

---

#### 6.21.4 Component Properties (Figma 2860-3497)

| Prop | Type | Demo | Description |
|---|---|---|---|
| `border` | boolean | `rbSetProp('border', true/false, btn)` | Show card border (56px) vs no-border (40px) |
| `showTitle` | boolean | `rbSetProp('showTitle', true/false, btn)` | Show `.rb-title` label |
| `showDescription` | boolean | `rbSetProp('showDesc', true/false, btn)` | Show `.rb-desc` description |
| `showBadge` | boolean | `rbSetProp('showBadge', true/false, btn)` | Show `.rb-badge` pill next to title |
| `showRadio` | boolean | — | Controls `.rd` dot visibility |
| `showIcon` | boolean | — | 24px icon slot |
| `showCircleIcon` | boolean | — | 36px icon circle |
| `showBadgeTitle` | boolean | — | Inline badge next to title |
| `state` | enum | — | `Default` · `Hover` · `Selected` · `Disabled-Checked` · `Disabled-Unchecked` |

---

#### 6.21.4a Icon Slots — showCircleIcon and showIcon

##### showCircleIcon — `CircleStethoscope` (Figma node 2796-5522)

> A 36×36 teal pill containing a stethoscope icon. Rendered between the radio dot and content column.

| Property | MIH DS Token | Value |
|---|---|---|
| Container size | `dimension/size/700` | 36 × 36 px |
| Container radius | `dimension/radius/full` | 9999 px (pill) |
| Container bg | `surface/brandprimary/quaternary` → `--surface-rb-circle-icon` | `#F4FBF8` |
| Icon size | `dimension/size/400` | 20 × 20 px |
| Icon color | `icon/brandPrimary/default` → `--icon-rb-primary` | `#007549` |
| Icon | Stethoscope — exact path from Figma node 2796:4679 | `fill="currentColor"` |

```html
<div class="rb-circle-icon">
  <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor"
       xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
    <path fill-rule="evenodd" clip-rule="evenodd" d="M9.16992 0.839844C9.6283 0.839844 9.99994 1.21158 10 1.66992V1.69043C10.6567 1.6..."/>
  </svg>
</div>
```

> **Source:** Figma `exportAsync({{ format: 'SVG_STRING' }})` on node `2796:4679` — exact MIH DS stethoscope vector.

##### showIcon — `SentimentSatisfied` (Figma node 2831-6734 · MIH DS)

> A 20×20 filled `sentiment_satisfied` icon exported directly from the MIH Design System (Figma node 2831:6699).
> Rendered between radio dot (or CircleIcon) and content column.

| Property | MIH DS Token | Value |
|---|---|---|
| Icon size | `dimension/size/400` | 20 × 20 px |
| Icon color | `icon/content/secondary` → `--icon-rb-default` | `#4D5358` (neutral-700) |
| Icon | `sentiment_satisfied` — exact path from Figma node 2831:6699 | `fill="currentColor"` |

```html
<div class="rb-icon">
  <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor"
       xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
    <path d="M10 15.5C10.8833 15.5 11.7208 15.3083 12.5125 14.925C13.3042 14.5417 13.9417 13...."/>
  </svg>
</div>
```

> **Source:** Figma `exportAsync({{ format: 'SVG_STRING' }})` on node `2831:6699` — no Figma asset URL (no 7-day expiry).

**Slot layout order (left → right):** `[radio dot]` `[circle-icon?]` `[icon?]` `[content]` `[badge?]`

> Both slots use **inline SVG** instead of Figma asset URLs — avoids the 7-day expiry limitation.

#### 6.21.5 Dimension Tokens

| Property | MIH DS Token | Value | Usage |
|---|---|---|---|
| Dot container | `dimension/size/400` | 16 × 16 px | `.rd` width + height |
| Dot stroke | `dimension/stroke/150` | 1.5 px | `.rd` border-width |
| Dot radius | `dimension/radius/full` | 9999 px | `.rd` border-radius |
| Inner dot | `dimension/size/200` | 8 px | `.rd::after` width + height |
| Card gap | `dimension/space/200` | 8 px | `.rb` gap (dot ↔ content) |
| Card padding-x | `dimension/space/400` | 16 px | `.rb` left + right padding |
| Card padding-y | `dimension/space/200` | 8 px | `.rb` top + bottom padding |
| Card radius | `dimension/radius/200` | 8 px | `.rb` border-radius |
| Card border | `dimension/stroke/100` | 1 px | Default / Disabled |
| Card border hover + selected | `dimension/stroke/200` | 2 px | `.rb:hover` and `.rb--selected` |
| Card min-height (border) | — | 56 px | `.rb` |
| Card min-height (no-border) | — | 40 px | `.rb.rb--no-border` |

---

#### 6.21.6 Typography

| Element | Token | Value |
|---|---|---|
| Title | `typograpphy/size/sm` · `weight/regular` | 14px · 400 · line-height 1.5 |
| Description | `typograpphy/size/xs` · `weight/regular` | 12px · 400 · line-height 1.5 |
| Font family | `typograpphy/family/body` | Sarabun |

---

#### 6.21.7 CSS Implementation

```css
/* ── Radio Dot (rd) — Figma 328-5812 ── */
.rd {
  width: 16px; height: 16px;               /* dimension/size/400 */
  border-radius: 9999px;                   /* dimension/radius/full */
  border: 1.5px solid var(--border-radio-default, #C9CDD0); /* dimension/stroke/150 */
  background: var(--surface-radio-default, #FFFFFF);
  flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  transition: border-color .15s, background .15s;
  box-sizing: border-box;
}
.rd::after {
  content: '';
  width: 8px; height: 8px;                /* inner dot = size/400 ÷ 2 */
  border-radius: 50%;
  background: transparent;
  transition: background .15s;
}

/* State modifiers */
.rd.rd--hover       { border-color: var(--border-radio-hover, #196C65); }
.rd.rd--hover::after { background: var(--dot-radio-hover, #196C65); }
.rd.rd--selected    { border-color: var(--border-radio-selected, #1B867D); }
.rd.rd--selected::after { background: var(--dot-radio-selected, #1B867D); }
.rd.rd--dis-checked { background: var(--surface-radio-disabled, #E2E4E6);
                      border-color: var(--surface-radio-disabled, #E2E4E6); }
.rd.rd--dis-checked::after { background: #FFFFFF; }
.rd.rd--dis-unchecked { background: var(--surface-radio-default, #FFFFFF);
                        border-color: var(--border-radio-disabled, #E2E4E6); }

/* ── RadioButton Card (rb) — Figma 2860-3497 ──
   Anti-jump: keep the CSS border at a permanent 2px transparent so the
   layout box stays identical in every state. The visible stroke is
   painted by an INSET box-shadow whose width swaps 1px → 2px and whose
   colour swaps neutral → brand-p200 → brand-p700 across default → hover
   → selected. The same shadow stack carries `--shadow-brand-top-200`
   on hover/selected for the lift cue.                                 */
.rb {
  display: flex; align-items: center;
  gap: 8px;                               /* dimension/space/200 */
  padding: 8px 16px;                      /* space/200 · space/400 */
  border-radius: 8px;                     /* dimension/radius/200 */
  min-height: 56px;
  border: 2px solid transparent;          /* reserves layout space */
  background: var(--surface-rb-default, #FFFFFF);
  box-shadow: inset 0 0 0 1px var(--border-rb-default, #E2E4E6);
  cursor: pointer;
  transition: background .15s, box-shadow .15s;
  box-sizing: border-box;
}
.rb:not(.rb--selected):not(.rb--disabled):hover {
  box-shadow:
    inset 0 0 0 2px var(--border-rb-hover, #C5EDDA),
    var(--shadow-rb-elevated, var(--shadow-brand-top-200));
}
.rb.rb--no-border   { border: none; box-shadow: none; min-height: 40px; }
.rb.rb--selected    {
  box-shadow:
    inset 0 0 0 2px var(--border-rb-selected, #007549),
    var(--shadow-rb-elevated, var(--shadow-brand-top-200));
}
.rb.rb--disabled    { cursor: default; }

/* Content column */
.rb-content { flex: 1; display: flex; flex-direction: column; gap: 2px; min-width: 0; }

/* Text */
.rb-title  { font-family: 'Sarabun', sans-serif; font-size: 14px; font-weight: 400;
             color: var(--text-rb-title, #363B3F); line-height: 1.5; }
.rb-title--disabled { color: var(--text-rb-title-disabled, #ADB2B7); }
.rb-desc   { font-family: 'Sarabun', sans-serif; font-size: 12px; font-weight: 400;
             color: var(--text-rb-desc, #858C92); line-height: 1.5; }
.rb-desc--disabled { color: var(--text-rb-desc-disabled, #ADB2B7); }

/* Circle icon slot — showCircleIcon (Figma 2796-5522) */
.rb-circle-icon {
  width: 36px; height: 36px;          /* dimension/size/700 */
  border-radius: 9999px;              /* dimension/radius/full */
  background: var(--surface-rb-circle-icon, #F4FBF8); /* surface/brandprimary/quaternary */
  flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
}
.rb-circle-icon svg {
  width: 20px; height: 20px;          /* dimension/size/400 */
  color: var(--icon-rb-primary, #007549); /* icon/brandPrimary/default */
}

/* Emoji icon slot — showIcon (Figma 2831-6710) */
.rb-icon {
  width: 24px; height: 24px;          /* dimension/size/500 */
  flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  color: var(--icon-rb-default, #858C92); /* icon/content/default */
}
.rb-icon svg { width: 24px; height: 24px; }

/* Badge pill (showBadge) */
.rb-badge {
  display: inline-flex; align-items: center;
  padding: 2px 8px;                       /* space/100 · space/200 */
  border-radius: 9999px;                  /* dimension/radius/full */
  background: var(--surface-pagination-selected, #E2F3EB); /* reused: surface/brandPrimary/light */
  color: var(--text-pagination-selected, #007549);          /* reused: text/brand/primary */
  font-family: 'Sarabun', sans-serif; font-size: 11px;
  white-space: nowrap;
}
```

---

#### 6.21.8 JavaScript — Interactive Demo

```javascript
/* State */
var _rbState   = { border: true, showTitle: true, showDesc: true, showBadge: false, showCircleIcon: false, showIcon: false };
var _rbItems   = [
  { id: 0, title: 'นัดหมายปกติ',      desc: 'ระบบจองคิวตามปกติ',           badge: 'ฟรี' },
  { id: 1, title: 'นัดหมายด่วน',      desc: 'ต้องการพบแพทย์ภายใน 24 ชม.',  badge: 'แนะนำ' },
  { id: 2, title: 'การปรึกษาออนไลน์', desc: 'Telemedicine consultation',    badge: 'ใหม่' },
  { id: 3, title: 'Emergency',          desc: 'ติดต่อเจ้าหน้าที่โดยตรง',      badge: null, disabled: true }
];
var _rbSelected = 1;

/* Called by demo-prop-btn onclick */
function rbSetProp(prop, val, btn) {
  _rbState[prop] = val;
  document.getElementById('rb-prop-controls')
    .querySelectorAll('button').forEach(function(b) {
      if ((b.getAttribute('onclick') || '').indexOf("'" + prop + "'") !== -1)
        b.classList.remove('active');
    });
  btn.classList.add('active');
  _renderRb();
}

/* Render the live group */
function _renderRb() {
  var grp = document.getElementById('rb-live-group');
  if (!grp) return;
  grp.innerHTML = '';
  _rbItems.forEach(function(item, idx) {
    var isSel = (idx === _rbSelected) && !item.disabled;
    var isDis = !!item.disabled;
    var card  = document.createElement('div');
    card.className = 'rb'
      + (_rbState.border ? '' : ' rb--no-border')
      + (isSel ? ' rb--selected' : '')
      + (isDis ? ' rb--disabled' : '');
    if (!isDis) { (function(i){ card.onclick = function(){ _rbSelected=i; _renderRb(); }; })(idx); }
    /* dot */
    var dot = document.createElement('div');
    dot.className = isDis && isSel ? 'rd rd--dis-checked'
                  : isDis         ? 'rd rd--dis-unchecked'
                  : isSel         ? 'rd rd--selected'
                  : 'rd';
    card.appendChild(dot);
    /* content */
    var content = document.createElement('div');
    content.className = 'rb-content';
    if (_rbState.showTitle) {
      var row = document.createElement('div');
      row.style.cssText = 'display:flex;align-items:center;gap:8px;flex-wrap:wrap;';
      var t = document.createElement('div');
      t.className = 'rb-title' + (isDis ? ' rb-title--disabled' : '');
      t.textContent = item.title;
      row.appendChild(t);
      if (_rbState.showBadge && item.badge) {
        var b = document.createElement('span');
        b.className = 'rb-badge';
        b.textContent = item.badge;
        row.appendChild(b);
      }
      content.appendChild(row);
    }
    if (_rbState.showDesc) {
      var d = document.createElement('div');
      d.className = 'rb-desc' + (isDis ? ' rb-desc--disabled' : '');
      d.textContent = item.desc;
      content.appendChild(d);
    }
    card.appendChild(content);
    grp.appendChild(card);
  });
}

/* Call once on page init */
_renderRb();
```

---

#### 6.21.9 HTML Snippet — Dot States Showcase

```html
<!-- 5 states side-by-side -->
<div class="rd"></div>                  <!-- Default -->
<div class="rd rd--hover"></div>         <!-- Hover -->
<div class="rd rd--selected"></div>      <!-- Selected -->
<div class="rd rd--dis-checked"></div>   <!-- Disabled-Checked -->
<div class="rd rd--dis-unchecked"></div> <!-- Disabled-Unchecked -->
```

---

#### 6.21.10 HTML Snippet — RadioButton Card

```html
<!-- Default -->
<div class="rb">
  <div class="rd"></div>
  <div class="rb-content">
    <div class="rb-title">ตัวเลือก</div>
    <div class="rb-desc">คำอธิบายเพิ่มเติม</div>
  </div>
</div>

<!-- Selected -->
<div class="rb rb--selected">
  <div class="rd rd--selected"></div>
  <div class="rb-content">
    <div class="rb-title">ตัวเลือก — Selected</div>
    <div class="rb-desc">คำอธิบายเพิ่มเติม</div>
  </div>
</div>

<!-- Disabled-Checked -->
<div class="rb rb--disabled">
  <div class="rd rd--dis-checked"></div>
  <div class="rb-content">
    <div class="rb-title rb-title--disabled">ตัวเลือก</div>
    <div class="rb-desc rb-desc--disabled">ปิดใช้งาน</div>
  </div>
</div>

<!-- No border + Badge -->
<div class="rb rb--no-border">
  <div class="rd"></div>
  <div class="rb-content">
    <div style="display:flex;align-items:center;gap:8px;">
      <span class="rb-title">ตัวเลือก</span>
      <span class="rb-badge">แนะนำ</span>
    </div>
  </div>
</div>
```

---

#### 6.21.11 Token Alias Chain

| CSS Variable | → Semantic Token | → Primitive | Hex |
|---|---|---|---|
| `--border-radio-default` | `border/switch/default` | `brand-p300` / neutral-300 | `#C9CDD0` |
| `--border-radio-hover` | `surface/switch/on-hover` | `teal-700` | `#196C65` |
| `--dot-radio-hover` | `surface/switch/on-hover` | `teal-700` | `#196C65` |
| `--border-radio-selected` | `surface/switch/on` | `teal-600` | `#1B867D` |
| `--dot-radio-selected` | `surface/switch/on` | `teal-600` | `#1B867D` |
| `--surface-radio-disabled` | `surface/disabled/secondary` | `neutral-200` | `#E2E4E6` |
| `--border-radio-disabled` | `border/switch/disabled` | `neutral-200` | `#E2E4E6` |
| `--surface-rb-hover` | `surface/neutral/quaternary` | `neutral-50` | `#F9FBFB` |
| `--border-rb-default` | `border/neutralbutton/tertiary` | `neutral-200` | `#E2E4E6` |
| `--border-rb-selected` | `border/brandprimarybutton/tertiary` | `emerald-200` | `#C5EDDA` |
| `--text-rb-title` | `text/content/default` | `neutral-900` | `#363B3F` |
| `--text-rb-title-disabled` | `text/content/quaternary` | `neutral-400` | `#ADB2B7` |
| `--text-rb-desc` | `text/content/tertiary` | `neutral-500` | `#858C92` |
| `--text-rb-desc-disabled` | `text/content/quaternary` | `neutral-400` | `#ADB2B7` |
| `--surface-rb-circle-icon` | `surface/brandprimary/quaternary` | `emerald-50` | `#F4FBF8` |
| `--icon-rb-primary` | `icon/brandPrimary/default` | `emerald-700` | `#007549` |
| `--icon-rb-default` | `icon/content/secondary` | `neutral-700` | `#4D5358` |

---


### 6.20 Form Builder

> **Figma nodes:** `2210:7821` (Content) · `2286:10086` / `2451:13968` (Signature)  
> **MIH Design System Foundation** · Used in: form section drag-and-drop UI, signature capture

Form Builder has two variants and two sizes:

- **Content** — 204 px wide card with icon box (36×36), title, and subtitle. Draggable field type selector.
- **Signature** — 323 px wide area with centered pen icon and label for signature capture.

**Sizes:**
- **Default** — vertical layout (`flex-direction: column`), `min-height: 192px`, `align-items: center`, `justify-content: center`. Content centered both axes.
- **Small** — horizontal layout (`flex-direction: row`), **`height: 80px`** fixed, **`flex-wrap: nowrap`**, `align-items: center` (vertical center). Inner structure: `[icon-pill + text-stack]` (gap `12px`, `flex: 1`) → `[Button]` (`flex-shrink: 0`). Figma node `2819:6415`.

Both variants share the same background and border tokens. The border is **always dashed**.

> **Border width rule:** `dimension/stroke/150` = **1.5 px** applies to Default, **Hover**, and **Disabled** states. Error and Filled states use 1 px.
>
> **Hover note:** `surface/formBuilder/hover` resolves to the **same primitive** as `surface/formBuilder/default` (`primary/50`). The only visual change on hover is the border color: `border/formBuilder/default` (`primary/200`) → `border/formBuilder/hover` (`primary/600`).
>
> **Brand cascade:** All Form Builder border tokens use semantic CSS vars (`--border-formbuilder-*`) in `:root` which are updated by `setBrandTheme()`. Border color changes automatically when switching brands.

---

#### Color Tokens

| Figma Alias Token | Brand | CSS Var | State | Usage |
|---|---|---|---|---|
| `surface/formBuilder/default` | `primary/50` | `var(--em50)` | Default | Card / Signature background |
| `border/formBuilder/default` | `primary/200` | `var(--border-formbuilder-default)` | Default | Dashed border — default · **1.5 px** |
| `surface/formBuilder/hover` | `primary/50` | `var(--em50)` | Hover | Card / Sig background — hover ¹ |
| `border/formBuilder/hover` | `primary/600` | `var(--border-formbuilder-hover)` ★ | Hover | Dashed border — hover · **1.5 px** (Figma `2210:7836`) |
| `surface/formBuilder/disabled` | `neutral/50` | `var(--n50, #f4f5f5)` | Disabled | Background — disabled (opacity 0.55) |
| `border/formBuilder/disabled` | `neutral/200` | `var(--n200, #e2e4e6)` | Disabled | Dashed border — disabled · **1.5 px** |
| `icon/formBuilder/iconBox-default` | `primary/100` | `var(--em100)` | Default | Icon box background — default |
| `icon/formBuilder/iconBox-hover` | `primary/100` | `var(--em100)` | Hover | Icon box background — hover ¹ |
| `icon/formBuilder/content` | `primary/700` | `var(--icon-formbuilder-content)` | All | **"Type"** icon inside icon box (Content variant) |
| `icon/formBuilder/default` | `primary/700` | `var(--icon-formbuilder-default)` | All | **"Edit 3"** pen icon (Signature variant) |
| `text/formBuilder/text` | `primary/700` | `var(--em700)` | All | Card title — h7 style |
| `text/formBuilder/subtext` | `neutral/500` | `#858c92` (fixed) | All | Subtitle / hint label — label2 style |
| `surface/brandPrimaryButton/default` | `primary/700` | `var(--surface-brandPrimaryButton-default)` | All | Submit button + Toast bg |
| `border/formBuilder/filled` | `primary/100` | `var(--border-formbuilder-filled)` | Filled | DnD filled-card border · 1 px solid |

> ¹ Background resolves to the same hex as the Default state — border color is the sole visible differentiator.  
> ★ `--border-formbuilder-hover` is set in `:root` (default `#08a768`) and updated by `setBrandTheme()` → always matches current brand primary/600.

---

#### Dimension Tokens

| Property | Token | Value | CSS Var | Role |
|---|---|---|---|---|
| Border stroke | `dimension/stroke/150` | **1.5 px** | `var(--fb-stroke)` → `var(--dim-stroke-150)` | Border width (always dashed) — Figma `2210:7836` |
| Card radius | `dimension/radius/200` | **8 px** | Card corner radius |
| Icon box size | `dimension/size/700` | **36 px** | Icon box width × height |
| Icon size | `dimension/size/400` | **20 px** | Icon inside box / Signature pen |
| Gap (Content) | `dimension/space/200` | **8 px** | Gap between icon box and text stack |
| Padding (Content) | `dimension/space/300` | **12 px** | — | Content card internal padding |
| Padding (Signature) | `dimension/space/600` | **24 px** | — | Signature area internal padding |
| Gap (Signature) | `dimension/space/200` | **8 px** | — | Gap between pen icon and label |

---

#### Typography

| Token | Value | Usage |
|---|---|---|
| `typograpphy/family/heading-content` | Sarabun | Card title font family |
| `typograpphy/weight/bold` | 700 | Card title weight |
| `typograpphy/size/sm` | 14 px | Card title size → **h7** style |
| `typograpphy/family/body` | Sarabun | Subtitle / signature label font |
| `typograpphy/weight/medium` | 500 | Subtitle weight |
| `typograpphy/size/xs` | 12 px | Subtitle size → **label2** style |

---

#### States Summary

| State | Background | Border color | Border width | CSS Var | Interaction |
|---|---|---|---|---|---|
| **Default** | `#f4fbf8` (`primary/50`) | `#c5edda` (`primary/200`) | **1.5 px** dashed | `var(--border-formbuilder-default)` | — |
| **Hover** | `#f4fbf8` (same) | `#08a768` (`primary/600`) ★ | **1.5 px** dashed | `var(--border-formbuilder-hover)` | Mouse enter zone |
| **Disabled** | `#f4f5f5` (`neutral/50`) | `#e2e4e6` (`neutral/200`) | **1.5 px** dashed | `var(--n200)` | `opacity: 0.55`, `pointer-events: none` |
| **Error** | `#fff0f0` (red/50) | `#dc2626` (red/600) | 1 px dashed | — | Invalid file / upload failed |
| **Filled** | `#ffffff` | `#e2f3eb` (`primary/100`) | 1 px solid | `var(--border-formbuilder-filled)` | File uploaded |

> ★ Brand-responsive — updates automatically on `setBrandTheme()`. Verified via Figma node `2210:7836` (`dimension/stroke/150` = 1.5 px, `border/formBuilder/hover` = `#08a768`).  
> **Border rule:** Default · Hover · Disabled all use `dimension/stroke/150` = **1.5 px**. Error and Filled use 1 px.

---

#### Variant Anatomy

| Sub-element | Content (Default size) | Content (Small size) | Signature |
|---|---|---|---|
| Figma node | `2210:7821` | `2819:6415` | `2286:10086` |
| `flex-direction` | `column` | **`row`** | `column` |
| `align-items` | `center` | **`center`** (vertical) | `center` |
| `justify-content` | `center` | `flex-start` | `center` |
| `flex-wrap` | `nowrap` | **`nowrap`** | `nowrap` |
| Height | `min-height: 192px` | **`height: 80px`** fixed | auto |
| Padding | 24 px (`space/600`) | 24 px (`space/600`) | 24 px (`space/600`) |
| Icon pill | 40×40 px round, white bg, `file-up` icon | 40×40 px round, white bg, `flex-shrink: 0` | — |
| Text group | Label + sub-label (column stack) | Grouped with icon, `gap: 12px`, wrapper `flex: 1` | — |
| Gap (outer) | `gap: 8px` (items) | `gap: 12px` (icon↔text) | `gap: 8px` |
| Button | Brand outline, `flex-shrink: 0` | Brand outline, `flex-shrink: 0`, at end | — |
| Center icon | — | — | 20 px **"Edit 3"** pen icon, `#007549` |
| Action buttons | — | — | **"ล้าง"** (Neutral Subdue) + **"ยืนยันลายเซ็น"** (Brand Fill) |

> **Small size implementation note:** Icon pill and text stack must be wrapped in a single group element (`flex: 1; align-items: center; gap: 12px`) so the button always stays at the trailing edge. Do **not** use `flex-wrap: wrap` — this breaks vertical centering within the fixed 80px height.

---


---

#### Signature Capture Button Reference

The signature capture area includes two action buttons aliased directly from the MIH Design System button library.

| Button | Label | Component | Figma Node | Style |
|---|---|---|---|---|
| Clear | ล้าง | **Neutral Subdue Button** | `2024:2940` | Outline · Small |
| Confirm | ยืนยันลายเซ็น | **Brand Button** | `377:11596` | Fill · Small |

##### "ล้าง" — Neutral Subdue Button · Outline · Small

| Token | Hex | Usage |
|---|---|---|
| `border/neutralButton/tertiary` | `#e2e4e6` | Border — default |
| `surface/neutralButton/quaternary` | `#f9fbfb` | Background |
| `text/neutralButton/default` | `#4d5358` | Label text |
| `border/neutralButton/tertiary-hover` | `#c9cdd0` | Border — hover |
| `surface/disabledButton/tertiary` | `#f4f5f5` | Background — disabled |
| `border/disabledButton/tertiary` | `#e2e4e6` | Border — disabled |
| `text/disabledButton/tertiary` | `#adb2b7` | Text — disabled |

##### "ยืนยันลายเซ็น" — Brand Button · Fill · Small

| Token | Hex | Usage |
|---|---|---|
| `surface/brandPrimaryButton/default` | `#007549` | Background — default |
| `text/brandPrimaryButton/on-brand` | `#ffffff` | Label text |
| `surface/brandPrimaryButton/default-hover` | `#005a35` | Background — hover (emerald-800) |
| `Brand Drop Shadow Bottom/300` | `PrimaryShadow/600 + /601` | Button elevation shadow |
| `surface/disabledButton/tertiary` | `#f4f5f5` | Background — disabled |
| `border/disabledButton/tertiary` | `#e2e4e6` | Border — disabled |
| `text/disabledButton/tertiary` | `#adb2b7` | Text — disabled |

> Both buttons: `height: dimension/size/700` = **36 px** · `border-radius: dimension/radius/200` = **8 px** · `padding: 0 dimension/space/400` (16 px) · font: **button sm** (Sarabun Medium 14 px).
>
> **Disabled state** (before user draws a signature): "ยืนยันลายเซ็น" is disabled using `surface/disabledButton/tertiary` tokens. It becomes active only after signature strokes are drawn on the canvas.

#### 6.20.1 Drag and Drop

> **Figma node:** `2451:13908` · Sizes: **Default** (vertical, min-h 192 px) · **Small** (horizontal, h 80 px)  
> **Styles:** Default · On (same visual, different interaction context)

A file-upload drop zone with 4 states. The border is **always dashed** except in Filled state (solid). The upload icon is **file-up** (node `2452:13980`) aliased from the MIH Design System. The "Select files" button variant changes per state: **Brand Subdue** (Default) → **Brand Button** (Hover) → **Danger Button** (Error).

---

##### Color Tokens

| Figma Alias Token | Hex | State | Usage |
|---|---|---|---|
| `surface/formBuilder/default` | `#f4fbf8` | Default | Drop zone background |
| `border/formBuilder/default` | `#c5edda` | Default | Dashed border |
| `surface/formBuilder/hover` | `#f4fbf8` | Hover | Background — hover/drag-over ¹ |
| `border/formBuilder/hover` | `#08a768` | Hover | Dashed border — active drag |
| `surface/formBuilder/error` | `#fff0f0` | Error | Background — error state |
| `border/formBuilder/error` | `#dc2626` | Error | Dashed border — error |
| `text/formBuilder/error` | `#dc2626` | Error | Primary label color — error |
| `surface/formBuilder/card` | `#ffffff` | Filled | File card background |
| `border/formBuilder/filled` | `#e2f3eb` | Filled | Solid border — file card |
| `surface/card/100` | `#ffffff` | All | Upload icon circle background |
| `primary/100` | `#e2f3eb` | Filled | File thumbnail box background |
| `border/brandPrimaryButton/tertiary` | `#c5edda` | Default | **Brand Subdue Button** border — "Select files" |
| `text/brandPrimaryButton/default` | `#007549` | Default/Hover | Brand Subdue / Brand Button text |
| `border/brandPrimaryButton/default` | `#007549` | Hover | **Brand Button** border — "Select files" |
| `border/dangerButton/default` | `#dc2626` | Error | **Danger Button** border — "Select files" |
| `text/dangerButton/default` | `#b91c1c` | Error | **Danger Button** text — "Select files" |

> ¹ `surface/formBuilder/hover` = same hex as Default — border color is the sole visual change.

---

##### Dimension Tokens

| Property | Token | Value | Role |
|---|---|---|---|
| Border — Default size | `dimension/stroke/150` | **1.5 px dashed** | Vertical layout border |
| Border — Small / Filled | `dimension/stroke/100` | **1 px** | Horizontal & card border |
| Zone radius | `dimension/radius/200` | **8 px** | Drop zone & file card corners |
| Icon circle size | `dimension/size/800` | **40 px** | Upload icon pill diameter |
| Icon circle radius | `dimension/radius/full` | **9999 px** | Pill/circle shape |
| Icon in circle | `dimension/size/400` | **20 px** | File-upload icon |
| X close icon | `dimension/size/300` | **16 px** | Remove file (Filled state) |
| Button height | `dimension/size/700` | **36 px** | "Select files" button |
| Small height | `dimension/size/1700` | **80 px** | Small size fixed height |
| Zone padding | `dimension/space/600` | **24 px** | Drop zone inner padding |
| Filled padding | `dimension/space/400` | **16 px** | File card inner padding |
| Icon ↔ text gap | `dimension/space/300` | **12 px** | Gap: icon circle to text stack |
| Element gap | `dimension/space/200` | **8 px** | Between elements (vertical layout) |
| Button pad-v / gap | `dimension/space/150` | **6 px** | Button vertical padding & gap |
| Filename ↔ size gap | `dimension/space/050` | **2 px** | Text stack gap in Filled state |

---

##### Shadow Token (Icon Circle)

| Token | Value | Usage |
|---|---|---|
| `Drop Shadow Bottom/200` | `0 1px 4px rgba(100,116,139,.15), 0 1px 2px rgba(100,116,139,.10)` | Upload icon circle elevation |
| `color/shadow/100` | `rgba(100,116,139,.15)` | Outer shadow layer |
| `color/shadow/050` | `rgba(100,116,139,.10)` | Inner shadow layer |

---

##### States Summary

| State | Background | Border | Button variant | Border style |
|---|---|---|---|---|
| **Default** | `#f4fbf8` | `#c5edda` | **Brand Subdue Button** (`border/brandPrimaryButton/tertiary`) | Dashed |
| **Hover** | `#f4fbf8` ¹ | `#08a768` | **Brand Button** (`border/brandPrimaryButton/default`) | Dashed |
| **Error** | `#fff0f0` | `#dc2626` | **Danger Button** (`border/dangerButton/default`) | Dashed |
| **Filled** | `#ffffff` | `#e2f3eb` | — (file card shown) | Solid |

---

---

##### Icon Reference

Icons in Drag and Drop are aliased **directly from the MIH Design System icon library** (Simple Design System Community). Do not substitute arbitrary icons.

| Role | Icon Name | Figma Node | Color Alias | Hex | Size |
|---|---|---|---|---|---|
| Upload zone icon (Default/Hover) | **file-up** | `2452:13980` | `icon/formBuilder/default` | `#007549` | 20 px |
| Upload zone icon (Error) | **file-up** | `2452:13980` | `icon/formBuilder/error` | `#dc2626` | 20 px |
| Remove file (Filled state) | **X** | `2264:8864` | — (inherits text color) | `#64748b` | 16 px |

**file-up icon SVG** (20×20, viewBox 0 0 24 24):
```html
<svg width="20" height="20" viewBox="0 0 24 24" fill="none"
     stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <!-- file-up · node 2452:13980 · icon/formBuilder/default=#007549 -->
  <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
  <polyline points="14 2 14 8 20 8"/>
  <line x1="12" y1="18" x2="12" y2="12"/>
  <polyline points="9 15 12 12 15 15"/>
</svg>
```

**X close icon SVG** (16×16, viewBox 0 0 24 24):
```html
<svg width="16" height="16" viewBox="0 0 24 24" fill="none"
     stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <!-- X · node 2264:8864 -->
  <line x1="18" y1="6" x2="6" y2="18"/>
  <line x1="6" y1="6" x2="18" y2="18"/>
</svg>
```

> **Icon container:** The upload icon sits inside a 40 px circle pill (`surface/card/100` = `#ffffff`, `Drop Shadow Bottom/200`, `dimension/radius/full` = 9999 px).

---

##### Button Reference (per state)

| State | Button Component | Border Token | Border Hex | Text Token | Text Hex |
|---|---|---|---|---|---|
| Default | **Brand Subdue Button** | `border/brandPrimaryButton/tertiary` | `#c5edda` | `text/brandPrimaryButton/default` | `#007549` |
| Hover | **Brand Button** | `border/brandPrimaryButton/default` | `#007549` | `text/brandPrimaryButton/default` | `#007549` |
| Error | **Danger Button** | `border/dangerButton/default` | `#dc2626` | `text/dangerButton/default` | `#b91c1c` |
| Filled | — | — | — | — | — |

> All three button variants use `height: dimension/size/700` = **36 px**, `border-radius: dimension/radius/200` = 8 px, `padding: 0 dimension/space/300` (12 px horizontal), font: **button sm** (Sarabun Medium 14 px).

##### Size Variants

| Property | Default size | Small size |
|---|---|---|
| Layout | Vertical (column) | Horizontal (row) |
| Min-height | 192 px | 80 px (fixed) |
| Border stroke | `dimension/stroke/150` = 1.5 px | `dimension/stroke/100` = 1 px |
| Content order | Icon pill → label → button → hint | Icon pill + text stack → button |

---

### 6.20.5 VitalSignTimelineCard (MIH Design System / Card)

> **Figma node:** `133:1031` (variant matrix) — `133:931` (Default), `133:1448` (Hover), `133:913` (Selected), `133:1468` (Selected-Hover)
> **Project component:** `src/components/card/VitalSignTimelineCard.jsx`
> **Used in:** `HistoryTimeline` aside on OPD Screening (`/opd/screening` step 1), Treatment, and MR forms — one card per measurement round in the round picker.

A clickable card representing one vitals-measurement round. Five states. All non-default active states share the same brand drop shadow (`--shadow-card-hover`) so they read as one elevation family; only the surface / border colour distinguishes them.

#### States

| State | Surface | Border | Drop shadow | Chevron | Time colour |
|---|---|---|---|---|---|
| **default** | `surface/card/100` (white) | `border/card/default` · **1 px** neutral (`#E2E4E6`) | none | hidden | `text/content/secondary` |
| **hover** | `surface/card/100` (white) | `border/brandprimary/tertiary` · **2 px** light-brand (`#C5EDDA`) | `--shadow-card-hover` (brand-tinted) | hidden | `text/brandprimarybutton/default` |
| **selected** | `surface/card/100` (white) | `border/brandprimary/default` · **2 px** saturated-brand (`#007549`) | `--shadow-card-hover` | shown | `text/brandprimarybutton/default` |
| **selected-hover** | `surface/card/brand-100` (`#F4FBF8`, brand-p50) | `border/brandprimary/default` · **2 px** saturated-brand (`#007549`) | `--shadow-card-hover` | shown | `text/brandprimarybutton/default` |
| **disabled** | `surface/card/100` (white) | `border/card/default` · 1 px neutral | none | hidden | `text/content/tertiary` (dimmed, `opacity: 0.6`) |

#### Geometry

- **Width:** 260 px (Figma); the React component lets the parent grid stretch it via `flex-1` / explicit `width`.
- **Radius:** `dimension/radius/200` = 12 px (project alias `--dim-radius-400`).
- **Padding:** `dimension/space/300` = 12 px on every edge. When border-width grows 1 → 2 px on the active states the padding is compensated by **+1 px on each side** so inner content never reflows (the "กระตุก" guard from Figma 133:1448).
- **Gap (inner):** title row → vitals grid = `dimension/space/200` = 8 px; vitals rows themselves = `dimension/space/100` = 4 px.

#### Token aliases (project · brand-aware)

```
surface/card/100                → --surface-card-100             (white card face)
surface/card/brand-100          → --surface-card-brand100        (brand-p50 — selected-hover only)
border/card/default             → --border-card-default          (default 1 px stroke)
border/brandprimary/tertiary    → --border-brandPrimary-tertiary (hover 2 px stroke · brand-p200)
border/brandprimary/default     → --border-brandprimary-default  (selected / selected-hover 2 px stroke · brand-p700)
text/brandprimarybutton/default → --text-brandprimarybutton-default
text/brandprimarybutton/secondary → --text-brandprimarybutton-secondary
text/content/secondary          → --text-content-secondary
text/content/tertiary           → --text-content-tertiary
shadow/card/hover (Brand Drop Shadow Bottom/200) → --shadow-card-hover (composes PrimaryShadow/600 + 601)
```

Every surface / border / shadow alias is brand-aware — flipping `<html data-brand="…">` retints all five states atomically.

#### Micro-interaction contract

- Only `border-color`, `box-shadow`, and `background-color` are CSS-transitioned (150 ms `ease-out`). Border-**width** and padding snap instantly — animating 1 px → 2 px stroke produces visible subpixel jitter (Figma 133:1448).
- Default → selected-hover transitions the background fill smoothly so the surface swap on a click reads as one motion.
- `aria-pressed` reflects the `selected` prop; Enter / Space activate the card when an `onClick` is supplied.

---

### 6.20.6 VitalReadingCard (MIH Design System / Card)

> **Figma file:** `KGU3Sk3s04cgRNSbE2M78w` — MIH Design System · Card
> **Figma nodes:** `17:1504` (borderless "vital card") · `17:1458` (bordered "vital card with Border")
> **Project component:** `src/components/card/VitalReadingCard.jsx`
> **Used in:** Read-only displays — Dashboard vitals tiles, Treatment Step 2 summary header, MR patient history snapshot.

Read-only display variant of the vital-card family. **Structurally different** from the interactive `<VitalCard>` (which embeds an input + ± stepper for clinician data entry) — VitalReadingCard renders just label + value + optional icon + optional status badge, with no input controls.

#### Variant matrix (Figma)

| Set | Node | bordered | Default padding | Default width | Notes |
|---|---|---|---|---|---|
| borderless | `17:1504` | `false` | `0` | `215px` (horizontal) / `130px` (vertical) | Embedded inside other panels with own chrome |
| with Border | `17:1458` | `true`  | `--dim-space-300` (12 px) | same | Standalone card |

Each set has 4 prop combinations: `horizontal: Yes/No` × `type: reading / bmi`. Both expose `showIcon` and `showBadge` toggles.

#### Tokens

| Slot | Token | Value |
|---|---|---|
| Container surface | `--surface-card-100` | `#FFFFFF` |
| Container border (when `bordered`) | `--border-card-default` | `#E2E4E6` |
| Container border-radius | `--dim-radius-400` | **12 px** (Figma `--sds-size-radius-200`) |
| Container padding (when `bordered`) | `--dim-space-300` | 12 px |
| Inner gap | `--dim-space-200` | 8 px |
| Icon circle bg | `--surface-brandPrimary-quaternary` | `#F4FBF8` |
| Icon circle size | `--dim-size-700` | 36 px |
| Icon glyph size | `--dim-size-400` | 20 px |
| Label text | `--text-content-tertiary` / `--type-size-xs` / Medium | `#858C92` · 12 px |
| Value text | `--text-content-default` / `--type-size-base` / Bold | `#363B3F` · 16 px |
| Status badge | `<Badge style="success/warning/error/info" size="small">` | inherits semantic token |

#### BMI variant

- Replaces the value-only row with a **GradientBar gauge** (clinical scale: blue → green → orange → red).
- `bmiMarkerPercent` (0–100) positions a white circle marker over the gauge.
- Optional `bmiLabels` show ผอม / ปกติ / น้ำหนักเกิน / อ้วน beneath the bar.

#### Drift note (2026-05-25 audit)

- Tailwind `rounded-card` utility was hardcoded to **16 px** (`--dim-radius-500`) — now aliased to `var(--dim-radius-500, 16px)` so existing cards keep their visual. Figma 17:1458 specs **12 px** (`--dim-radius-400`) for these new variants — VitalReadingCard uses the smaller value directly to match Figma. If/when the rest of the card family migrates to 12 px, flip the Tailwind alias from radius-500 → radius-400.

---

---

## 6.19 Header

**Figma nodes:** Container `1-2466` · Apps Button `1-2469` · Tag `1-2473` · Calendar `1-2477` · Manual Button `193-3723` · Dropdown open state `185-7220`
**MIH Design Template:** `Urt0JAQES9364u2ud3XtpY`

The Header is a 64 px fixed top bar. Left zone: Brand Subdue Icon Button (40 px) + program name + inline breadcrumb. Centre: Brand Tag pill (location selector with dropdown). Right zone: calendar date, font-size switcher (ก ก ก), **Neutral Subdue Button** (คู่มือใช้งานระบบ).

**Page-title SubHeader density:** buttons in the row that contains the page
title are wrapped by `ButtonSizeAliasProvider size="sm"`. All canonical button
families therefore resolve to `dimension/size/700` (36 px), including buttons
nested inside compound header actions. Buttons outside SubHeader keep the size
requested by their caller.

---

### 6.19.1 Component Properties

| Property | Type | Values | Default |
|---|---|---|---|
| `system` | enum | `Default` · `Community` · `PHCIS` · `Pharmacy` · `School` · `Veterinary` | `Default` |
| `withBtn` | boolean | `true` · `false` | `false` |
| `showSelection` | boolean | `true` · `false` | `true` |

---

### 6.19.2 Dimension Tokens

| Element | Token | Value | CSS variable |
|---|---|---|---|
| Bar height | `dimension/size/1400` | 64 px | — |
| Horizontal padding | `dimension/space/600` | 24 px | — |
| Section gap | `dimension/space/800` | 32 px | — |
| Apps button size | `dimension/size/1000` | 40 px | — |
| Apps button radius | `dimension/radius/200` | 8 px | `--dim-radius-200` |
| Apps icon wrapper | 20 × 20 px (icon component) | — | — |
| Apps icon inner inset | 16.67 % all sides | ~3.3 px | — |
| Tag height | `dimension/size/800` | 40 px (Brand Tag Large) | — |
| Tag radius | `dimension/radius/full` | 9999 px | — |
| Tag padding H | `dimension/space/300` | 12 px | — |
| Tag gap | `dimension/space/150` | 6 px | — |
| Tag icon size | 16 × 16 px | — | — |
| Map-pin inset (tag) | `inset: 0 8.33%` | 0 px top/bot · 1.3 px sides | — |
| Chevron inset (tag) | `inset: 33.33% 20.83%` | — | — |
| Dropdown radius | `dimension/radius/500` | 16 px | — |
| Dropdown padding | `dimension/space/100` | 4 px | — |
| List item min-height | `dimension/size/1100` | 48 px | — |
| List item radius | `dimension/radius/100` | 6 px | — |
| List item padding | `dimension/space/200` / `dimension/space/300` | 8 px / 12 px | — |
| List item gap | `dimension/space/200` | 8 px | — |
| Map-pin inset (list) | `inset: 0 8.33%` | 0 px top/bot · 1.3 px sides | — |
| Right zone gap | `dimension/space/800` | 32 px | — |
| Calendar/Book icon | 16 × 16 px | — | — |
| Calendar inner inset | `inset: 4.17% 8.33%` | ~0.67 px top/bot · ~1.33 px sides | — |
| Manual button padding | `space/150` / `space/300` | 6 px top/bot · 12 px sides | — |
| Manual button radius | `dimension/radius/200` | 8 px | `--dim-radius-200` |
| Manual button gap | `dimension/space/150` | 6 px | — |
| Book inner inset (manual btn) | `inset: 4.17% 12.5%` | ~0.67 px top/bot · ~2 px sides | — |
| Font-size gap | `dimension/space/300` | 12 px | — |

---

### 6.19.3 Color Tokens

| Element | Token | Hex | CSS variable |
|---|---|---|---|
| Bar background | `surface/navigation/default` | `#FFFFFF` | `--surface-navigation-default` |
| Bottom border | `border/brandsecondary/quaternary` | `#EFF5EF` | `--border-brandsecondary-quaternary` |
| Drop shadow | `effect/brand-drop-shadow-bottom/100` → `PrimaryShadow/600` | `rgba(8,167,104,0.08)` | `--shadow-header` (aliases `--shadow-brand-drop-bottom-100`) |
| Apps button BG | `surface/brandprimarybutton/tertiary` | `#E2F3EB` | `--surface-brandprimarybutton-tertiary` |
| Program name | `text/navigation/title` | `#007549` | `--text-navigation-title` |
| Breadcrumb page links | `text/breadcrumb/pagename` | `#858C92` | — |
| Breadcrumb current | `text/breadcrumb/current` | `#363B3F` | — |
| **Tag — Default state** | | | |
| Tag BG | `surface/brandtag/default` | `#FFFFFF` | `--surface-brandtag-default` |
| Tag border | `border/brandtag/default` | `#E2E4E6` | `--border-brandtag-default` |
| Tag text | `text/brandtag/default` | `#4D5358` | `--text-brandtag-default` |
| **Tag — Hover / Open state** | | | |
| Tag BG | `surface/brandtag/default-hover` | `#F4FBF8` | `--surface-brandtag-hover` |
| Tag border | `border/brandtag/default-hover` | `#C5EDDA` | `--border-brandtag-hover` |
| Tag text | `text/brandtag/hover` | `#007549` | `--text-brandtag-hover` |
| Map-pin icon | `icon/brand/default` | `primary/700` | `filter:var(--icon-sidemenu-hover-filter)` | ✓ |
| **Dropdown** | | | |
| Dropdown BG | `surface/dropdown/default` | `#FFFFFF` | `--surface-dropdown` |
| Dropdown border | `border/dropdown/default` | `#EBEBEB` | `--border-dropdown` |
| Dropdown shadow | `color/shadow/100` · `color/shadow/050` | rgba(100,116,139,.15/.10) | `--shadow-dropdown` |
| **List item — Default** | | | |
| List item BG | `surface/list/default` | `#FFFFFF` | `--surface-list-default` |
| List item text | `text/list/default` | `#4D5358` | `--text-list-default` |
| **List item — Hover / Active** | | | |
| List item BG | `surface/list/hover` | `#F4FBF8` | `--surface-list-hover` |
| List item text | `text/list/hover` | `#007549` | `--text-list-hover` |
| List map-pin (hover/active) | `icon/list/primary-selected-hover` | `primary/700` | `filter:var(--icon-sidemenu-hover-filter)` | ✓ |
| List map-pin (default) | `icon/list/secondary-default` | neutral-500 (fixed) | `filter:var(--icon-list-secondary-filter)` | — |
| **Right zone** | | | |
| Date text | `text/content/secondary` | `#636B72` | `--text-content-secondary` |
| Font-size hover BG | `surface/brandprimary/quaternary` | `#F4FBF8` | `--surface-sidemenu-active` |
| Font-size active text | `text/navigation/title` | `#007549` | `--text-navigation-title` |
| **Manual Button (node 193:3723)** | | | |
| Button border | `border/neutralbutton/tertiary` | `#E2E4E6` | `--border-neutralbutton-tertiary` |
| Button text | `text/neutralbutton/default` | `#4D5358` | `--text-neutralbutton-default` |
| Button hover BG | `surface/neutralbutton/tertiary-hover` | `#F4F5F5` | `--surface-neutralbutton-tertiary-hover` |
| Button radius | `dimension/radius/200` | 8 px | `--dim-radius-200` |

---

### 6.19.4 Typography

| Element | Style | Size | Weight |
|---|---|---|---|
| Program name | `h6` · heading-content Bold | `typograpphy/size/base` 16 px | 700 |
| Breadcrumb links | `body4` · body Regular | `typograpphy/size/sm` 14 px | 400 |
| Tag text | `label1` · body Medium | 14 px | 500 |
| List item text | `body4` · body Regular | `typograpphy/size/sm` 14 px | 400 |
| Date text | `body4` · body Regular | `typograpphy/size/sm` 14 px | 400 |
| Manual button label | `button sm` · Sarabun Medium | `typograpphy/size/sm` 14 px | 500 |
| Font-size label | `caption1` · body Regular | `typograpphy/size/xs` 12 px | 400 |
| Font-size ก (S/M/L) | — | 14 / 16 / 18 px | — |

---

### 6.19.5 Figma Asset UUIDs

> Assets expire 7 days after fetch — re-fetch from the source node when stale.
> Icon color change between states is handled via **CSS filter** (not separate image assets).
> **Map-pin icons** must use the MIH DS Foundation asset (node `2056:4442;381:12859`, UUID `5086c8d9`) — not custom SVG. Both the Tag icon and Dropdown list icons use the same asset; tint is applied via CSS filter.

| Element | State | Figma node | UUID | Color |
|---|---|---|---|---|
| Apps icon (apps grid) | — | `I1:2469;2024:4372;2566:10943` (DS Template) | `888ea972-fd03-4a81-bf28-9aec26423463` | — |
| **Tag — leading icon (map-pin)** | All states | `2056:4442;381:12859` (**MIH DS Foundation**) | `5086c8d9-c55e-405d-b5fe-6bc132d2b500` | `filter:var(--icon-sidemenu-hover-filter)` = `primary/700`, brand-responsive |
| **Tag — chevron-down (closed)** | Default | `I2056:4441;2056:4595;328:4566` (DS Foundation) | `a6eded4b-1ab5-4d9c-95e7-9ab176964138` | gray/neutral |
| Tag — chevron-down (closed) | Hover | same UUID + CSS filter | `2a928811-e727-4608-abb5-ef31d21dc76e` ref | emerald-700 |
| **Tag — chevron-up (open)** | Open | `3064:6140;328:4637` (DS Foundation) | `da9e8e6b-e707-4da2-9c75-2030a588fe47` | emerald-700 |
| Tag — chevron-down (selected) | Selected | `2056:4604;328:4566` (DS Foundation) | `97c2d72f-9c0c-4c88-b132-e1af60890c15` ref | emerald-700 |
| **Dropdown list — map-pin** | Default | `2056:4442;381:12859` (**MIH DS Foundation**) | `5086c8d9-c55e-405d-b5fe-6bc132d2b500` | `filter:var(--icon-list-secondary-filter)` ≈ `#858c92` (neutral) |
| Dropdown list — map-pin | Hover/Active | same asset + CSS filter | — | `filter:var(--icon-sidemenu-hover-filter)` = `primary/700`, brand-responsive |
| Calendar icon | — | `I1:2477;328:4458` (DS Template) | `62838aea-c875-4565-a9d4-c3adde4cdf5e` | gray/neutral |
| **Manual button book icon** | — | `I193:3723;2037:3290;328:4454` (DS Template node 193:3723) | `071a8221-ffe3-49eb-8abd-99b8701b4fde` | gray/neutral |

**Chevron implementation — single asset + CSS `rotate(180deg)` (spring easing):**

The chevron uses **one gray-down asset** rotated 180° via CSS transform on open state. Spring easing (`cubic-bezier(.34,1.56,.64,1)`) gives a snappy, physical feel.

```css
/* Single asset: gray chevron-down — CSS rotate handles direction */
.hdr-tag-icon-chevron {
  transition: transform .28s cubic-bezier(.34,1.56,.64,1),   /* spring */
              filter .18s ease;
}
/* Open: rotate 180° → chevron points up */
.hdr-tag.open .hdr-tag-icon-chevron { transform: rotate(180deg); }
/* Hover (not open): tint to brand primary/700 */
.hdr-tag:hover:not(.open) .hdr-tag-icon-chevron {
  filter: var(--icon-sidemenu-hover-filter);
}
/* Open: brand primary/700 tint */
.hdr-tag.open .hdr-tag-icon-chevron {
  filter: var(--icon-sidemenu-hover-filter);
}
```

> **Brand-responsive:** `--icon-sidemenu-hover-filter` is updated by `setBrandTheme()` via the `SIDEMENU_FILTERS` lookup table. It holds the precomputed CSS `filter` string for each brand's `primary/700` colour. Never hardcode the filter string directly.

**Map-pin filter (hover + open):**
```css
.hdr-tag:hover .hdr-tag-icon-pin img,
.hdr-tag.open  .hdr-tag-icon-pin img {
  filter: var(--icon-sidemenu-hover-filter);   /* brand primary/700 — updated by setBrandTheme() */
}
```

---

### 6.19.6 CSS Implementation (prefix: `hdr-`)

```css
/* ── Bar container ── */
.hdr-wrap {
  height: 64px;                                      /* dimension/size/1400 */
  background: var(--surface-navigation-default, #FFFFFF);
  border-bottom: 1px solid var(--border-brandsecondary-quaternary, #EFF5EF);
  box-shadow: 0 1px 2px var(--primaryshadow-600, rgba(8,167,104,.08));
  padding: 0 24px;                                   /* dimension/space/600 */
  display: flex; align-items: center;
  gap: 32px;                                         /* dimension/space/800 */
}

/* ── Brand Subdue Icon Button (node 1:2469) ── */
.hdr-apps-btn {
  width: 40px; height: 40px;                         /* dimension/size/1000 */
  border-radius: var(--dim-radius-200, 8px);         /* dimension/radius/200 */
  background: var(--surface-brandprimarybutton-tertiary, #E2F3EB);
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0;
}
.hdr-apps-icon-wrap { position: relative; width: 20px; height: 20px; }
.hdr-apps-icon-inner { position: absolute; inset: 16.67%; }
.hdr-apps-icon-inner img { position: absolute; inset: 0; width: 100%; height: 100%;
                          filter: var(--icon-sidemenu-hover-filter); /* brand primary/700 */ }

/* ── Program name + breadcrumb ── */
.hdr-title {
  font-family: 'Sarabun', sans-serif;
  font-size: 16px; font-weight: 700; line-height: 1.5; /* h6 */
  color: var(--text-navigation-title, #007549);
}

/* ── Location Tag (Brand Tag Large, node 1:2473) ── */
.hdr-tag {
  display: inline-flex; align-items: center; justify-content: center;
  gap: 6px;                                          /* dimension/space/150 */
  height: 40px;                                      /* dimension/size/800 */
  padding: 4px 12px;
  border-radius: 999px;
  background: var(--surface-brandtag-default, #FFFFFF);
  border: 1px solid var(--border-brandtag-default, #E2E4E6);
  cursor: pointer;
  transition: background .18s ease, border-color .18s ease, box-shadow .18s ease, transform .1s ease;
}
.hdr-tag:hover, .hdr-tag.open {
  background: var(--surface-brandtag-hover, #F4FBF8);
  border-color: var(--border-brandtag-hover, #C5EDDA);
  box-shadow: 0 2px 10px rgba(0,117,73,.10);
}
.hdr-tag:active { transform: scale(0.96); box-shadow: none; }

/* Icon wrapper 16×16 */
.hdr-tag-icon { position: relative; width: 16px; height: 16px; flex-shrink: 0; }
/* Map-pin — inset: 0 8.33% (node I1:2473;2056:4442) */
.hdr-tag-icon-pin  { position: absolute; top: 0; bottom: 0; left: 8.33%; right: 8.33%; }
/* Chevron — single gray-down asset, CSS rotate(180°) on open */
.hdr-tag-icon-chevron {
  position: absolute;
  top: 33.33%; bottom: 33.33%; left: 20.83%; right: 20.83%;
  transition: transform .28s cubic-bezier(.34,1.56,.64,1),  /* spring easing */
              filter .18s ease;
}
.hdr-tag-icon-pin img,
.hdr-tag-icon-chevron img { position: absolute; inset: 0; width: 100%; height: 100%; display: block; }
/* Open: rotate 180° with spring overshoot */
.hdr-tag.open .hdr-tag-icon-chevron { transform: rotate(180deg); }
/* Hover / Open: tint to brand primary/700 — brand-responsive via --icon-sidemenu-hover-filter */
.hdr-tag:hover:not(.open) .hdr-tag-icon-chevron,
.hdr-tag.open .hdr-tag-icon-chevron {
  filter: var(--icon-sidemenu-hover-filter);
}
/* Tag text */
.hdr-tag-text {
  font-family: 'Sarabun', sans-serif;
  font-size: 14px; font-weight: 500; line-height: 1.5; /* label1 */
  color: var(--text-brandtag-default, #4D5358); transition: color .12s;
}
.hdr-tag:hover .hdr-tag-text,
.hdr-tag.open  .hdr-tag-text { color: var(--text-brandtag-hover, #007549); }
/* Map-pin tint on hover/open — brand primary/700 */
.hdr-tag:hover .hdr-tag-icon-pin img,
.hdr-tag.open  .hdr-tag-icon-pin img {
  filter: var(--icon-sidemenu-hover-filter);
}

/* ── Dropdown (node 185-7220) ── */
.hdr-dropdown {
  position: absolute; top: calc(100% + 4px); left: 0; width: 100%;
  background: var(--surface-dropdown, #FFFFFF);
  border: 1px solid var(--border-dropdown, #EBEBEB);
  border-radius: 16px; padding: 4px;
  box-shadow: 0 16px 32px rgba(100,116,139,.15), 0 4px 4px rgba(100,116,139,.10);
  display: none; flex-direction: column; z-index: 200;
}
.hdr-dropdown.open { display: flex; }
/* List item states — surface/list/default · surface/list/hover (#F4FBF8 = emerald-50) */
.hdr-dd-item {
  display: flex; align-items: center; gap: 8px; min-height: 48px;
  padding: 8px 12px; border-radius: 6px;
  background: var(--surface-list-default, #FFFFFF);
  cursor: pointer; transition: background .12s, color .12s;
}
.hdr-dd-item:hover  { background: var(--surface-list-hover, #F4FBF8); }
.hdr-dd-item.active { background: var(--surface-list-hover, #F4FBF8); }
.hdr-dd-icon-wrap { position: relative; width: 16px; height: 16px; flex-shrink: 0; }
.hdr-dd-icon-inner { position: absolute; top: 0; bottom: 0; left: 8.33%; right: 8.33%; }
.hdr-dd-icon-inner img { position: absolute; inset: 0; width: 100%; height: 100%; transition: filter .2s ease; }
.hdr-dd-item span {
  font-size: 14px; font-weight: 400; line-height: 1.5;
  color: var(--text-list-default, #4D5358); flex: 1; transition: color .12s;
}
.hdr-dd-item:hover span  { color: var(--text-list-hover, #007549); }
.hdr-dd-item.active span { color: var(--text-list-hover, #007549); font-weight: 500; }
.hdr-dd-item:hover  .hdr-dd-icon-inner img,
.hdr-dd-item.active .hdr-dd-icon-inner img {
  filter: invert(30%) sepia(70%) saturate(600%) hue-rotate(118deg) brightness(85%) contrast(105%);
}

/* ── Right zone ── */
.hdr-right { display: flex; align-items: center; gap: 32px; height: 100%; }
.hdr-info-item { display: flex; align-items: center; gap: 8px; flex-shrink: 0; }
.hdr-info-icon { position: relative; width: 16px; height: 16px; overflow: hidden; }
.hdr-info-icon-cal { position: absolute; top: 4.17%; bottom: 4.17%; left: 8.33%; right: 8.33%; }
.hdr-info-icon-cal img { position: absolute; inset: 0; width: 100%; height: 100%; }
.hdr-info-item span { font-size: 14px; font-weight: 400; line-height: 1.5; color: var(--text-content-secondary, #636B72); white-space: nowrap; }

/* ── Neutral Subdue Button — node 193:3723 ──
   border/neutralbutton/tertiary · text/neutralbutton/default · dimension/radius/200 */
.hdr-manual-btn {
  display: inline-flex; align-items: center; justify-content: center;
  gap: 6px;                                          /* dimension/space/150 */
  padding: 6px 12px;                                 /* space/150 · space/300 */
  border: 1px solid var(--border-neutralbutton-tertiary, #E2E4E6);
  border-radius: var(--dim-radius-200, 8px);         /* dimension/radius/200 */
  background: transparent;
  cursor: pointer; white-space: nowrap; flex-shrink: 0;
  transition: background .15s ease, border-color .15s ease;
}
.hdr-manual-btn:hover { background: var(--surface-neutralbutton-tertiary-hover, #F4F5F5); }
.hdr-manual-btn:active { transform: scale(0.98); }
/* Book icon — inset: 4.17% 12.5% (node I193:3723;2037:3290;328:4454) */
.hdr-manual-icon { position: relative; width: 16px; height: 16px; flex-shrink: 0; overflow: hidden; }
.hdr-manual-icon-inner { position: absolute; top: 4.17%; bottom: 4.17%; left: 12.5%; right: 12.5%; }
.hdr-manual-icon-inner img { position: absolute; inset: 0; width: 100%; height: 100%; display: block; }
/* Button label — button sm: Sarabun Medium 14px */
.hdr-manual-label {
  font-family: 'Sarabun', sans-serif;
  font-size: 14px; font-weight: 500; line-height: 1.5; /* typography/size/sm · weight/medium */
  color: var(--text-neutralbutton-default, #4D5358); /* text/neutralbutton/default → neutral-700 */
}

/* ── Font-size switcher ── */
.hdr-fontsize { display: flex; align-items: center; gap: 12px; }
.hdr-fontsize-label { font-size: 12px; color: var(--text-content-secondary, #636B72); }
.hdr-fs-btn { cursor: pointer; color: var(--text-content-secondary, #636B72); border-radius: 4px; padding: 2px 4px; transition: color .12s, background .12s; }
.hdr-fs-btn:hover  { color: var(--text-navigation-title, #007549); background: var(--surface-list-hover, #F4FBF8); }
.hdr-fs-btn.active { color: var(--text-navigation-title, #007549); font-weight: 700; }
.hdr-fs-sm { font-size: 14px; } .hdr-fs-md { font-size: 16px; } .hdr-fs-lg { font-size: 18px; }
```

---

### 6.19.7 Token Alias Chain

```
surface/navigation/default          → neutral/white   → #FFFFFF
border/brandsecondary/quaternary    → emerald/50      → #EFF5EF
surface/brandprimarybutton/tertiary → emerald/100     → #E2F3EB
text/navigation/title               → emerald-700     → #007549
surface/brandtag/default            → neutral/white   → #FFFFFF
border/brandtag/default             → neutral/200     → #E2E4E6
text/brandtag/default               → neutral/700     → #4D5358
surface/brandtag/default-hover      → emerald/50      → #F4FBF8
border/brandtag/default-hover       → emerald/200     → #C5EDDA
text/brandtag/hover                 → emerald-700     → #007549
surface/dropdown/default            → neutral/white   → #FFFFFF
border/dropdown/default             → neutral/100     → #EBEBEB
surface/list/default                → neutral/white   → #FFFFFF
surface/list/hover                  → emerald/50      → #F4FBF8   ← NOT emerald-100
text/list/default                   → neutral/700     → #4D5358
text/list/hover                     → emerald-700     → #007549
text/content/secondary              → neutral/600     → #636B72
border/neutralbutton/tertiary       → neutral/200     → #E2E4E6
text/neutralbutton/default          → neutral/700     → #4D5358
surface/neutralbutton/tertiary-hover → neutral/50     → #F4F5F5
surface/brandprimary/quaternary     → emerald/50      → #F4FBF8   ← tab underline hover bg
```

> **Note on emerald-50 vs emerald-100:** `surface/list/hover` and `surface/brandprimary/quaternary` both resolve to **#F4FBF8** (emerald-50 = สีเขียวอ่อน). This is distinct from `#E2F3EB` (emerald-100) used for button/card backgrounds. Do not interchange these values.

---

## 6.20 Side Menu + Sub Menu

**Figma nodes:** Side Menu `1-2253` · Menu Item States `1-2254` · Menu Item `1-2255` · Hover State `1-2260` · Collapsed `1-2390` · PHCIS Icons `1-2447` · Notification `1-2363` · Sub Menu A `6-1491` · [Design Template — SideMenu shell `1-2447`](https://www.figma.com/design/Urt0JAQES9364u2ud3XtpY/Design-Template?node-id=1-2447)

The Side Menu is a collapsible vertical navigation panel (open: 256 px / closed: 64 px). It uses a hover-to-expand interaction — the sidebar widens from 64 px to 256 px on hover, or can be locked open via a toggle button. Icon colour changes are implemented via CSS filter using the `icon/sidemenu/hover` token.

### Dimensions — Side Menu

| Property | Token | Value |
|---|---|---|
| Open width | `dimension/size/6400` | 256 px |
| Closed width | `dimension/size/1600` | 64 px |
| Logo area height | `dimension/size/1400` | 56 px |
| Item min-height | — | 48 px |
| Item padding H (closed) | `dimension/space/600` | 24 px |
| Item gap (closed) | `dimension/space/300` | 0 px (collapsed, true-centers icon) |
| Item gap (open) | `dimension/space/300` | 12 px |
| Item border-radius | `dimension/radius/100` | 6 px |
| Active left border | `dimension/stroke/400` | 4 px |
| Active padding-left | `dimension/space/600 − stroke/400` | 20 px |
| Icon size | — | 16 × 16 px |
| Avatar size | `dimension/radius/full` | 40 × 40 px |
| User area height | — | 56 px |
| User area padding | `dimension/space/300` | 12 px |
| Bottom gap | `dimension/space/200` | 8 px |
| Top section gap | `dimension/space/150` | 6 px |

### Dimensions — Sub Menu

| Property | Token | Value |
|---|---|---|
| Panel width | `dimension/size/7000` | 280 px |
| New button height | `dimension/size/900` | 36 px |
| Sub-item height | `dimension/size/800` | 32 px |

### Color Tokens — Side Menu

| Element | Figma token | CSS var | Primitive | Hex | Brand? |
|---|---|---|---|---|---|
| Background | `surface/navigation/default` | `--wh` | `neutral/0` | `#FFFFFF` | — |
| Right border | `border/brandprimary/quaternary` | `--border-sidemenu-shell` → `--border-brandPrimary-quaternary` | `primary/100` | `var(--brand-p100)` | ✓ |
| Drop shadow | `effect/Brand Drop Shadow Top/200` | `--shadow-sidemenu` → `--shadow-brand-top-200` | `PrimaryShadow/600,601` (brand-p600 @ 8/10%) | `0 -1px 4px ...; 0 -1px 8px ...` | ✓ |
| Logo / nav divider (horizontal) | `border/neutral/quaternary` | `--border-neutral-quaternary` | `neutral/100` | `#F4F5F5` | — |
| Group label | `text/sidemenu/group` | `--text-sidemenu-tertiary` | `neutral/500` | `#858C92` | — |
| Item default text | `text/sidemenu/default` | `--text-sidemenu-default` | `neutral/700` | `#4D5358` | — |
| Item default icon | `icon/sidemenu/default` | (via CSS filter default) | `neutral/700` | `#4D5358` | — |
| Item hover text | `text/sidemenu/hover` | `--text-sidemenu-hover` | **primary/700** | `var(--em700)` | ✓ |
| Item hover icon | `icon/sidemenu/hover` | `--icon-sidemenu-hover-filter` | **primary/700** | `var(--em700)` | ✓ |
| Item hover / active BG | `surface/brandprimary/quaternary` | `--surface-sidemenu-active` | **primary/50** | `var(--em50)` | ✓ |
| Item selected text | `text/sidemenu/selected` | `--text-sidemenu-selected` | **primary/700** | `var(--em700)` | ✓ |
| Item selected icon | `icon/sidemenu/selected` | `--icon-sidemenu-hover-filter` | **primary/700** | `var(--em700)` | ✓ |
| Active left border | `surface/brandprimary/default` | `--border-sidemenu-active` | **primary/700** | `var(--em700)` | ✓ |
| Disabled text | `text/sidemenu/disabled` | `--text-sidemenu-disabled` | `neutral/400` | `#ADB2B7` | — |
| **Notification badge** | `surface/notification/default` | `--surface-notification-default` | `pink/700` | `#db2777` | — |
| Badge text | `text/danger/on-default` | — | `neutral/0` | `#FFFFFF` | — |
| User name | `text/sidemenu/default` | `--text-sidemenu-default` | `neutral/700` | `#4D5358` | — |
| User dept | `text/content/tertiary` | `--text-sidemenu-tertiary` | `neutral/500` | `#858C92` | — |
| User area top border | `border/neutral/quaternary` | `--border-neutral-quaternary` | `neutral/100` | `#F4F5F5` | — |

> **Brand ✓ vars** (`--border-sidemenu-shell`, `--shadow-sidemenu`, `--text-sidemenu-hover`, `--text-sidemenu-selected`, `--border-sidemenu-active`, `--surface-sidemenu-active`, `--icon-sidemenu-hover-filter`) อัปเดตอัตโนมัติใน `setBrandTheme()` — icon ใช้ precomputed CSS filter string ต่อ brand แทน `color` เนื่องจากเป็น `<img>` asset จาก Figma CDN. Shadow tint resolves via `--PrimaryShadow-600/601` (`rgb(var(--brand-p600-rgb) / 0.08|0.10)`) so it follows the active brand without per-mode overrides.

> **Notification badge placement:** Badge appears **only** on the แจ้งเตือน (notification bell) bottom item — `surface/notification/default = #db2777`. It does **not** appear on ตั้งค่าระบบ or any other item.

### Color Tokens — Sub Menu

| Element | Token | Alias | Hex |
|---|---|---|---|
| Panel background | `surface/neutral/quaternary` | `neutral/50` | `#F9FBFB` |
| Card background | `surface/navigation/default` | `neutral/0` | `#FFFFFF` |
| Card border | `border/brandsecondary/quaternary` | `emerald/50` | `#EFF5EF` |
| New button BG | `surface/brandPrimaryButton/default` | `brand-p700` | `#007549` |
| New button text | `text/brandPrimaryButton/default` | `neutral/0` | `#FFFFFF` |
| Sub-item default | `text/sidemenu/default` | `neutral/700` | `#4D5358` |
| Sub-item selected BG | `surface/brandsecondary/quaternary` | `emerald/25` | `#FAFCFA` |
| Sub-item selected text | `text/sidemenu/selected` | `brand-p700` | `#007549` |
| Collapse button BG | `surface/brandPrimaryButton/default` | `brand-p700` | `#007549` |

### Icon Assets — PHCIS System (node 1:2447)

Each menu item uses a dedicated icon from the MIH Design System. Icons are neutral/dark by default; hover and active states apply `icon/sidemenu/hover` (#007549) via CSS filter.

| Menu item (Thai) | Icon name | Inset |
|---|---|---|
| ภาพรวม | `bar_chart_4_bars` | `12.5% 8.33%` |
| ผู้ป่วย | `face` | `8.33%` |
| นัดหมาย | `assignment_add` | `4.17% 4.17% 4.17% 12.5%` |
| ประวัติการรักษา | `clipboard-list` | `4.2% 12.65% 4.35% 12.65%` |
| ยาและเวชภัณฑ์ | `pill` | `12.5%` |
| รายงาน | `history` | `12.5%` |
| ห้องตรวจ | `Stethoscope` | `4.2% 4.2% 8.42% 4.3%` |
| การเงิน | `Baht` | `8.48% 23.52% 6.52% 26.48%` |
| บุคลากร | `calendar_clock` | `8.33% 4.17% 4.17% 12.5%` |
| ตั้งค่าระบบ | `Users` | `8.33% 0` |
| เพิ่มเติม | `more_horizontal` | `41.67% 12.5%` |
| แจ้งเตือน (bell) | `Notification` | `4.17% 8.33% 4.18% 8.33%` |

### Item State Summary

| State | BG (CSS var) | Text/Icon (CSS var) | Left border |
|---|---|---|---|
| Default | transparent | `--text-sidemenu-default` = `#4D5358` (fixed) | — |
| Hover | `--surface-sidemenu-active` = `var(--em50)` ✓ | `--text-sidemenu-hover` = `var(--em700)` ✓ · icon via `--icon-sidemenu-hover-filter` ✓ | — |
| Active (Selected) | `--surface-sidemenu-active` = `var(--em50)` ✓ | `--text-sidemenu-selected` = `var(--em700)` ✓ · icon via `--icon-sidemenu-hover-filter` ✓ | 4px `--border-sidemenu-active` = `var(--em700)` ✓ |
| Disabled | transparent | `--text-sidemenu-disabled` = `#ADB2B7` (fixed) · icon opacity 35% | — |

### Collapsed State Icon Centering (node 1:2390)

When collapsed (width 64 px), each item uses `gap: 0` and `justify-content: center`. Padding remains `8px 24px` on both sides, so the 16 px icon sits in the exact centre of the 64 px rail. The notification badge must use `padding: 0` (not `2px 8px`) when collapsed — otherwise the `box-sizing: content-box` model adds 16 px of padding width and shifts the flex centre off.

### Active Item Anatomy

```
Collapsed (64 px rail):
┌──────────────────────┐
│ ▌ [Icon↗]            │  gap=0, justify-center, padding-left: 20px (24−4px border)
└──────────────────────┘

Expanded (256 px):
┌──────────────────────────────────────┐
│ ▌   [Icon]   Item Label         [›]  │  ← 4px left border (border/sidemenu/active)
└──────────────────────────────────────┘
  BG:     surface/brandprimary/quaternary (#F4FBF8)
  Text:   text/sidemenu/selected (#007549)
  Icon:   icon/sidemenu/selected → CSS filter to #007549
  Border: border/sidemenu/active (brand-p700) · width: dimension/stroke/400 = 4px
```

### Component Properties

| Property | Type | Values | Default |
|---|---|---|---|
| `system` | enum | `Default` · `Community` · `PHCIS` · `Pharmacy` · `School` · `Veterinary` | `Default` |
| `open` | enum | `Yes` · `No` | `No` |
| `showMenu1`…`showMenu11` | boolean | `true` · `false` | `true` |

Sub Menu A properties:

| Property | Type | Values | Default |
|---|---|---|---|
| `open` | enum | `Yes` · `No` | `Yes` |
| `withButton` | boolean | `true` · `false` | `true` |

### CSS Implementation

```css
/* ── Tokens ── */
:root {
  --surface-navigation-default: #FFFFFF;         /* surface/navigation/default */
  --border-neutral-quat:        #F4F5F5;         /* border/neutral/quaternary */
  --surface-sidemenu-active:    #F4FBF8;         /* surface/brandprimary/quaternary */
  --border-sidemenu-active:     #007549;         /* border/sidemenu/active → emerald-700 */
  --text-sidemenu-default:      #4D5358;         /* text/sidemenu/default → neutral-700 */
  --text-sidemenu-hover:        #007549;         /* text/sidemenu/hover → emerald-700 */
  --text-sidemenu-selected:     #007549;         /* text/sidemenu/selected → emerald-700 */
  --text-sidemenu-disabled:     #ADB2B7;         /* text/sidemenu/disabled → neutral-400 */
  --text-sidemenu-tertiary:     #858C92;         /* text/content/tertiary → neutral-500 */
  --surface-notification-default: #db2777;       /* surface/notification/default → pink-700 */

  /* icon/sidemenu/hover → surface/brandprimary/default → emerald-700 → #007549 */
  --icon-sidemenu-hover-filter:
    brightness(0) invert(27%) sepia(100%) saturate(600%)
    hue-rotate(148deg) brightness(89%) contrast(99%);
}

/* ── Collapsed/Expanded widths ── */
.sm-wrap { width: 64px; transition: width 220ms; overflow: hidden; }
.sm-wrap:hover, .sm-wrap.open { width: 256px; }

/* ── Menu item — collapsed: icon true-centered ── */
.sm-item {
  display: flex; align-items: center;
  min-height: 48px;
  padding: 8px 24px;              /* dimension/space/600 */
  gap: 0;                         /* collapsed: gap=0 → icon stays centered */
  justify-content: center;
}
.sm-wrap:hover .sm-item,
.sm-wrap.open  .sm-item { justify-content: flex-start; gap: 12px; } /* dimension/space/300 */

/* ── Hover state: text + icon → icon/sidemenu/hover = #007549 ── */
.sm-item:hover:not(.sm-disabled) { background: var(--surface-sidemenu-active); }
.sm-item:hover:not(.sm-disabled) .sm-label { color: var(--text-sidemenu-hover); }
.sm-item:hover:not(.sm-disabled) .sm-item-icon img { filter: var(--icon-sidemenu-hover-filter); }

/* ── Active state ── */
.sm-item.sm-active {
  background: var(--surface-sidemenu-active);
  border-left: 4px solid var(--border-sidemenu-active); /* dimension/stroke/400 */
  border-radius: 0;
  padding-left: 20px;             /* 24px − 4px border = keeps icon centered */
}
.sm-item.sm-active .sm-label    { color: var(--text-sidemenu-selected); }
.sm-item.sm-active .sm-item-icon img { filter: var(--icon-sidemenu-hover-filter); }

/* ── Notification badge (แจ้งเตือน only) ── */
.sm-noti-badge {
  background: var(--surface-notification-default, #db2777); /* surface/notification/default */
  color: #FFFFFF;
  font-size: 12px; font-weight: 500;       /* typography/size/xs */
  padding: 0;                              /* collapsed: 0 prevents box-model width shift */
  border-radius: 24px;                    /* dimension/radius/600 */
  opacity: 0; max-width: 0; overflow: hidden;
  transition: opacity 160ms, max-width 220ms, padding 220ms;
}
.sm-wrap:hover .sm-noti-badge,
.sm-wrap.open  .sm-noti-badge {
  padding: 2px 8px;                       /* dimension/space/050 · space/200 — restored when open */
  opacity: 1; max-width: 60px;
}
```

---

## 6.21 Breadcrumb

**Figma node:** `14-3191` · MIH Design Template

The Breadcrumb shows the user's location in the navigation hierarchy. It starts with a home icon, followed by "/" separators and page links, ending with the current page name in darker text.

### Dimensions

| Property | Token | Value |
|---|---|---|
| Row height | `dimension/size/900` | 36 px |
| Home icon size | `dimension/size/500` | 20 px |
| Icon/text gap | `dimension/space/100` | 4 px |
| Separator gap | `dimension/space/100` | 4 px |
| Font size | — | 13 px |

### Color Tokens

| Element | Token | Alias | Hex |
|---|---|---|---|
| Home icon | `icon/breadcrumb/home` | `neutral/500` | `#858C92` |
| Page link | `text/breadcrumb/pagename` | `neutral/500` | `#858C92` |
| Current page | `text/breadcrumb/current` | `neutral/900` | `#363B3F` |
| Separator "/" | `text/breadcrumb/separator` | `neutral/300` | `#C4C9CE` |
| Ellipsis pill BG | `surface/brandprimary/quaternary` | `emerald/50` | `#F4FBF8` |
| Ellipsis border | `border/brandsecondary/quaternary` | `emerald/50` | `#EFF5EF` |
| Page link :hover | `text/link/default` | `brand-p700` | `#007549` |
| Home :hover | `icon/breadcrumb/hover` | `brand-p700` | `#007549` |

### Component Properties

| Property | Type | Values | Description |
|---|---|---|---|
| `levels` | enum | `2` · `3` · `4` · `Truncated` | Number of crumb levels; `Truncated` collapses middle levels to `…` |

### Level Examples

| Value | Output |
|---|---|
| `2` | 🏠 / บันทึกผลการตรวจ |
| `3` | 🏠 / บริการประชาชน / บันทึกผลการตรวจ |
| `4` | 🏠 / บริการประชาชน / ข้อมูลสุขภาพ / บันทึกผลการตรวจ |
| `Truncated` | 🏠 / … / ข้อมูลสุขภาพ / บันทึกผลการตรวจ |

### CSS Implementation

```css
.bc-wrap { display: flex; align-items: center; gap: 4px; }

.bc-link {
  font-size: 13px;
  color: var(--text-breadcrumb-pagename);   /* neutral/500 → #858C92 */
}
.bc-link:hover { color: var(--brand-p700); }

.bc-cur {
  font-size: 13px; font-weight: 600;
  color: var(--text-breadcrumb-current);    /* neutral/900 → #363B3F */
}

.bc-sep {
  font-size: 13px;
  color: #C4C9CE;                           /* text/breadcrumb/separator */
}

.bc-home { color: var(--icon-breadcrumb-home); }  /* neutral/500 */

.bc-ellipsis {
  background: var(--surface-brandprimary-quaternary);  /* emerald/50 */
  border: 1px solid var(--border-brandsecondary-quaternary);
  border-radius: 6px; padding: 1px 8px;
}
```

---

## 6.22 Footer

**Figma node:** `1-2629` · Design-Template · `Urt0JAQES9364u2ud3XtpY`

Footer แสดงที่ท้ายหน้าทุกหน้า รองรับ 3 variants: **with BTN** (indicator + action buttons), **with Pagination** (รายการ + ปุ่ม page), **No** (copyright bar เดี่ยว) ทุก token alias จาก MIH Design System — prefix `ftr-`

### 6.22.1 Component Properties (Figma)

| Property | Type | Values | Default |
|---|---|---|---|
| `type` | enum | `with BTN` · `with Pagination` · `No` | `with BTN` |
| `showAutoSave` | boolean | `true` · `false` | `true` |
| `showButton1` | boolean | `true` · `false` | `true` |
| `showButton2` | boolean | `true` · `false` | `true` |
| `showButton3` | boolean | `true` · `false` | `true` |

> Button1 = Fill (ยืนยันและส่ง), Button2 = Outline (บันทึกร่าง), Button3 = Outline (ยกเลิก). ลำดับจากขวาไปซ้าย

### 6.22.2 Variants Behavior

| `type` | Content | Notes |
|---|---|---|
| `with BTN` | Auto-save dot + text (ซ้าย) + 2 outline + 1 fill button (ขวา) | Form pages |
| `with Pagination` | Record count (ซ้าย) + pagination component (ขวา) | List/table pages |
| `No` | Copyright bar เดี่ยว ไม่มี top section | Minimal pages |

### 6.22.3 Color Tokens

| Element | Token Path | Primitive | Hex |
|---|---|---|---|
| Footer background | `surface/navigation/default` | neutral-0 | `#FFFFFF` |
| Top border | `border/brandsecondary/quaternary` | emerald-50 | `#EFF5EF` |
| Row separator | `border/neutral/quaternary` | neutral-100 | `#F4F5F5` |
| Auto-save dot | `surface/positive/secondary` | teal-200 | `#A2EDDE` |
| Auto-save / info text | `text/content/secondary` | neutral-600 | `#636B72` |
| Copyright / link text | `text/content/secondary` | neutral-600 | `#636B72` |
| Fill button background | `surface/brandprimarybutton/default` | emerald-700 | `#007549` |
| Fill button text | `text/brandprimarybutton/on-brand` | neutral-0 | `#FFFFFF` |
| Fill button shadow | `primaryShadow/600` + `primaryShadow/601` | — | `rgba(8,167,104,.08–.10)` |
| Outline button border | `border/brandprimarybutton/tertiary` | emerald-200 | `#C5EDDA` |
| Outline button text | `text/brandprimarybutton/default` | emerald-700 | `#007549` |
| Outline hover bg | `surface/brandtag/default-hover` | emerald-50 | `#F4FBF8` |
| Pagination button active bg | `surface/pagination/hover` | neutral-100 | `#F4F5F5` |
| Pagination text | `text/pagination/default` | neutral-700 | `#4D5358` |

### 6.22.4 Dimension Tokens

| Property | Token | Value |
|---|---|---|
| Copyright bar height | `dimension/size/1200` | 56 px |
| Button height | `dimension/size/900` | 44 px |
| Button corner radius | `dimension/radius/200` | 8 px |
| Button horizontal padding | `dimension/space/400` | 16 px |
| Footer horizontal padding | `dimension/space/600` | 24 px |
| Gap between action buttons | `dimension/space/300` | 12 px |
| Gap (auto-save dot ↔ text) | `dimension/space/200` | 8 px |
| Auto-save dot size | — | 8 px (border-radius 4px) |
| All border widths | `dimension/stroke/100` | 1 px |
| Pagination button size | — | 36 × 36 px |
| Pagination button radius | `dimension/radius/200` | 8 px |
| Pagination gap | `dimension/space/100` | 4 px |

### 6.22.5 CSS Implementation (prefix: `ftr-`)

```css
/* Wrapper */
.ftr-wrap {
  background: var(--surface-footer-bg, #FFFFFF);   /* surface/navigation/default */
  border-top: 1px solid var(--border-footer-top, #EFF5EF); /* border/brandsecondary/quaternary */
  display: flex; flex-direction: column; width: 100%;
  box-shadow: 0 -1px 2px rgba(8,167,104,.08);
}

/* Button row (with BTN variant) */
.ftr-btn-row {
  display: flex; align-items: center;
  gap: 24px;                                       /* dimension/space/600 */
  padding: 16px 24px;                              /* dimension/space/400 / space/600 */
}
.ftr-autosave { display: flex; align-items: center; gap: 8px; flex: 1; }
.ftr-autosave-dot {
  width: 8px; height: 8px; border-radius: 4px;
  background: var(--surface-positive-secondary, #A2EDDE); /* surface/positive/secondary */
  opacity: 0.74;
}
.ftr-autosave-text {
  font-size: 14px;                                 /* typography/size/sm */
  color: var(--text-footer-secondary, #636B72);   /* text/content/secondary */
}
.ftr-btn-group { display: flex; align-items: center; gap: 12px; } /* dimension/space/300 */

/* Fill button */
.ftr-btn-fill {
  height: 44px;                                    /* dimension/size/900 */
  padding: 0 16px;                                 /* dimension/space/400 */
  border-radius: 8px;                              /* dimension/radius/200 */
  background: var(--surface-btn-fill, #007549);   /* surface/brandprimarybutton/default */
  color: var(--text-btn-fill, #FFFFFF);            /* text/brandprimarybutton/on-brand */
  font-size: 16px; font-weight: 500;               /* typography/size/base / weight/medium */
  box-shadow: 0 4px 4px rgba(8,167,104,.10);
  border: none; cursor: pointer;
}

/* Outline button */
.ftr-btn-outline {
  height: 44px;                                    /* dimension/size/900 */
  padding: 0 16px;                                 /* dimension/space/400 */
  border-radius: 8px;                              /* dimension/radius/200 */
  border: 1px solid var(--border-btn-outline, #C5EDDA); /* border/brandprimarybutton/tertiary */
  color: var(--text-btn-outline, #007549);         /* text/brandprimarybutton/default */
  background: transparent;
  font-size: 16px; font-weight: 500;
  cursor: pointer;
}
.ftr-btn-outline:hover { background: #F4FBF8; }   /* surface/brandtag/default-hover */

/* Copyright bar */
.ftr-bar {
  display: flex; align-items: center; gap: 24px;
  height: 56px;                                    /* dimension/size/1200 */
  padding: 0 24px;                                 /* dimension/space/600 */
  font-size: 12px;                                 /* typography/size/xs */
  color: var(--text-footer-secondary, #636B72);
}
.ftr-bar-sep { border-top: 1px solid var(--border-footer-sep, #F4F5F5); } /* border/neutral/quaternary */

/* Pagination row */
.ftr-pgn-row { display: flex; align-items: center; gap: 24px; padding: 16px 24px; }
.ftr-pgn-info { flex: 1; font-size: 14px; color: #636B72; }
.ftr-pgn { display: flex; align-items: center; gap: 4px; } /* dimension/space/100 */
.ftr-page-btn {
  height: 36px; min-width: 36px;
  border-radius: 8px;                              /* dimension/radius/200 */
  border: none; cursor: pointer;
  background: var(--surface-pagination-default, #FFFFFF);
  color: var(--text-pagination-default, #4D5358);
  font-size: 14px;
}
.ftr-page-btn:hover,
.ftr-page-btn.active { background: var(--surface-pagination-hover, #F4F5F5); } /* surface/pagination/hover */
.ftr-page-arrow {
  width: 36px; height: 36px; border-radius: 8px;
  border: none; background: transparent; cursor: pointer;
}
.ftr-page-arrow:hover { background: #F4F5F5; }
.ftr-page-arrow.ftr-next img { transform: rotate(180deg); }
```

### 6.22.6 Figma Asset UUIDs

| Asset | UUID | Expires |
|---|---|---|
| Chevron left (pagination arrow) | `cac561e9-0d17-438d-9677-41a0a0ea1ee6` | 7 days |
| Icon fill button leading | `5724afc6-b692-405f-841d-3ffe8c2b5a44` | 7 days |
| Icon outline button | `77e8f9e6-bbdc-4bd7-bfa3-57d296083400` | 7 days |

---

*End of Design Token Reference — MIH Design System 2.0 · Updated May 2026*

---

### 6.22 Side Menu + Side Menu Item

> Figma: **1-2281** (Side Menu), **1-2254** (Side Menu Item), **1-2282** (Notification item), **1-2390** (closed-state spec), **185-8826** (Notification badge)

#### 6.22.1 Rail / Container

| Property | CSS Variable | Primitive | Value |
|---|---|---|---|
| Background | `--surface-navigation-default` | `white` | `#FFFFFF` |
| Border-right | `--border-neutral-quaternary` | `neutral/50` | `#F4F5F5` |
| Width (closed) | — | `dimension/size/1600` | `64px` |
| Width (open) | — | `dimension/size/6400` | `256px` |
| Transition | — | cubic-bezier(.4,0,.2,1) | `width .25s` |

#### 6.22.2 Side Menu Item — States

| State | Surface | Text | Left Border |
|---|---|---|---|
| Default | transparent | `--text-sidemenu-default` (`neutral/600 #4D5358`) | — |
| Hover | `--surface-brandprimary-quaternary` | `--text-sidemenu-hover` (`brand/p700 #007549`) | — |
| Active (open) | `--surface-brandprimary-quaternary` | `--text-sidemenu-selected` (`brand/p700 #007549`) | `4px solid --surface-brandprimary-default` |
| Disabled | transparent | `--text-sidemenu-disabled` (`neutral/400 #ADB2B7`) | — |

#### 6.22.3 Side Menu Item — Dimensions

| Property | CSS Variable | Primitive | Value |
|---|---|---|---|
| Min height | — | `dimension/size/1200` | `48px` |
| Padding (open) | — | `space/200 space/600` | `8px 24px` |
| Padding-left (active) | — | `space/200 space/500` | `8px 20px` (border offset) |
| Gap | — | `dimension/space/300` | `12px` |
| Border-radius | — | `dimension/radius/200` | `6px` |
| Icon size | — | `dimension/size/400` | `16px` |
| Font size | — | `typography/size/400` | `16px` |

#### 6.22.4 Closed-state Centering (Figma 1-2390)

When `.sm-rail` is **64px wide**, icons center by:  
`padding: 8px 24px` → 24 + 16 + 24 = **64px** ✓  
Active closed: `padding-left: 20px; padding-right: 24px` to offset the 4px left border.

```css
.sm-rail:not(.open) .sm-item { justify-content: center; padding: 8px 24px; gap: 0; }
.sm-rail:not(.open) .sm-item.active { padding-left: 20px; padding-right: 24px; }
```

#### 6.22.5 Notification Badge (Figma 185-8826)

| Property | CSS Variable | Primitive | Value |
|---|---|---|---|
| Background | `--color-red-500` | `color/red/500` | `#ef4444` |
| Text color | `--text-danger-on-default` | `white` | `#FFFFFF` |
| Font size | — | `typography/size/300` | `12px` |
| Font weight | — | `typography/weight/medium` | `500` |
| Padding | — | `space/050 space/200` | `2px 8px` |
| Border-radius | `--dimension-radius-600` | `dimension/radius/600` | `24px` |

**Visibility rules** (Figma 1-2254 `showNoti` prop):
- Badge shows only when `open = Yes` AND `state ∈ {Default, Hover}` AND item is the Notification item
- Hidden on all other menu items (`buildItem()` checks `item.noti` flag)
- Hidden in closed state via CSS: `.sm-rail:not(.open) .sm-noti-badge { display: none }`

**Closed-state dot** (always visible when sidebar is closed):
```css
/* Absolute dot on bell icon — 7×7px, red/500, white border */
position: absolute; top: -3px; right: -4px;
width: 7px; height: 7px; border-radius: 50%;
background: var(--color-red-500);
border: 1.5px solid var(--surface-navigation-default);
```

#### 6.22.7 Logo Area — Figma 1:2283 + 1:2284

| Element | Property | Token / Value | Figma Node |
|---|---|---|---|
| Logo image | width × height | `40px × 40px` = `dimension/size/1000` | 1:2284 |
| Logo image | border-radius | `var(--dimension-radius-full)` = `9999px` | 1:2284 |
| Logo image | background | `#E7F1F8` (light blue) | 1:2284 DoctorA |
| Logo container | height | `64px` (matches header bar) | 1:2283 |
| Logo container | padding | `12px` horiz = `dimension/space/300` | 1:2283 |
| Logo container | gap | `12px` = `dimension/space/300` | 1:2283 |
| Logo container | border-bottom | `1px solid var(--border-neutral-quaternary)` | 1:2283 |
| Brand name | font-family | `var(--font-heading-graphic)` = Sao Chingcha Bold | 1:2286 |
| Brand name | font-size | `var(--typograpphy-size-base)` = `16px` | 1:2286 |
| Brand name | color | `#383B3D` | 1:2286 |
| Sub-text | font-size | `var(--typograpphy-size-xs)` = `12px` | 1:2287 |
| Sub-text | color | `#565D6C` | 1:2287 |

#### 6.22.8 System Switcher Icon — swap_horiz (Figma 1:2288)

Material icon `swap_horiz` — image asset rendered via Figma-exact relative/absolute inset layout.

| Property | Token | Value | Source |
|---|---|---|---|
| Outer container width/height | `dimension/size/600` | `24px × 24px` | Figma 1:2288 outer |
| Icon inner inset | — | `inset: 18.44% 10.1%` | Figma node `I1:2288;2116:2873` |
| Icon color (alias) | `icon/sidemenu/default` | `#4D5358` (neutral/600) | Baked into image asset |
| Opacity default | — | `0.7` | design token: muted state |
| Opacity hover | — | `1.0` | design token: active state |
| Closed-state | — | `opacity:0; pointer-events:none` | Figma: hidden when open=No |

**HTML structure (exact Figma layout):**
```html
<!-- Outer: dimension/size/600 = 24px — Figma 1:2288 -->
<span style="position:relative;display:inline-block;width:24px;height:24px;flex-shrink:0;">
  <!-- Inner: Figma inset-[18.44%_10.1%] node I1:2288;2116:2873 -->
  <span style="position:absolute;inset:18.44% 10.1%;">
    <img src="[figma-asset-url]"
         style="position:absolute;inset:0;width:100%;height:100%;display:block;"
         alt="swap_horiz" />
  </span>
</span>
```

**Behavior:** Click → dropdown popup เลือกระบบ (Default / Community / PHCIS / …)  
**Visibility:** fade `opacity:0; pointer-events:none` เมื่อ sidebar closed (เหมือน brand text)

```css
.sm-swap-icon {
  width: 24px; height: 24px;              /* dimension/size/600 — Figma 1:2288 */
  flex-shrink: 0; cursor: pointer;
  opacity: 0.7; transition: opacity .15s;
}
.sm-swap-icon:hover { opacity: 1; }
.sm-rail:not(.open) .sm-swap-icon { opacity: 0; pointer-events: none; transition: opacity .15s; }
```

#### 6.22.9 Profile / User Row — unfold_more (Figma 1:2303 · 1:2308)

| Element | Property | Token / Value | Figma Node |
|---|---|---|---|
| User row | height | `56px` | 1:2303 |
| User row | padding | `8px 12px` = `space/200 space/300` | 1:2303 |
| User row | gap | `12px` = `dimension/space/300` | 1:2303 |
| User row | border-top | `1px solid var(--border-neutral-quaternary)` | 1:2303 |
| Avatar | width × height | `40×40px` = `dimension/size/1000` | 1:2304 |
| Avatar | border-radius | `var(--dimension-radius-full)` = `9999px` | 1:2304 |
| Avatar | background | `#E7F1F8` | 1:2304 |
| User name | font-size | `var(--typograpphy-size-sm)` = `14px` | 1:2306 |
| User name | color | `var(--text-sidemenu-default)` = `#4D5358` | 1:2306 |
| Department | font-size | `var(--typograpphy-size-xs)` = `12px` | 1:2307 |
| Department | color | `var(--text-content-tertiary)` = `#858C92` | 1:2307 |
| unfold_more | width × height | `24×24px` = `dimension/size/600` | 1:2308 |

Material icon `unfold_more` — image asset rendered via Figma-exact relative/absolute inset layout.

| Property | Token | Value | Source |
|---|---|---|---|
| Outer container width/height | `dimension/size/600` | `24px × 24px` | Figma 1:2308 outer |
| Icon inner inset | — | `inset: 14.69% 33.02% 14.27% 33.02%` | Figma node `I1:2308;380:12559` |
| Icon color (alias) | `icon/sidemenu/default` | `#4D5358` (neutral/600) | Baked into image asset |

**HTML structure (exact Figma layout):**
```html
<!-- Outer: dimension/size/600 = 24px — Figma 1:2308 -->
<span style="position:relative;display:inline-block;width:24px;height:24px;flex-shrink:0;">
  <!-- Inner: Figma inset-[14.69%_33.02%_14.27%_33.02%] node I1:2308;380:12559 -->
  <span style="position:absolute;inset:14.69% 33.02% 14.27% 33.02%;">
    <img src="[figma-asset-url]"
         style="position:absolute;inset:0;width:100%;height:100%;display:block;"
         alt="unfold_more" />
  </span>
</span>
```

**Behavior:** Click user row → profile popup menu (โปรไฟล์ / ตั้งค่า / ออกจากระบบ)

#### 6.22.6 CSS Quick Reference

```css
/* Rail */
.sm-rail {
  width: 64px;
  transition: width .25s cubic-bezier(.4,0,.2,1);
  overflow: hidden;
  background: var(--surface-navigation-default);
  border-right: 1px solid var(--border-neutral-quaternary);
}
.sm-rail.open { width: 256px; }

/* Item */
.sm-item {
  min-height: 48px;
  gap: 12px;
  padding: 8px 24px;
  border-radius: 6px;
  color: var(--text-sidemenu-default);
  font-size: 16px;
}
.sm-item:hover:not(.sm-item-disabled) {
  background: var(--surface-brandprimary-quaternary);
  color: var(--text-sidemenu-hover);
}
.sm-item.active {
  background: var(--surface-brandprimary-quaternary);
  color: var(--text-sidemenu-selected);
  border-left: 4px solid var(--surface-brandprimary-default);
  border-radius: 0;
  padding-left: 20px;
}

/* Notification badge pill */
.sm-noti-badge {
  background: var(--color-red-500);           /* color/red/500 */
  color: var(--text-danger-on-default);       /* white */
  font-size: 12px; font-weight: 500;
  padding: 2px 8px;
  border-radius: var(--dimension-radius-600); /* 24px */
}
.sm-rail:not(.open) .sm-noti-badge { display: none; }
```

---

*End of Design Token Reference — MIH Design System 2.0 · Updated May 2026*

---

### 6.23 Profile Dropdown Menu (Figma 186:2898)

> Component: `Dropdown/Menu` · node **186:2883** · positioned to the **RIGHT** of the sidebar

#### 6.23.1 Container Tokens

| Property | Token | Value |
|---|---|---|
| Background | `surface/dropdown/default` | `#FFFFFF` |
| Border | `border/dropdown/default` | `1px solid #EBEBEB` |
| Border-radius | `dimension/radius/500` | `16px` |
| Padding | `dimension/space/100` | `4px` |
| Shadow | `Drop Shadow Bottom/400` | `0px 4px 4px 0px rgba(100,116,139,0.10), 0px 16px 32px 0px rgba(100,116,139,0.15)` |
| Width | — | `256px` |

#### 6.23.2 User Header Section (Figma 186:2884)

| Element | Token | Value |
|---|---|---|
| Gap | `dimension/space/300` | `12px` |
| Padding | `space/200 space/300` | `8px 12px` |
| Border-bottom | `border/neutral/quaternary` | `#F4F5F5` |
| Avatar size | `dimension/size/1000` | `40×40px` |
| Avatar border-radius | `dimension/radius/full` | `9999px` |
| Avatar bg | — | `#E7F1F8` |
| Name font-size | `typograpphy/size/sm` | `14px` |
| Name color | `text/modal/text` | `#4D5358` |
| Dept font-size | `typograpphy/size/xs` | `12px` |
| Dept color | `text/modal/subtext` | `#ADB2B7` |

#### 6.23.3 Menu Item Rows (Figma 186:2890–2894)

| Property | Token | Value |
|---|---|---|
| Min-height | — | `48px` |
| Padding | `space/200 space/300` | `8px 12px` |
| Gap | `dimension/space/200` | `8px` |
| Border-radius | `dimension/radius/100` | `6px` |
| Background default | `surface/list/default` | `#FFFFFF` |
| Background hover | `surface/brandprimary/quaternary` | `#F4FBF8` |
| Text color | `text/list/default` | `#4D5358` |
| Text font-size | `typograpphy/size/sm` | `14px` |
| Icon size | `dimension/size/400` | `16×16px` |

**Icon Figma insets (node → inset):**
| Icon | Figma Node | Inset |
|---|---|---|
| Language (กล) | `I186:2890;382:14365;2657:5299` | `24.79% 8.33% 21.04% 8.33%` |
| Chevron right (→) | `I186:2890;382:14422;328:4628` | `20.83% 33.33%` |
| Palette (โหมด) | `I186:2892;382:14365;2657:5325` | `8.33%` |
| Lock (รหัสผ่าน) | `I186:2893;382:14365;381:12847` | `4.17% 8.33%` |
| Bell (แจ้งเตือน) | `I186:2894;382:14365;328:4450` | `4.17% 8.33% 4.18% 8.33%` |
| Log out | `I186:2897;382:14362;381:12853` | `8.33%` |

#### 6.23.4 Colorful Badge TH (Figma I186:2890;2798:4262)

| Property | Token | Value |
|---|---|---|
| Background | `surface/colorfulbadge/blue` | `#DBEAFE` |
| Text color | `text/colorfulbadge/blue` | `#1D4ED8` |
| Height | — | `24px` |
| Padding | `space/100 space/200` | `4px 8px` |
| Border-radius | — | `999px` |
| Font | `typograpphy/family/body Sarabun:Medium` | `12px weight:500` |

#### 6.23.5 Notification Badge Color Fix (Figma 185:8823)

**Was:** `background: color/red/500 = #ef4444`  
**Correct:** `background: surface/pink/default = #be185d`

| Property | Token | Value |
|---|---|---|
| Background | `surface/pink/default` | `#BE185D` |
| Text color | `text/danger/on-default` | `#FFFFFF` |
| Font | `typograpphy/family/body Sarabun:Medium` | `typograpphy/size/xs = 12px` |
| Padding | `space/050 space/200` | `2px 8px` |
| Border-radius | `dimension/radius/600` | `24px` |

```css
:root {
  --surface-pink-default: #be185d;  /* surface/pink/default — Figma 185:8823 Noti badge */
}
.sm-noti-badge {
  background: var(--surface-pink-default);  /* NOT color/red/500 */
  color: var(--text-danger-on-default);     /* white */
  font-size: 12px; font-weight: 500;
  padding: 2px 8px;
  border-radius: var(--dimension-radius-600); /* 24px */
}
```

---

*End of Design Token Reference — MIH Design System 2.0 · Updated May 2026*

---

## 6.24 Breadcrumb (Figma 14:3192)

Breadcrumb shows the current navigation path. Supports 4 variants: **Level 2**, **Level 3**, **Level 4**, and **Truncated**. Implemented in `mih-components-all-v2.html` · panel `#panel-breadcrumb`.

---

### 6.24.1 Container

| Property | Design Token | CSS Value |
|---|---|---|
| Layout | — | `display: flex; align-items: center; flex-wrap: wrap` |
| Gap between cells | `dimension/space/200` | `8px` |

**CSS class:** `.bc`

```css
.bc {
  display: flex;
  align-items: center;
  gap: var(--dim-space-200, 8px);   /* dimension/space/200 */
  flex-wrap: wrap;
}
```

---

### 6.24.2 Breadcrumb Cell (each node)

Each segment (home, separator, link, current) is wrapped in a `.bc-cell`:

| Property | Design Token | CSS Value |
|---|---|---|
| Padding Y | `dimension/space/100` | `4px` (top + bottom) |
| Padding X | — | `0` |
| Display | — | `flex; align-items:center; justify-content:center` |

```css
.bc-cell {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--dim-space-100, 4px) 0;   /* dimension/space/100 */
  flex-shrink: 0;
}
```

---

### 6.24.3 Home Icon (Figma I14:3195;7758:11735)

Container: **16 × 16 px** with `overflow: hidden`. Uses inline SVG (stroke-based, inherits `currentColor` from `.bc-home-wrap`).

| Property | Design Token | Value |
|---|---|---|
| Icon container size | `dimension/size/400` | `16 × 16 px` |
| Icon color | `text/breadcrumb/pagename` | `#858C92` (neutral-500) |
| Stroke width | `dimension/stroke/100` | `1.8 px` |

```css
.bc-home-wrap {
  width: 16px; height: 16px;
  position: relative; overflow: hidden;
  flex-shrink: 0;
  color: var(--text-breadcrumb-pagename, #858C92);   /* text/breadcrumb/pagename */
}
.bc-home-wrap svg {
  position: absolute; inset: 0;
  width: 100%; height: 100%;
}
```

SVG path (Material Design "home", stroke style):
```html
<svg viewBox="0 0 24 24" fill="none" stroke="currentColor"
     stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
  <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/>
  <polyline points="9 22 9 12 15 12 15 22"/>
</svg>
```

> **Note:** Figma uses an image asset (`inset: 8.33% 12.5%` container, `-3.75% -4.17%` overflow image) but the implementation substitutes an inline SVG for stability (Figma asset URLs expire after 7 days).

---

### 6.24.4 Typography Tokens

All text in the breadcrumb (links, separator, current page) shares the same base typography:

| Property | Design Token | Value |
|---|---|---|
| Font family | `typograpphy/family/body` | `'Sarabun', sans-serif` |
| Font size | `typograpphy/size/sm` | `14 px` |
| Font weight | `typograpphy/weight/regular` | `400` |
| Line height (links) | — | `1.5` |
| Line height (separator) | — | `normal` |

---

### 6.24.5 Color Tokens

Semantic token → Primitive alias:

| Element | Semantic Token | CSS Variable | Primitive | Hex |
|---|---|---|---|---|
| Separator `/` | `text/breadcrumb/pagename` | `--text-breadcrumb-pagename` | `neutral/500` | `#858C92` |
| Inactive link text | `text/breadcrumb/pagename` | `--text-breadcrumb-pagename` | `neutral/500` | `#858C92` |
| Ellipsis `...` | `text/breadcrumb/pagename` | `--text-breadcrumb-pagename` | `neutral/500` | `#858C92` |
| Current page text | `text/breadcrumb/current` | `--text-breadcrumb-current` | `neutral/900` | `#363B3F` |
| Home icon | `text/breadcrumb/pagename` | `--text-breadcrumb-pagename` | `neutral/500` | `#858C92` |

Defined in `:root`:
```css
:root {
  --text-breadcrumb-pagename: #858C92;   /* text/breadcrumb/pagename → neutral-500 */
  --text-breadcrumb-current:  #363B3F;   /* text/breadcrumb/current  → neutral-900 */
}
```

---

### 6.24.6 CSS Component Classes

```css
/* Separator "/" */
.bc-sep {
  font-family: 'Sarabun', sans-serif;
  font-size: 14px;                  /* typograpphy/size/sm */
  font-weight: 400;                 /* typograpphy/weight/regular */
  line-height: normal;
  color: var(--text-breadcrumb-pagename, #858C92);
  white-space: nowrap;
}

/* Page link (non-current segments) */
.bc-link {
  font-family: 'Sarabun', sans-serif;
  font-size: 14px;                  /* typograpphy/size/sm */
  font-weight: 400;                 /* typograpphy/weight/regular */
  line-height: 1.5;
  color: var(--text-breadcrumb-pagename, #858C92);
  white-space: nowrap;
}

/* Current / last page */
.bc-current {
  font-family: 'Sarabun', sans-serif;
  font-size: 14px;                  /* typograpphy/size/sm */
  font-weight: 400;                 /* typograpphy/weight/regular */
  line-height: 1.5;
  color: var(--text-breadcrumb-current, #363B3F);
  white-space: nowrap;
}
```

---

### 6.24.7 Breadcrumb Variants

| Variant | Structure | Figma node |
|---|---|---|
| Level 2 | 🏠 `/` **Current Page** | 14:3193 |
| Level 3 | 🏠 `/` Products `/` **Product Detail** | 14:3200 |
| Level 4 | 🏠 `/` Products `/` Category `/` **Product Detail** | 14:3211 |
| Truncated | 🏠 `/` `...` `/` Category `/` **Current Page** | 14:3226 |

HTML structure for **Level 3**:
```html
<div class="bc">
  <!-- Home -->
  <div class="bc-cell">
    <div class="bc-home-wrap"><!-- SVG home icon --></div>
  </div>
  <!-- Separator -->
  <div class="bc-cell"><span class="bc-sep">/</span></div>
  <!-- Link -->
  <div class="bc-cell"><span class="bc-link">Products</span></div>
  <!-- Separator -->
  <div class="bc-cell"><span class="bc-sep">/</span></div>
  <!-- Current -->
  <div class="bc-cell"><span class="bc-current">Product Detail</span></div>
</div>
```

---

### 6.24.8 Full Token Map

| CSS Property | Design Token path | Resolved Value |
|---|---|---|
| `.bc` → gap | `dimension/space/200` | `8px` |
| `.bc-cell` → padding-y | `dimension/space/100` | `4px` |
| `.bc-home-wrap` → size | `dimension/size/400` | `16×16 px` |
| `.bc-home-wrap` → color | `text/breadcrumb/pagename` | `#858C92` |
| `.bc-sep` → font-size | `typograpphy/size/sm` | `14px` |
| `.bc-sep` → color | `text/breadcrumb/pagename` | `#858C92` |
| `.bc-link` → font-size | `typograpphy/size/sm` | `14px` |
| `.bc-link` → font-weight | `typograpphy/weight/regular` | `400` |
| `.bc-link` → color | `text/breadcrumb/pagename` | `#858C92` |
| `.bc-current` → font-size | `typograpphy/size/sm` | `14px` |
| `.bc-current` → font-weight | `typograpphy/weight/regular` | `400` |
| `.bc-current` → color | `text/breadcrumb/current` | `#363B3F` |

---

## 6.25 Select (Figma 322:2389)

**Source:** `MIH-Design-System-Foundation` · node `322:2389` · Default Select · Sizes: Large / Medium / Small

---

### 6.25.1 Component Structure

```
select-wrap (flex-col, gap: 6px)
├── field-name row (flex-row, gap: 2px)
│   ├── Label   — text/neutral/secondary
│   └── *       — text/error/default (required asterisk)
├── select-box (flex-row, gap: 8px, height: 40/36/32px by size)
│   ├── [optional] leading icon   — 16×16 px
│   ├── select-text               — placeholder or selected value
│   └── chevron-down icon         — 16×16 px, rotates 180° when open
└── helper-text                   — normal or error variant
```

**Popover list (`<Popover>` under the trigger):** Options are plain text rows with optional `icon`, **or** — when `BadgeSelectForm` sets `badgeKind="severity"` — each option may include `colorfulBadgeStyle` (`red` \| `orange` \| `blue` \| `grey` \| …). Those rows render a **ColorfulBadge** (small, secondary hierarchy) per **PHCIS \| OPD [334:56417](https://www.figma.com/design/tobdR9prwt00v4X3aSMsxF/PHCIS-%7C-OPD?node-id=334-56417)** (OPD allergy severity: **รุนแรง · ปานกลาง · เล็กน้อย** — three levels). With `badgeKind="status"` and allergy likelihood option values (`definite` \| `probable` \| `possible` \| `unlikely`), the same pattern applies per **PHCIS \| OPD [334:56749](https://www.figma.com/design/tobdR9prwt00v4X3aSMsxF/PHCIS-%7C-OPD?node-id=334-56749)** (**แน่นอน · น่าจะใช่ · เป็นไปได้ · สงสัย** — teal / blue / indigo / purple badges; stored value `unlikely` maps to label **สงสัย**). The active row shows a trailing **check** tinted `icon/positive/success` (`--icon-positive-success`).

---

### 6.25.2 States (Figma 268:55 · Default Select)

| State | Border | Border Width | Focus Ring (box-shadow) | Background | Text Color | Icon Color |
|---|---|---|---|---|---|---|
| Default | `border/input/default` → `#E2E4E6` | 1px | — | `#FFFFFF` | `#ADB2B7` (placeholder) | `#ADB2B7` |
| Hover | `border/input/hover` → `primary/600` → `#08A768` | 2px | `primaryBorder/600` → 4px | `#FFFFFF` | `#ADB2B7` (placeholder) | `#ADB2B7` |
| Open | `border/input/hover` → `primary/600` → `#08A768` | 2px | `primaryBorder/600` → 4px | `#FFFFFF` | `#4D5358` (value) | `#4D5358` |
| Filled | `border/input/default` → `#E2E4E6` | 1px | — | `#FFFFFF` | `#4D5358` (value) | `#4D5358` |
| Error | `border/input/error` → `#DC2626` | 2px | — | `#FFFFFF` | `#ADB2B7` (placeholder) | `#ADB2B7` |
| Disabled | `border/input/default` → `#E2E4E6` | 1px | — | `#F4F5F5` | `#ADB2B7` (placeholder) | `#ADB2B7` |

> **Token alias chain for Hover / Open:**
> `border/input/hover` (Semantic) → `primary/600` (Brand) → `color/emerald/600` (Primitive) → `#08A768`
> Focus ring: `primaryBorder/600` (Brand) → `rgba(primary/600, 0.15)` — changes automatically with brand theme.

---

### 6.25.3 Size Variants

| Size | Height | Font Size | Padding (T/R/B/L) |
|---|---|---|---|
| Large | 40px (`dimension/size/800`) | 14px | 8px / 16px / 8px / 16px |
| Medium | 36px (`dimension/size/700`) | 14px | 8px / 16px / 8px / 16px |
| Small | 32px (`dimension/size/600`) | 13px | 8px / 16px / 8px / 16px |

---

### 6.25.4 Color Tokens

| Token Path | Resolved Value | Usage |
|---|---|---|
| `surface/input/default` | `#FFFFFF` | Select bg (default / hover / open / filled / error) |
| `surface/input/disabled` | `#F4F5F5` (n100) | Select bg disabled |
| `border/input/default` | `#E2E4E6` (n200) | Border: default / filled / disabled (1px) |
| `border/input/focus` | `#08A768` (em600) | Border: hover / open (2px) |
| `border/input/error` | `#DC2626` (rd600) | Border: error (2px) |
| `text/neutral/tertiary` | `#ADB2B7` (n400) | Placeholder text · helper text · icon default |
| `text/neutral/secondary` | `#4D5358` (n700) | Label · filled value text · icon open/filled |
| `text/error/default` | `#DC2626` (rd600) | Helper text error · required asterisk |
| `border/input/hover` | `#08A768` (primary/600) | Border: hover / open (2px) — aliases Brand `primary/600` |
| `primaryBorder/600` | `rgba(8,167,104,0.15)` | Focus ring shadow (4px) — Hover and Open states |
| `icon/brand/check` | `#007549` (em700) | Dropdown list item check icon (selected) |
| `surface/list/hover` | `#f4fbf8` (em50) | List item bg on hover |
| `surface/list/selected` | `#f4fbf8` (em50) | List item bg when selected |

---

### 6.25.5 Dropdown Panel Tokens

| Property | Value | Token |
|---|---|---|
| Background | `#FFFFFF` | `surface/input/default` |
| Border | `1px solid #EBEBEB` | `border/dropdown/default` |
| Border radius | `16px` | `dimension/radius/400` |
| Padding | `4px` | `dimension/space/100` |
| Box shadow | `0 4px 4px rgba(100,116,139,.1), 0 16px 32px rgba(100,116,139,.15)` | elevation/dropdown |
| List item radius | `6px` | `dimension/radius/100` |
| List item padding | `8px 12px` | `dimension/space/200+300` |
| List item min-height | `40px` | `dimension/size/800` |
| List item gap | `8px` | `dimension/space/200` |

---

### 6.25.6 Figma Variable IDs

| Element | Variable ID | Resolved |
|---|---|---|
| Select bg | `VariableID:295:545` | `surface/input/default` → `#FFFFFF` |
| Border default | `VariableID:2049:3921` / `2394:3967` / `2394:3968` | `border/input/default` → `#E2E4E6` |
| Border hover/open | `VariableID:2393:3317` | `border/input/hover` → `primary/600` → `#08A768` |
| Focus ring shadow | `VariableID:2933:17188` | `primaryBorder/600` → `rgba(8,167,104,0.15)` |
| Border error | `VariableID:2402:4518` | `border/input/error` → `#DC2626` |
| Placeholder text | `VariableID:2056:4213` | `text/neutral/tertiary` → `#ADB2B7` |
| Filled text | `VariableID:2056:4214` | `text/neutral/secondary` → `#4D5358` |
| Disabled text | `VariableID:2056:4215` | `text/neutral/disabled` → `#ADB2B7` |
| Label text | `VariableID:2402:4515` | `text/neutral/secondary` → `#4D5358` |
| Required * | `VariableID:2402:4516` | `text/error/default` → `#DC2626` |
| Helper text | `VariableID:2402:4513` | `text/neutral/tertiary` → `#ADB2B7` |
| Helper error | `VariableID:2402:4514` | `text/error/default` → `#DC2626` |
| Icon default | `VariableID:2394:3959` | `icon/neutral/tertiary` → `#ADB2B7` |
| Icon open/filled | `VariableID:2394:3961` / `2394:3966` | `icon/neutral/secondary` → `#4D5358` |

---

### 6.25.7 CSS Implementation (prefix: `select-`)

```css
/* ── Wrapper ── */
.select-wrap {
  display: flex;
  flex-direction: column;
  gap: 6px;                       /* label → field → helper spacing */
  width: 100%;
}

/* ── Label row ── */
.field-label {
  color: var(--n700);             /* text/neutral/secondary = #4D5358 */
  font-weight: 500;               /* Medium */
  font-size: 14px;
  font-family: 'Sarabun', sans-serif;
}
.field-required { color: var(--rd600); font-size: 14px; }  /* text/error/default */

/* ── Select trigger ── */
.select-box {
  display: flex;
  align-items: center;
  gap: 8px;                       /* dimension/space/200 */
  background: var(--wh);          /* surface/input/default */
  height: 40px;                   /* dimension/size/800 — Large */
  padding: 8px 16px;              /* dimension/space/200 + 400 */
  border-radius: 8px;             /* dimension/radius/200 */
  transition: border-color .15s, box-shadow .15s;
}

/* Size variants */
.select-box.sz-md { height: 36px; }   /* dimension/size/700 */
.select-box.sz-sm { height: 32px; font-size: 13px; }  /* dimension/size/600 */

/* State borders */
.select-box.state-default  { border: 1px solid var(--n200); }
.select-box.state-hover    { border: 2px solid var(--em600); box-shadow: 0 0 0 4px var(--primaryBorder-600); }
.select-box.state-open     { border: 2px solid var(--em600); box-shadow: 0 0 0 4px var(--primaryBorder-600); }
.select-box.state-filled   { border: 1px solid var(--n200); }
.select-box.state-error    { border: 2px solid var(--rd600); }
.select-box.state-disabled { border: 1px solid var(--n200); background: var(--n100); cursor: not-allowed; }

/* Icon colors (via currentColor) */
.select-box.state-default  .select-icon,
.select-box.state-default  .chevron  { color: var(--n400); }
.select-box.state-hover    .select-icon,
.select-box.state-hover    .chevron  { color: var(--n400); }
.select-box.state-open     .select-icon,
.select-box.state-open     .chevron  { color: var(--n700); }
.select-box.state-filled   .select-icon,
.select-box.state-filled   .chevron  { color: var(--n700); }
.select-box.state-error    .select-icon,
.select-box.state-error    .chevron  { color: var(--n400); }
.select-box.state-disabled .select-icon,
.select-box.state-disabled .chevron  { color: var(--n400); opacity: .6; }

/* Text colors */
.select-text {
  flex: 1;
  font-family: 'Sarabun', sans-serif;
  font-size: 14px;
  font-weight: 400;
  overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
}
.select-box.state-default  .select-text { color: var(--n400); }  /* placeholder */
.select-box.state-hover    .select-text { color: var(--n400); }  /* placeholder */
.select-box.state-open     .select-text { color: var(--n700); }  /* value */
.select-box.state-filled   .select-text { color: var(--n700); }  /* value */
.select-box.state-error    .select-text { color: var(--n400); }  /* placeholder */
.select-box.state-disabled .select-text { color: var(--n400); }  /* placeholder */

/* Chevron */
.chevron {
  flex-shrink: 0; width: 16px; height: 16px;
  display: flex; align-items: center; justify-content: center;
  transition: transform .15s ease;
}
.chevron.open { transform: rotate(180deg); }

/* Helper text */
.helper-text { font-size: 14px; font-weight: 400; color: var(--n400); }
.helper-text.error { color: var(--rd600); }

/* ── Dropdown panel ── */
.dropdown-panel {
  display: none;
  background: var(--wh);
  border: 1px solid #EBEBEB;
  border-radius: 16px;            /* dimension/radius/400 */
  padding: 4px;                   /* dimension/space/100 */
  box-shadow: 0 4px 4px -1px rgba(100,116,139,.10), 0 16px 32px -1px rgba(100,116,139,.15);
  width: 100%;
  margin-top: 4px;
}
.dropdown-panel.open,
.dropdown-panel.static-open { display: block; }

/* ── List items ── */
.list-item, .sel-list-item {
  display: flex; align-items: center; gap: 8px;
  background: var(--wh);
  min-height: 40px;
  padding: 8px 12px;              /* dimension/space/200 + 300 */
  border-radius: 6px;             /* dimension/radius/100 */
  cursor: pointer;
  transition: background .1s;
}
.sel-list-item:hover { background: var(--em50); }   /* surface/list/hover → primary/50 */
.sel-list-item.selected { background: var(--em50); } /* surface/list/selected */

/* Check icon (selected state) */
.sel-check-icon {
  margin-left: auto; flex-shrink: 0;
  width: 16px; height: 16px;
  display: flex; align-items: center; justify-content: center;
  opacity: 0;
  color: var(--em700);            /* icon/brand/check = #007549 */
}
.sel-list-item.selected .sel-check-icon { opacity: 1; }
```

---

### 6.25.8 Icons Used

| Icon | Figma Node | Size | SVG viewBox | Usage |
|---|---|---|---|---|
| `calendar_today` | `324:2528` | 24×24 | `0 0 24 24` | Leading icon (optional) |
| `Chevron down` | `294:7948` | 24×24 | `0 0 24 24` | Trailing chevron (rotates 180° when open) |
| `Check` | `294:8259` | 24×24 | `0 0 24 24` | List item selected indicator |
| `User` | `295:432` | 24×24 | `0 0 24 24` | List item leading icon |

All icons use `fill="currentColor"` so CSS `color` controls the fill.

---

### 6.25.9 Behavior (Tailwind Headless UI pattern)

```javascript
// Toggle open/close
function toggleSelect() {
  const panel = document.getElementById('sel-live-panel');
  const box   = document.getElementById('sel-live-box');
  const chev  = document.getElementById('sel-live-chev');
  const isOpen = panel.classList.contains('open');
  panel.classList.toggle('open');
  const hasSel = !!document.querySelector('#sel-live-panel .sel-list-item.selected');
  box.className = 'select-box sz-lg ' + (isOpen ? (hasSel ? 'state-filled' : 'state-default') : 'state-open');
  chev.style.transform = isOpen ? '' : 'rotate(180deg)';
  // Auto-close on outside click
  if (!isOpen) {
    setTimeout(() => {
      document.addEventListener('click', function h(e) {
        if (!box.contains(e.target) && !panel.contains(e.target)) {
          panel.classList.remove('open');
          box.className = 'select-box sz-lg ' + (hasSel ? 'state-filled' : 'state-default');
          chev.style.transform = '';
          document.removeEventListener('click', h);
        }
      });
    }, 10);
  }
}

// Pick an option
function pickSelect(val) {
  document.getElementById('sel-live-val').textContent = val;
  document.getElementById('sel-live-box').className = 'select-box sz-lg state-filled';
  document.getElementById('sel-live-panel').classList.remove('open');
  document.getElementById('sel-live-chev').style.transform = '';
  document.querySelectorAll('#sel-live-panel .sel-list-item').forEach(i => {
    const label = i.querySelector('.list-text');
    i.classList.toggle('selected', label && label.textContent.trim() === val);
  });
}
```

---

## 6.26 Search (Figma 2056:4246)

### Overview

**Figma node:** `2056:4246` — Primary Search + Secondary Search
**Component variants:** Primary Search (5 states) · Secondary Search (4 states × 2 sizes)

The Search component is divided into two variants aligned with the MIH brand system. All colors, spacing, and shadows reference semantic tokens that alias MIH primitive tokens.

---

### 6.26.1 Primary Search

> Current implementation follows §6.17 and Figma `2056:4246`. The legacy avatar-pile treatment has been retired.

**Dimensions (from Figma):**
- Height: `56px` (`dimension/size/1200`)
- Width: `min-width: 160px` (Default/Typing use `320px`), `max-width: 640px`
- Padding: `16px 24px` (`dimension/space/400` × `dimension/size/500`)
- Border-radius: `9999px` (`dimension/radius/full`)
- Border-width: `2px` (`dimension/stroke/200`)
- Inner gap (icon ↔ text): `16px` (`dimension/space/400`)
- Outer gap (left-block ↔ right-block): `24px` (`dimension/space/600`)
- X Button: `40px × 40px`, radius `24px` (`dimension/radius/600`) — Typing only
- Submit button: `40px × 40px`, radius full — Typing only

**State Token Table:**

| State | Border Token | Primitive | Hex |
|-------|-------------|-----------|-----|
| Default | `border/primarySearch/default` | emerald/600 | `#08A768` |
| Hover | `border/primarySearch/hover` | emerald/700 | `#007549` |
| Focus | `border/primarySearch/focus` | emerald/600 | `#08A768` |
| Typing | `border/primarySearch/typing` | emerald/700 | `#007549` |
| Filled | `border/primarySearch/filled` | emerald/600 | `#08A768` |

| State | Icon Token | Primitive | Hex |
|-------|-----------|-----------|-----|
| Default | `icon/primarySearch/default` | primary/600 = emerald/600 | `#08A768` |
| Hover | `icon/primarySearch/hover` | primary/700 = emerald/700 | `#007549` |
| Focus | `icon/primarySearch/focus` | primary/600 = emerald/600 | `#08A768` |
| Typing | `icon/primarySearch/typing` | primary/700 = emerald/700 | `#007549` |
| Filled | `icon/primarySearch/filled` | primary/600 = emerald/600 | `#08A768` |

| State | Text/Placeholder Token | Primitive | Hex |
|-------|----------------------|-----------|-----|
| Default | `text/primarySearch/default` | neutral/400 | `#ADB2B7` |
| Hover | `text/primarySearch/hover` | neutral/700 | `#4D5358` |
| Focus | `text/primarySearch/focus` | neutral/400 | `#ADB2B7` |
| Typing | `text/primarySearch/typing` | neutral/700 | `#4D5358` |
| Filled | `text/primarySearch/filled` | neutral/700 | `#4D5358` |

**Shadow (all states):**
- Token: `Brand Drop Shadow Bottom/200` (PrimaryShadow/600 + PrimaryShadow/601)
- CSS: `box-shadow: 0 1px 4px rgba(8,167,104,0.10), 0 1px 2px rgba(8,167,104,0.08)`
- Variable: `--PrimaryShadow-600: rgba(8,167,104,0.08)` · `--PrimaryShadow-601: rgba(8,167,104,0.10)`

**Right side — Default / Hover / Focus / Filled:** no trailing controls.

**Right side — Typing state:**
- X Button (40×40, brand-quaternary surface)
- Green submit button (40×40, full radius, bg `surface/brandPrimaryButton/default = #007549`)
- Arrow-up icon inside button: 16×16, white (`icon/brandPrimary/on-brand`)
- Token: `surface/brandPrimaryButton/default` = emerald/700 = `#007549`

**Typing dropdown (aliased from Dropdown/Default):**
- Background: `surface/dropdown/default` = `#FFFFFF`
- Border: `1px solid border/dropdown/default = #EBEBEB`
- Radius: `16px`
- Padding: `4px`
- Shadow: `0 16px 32px 0 rgba(100,116,139,0.15), 0 4px 4px 0 rgba(100,116,139,0.10)` (`Drop Shadow Bottom/400`)
- List items: `min-height: 48px`, `padding: 8px 12px`, `border-radius: 6px`
- List text: `color: text/list/default = #4D5358` (neutral/700)
- Hover bg: `surface/list/hover = #F4FBF8` (emerald/50)
- Hover text: `text/list/hover = #007549` (emerald/700)

---

### 6.26.2 Secondary Search

**Dimensions (from Figma):**
- Height: `40px` (default) · `36px` (small)
- Padding: `0 12px`
- Border-radius: `8px` (`dimension/radius/300`)
- Border-width: Default/Filled `1px` · Hover/Typing `2px`
- Focus ring: `0 0 0 4px rgba(8,167,104,0.15)` (`primaryBorder/600`)

**State Token Table:**

| State | Border Token | Primitive | Hex | Width |
|-------|-------------|-----------|-----|-------|
| Default | `border/secondarySearch/default` | neutral/300 | `#C9CDD0` | 1px |
| Hover | `border/secondarySearch/hover` | emerald/700 | `#007549` | 2px |
| Typing | `border/secondarySearch/typing` | emerald/700 | `#007549` | 2px |
| Filled | `border/secondarySearch/filled` | neutral/300 | `#C9CDD0` | 1px |

| State | Icon Token | Hex |
|-------|-----------|-----|
| Default | `icon/secondarySearch/default` = neutral/400 | `#ADB2B7` |
| Hover / Typing / Filled | `icon/secondarySearch/typing` = neutral/700 | `#4D5358` |

**Clear (×) button:** Visible in Typing + Filled states · color `#9AA5AE` (neutral/500)

---

### 6.26.3 CSS Implementation

```css
/* ── Primary Search ── */
.primary-search {
  background: var(--wh);                /* surface/search/default */
  border-radius: 16px;                  /* dimension/radius/500 */
  padding: 16px 24px;                   /* dimension/space/400 × dimension/size/500 */
  height: 64px;
  border-style: solid;
  border-width: 2px;                    /* dimension/stroke/200 */
  min-width: 320px; max-width: 640px; width: 100%;
  box-shadow: 0 1px 4px var(--PrimaryShadow-601), 0 1px 2px var(--PrimaryShadow-600);
  display: flex; align-items: center; gap: 24px;   /* dimension/space/600 */
  box-sizing: border-box; transition: border-color .15s;
}
.primary-search.state-default { border-color: var(--em600); } /* border/primarySearch/default */
.primary-search.state-hover   { border-color: var(--em700); } /* border/primarySearch/hover */
.primary-search.state-typing  { border-color: var(--em700); } /* border/primarySearch/typing */
.primary-search.state-filled  { border-color: var(--em600); } /* border/primarySearch/filled */

/* Icon colors via currentColor */
.primary-search.state-default .ps-icon { color: var(--em600); }
.primary-search.state-hover   .ps-icon { color: var(--em700); }
.primary-search.state-typing  .ps-icon { color: var(--em700); }
.primary-search.state-filled  .ps-icon { color: var(--em600); }

/* Placeholder / value text */
.primary-search.state-default .ps-placeholder { color: var(--n400); } /* text/primarySearch/default */
.primary-search.state-hover   .ps-placeholder { color: var(--n700); }
.primary-search.state-typing  .ps-value       { color: var(--n700); }
.primary-search.state-filled  .ps-value        { color: var(--n700); }

/* Caption (ค้นหาล่าสุด) */
.ps-caption { font-size: 12px; white-space: nowrap; font-family: 'Sarabun', sans-serif; }
.primary-search.state-default .ps-caption { color: var(--n400); }
.primary-search.state-hover   .ps-caption { color: var(--n700); }
.primary-search.state-filled  .ps-caption { color: var(--n700); }

/* Avatar stack */
.ps-avatars { display: flex; padding-right: 12px; }
.ps-avatar {
  width: 32px; height: 32px; border-radius: 16px;
  border: 2px solid var(--wh); overflow: hidden;
}
.ps-avatar + .ps-avatar { margin-left: -12px; }

/* Submit button (Typing) */
.ps-submit {
  width: 36px; height: 36px; border-radius: 24px;
  background: var(--em700);            /* surface/brandPrimaryButton/default */
  display: flex; align-items: center; justify-content: center; flex-shrink: 0;
  box-shadow: 0 4px 4px var(--PrimaryShadow-600), 0 4px 4px var(--PrimaryShadow-601);
}

/* Typing-state dropdown — aliases Dropdown/Default tokens */
.ps-dropdown {
  background: var(--wh);              /* surface/dropdown/default */
  border: 1px solid #EBEBEB;         /* border/dropdown/default */
  border-radius: 16px; padding: 4px; /* dimension/radius/500 · dimension/space/100 */
  box-shadow: 0 16px 32px 0 rgba(100,116,139,.15), 0 4px 4px 0 rgba(100,116,139,.10);
}
.ps-list-item {
  display: flex; align-items: center; gap: 8px;
  padding: 8px 12px; border-radius: 6px;
  min-height: 48px; box-sizing: border-box;
  font-size: 14px; color: var(--n700);        /* text/list/default */
  cursor: pointer; transition: background .1s;
}
.ps-list-item:hover {
  background: var(--em50);                     /* surface/list/hover = #F4FBF8 */
  color: var(--em700);                         /* text/list/hover = #007549 */
}

/* ── Secondary Search ── */
.secondary-search {
  display: flex; align-items: center; gap: 8px;
  padding: 0 12px; border-radius: 8px;
  background: var(--wh);
  border-style: solid;
  box-sizing: border-box; transition: all .15s;
}
.secondary-search.sz-md { height: 40px; }
.secondary-search.sz-sm { height: 36px; }
.secondary-search.ss-default { border-width: 1px; border-color: var(--n300); }
.secondary-search.ss-hover   { border-width: 2px; border-color: var(--em700); box-shadow: 0 0 0 4px rgba(8,167,104,.15); }
.secondary-search.ss-typing  { border-width: 2px; border-color: var(--em700); box-shadow: 0 0 0 4px rgba(8,167,104,.15); }
.secondary-search.ss-filled  { border-width: 1px; border-color: var(--n300); }
```

---

### 6.26.4 Behavior — State Machine (Tailwind / Headless UI Combobox)

**Primary Search state transitions:**

```
default ──mouseenter──▶ hover ──mouseleave──▶ default
default / hover ──focus/click──▶ typing (dropdown opens)
typing ──pick item──▶ filled (dropdown closes, input populated)
typing ──Enter (has value)──▶ filled
typing ──Esc──▶ default (input cleared)
typing ──outside click / blur──▶ filled (has value) | default (empty)
filled ──focus/click──▶ typing (re-opens dropdown)
```

**Right side switches by state:**

| State | Right element |
|-------|--------------|
| Default / Hover / Filled | Avatar stack (3 × 32px circles, overlap -12px) + "ค้นหาล่าสุด" caption |
| Typing | Green submit button (36×36, radius 24px, bg `surface/brandPrimaryButton/default`) |

**Dropdown behavior:**
- Opens on `focus`; **content depends on whether the input is empty**:
  - **Empty input → Recent-search history** (`history` prop, persisted to `localStorage`). Header reads `ค้นหาล่าสุด`; rows use a `schedule` leading icon. If `onHistoryClear` is wired, a `ล้างประวัติ` affordance appears in the header. When `history` is empty the dropdown still opens and shows `historyEmptyMessage` (default `ยังไม่มีประวัติการค้นหา`) so the user gets feedback that the section exists. Pass `historyEmptyMessage={null}` to hide the dropdown entirely on empty history.
  - **Typing (≥ 1 char) → Live suggestions** (`suggestions` prop, derived by the page from its row set). Rows use a `search` leading icon, render a primary `label` + tertiary `sublabel`, and may carry a custom `icon`. When no result matches, the popover shows `emptyMessage` (default `ไม่พบรายการที่ค้นหา`) so the user gets feedback instead of a silent close.
- Both `history` and `suggestions` accept either plain strings or rich records `{ id?, label, sublabel?, value?, icon? }`. Clicking a row commits `value ?? label` through `onChange` + `onSubmit` so the page can also push it back into `history`.
- Search fields advertised by `placeholder` should match the fields the page actually filters (e.g. OPD Queue: คิว / HN / ชื่อ-สกุลผู้ป่วย / ชื่อแพทย์).
- Keyboard: `Enter` commits the current value (and triggers `onSubmit` → history push), `Esc` blurs the input which closes the dropdown.
- Outside click closes via `document.addEventListener('mousedown')` guard inside the component.
- `mousedown.preventDefault()` on rows prevents blur firing before click.

**Row hover state (Figma list-item spec):** Every dropdown row (history + suggestion + empty-state filler) inherits the canonical list-item tokens so the whole row — surface, label, sublabel, leading icon — animates to the brand tint together on `:hover`:

| Element | Default token | Hover token | Notes |
|---|---|---|---|
| Surface | `surface/list/default` (`#FFFFFF`) | `surface/list/hover` (`primary/50`, brand-aware) | applied to the `<button>` row |
| Label | `text/list/default` (`#4D5358`) | `text/list/hover` (`primary/700`, brand-aware) | `font-weight: medium` |
| Sublabel | `text/content/tertiary` (`#858C92`) | `text/list/hover` | tertiary in resting tone, joins the brand color on hover |
| Leading icon | `text/content/tertiary` | `text/list/hover` | uses `currentColor` + `group-hover:` so the icon animates with the label |
| Transition | `transition-colors duration-150 ease-out` on each of icon, label-row, sublabel | — | one cubic-bezier so the row morphs cohesively |

> ⚠️ **Specificity trap:** Tailwind's `hover:text-...` cannot override an inline `style.color`. Use a `text-[color:var(...)]` *class* for the resting tone too — see [`SuggestionRow`](src/components/search/PrimarySearch.jsx) for the canonical pattern. The button must carry the `group` class so `group-hover:` cascades to nested icons / sublabels.

**Secondary Search state transitions:**

```
default ──mouseenter──▶ hover ──mouseleave──▶ default
default / hover ──focus──▶ typing  (border 2px em700 + focus ring 4px rgba(8,167,104,.15))
typing ──blur──▶ filled (has value) | default (empty)
filled / typing ──× click──▶ typing (cleared)
typing / filled ──Esc──▶ default (cleared + blurred)
```

**Full implementation (Primary Search):**

```javascript
(function() {
  var DATA = [
    { id: 'HN 1234567', name: 'นาย สมชาย ใจดี',  dept: 'อายุรกรรม' },
    { id: 'HN 2345678', name: 'น.ส. มานี รักดี',  dept: 'สูตินรีเวช' },
    // … more entries
  ];

  var STYLE = {
    default: { bdc: 'var(--em600)', sh: '',                                      ic: 'var(--em600)' },
    hover:   { bdc: 'var(--em700)', sh: '0 1px 4px rgba(8,167,104,.10), …',      ic: 'var(--em700)' },
    typing:  { bdc: 'var(--em700)', sh: '0 1px 4px rgba(8,167,104,.10), …',      ic: 'var(--em700)' },
    filled:  { bdc: 'var(--em600)', sh: '0 1px 4px rgba(8,167,104,.10), …',      ic: 'var(--em600)' },
  };

  function setState(st) {
    var s = STYLE[st];
    box.style.borderColor = s.bdc;
    box.style.boxShadow   = s.sh;
    ico.style.color       = s.ic;
    // swap right side
    var isTyping = st === 'typing';
    avatarsWrap.style.display = isTyping ? 'none' : 'flex';
    submitBtn.style.display   = isTyping ? 'flex'  : 'none';
  }

  // Hover
  wrap.addEventListener('mouseenter', () => { hovering = true;  if (state === 'default') setState('hover'); });
  wrap.addEventListener('mouseleave', () => { hovering = false; if (state === 'hover')   setState('default'); });

  // Focus → Typing + open dropdown
  input.addEventListener('focus', () => { renderDD(); openDD(); });

  // Real-time filtering
  input.addEventListener('input', () => { if (!open) openDD(); renderDD(); });

  // Keyboard navigation (Headless UI Combobox pattern)
  input.addEventListener('keydown', (e) => {
    if (e.key === 'ArrowDown') { e.preventDefault(); setActive(Math.min(activeIdx + 1, items.length - 1)); }
    if (e.key === 'ArrowUp')   { e.preventDefault(); setActive(Math.max(activeIdx - 1, 0)); }
    if (e.key === 'Enter')     { e.preventDefault(); activeIdx >= 0 ? pickItem(items[activeIdx].name) : closeDD(true); }
    if (e.key === 'Escape')    { input.value = ''; closeDD(false); input.blur(); }
  });

  // Outside click
  document.addEventListener('click', (e) => {
    if (open && !wrap.contains(e.target)) closeDD();
  });

  // Item mousedown must preventDefault to stop blur firing before click
  item.addEventListener('mousedown', (e) => e.preventDefault());
  item.addEventListener('click', () => pickItem(val));

  // Blur guard (timeout lets dropdown clicks fire first)
  input.addEventListener('blur', () => {
    setTimeout(() => { if (open) closeDD(); }, 160);
  });
})();
```

**Full implementation (Secondary Search):**

```javascript
// State map
var STYLE = {
  default: { bdc: 'var(--n300)',  bdw: '1px', sh: '' },
  hover:   { bdc: 'var(--em700)', bdw: '2px', sh: '0 0 0 4px rgba(8,167,104,.15)' },
  typing:  { bdc: 'var(--em700)', bdw: '2px', sh: '0 0 0 4px rgba(8,167,104,.15)' },
  filled:  { bdc: 'var(--n300)',  bdw: '1px', sh: '' },
};

box.addEventListener('mouseenter', () => { if (state === 'default') setState('hover'); });
box.addEventListener('mouseleave', () => { if (state === 'hover')   setState('default'); });
input.addEventListener('focus',  () => setState('typing'));
input.addEventListener('blur',   () => setState(input.value.trim() ? 'filled' : (hovering ? 'hover' : 'default')));
input.addEventListener('keydown', (e) => { if (e.key === 'Escape') { input.value = ''; input.blur(); } });

// Clear button
clearBtn.addEventListener('mousedown', (e) => e.preventDefault());   // keep focus
clearBtn.addEventListener('click', () => { input.value = ''; input.focus(); setState('typing'); });
```

---

### 6.26.5 Icon Reference

**Search icon — exact Figma path (node I2056:4073;2114:2805)**

The MIH search icon is a **filled compound path** (not stroked), rendered in a `16.667 × 16.667` coordinate space inside a `20 × 20` container.

```svg
<!-- 20px (Primary Search left icon) -->
<svg width="20" height="20" viewBox="0 0 16.667 16.667" fill="currentColor">
  <path fill-rule="nonzero" d="M13.333 7.5C13.333 4.278 10.722 1.667 7.5 1.667C4.278 1.667 1.667 4.278 1.667 7.5C1.667 10.722 4.278 13.333 7.5 13.333C9.075 13.333 10.503 12.708 11.553 11.694C11.573 11.668 11.596 11.643 11.619 11.619C11.643 11.596 11.668 11.573 11.694 11.553C12.708 10.503 13.333 9.075 13.333 7.5ZM15 7.5C15 9.271 14.385 10.897 13.359 12.18L16.423 15.244C16.748 15.57 16.748 16.097 16.423 16.423C16.097 16.748 15.57 16.748 15.244 16.423L12.18 13.359C10.897 14.385 9.271 15 7.5 15C3.358 15 0 11.642 0 7.5C0 3.358 3.358 0 7.5 0C11.642 0 15 3.358 15 7.5Z"/>
</svg>

<!-- 16px (Secondary Search / dropdown item icon) -->
<svg width="16" height="16" viewBox="0 0 16.667 16.667" fill="currentColor">
  <path fill-rule="nonzero" d="M13.333 7.5C…15 7.5Z"/>
</svg>
```

**Submit (arrow-up) icon — Typing state circle button**

```svg
<svg width="16" height="16" viewBox="0 0 24 24" fill="none">
  <path d="M12 19V5M12 5L5 12M12 5l7 7"
        stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
</svg>
```

**Clear (×) icon — Secondary Search Typing/Filled**

```svg
<svg width="14" height="14" viewBox="0 0 14 14" fill="none">
  <path d="M11 3L3 11M3 3l8 8"
        stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
</svg>
```

| Icon | Figma node | Size | Fill type | Color token |
|------|-----------|------|-----------|-------------|
| Search | `I2056:4073;2114:2805` | 20 / 16px | Filled path | `icon/primarySearch/*` or `icon/secondarySearch/*` |
| Arrow Up | `I2103:633;2103:550;325:2603` | 16px | Stroke, white | `icon/brandPrimary/on-brand` = `#FFFFFF` |
| Clear ×  | — | 14px | Stroke | `var(--n400)` = `#ADB2B7` |

---

### 6.26.6 Live Demo Summary

The preview file (`mih-components-all-v2.html → Search tab`) includes two fully interactive demos:

**Primary Search:**
- Click the bar → transitions to Typing, dropdown opens with 8 patient records
- Type Thai name / HN / department → real-time filtering with highlight
- `↑↓` arrow keys navigate items, `Enter` picks, `Esc` clears
- Clicking an item → transitions to Filled, input shows selected name
- Click outside → closes dropdown, persists Filled/Default based on value
- State badge + token chips update live with each transition

**Secondary Search:**
- Focus → Typing (2px border + 4px focus ring)
- Type anything → `×` clear button appears
- Clear or `Esc` → back to Default
- Blur with value → Filled
- Size toggle: 40px (default) / 36px (small)

---

### 6.28 Treatment Form — Step 1 Medical History (CPOE)

> **Figma:** [PHCIS | CPOE · node `176:41635`](https://www.figma.com/design/cOjwwHvgGURA3IEuls8RT5/PHCIS-%7C-CPOE?node-id=176-41635)  
> **Implementation:** `TreatmentFormShell.jsx` · `TreatmentSteps.jsx` (`Step1History`) · `ServiceDateList.jsx`

#### Layout

| Region | Component | Notes |
|---|---|---|
| Page shell | `TreatmentFormShell` | Single unified card (`--dim-radius-400`, `--shadow-card-strong`): framed bilingual `StepProgress` on top, body below, sticky footer |
| Left rail (316 px) | `ServiceDateList` | Figma node [`176:38629`](https://www.figma.com/design/cOjwwHvgGURA3IEuls8RT5/PHCIS-%7C-CPOE?node-id=176-38629) — topic + `ActionList` rows (`gap` 0, row radius 6px); selected: `--surface-list-hover`, 4px left `--text-brandPrimary-default`, chevron visible; badge on any row with `center` |
| Right panel | `Step1History` | `SectionTitle` · `TabCapsule` `variant="filter"` ([`176:38632`](https://www.figma.com/design/cOjwwHvgGURA3IEuls8RT5/PHCIS-%7C-CPOE?node-id=176-38632)) · vitals [`176:38640`](https://www.figma.com/design/cOjwwHvgGURA3IEuls8RT5/PHCIS-%7C-CPOE?node-id=176-38640) · อาการป่วย · pain · allergy [`176:38678`](https://www.figma.com/design/h0fghZdYHA7HRJcjmRsUmF/PHCIS-ALL?node-id=2624-122777) · chronic [`176:38700`](https://www.figma.com/design/h0fghZdYHA7HRJcjmRsUmF/PHCIS-ALL?node-id=2624-122777) |

#### CPOE step labels (bilingual)

| # | Thai | English subtitle |
|---|---|---|
| 1 | ประวัติการรักษา | Medical History |
| 2 | คัดกรองและซักประวัติ | Screening |
| 3 | ตรวจร่างกาย | Physical Examination |
| 4 | วินิจฉัย | Diagnosis |
| 5 | สั่งการรักษา | Treatment Order |
| 6 | นัดหมาย | Appointment |
| 7 | ใบรับรองแพทย์ | Medical Certificate |

`StepProgress` accepts `steps` as `string[]` or `{ title, subtitle }[]`; subtitles use `--text-steps-subtitle`.

#### Token aliases — Step 1 body

| Element | Figma token | CSS var |
|---|---|---|
| Rail title | `text/topic/tertiary-title` | `--text-topic-tertiary-title` |
| List row default | `surface/list/default` | `--surface-list-default` · label `--text-list-default` |
| List row hover | `surface/list/hover` | `--surface-list-hover` · `--text-list-hover` |
| List row selected (at rest) | `surface/list/hover` + 4px `primary/700` left | `--surface-list-hover` · label stays `--text-list-default` · chevron visible |
| List row selected-hover | `surface/list/selected-hover` + left accent | `--surface-list-selected-hover` · keeps left border + chevron |
| Rail divider (no drop shadow) | `border/brandPrimary/quaternary` right edge | `var(--dim-stroke-100) solid var(--border-brandPrimary-quaternary)` — [PHCIS \| Appointment `4045:13522`](https://www.figma.com/design/7zBAxKlVPzTJJY9joje5WB/PHCIS-%7C-Appointment?node-id=4045-13522) |
| Row radius | 6px | `--sm-radius-100` |
| Section heading | `text/topic/primary-title` | `--text-brandPrimary-default` (via `SectionTitle`) |
| Vitals panel (`176:38640`) | `VitalsSignPanel` | Outer: padding `space/400` (16px), gap `space/300` (12px), radius `radius/500` (16px), `--surface-neutral-quaternary`, border `--border-neutral-quaternary`; title `--text-topic-tertiary-title`; header→action gap `space/400` (16px); two inner rows on `--surface-card-100`, radius `radius/400` (12px), py `space/300`; tile dividers 1×74px; row 2 pads empty 5th column |
| Allergy / chronic (`176:38678` · `176:38700`) | `SummaryPanel` + `HistoryInnerCard` | Outer same as vitals (`titleVariant="tertiary"`, `radius="lg"`); inner card `--surface-card-100`, p `space/600`, gap `space/600`, radius `radius/400`; fields alias `Information` (2550:10442); severity/probability/status pills: `<ColorfulBadge width="hug" size="small" hierarchy="secondary">` — width follows label |
| Vitals panel border | `border/neutral/quaternary` | `--border-neutral-quaternary` |
| Tab selected | `surface/tabcapsule/selected` | `--surface-tabcapsule-selected` |
| BP chip | `surface/statusbadge/warning` | `--surface-statusbadge-warning` · `StatusBadge` style `warning` |

#### ServiceDateList props

| Prop | Type | Default |
|---|---|---|
| `title` | string | `วันที่รับบริการ` |
| `items` | `{ id?, date, center? }[]` | `[]` |
| `selectedIndex` | number | `0` |
| `onSelect` | `(index) => void` | — |
| `width` | number | `316` |

---

### 6.29 Treatment Form — Step 3 Physical Examination

> **Figma:** [PHCIS-ALL · node `2431:213971`](https://www.figma.com/design/h0fghZdYHA7HRJcjmRsUmF/PHCIS-ALL?node-id=2431-213971)  
> **Implementation:** `TreatmentSteps.jsx` (`Step3Examination`) · `BodyDiagramPanel.jsx` · `ExamSupportList.jsx` · `ExamNotesPanel.jsx`

#### Layout sections

| # | Section | Components |
|---|---|---|
| 1 | Topic | `SectionTitle` — ตรวจร่างกาย (Physical Examination) |
| 2 | Vitals | `VitalsSignPanel` (Figma CPOE [`176:38640`](https://www.figma.com/design/cOjwwHvgGURA3IEuls8RT5/PHCIS-%7C-CPOE?node-id=176-38640)); BP uses `--text-danger-default` + `StatusBadge` warning; **ดูแบบกราฟ** → `VitalsSignGraphDialog` / `VitalsGraphPopup` (`extraLarge`, max **1440×1024**) |
| 3 | Body map | `BodyDiagramPanel` + `ExamSupportList` (320px); **บันทึกร่างกาย** → `BodyRecordPopup` (`extraLarge`, max **1440×1024**) |
| 4 | Notes | `ExamNotesPanel` — nurse copy column + doctor `Textarea` + `DragAndDrop` |

#### BodyDiagramPanel

| Prop | Type | Default |
|---|---|---|
| `systemKey` | string | `heent` |
| `viewKey` | string | `both` |
| `markers` | `{ id, side, leftPercent, topPercent, note?, color? }[]` | `[]` — `color` is hex from `DRAW_COLORS`; all marks update when sidebar swatch changes |
| `onSystemChange` | fn | — |
| `onViewChange` | fn | — |

System tabs: `EXAM_SYSTEM_TABS` (Heent · Heart · Lungs · Abdomen · Extremities). View: `TabBar` + `TabPill` (ทั้งสองด้าน · ด้านหน้า · ด้านหลัง · …). Anatomy assets: `public/illustration/body-anatomy-front.png`, `body-anatomy-back.png` (848×1264, exported from Figma `image group_man` 77:44466 · frame Tools 223:29724). Display: 294×672 per figure, `object-cover`, gap 64px (`--dim-space-1600`).

#### DrawingToolSidebar (Figma 77:44504)

> **Figma:** [PHCIS \| Medical-examination · Drawing Tool](https://www.figma.com/design/lZFdHsmNE4OSgF1SBtZAHK/PHCIS-%7C-Medical-examination?node-id=77-44504)  
> **Implementation:** `BodyDrawingCanvas.jsx` (`DrawingToolSidebar`, `DRAW_TOOLS`) · annotations `bodyDrawingAnnotations.jsx`

Each tool button uses `Tooltip` (position **top**) with a concise Thai action verb on hover/focus (Figma 285:28636). Default `activeTool` is `cursor` — opens in inspect-only mode so the user can hover existing marks without accidentally placing a new one. **Clicking the already-active tool toggles back to `cursor`** (except `cursor` itself, which isn't toggleable).

| Tool key | Icon (Material Symbols) | Tooltip | Behaviour |
|---|---|---|---|
| `cursor` | `arrow_selector_tool` (Figma 75:32677) | เลือก | Mouse-pointer mode. Hover any marker / drawing → hover state. Click → select (blue **selection frame** with corner + edge resize handles). Drag → move. Color swatch / size slider edit the selected element in place. Re-clicking the active tool toggles back to cursor; no toggle effect when already on cursor. |
| `select` | `trip_origin` (Figma 285:28450) | มาร์คจุด | Markpoint placement. Click on the body → drops a new marker; opens `MarkNoteEditor` for note entry. |
| `pen` | `edit` | วาดเส้น | Freehand stroke — width = **ขนาดแปรง**. |
| `eraser` | `ink_eraser` | ลบ | Removes any stroke/shape under the cursor; radius = **ขนาดแปรง**. Custom eraser icon follows the pointer. |
| `text` | `title` | พิมพ์ข้อความ | Click → inline text input. Click an existing text drawing to re-edit. |
| `circle` | `circle` | วาดวงกลม | Drag → ellipse. Stroke width = **ความกว้างเส้น**. |
| `square` | `square` | วาดสี่เหลี่ยม | Drag → rectangle. |
| `arrow` | `north_east` | วาดลูกศร | Drag → arrow with equilateral triangle head. |
| `ruler` | `straighten` | วัดระยะ (ซม.) | Drag → straight measuring line with end caps, scale ticks (1 / 5 / 10 cm step, whichever keeps ticks ≥ 8 px apart; major tick every 5 cm at 1-cm step, otherwise every 2nd tick) and a `NN.N ซม.` label with a white halo. Scale = figure body height (96.4 % of the image) ÷ `figureHeightCm` (default `BODY_FIGURE_HEIGHT_CM` = 170; pass the patient's real height through `DrawingCanvasPopup` / `BodyDrawingCanvas` for true-to-patient readings). Same select / move / resize / erase / slider (`strokeWidth`) behaviour as `arrow`. |
| `image` | `add_photo_alternate` | เพิ่มรูปภาพ | Opens a file picker (not a draw mode); the image drops centred on the board and the tool switches to cursor for move / resize. |

**ล้างกระดาน** clears markers + drawings for the current view. Floating toolbar **เลิกทำ / ทำซ้ำ** drives the undo/redo snapshot stack (markers + drawings). Hand/pan tool (`arrow_selector_tool` on the left zoom rail) is mutually exclusive with the draw tools — entering pan clears the right-rail focus.

**Selection chrome (cursor mode only):** The blue **selection frame** (Figma reference) only appears when the **cursor** tool is active and an object is selected. The frame has:
- 4 visible corner handles (`nwse-resize` / `nesw-resize` cursors) — free-ratio resize.
- 4 invisible edge hit-zones (`ns-resize` / `ew-resize` cursors) — single-axis stretch.
- No size label.

Switching to the markpoint tool clicks-to-place; switching to a drawing tool hides the frame but keeps the underlying selection so it reappears when the user returns to cursor.

**Selection / move / delete / z-order (`BodyImageGroup`):** With cursor or markpoint active, a board-level capture layer hit-tests markers, pen strokes, shapes, and text (`buildAnnotationLayers` + `findTopAnnotationLayer`). Click to select; drag to move; **Delete** or **Backspace** removes the selected marker or drawing. Each new marker/drawing gets `stackOrder`; later actions render above earlier ones (`BoardDrawingLayer` / board marker overlay). Text tool: click existing text to edit. While a mark note editor is open, clicking empty board dismisses the editor instead of placing a new marker.

**Sidebar slider override (cursor mode + selected drawing):** When a drawing is selected, the right-rail size slider rebinds from the per-tool default to the drawing's own size field (`pen → width`, `circle/square/arrow/ruler → strokeWidth`, `text → fontSize`). Editing the slider mutates that drawing in place + commits to history. Picking a color swatch with a drawing selected recolors the same drawing.

#### Tab isolation (per-system markers + drawings; per-view drawings)

> **Implementation:** `TreatmentSteps.jsx` (Step 3) · `BodyDiagramPanel.jsx` · `BodyDrawingCanvas.jsx`

Two orthogonal isolation rules keep annotations scoped to where they were authored:

1. **Per-system (HEENT / Heart / Lungs / Abdomen / Extremities)** — `form.bodyMarkers` and `form.bodyDrawings` are both `Record<systemKey, …[]>`. The panel reads the active system's slice and writes back only to that slot, so switching system tabs swaps annotation sets atomically. Defaults: `DEFAULT_BODY_MARKERS = {}` and `DEFAULT_BODY_DRAWINGS = {}` (both exported from `data/treatmentVisits.js`).
2. **Per-view (ทั้งสองด้าน / ด้านหน้า / ด้านหลัง / ทั้งสองข้าง / ด้านซ้าย / ด้านขวา)** — every drawing is stamped with a `viewKey` at creation (`handleAddDrawing`). The render path filters by `!d.viewKey || d.viewKey === viewKey`, so strokes made on the front view don't appear on the back view. Markers are NOT tagged — they use their existing `side` (`'front'`/`'back'`) field and the standard front/back figure rendering rules.

Legacy drawings without a `viewKey` (anything saved before this rule landed) render on every view for backward compatibility — they'll naturally adopt a `viewKey` next time they're recreated.

#### BodyMark (Figma 217:27084)

> **Figma:** [PHCIS \| Medical-examination · mark](https://www.figma.com/design/lZFdHsmNE4OSgF1SBtZAHK/PHCIS-%7C-Medical-examination?node-id=217-27084)  
> **Implementation:** `BodyDrawingCanvas.jsx` — `BodyMark`, `MarkNoteEditor`, `BodyMarkNoteField`, `BodyMarkNotePopover`, `getMarkNoteSaveDisabled`

| Figma state | Node | Runtime | Behaviour |
|---|---|---|---|
| Default | `220:27727` | `BodyMark` `state="default"` + `color` | 24×24 px dot · swatch fill **60%** · 2px white border (opaque) |
| Hover | `220:27777` | `state="hover"` | **32×32** px · swatch fill **80%** (`rgba` via `resolveMarkDotBackground`) · **4px** border |
| Focus — Empty | `217:27083` | `MarkNoteEditor` + empty note | Dot opacity **0.8** + `VoiceTextarea` `220:27484` (447×160) · **บันทึก** disabled (`--surface-disabledButton-tertiary`) |
| Focus — Type | `231:28146` | `MarkNoteEditor` + typed note (new mark) | Same shell · **บันทึก** enabled when note is non-empty (`saveRequiresValue`) |
| Focus — Re-open | — | `MarkNoteEditor` + `baselineNote` | Click saved mark again · note shown but **บันทึก** disabled until text differs from `markerEditBaseline` (trimmed) · `getMarkNoteSaveDisabled` |
| Active | `220:27806` | Read-only hover on saved mark | 24×24 dot + `BodyMarkNotePopover` (`fit-content`, **max 480px**, `--text-modal-subtext`) |

**Mark color:** `DrawingToolSidebar` swatches (`DRAW_COLORS`) set `activeColor`. New marks store `color` on place; changing swatch updates **all** placed marks via `handleColorChange`. `resolveMarkDotBackground` applies fill alpha per state (default **60%**, hover/focus **80%**, active **60%**); border stays fully opaque.

**Mark note field:** `BodyMarkNoteField` → `VoiceTextarea` with `showSaveButton`. Save rules (`getMarkNoteSaveDisabled`): **new mark** — disabled until non-empty; **re-open saved mark** — disabled until edited vs baseline captured on select (`markerEditBaseline` in `BodyDrawingCanvas`). **บันทึก** closes the editor (`onMarkerSave`). Library: `BodyMarkStatesPreview` (6 states) · `BodyMarkReeditLogicPreview` (interactive re-open).

**Tokens:** `--surface-pink-default` / `--surface-pink-default-hover`, `--border-image-default`, `--shadow-dropdown` (popover), input focus ring `primaryBorder/600`.

#### ExamSupportList

`ActionList` rows with `circleIcon`, `showSuccess` when `hasResult`, `showChevronRight`. Title uses `--text-topic-secondary-title`.

#### ExamNotesPanel

| Column | Width | Behaviour |
|---|---|---|
| บันทึกจากพยาบาล | 448px | Read-only rows + `BrandCircleSubdueIconButton` copy to doctor |
| บันทึกของแพทย์ | flex | `SuggestibleTextarea` (cc · pi · pe · duration) + attachments — autocomplete pool per field: `CHIEF_COMPLAINT_SUGGESTIONS` · `PRESENT_ILLNESS_SUGGESTIONS` · `PHYSICAL_EXAM_SUGGESTIONS` · `DURATION_SUGGESTIONS` (`minChars: 1` for duration, `3` elsewhere). Popover anchors 4 px below the typed glyph row — see §6.12.10b. |

Form state keys: `nurseExam`, `doctorExam`, `examSystem`, `examView`, `bodyMarkers`, `examAttachments`.

---

## §6.26b PatientCard — sticky / parallax header

> **Component:** [`PatientCard`](src/components/card/PatientCard.jsx)
> **Figma:** `150:33649` (full default) · `343:1926` (compact mini gradient row) · `343:5629` (compact + alert row) · `343:1288` (source 44 px avatar; product aliases it to 40 px)
> **Library demo:** [/components/patient-card](src/pages/components/ComponentsLibraryPage.jsx)
> **Used by:** [TreatmentFormPage](src/pages/treat/TreatmentFormPage.jsx), [ScreeningPage](src/pages/opd/ScreeningPage.jsx)

The treatment / screening header shrinks as the user scrolls so the patient identity stays visible without occupying half the viewport. The card owns the chrome end-to-end — pages opt in with one prop.

### Modes

| Prop | Behaviour |
|---|---|
| _(none)_ | Static **full** layout adapted from Figma 150:33649. Avatar, identity, HN/VIP, and clinical fields share one horizontal row; the field group scrolls horizontally when space is constrained. |
| `compact` | Static **mini** layout (Figma 343:5629). Single row, 40 px avatar, 24h HN/VIP pills, four right-aligned summary fields. Use in the Components Library to freeze the stuck state. |
| `sticky` | **Parallax** — PatientCard renders a 1 px sentinel above itself, observes intersection with the viewport (offset by `stickyTop`), and toggles `compact` automatically. The card is wrapped in `position: sticky; top: stickyTop; z-index: 1100` so it pins under the app header (z-1200) while page content scrolls past. |
| `stickyTop` | Pixel offset for the sticky top edge. Defaults to **64** to clear the PHCIS app `Header` (h-16, sticky, z-1200). |

### What changes between full and compact

| Element | Full (default) | Compact (stuck) |
|---|---|---|
| Outer wrapper | `pt-12 pb-16 px-16` | `pt-12 pb-0 px-16` (alert row carries its own pb) |
| Back · title · actions row | visible, `max-height 56 / opacity 1` | collapsed: `max-height 0 / opacity 0 / pointer-events: none / aria-hidden` |
| Gradient card | row · no-wrap · `p-12` · gap 12 | row + flex-wrap · `px-12 py-4` · gap 12 |
| Avatar | 56 × 56 (border-box, white ring) | 40 × 40 (smaller alias of Figma 343:1288) |
| HN / VIP pills | h-24 · `px-8` · font 12 | h-24 · `px-8` · font 12 |
| Fields | same row as avatar: เพศ · อายุ · เลขบัตรประชาชน · หมู่เลือด · ประเภท · แพทย์ · สิทธิ์ (horizontal scroll when constrained) | row, right-aligned: สิทธิ์ · หมู่เลือด · ประเภท · แพทย์ |
| `แพ้ยา` / `โรคประจำตัว` row | label **stacked above** badges (`flex-col`, gap 4) | label **inline with** badges (`flex-row flex-wrap`, gap 4) |
| Initials fallback font | 20 px | 16 px |
| Clinical field values | Sarabun Regular (`--type-weight-regular`) | same |
| History action | `BrandSubdueButton sm` after the โรคประจำตัว group → `MedicalHistoryPopup` | same |

### Transition

All resizes share **`300ms cubic-bezier(0.4, 0, 0.2, 1)`** so the parallax reads as one motion:

- `padding-top`, `padding-bottom`, `gap`, `flex-direction` on the gradient card
- `width` / `height` on the avatar (border-box)
- `font-size` on the initials fallback
- `height`, `padding`, `font-size` on the HN / VIP pills
- `max-height`, `opacity`, `margin-bottom` on the title row
- `margin-top`, `padding-bottom` on the alert row

### Internal contract

The sentinel is a 1 px aria-hidden div placed **immediately before** the card in the DOM. `IntersectionObserver` watches it against the viewport with `rootMargin: '-${stickyTop}px 0px 0px 0px'`, so the swap fires the instant the card slides under the app header. When the observer reports `intersectionRatio === 0`, internal `stuck` state becomes true and the card derives `compact = stuck`. The page never needs to manage refs, observers, or sticky CSS.

Z-index: **1100** — above page content (default 0) and dropdown popovers (`--sm-z-popover 1199`) but below the app `Header` (z-1200), `Sidebar` (`--sm-z-overlay 1300`), modals (`--sm-z-modal 1400`), and tooltips (`--sm-z-tooltip 1500`).

### Allergy + chronic disease overlay

`drugAllergies` and `chronicDiseases` (each `{ name, severity }`) render the `AlertBadgeColumn` row below the gradient card and remain visible across the parallax swap so clinicians never lose sight of safety flags. Severity → ColorfulBadge style via [`resolveSeverityBadgeStyle`](src/components/badge/clinicalBadgeStyles.js): `critical → red`, `high → orange`, `medium → blue`, `low → teal`, `mild → yellow`.

---

## §6.27 Brand Color Theme System

MIH Design System รองรับ **10 brand color modes** ที่สามารถเปลี่ยนได้ real-time โดยไม่ต้อง reload ทุก component ที่ alias จาก `primary/*` จะ cascade อัตโนมัติผ่าน CSS custom properties

---

### 6.27.1 Architecture

```
Figma Brand Collection
  └─ primary/50 … primary/700  ← per-mode aliases
        ↓
CSS :root  (--em50 … --em700, --primaryBorder-600, --PrimaryShadow-*)
        ↓
Semantic CSS variables  (--surface-brandPrimaryButton-default, --border-input-hover, …)
        ↓
Component CSS classes   (.btn-brand-fill, .select-box.state-hover, .st-circle--current, …)
```

The theme switcher calls `setBrandTheme(key)` which updates ~80 CSS custom properties via `document.documentElement.style.setProperty()`. Because all components alias through these variables, the entire UI responds instantly.

---

### 6.27.2 The 10 Brand Modes

Sourced from the MIH token JSON files + the latest Figma palette updates (2026-05-12). Four modes were retired in favor of softer palettes — see §6.27.2a below for the migration map.

| Key | Name | `/50` | `/100` | `/200` | `/400` | `/600` | `/700` |
|---|---|---|---|---|---|---|---|
| `default` | Default (Emerald) | `#F4FBF8` | `#DFF2E9` | `#B1E6CD` | `#6AD6A5` | `#08A768` | `#007549` |
| `soft-sky` | Soft Sky | `#F5FBFF` | `#E8F5FD` | `#D6ECFA` | `#A8D3EE` | `#8EC4E5` | `#5594C4` |
| `soft-peach` | Soft Peach | `#FFFAF7` | `#FEF0E8` | `#FDE2D0` | `#FBBEA0` | `#E88A66` | `#D0714D` |
| `rose-mist` | Rose Mist | `#FFF9F9` | `#FCEEED` | `#F8E0DF` | `#EEBFBE` | `#D49A99` | `#BC8584` |
| `sakura` | Sakura | `#FFF8FA` | `#FFEDF3` | `#FFE0EC` | `#FFC0D4` | `#E89DBA` | `#D687A8` |
| `soft-lavender` | Soft Lavender | `#FAF7FF` | `#F2ECFF` | `#E5D9FF` | `#C5B2F5` | `#9D87D8` | `#836FC0` |
| `blue-serenity` | Blue Serenity | `#F0F2FF` | `#E4E8FF` | `#D0D7FF` | `#A5B3FC` | `#7A8CF0` | `#6676E0` |
| `muted-teal` | Muted Teal | `#F0F8F7` | `#DAEEED` | `#B5DBD9` | `#7AABA8` | `#5A7D7A` | `#486462` |
| `warm-gray` | Warm Gray | `#F5F3F1` | `#EBE7E4` | `#D5CEC9` | `#A39A94` | `#726760` | `#5C504A` |
| `dusty-mauve` | Dusty Mauve | `#F9F4F5` | `#F2E6E8` | `#E4CDD0` | `#B58E94` | `#7E5A60` | `#664750` |

Each palette maps directly to `primary/50 … primary/700` in the Figma Brand collection. The `default` (Emerald) mode is the standard MIH brand.

#### 6.27.2a Migration map (2026-05-12)

The following `[data-brand="..."]` selectors were renamed/replaced. The new palettes come from the primitive palettes added to §1a-2.

| Retired key | Replaced by | Notes |
|-------------|-------------|-------|
| `sky-cyan` | `soft-sky` | softer mid-tones, less saturated blue |
| `pink` | `sakura` | feminine cherry-blossom pink |
| `rose` | `rose-mist` | muted mauve/cocoa rose |
| `cool-lavender` | `soft-lavender` | airier purple, less inky deep tones |
| `soft-peach` (legacy) | `soft-peach` (updated) | same key, retuned palette (orange → tan/cocoa) |

Consumers reading `localStorage('phcis.theme')` with a retired key should be migrated proactively (one-time map on read). The `BrandThemeSwitcher` in `/components` and `SettingsModal` already expose the new keys.

---

### 6.27.3 CSS Variables Updated per Theme

| CSS Variable | Figma Token | Role |
|---|---|---|
| `--em50` | `primary/50` | Lightest tint — list hover bg, actionlist hover, etc. |
| `--em100` | `primary/100` | Light tint — button tertiary bg, calendar hover, stepper btn hover |
| `--em200` | `primary/200` | Medium tint — button tertiary hover, step connector |
| `--em400` | `primary/400` | Mid — badge, tag accents |
| `--em600` | `primary/600` | Primary — border hover/open (input, select, search) |
| `--em700` | `primary/700` | Dark — button fill, text primary, selected surface |
| `--primaryBorder-600` | `primaryBorder/600` | `rgba(primary/600, 0.15)` — 4px focus ring on Select, Step, Search, Form Builder |
| `--PrimaryShadow-600` | `PrimaryShadow/600` | Button fill drop shadow (8% opacity) |
| `--PrimaryShadow-601` | `PrimaryShadow/601` | Button fill drop shadow (10% opacity) |
| `--focus-ring-brand` | — | Button focus ring (25% opacity) |
| `--focus-ring-input-hover` | `focus-ring/input/hover` | Input field hover glow (14% opacity) |
| `--focus-ring-input-typing` | `focus-ring/input/typing` | Input field typing glow (14% opacity) |
| `--focus-ring-stepper` | `focus-ring/stepper` | Stepper btn focus glow (14% opacity) |
| `--surface-brandPrimaryButton-default` | `surface/brandPrimaryButton/default` | Button fill bg |
| `--surface-brandPrimaryButton-default-hover` | `surface/brandPrimaryButton/default-hover` | Button fill hover bg |
| `--surface-brandPrimaryButton-tertiary` | `surface/brandPrimaryButton/tertiary` | Button tertiary bg |
| `--surface-brandPrimaryButton-tertiary-hover` | `surface/brandPrimaryButton/tertiary-hover` | Button tertiary hover |
| `--surface-calendar-hover` | `surface/calendar/hover` | Calendar day hover bg |
| `--surface-calendar-selected` | `surface/calendar/selected` | Calendar day selected bg |
| `--icon-calendar-hover` | `icon/calendar/hover` | Calendar nav icon hover |
| `--surface-stepper-btn-hover` | `surface/brandprimarybutton/quaternary-hover` | Stepper ± button hover |
| `--surface-timeslot-hover` | `surface/timeslot/hover` | Timeslot slot hover |
| `--surface-timeslot-selected` | `surface/timeslot/selected` | Timeslot slot selected |
| `--surface-tooltip` | `surface/tooltip/default` | Tooltip background |
| `--surface-actionlist-hover` | `surface/actionList/hover` | Action list row hover bg |
| `--text-actionlist-primary-hover` | `text/list/hover` | Action list row text hover |
| `--text-actionlist-topic-title` | `text/topic/secondary-title` | Action list topic title |
| `--icon-actionlist-primary-hover` | `icon/list/primary-hover` | Action list leading icon hover |
| `--icon-actionlist-secondary-hover` | `icon/list/secondary-hover` | Action list trailing icon hover |
| `--icon-positive-success` | `icon/positive/success` | ActionList SuccessIcon (TickOnCircle) — fixed teal-500 `#24a899` |
| `--border-input-hover` | `border/input/hover` | Input / Select border hover |
| `--border-input-typing` | `border/input/typing` | Input border typing |
| `--icon-input-hover` | `icon/input/hover` | Input leading icon hover |
| `--icon-input-typing` | `icon/input/typing` | Input leading icon typing |
| `--surface-pagination-selected` | `surface/pagination/selected` | Pagination active page bg |
| `--text-pagination-selected` | `text/pagination/selected` | Pagination active page text |
| `--surface-datepicker-nav-hover` | `surface/datepicker/nav-hover` | Date picker nav hover |
| `--icon-datepicker-nav-hover` | `icon/datepicker/nav-hover` | Date picker nav icon hover |
| `--surface-list-hover` | `surface/list/hover` | Dropdown / Select / Topic list item hover bg |
| `--surface-steps-default` | `surface/steps/default` | Step circle default bg |
| `--surface-steps-hover` | `surface/steps/hover` | Step circle hover bg |
| `--surface-steps-selected` | `surface/steps/selected` | Step circle selected (current) bg |
| `--border-steps-default` | `border/steps/default` | Step connector line |
| `--text-steps-default` | `text/steps/default` | Step number text |
| `--text-steps-title` | `text/steps/title` | Step title default (50% opacity brand) |
| `--text-steps-title-hover` | `text/steps/title-hover` | Step title hover |
| `--text-steps-title-select` | `text/steps/title-select` | Step title selected |
| `--text-topic-primary-title` | `text/topic/primary-title` | Topic primary title color |
| `--text-topic-secondary-title` | `text/topic/secondary-title` | Topic secondary title color |
| `--text-topic-primary-hover` | `text/topic/primary-hover` | Topic title hover color |
| `--icon-topic-default` | `icon/topic/default` | Topic icon default color |
| `--icon-topic-hover` | `icon/topic/hover` | Topic icon hover color |
| `--surface-topic-hover` | `surface/topic/hover` | Topic card hover bg |
| `--surface-badge-green` | `surface/colorfulBadge/green` | Colorful badge green bg |
| `--text-badge-green` | `text/colorfulBadge/green` | Colorful badge green text |
| `--surface-formbuilder-default` | `surface/formBuilder/default` | Form Builder card bg |
| `--border-formbuilder-default` | `border/formBuilder/default` | Form Builder dashed border — default (`primary/200`) |
| `--border-formbuilder-hover` | `border/formBuilder/hover` | Form Builder dashed border — hover (`primary/600`) ★ brand-responsive |
| `--border-formbuilder-filled` | `border/formBuilder/filled` | Form Builder filled card border (`primary/100`) |
| `--fb-stroke` | `dimension/stroke/150` | Form Builder border width = 1.5 px (Figma `2210:7836`) |
| `--icon-formbuilder-iconbox` | `icon/formBuilder/iconBox` | Form Builder icon box bg |
| `--icon-formbuilder-default` | `icon/formBuilder/default` | Signature pen icon color |
| `--icon-formbuilder-content` | `icon/formBuilder/content` | Content "Type" icon color |
| `--text-formbuilder-text` | `text/formBuilder/text` | Form Builder card title |
| `--surface-card-brand100` | `surface/card/brand-100` | Selected-Add card bg |
| `--text-navigation-title` | `text/navigation/title` | Header/Sidebar nav title color |
| `--surface-sidemenu-active` | `surface/sidemenu/active` | Side menu active item bg |
| `--surface-sidemenu-hover` | `surface/sidemenu/hover` | Side menu hover item bg |
| `--icon-sidemenu-hover` | `icon/sidemenu/hover` | Side menu icon hover color |
| `--surface-brandtag-hover` | `surface/brandTag/default-hover` | Brand tag hover bg |
| `--border-brandtag-hover` | `border/brandTag/default-hover` | Brand tag hover border |
| `--text-brandtag-hover` | `text/brandTag/hover` | Brand tag hover text |
| `--surface-brandtag-selected` | `surface/brandTag/selected` | Brand tag selected bg |
| `--border-brandtag-selected` | `border/brandTag/selected` | Brand tag selected border |
| `--text-brandtag-selected` | `text/brandTag/selected` | Brand tag selected text |
| `--surface-brandtag-selected-hover` | `surface/brandTag/selected-hover` | Brand tag selected-hover bg |
| `--border-brandtag-selected-hover` | `border/brandTag/quaternary` | Brand tag selected-hover border |
| `--text-brandtag-selected-hover` | `text/brandTag/selected-hover` | Brand tag selected-hover text |

---

### 6.27.4 Components That Respond Automatically

When `setBrandTheme(key)` is called, the following components update with no additional code:

Button · Form Input · Select · Tab Underline · Tab Capsule · Tag (Brand) · Interactive Tag Chip (Brand) · Dropdown Menu · Form Builder (card · signature · submit button · toast · drag-drop zone) · Badge (green fill + outline) · Icon Badge (green) · Pagination · Radio (badge + circle icon) · Step Indicator (all states) · Calendar · Chevron Button · Checkbox (via `--em*`) · Date Picker · Stepper · Timeslot · Tooltip · Action List (hover + success badge text) · Topic (title + icon + hover bg) · Select list hover · Search (Primary + Secondary — all border states) · Header · Side Menu · Footer

---

### 6.27.5 JavaScript API

```javascript
// Switch theme — all components update immediately
setBrandTheme('emerald');   // default MIH green
setBrandTheme('teal');
setBrandTheme('blue');
setBrandTheme('indigo');
setBrandTheme('violet');
setBrandTheme('rose');
setBrandTheme('orange');
setBrandTheme('amber');
setBrandTheme('pink');
setBrandTheme('sky');

// Internally: hex2rgba helper
function hex2rgba(hex, alpha) {
  var r = parseInt(hex.slice(1,3),16);
  var g = parseInt(hex.slice(3,5),16);
  var b = parseInt(hex.slice(5,7),16);
  return 'rgba(' + r + ',' + g + ',' + b + ',' + alpha + ')';
}
```

Selection persists across page reloads via `localStorage.setItem('mih-brand-theme', key)`.

---

### 6.27.6 UI — Theme Switcher Strip

Located in the fixed top bar (right side). 10 circular color swatches, 20×20px. Active swatch has a dark ring border + inner white inset ring. Tooltip shows mode name on hover.

```html
<!-- Top bar theme switcher -->
<div class="theme-switcher">
  <span class="theme-switcher-label">Brand</span>
  <button class="theme-swatch active" id="ts-emerald"
    title="Emerald (Default)" style="background:#08A768"
    onclick="setBrandTheme('emerald')"></button>
  <!-- …9 more swatches… -->
</div>
```

```css
.theme-swatch { width:20px; height:20px; border-radius:50%; border:2px solid transparent; }
.theme-swatch.active { border-color:#363b3f; }
.theme-swatch.active::after { content:''; position:absolute; inset:2px; border-radius:50%; border:2px solid #fff; }
```

#### React implementation — `<BrandThemeSwitcher>` (in `/components` preview)

**Source:** [src/pages/components/ComponentsLibraryPage.jsx](src/pages/components/ComponentsLibraryPage.jsx) (search for `BrandThemeSwitcher`)

A working version of the swatch strip lives in the design-system preview's TopBar. Clicking a swatch flips `<html data-brand="...">` and persists to `localStorage('phcis.theme')`, which is the **same key** Sidebar's `SettingsModal` reads — so the theme carries over to the production PHCIS shell on navigation.

Swatch colors (representative `brand-p600` per mode, pulled from index.css after the 2026-05-12 palette migration — see §6.27.2a):

| Mode | Swatch hex |
|------|-----------|
| `default` (Emerald) | `#08A768` |
| `soft-sky` | `#8EC4E5` |
| `soft-peach` | `#E88A66` |
| `rose-mist` | `#D49A99` |
| `sakura` | `#E89DBA` |
| `soft-lavender` | `#9D87D8` |
| `blue-serenity` | `#7A8CF0` |
| `muted-teal` | `#5A7D7A` |
| `warm-gray` | `#726760` |
| `dusty-mauve` | `#7E5A60` |

**Visual rules:**
- 20×20 circles in a pill container (4 px padding · 24 px radius capsule · `surface/card/200` bg · 1 px `border/card/default`)
- Active: `2 px solid text/content/default` outline + `1 px white` inner ring + `scale(1.0)`
- Inactive: 2 px transparent border + `scale(0.9)`
- Hover (any swatch): `scale(1.05)` + 160 ms `cubic-bezier(0.32, 0.72, 0, 1)` transition

**Behavior:**
- On mount, reads `localStorage.phcis.theme` (defaults to `'default'`) and applies via `document.documentElement.setAttribute('data-brand', key)`. Passing `'default'` removes the attribute (lets `:root` win).
- `aria-radiogroup` semantics — `role="radio"` per swatch, `aria-checked` reflects active.
- `title` on each swatch shows the mode label for hover-tooltip.

---

### 6.27.7 Token: `primaryBorder/600`

This Brand-collection token is the single source for **all 4px focus/glow rings** in the system.

| CSS Variable | `--primaryBorder-600` |
|---|---|
| Figma token | `primaryBorder/600` (Brand collection) |
| Variable ID | `VariableID:2933:17188` |
| Formula | `rgba(primary/600, 0.15)` |
| Default value | `rgba(8, 167, 104, 0.15)` |
| Used by | Select hover+open · Secondary Search hover+typing · Step current outline · Form Builder drag-active zone |

In Figma, this token has a resolved value per brand mode (e.g. Pink mode = `rgba(219,39,119,0.15)`). In CSS, it is computed dynamically inside `setBrandTheme()` using `hex2rgba(p[600], 0.15)`.

---

### 6.28 Topic

**Node:** `2334-1841` · Figma library: **MIH Design System Foundation / Semantic**  
**Source:** [src/components/topic/Topic.jsx](src/components/topic/Topic.jsx)

> Section header row. Comes in 5 hierarchies × 3 states (states apply to Primary only). Primary lays out as a card row with leading icon + bold title + subtext + trailing badge / success / chevron. Secondary / Tertiary / *-on are text-only blocks.

#### 6.28.1 Component Properties (Figma 1:1)

| Property | Values |
|----------|--------|
| Hierarchy | `primary` · `secondary` · `tertiary` · `secondary-on` · `tertiary-on` |
| State (Primary only) | `default` · `hover` · `disabled` |
| Show Sub Text | bool |
| Show Badge | bool — trailing green ColorfulBadge |
| Show Chevron (Primary) | bool — `keyboard_arrow_up` 24px |
| Show Leading Icon (Primary) | bool — 24×24 Material Symbol |
| Show Success | bool — 24×24 brand-p700 TickOnCircle |

#### 6.28.2 Dimension Tokens (Primary card row)

| Property | Token | Value |
|----------|-------|-------|
| Horizontal padding | `dim/space/300` | 12 px |
| Vertical padding | `dim/space/200` | 8 px |
| Row gap | `dim/space/300` | 12 px |
| Icon / Title gap | `dim/space/200` | 8 px |
| Title / Subtext gap | `dim/space/050` | 2 px |
| Container radius | `dim/radius/400` | 12 px |
| Leading icon size | `dim/size/icon-md` | 24 px |
| Chevron icon size | `dim/size/icon-md` | 24 px |
| Success icon size | `dim/size/icon-md` | 24 px |

#### 6.28.3 Color Tokens

**Primary** (state-driven)

| Role | Default | Hover | Disabled |
|------|---------|-------|----------|
| Surface | `--surface-topic-default` → `#FFFFFF` | `--surface-topic-hover` → `brand-p50` | `--surface-topic-disabled` → `#F9FBFB` |
| Title | `--text-topic-primary-title` → `brand-p700` | `--text-topic-primary-hover` → `brand-p700` | `--text-topic-primary-disabled` → `#ADB2B7` |
| Subtext | `--text-topic-primary-subtext-default` → `#858C92` | `--text-topic-primary-subtext-hover` → `#858C92` | `--text-topic-primary-subtext-disabled` → `#C9CDD0` |
| Icon / Chevron | `--icon-topic-primary-default` → `brand-p700` | `--icon-topic-primary-default` → `brand-p700` | `--icon-topic-primary-disabled` → `#C9CDD0` |

**Secondary / Tertiary** (text-only)

| Role | Variable | Hex | Primitive |
|------|----------|-----|-----------|
| Secondary title | `--text-topic-secondary-title` | `#007549` | brand-p700 |
| Tertiary title | `--text-topic-tertiary-title` | `#363B3F` | neutral-800 |
| Subtext (off-surface) | `--text-topic-tertiary-subtext` | `#ADB2B7` | neutral-400 |
| Subtext (-on tinted bg) | `--text-topic-tertiary-subtext-on` | `#858C92` | neutral-500 |

#### 6.28.4 Typography

| Role | Family | Size | Weight | Line-height |
|------|--------|------|--------|-------------|
| Title | `type/family/heading-content` (fallback body) | `type/size/base` 16 px | `type/weight/bold` 700 | 1.5 |
| Subtext (Primary) | `type/family/body` | `type/size/sm` 14 px | `type/weight/regular` 400 | 1.5 |
| Subtext (Secondary/Tertiary) | `type/family/body` | `type/size/xs` 12 px | `type/weight/medium` 500 | 1.5 |

#### 6.28.5 Interactive Behavior

- **Click target.** Primary is the only hierarchy that accepts `onClick`. When `onClick` is provided and the row is not disabled, the container takes `role="button"` + `tabIndex=0` + pointer cursor.
- **Hover.** CSS `:hover` is handled via internal `useState(internalHover)`; pass `state` to force the visual for the design-system matrix.
- **Focus.** Hover state also applies on focus (keyboard parity). Browser focus-rings are not added; rely on the surface tint and your parent focus-visible policy.
- **Disabled.** `aria-disabled="true"`, cursor `not-allowed`, surface neutral-50, title/subtext dimmed, click handler skipped.
- **Brand-aware.** All `brand-p*` aliases retint automatically under `[data-brand]`.

#### 6.28.6 Do / Don't

✅ **Do**
- Use Primary for *expandable* row headers inside cards (Vitals, Lab Result, Allergies). Pair the chevron with a real disclosure handler.
- Use Secondary for in-card section titles where the surface is white.
- Use `*-on` variants when the topic sits on a `brand-p50` or neutral tinted surface (the subtext gets darker for contrast).
- Provide a meaningful `leadingIcon` from Material Symbols (`face`, `medication`, `health_metrics`, etc.) — the icon is the primary visual anchor.

🚫 **Don't**
- Don't apply `state="hover"` permanently — it's a design-system grid affordance only.
- Don't nest a Topic inside a clickable card and add `onClick` on both — pick one click target.
- Don't use Tertiary on a tinted brand surface — the subtext fades into the background. Use `tertiary-on` instead.
- Don't change `showChevron` based on hover alone; the chevron should reflect *expanded* state, not pointer position.

---

### 6.29 Information

**Node:** `2550-10442` · Figma library: **MIH Design System Foundation / Semantic**  
**Source:** [src/components/information/Information.jsx](src/components/information/Information.jsx)

> Compact key/value block. Single hierarchy (`secondary`) and single state (`default`) in Figma. Used for stat readouts inside patient cards, queue rows, schedule chips — a tertiary subtext label above the topic, plus optional trailing green badge and an optional inline red Badge2 below the text.

#### 6.29.1 Component Properties (Figma 1:1)

| Property | Values |
|----------|--------|
| Hierarchy | `secondary` (only variant in Figma) |
| State | `default` (only variant in Figma) |
| Show Sub Text | bool — xs medium tertiary label on top |
| Show Text | bool — sm regular default topic |
| Show Badge | bool — trailing **green** ColorfulBadge (no dot/icon) |
| Show Badge2 | bool — inline **red** ColorfulBadge (with dot + icon) below the text |

#### 6.29.2 Dimension Tokens

| Property | Token | Value |
|----------|-------|-------|
| Column gap (text · trailing badge) | `dim/space/400` | 16 px |
| Sub Text → Text gap | `dim/space/100` | 4 px |
| Text → Badge2 gap | `dim/space/100` | 4 px |
| Max width | — | `maxWidth` prop (default 640) |

#### 6.29.3 Color Tokens

| Role | CSS Variable | Hex | Primitive |
|------|-------------|-----|-----------|
| Sub Text | `--text-content-tertiary` | `#858C92` | neutral-500 |
| Text | `--text-content-default` | `#363B3F` | neutral-800 |
| Badge | inherits `<ColorfulBadge style='green'>` | — | green / brand emerald |
| Badge2 | inherits `<ColorfulBadge style='red'>` | — | red palette |

#### 6.29.4 Typography

| Role | Family | Size | Weight | Line-height |
|------|--------|------|--------|-------------|
| Sub Text | `type/family/body` | `type/size/xs` 12 px | `type/weight/medium` 500 | 1.5 |
| Text | `type/family/body` | `type/size/sm` 14 px | `type/weight/regular` 400 | 1.5 |
| Badge / Badge2 | inherits ColorfulBadge (size `small`) | — | — | — |

#### 6.29.5 Interactive Behavior

- **Static.** Information is presentational — no hover, click, focus, or keyboard handling. It renders a single layout regardless of pointer state.
- **Truncation.** Each text node uses `whiteSpace: nowrap`. Wrap the parent with `min-w-0` and a fixed width if you need ellipsis behavior at narrow widths.
- **Brand-aware.** Trailing green badge tints with `[data-brand]` because it aliases the brand palette. Badge2 (red) is non-brand and stays red across all themes.

#### 6.29.6 Do / Don't

✅ **Do**
- Stack multiple Information rows with `gap: var(--dim-space-300)` for a compact stat panel (`HN`, `วันนัด`, `แผนก`).
- Use Badge2 sparingly for *alert* metadata (allergy flag, emergency room) — its red dot draws attention.
- Use translated Thai labels for the SubText to match the rest of the app.

🚫 **Don't**
- Don't wrap Information in an `<a>` or `<button>` — it's not an action. Use Topic (Primary) when you need a clickable row.
- Don't put both badges (`showBadge` + `showBadge2`) on every row — the red Badge2 loses urgency if every entry uses it.
- Don't override `text` color to a brand color — the topic is meant to read neutral-800 against any surface.

---

### 6.30 Component Interaction & Usage Guidelines

A central reference for *behavior, accessibility, and usage rules* across recently-shipped components. Token tables and dimensions live in each component's own section; this table is for the **rules of engagement**.

> The same content is mirrored inside each component demo on `/components` via the `<Guidelines>` block, so designers and engineers reading either source see the same do's and don'ts.

#### 6.30.1 Tooltip ([6.8](#68-tooltip) · `src/components/tooltip/Tooltip.jsx`)

**Behavior**
- Opens on `mouseenter` / `focusin` of the wrapped child after a 200 ms `delay` (debounce — prevents flashes on rapid cursor passes).
- Closes on `mouseleave`, `focusout`, or `Escape`.
- Portals into `document.body` with `position: fixed`; re-measures on `resize` and capture-phase `scroll`. The caret tip sits exactly `ANCHOR_GAP = 8 px` from the anchor edge regardless of `position`.
- The interactive wrapper uses `display: inline-block; line-height: 0` on the anchor span so `getBoundingClientRect()` matches the child's actual edges.

**Do**
- Wrap interactive controls (icon buttons, chips) when the label would otherwise be missing or truncated.
- Keep `content` to one short line; the bubble uses `whiteSpace: nowrap` and a `maxWidth` clamp.
- Prefer `position="top"` for toolbar icons; `position="right"` for trailing affordances inside lists.

**Don't**
- Don't put critical information (errors, required actions) inside a tooltip — hover/focus is not reachable on touch and not announced by all screen readers.
- Don't wrap a non-focusable element (plain `<span>`, `<div>` without `tabIndex`) — keyboard users won't get the tooltip.
- Don't disable `delay` for decoration — the 200 ms debounce is what keeps the system from feeling twitchy.

#### 6.30.2 Topic ([6.28](#628-topic) · `src/components/topic/Topic.jsx`)

**Behavior**
- Hover and focus both produce the hover visual; the component manages an internal `internalHover` flag.
- `onClick` makes the row a real button (`role="button"`, `tabIndex=0`); without `onClick` the row is presentational.
- `state` prop forces a visual for the design-system grid and *suppresses* internal hover/focus tracking.
- `disabled` short-circuits hover, focus, and click.

**Do** — see [6.28.6](#6286-do--dont).  
**Don't** — see [6.28.6](#6286-do--dont).

#### 6.30.3 Information ([6.29](#629-information) · `src/components/information/Information.jsx`)

**Behavior**
- Static. No event handlers. Renders identically across pointer states.

**Do / Don't** — see [6.29.6](#6296-do--dont).

#### 6.30.4 Timeslot ([6.7](#67-timeslot) · `src/components/timeslot/Timeslot.jsx`)

**Behavior**
- A real `<button>` — receives keyboard focus and fires `onClick` on `Enter` / `Space`.
- `disabled` appends `disabledLabel` (default `"(เต็ม)"`) to the value and sets `aria-disabled="true"`. The button is also natively `disabled`.
- `selected` exposes `aria-pressed="true"` (toggle-button semantics).
- Hover is tracked internally unless `state` is passed.

**Do**
- Use inside a horizontally-scrolling row for booking grids; group by date with a heading above each row.
- Pair with `disabledLabel="(เต็ม)"` (Thai) or `"(Full)"` (English) per locale.
- Make selection single-choice unless your booking flow explicitly supports multi-select; mirror radio semantics if single.

**Don't**
- Don't use Timeslot for non-time chips (status filters, departments) — use `FilterTag` or `ColorfulBadge`.
- Don't render disabled slots invisible — keeping them visible communicates *capacity*. Just dim them and append the suffix.
- Don't force `state="selected"` on multiple slots in a single-select group.

#### 6.30.5 Switch ([6.6](#66-switch) · `src/components/switch/Switch.jsx`)

**Behavior**
- `role="switch"` + `aria-checked` — announced as "switch" by screen readers.
- Click and `Enter` / `Space` both toggle. Controlled (`checked` + `onChange`) or uncontrolled (`defaultChecked`).
- Disabled suppresses click, hover, and the focus-driven hover visual.

**Do**
- Use for boolean settings that *take effect immediately* (notifications on/off, brand mode preview).
- Pair with a leading label so screen readers can announce the setting name.

**Don't**
- Don't use Switch when a form submit is required to commit the change — use Checkbox in a form instead. Switch implies "applies now."
- Don't toggle Switch programmatically based on remote state without showing a saving indicator — users expect immediate confirmation.
- Don't use Switch inside a list of three or more mutually-exclusive options — use Radio.

#### 6.30.6 Stepper ([6.5](#65-stepper) · `src/components/stepper/Stepper.jsx`)

**Behavior**
- Numeric input flanked by `−` / `+` buttons. The `−` button is disabled when `value <= min`; `+` is disabled when `value >= max`.
- Direct keyboard input is allowed; non-numeric characters are filtered. `Enter` blurs the input.
- Browser focus indicators are suppressed (`caretColor: transparent`, `outline: none`) — focus is communicated through container border-color only.

**Do**
- Set `min={0}` for quantities; the boundary-aware `−` button prevents negatives without explicit validation.
- Use Small (36 h × 112 w) inside compact form cells; Large (40 h × 128 w) for primary booking flows.

**Don't**
- Don't use Stepper for arbitrary integer ranges (year selection, age) — use `InputField` with `type="number"` and explicit min/max.
- Don't omit `min` if your value model can't represent negatives — the boundary check needs both bounds.

#### 6.30.7 Calendar / DatePicker ([6.2](#62-calendar) · [6.4](#64-date-picker))

**Behavior**
- DatePicker's trigger inherits Select's state contract (`default · hover · open · filled · error · disabled`). Clicking toggles the Calendar popup.
- Calendar arrow-keys move between days; `Enter` commits selection; `Escape` closes the popup.
- Today is marked with the brand-p* outline; selected day uses the brand-p* surface.

**Do**
- Pass an explicit `format` (`DD/MM/YYYY` or `YYYY-MM-DD`) per locale — don't rely on browser defaults.
- Disable past dates with `minDate={new Date()}` for booking flows.

**Don't**
- Don't ship DatePicker without a clear-X affordance in the trigger — users need a way to reset.
- Don't use Calendar standalone for *range* selection without explicit `start`/`end` props — the single-day visual will mislead.

#### 6.30.8 Dropdown ([6.14](#614-dropdown-menu) · `src/components/forms/Dropdown.jsx`)

**Behavior**
- Trigger uses Select-style state machine; the panel renders rows from the `<DropdownRowPreview>` configuration (icon · switch · badge · check · chevron).
- Keyboard: `↑` / `↓` move focus, `Enter` activates, `Escape` closes, `Tab` closes and moves focus to the next element.

**Do**
- Use Dropdown when the user is *picking one or more* values from a constrained list.
- Show a leading icon only when it disambiguates the choice (e.g. brand modes); otherwise rely on the label.

**Don't**
- Don't put destructive actions (delete account, drop table) inside a Dropdown without a confirmation step.
- Don't use Dropdown for navigation — the chevron sets the expectation of *value selection*. Use a menu (`ActionList`) for navigation.

#### 6.30.9 Step / Stepper Indicator ([6.18](#618-step-step-indicator))

**Behavior**
- Visual progress indicator — no internal state. Parent flow tracks `current` and renders Steps accordingly.
- Hover/focus on a *clickable* step (provide `onClick`) shows the hover visual; non-clickable steps are static.

**Do**
- Make completed steps clickable (back-navigation); make future steps non-clickable.
- Pair with a sticky header so the user can see progress while scrolling the active step's form.

**Don't**
- Don't allow skipping forward in a linear flow by clicking future steps.
- Don't combine "current" + "success" on the same step — pick one.

#### 6.30.10 Form Builder · Content / Signature / DragAndDrop ([6.20](#620-form-builder))

**Behavior**
- **Content** — dashed palette chip. Drag-handle on the left; clickable. `state="hover"` exposes the hover visual for docs.
- **Signature** — empty pad enters drawing mode on click; strokes saved as base64 PNG to `onChange`. `Clear` resets.
- **DragAndDrop** — dropzone with native file input fallback. Validates `accept` and `maxSize`; renders Filled `FileCard` rows with a remove `×`. `error` state shows a red border + message.

**Do**
- Show a saving spinner while Signature commits to the server; the canvas can produce large base64 strings.
- Allow multi-file in DragAndDrop only when the form expects an array; otherwise the second drop overwrites the first.

**Don't**
- Don't omit the `accept` prop on DragAndDrop — without it, users can upload anything and you'll catch errors server-side.
- Don't disable the Clear control on Signature — users *will* want to restart.

#### 6.30.11 Button Family ([5.1](#51-brand-button-primary)–[5.7](#57-icon-only-buttons))

**Behavior**
- Every button is a real `<button>`: keyboard focus, `Enter` / `Space` activation, `aria-disabled` honored on programmatic disable.
- Hover and focus apply the same hover tint (keyboard parity). Active (press) tint is one shade darker.
- Loading sets `aria-busy="true"`, hides the label, and renders a centered spinner; click handlers are suppressed until the parent clears `loading`.
- Icon-only variants require an `aria-label`; the icon is `aria-hidden` so screen readers don't announce it twice.

**Do**
- Use **BrandButton (Fill)** for the page's primary action — exactly one per surface.
- Use **BrandSubdueButton** or **NeutralSubdueButton** for secondary actions that share a row with the primary.
- Use **DangerButton** for destructive *commits* (Delete, Discard, Sign-out); pair with a confirm dialog.
- Use **Outline** variants on tinted brand surfaces where Fill would over-saturate.
- Use **Ghost** only for tertiary actions sharing a row with stronger buttons (cancel · close · "ดูเพิ่มเติม").

**Don't**
- Don't ship two Fill-primary buttons in the same row — pick the most important and demote the rest to Subdue / Outline / Ghost.
- Don't put a **Danger** button next to a **Brand** button without separation; users mis-click when red sits beside green.
- Don't use an **Icon-Only** button as the *only* trigger for a critical action — pair with a label or a Tooltip.
- Don't disable a button on validation error if you can show *why* it's disabled — surface the helper text instead, then enable.
- Don't change the button's text mid-press (e.g. "Save" → "Saving…") without entering `loading` state — the layout will jump.

#### 6.30.12 Checkbox ([6.3](#63-checkbox))

**Behavior**
- `role="checkbox"` + `aria-checked` (or `aria-checked="mixed"` for indeterminate).
- Click and `Enter` / `Space` toggle; `disabled` suppresses both.
- `indeterminate` is a *visual* property — controllers must manage the underlying boolean(s) and decide when to flip to fully-checked / unchecked.

**Do**
- Use a parent Checkbox with `indeterminate` for "select all" + per-row Checkboxes underneath.
- Use **CheckboxBox** when you need a *card-style* selectable surface (filters, addons) — the whole card is the hit target.
- Use **CheckboxButton** when the checkbox sits inline with action text (terms acceptance row, dialog confirmations, PHCIS Registration “เหมือนที่อยู่ตามทะเบียนบ้าน”). Follow Figma **2254:8344** card states (1px hover border + bottom shadow; 2px selected border + top shadow) — do not copy RadioButton inset-ring chrome. Still toggles on/off — not single-select radio.

**Don't**
- Don't use a Checkbox group when only one option is valid — use Radio.
- Don't use Checkbox for "applies immediately" toggles — use Switch.
- Don't wire `indeterminate` to "partially valid" form state — it should reflect partial *selection*, not partial *correctness*.

#### 6.30.13 Radio ([6.21](#621-radio))

**Behavior**
- `role="radio"` inside a `role="radiogroup"`; `aria-checked` on each option.
- Arrow keys move between options *within* a group (Left/Up = previous, Right/Down = next) and immediately select.
- `Tab` enters/leaves the group as a single stop; the selected radio is the tab-stop on re-entry.

**Do**
- Use Radio when exactly **one** answer is valid (gender, blood type, payment method).
- Always render a default selection unless "none" is semantically meaningful (then use a "ไม่ระบุ" radio).
- Use **RadioBox** for card-style mutually exclusive choices (rights, plan tiers) — same hit-target rules as CheckboxBox.

**Don't**
- Don't use Radio for boolean toggles — use Switch (immediate) or Checkbox (form-submit).
- Don't ship a Radio group with a single option — it's not a choice.
- Don't put two unrelated Radio groups inside the same `<RadioGroup name>` — they will share value state.

#### 6.30.14 Tab — Underline / Pill / Capsule ([6.15](#615-tab))

**Behavior**
- `role="tablist"` + `role="tab"` + `role="tabpanel"`; `aria-selected` flips on the active tab.
- Arrow keys move between tabs within the list; `Enter` / `Space` activate. Optionally auto-activate on arrow (recommended for navigation).
- Active tab gets the brand accent (underline / pill surface / capsule fill); disabled tabs are dim and unfocusable.

**Do**
- Use **TabUnderline** as the page-level navigation inside a section (queue / register / history).
- Use **TabPill** when tabs live inside a card and need a softer chrome.
- Use **TabCapsule** for filter-style tabs that toggle list content (status: all · today · pending).
- Keep tab labels short (≤2 words / 14 chars Thai); long labels collapse the layout.

**Don't**
- Don't mix Underline and Pill inside the same surface — pick one Tab family per screen.
- Don't use Tabs for fewer than 3 sections — use a SegmentedControl or two buttons instead.
- Don't use Tabs as a workflow stepper — use **Step / StepProgress**. Tabs imply parallel content, not sequence.

#### 6.30.15 Search — Primary / Secondary ([6.26](#626-search-figma-2056-4246))

**Behavior**
- **PrimarySearch** — 56px brand-bordered pill. Typing exposes the 40px X + submit controls; it opens a Combobox panel on focus. Panel content is condition-driven:
  - empty input → **recent-search history** (`history` prop, persist in `localStorage`; header `ค้นหาล่าสุด`, optional `ล้างประวัติ`)
  - typing (≥ 1 char) → **live suggestions** (`suggestions` prop) with rich rows (`label` + tertiary `sublabel`); falls back to `emptyMessage` when no match.
  - `history` / `suggestions` default to `[]` — pages must opt-in. `onSubmit` (Enter / suggestion click) is the hook for pushing the committed query into history.
- **SecondarySearch** — compact inline search with brand-tinted ring on hover/typing; clear X on filled.
- Both honor `Enter` to commit and `Escape` to clear.
- The Combobox panel uses standard `↑`/`↓`/`Enter` keyboard semantics.

**Do**
- Use PrimarySearch at the top of queue / register screens where search is the page's primary action.
- Use SecondarySearch for in-list filters (filter by name inside a long list).
- Always provide a `placeholder` describing the *fields* searched (e.g. คิว / HN / ชื่อ-สกุลผู้ป่วย / ชื่อแพทย์), and feed `suggestions` from the *same* fields so the dropdown matches the placeholder's promise.
- Persist `history` per-page (key like `mih:primarysearch-history:<page>`) and cap at ~8 entries to keep the popover compact.

**Don't**
- Don't ship two PrimarySearches on one page — there's only one "primary" search.
- Don't put SecondarySearch on a tinted surface without checking contrast — the neutral border can disappear.
- Don't strip the clear X — it's how touch users reset.
- Don't show fake placeholder suggestions when the page hasn't wired real data — leave `suggestions` empty so the dropdown stays closed.

#### 6.30.16 Pagination ([6.19](#619-pagination))

**Behavior**
- Renders page numbers + previous/next chevrons; current page is bold + brand-accented.
- Page numbers are real buttons; arrow keys move focus, `Enter` activates.
- Disabled state on the first page's "previous" and the last page's "next".

**Do**
- Pair Pagination with a "Showing X–Y of Z" label so users know the range.
- Use Pagination for *server-paged* lists; for short client-side lists, prefer infinite scroll or expand-all.

**Don't**
- Don't render Pagination if the result set fits on one page — hide it instead.
- Don't auto-advance the page on partial result loading — wait for the click.
- Don't use Pagination for non-list flows (wizard navigation) — use Step.

#### 6.30.17 Alert & SmallAlert ([6.13](#613-alert) · [6.13.1](#6131-smallalert))

**Behavior**
- Renders an icon + title + (optional) description and (optional) trailing actions.
- 4 tones: `info` (blue) · `success` (green) · `warning` (amber) · `danger` (red).
- Dismiss is opt-in (`onClose`); when present, a × button removes the alert and fires the callback.
- For accessibility, prefer `role="alert"` for *transient* notifications and `role="status"` for non-urgent confirmations.

**SmallAlert** (see [6.13.1](#6131-smallalert)): single-line pill with solid `surface/smallalert/*` (500-tier) and white on-surface text — use for *very* compact confirmations (e.g. clipboard), not for explanatory banners.

**Do**
- Use **danger** for blocking failures the user must act on; **warning** for non-blocking risks; **success** for transient confirmations.
- Put primary action(s) in the trailing slot (e.g. "ลองใหม่", "ดูรายละเอียด").
- Pair the icon with the tone — the icon is the first thing scanned.

**Don't**
- Don't stack more than 2 alerts on a single surface — they cancel each other's urgency.
- Don't use Alert for *form-field* errors — use the field's built-in error state.
- Don't auto-dismiss `danger` alerts; let the user dismiss after reading.

#### 6.30.18 Tag / Filter Tag ([6.16](#616-tag))

**Behavior**
- Small chip surface for metadata or filter state. Optional leading icon, optional close (×) button.
- Click on the body toggles selected; click on the × removes the chip and fires `onRemove`.
- Selected uses brand surface; unselected uses neutral surface.

**Do**
- Use **FilterTag** for *removable* applied filters above a list; show each active filter as a separate chip.
- Use **FilterDateTag** when a chip represents a date range; show the formatted range in the body.
- Group selected filter chips on one row and unapplied filters on another.

**Don't**
- Don't use Tag for actions — they look passive. Use a Button.
- Don't render Tag without an onRemove if the chip represents an applied filter — users need a way to undo.
- Don't mix selectable and removable tags in the same row — the affordances conflict.

---

### 6.31 SideSubMenu (Figma 299:3610)

**Source:** [src/components/layout/SideSubMenu.jsx](src/components/layout/SideSubMenu.jsx)

> Companion panel that opens to the right of `<Sidebar />` when a primary menu has its own sub-departments (e.g. all clinics under "ระบบบันทึกการรักษา"). Tinted brand-p50 shell, 384 px wide.

#### 6.31.1 Component Properties (Figma 1:1)

| Property | Values |
|----------|--------|
| Header | string · ReactNode — section title at top |
| Items | `[{ key, label, disabled? }]` — list of sub-departments |
| State (per item) | `default` · `hover` · `selected` · `disabled` |
| showBottomSpacer | bool — reserves Figma "Bottom" 112 px slot |
| visible *(Sidebar only)* | bool — drives slide-in/fade-out lifecycle |

#### 6.31.2 Container Tokens

| Property | Token | Value |
|----------|-------|-------|
| Width | `dim/size/2600` | 384 px |
| Background | `--surface-card-brand-100` | brand-p50 (#F4FBF8) |
| Right border | `dim/stroke/100` + `border/brandPrimary/quaternary` | 1 px brand-p100 |
| Shadow | `--shadow-sidemenu` | Brand Drop Shadow Bottom/200 |
| Padding top | `dim/space/600` | 24 px |
| Padding x | `dim/space/100` | 4 px |
| Padding bottom | `dim/space/0` | 0 |
| Item gap | `dim/space/150` | 6 px |

#### 6.31.3 Item · State Tokens

| Role | Default | Hover | Selected | Disabled |
|------|---------|-------|----------|----------|
| Surface | transparent | `--surface-sidesubmenu-hover` → brand-p100 | `--surface-sidesubmenu-selected` → brand-p100 | transparent |
| Text color | `--text-sidesubmenu-default` → brand-p700 | brand-p700 | `--text-sidesubmenu-selected` → brand-p700 | `--text-sidesubmenu-disabled` → #C9CDD0 |
| Font weight | regular 400 | regular 400 | **bold 700** | regular 400 |
| Font family | body | body | `heading-content` | body |
| Min height | 48 px | 48 px | 48 px | 48 px |
| Padding | 8 / 24 | 8 / 24 | 8 / 24 | 8 / 24 |
| Border radius | `dim/radius/200` 6 px | 6 px | 6 px | 6 px |

#### 6.31.4 Header (Section Title)

| Property | Value |
|----------|-------|
| Min height | 48 px (matches item row) |
| Color | `--text-neutral-tertiary` → #ADB2B7 |
| Font | body Medium, `--type-size-xs` 12 px |
| Padding | 8 / 24 |

#### 6.31.5 Interactive Behavior

- **Opens** when a primary `<Sidebar />` menu item with `subItems` is clicked. Currently only `treat` (ระบบบันทึกการรักษา) has sub-departments.
- **Toggles** on the same item — clicking the open primary menu again closes the panel.
- **Slide animation** (Sidebar wraps the panel): mount → next-frame flip to visible → `translateX(-16px) → 0` + `opacity 0 → 1` over 280 ms with `cubic-bezier(0.32, 0.72, 0, 1)`. On close: reverse, then unmount after the transition.
- **Outside click** (anywhere outside Sidebar + SubMenu) closes the panel.
- **Escape** closes the panel.
- **Scroll** — when items overflow the viewport, the panel's internal nav scrolls (`overflow-y: auto`) with a thin scrollbar; shell border + shadow stay fixed.

#### 6.31.6 Do / Don't

✅ **Do**
- Pair the header label with the parent primary menu name so users see where they are.
- Use `onSelect(key)` to drive the parent route (e.g. `/treat/anc`) and reflect the choice via `selectedKey`.
- Keep most-used departments at the top — users scan top-down.

🚫 **Don't**
- Don't nest a SideSubMenu inside a card — it's a full-height fixed panel, not a popover.
- Don't mark more than one item `selected` — bold heading-content font signals a single active page.
- Don't show a SideSubMenu for primary menus without sub-departments — close the panel instead.

---

### 6.32 Sidebar — Sub-Menu Integration (Figma 299:4913)

**Source:** [src/components/layout/Sidebar.jsx](src/components/layout/Sidebar.jsx)

Builds on the original Sidebar (§6.20) with two integrations:

#### 6.32.1 New menu items (Figma node 1:2363)

| Order | key | Label | Icon (Material Symbols) | Path / subItems |
|-------|-----|-------|-------------------------|-----------------|
| 1 | dashboard | แดชบอร์ด | `bar_chart_4_bars` | `/dashboard` |
| 2 | consent | คลังแบบฟอร์ม | `assignment_add` | — |
| 3 | mr | งานเวชระเบียน | `assignment` | `/mr/queue` |
| 4 | opd | ระบบงานผู้รับบริการนอก | `face` | `/opd/queue` |
| 5 | **treat** | **ระบบบันทึกการรักษา** | `stethoscope` | **subItems × 10** |
| 6 | radiology | แผนกรังสีวินิจฉัย | `radiology` | — |
| 7 | lab | ห้องปฏิบัติการ | `biotech` | — |
| 8 | pharmacy | ระบบงานเภสัชกรรม | `medication` | — |
| 9 | finance | งานการเงิน | `payments` | — |
| 10 | appoint | ระบบนัดหมาย | `calendar_clock` | — |
| 11 | log | ประวัติการใช้งาน | `history` | — |
| 12 | users | ผู้ใช้และสิทธิ์การเข้าถึง | `group` | — |

- Menu item **icon size = 16 px** (per Figma) — down from the previous 20 px.
- Notification bell also 16 px with 6 × 6 red dot (was 20 / 7).

#### 6.32.2 `treat` sub-departments (Figma 299:3610)

```
ตรวจรักษาทั่วไป (OPD)               → /treat/queue
แผนกส่งเสริมสุขภาพ(วางแผนครอบครัว)
แผนกโรคติดต่อทางเพศสัมพันธ์และโรคเอดส์
แผนกบำบัดและรักษาผู้ป่วยติดสารเสพติด
แผนกฝากครรภ์
แผนกส่งเสริมสุขภาพเฉพาะกลุ่มที่สำคัญ
แผนกสร้างเสริมภูมิคุ้มกันโรค
แผนกทันตกรรม
แผนกวัณโรค
แผนกสุขภาพจิตและคลินิกสุขภาพจิต
```

#### 6.32.3 Click flow

| Menu type | onClick behavior |
|-----------|------------------|
| With `subItems` | Toggle SideSubMenu open / closed. No navigation. The primary menu becomes "active" (brand-p50 surface + 4 px left border + chevron-right) while the panel is open. |
| With `path` only | Navigate + close any open SideSubMenu. |
| No `subItems`, no `path` | No-op. |
| Sub-item with `path` | Navigate + close SideSubMenu. |
| Sub-item without `path` | Update `selectedKey` in SideSubMenu only. |

#### 6.32.4 Hover-row smoothness (Sidebar items)

- 4 px left border is always present (transparent → brand-p700 on active) — no width shift between idle and active.
- `paddingLeft: 20` always (was conditional 20 vs 24) — keeps icon column stable.
- Transition `200 ms cubic-bezier(0.32, 0.72, 0, 1)` on `background-color` + `border-color`.

#### 6.32.5 Nav scroll-shadow (new)

When the menu items overflow the viewport, gradient fade overlays appear at the top/bottom of the nav (mirrors `<HistoryTimeline>` "ประวัติการวัด" pattern):

| Property | Value |
|----------|-------|
| Height | 4 px |
| Background (top) | `linear-gradient(to bottom, --PrimaryShadow-602, --PrimaryShadow-600, transparent)` |
| Background (bottom) | reverse |
| Opacity | toggled by scroll position (`scrollTop > eps` → top fade; `scrollTop < scrollHeight - clientHeight - eps` → bottom fade) |
| Transition | `opacity 150 ms` |
| `pointer-events` | none — never blocks clicks |

Brand-aware: gradient tints retint with `[data-brand]` because `--PrimaryShadow-*` resolves to `rgb(var(--brand-p600-rgb) / α)`.

---

### 6.33 Switch — Enhanced Interactions

**Source:** [src/components/switch/Switch.jsx](src/components/switch/Switch.jsx)

Updates layered on top of §6.6 (which remains the canonical token reference):

#### 6.33.1 Press stretch (iOS-style)

| State | Knob width | Knob X offset | Result |
|-------|-----------|---------------|--------|
| Idle (off) | 14 px | 0 | Standard disc at left |
| **Pressed (off)** | **18 px** | 0 | Stretched right toward destination |
| Idle (on) | 14 px | 14 px | Standard disc at right |
| **Pressed (on)** | **18 px** | **10 px** | Stretched left toward destination |

- Transitions: `transform 240 ms` + `width 220 ms` with `cubic-bezier(0.16, 1, 0.3, 1)` (premium ease-out spring).
- Tracked via `useState(pressed)`, set on `pointerdown` / cleared on global `pointerup` + `pointercancel` — gracefully handles drag-off-control cancellation.
- Keyboard parity: `Space` / `Enter` keydown sets pressed, keyup releases.

#### 6.33.2 Focus ring (keyboard-only)

| Property | Value |
|----------|-------|
| Box shadow | `0 0 0 3px var(--primaryBorder-600)` |
| Fallback | `rgba(8,167,104,0.30)` |
| Trigger | only when `e.target.matches(':focus-visible')` — hidden on click-focus |

Brand-aware: `--primaryBorder-600` retints to the active brand's hue at 15 % α.

#### 6.33.3 Hover knob shadow

Stacks a second elevation layer on hover: `0 1px 2px rgba(0,0,0,0.18), 0 2px 4px rgba(0,0,0,0.08)` (was single 0 1px 2px).

---

### 6.34 Table — Composable Atoms (Figma "MIH Design System · Table")

**Source:** [src/components/table/Table.jsx](src/components/table/Table.jsx)

Generic `<Table>` orchestrator (column-driven, sticky cols, drag-reorder) stays the canonical entry for queue/list pages. Six new **named exports** added for pixel-exact composition and design-system docs:

| Export | Figma | Property panel (1:1 with Figma) |
|--------|-------|---------------------------------|
| `HeaderCell` | 177:6297 | `state` (default/hover) · `showSorting` · `showSubtext` · `title` · `subtext` · `sortDirection` · `align` · `onClick` |
| `DataCell` | 20:1894 | `state` · 13 show flags: `showCheckbox` · `showProfile` · `showTextGroup` · `showText` · `showSubtext` · `showStepper` · `showInputField` · `showSelect` · `showBadge` · `showSwitch` · `showButton1` · `showButton2` · `showButton3` · live handlers: `onCheckChange` · `onSwitchChange` · `onButton{1,2,3}Click` |
| `FooterCell` | 41:6279 | `title` · `align` |
| `TableRow` | 1:277 | `state` (default/hover) · `onClick` — row-level hover container |
| `TableSelectAllRow` | 24:25939 | `label` · `cells` · `cellWidth` |
| `TableSummary` | 41:6764 | `cells: [{ title, width? }]` |

#### 6.34.1 HeaderCell Tokens

| Role | Default | Hover |
|------|---------|-------|
| Surface | `--surface-table-header` → brand-p50 | `--surface-table-hover` → brand-p100 |
| Title | `--text-table-title` → brand-p800 (#004C31), heading-content Bold xs (12px) | same |
| Subtext | `--text-table-subtext` → text-content-tertiary (#858C92), body Regular xs | same |
| Sort icon (idle) | `--icon-brandSecondary-quaternary` → muted gray | (hover) `--icon-brandPrimary-default` |
| Sort icon (sorted) | `--icon-brandPrimary-default` → brand-p700 | brand-p700 |
| Border bottom | `dim/stroke/200` (2 px) `--border-table-default` (brand-p100) | same |
| Min height | 44 px | 44 px |
| Padding | 8 / 16 | 8 / 16 |

Sort icon glyph: `unfold_more` (idle) → `expand_less` (asc) → `expand_more` (desc).

#### 6.34.2 DataCell Tokens

| Role | Default | Hover |
|------|---------|-------|
| Surface | `--surface-table-default` (white) | `--surface-table-hover` → brand-p50 |
| Border bottom | `dim/stroke/100` (1 px) `--border-table-line` (brand-p50) | same |
| Min height | 56 px | 56 px |
| Padding | 8 / 16 | 8 / 16 |
| Gap between slots | `dim/space/150` (6 px) | same |

**Slot composition** (every slot is the canonical design-system component — no duplicates):

| Slot | Component used |
|------|----------------|
| Checkbox | `<Checkbox>` (§6.3 / 6.30.12) |
| Profile | local 36 × 36 rounded avatar (no canonical yet) |
| TextGroup | local stacked text+subtext (no canonical yet) |
| Stepper | `<Stepper size='small'>` (§6.5 / 6.30.6) |
| InputField | `<InputField size='sm' showLabel={false} showHelperText={false}>` (§6.12) |
| Select | `<Select size='sm' showFieldName={false} showHelperText={false}>` (§6.25) |
| Badge | `<ColorfulBadge style='teal' size='default' hierarchy='secondary' showDot>` (§5.11) |
| Switch | `<Switch>` (§6.6 / 6.33) |
| Button1, Button2 | `<BrandSubdueIconButton variant='outline' size='sm'>` (§5.7) |
| Button3 | `<BrandIconButton variant='fill' size='sm'>` (§5.7) |

Icon-button clicks call `e.stopPropagation()` inline so they don't bubble to a row-level click.

#### 6.34.3 FooterCell Tokens

| Property | Value |
|----------|-------|
| Surface | `--surface-table-footer` → brand-p100 (#E2F3EB) |
| Title color | `--text-table-footer` → brand-p800 (#004C31) |
| Font | heading-content Bold sm (14 px) |
| Min height | 44 px |
| Padding | 8 / 16 |

#### 6.34.4 TableRow — Row-level Hover Propagation

`<TableRow>` tracks hover once on the row container, then propagates `state="hover"` down to every `DataCell` child via `React.cloneElement`. The whole row tints together instead of cell-by-cell — visually consistent and free of mid-row "flicker" as the cursor crosses column boundaries.

**Opt-out per cell:** pass `state="default"` (or any explicit value) on a `DataCell` — `TableRow` never overrides an already-set state. Use this for action columns where the row tint would compete with the cell's own controls.

**Default behavior:** action cells follow row hover; the `Switch` / `IconButton` slots stay interactive because their internal clicks call `e.stopPropagation()` and they have their own surfaces.

**onClick:** the row is keyboard-activatable (`role="button"`, `tabIndex=0`, `Enter` / `Space`). Clicks on descendant `<button>` / `<a>` / `<input>` are skipped via `e.target.closest(...)` so action buttons can fire without triggering row clicks.

#### 6.34.5 Do / Don't

✅ **Do**
- Use `HeaderCell` / `DataCell` / `FooterCell` when you need pixel-exact Figma composition (design-system docs, mockups).
- Use the generic `<Table>` with `column.render(row)` for data-driven lists — it handles sticky columns, drag-reorder, sort, and row-click.
- Wrap row cells in `<TableRow>` so hover is consistent across the row.
- Match column count in `TableSummary` to the underlying Table's columns so totals line up visually.

🚫 **Don't**
- Don't render TableSummary on a different total width than the main Table — it'll look misaligned on horizontal scroll.
- Don't apply `state="hover"` permanently — it's a docs affordance only; runtime resolves hover from pointer.
- Don't compose a separate inline switch / checkbox / badge / icon-button visual inside a custom cell — always alias the canonical design-system component (§6.34.7).

#### 6.34.6 Live Demos (`/components/table`)

| Demo | Behavior |
|------|----------|
| **Sorting** | Click a column header to cycle `null → asc → desc → null`. Rows re-sort live; sort icon flips glyph + color (icon-brandPrimary-default when active or hovered). |
| **Select-all + per-row** | Header has a master checkbox; each row has its own. Tri-state on header (empty / `–` / `✓`). |
| **Row actions** | Switch + 3 icon buttons per row. Whole row hovers together; individual actions stay clickable via `e.stopPropagation()`. |

#### 6.34.7 Canonical Component Aliasing (project-wide rule)

**Every place that uses a Checkbox / Badge / Button / Stepper / Switch / Form Input / Select MUST alias the canonical design-system component — never re-implement the visual inline.**

Replacements applied:

| File | Was | Now |
|------|-----|-----|
| `Table.jsx · DataCell` | local `SLOT.checkbox` static span | `<Checkbox>` |
| `Table.jsx · DataCell` | local `SLOT.badge` static span | `<ColorfulBadge>` |
| `Table.jsx · DataCell` | local `SLOT.iconButton` static span | `<BrandIconButton>` + `<BrandSubdueIconButton>` |
| `Table.jsx · DataCell` | local `SLOT.stepper` static span | `<Stepper>` |
| `Table.jsx · DataCell` | local `SLOT.inputField` static span | `<InputField>` |
| `Table.jsx · DataCell` | local `SLOT.select` static span | `<Select>` |
| `Table.jsx · DataCell` | local `SLOT.switch` static span | `<Switch>` |
| `Dropdown.jsx · DropdownRowPreview` | local `SwitchVisual` inline | `<Switch>` |
| `Dropdown.jsx · DropdownRowPreview` | local `BadgeVisual` inline | `<ColorfulBadge style='indigo'>` |
| `ActionList.jsx` | local `MiniSwitch` 44 × 24 | `<Switch>` (spread of `switchProps`) |

Benefit: brand-aware tokens, focus ring, press stretch, accessibility (`role`, `aria-*`), and all interaction polish flow from one source — switching themes or upgrading the design-system component automatically updates every consumer.

---

### 6.35 Product Components ที่ใช้ซ้ำทั่วระบบ (ลงทะเบียน 20 ก.ย. 69)

Component ชั้น product ที่เกิดใน vibe code แล้วถูกเรียกใช้ซ้ำหลายสิบหน้า (นับจากไฟล์ใน
`src/` ที่ import ณ 20 ก.ย. 69) แต่ยังไม่เคยอยู่ใน Components Library / เว็บ
bma-design-system.mih.co.th จึงลงทะเบียนพร้อมกันชุดนี้ ทุกตัว **alias token ของ
Foundation ทั้งหมด ไม่มีค่าใหม่** และเปิดดูได้ที่ `/components/<slug>`
กติกาเดิมของ [`docs/awaiting-components.md`](docs/awaiting-components.md) ยังใช้:
ยกขึ้น DS เมื่อมีผู้ใช้ตั้งแต่ 2 ที่ — ชุดนี้ทุกตัวเกินเกณฑ์ไปไกล

| # | Component | slug | ไฟล์ | ใช้ใน (ไฟล์) | ประกอบจาก |
|---|---|---|---|---:|---|
| 6.35.1 | ClearFiltersTag | `clear-filters-tag` | `src/components/forms/ClearFiltersTag.jsx` | 120 | FilterTag + motion tokens |
| 6.35.2 | SearchResultCount | `search-result-count` | `src/components/search/SearchResultCount.jsx` | 103 | text tokens |
| 6.35.3 | SuccessToast | `success-toast` | `src/components/toast/SuccessToast.jsx` | 100 | Small Alert · Success (139:20246) |
| 6.35.4 | OverflowFlexRow | `overflow-flex-row` | `src/components/overflow/OverflowFlexRow.jsx` | 67 | FilterTag + Popover |
| 6.35.5 | SelectableCardButton | `selectable-card-button` | `src/components/button/SelectableCardButton.jsx` | 38 | PHCIS-OPD 160:26432 |
| 6.35.6 | QueuePatientAvatar | `queue-patient-avatar` | `src/components/patient/QueuePatientAvatar.jsx` | 33 | Avatar §6.11 · Register Home 3891:7747 |
| 6.35.7 | MaskedWorklistName | `masked-worklist-name` | `src/components/privacy/MaskedWorklistName.jsx` | 43 | EllipsisText (Table) + Tooltip |
| 6.35.8 | PhcisTopToast | `phcis-top-toast` | `src/components/alert/PhcisTopToast.jsx` | 31 | SmallAlert (Appointment 19:8245 · OPD 323:58923) |
| 6.35.9 | UnsavedChangesPopup | `unsaved-changes-popup` | `src/components/popup/UnsavedChangesPopup.jsx` | 29 | AlertDialog · Warning (3106:136000) |
| 6.35.10 | RowActionMenu | `row-action-menu` | `src/components/overflow/RowActionMenu.jsx` | 21 | BrandIconButton + Popover |
| 6.35.11 | PatientSummaryCard | `patient-summary-card` | `src/components/card/PatientSummaryCard.jsx` | 18 | Popup Large 145:6155 · Consent Timeline 270:72714 |
| 6.35.12 | TimePicker | `time-picker` | `src/components/forms/TimePicker.jsx` | 16 | Select trigger (§6.25) + Popover |
| 6.35.13 | DatePickerRange | `date-picker-range` | `src/components/forms/DatePickerRange.jsx` | 15 | Input Field Range (2496:5498) + DatePicker ×2 |
| 6.35.14 | SuccessPopup | `success-popup` | `src/components/popup/SuccessPopup.jsx` | 13 | Popup + Lottie ของ SuccessScreen |
| 6.35.15 | ViewToggle | `view-toggle` | `src/components/toggle/ViewToggle.jsx` | 12 | TabBar + TabPill style=icon (§6.15.2) |
| 6.35.16 | StickyStatusTabBar | `sticky-status-tab-bar` | `src/components/tab/StickyStatusTabBar.jsx` | 12 | TabBar + TabPill + sticky vars ของ QueuePageShell |

#### 6.35.1 Clear Filters Tag (`ClearFiltersTag`)

ชิป "ล้างตัวกรอง" ที่โผล่เฉพาะเมื่อมีตัวกรองทำงาน และหุบ + จางออกเมื่อล้าง ใช้คู่กับ
`useSmoothClearFilters` (`src/hooks/useSmoothClearFilters.js`) และ `SmoothFilterResults`
(export ในไฟล์เดียวกัน) เพื่อให้ผลลัพธ์ใต้แถบตัวกรองเฟดตามกัน

| Prop | Default | ความหมาย |
|---|---|---|
| `active` | `false` | มีตัวกรองทำงาน → แสดงชิป · `false` → maxWidth 0 + opacity 0 (ยัง mount) |
| `onClear` | — | กดชิป |
| `value` | `'ล้างตัวกรอง'` | ข้อความบนชิป |
| `size` | `'lg'` | สเกลเดียวกับ FilterTag |

| Token | ใช้ที่ |
|---|---|
| `--dim-size-2280` (218px) | ความกว้างสูงสุดของชิปตอนกาง |
| `--motion-duration-popover` · `--motion-easing-standard` · `--motion-panel-offset-y` | transition กาง/หุบ และ SmoothFilterResults |
| ชุดสีของ FilterTag (§6.16) | ชิปเองเป็น FilterTag |

ห้าม `{active && <ClearFiltersTag/>}` — จะเสีย animation หุบ

#### 6.35.2 Search Result Count (`SearchResultCount`)

บรรทัดนับผลใต้ช่องค้นหา / แถบตัวกรอง สองข้อความตามมติผู้ใช้ 25 ส.ค. 69

| สถานะ | ข้อความ |
|---|---|
| ยังไม่ค้นหา/กรอง (`found === total` หรือ `filtered={false}`) | ทั้งหมด **N** รายการ |
| ค้นหา/กรองแล้ว | พบ **n** รายการ จาก N รายการ |

Props: `found` · `total` · `unit` (`'รายการ'`) · `filtered` (บังคับสถานะเมื่อผู้เรียกรู้เอง —
การเลือกแท็บสถานะไม่นับเป็นการค้นหา) · `aria-live="polite"`
Typography: `text-xs` + `--text-content-tertiary`, ตัวเลข `--text-content-secondary`

#### 6.35.3 Success Toast (`SuccessToast`)

Toast pill ลอยจากขอบบนจอ = Small Alert · State=Success (Figma 139:20246) ปิดเองหลัง `duration`

| Token | ค่า |
|---|---|
| surface | `--surface-smallalert-success` (#24A899) |
| icon + label | ขาว (`--text-brandPrimary-on-brand`) · TickOnCircle |
| radius | `--dim-radius-full` |
| padding | `--dim-space-200` / `--dim-space-400` / `--dim-space-600` (บน-ล่าง / ซ้าย / ขวา) |
| gap icon↔text | `--dim-space-100` |
| shadow | Brand Drop Shadow Bottom/200 |
| motion | fade + slide-up 200ms |

Props: `open` · `message` · `onClose` · `duration` (2400) · `topOffset` (`--dim-space-600`)
มติผู้ใช้ 27 ส.ค. 69: งานที่กดบันทึก **ใน popup** ต้องขึ้น SuccessPopup (§6.35.14) ไม่ใช่ toast

#### 6.35.4 Overflow Flex Row (`OverflowFlexRow`)

แถวชิปแบบ greedy — วางลูกตามความกว้างที่มี ตัวที่ล้นย้ายไปเมนู "เพิ่มเติม"
(`OverflowMoreTrigger`) วัดรวม flex gap และความกว้างปุ่ม more แล้ว ลูกที่ซ่อนยัง mount
(display:none) เพื่อวัด · เมนูเปิดใน portal (`.mih-search-popover`) · ปิดหลังเลือก

Props: `children` · `itemGap` (`--dim-space-200`) · `moreAriaLabel` · `menuId` · `menuZIndex`
(ค่าเริ่มต้น `--sm-z-modal-popover`)
เป็นตัวห่อบังคับของแถบตัวกรองทุก worklist (skill `mih-filter-bar`) — ห้าม `flex-wrap`

#### 6.35.5 Selectable Card Button (`SelectableCardButton`)

ปุ่มการ์ดเลือกได้แบบ radio (Figma PHCIS-OPD 160:26432) เส้นขอบเป็น **outset box-shadow**
(spread เท่านั้น) จึงสลับ 1px ↔ 2px ได้โดยไม่ดันปุ่มข้างและไม่ทิ้งช่องขาวบนพื้นขาว

| State | Stroke | Shadow | อื่น ๆ |
|---|---|---|---|
| default | 1px `--border-neutralbutton-tertiary` | — | |
| hover | 2px `--border-brandprimarybutton-tertiary` (#C5EDDA) | Brand Drop Shadow Top/200 | |
| selected | 2px `--brand-p600` | `--shadow-secondarysearch` | + `check_circle` ท้าย (`showTrailingCheck`) |
| disabled | 1px neutral/200 | — | ผิว neutral/100 · ตัวอักษร `--icon-disabledButton-tertiary` · ไม่รับคลิก |

Props: `selected` · `onClick` · `ariaLabel` · `showTrailingCheck` (true) · `disabled` ·
`minHeight` (`--dim-size-800` 40px = Button Medium, มติผู้ใช้ 31 ส.ค. 69) · `padX`
(`--dim-space-400`) · `padY` (`--dim-space-200`) — selected ชนะ hover

#### 6.35.6 Queue Patient Avatar (`QueuePatientAvatar`)

รูปผู้รับบริการในตารางคิว (Register Home UserA / New User 3891:7747) ลำดับ fallback:
รูป → วงกลมอักษรแรกของชื่อ (ตัดคำนำหน้าออกก่อน · สีพื้น 6 สีสุ่มคงที่จาก HN) → เงา New User
Props: `name` · `hn` · `avatarUrl` · `size` (36) · `newUser` · helper
`resolveQueuePatientAvatarProps({ hn, displayName, newUser, noProfilePhoto })` ดึงจากทะเบียนกลาง
สีพื้นอักษรแรก: `#F472B6 #A78BFA #60A5FA #34D399 #FBBF24 #FB7185` (primitive pink/violet/blue/
emerald/amber/rose 400 — palette เดียวกับหน้าลงทะเบียน)

#### 6.35.7 Masked Worklist Name (`MaskedWorklistName`)

ชื่อผู้รับบริการในเซลล์ worklist ทุกโมดูล — แสดงชื่อเต็ม (ไม่ปิดบังแล้ว มติผู้ใช้ 28 ส.ค. 69
ชื่อ component คงไว้เพราะมีผู้เรียก 40+ หน้า) ตัดด้วย … ผ่าน `EllipsisText` ของ Table และ
ต่อท้ายด้วย "ชื่อที่ต้องการให้เรียก" ในวงเล็บสีแดง (มติ 20 ก.ย. 69) hover เห็น Tooltip variant light

| Token | ใช้ที่ |
|---|---|
| `--text-neutral-default` + `--type-weight-medium` | ชื่อ (link / button) |
| `--text-danger-default` + `--type-weight-medium` | วงเล็บชื่อเรียก |
| `--dim-space-100` | ช่องว่างชื่อ↔วงเล็บ |

Props: `name` · `recordKey` (= HN → หา `preferredName` / `callByAlias` จาก `patientProfiles`)
· `linkTo` · `onRevealedClick` · `preferredName` / `callByAlias` (override) · `compact` ·
`truncate` · `emptyLabel` (`'—'`) · `stopPropagation` (true)

#### 6.35.8 PHCIS Top Toast (`PhcisTopToast`)

Toast กลางบนใต้ topbar (Figma Appointment 19:8245 / OPD 323:58923) = SmallAlert
success | error · `leadingGraphic="phcis"` · wrapper ใช้ `--filter-brand-drop-bottom-200`
ตำแหน่งบน = `MIH_TOPBAR_OFFSET` (`--dim-size-1400`) · motion enter 280 / dwell 3000 / exit 280 ms
(`PHCIS_TOP_TOAST_CYCLE_MS`) · hover หยุดนับ dwell · เคารพ `prefers-reduced-motion`
Props: `session` = `{ id, variant: 'success'|'error', message?, action?: { label, onClick } }`
| `null` · `onDismiss` — เปลี่ยน `id` ทุกครั้งที่ยิงใหม่

#### 6.35.9 Unsaved Changes Popup (`UnsavedChangesPopup`)

alias ของ `<AlertDialog type='warning' alignment='center'>` (PHCIS-ALL 3106:136000)
ปุ่มซ้าย→ขวา: **ไม่บันทึก** (`onDiscard` = cancel) · **บันทึกร่างและออก** (`onSave` = confirm)
· overlay / Esc = `onClose` (อยู่ต่อ ข้อมูลไม่หาย)
Props: `open` · `onClose` · `onSave` · `onDiscard` · `title` · `description` · `discardLabel` ·
`saveLabel` — ไม่มี token ของตัวเอง ใช้ของ AlertDialog §6.13.2 ทั้งหมด

#### 6.35.10 Row Action Menu (`RowActionMenu`)

เมนู ⋮ ประจำแถว/การ์ด = `BrandIconButton icon='more_vert'` (size sm · variant quaternary)
+ `Popover` รายการคำสั่ง `role='menu'`

| Token | ใช้ที่ |
|---|---|
| `--surface-card-100` · `--dim-radius-300` · `--dim-stroke-100` `--border-neutral-quaternary` | แผงเมนู |
| `--dim-size-2050` (220px) | `minWidth` ของแผง |
| ชุดสี BrandIconButton / DangerButton | ปุ่ม · รายการ `danger` |

Props: `items` = `[{ key, icon, label, onClick, disabled?, danger? }]` · `ariaLabel` · `size` ·
`variant` · `minWidth` — ไม่มี items → ไม่ render · คลิกหยุด propagation (ไม่เปิดแถว)

#### 6.35.11 Patient Summary Card (`PatientSummaryCard`)

แถบสรุปผู้รับบริการใน Popup Large (Figma 145:6155) — รูป 68px + `InfoCell` label/value คั่นด้วย
`Divider` 1×48px

| Variant | พื้น | label | value | divider |
|---|---|---|---|---|
| `default` | mint อ่อน (surface ของ PatientCard) | `--text-content-tertiary` | `--text-brandPrimary-default` | `--border-neutral-tertiary` |
| `gradient` | gradient teal/green ของ PatientCard (Consent Timeline 270:72714) | ขาว 90% | `--text-brandPrimary-on-brand` | ขาว 35% |

Props: `avatarUrl` (ส่ง prop = แสดงวงรูป แม้ค่าว่าง · ไม่ส่ง = ไม่มีรูป) · `avatarAlt` · `cells` =
`[{ label, value, valueColor?, children? }]` · `variant`

#### 6.35.12 Time Picker (`TimePicker`)

ช่องเลือกเวลา = trigger สไตล์ Select (§6.25 · `FieldShell` ของ `_inputBox`) + Popover สองคอลัมน์
HH : MM แทน `<input type="time">` ค่าเป็นสตริง `HH:MM` (24 ชม.) หรือ `''`

| size | สูง | padding-x | gap | font | icon |
|---|---|---|---|---|---|
| `sm` | `--dim-size-700` | `--dim-space-300` | `--dim-space-150` | `--type-size-xs` | `--dim-size-200` |
| `md` (default) | `--dim-size-800` | `--dim-space-300` | `--dim-space-150` | `--type-size-sm` | `--dim-size-300` |
| `lg` | `--dim-size-800` | `--dim-space-400` | `--dim-space-200` | `--type-size-sm` | `--dim-size-400` |

State tokens = ชุด `*-input-*` เดียวกับ InputField (§6.12): default / hover / open (= typing) /
filled / error / disabled · focus ring `--dim-stroke-400` `--primaryBorder-600` · disabled + มีค่า →
ตัวอักษร `--text-input-filled` (กฎ §6.12.3)
Props: `value` · `defaultValue` · `onChange(next)` · `label` (`'เวลา'`) · `size` · `minuteStep` (1) ·
`placeholder` (`'HH:MM'`) · `leadingIcon` (`schedule`) · `required` · `helperText` · `error` ·
`disabled` · `state` · `showLabel` / `showHelperText` / `showLeadingIcon` / `showRequired`

#### 6.35.13 Date Picker Range (`DatePickerRange`)

Input Field Range (Figma 2496:5498) ที่ใช้ DatePicker (§6.4) สองตัวใน `FieldShell` เดียว
ช่องสิ้นสุดใช้ค่าที่มากกว่าระหว่าง `minDate` กับวันเริ่มต้นเป็นวันแรกที่เลือกได้ (`endMinFromStart`)
Props: `value` = `[start, end]` สตริงตาม `format` · `onChange(next)` · `format` / `parse` (ต้องส่งคู่)
· `minDate` · `placeholders` · `label` · `required` · `helperText` · `error` · `size` (`lg`) ·
`leadingIcon` (`calendar_today`) · `disabled` · `showLabel` / `showHelperText` / `showLeadingIcon`
ไม่มี token ของตัวเอง — ใช้ของ DatePicker + Input Field Range

#### 6.35.14 Success Popup (`SuccessPopup`)

ป๊อปอัป "สำเร็จ" หลังบันทึก/ลงนามใน popup = `Popup size='default' showCloseButton={false}` +
Lottie เช็คถูก (`/illustration/success.json` ไฟล์เดียวกับ SuccessScreen) ไม่พาผู้ใช้ออกจากหน้า
(มติผู้ใช้ 27 ส.ค. 69) — โหลด Lottie ครั้งเดียว โหลดไม่ได้ก็แสดงเฉพาะข้อความ
Props: `open` · `title` · `description` · `buttonLabel` (`'ตกลง'`) · `onClose` · `secondaryAction` ·
`lottieSize` (160) · เนื้อหา gap `--dim-space-300` · padding-top `--dim-space-400`

#### 6.35.15 View Toggle (`ViewToggle`)

preset ของ `TabBar` + `TabPill style='icon'` (§6.15.2 · Figma 367-9620 · Tab Bar 2514:7929)
**ไม่มีสไตล์ของตัวเอง** — เดิมวาด track/ปุ่ม/hover เองแล้วสีหลุดจากชุด `*-tabpill-*` จึงห้ามใส่กลับ
Props: `value` · `onChange` · `options` = `[{ value, icon, label }]` (ค่าเริ่มต้น
`VIEW_TOGGLE_LIST_GRID` = รายการ `format_list_bulleted` / ตาราง `grid_view`) · `aria-label`

#### 6.35.16 Sticky Status Tab Bar (`StickyStatusTabBar`)

แถว TabPill สถานะเหนือช่องค้นหาของหน้าคิว — อยู่ในลำดับปกติ (inset บน 24px) เมื่อไม่เลื่อน และ
ตรึงใต้ SubNav (`[data-mih-sticky-subnav]`) เมื่อเลื่อน ชิป large → small ตอนติด
CSS vars ที่ประกาศบน `:root`: `--mih-status-tabbar-sticky-top` · `--mih-status-tabbar-sticky-bottom`
(= `calc()` ที่อ้าง `--mih-queue-sticky-under-tabs` เพื่อให้ thead ขยับพร้อม Topbar ที่ซ่อนตัว —
มติผู้ใช้ 10 ก.ย. 69) · export `QUEUE_STICKY_UNDER_STATUS_TABS` ให้ Table ใช้เป็น sticky top
Props: `tabs` = `[{ key, label, count? }]` · `value` · `onChange` · `size` (`small` 32px | `large`
48px — ผู้ใช้สั่ง 8 ก.ย. 69) · `sticky` (true · `false` = ไม่ฟัง scroll/resize เลย) · `aria-label`

---

### 6.36 Charts & Dashboard (ลงทะเบียน 20 ก.ย. 69)

กราฟและโครงหน้าแดชบอร์ดที่ทุกหน้าใน `/dashboard/overview` และ `/dashboard/reports/*` ใช้
(นับจากไฟล์ที่ import ณ 20 ก.ย. 69) เดิมกราฟทุกชนิดรวมอยู่ในหน้า "Charts" หน้าเดียว และ
วิดเจ็ต/โครงหน้าไม่เคยลงทะเบียน — ตอนนี้แต่ละตัวมี demo ที่ `/components/<slug>`
ทุกตัวเป็น SVG/HTML ล้วน (ไม่มี chart library) และ**สีของชุดข้อมูลส่งมาจากผู้เรียก**
(`phcisChartPalette` หรือ token) ไม่ฝังในกราฟ

#### 6.36.1 Chart primitives (`src/components/graph/Charts.jsx`)

| Component | slug | ใช้ใน (ไฟล์) | ข้อมูล | Interaction |
|---|---|---|---:|---|---|
| Donut · DonutLegend · DonutWithLegend | `donut-chart` | 45 | `segments = [{ value, color, label }]` · `centerLabel / centerSub / centerExtra` · `legendMaxVisibleItems` | hover ชิ้น → ตัวเลขกลาง · `activeIndex` + `onSegmentClick` |
| HBarList | `hbar-list` | 50 | `rows = [{ label, value, second?, segments? }]` · `valueLabel` `secondLabel` · `maxVisibleRows` · `startIndex` + `max` (ต่อลำดับ/สเกลข้ามคอลัมน์) · `maxValue` | `activeIndex` + `onRowClick` · segment tooltip |
| VBars | `vbars` | 48 | `months[]` + `series = [{ label, color, values[] }]` · `stacked` · `showValues` · `captions` · `barMaxWidth` `stackedMaxWidth` · `columnGap` (ระหว่างกลุ่ม) `barGap` (ในกลุ่ม) · `legendPerRow` | `activeMonth` + `onMonthClick` |
| LineChart · AreaLineChart | `line-chart` | 20 · 5 | `months` + `series` · Area: `valueLabels` `xAxisLabel` | `activeMonth` + `onMonthClick` |
| Gauge | `gauge` | 15 | `value` `max` `color` `label` `sub` `size` `tooltip` | — |
| Pyramid · PyramidPct | `pyramid` | 3 | `groups = [{ label, male, female }]` `maleColor` `femaleColor` | — |
| Pie | `pie` | 1 | `slices = [{ value, color, label }]` `size` | `activeIndex` + `onSliceClick` |
| Reveal | `reveal` | 11 | `delay` `threshold` — fade + slide-up ครั้งเดียวเมื่อเลื่อนถึง | — |

กฎที่ทุกตัวใช้ร่วม: แท่ง/ชิ้น/เส้น **วิ่งเข้าเมื่อเลื่อนถึง** (IntersectionObserver เล่นครั้งเดียว) ·
ป้ายแกนเป็น HTML ขนาดคงที่บน SVG `viewBox` 600 หน่วยที่ยืดเต็มกล่อง · ป้ายแกน Y วัดความกว้างจริง
(เลขหลักแสนไม่ล้น — 13 ก.ย. 69) · ≥ 7 หมวด × หลายชุด → `showValues={false}` (13 ก.ย. 69) ·
แท่งศูนย์เดียวกันชิดกัน `barGap=0` และเว้นระหว่างกลุ่มด้วย `columnGap` (15 ก.ย. 69)

| Token | ใช้ที่ |
|---|---|
| `--type-size-xs` · `--text-content-tertiary` | ป้ายแกน · legend |
| `--text-content-default` | ตัวเลขบนแท่ง / ตัวเลขกลางโดนัท |
| `--dim-space-050` `--dim-space-100` `--dim-space-150` | `barGap` · `columnGap` · gap ของ legend/HBarList |
| `--dim-size-1100` + `--dim-space-300` | ความสูงหนึ่งแถวของ HBarList (ใช้คำนวณ `maxVisibleRows`) |
| `--text-pill-blue` · `--text-pill-red` | สีตั้งต้นชาย/หญิงของ PyramidPct |
| `--surface-card-100` · `--border-card-default` · `--shadow-card` | tooltip ของกราฟ |

#### 6.36.2 Dashboard cards (`src/components/graph/DashboardCards.jsx`)

การ์ดสำเร็จรูป = `DashboardWidgetCard` + primitive ข้างบน + ข้อมูลตัวอย่าง `DEFAULT_*`
ที่ลงทะเบียนเพิ่มรอบนี้: **ClinicCountsCard** (`clinic-counts-card` — HBarList 2 คอลัมน์ ลำดับต่อกัน สเกลร่วม
แสดงครบไม่เลื่อน 14 ก.ย. 69) · **ClinicCompareCard** (`clinic-compare-card` — VBars grouped ปีงบนี้/ปีงบก่อน) ·
**SingleGaugeCard** (`single-gauge-card` — ฐานของ Dm/Ht/WorkingAge/Consent gauge cards) ·
**TelemedCallStatsCard** (`telemed-call-stats-card` — โดนัทสายเข้า/ออก + รายแพทย์ทุกคน ไม่ยุบ "อื่นๆ" 14 ก.ย. 69)
การ์ดเดิม 14 ใบ (Top10 · VisitTotal · AgeServicePyramid · RightsDonut · MonthlyVisits · Disease21Groups ·
DmHtNewPatients · DmControlGauge · HtControlGauge · WorkingAgeGroups · Coc · Consent · DmComplications ·
Top10Short) ยังอยู่ในกลุ่ม graph ตามเดิม

#### 6.36.3 Dashboard shell (`src/components/dashboard/*` + `src/pages/dashboard/dashboardReportKit.jsx`)

| Component | slug | หน้าที่ | Props หลัก |
|---|---|---|---|
| DashboardWidget | `dashboard-widget` | วิดเจ็ตทุกชนิดของ `/dashboard/overview` เรียกด้วย `type` — registry `DASHBOARD_WIDGET_RENDERERS` 24 ชนิด (top10 · visitTotal · clinicCounts · clinicCompare · agePyramid · rights · top10Short · monthlyVisits · anc · allergy · disease21 · dmhtNew · dmGauge · htGauge · workingAge · coc · consent · dmComplications · telemed · recentPatients · populationPyramid · opdTypeLine · residencePie · serviceCenterMap) เทมเพลตที่ผู้ใช้สร้างเองประกอบจากชุดนี้ | `type` · `yearLabel` · scope จาก `DashboardScopeProvider` (นอก provider = ไม่กรอง) |
| DashboardWidgetCard | `dashboard-widget-card` | chrome การ์ดวิดเจ็ตทุกใบ — หัว + subtitle + action · ไม่มีไอคอนนำ ไม่มี ⋮ (14 ก.ย. 69) | `title` `subtitle` `action` `bodyClassName` (`px-4 pb-4 pt-10`) |
| DashboardPageHeader | `dashboard-page-header` | หัวหน้าจอต้นแบบเดียวของทุกหน้าแดชบอร์ด (27 ส.ค. 69) สองแถวเสมอ: ชื่อ + ปุ่ม 36px / ชิป [Template] [วัน] [ปีงบ] [เฉพาะหน้า] [ล้างตัวกรอง] (14 ก.ย. 69) | `title` `subtitle` `dateFilter` `onDateFilterChange` `templateValue` `filters` `actions` `showDateFilter` `showFiscalYear` |
| DashboardHeaderTools | `dashboard-header-tools` | ปุ่มมาตรฐานเรียงคงที่ [แชร์] [ตั้งค่าเป้าหมาย] [นำส่งออกรายงาน] (15 ก.ย. 69) แชร์ → `DashboardSharePopup` · เป้าหมาย → `DashboardTargetsPopup` (localStorage ผ่าน `useDashboardTargets`) หรือ `targets.onOpen` | `share = { title, scope }` · `targets = { storageKey, fields[{ key,label,unit,default,min,max,direction }], current }` · `onExport` |
| DashboardTemplateSwitcher | `dashboard-template-switcher` | ชิปเลือกเทมเพลตหน้าจอ (แทนแท็บกลุ่มแดชบอร์ด 27 ส.ค. 69) ลิสต์ = สร้างใหม่ + `screen:<key>` 5 หน้าจอ + เทมเพลตผู้ใช้ · ขนาด lg เท่าชิปตัวกรอง | `value` (`screenValue(key)`) `onSelectTemplate` `onCreate` `onApply` `templates` `options` |
| DashboardSubNav | `dashboard-sub-nav` | TabUnderline ข้ามหน้ากลุ่ม §1.20 เรียงตาม TOR (14 ก.ย. 69) · `DASHBOARD_NAV_LINKS` แหล่งเดียว | — (อ่าน location) |
| DataSourceStrip | `data-source-strip` | แถบสถานะ Read Replica §1.20.9 (ปกติ/ล่าช้า · sync ล่าสุด · ประวัติ) | `scopeLabel` |
| StatTile · StatTileRow | `stat-tile` | ตัวเลข KPI + ป้าย (+ หน่วย โทน hint) กริด 2/3/4 · ตัวเลขใช้ `--type-family-heading-content` ไม่ใช่ Mitr (เลข 0 มีขีดทับ 25 ส.ค. 69) | `label` `value` `unit` `tone` (default/info/warning/danger) `hint` · Row: `columns` `framed` |
| ChartPanel · ChartsRow · EmptyChartNote | `chart-panel` | กล่องกราฟหนึ่งใบ (deep-link `#id` + กะพริบ) · แถวกล่องเท่ากัน · สถานะว่างมาตรฐาน | `id` `title` `hint` `actions` `alignContent` (center/start) `framed` · Empty: `text` `hint` `icon` |
| SummaryBannerCard | `summary-banner-card` | การ์ดสรุปแถวแรกใต้หัวหน้าจอ = WidgetCard + `SummaryCardsBar variant=banner` (14 ก.ย. 69) | `cards` (รูปแบบ SummaryCardsBar) `columns` |
| ReportGroupCard · SectionTopic | `report-group-card` | หนึ่งกลุ่ม = หนึ่งข้อ TOR · ไม่มีการ์ดขาวรอง ไม่มีหัวกลุ่ม/ปุ่มนำส่งออกรายกลุ่ม (14 ก.ย. 69) แถวหัวโผล่เฉพาะเมื่อมี `extraActions` · ลูก `selfCarded` ไม่ถูกห่อซ้ำ | `title` `subtitle` `extraActions` · Topic: `title` `subtitle` `trailing` |
| ReportToolbar | `report-toolbar` | แถบเครื่องมือตรึงใต้ SubNav ด้วย `--mih-queue-sticky-under-tabs` (ห้ามเขียน px) | `children` `trailing` |

| Token (shell) | ค่า | ใช้ที่ |
|---|---|---|
| `CARD` | `--surface-card-100` · `--dim-radius-500` · `--dim-stroke-100` `--border-brandPrimary-quaternary` · `--shadow-card` · padding `--dim-space-600` | การ์ดกลุ่ม/ReportToolbar |
| `CHART_PANEL` | เส้นขอบ `--border-card-default` (ไม่ใช่ brand) | ChartPanel |
| `--text-brandPrimary-default` · `--text-info-default` · `--text-warning-default` · `--text-danger-default` | โทนของ StatTile (success = brand) |
| `--type-size-2xl` · `--type-weight-bold` | ตัวเลข StatTile |
| `--dim-space-400` | gap ระหว่างกล่องใน ChartsRow / StatTileRow / ReportGroupCard |
| `--border-brandPrimary-quaternary` | เส้นใต้หัว DashboardWidgetCard |

---

### 6.37 Component Index — สเปกย่อของ component ที่ยังไม่มีหัวข้อเฉพาะ (20 ก.ย. 69)

หัวข้อ §6.1–6.36 ครอบคลุม component ราว 80 ตัว แต่ Components Library มี 163 รายการ — 79 รายการต่อไปนี้
มี demo + Behavior / Do / Don't ครบที่ `/components/<slug>` แล้ว แต่ไม่มีหัวข้อสเปกใน design.md จึงเปิดหน้า
Reference แล้วช่อง Spec ว่าง หัวข้อย่อยชุดนี้สรุปหน้าที่ · props · ไฟล์ต้นทาง ของแต่ละตัวจากคำอธิบายในไลบรารี
(ค่าทุกค่าในไฟล์เหล่านี้ alias token ของ Foundation ตามกติกา CLAUDE.md — สเปก token รายตัวดูจากโค้ดที่อ้าง)
เมื่อมีสเปกเต็มจาก Figma ให้ย้ายขึ้นเป็นหัวข้อของตัวเองแล้วลบรายการที่นี่

**button**

#### 6.37.1 BrandRadiusSubdueIconButton (brand-radius-subdue-icon-button)

Figma "Brand Radius Subdue Icon Button" (node 3871:4640). Pill-shaped subdue brand button with a leading icon + trailing chevron — a compact icon dropdown trigger (distinct from the circle, icon-only BrandCircleSubdueIconButton). Properties: Style (fill · outline) · State (default · hover · active · focus · disabled) · Size (xl 52 · lg 44 · md 40 · sm 36 · xs 32). Radius is 24 except **xLarge** which is fully round. Tables below mirror the Figma variant matrix — each table is one Style, rows are 5 sizes, columns are 5 states.

ไฟล์: `src/components/button/BrandRadiusSubdueIconButton.jsx` · demo: `/components/brand-radius-subdue-icon-button`

#### 6.37.2 BrandSubdueFloatButton (brand-subdue-float-button)

Figma "Brand Subdue Float Button" (node 2293:11088). FAB-style circular button with brand-tinted shadow. Single style — default + hover only (no disabled per Figma). Sizes: 48 · 56 · 72. Designed for fixed / floating placement (e.g. bottom-right of a page). Table below mirrors the Figma variant matrix — rows are sizes, columns are 2 states.

ไฟล์: `src/components/button/BrandSubdueFloatButton.jsx` · demo: `/components/brand-subdue-float-button`

#### 6.37.3 DangerCircleIconButton (danger-circle-icon-button)

Figma "Danger Circle Icon Button" (node 3853:1177). Round (50%) destructive icon button — the danger (red) sibling of BrandCircleIconButton. Properties: Style (fill · outline · ghost) · State (default · hover · selected · focus · disabled) · Size (xl 52 · lg 44 · md 40 · sm 36 · xs 32) — X Large is fully round (20px icon); XSmall is 32 with a 14px icon. Tables below mirror the Figma variant matrix — each table is one Style, rows are sizes, columns are states.

ไฟล์: `src/components/button/DangerCircleIconButton.jsx` · demo: `/components/danger-circle-icon-button`

#### 6.37.4 DangerCircleSubdueIconButton (danger-circle-subdue-icon-button)

Figma "Danger Circle Subdue Icon Button" (node 3853:1298). Low-emphasis destructive round icon button — the danger (red) sibling of BrandCircleSubdueIconButton. Properties: Style (fill · outline) · State (default · hover · active · focus · disabled) · Size (xl 52 · lg · md · sm · **xs/XSmall 32**) — spans the widest ladder (XSmall → X Large); X Large is fully round. Tables below mirror the Figma variant matrix — each table is one Style, rows are 5 sizes, columns are 5 states.

ไฟล์: `src/components/button/DangerCircleSubdueIconButton.jsx` · demo: `/components/danger-circle-subdue-icon-button`

**checkbox**

#### 6.37.5 CheckboxBox (checkbox-box)

The 16×16 box atom (Figma 306:1099) — 6 states arranged as 2 checked modes (Unchecked / Selected) × 3 interaction states (Default / Hover / Disabled). Stateless building block — parent owns `checked` + `hover`. Used by `Checkbox`, `CheckboxButton`, `CheckboxGroup`.

Props: `checked` · `hover` · `disabled` · `className` · `style`  
ไฟล์: `src/components/checkbox/CheckboxBox.jsx` · demo: `/components/checkbox-box`

#### 6.37.6 CheckboxGroup (checkbox-group)

Group label (with optional required asterisk) + multiple Checkbox rows + optional error message. 4 states: default / withDescription / error / withDescription-error.

Props: `legend` · `required` · `options` · `value` · `defaultValue` · `onChange` · `withDescription` · `error` · `name` · `className` · `style` · `columns`  
ไฟล์: `src/components/checkbox/CheckboxGroup.jsx` · demo: `/components/checkbox-group`

**radio**

#### 6.37.7 RadioBox (radio-box)

The 16×16 radio atom (Figma 328:5812) — 6 states arranged as 2 checked modes (Unchecked / Selected) × 3 interaction states (Default / Hover / Disabled). Stateless — parent owns `checked` + `hover`. Used by `Radio`, `RadioGroup`, and `RadioButton`. Mirrors the `CheckboxBox` matrix.

Props: `checked` · `hover` · `disabled` · `className` · `style`  
ไฟล์: `src/components/radio/RadioBox.jsx` · demo: `/components/radio-box`

#### 6.37.8 RadioGroup (radio-group)

Legend (with optional required asterisk) + multiple Radio rows + optional error message. 4 states from Figma 314:76: default / withDescription / error / withDescription-error. `options` shape: `{ value, label, description?, disabled? }`.

Props: `legend` · `required` · `options` · `value` · `defaultValue` · `onChange` · `withDescription` · `error` · `name` · `direction` · `disabled` · `className` · `style`  
ไฟล์: `src/components/radio/RadioGroup.jsx` · demo: `/components/radio-group`

**popup**

#### 6.37.9 DateFilterPopup (date-filter-popup)

Quick-options + custom-range date filter popup. Wrapped by FilterDateTag (recommended). Use directly when you need a standalone trigger.

Props: `open` · `onClose` · `initialQuick` · `initialTab` · `onApply`  
ไฟล์: `src/components/popup/DateFilterPopup.jsx` · demo: `/components/date-filter-popup`

#### 6.37.10 DrawingCanvasPopup (drawing-canvas-popup)

ข้อมูลประกอบการตรวจร่างกาย sidebar — ActionList + circle icon + success tick.

Props: `frontSrc` · `backSrc` · `open` · `onClose` · `onSave` · `title` · `subtitle` · `viewKey` · `markers` · `onMarkersChange` · `drawings` · `onDrawingsChange` · `onViewChange` · `saveLabel` · `cancelLabel` · `minBodyHeight` · `showViewSelector` · `figureAspect` · `figureHeightCm`  
ไฟล์: `src/components/popup/DrawingCanvasPopup.jsx` · demo: `/components/drawing-canvas-popup`

#### 6.37.11 ProfilePopover (profile-popover)

Floating profile menu anchored to the Sidebar avatar. Lists profile / language / settings / logout.

Props: `open` · `anchorRect` · `anchorRef` · `onClose` · `onSelect`  
ไฟล์: `src/components/popup/ProfilePopover.jsx` · demo: `/components/profile-popover`

#### 6.37.12 SettingsModal (settings-modal)

Modal for switching theme + light/dark mode. Emits onApply({ mode, theme }).

Props: `open` · `onClose` · `onApply` · `initialMode` · `initialTheme` · `initialSection`  
ไฟล์: `src/components/popup/SettingsModal.jsx` · demo: `/components/settings-modal`

#### 6.37.13 VitalsGraphPopup (vitals-graph-popup)

Popup size=extraLarge — fluid width/height up to 1440×1024 (24px viewport inset), same as BodyRecordPopup. Wraps VitalsSignGraph; footer ยกเลิก + ปิด. Prefer VitalsSignGraphDialog for the full trigger + popup pattern.

Props: `open` · `onClose` · `onSave` · `title` · `subtitle` · `showFooter` · `children`  
ไฟล์: `src/components/popup/VitalsGraphPopup.jsx` · demo: `/components/vitals-graph-popup`

**forms**

#### 6.37.14 AllergyForm (allergy-form)

Domain form for drug/material allergies. Thin wrapper around HistoryListForm with an allergy-specific schema.

Props: `onValidityChange` · `onRecordsChange`  
ไฟล์: `src/components/forms/AllergyForm.jsx` · demo: `/components/allergy-form`

#### 6.37.15 DiseaseForm (disease-form)

Domain form for past medical conditions. Thin wrapper around HistoryListForm with a disease-specific schema.

Props: `onValidityChange` · `onRecordsChange`  
ไฟล์: `src/components/forms/DiseaseForm.jsx` · demo: `/components/disease-form`

#### 6.37.16 FilterDateTag (filter-date-tag)

Composite of FilterTag (dropdown chip) + DateFilterPopup. Click to open the popup; pick a quick option (today / 7days / thismonth …) or a custom range. Value shape: { quick, tab?, single?, range? }.

Props: `prefix` · `value` · `defaultValue` · `onChange` · `defaultQuick` · `quickLabels` · `placeholder` · `showResolvedDate` · `selectedWhen` · `className` · `style`  
ไฟล์: `src/components/forms/FilterDateTag.jsx` · demo: `/components/filter-date-tag`

#### 6.37.17 HistoryListForm (history-list-form)

Generic schema-driven repeating-history list. Pass a `schema` describing topic, options, defaults, and badges. AllergyForm and DiseaseForm are thin wrappers around this — see those for example schemas.

Props: `schema` · `onValidityChange` · `onRecordsChange`  
ไฟล์: `src/components/forms/HistoryListForm.jsx` · demo: `/components/history-list-form`

#### 6.37.18 InputFieldRange (input-field-range)

Two inputs separated by a literal `-` (Figma 2496:5498). Aliases InputFieldSlash with separator="-". 5 states. Tables below mirror the Figma variant matrix — rows are sizes, columns are states.

ไฟล์: `src/components/forms/InputFieldRange.jsx` · demo: `/components/input-field-range`

#### 6.37.19 PhotoUploadField (photo-upload-field)

Figma "Photo" (node 3420:2565 · Telemed 127:56901). Dashed-border photo / image upload dropzone for the form builder. 2 states: Default (border brand/p200 #C5EDDA) · Hover (border darkens to brand/p600 #08A768, fill holds). Icon (image 20) + centered label. Pass `label` to relabel ("ถ่ายรูป" / "อัปโหลดภาพ"), `onFiles` to wire a hidden file input (add `capture` for direct camera), or `fullWidth` to stretch. Table below mirrors the Figma variant matrix.

ไฟล์: `src/components/forms/PhotoUploadField.jsx` · demo: `/components/photo-upload-field`

#### 6.37.20 Fields · Section / SubCard / Field (registration-form-fields-layout)

Layout building blocks for the Registration form. Composed together to lay out a section.

ไฟล์: `src/components/forms/Fields.jsx` · demo: `/components/registration-form-fields-layout`

#### 6.37.21 RegistrationSteps (registration-steps)

Per-step form bodies for the Registration (เวชระเบียน) workflow (6 steps · Figma 292:104252 + 2373:119045..). Each step accepts (form, setForm) and is rendered inside FormShell. Demo shows Step 2 (ข้อมูลส่วนตัว).

ไฟล์: `src/components/forms/RegistrationSteps.jsx` · demo: `/components/registration-steps`

#### 6.37.22 SelectableTag (selectable-tag)

Toggleable tag — used for multi-select filters and chip groups. Three sizes: sm 28h (default · NotePreset chips) · md 32h · lg 40h (Figma 3362:3326 — matches FilterTag Large so SelectableTag + FilterTag can share a toolbar row).

Props: `label` · `selected` · `size` · `leadingGlyph` · `onClick` · `className` · `style` · `ariaLabel` · `disabled` · `children`  
ไฟล์: `src/components/forms/SelectableTag.jsx` · demo: `/components/selectable-tag`

#### 6.37.23 TreatmentSteps (treatment-steps)

Per-step form bodies for the treatment record flow (PHCIS-ALL / CPOE).

ไฟล์: `src/components/forms/TreatmentSteps.jsx` · demo: `/components/treatment-steps`

#### 6.37.24 VitalsForm (vitals-form)

Full vitals capture form. Composes VitalCard / PressureCard / BMICard plus HistoryTimeline aside.

Props: `onValidityChange` · `onValuesChange` · `patientSex` · `patientAgeYears` · `patientHn` · `patientName` · `patientDob` · `ageBand` · `onAgeBandChange` · `carriedHeight` · `headerAction` · `beforeAddRoundSlot` · `compact`  
ไฟล์: `src/components/forms/VitalsForm.jsx` · demo: `/components/vitals-form`

#### 6.37.25 VoiceInputField (voice-input-field)

Input with trailing mic button (Figma 3026:23094). Click the mic to dictate — uses native SpeechRecognition where available (Chrome/Edge); otherwise fires onMicClick. 5 states (default · hover · filled · error · disabled). Tables below mirror the Figma variant matrix — rows are sizes, columns are states.

ไฟล์: `src/components/forms/VoiceInputField.jsx` · demo: `/components/voice-input-field`

#### 6.37.26 VoiceTextarea (voice-textarea)

Multiline input (Figma 268:26). 5 states (default · hover · filled · error · disabled); min-height 96px. Vertical resize handle by default. Tables below mirror the Figma variant matrix.

ไฟล์: `src/components/forms/VoiceTextarea.jsx` · demo: `/components/voice-textarea`

**examination**

#### 6.37.27 BodyMark (body-mark)

Figma PHCIS-ALL 2431:213971 — system TabCapsule row, view TabBar, dual anatomy figures, pain markers. ปุ่ม "บันทึกบนรูปภาพ" เปิด BodyRecordPopup.

ไฟล์: `src/components/examination/BodyMark.jsx` · demo: `/components/body-mark`

#### 6.37.28 BodyRecordPopup (body-record-popup)

ข้อมูลประกอบการตรวจร่างกาย sidebar — ActionList + circle icon + success tick.

Props: `systemKey` · `title`  
ไฟล์: `src/components/examination/BodyRecordPopup.jsx` · demo: `/components/body-record-popup`

**step**

#### 6.37.29 StepProgress (step-progress)

Horizontal stepper for multi-step forms. Past steps render as success (check), the current step is highlighted, future steps are dimmed. Pass `framed` to wrap the row in a white card chrome (the previous "Stepper" component is now this variant).

Props: `steps` · `current` · `completedSteps` · `onStepClick` · `className` · `framed` · `compact` · `labelNoWrap` · `hideLabels` · `morph`  
ไฟล์: `src/components/step/StepProgress.jsx` · demo: `/components/step-progress`

**timeline**

#### 6.37.30 HistoryTimeline (history-timeline)

Aside panel of past-visit rounds + note presets. Used inside OPD / MR / Treatment forms — emits onRoundsChange / onActiveRoundChange.

ไฟล์: `src/components/timeline/HistoryTimeline.jsx` · demo: `/components/history-timeline`

**list**

#### 6.37.31 ServiceDateList (service-date-list)

Grid per Figma FormCard (150:1223) + footer Button Group (162:20070): cover art `public/illustration/form-cover01–05.svg` by category (consent/certificate/request/assessment/document), full-bleed · aspect 227:113 · object-cover. ColorfulBadge (ประเภท); optional ColorfulBadge magenta **version** pill (DS Card `161:920`); StatusBadge “ประเมินแล้ว” เฉพาะแถวที่ประเมินแล้ว — กริดใช้บ่อยไม่มี success badge. Frame default 1px, hover 4px+Brand Drop Bottom/200; pass selected for Figma Selected (4px brand+shadow). Actions: BrandSubdueIcon (visibility) + NeutralSubdue outline+edit (แก้ไข) when open else Brand outline+edit (ประเมิน); section footer NeutralSubdue lg+chevron_right (ดูทั้งหมด) · 32px below grid. Picker for "บันทึกแบบฟอร์มอื่น".

Props: `title` · `items` · `selectedIndex` · `onSelect` · `width` · `className` · `showRailShadow` · `fillHeight`  
ไฟล์: `src/components/list/ServiceDateList.jsx` · demo: `/components/service-date-list`

**card**

#### 6.37.32 InventoryCard (inventory-card)

Time-series chart for vital signs with a TabPill switcher (อุณหภูมิ / ความดัน / ชีพจร / SpO₂). Uses an internal sample dataset.

Props: `image` · `imageAlt` · `title` · `titleSecondary` · `subtext` · `badge` · `style` · `statusBadge` · `style` · `showBadge` · `showStatusBadge` · `showTitle` · `showSubtext` · `state` · `selected` · `disabled` · `onClick` · `className` · `style`  
ไฟล์: `src/components/card/InventoryCard.jsx` · demo: `/components/inventory-card`

#### 6.37.33 VitalCard (vital-card)

Vitals input cards used in VitalsForm. VitalCard takes label/unit/value/onChange; PressureCard wraps systolic/diastolic; BMICard shows BMI on a 5-band gradient — pass weight (kg) + height (cm) for live BMI, or value alone.

Props: `label` · `required` · `compact` · `unit` · `value` · `step` · `hint` · `badge` · `children` · `onChange` · `onValidityChange` · `labelAction` · `unmeasurableLabel`  
ไฟล์: `src/components/card/VitalCard.jsx` · demo: `/components/vital-card`

**graph**

#### 6.37.34 AgeServicePyramidCard (age-service-pyramid-card)

Population pyramid card showing service usage by age group. Self-contained with default PYRAMID data; pass `groups` to override.

Props: `groups` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/age-service-pyramid-card`

#### 6.37.35 BigCharts (big-charts)

Dashboard composite chart cards built on the Charts primitives. Exports: PopulationPyramidCard · OpdTypeLineCard · ResidencePieCard · ServiceCenterMapCard. All four ship with internal sample data and filters.

ไฟล์: `src/components/graph/BigCharts.jsx` · demo: `/components/big-charts`

#### 6.37.36 ClinicCompareCard (clinic-compare-card)

เปรียบเทียบผู้รับบริการระหว่างคลินิก ปีงบนี้ vs ปีงบก่อน (VBars grouped) rows เดียวกับ ClinicCountsCard · yearLabel.

Props: `rows` · `title` · `yearLabel`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/clinic-compare-card`

#### 6.37.37 ClinicCountsCard (clinic-counts-card)

การ์ดจำนวนผู้รับบริการรายคลินิก (§1.20.1) — HBarList แบ่ง 2 คอลัมน์ ลำดับต่อกันและสเกลร่วม แสดงครบไม่ต้องเลื่อน (มติผู้ใช้ 14 ก.ย. 69) rows = DEFAULT_CLINIC_COUNTS [{ key, label, site, value, second, prev }] · title.

Props: `rows` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/clinic-counts-card`

#### 6.37.38 CocCard (coc-card)

Continuity-of-care card with 2 PersonStats + 2 BigStats. Pass `people` and `stats` to override.

Props: `people` · `value` · `320'` · `label` · `value` · `label` · `stats` · `label` · `tone` · `label` · `tone` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/coc-card`

#### 6.37.39 ConsentCard (consent-card)

Single-gauge card for the consent service indicator. Default value=0 (ยังไม่มีข้อมูล); pass `value`, `color`, `label` to override.

Props: `value` · `color` · `label` · `title` · `className`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/consent-card`

#### 6.37.40 Disease21GroupsCard (disease-21-groups-card)

ICD-10 21-chapter scrollable HBarList card with summary footer (totals + top group). Pass `groups` to override.

Props: `groups` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/disease-21-groups-card`

#### 6.37.41 DmComplicationsCard (dm-complications-card)

Two side-by-side gauges for diabetes complications (Diabetic foot, CKD). Pass `gauges` to override.

Props: `gauges` · `color` · `label` · `size` · `color` · `label` · `size` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/dm-complications-card`

#### 6.37.42 DmControlGaugeCard (dm-control-gauge-card)

Gauge card for diabetes glycemic control (DM ผู้ป่วยเบาหวานควบคุมน้ำตาลได้).

ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/dm-control-gauge-card`

#### 6.37.43 DmHtNewPatientsCard (dm-ht-new-patients-card)

Stacked VBars + 3 BigStats summary for new DM/HT patients over 12 months. Pass `data` and `summary` to override.

Props: `data` · `summary` · `004'` · `label` · `tone` · `518'` · `label` · `tone` · `754'` · `label` · `tone` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/dm-ht-new-patients-card`

#### 6.37.44 Donut · DonutWithLegend (donut-chart)

โดนัทสัดส่วน (SVG) — Donut = วงอย่างเดียว · DonutLegend = รายการสี ● ป้าย ค่า · % · DonutWithLegend = ทั้งสองอย่างเรียงซ้าย-ขวา (ใช้อยู่ 45 หน้า มากที่สุดในตระกูลกราฟ) segments = [{ value, color, label }] · centerLabel / centerSub / centerExtra (บรรทัดที่ 3) · activeIndex + onSegmentClick สำหรับไฮไลต์ · legendMaxVisibleItems จำกัดความสูง legend แล้ว scroll (รายชื่อ 69 ศูนย์).

Props: `segments` · `size` · `thickness` · `centerLabel` · `centerSub` · `centerExtra` · `activeIndex` · `onSegmentClick`  
ไฟล์: `src/components/graph/Charts.jsx` · demo: `/components/donut-chart`

#### 6.37.45 Gauge (gauge)

เกจครึ่งวงแสดงค่าเทียบเป้า/เพดาน (ควบคุมเบาหวาน · ความดัน · เป้าหมายรายงาน) ใช้อยู่ 15 หน้า value · max (100) · color · label · sub · size (160) · tooltip.

Props: `value` · `max` · `color` · `label` · `size` · `tooltip` · `sub`  
ไฟล์: `src/components/graph/Charts.jsx` · demo: `/components/gauge`

#### 6.37.46 HBarList (hbar-list)

รายการแท่งแนวนอนเรียงอันดับ (Top 10 โรค · ผู้รับบริการรายคลินิก · คลังยา) ใช้อยู่ 50 หน้า rows = [{ label, value, second?, segments? }] · valueLabel / secondLabel หน่วย (ค่าเริ่มต้น คน / ครั้ง) · activeIndex + onRowClick · maxVisibleRows จำกัดความสูงแล้ว scroll · startIndex + max ใช้ต่อลำดับ/สเกลข้ามหลายคอลัมน์ · maxValue ล็อกสเกล (เช่น % ของความจุ 0–100).

Props: `rows` · `valueLabel` · `secondLabel` · `activeIndex` · `onRowClick` · `maxVisibleRows` · `startIndex` · `max` · `maxValue`  
ไฟล์: `src/components/graph/Charts.jsx` · demo: `/components/hbar-list`

#### 6.37.47 HtControlGaugeCard (ht-control-gauge-card)

Gauge card for hypertension blood-pressure control (ผู้ป่วยความดันโลหิตควบคุมได้).

ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/ht-control-gauge-card`

#### 6.37.48 LineChart · AreaLineChart (line-chart)

เส้นแนวโน้มหลายชุด 12 เดือน (LineChart ใช้อยู่ 20 หน้า · AreaLineChart แบบถมพื้น 5 หน้า) months + series = [{ label, color, values[] }] · activeMonth + onMonthClick · AreaLineChart เพิ่ม valueLabels และ xAxisLabel ขอบซ้ายวัดจากความกว้างป้ายแกน Y จริง (เลขหลักแสนไม่ล้น — 13 ก.ย. 69).

Props: `months` · `series` · `height` · `onMonthClick` · `activeMonth`  
ไฟล์: `src/components/graph/Charts.jsx` · demo: `/components/line-chart`

#### 6.37.49 MonthlyVisitsCard (monthly-visits-card)

LineChart card for monthly visits with active-month highlight. Pass `data` to override (months + series).

Props: `data` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/monthly-visits-card`

#### 6.37.50 Pie (pie)

วงกลมเต็ม (Figma 4:8610) สำหรับสัดส่วนที่ต้องการชิ้นใหญ่เห็นชัด (ที่อยู่อาศัยของผู้รับบริการ) slices = [{ value, color, label }] · size (240) · activeIndex + onSliceClick — โดยทั่วไปใช้ Donut แทนเพราะมีที่ว่างกลางสำหรับตัวเลขรวม.

Props: `slices` · `size` · `activeIndex` · `onSliceClick`  
ไฟล์: `src/components/graph/Charts.jsx` · demo: `/components/pie`

#### 6.37.51 Pyramid · PyramidPct (pyramid)

พีระมิดประชากร อายุ × เพศ (ผู้รับบริการตามช่วงอายุ) groups = [{ label, male, female }] · maleColor / femaleColor · Pyramid แสดงค่าจริง · PyramidPct แสดงเป็น % ต่อแถว (rowHeight).

Props: `groups` · `maleColor` · `femaleColor` · `height`  
ไฟล์: `src/components/graph/Charts.jsx` · demo: `/components/pyramid`

#### 6.37.52 Reveal (reveal)

ตัวห่อ fade + slide-up เมื่อเลื่อนถึง (IntersectionObserver เล่นครั้งเดียวต่อ mount) ใช้ห่อการ์ด/แถวกราฟทุกหน้าแดชบอร์ด (11 หน้า) เพื่อให้หน้าที่ยาวมากไม่กระโดดขึ้นพร้อมกันทั้งหน้า delay (ms) · threshold (0.15) · className / style / id.

Props: `children` · `delay` · `threshold` · `className` · `style` · `id`  
ไฟล์: `src/components/graph/Charts.jsx` · demo: `/components/reveal`

#### 6.37.53 RightsDonutCard (rights-donut-card)

Donut card for healthcare-rights distribution with clickable legend. Pass `slices` to override.

Props: `slices` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/rights-donut-card`

#### 6.37.54 SingleGaugeCard (single-gauge-card)

การ์ดเกจเดี่ยว — ฐานของ DmControlGaugeCard / HtControlGaugeCard / WorkingAgeGroupsCard / ConsentCard value · color · label · sub · title · size.

Props: `value` · `color` · `label` · `sub` · `title` · `icon` · `size`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/single-gauge-card`

#### 6.37.55 TelemedCallStatsCard (telemed-call-stats-card)

สถิติการปรึกษาทางไกล — ซ้าย โดนัทสายเข้า/ออก + ค่าเฉลี่ยเวลา · ขวา จำนวนสายรายแพทย์ (แสดงทุกคน ไม่ยุบ "อื่นๆ" — มติผู้ใช้ 14 ก.ย. 69) avgDuration · callsIn · callsOut · doctors [{ name, count, share }] · donut · title.

Props: `avgDuration` · `callsIn` · `callsOut` · `doctors` · `donut` · `title` · `className`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/telemed-call-stats-card`

#### 6.37.56 Top10DiseasesCard (top10-diseases-card)

HBarList card showing top-10 diseases with click-to-pin highlight + summary tag. Pass `rows` to override.

Props: `rows` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/top10-diseases-card`

#### 6.37.57 Top10DiseasesShortCard (top10-diseases-short-card)

Compact 8-row variant of Top10DiseasesCard for tighter layouts.

Props: `rows` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/top10-diseases-short-card`

#### 6.37.58 VBars (vbars)

แท่งแนวตั้งตามหมวด/เดือน หลายชุดข้อมูล ทั้งแบบเคียงกันและซ้อน (stacked) ใช้อยู่ 48 หน้า months = ป้ายแกน X · series = [{ label, color, values[] }] · stacked · showValues (ปิดเมื่อแท่งเยอะจนตัวเลขทับกัน) · captions ป้ายเต็มสำหรับ tooltip เมื่อแกน X ย่อ · barMaxWidth / stackedMaxWidth · columnGap ระหว่างกลุ่ม · barGap ในกลุ่ม (ส่ง 0 ให้แท่งศูนย์เดียวกันชิดกัน — มติผู้ใช้ 15 ก.ย. 69) · legendPerRow.

Props: `months` · `series` · `height` · `onMonthClick` · `activeMonth` · `stacked` · `barMaxWidth` · `showLegend` · `showValues` · `legendPerRow` · `captions` · `stackedMaxWidth` · `columnGap` · `barGap`  
ไฟล์: `src/components/graph/Charts.jsx` · demo: `/components/vbars`

#### 6.37.59 VisitTotalCard (visit-total-card)

Donut + Stat-row card showing total visit population. Pass `segments`, `centerLabel`, `stats` to override.

Props: `segments` · `centerLabel` · `373'` · `centerSub` · `stats` · `value` · `313'` · `label` · `value` · `060'` · `label` · `value` · `327'` · `label` · `outsideBreakdown` · `updatedAt` · `title`  
ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/visit-total-card`

#### 6.37.60 VitalsSignGraph (vitals-sign-graph)

Time-series chart for vital signs with a TabPill switcher (อุณหภูมิ / ความดัน / ชีพจร / SpO₂). Uses an internal sample dataset.

Props: `initialTab` · `activeTab` · `onActiveTabChange` · `fillHeight` · `compact` · `series`  
ไฟล์: `src/components/graph/VitalsSignGraph.jsx` · demo: `/components/vitals-sign-graph`

#### 6.37.61 WorkingAgeGroupsCard (working-age-groups-card)

Gauge card for working-age groups indicator (กลุ่มวัยทำงาน 8 กลุ่ม).

ไฟล์: `src/components/graph/DashboardCards.jsx` · demo: `/components/working-age-groups-card`

**dashboard**

#### 6.37.62 DashboardHeaderTools (dashboard-header-tools)

ชุดปุ่มคำสั่งมาตรฐานของหัวหน้าจอแดชบอร์ด เรียงคงที่ [แชร์] [ตั้งค่าเป้าหมาย] [นำส่งออกรายงาน] (มติผู้ใช้ 15 ก.ย. 69) แชร์ → DashboardSharePopup (อีเมลหลายคน + เรื่อง + ข้อความ แนบลิงก์หน้าพร้อมตัวกรอง) · ตั้งค่าเป้าหมาย → DashboardTargetsPopup ตาม fields (เก็บต่อผู้ใช้ใน localStorage ผ่าน useDashboardTargets) หรือ targets.onOpen สำหรับหน้าที่มีป๊อปอัปของตัวเอง · นำส่งออก → onExport share · targets · onExport · exportLabel · exportProps · children.

Props: `share` · `targets` · `onExport` · `exportLabel` · `exportProps` · `children`  
ไฟล์: `src/components/dashboard/DashboardHeaderTools.jsx` · demo: `/components/dashboard-header-tools`

#### 6.37.63 DashboardPageHeader (dashboard-page-header)

หัวหน้าจอ "ต้นแบบเดียว" ของทุกหน้าแดชบอร์ด (มติผู้ใช้ 27 ส.ค. 69: แก้ที่นี่ที่เดียว ทุกหน้าเปลี่ยนตาม) สองแถวเสมอ (14 ก.ย. 69): แถวบน = ชื่อหน้า + ปุ่มคำสั่งชิดขวาสูง 36px · แถวล่าง = ชิปตัวกรอง [Template] [วัน] [ปีงบ] [ตัวกรองเฉพาะหน้า] [ล้างตัวกรอง] title · subtitle · dateFilter / onDateFilterChange · defaultQuick · templateValue / templates / onSelectTemplate / onApplyTemplate · filters · actions · showDateFilter · showFiscalYear.

Props: `title` · `dateFilter` · `onDateFilterChange` · `defaultQuick` · `templateValue` · `templates` · `onSelectTemplate` · `onApplyTemplate` · `switcherSize` · `filters` · `actions` · `showDateFilter` · `showFiscalYear` · `subtitle`  
ไฟล์: `src/components/dashboard/DashboardPageHeader.jsx` · demo: `/components/dashboard-page-header`

#### 6.37.64 DashboardSubNav (dashboard-sub-nav)

แถว TabUnderline ข้ามหน้าในกลุ่มแดชบอร์ด/รายงาน §1.20 เรียงตามลำดับหัวข้อ TOR (มติผู้ใช้ 14 ก.ย. 69) อ่าน active จาก location และ push route เมื่อเลือก · morphOnScroll ย่อเมื่อเลื่อน DASHBOARD_NAV_LINKS เป็นแหล่งเดียวของรายการหน้า (ชิป Template ก็ใช้ชุดนี้).

ไฟล์: `src/components/dashboard/DashboardSubNav.jsx` · demo: `/components/dashboard-sub-nav`

#### 6.37.65 DashboardTemplateSwitcher (dashboard-template-switcher)

ชิปเลือก "เทมเพลตหน้าจอ" (แทนแถบแท็บของกลุ่มแดชบอร์ด มติผู้ใช้ 27 ส.ค. 69) ลิสต์เดียวมีสองกลุ่ม: หน้าจอมาตรฐาน screen:<key> (เลือกแล้ว push route) และเทมเพลตที่ผู้ใช้สร้างเอง รายการแรก = สร้างเทมเพลทใหม่ (เปิด wizard เอง) value · onSelectTemplate · onCreate · onApply · templates · options (override ลิสต์ทั้งหมดสำหรับระบบที่มีทะเบียนของตัวเอง).

Props: `value` · `onSelectTemplate` · `onCreate` · `onApply` · `templates` · `options` · `size`  
ไฟล์: `src/components/dashboard/DashboardTemplateSwitcher.jsx` · demo: `/components/dashboard-template-switcher`

#### 6.37.66 DashboardWidget (24 ชนิดของ /dashboard/overview) (dashboard-widget)

วิดเจ็ตทุกชนิดของหน้าภาพรวมโรงพยาบาล (/dashboard/overview) เรียกด้วย type เดียว — dashboardWidgetRegistry.DASHBOARD_WIDGET_RENDERERS แม็พ 24 ชนิดไปยังการ์ดกราฟ (DashboardCards / BigCharts) พร้อมข้อมูลตัวอย่างและตัวกรอง scope จาก DashboardScopeProvider เทมเพลตแดชบอร์ดที่ผู้ใช้สร้างเองก็ประกอบจากชนิดชุดนี้ type · yearLabel.

Props: `type` · `yearLabel`  
ไฟล์: `src/components/dashboard/dashboardWidgetRegistry.jsx` · demo: `/components/dashboard-widget`

#### 6.37.67 DashboardWidgetCard (dashboard-widget-card)

chrome ของการ์ดวิดเจ็ตทุกใบบนแดชบอร์ด (หัวการ์ด + บรรทัดย่อย + action ท้ายหัว) ไม่มีไอคอนนำและไม่มีปุ่ม ⋮ แล้ว (มติผู้ใช้ 14 ก.ย. 69) title · subtitle · action · bodyClassName (ค่าเริ่มต้น px-4 pb-4 pt-10 · การ์ดกลุ่มรายงานส่งระยะบนแคบกว่า) · className / headerClassName.

ไฟล์: `src/components/dashboard/DashboardWidgetCard.jsx` · demo: `/components/dashboard-widget-card`

#### 6.37.68 DataSourceStrip (data-source-strip)

แถบสถานะแหล่งข้อมูล §1.20.9 (Read Replica สำหรับงานวิเคราะห์) — บอกว่ารายงานอ่านจากฐานวิเคราะห์ ไม่ใช่ฐานให้บริการ แสดงสถานะ ปกติ/ล่าช้า + เวลา sync ล่าสุด และเปิดป๊อปอัปประวัติการ sync (Table) ได้ scopeLabel.

Props: `scopeLabel`  
ไฟล์: `src/components/dashboard/DataSourceStrip.jsx` · demo: `/components/data-source-strip`

**dashboard (report kit)**

#### 6.37.69 ChartPanel · ChartsRow (chart-panel)

กล่องกราฟของหน้ารายงาน (§1.20) — ChartPanel = การ์ดกราฟหนึ่งใบ (id สำหรับ deep-link #id แล้วกะพริบ · title · hint · actions · alignContent center|start · framed) · ChartsRow = แถวที่วาง ChartPanel หลายใบเท่า ๆ กัน · EmptyChartNote = สถานะว่างมาตรฐาน (text · hint · icon).

ไฟล์: `src/pages/dashboard/dashboardReportKit.jsx` · demo: `/components/chart-panel`

#### 6.37.70 ReportGroupCard · SectionTopic (report-group-card)

หนึ่งกลุ่มรายงาน (หนึ่งหัวข้อ TOR) = ลูกของมันเท่านั้น ไม่มีการ์ดขาวรองและไม่มีแถวหัวกลุ่ม/ปุ่มนำส่งออกรายกลุ่ม (มติผู้ใช้ 14 ก.ย. 69) แถวหัวโผล่เฉพาะเมื่อส่ง extraActions (ชิปเลือกช่วง · ปุ่มกติกา) SectionTopic = หัวข้อส่วนแบบ Topic primary ตัวหนา + subtitle + trailing.

ไฟล์: `src/pages/dashboard/dashboardReportKit.jsx` · demo: `/components/report-group-card`

#### 6.37.71 ReportToolbar (report-toolbar)

แถบเครื่องมือของหน้ารายงาน (ชิปช่วง/ตัวกรอง ซ้าย · ปุ่ม ขวา) เป็นการ์ดบางที่ตรึงใต้ SubNav ด้วยค่า sticky เดียวกับหัวตารางของ QueuePageShell — ห้ามเขียน px เอง children · trailing.

ไฟล์: `src/pages/dashboard/dashboardReportKit.jsx` · demo: `/components/report-toolbar`

#### 6.37.72 StatTile · StatTileRow (stat-tile)

ตัวเลข KPI + ป้าย (+ หน่วย · โทน · hint) เรียงเป็นกริด 2/3/4 คอลัมน์ ใช้ทุกหน้ารายงานใน /dashboard/reports/* label · value · unit · tone (default | info | warning | danger — success ใช้สีแบรนด์เดียวกับ default) · hint · StatTileRow columns (4) · framed (การ์ดของตัวเอง; false เมื่ออยู่ในการ์ดอื่นแล้ว) ตัวเลขใช้ heading-content ไม่ใช่ Mitr เพราะเลข 0 ของ Mitr มีขีดทับ (25 ส.ค. 69).

ไฟล์: `src/pages/dashboard/dashboardReportKit.jsx` · demo: `/components/stat-tile`

#### 6.37.73 SummaryBannerCard (summary-banner-card)

การ์ดสรุปตัวเลขแถวบนสุดของหน้ารายงาน = DashboardWidgetCard ห่อ SummaryCardsBar variant=banner (ไอคอน + ตัวเลข + บรรทัดรอง) cards = รูปแบบเดียวกับ SummaryCardsBar [{ key, label, value, color, icon, indicator }] · columns (4).

ไฟล์: `src/pages/dashboard/dashboardReportKit.jsx` · demo: `/components/summary-banner-card`

**layout**

#### 6.37.74 SubHeader (sub-header)

Page sub-header that mirrors the Header chrome (1px brand-quaternary bottom border + brand-tinted shadow). Sits directly under Header for in-page action rows. Props: title (required), onBack, backLabel, children (right-aligned actions).

Props: `title` · `afterTitle` · `belowTitle` · `onBack` · `backLabel` · `children` · `leadingActions` · `flushSummaryCardsBelow` · `hideBottomBorder` · `hideConsentAction`  
ไฟล์: `src/components/layout/SubHeader.jsx` · demo: `/components/sub-header`

#### 6.37.75 SummaryCardsBar (summary-cards-bar)

Single component for both queue-page summary tiles and dashboard overview banner. Switch via `variant` prop — "panel" (default, used on queue pages) renders inside a flush surface-card-100 panel; "banner" (used on /dashboard) renders flat with bordered cards that wrap.

Props: `cards` · `title` · `variant` · `flushUnderSubHeader` · `bottomShadow` · `detached` · `columns` · `bottomPad`  
ไฟล์: `src/components/layout/SummaryCardsBar.jsx` · demo: `/components/summary-cards-bar`

**templates**

#### 6.37.76 FormShell (form-shell)

Full-page registration form chrome: SubHeader (ประวัติการแก้ไข popup · บัตรคิว · …) + 6-step Stepper + body + sticky footer.

Props: `stepIndex` · `onStepChange` · `onReadSmartCard` · `smartCardRead` · `completedSteps` · `onSubmit` · `onSaveDraft` · `beforeCard` · `patientHn` · `onPrintQueueTicket` · `onPrintPatientCard` · `bodyCard` · `children`  
ไฟล์: `src/templates/FormShell.jsx` · demo: `/components/form-shell`

#### 6.37.77 QueuePageShell (queue-page-shell)

Shared chrome for queue / list pages: Sidebar + Header + SubHeader + SummaryCardsBar + sticky TabUnderline + body + Footer. Pass title, breadcrumb, summaryCards, tabs, etc. via props; children render in the body.

Props: `activeKey` · `sidebarItems` · `sidebarFallbackKey` · `sidebarOpenWidth` · `sidebarWrapLabels` · `sidebarHideActiveChevron` · `breadcrumb` · `brandTitle` · `title` · `afterTitle` · `belowTitle` · `onBack` · `backLabel` · `actions` · `leadingActions` · `hideConsentAction` · `summaryCards` · `summaryTitle` · `summaryFlushUnderSubHeader` · `summaryBottomPad` · `summaryBelowTabs` · `headerInfo` · `sti  
ไฟล์: `src/templates/QueuePageShell.jsx` · demo: `/components/queue-page-shell`

#### 6.37.78 TreatmentFormShell (treatment-form-shell)

CPOE treatment record shell. Unified card: bilingual StepProgress (framed) + body + sticky footer.

Props: `stepIndex` · `onStepChange` · `completedSteps` · `children` · `steps` · `queuePath` · `morphTitle` · `hideMorphTitle` · `morphBandClassName` · `aboveBand` · `bandAfterTitle` · `belowBand` · `morphStickyTop` · `plainBodySteps` · `sidePanel` · `sidePanelCollapsed` · `step0FlushLeft` · `onLeave` · `onSaveDraft` · `showSaveDraft` · `draftStatusText` · `draftDirty` · `draftSaving` · `fromDepartment` ·  
ไฟล์: `src/templates/TreatmentFormShell.jsx` · demo: `/components/treatment-form-shell`

**vitals**

#### 6.37.79 VitalsSignGraphDialog (vitals-sign-graph-dialog)

Shared chrome for queue / list pages: Sidebar + Header + SubHeader + SummaryCardsBar + sticky TabUnderline + body + Footer. Pass title, breadcrumb, summaryCards, tabs, etc. via props; children render in the body.

Props: `open` · `onOpenChange` · `title` · `subtitle` · `disabled` · `triggerLabel` · `triggerIcon` · `triggerSize` · `triggerVariant` · `showTrigger` · `showFooter` · `initialTab` · `onSave` · `onClose` · `renderTrigger` · `className`  
ไฟล์: `src/components/vitals/VitalsSignGraphDialog.jsx` · demo: `/components/vitals-sign-graph-dialog`

---

## 7. Mobile Application Design System

The mobile citizen app (`/m/*` routes, see [src/pages/mobile/](src/pages/mobile/)) reuses the **foundation** tokens — colour, typography family, icon size, motion — verbatim from §1–§4. Only **shape, height, and density** differ, because thumb reach and one-handed use are the dominant constraints on a 390-wide viewport.

The mobile system is intentionally lean: ~7 primitives that compose every screen. There is no mobile `Table`, `Tabs`, `Accordion`, or `Dropdown` yet — when those land, they extend this layer, not duplicate it.

### 7.1. Layering rules

```
┌──────────────────────────────────────────────────┐
│  Foundation (shared)                             │  §1–§4
│  --brand-p*, --type-family-*, --text-*, etc.     │
└──────────────────────────────────────────────────┘
              ▲                      ▲
              │                      │
┌─────────────────────────┐  ┌──────────────────────┐
│  Web tokens (default)   │  │  Mobile tokens       │  §7.2
│  --dim-*, --type-size-* │  │  --mobile-*          │
└─────────────────────────┘  └──────────────────────┘
              ▲                      ▲
              │                      │
   Web components (§5–§6)     Mobile components (§7.3)
```

**Hard rules:**

| Rule | What it means | Where it's enforced |
|------|---------------|--------------------|
| ✅ Mobile components alias `--mobile-*` and foundation tokens | shape/height/density tokens are platform-specific; everything else is shared | [`src/components/mobile/*`](src/components/mobile/) — every primitive |
| ❌ Web components MUST NOT consume `--mobile-*` | if a value belongs on both, it lives in `--dim-*` / `--type-*` instead | review |
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
| `--mobile-bottom-nav-height` | — | `80px` |
| `--mobile-touch-target-min` | — | `44px` |

#### Semantic colour (`--mobile-surface-*` / `--mobile-text-*` / `--mobile-border-*` / `--mobile-icon-*`)

Mobile colour tokens are **separate from web** (web uses `--surface-*` / `--text-*` directly). This split lets the citizen app run a different palette — currently a brighter emerald (`#08A768`) versus the desktop deep-green (`#007549`) — without re-skinning the desktop EMR. Both palettes inherit foundation tokens (typography family, motion, icon families).

Source of truth: Figma file `opz7X9AAKGQf5prHhsPDGn` Light/Dark variable collections. Values exported to [`src/index.css`](src/index.css) — light values live under `:root`, dark overrides under `[data-mobile-theme="dark"]` (toggleable per-subtree).

Each tone covers 4 categories × 5 surface tiers + on-* text variants. The button system reads from these tokens via the `buildPalette(tone, options)` helper in [_base.jsx](src/components/mobile/button/_base.jsx):

| Tone | Used by | Default fill (light) | Default fill (dark) |
|------|---------|----------------------|---------------------|
| `brandPrimaryButton` | BrandButton, BrandSubdueButton, BrandIconButton, BrandSubdueIconButton, BrandCircleIconButton, BrandCircleSubdueIconButton, BrandSubdueFloatButton | `#08A768` | `#6AD6A5` |
| `brandSecondaryButton` | reserved for second-tier brand actions | `#427C3D` | `#78B573` |
| `dangerButton` | DangerButton, DangerSubdueButton, DangerIconButton, DangerSubdueIconButton | `#B91C1C` | `#FCA5A5` |
| `neutralButton` | NeutralButton, NeutralSubdueButton, NeutralIconButton, NeutralSubdueIconButton, HistoryListFormButton | `#4D5358` | `#C9CDD0` |
| `disabledButton` | every variant's `disabled` state (all tones) | `#F4F5F5` | `#363B3F` |

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

Text + icon tokens follow the same naming (`text-{tone}-default | secondary | tertiary | quaternary | on-brand | on-default | on-neutral`). `on-*` keys carry the contrast colour painted ON TOP of the matching solid fill (`text-brandPrimaryButton-on-brand` = white in light mode, `#002318` in dark mode).

Focus ring stroke comes from `--mobile-focus-ring-width` (4px); colour is `--mobile-border-brandPrimaryButton-tertiary` regardless of tone (the focus ring's job is to mark _focus_, not _intent_).

> **How to add a new tone**: append the full tier+text+border+icon set to BOTH `:root` and `[data-mobile-theme="dark"]` in [src/index.css](src/index.css), then call `buildPalette('newTone')` from the consuming button file. No changes to `_base.jsx` should be needed.

### 7.3. Component map

Each primitive lives in its own file under [src/components/mobile/](src/components/mobile/). Pages can import from `'src/components/mobile'` (barrel) or from `'./MobileShell'` (legacy re-exports — kept so the existing 19 pages don't need to be touched).

**Naming convention** — mobile components use the **same names as the Figma file** (no `Mobile` prefix). The folder location (`src/components/mobile/button/BrandButton.jsx` vs `src/components/button/BrandButton.jsx`) is what disambiguates them from the web equivalents. Import path is the source of truth:

```js
import { BrandButton } from '@/components/button';         // web (36/40/44h, radius-150)
import { BrandButton } from '@/components/mobile';         // mobile (40/48/56h, pill)
import { BrandButton } from '@/components/mobile/button';  // mobile, explicit
```

#### Button family (Figma App `opz7X9AAKGQf5prHhsPDGn` · Brand Button `231:29` + siblings)

All text buttons live under [src/components/mobile/button/](src/components/mobile/button/). They share `MobileButtonBase` + `createFigmaTextButton` ([_base.jsx](src/components/mobile/button/_base.jsx)) — each file builds a tone palette then exposes the **same Figma property surface**:

| Figma property | Code prop | Values |
|----------------|-----------|--------|
| **Style** | `variant` / `buttonStyle` | `Fill` · `Outline` · `Ghost` (Subdue families: Fill · Outline only) |
| **State** | `state` or `hovered` / `active` / `focused` / `disabled` | `Default` · `Hover` · `Active` · `Focus` · `Disable` |
| **Size** | `size` | `Large` (56) · `Medium` (48) · `Small` (40) — also `lg` / `md` / `sm` |
| **Show Leading icon** | `showLeadingIcon` + `leadingIcon` | boolean + Material Symbol name |
| **Show Tailing icon** | `showTailingIcon` + `trailingIcon` | boolean + Material Symbol name (Figma spelling *Tailing*) |

**Size matrix (Figma App):**

| Size | Height | Icon | Gap | Type |
|------|--------|------|-----|------|
| Small | 40px | 16 | 6px | `--type-size-base` (16) |
| Medium | 48px | 20 | 8px | `--type-size-lg` (18) |
| Large | 56px | 24 | 8px | `--type-size-lg` (18) |

Padding X = `24px` (`--dim-space-600`) · radius = pill · font = `--type-family-heading-graphic2` regular. Fill · Default uses `--mobile-button-shadow-fill` (Brand Drop Shadow Bottom/300).

**State matrix** — every Style resolves to 5 states (`default` / `hover` / `active` / `focus` / `disabled`). The renderer tracks hover/focus/pressed; callers can force a state via props (used by `/m/components`). Focus ring uses CSS `outline` outside the box.

| Component | Figma node | Styles | Notes |
|-----------|-----------|--------|-------|
| [`BrandButton`](src/components/mobile/button/BrandButton.jsx) | [231:29](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=231-29) | Fill · Outline · Ghost | Primary CTA |
| [`BrandSubdueButton`](src/components/mobile/button/BrandSubdueButton.jsx) | [2024:3589](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3589) | Fill · Outline | Soft brand · no Ghost |
| [`DangerButton`](src/components/mobile/button/DangerButton.jsx) | [2024:3590](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3590) | Fill · Outline · Ghost | Destructive |
| [`DangerSubdueButton`](src/components/mobile/button/DangerSubdueButton.jsx) | [2024:3592](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3592) | Fill · Outline | Soft destructive · no Ghost |
| [`NeutralButton`](src/components/mobile/button/NeutralButton.jsx) | [2024:3591](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-3591) | Fill · Outline · Ghost | Greyscale secondary |
| [`NeutralSubdueButton`](src/components/mobile/button/NeutralSubdueButton.jsx) | [2024:2940](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2024-2940) | Fill · Outline | Soft neutral · no Ghost |
| [`BrandIconButton`](src/components/mobile/button/BrandIconButton.jsx) | 377:11596 | pill (square) | Icon-only |
| [`BrandSubdueIconButton`](src/components/mobile/button/BrandSubdueIconButton.jsx) | 2024:4352 | pill (square) | Icon-only · soft brand |
| [`DangerIconButton`](src/components/mobile/button/DangerIconButton.jsx) | 2024:3701 | pill (square) | Icon-only · destructive |
| [`DangerSubdueIconButton`](src/components/mobile/button/DangerSubdueIconButton.jsx) | 2024:4434 | pill (square) | Icon-only · soft destructive |
| [`NeutralIconButton`](src/components/mobile/button/NeutralIconButton.jsx) | 2024:4189 | pill (square) | Icon-only · neutral |
| [`NeutralSubdueIconButton`](src/components/mobile/button/NeutralSubdueIconButton.jsx) | 2024:4516 | pill (square) | Icon-only · soft neutral |
| [`BrandCircleIconButton`](src/components/mobile/button/BrandCircleIconButton.jsx) | 2103:548 | circle (50%) | Force-circular |
| [`BrandCircleSubdueIconButton`](src/components/mobile/button/BrandCircleSubdueIconButton.jsx) | 2103:645 | circle (50%) | Force-circular · soft brand |
| [`BrandSubdueFloatButton`](src/components/mobile/button/BrandSubdueFloatButton.jsx) | 2293:11088 | circle (50%) | FAB · drop-shadow |
| [`HistoryListFormButton`](src/components/mobile/button/HistoryListFormButton.jsx) | 3360:2711 | pill | Wide row · forced Medium |
| [`KeypadButton`](src/components/mobile/button/KeypadButton.jsx) | [314:43835](https://www.figma.com/design/4WXeI8fgBLQGgQ9VheM2yD/-Archive--Mobile-Application?node-id=314-43835) | circle 72 | PIN keypad · Number / icon · Default / Pressed |

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

> ⚠️ **Deprecated**: `MobileButton` with `variant='primary' | 'secondary' | 'ghost' | 'danger' | 'neutral'` is kept as a translation shim ([MobileButton.jsx](src/components/mobile/MobileButton.jsx)) so existing `/m/*` pages keep working. New code MUST use the canonical Figma names above.

#### KeypadButton — Archive `314:43835`

> Mobile · [`KeypadButton.jsx`](src/components/mobile/button/KeypadButton.jsx) · Live: [`/m/components#keypad-button`](/m/components#keypad-button)

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
| [`Password`](src/components/mobile/Password.jsx) | Password.jsx | `PasswordField` (§6) | Figma App [`5021:1008`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=5021-1008) · pill · lock + eye · Large 56 / Medium 48 |
| [`Textarea`](src/components/mobile/Textarea.jsx) | Textarea.jsx | `Textarea` (§6.12) | Figma App [`268:26`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=268-26) · min-h 96 · radius-200 · Default/Press/Filled/Error/Disabled |
| [`SuccessCard`](src/components/mobile/SuccessCard.jsx) | SuccessCard.jsx | — | Archive [`178:122765`](https://www.figma.com/design/4WXeI8fgBLQGgQ9VheM2yD/?node-id=178-122765) · teal gradient + arch card (radius-1000 / 600) · award hero · optional rows / dual CTA |
| [`MobileCard`](src/components/mobile/Card.jsx) | Card.jsx | `Card` (web) | Radius 16 vs 8; subtle elevation opt-in |
| [`ServiceRequestCard`](src/components/mobile/ServiceRequestCard.jsx) | ServiceRequestCard.jsx | — | Archive [`194:153209`](https://www.figma.com/design/4WXeI8fgBLQGgQ9VheM2yD/?node-id=194-153209) · 160w · radius-600 · Colorful icon Badge pill · Home ส่งคำขอ |
| [`MobileInput`](src/components/mobile/Input.jsx) | Input.jsx | `InputField` (§6.12) | 16px base font (iOS zoom guard); 48h; pill-leaning radius |
| [`MobileField`](src/components/mobile/Field.jsx) | Field.jsx | `FormControl` wrapper | Stacks label / hint / error around any control |
| [`MobileBadge`](src/components/mobile/Badge.jsx) | Badge.jsx | soft tone chips | countdown / distance pills — prefer ColorfulBadge for categorical tags |
| [`ColorfulBadge`](src/components/badge/ColorfulBadge.jsx) | shared web | Figma App [`2076:8413`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=2076-8413) · 11 styles × Small/Default × Default/Outline × Primary/Secondary |
| [`MobileHeader`](src/components/mobile/Header.jsx) | Header.jsx | `Header` (§6.19) | 56h app-bar pattern; back arrow + title + right-slot; `transparent` mode over hero gradient |
| [`MobileSectionLabel`](src/components/mobile/SectionLabel.jsx) | SectionLabel.jsx | — | Section header above content blocks |
| [`MobileShell`](src/components/mobile/MobileShell.jsx) | MobileShell.jsx | — | Phone frame + status bar + bottom nav + auth gate |
| [`Sym`](src/components/mobile/Sym.jsx) | Sym.jsx | `Sym` (web) | Same shape, separate file so the mobile tree has no web-only coupling |

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

> Mobile · [`Password.jsx`](src/components/mobile/Password.jsx) · Live: [`/m/components#mobile-password`](/m/components#mobile-password)

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

> Mobile · [`Textarea.jsx`](src/components/mobile/Textarea.jsx) · Live: [`/m/components#mobile-textarea`](/m/components#mobile-textarea)

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

> Mobile · [`SuccessCard.jsx`](src/components/mobile/SuccessCard.jsx) · Live: [`/m/components#mobile-success-card`](/m/components#mobile-success-card)

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

> Shared · [`ColorfulBadge.jsx`](src/components/badge/ColorfulBadge.jsx) · Live: [`/m/components#mobile-colorful-badge`](/m/components#mobile-colorful-badge) · Web: `/components`

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

The shell wraps the page in a 402×874 iPhone 17 mock so desktop reviewers can demo the citizen flow. On touch devices ≤ 430 wide the chrome collapses and the app fills the viewport.

### 7.4. What does **not** exist yet on mobile

These web primitives have **no** mobile equivalent today. Pages that need them must either compose from existing mobile primitives or motivate the new addition in a PR:

- `Table` — long list patterns use stacked `MobileCard` rows instead.
- `Tabs` — segmented control should land here when first needed.
- `Accordion` — `MobileCard` + collapsible state is the current pattern.
- `Dropdown` / `Select` — bottom-sheet picker is the mobile-native pattern; not yet built.
- `DatePicker` / `TimeSlot` — wizards currently use a custom grid (see [MobileBookPage.jsx](src/pages/mobile/MobileBookPage.jsx)). A canonical `MobileDatePicker` belongs here.

When adding a new mobile primitive: create the file under [src/components/mobile/](src/components/mobile/), add tokens to §7.2, and register the entry in §7.3's component-map table.

### 7.5. Figma reference

**Mobile button source-of-truth** — file `opz7X9AAKGQf5prHhsPDGn` (MIH-Design-System-Foundation Copy), Button section `319:2012`. The 16 frames in that section map 1:1 to the 16 files in [src/components/mobile/button/](src/components/mobile/button/) — see §7.3 table for per-component node IDs.

**Web button source-of-truth** — file `GXI1CGpLhpImm8JYpW9ksh`, Button section `319:2012`. Web buttons live at [src/components/button/](src/components/button/) and use the same component names (BrandButton, etc.) with web-specific tokens (radius-150 / 4-6 px, no pill).

**Password source-of-truth** — Figma App file `opz7X9AAKGQf5prHhsPDGn`, Password set [`5021:1008`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=5021-1008) → [`Password.jsx`](src/components/mobile/Password.jsx).

**Textarea source-of-truth** — Figma App file `opz7X9AAKGQf5prHhsPDGn`, Textarea set [`268:26`](https://www.figma.com/design/opz7X9AAKGQf5prHhsPDGn/MIH-Design-System-Foundation--App-?node-id=268-26) → [`Textarea.jsx`](src/components/mobile/Textarea.jsx).

Other mobile primitives (Card / Input / Field / Badge / Header / SectionLabel / Shell) do not yet have a Figma master — when they do, add Figma node IDs to the §7.3 component map and drop the `Mobile*` prefix following the button-family convention.

---
