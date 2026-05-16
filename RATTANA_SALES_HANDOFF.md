# Rattana Sales App — Handoff Document
**Current version: v6.5.4** · 2026-05-14 · 11,620 lines · 563 KB

> Paste this entire doc as the FIRST message in your new chat, then attach `rattana-sales-app.html`.

---

## 0. How to start the new chat

1. Upload `rattana-sales-app.html` (latest = v6.5.4)
2. Paste this document
3. Say what you want to work on next (e.g. "Task 4: Customer UI redesign → v7.0")

**Critical rule for next Claude:** read the existing file before changing anything. Use `str_replace` for surgical edits — never rewrite from scratch. The `preserve-existing-code` skill applies. Bump version to **v7.0** since it's a new chat.

---

## 1. Project Identity

- **File:** `rattana-sales-app.html` — single-file HTML PWA
- **Owner:** Surat (ภูมิ), Role 7, บริษัท รัตนไพบูลย์ สมุทรสงคราม 2555
- **Deployed at:** https://rattanaphaiboon.github.io/app/rattana-sales-app.html
- **Working dir in new chat:** copy upload from `/mnt/user-data/uploads/` → `/home/claude/` → edit → output to `/mnt/user-data/outputs/rattana-sales-app.html`
- **Filename is stable** — never change it (breaks deploy link)

## 2. Tech Stack

