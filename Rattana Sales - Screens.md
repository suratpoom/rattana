# Rattana Sales — Screens

Two reference screens from the Rattana Sales mobile app. Use as design context in a Claude chat.

---

## 01 · Route & Customer List

> รูทวันนี้ — today's route view

![Route screen](uploads/pasted-1778727496719-0.png)

**Elements**
- App header — hamburger menu, brand mark `RATTANA SALES`, version pill `v5.0`, refresh, theme toggle, home
- Route header — route name (`รูทวันนี้`), weekday · date · area (`อังคาร · 2026 / 05 / 12 · เขต ราชบุรี`), progress bar
- Counters — `0 / 6 เสร็จ`, completion ring (`0%`)
- Search + view toggle (list / map)
- Segmented tab — `ในรูท (6)` · `นอกรูท (22)`
- KPI strip — `เช็คอินแล้ว` · `ค้าง` · `รวม`
- Customer card — facade illustration, debt chip (red warning `฿834,970`), shop name (`ไชยยงค์การค้า`), address, phone CTA, Google Maps CTA, check-in time pill (`09:56`)

**Palette**
- Background: dark navy `#0B1426`
- Surface (cards): `#FFFFFF`
- Accent: gold `#E8B23A`
- Warning: red `#D63A3A`
- Success: green `#1F9D6B`

---

## 02 · Order Screen

> `ใบสั่งซื้อ #PO-26051201`

![Order screen](uploads/pasted-1778727485113-0.png)

**Elements**
- Navy header — back button, PO number, customer thumbnail + name + address
- Product search input (`ค้นหาสินค้า… (7 รายการ)`) with barcode-scan button
- Per-product card, top → bottom:
  1. Name line — product name, `PROMO` chip (when promo), unit selector (`EA / PA / CS`), pallet chip (`75CS`)
  2. **Stock row** — five depots `W1 W2 W3 W4 C4`, each a chip with mini vertical bar + qty
     - Bar **height** = relative to max depot for this product
     - Bar **color** = absolute level: `0` out (gray + strikethrough), `<30` low (red), `<100` mid (amber), `100+` ok (green)
     - Header total: `รวม 680 CS`
  3. Qty stepper · Price input (`฿ 635`) with tier badge (`S03`)
  4. **Tier ladder** — 4 tiles (`S03 <5 CS` / `S02 5–19` / `S01+2 ≥20*` / `S01 ≥20`), price below; active tier is tinted/ringed
  5. Footer — `ตั้งจำนวนเพื่อยืนยัน` left, `ยืนยัน` CTA right

**Tier logic**
| Tier | Qty (CS) | Note |
|------|----------|------|
| S03  | < 5      | Highest unit price |
| S02  | 5–19     | Mid |
| S01+2 | ≥ 20 *   | Restricted (AllowBelowSuggest = 0) |
| S01  | ≥ 20     | Lowest unit price |

**Card surface**
- Cards: white on warm beige scroll background (`linear-gradient(#ECE9E2 → #E8E4DA)`)
- Layered card shadow lifts off background
- Active product (pending qty): gold-tinted with halo; confirmed: green-tinted

---

## Type & spacing

- UI font: system stack (`-apple-system, BlinkMacSystemFont, Segoe UI`)
- Numerals: monospaced (`ui-monospace`) for prices, qty, stock numbers
- Card padding `12px`, radius `var(--r-lg)` (~14px)
- Chip / pill radii `99px` for tags, `8px` for tiles

---

*Generated from live screenshots. Inline-viewer HTML version: `Rattana Sales - Screens.html`.*