- **Frontend:** Single-file HTML + vanilla JS + CSS variables (no framework, no bundler)
- **Auth:** Google OAuth (`615875645128-gasjjvkt6lu8g449cbnhl40k1pu25r0b.apps.googleusercontent.com`)
- **Backend Sheet:** `1dAxF4Mz-zhJIVbuJ_Ql126gt08qvVS1wndJD50pqCZI`
- **User Sheet:** `1M6HdISsLN684qRWyQ73CA4AmUzmYtZaOlffDJXZZIXQ` (tab `Sheet1`)
- **Apps Script:** `https://script.google.com/macros/s/AKfycbxGt9gaXDyunLu_BVBpEdY0zwM49ihlphxh5d-3FjhlXXDhkoLNP8gWzxDb-PJ1LxdT/exec`
- **Cache:** IndexedDB (`idb`) for products/customers/promotions; **NEVER for orders** (rule #1)

## 3. Hard Rules — never violate

1. **NEVER restore orders from idb cache on boot.** Orders fetched fresh from Sheet every session. Clear `rsa_orders` localStorage on every `enterApp()` call.
2. **NEVER hardcode `background: #fff`** or `linear-gradient(#ffffff...)` on cards/modals. Always use `var(--card-bg)`. Dark mode = `body.dark` class saved to localStorage key `rattana_dark`.
3. **NEVER bypass/hardcode login.** `checkSession()` must call `showLogin()` when no session. No hardcoded email/name/role fallback.
4. **PRICE CC00-CC03 columns are cost prices** — IGNORE in this app (reserved for future shop-front app).
5. **AVERAGE 60D_W is daily EA rate, ALREADY AVERAGED. DO NOT divide by 60.** The "60D" indicates lookback window only. Formula: `parseStockCsEa(STOCK_W).totalEa / AVERAGE_60D_W → days`.
6. **STOCK_W format = "CS.EA"** (e.g. `'6804.10'` = 6804 CS + 10 EA, totalEa = 6804×csSize + 10).
7. **Edit surgically.** Use `str_replace`. Never regenerate untouched code from memory. Read full file first.

## 4. Product Sheet v2 Column Order (2026-05-14)

```
PRODUCT NAME | CODE EA/PA/BP/CS | RATTANA UNIT EA/PA/BP/CS |
UNIT EA/PA/BP/CS (e.g. 'ชิ้นx1(EA)', 'แพ็คx10(PA)', 'ลังx12(CS)') |
QTY EA/PA/BP/CS | CS PALLET | CS LAYER | NUMBER OF LAYERS |
PRICE CC00-03 EA/PA/BP/CS (cost—ignore) |
PRICE S00/S01/S02/S03 EA/PA/BP/CS |
CAT BU/TYPE/GROUP/BRAND/SIZE/VENDOR |
VENDOR CODE | VENDOR NAME |
SKU LIFE (DAYS) | UNIT VALUE | DESCRIPTION | FIRST STOCK DATE | SHORT NAME |
STOCK W1..C4 | AVERAGE 60D_W1..C4 | RANK W1..C4 (A/B/C/D/F) |
IB W1..C4 (last inbound dd/mm/yyyy) |
SKU_USAGE (Google Drive image URL)
```

**Key field semantics:**
- `STOCK_W` format `"CS.EA"` (e.g. `6804.10` = 6804 CS + 10 EA)
- `AVERAGE 60D_W` = **daily EA/day** (already averaged over 60-day window) — use as-is
- `RANK W` = A/B/C/D/F sales rank per depot — **not yet displayed**, future enhancement
- `IB W` = last inbound date per depot — **not yet displayed**, future enhancement
- `SKU_USAGE` = product photo URL (primary source)
- `UNIT EA/PA/BP/CS` labels = e.g. `'ลังx12(CS)'` — **not yet used in UI** (still hardcoded EA/PA/BP/CS)

## 5. User Sheet Critical Field Mappings

`buildUserFromRow()` at line ~3191. **Field name confusion was a real bug in v6.5 → fixed in v6.5.1:**

| User Sheet column | `currentUser` key | Used for |
|---|---|---|
| `รหัสพนักงาน` | `empId` | Employee ID (NOT used in Order Sheet) |
| `รหัสเซลล์` (column AD) | `salesmanCode` | Order Sheet `รหัสเซลล์` column — sales code |
| `W` | `warehouse` | Order Sheet `คลัง` column — sales team / sangkat |
| `คลังส่ง` | `shipFromWh` | Order Sheet `คลังส่ง` column — depot to ship from (may differ from W) |
| `รหัสขาย Bplus` | `bplusCode` | Bplus ERP code (not in Order Sheet) |
| `AllowBelowSuggest` | `allowBelowSuggest` | Boolean — false = can't go below suggested tier price |

`รหัสเซลล์` (with ์) is in column AD of User Sheet. Don't confuse with `รหัสเซล` (legacy, no ์) — fallback only.

## 6. Order Sheet Columns (current orderRow shape)

`orderRow` in `orderConfirm()` (line ~7220). Add these headers to Order Sheet **manually** (Apps Script uses key matching, position doesn't matter):

```
1.  วัน              (date ISO)
2.  เวลา             (Thai time)
3.  email
4.  ชื่อ-สกุล
5.  รหัสเซลล์         ← from User Sheet column AD
6.  คลัง              ← User Sheet 'W' (sales team)
7.  คลังส่ง           ← User Sheet 'คลังส่ง' (NEW v6.5)
8.  ชื่อร้าน
9.  รหัสร้าน
10. รูปแบบ            ('ขาย' or 'แถม')
11. Barcode
12. ชื่อสินค้า
13. จำนวน
14. หน่วย             (e.g. 'ลังx12(CS)')
15. ราคา              (actual price billed)
16. ราคาแนะนำ         ← suggested tier price (NEW v6.5)
17. โปรที่ใช้          ← name of promo user picked (NEW v6.5.2; blank if none)
18. ยอดเงินรวม
19. orderId
20. หมายเหตุ
```

## 7. Version History (this thread)

| Version | Major changes |
|---|---|
| v6.0 | Order modal stock section, beige backdrop, depot chips |
| v6.2 | Inline order cards (no popup), tier ladder, pallet chip, promo pin |
| v6.3 | Card color/layout fixes, 5-state system (PROMO red border, focus gold, pending gold, confirmed green, idle clean) |
| v6.3.1 | AVERAGE 60D initial — wrongly divided by 60 |
| v6.4 | Stock calc fix (no ÷60), bigger thumb 56→72px, price centered, gray bar when no avg |
| v6.5 | Promo bar inline replaces pin, SKU_USAGE photo everywhere, Order Sheet new columns `คลังส่ง` + `ราคาแนะนำ` |
| v6.5.1 | `รหัสเซลล์` column AD mapping fix |
| v6.5.2 | Multi-promo bars, removed red/gold border-left, tile white at qty=0, FAB lifted 90→152px, `โปรที่ใช้` column |
| v6.5.3 | **Price guardrails restored** (Ceiling = S03 × 1.10, Floor = S00, tier lock for allowBelowSuggest=false users) |
| **v6.5.4** | **CURRENT** — Promo math fixed (per-SKU convert, not via sample); auto-pick single promo; multi-promo gated until user selects; Summary screen respects per-line `selectedPromoName` |

## 8. Promo Logic Matrix (CRITICAL — has nuances)

### 8.1 Promo `วิธีวัด` field

| วิธีวัด | What it matches |
|---|---|
| `PRODUCT NAME` | Specific SKU only |
| `CAT SIZE` | All SKUs sharing CAT SIZE value |
| `CAT BRAND` | All SKUs in same brand (e.g. ช้างคลาสสิก) |
| `CAT TYPE` | Broader product type |
| `CAT GROUP` | |
| `CAT VENDOR` | All SKUs from same vendor |
| `CAT BU` | Business unit (e.g. Colgate) |

Specificity ranking: `PRODUCT NAME > CAT SIZE > CAT BRAND > CAT TYPE > CAT GROUP > CAT VENDOR > CAT BU` (1 = most specific).

### 8.2 Stacking rules (cross-SKU sum behavior)

| รูปแบบขาย | PRODUCT NAME promo | CAT-based promo |
|---|---|---|
| **CS** | SKU-only | ✅ **Stacks across SKUs** (`cartMatchesPromo` per-SKU convert) |
| **PA / BP** | SKU-only | ⚠️ **WARNING — invalid config** (`_validatePromoConfig` blocks, asks user to consult จัดซื้อ) |
| **EA** | SKU-only | ✅ Stacks across SKUs |
| **บาท** | SKU-only | ✅ Stacks (baht is universal unit) |

**Why PA/BP is forbidden for CAT-based:** different SKUs in same brand have different pack sizes (e.g. Deedo รสส้ม PA = 6 EA, Deedo รสองุ่น PA = 12 EA — can't sum "10 PA" meaningfully).

**Why CS is OK for CAT-based:** CS is treated as a business unit ("one carton" = one carton regardless of contents). User explicitly confirmed: "CS = หน่วยใช้งาน ไม่ต้อง convert".

### 8.3 Promo selection rules (v6.5.4)

- **1 promo matches** → auto-pick (no user action)
- **≥2 promos match** → all bars shown as info-only ("เลือกโปรนี้เพื่อดูความคืบหน้า"); user MUST click `[เลือก]` to commit
- **No default** when multi-promo — supervisor needs explicit choice from sales rep
- After pick: only the selected bar runs `cartMatchesPromo` for progress calc; others stay info-only
- Selected promo persists in `_orderDraft[pid].selectedPromoIdx` until unit changes (then cleared — different unit may match different promos)

### 8.4 Reward calculation (Math.floor)

- `tiersEarned = Math.floor(currentValue / threshold)`
- Buying 5 CS at promo 4 CS → 2 EA gives **1 set** (= 2 EA) — extra 1 CS not earning
- No fractional rewards — user confirmed: "ลูกค้าได้แถมตามจำนวนที่ตั้งไว้"

### 8.5 `cartMatchesPromo()` (line ~4111) — math fix in v6.5.4

```js
// OLD (v6.5.x — WRONG): convert via single sample SKU
const sample = matchingLines[0]?.p;
const piecesPerTarget = unitToPieces(sample, sellMode);
currentValue = totalPieces / piecesPerTarget;  // ← 165 instead of 100 when SKUs differ

// NEW (v6.5.4 — CORRECT): convert each line via ITS OWN SKU's QTY ratios
for line:
  piecesPerTarget = unitToPieces(p, _targetUnit)  // per-SKU
  totalInTargetUnit += linePieces / piecesPerTarget
currentValue = totalInTargetUnit  // correct sum across heterogeneous SKUs
```

## 9. Price Guardrails (v6.5.3) — line ~7176

When user types a price in Order card, 3 checks fire in order:

1. **Ceiling** = `Math.floor(S03 × 1.10)` per unit
   - Border yellow + warning "⚠️ เกินราคาสูงสุด ฿X — กดยืนยันจะถูกลดเป็น ฿X"
   - On confirm: cap to ceiling + toast

2. **Tier lock** (only if `currentUser.allowBelowSuggest === false`)
   - User can't drop below suggested tier for current qty
   - Border red + warning "🔒 คุณไม่ได้รับสิทธิ์ลดต่ำกว่า S0X (฿Y)"
   - On confirm: bump up to suggested tier + toast

3. **Floor** = `PRICE S00` per unit
   - Border red + warning "❌ ต่ำกว่าราคาต่ำสุด ฿X — กดยืนยันจะใช้ราคา S0X (฿Y)"
   - On confirm: fallback to **suggested tier price** (NOT floor) + toast

Real-time check: `_runPriceGuardrails()` runs in `_updateCardLive()` on every keystroke.

## 10. 5-state Card Color System (v6.3+)

Cards in Order screen use these state combinations:

| State | When | Visual |
|---|---|---|
| **idle** | qty=0, no PROMO, not in cart | white + soft shadow |
| **PROMO** | SKU matches any promo | red left-accent gradient (v6.3-v6.5.1 had border-left red; **v6.5.2 removed** — promo bar carries signal) |
| **focus** | `isFocusProduct(p) === true` | gold left-accent (similar — also dropped in v6.5.2) |
| **pending** | qty > 0, not yet confirmed | gold halo + gradient |
| **confirmed** | line in cart | green halo + gradient |

In v6.5.2+ the border-left was removed because promo bars (inline) already signal promo presence. Card stays clean — only halo (pending/confirmed) is on the card itself.

## 11. Key Code Locations

| Function/var | Line (~) | Purpose |
|---|---|---|
| `buildUserFromRow` | 3191 | OAuth row → currentUser object |
| `parseStockCsEa(raw, csSize)` | 3897 | "6804.10" → {cs, ea, totalEa, raw, empty} |
| `getPromotionFor(p)` | 3999 | First matching promo (legacy) |
| `getAllPromotionsFor(p)` | 4065 | All matching promos sorted by specificity (v6.5.2) |
| `_validatePromoConfig(promo)` | 4095 | Warn if CAT-based + PA/BP |
| `cartMatchesPromo(promo, draftItem)` | 4111 | Per-SKU correct progress calc (v6.5.4 rewrite) |
| `unitToPieces(p, unitCode)` | 4187 | EA conversion per SKU |
| `buildDepotChipsCompact(p)` | 6545 | Order card stock bars |
| `buildDepotChips(p)` | 7134 | Modal stock bars (legacy) |
| `_buildPromoBarsHtml` | 6718 | All promo bars renderer (v6.5.2+) |
| `_buildOnePromoBar` | 6749 | Single promo bar (v6.5.4 takes `isMulti` flag) |
| `_buildPromoBarHtml` (legacy) | (deprecated) | Single-promo legacy stub |
| `orderPromoPick(safeId, idx)` | (helper) | Radio-style promo selector handler |
| `getOrderTiers(p, unit, qty, pickedTier)` | 6500 | Tier ladder state (S03/S02/S01+2/S01) |
| `getPriceBounds` / `_getPriceGuardrails` | 7415 | Ceiling/floor/tier-lock validation |
| `_runPriceGuardrails(el, p, d, unitCode)` | 7464 | Visual feedback (border + warn text) |
| `buildOrderCardV6(p, idx)` | 6742 | Single order card HTML |
| `_updateCardLive(safeId)` | 7167 | Live refresh on qty/unit/price change |
| `orderConfirm(safeId)` | 7147 | Push to cart with guardrails + selectedPromo |
| `buildPromoGroups()` | 8723 | Summary screen — filters by `selectedPromoName` (v6.5.4) |

## 12. Draft State (`window._orderDraft[productId]`)

Each product's working state before confirm:
```js
{
  unit: 'case' | 'pack' | 'bigpack' | 'piece',
  qty: number,
  price: number | '',
  pickedTier: 'S03' | 'S02' | 'S01+2' | 'S01' | undefined,  // user override
  selectedPromoIdx: number | undefined,                       // chosen promo index in getAllPromotionsFor
  status: 'confirmed' | undefined,                            // set after orderConfirm
}
```

## 13. Cart Line Shape (after `orderConfirm`)

```js
{
  productId, unit, qty, price,
  shopId: currentShopId,
  suggestedPrice,         // v6.5: tier-suggested price (VAT-incl)
  suggestedTier,          // 'S03' / 'S02' / 'S01+2' / 'S01'
  selectedPromoName,      // v6.5.2: empty if no promo or user didn't pick
  selectedPromoMeasureBy, // v6.5.2: 'CAT BRAND' / 'PRODUCT NAME' etc.
}
```

## 14. CSS Variables (key ones)

```css
--navy: #08142e      --navy-mid: #0f2044     --navy-light: #1a3060
--gold: #d4a843      --gold-light: #f0c547
--card-bg: #ffffff   (dark: #0c1a35)
--row-alt: #eef2f9   (dark: #091526)
--border: ...
--text: ...          --muted: ...
--danger: #D63A3A    --warning: #E8B23A      --success: #1F9D6B
```

Dark mode: `body.dark { --card-bg: #0c1a35; ... }` — all colors flip. Saved to `localStorage.rattana_dark`.

## 15. FAB / Cart Bar Positioning (v6.5.2)

- `.cart-bar` — `position: fixed; bottom: 76px` (above 64px bottom nav + 12px gap)
- `.back-to-top` (FAB ⬆) — `position: fixed; bottom: 152px` (above cart bar + gap)
- `.bottom-nav` — 64px tall, the lowest fixed element

## 16. Known Issues / Out of Scope (open items)

### Not yet implemented (planned future work)

1. **RANK W1..C4 badge** — show A/B/C/D/F sales rank on each depot chip
2. **IB W1..C4 tooltip** — last inbound date "ของเข้าล่าสุด: 12/05/2026"
3. **UNIT EA/PA/BP/CS labels** — use `'ลังx12(CS)'` etc. from Sheet instead of hardcoded
4. **Apps Script Products compression** — 5MB→1MB optimization
5. **Service Worker for PWA offline**
6. **BigQuery SKU suggestions**
7. **GPS privacy decision pending**
8. **Visit reasons via DataCenter sheet**, check-in/out GPS distance-based
9. **KPI Dashboard drill-down popups**

### Customer (Route) UI redesign → v7.0 ⏸️ PENDING

User has explicitly deferred to a new chat. Will need:
- Apply 5-state color system (PROMO red / focus gold / pending gold / confirmed green / idle clean)
- Inline bar pattern (not pin antipattern)
- Debt traffic light per shop
- Last-visit recency bar
- Frequency anomaly badge ("overdue by 1.5x avg gap")
- Credit bar visualization (was already planned earlier)
- Filter by route + delivery day

Reference design docs from session 1 (Screens.md, Screens.html) — Surat should reattach if available.

### Out-of-scope issue noted but not fixed in v6.5.4

- **Summary screen "Promo Picker"** (`renderPromoPickerSection`, line 8748) — has its own radio-choice UI that's somewhat independent of Order screen's `[เลือก]` chip. Currently filters by `selectedPromoName` (v6.5.4 fix) so it shouldn't show ghost promos, but the UI itself could be simplified to read-only display since selection now happens in Order screen. Ask user if they want this consolidation.

## 17. User Style / Communication Preferences

- **Language:** Thai (mostly). Mixed Thai+English fine for technical detail.
- **Workflow:** "Talk first, render preview, THEN build." Don't jump to code without confirmation on UX decisions.
- **Visual previews:** Save standalone HTML preview files to `/mnt/user-data/outputs/` (visualizer tool is unreliable, has timed out multiple times). Mobile-sized (~380px width).
- **Surgical edits:** Read file, identify exact change locations, use `str_replace`. Never regenerate untouched code.
- **Skills used:** `preserve-existing-code`, `rattana-sales-app`, `rattana-app-builder`, `version-bumper`.
- **Version bumping:**
  - Minor changes within same chat: bump patch (v6.5.3 → v6.5.4)
  - New chat / big feature: bump minor or major (v6.5 → v7.0)
  - **Version goes in: title tag, login footer, header pill, drawer footer** — NEVER in filename (breaks deploy)

## 18. Validation Pattern Before Deploy

```bash
cd /home/claude && node -e "
const fs = require('fs');
const html = fs.readFileSync('rattana-sales-app.html', 'utf8');
const scripts = html.match(/<script>([\s\S]*?)<\/script>/g);
let errs = 0;
scripts.forEach((s, i) => {
  const body = s.replace(/^<script[^>]*>/, '').replace(/<\/script>$/, '');
  try { new Function(body); } catch (e) { console.log('ERR:', e.message); errs++; }
});
console.log('Scripts:', scripts.length, '— Errors:', errs);
"
cp /home/claude/rattana-sales-app.html /mnt/user-data/outputs/rattana-sales-app.html
```

Always run this before `present_files`. Zero errors required.

## 19. Recommended Opening Move for Next Chat

```
[Attach: rattana-sales-app.html]
[Attach: RATTANA_SALES_HANDOFF_v6.5.4.md  ← this document]

"Continuing Rattana Sales work. Current = v6.5.4. 
Today I want to work on: [topic]"
```

Next Claude should:
1. Read the handoff doc fully
2. Read the uploaded HTML file (use `view` in ranges if needed)
3. Confirm understanding by stating: "Read file. v6.5.4 confirmed. Ready for [topic]."
4. Wait for direction before any code change.

---

## Appendix A — Promo logic quick reference (5 worked examples)

| # | Cart | Promo | Expected behavior |
|---|---|---|---|
| 1 | ช้าง 620ml × 100 CS | PRODUCT NAME 10 CS → 1 EA | ✅ 10 ชุด earned → แถม 320ml × 10 |
| 2 | ช้าง 320ml 5 CS + 620ml 5 CS | CAT BRAND ช้างคลาสสิก 10 CS → 1 EA | ✅ 1 ชุด (stacks across SKUs since both = CAT BRAND ช้างคลาสสิก, sellMode = CS) |
| 3 | Deedo ส้ม 5 PA + องุ่น 5 PA | CAT BRAND Deedo 10 PA → 1 EA | ⚠️ Warning, no calc (PA can't stack in CAT-based) |
| 4 | Colgate ยา ฿3,000 + บ้วน ฿2,500 | CAT BU Colgate 5,000฿ → ลด 20 | ✅ Stacks (baht universal), 1 round earned |
| 5 | ช้าง 620ml × 20 CS, 2 promos match | PROD NAME 10 CS→1 EA + CAT BRAND 20 CS→3 EA | Both bars show as info-only until user picks. After pick, only chosen one calcs progress. |

## Appendix B — File map mental model

```
/mnt/user-data/uploads/         ← User uploads land here (READ ONLY for you)
/home/claude/                   ← Your working dir (copy uploads here first, edit here)
/mnt/user-data/outputs/         ← Final deliverable here (cp from /home/claude/ at end)
/mnt/skills/user/*/SKILL.md     ← Surat's custom skills (READ before any major work)
/mnt/transcripts/               ← Previous session transcripts (auto-generated, searchable)
```

`present_files(["rattana-sales-app.html"])` from outputs makes it deployable to user.

---

**End of handoff document.**

Generated 2026-05-14 covering v6.5.4 state. Next version will be **v7.0** (new chat boundary).
