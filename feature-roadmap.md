# Termodan Portal — Feature Roadmap & Product Spec

## Status legend
- ✅ Done
- 🔄 In progress / partially done
- ⬜ Not started
- 🔁 Mockup differs from real product (see delta notes)

---

## Source files

- **App:** `/Users/israelbiton25/claude-skills/.agents/skills/web-artifacts-builder/termodan-portal/src/App.tsx`
- **CSS:** `/Users/israelbiton25/claude-skills/.agents/skills/web-artifacts-builder/termodan-portal/src/index.css`
- **Backup (pre-admin):** `App.backup.tsx` + `mockup.backup.html`

Rebuild command:
```bash
cd /Users/israelbiton25/claude-skills/.agents/skills/web-artifacts-builder/termodan-portal
export PATH="$HOME/.npm-global/bin:$PATH"
bash ../scripts/bundle-artifact.sh
cp bundle.html /Users/israelbiton25/beber-assistant/projects/termodan/sales-portal-mockup/mockup.html
```

---

## Mockup vs. Real Product — Key Differences

These are features intentionally simplified in the mockup that must be built properly in production.

| Area | Mockup | Real Product |
|------|--------|--------------|
| **Authentication** | No real auth — any credentials work | Proper auth (JWT / session), role-based access, potentially SSO for enterprise accounts |
| **Product prices** | Hardcoded from Priority ERP Excel export (one-time import) | Per-customer pricing pulled live from Priority ERP (Phase 5.2) |
| **Inventory** | Mock in-stock / out-of-stock per product | Live inventory from Priority ERP in real time (Phase 5.1) |
| **Order submission** | Local state only — confirmation screen is cosmetic | POST to Priority API → creates real order in ERP (Phase 5.3) |
| **Order status** | Static mock statuses | Synced from Priority delivery workflow (Phase 5.4) |
| **Delivery zones & pricing** | Hardcoded zone map + costs | Pulled from Priority ERP per delivery zone |
| **Delivery date** | Calendar UI only — date is not sent anywhere | Submitted with order, used for logistics planning |
| **Truck calculation** | Frontend-only, for display purposes | Same logic but server-side; feeds into logistics system |
| **Chat widget** | Keyword-matching on hardcoded Q&A pairs | RAG-backed AI over Termodan's product catalog + FAQ; escalation routes to WhatsApp / CRM |
| **Obligo / credit limit** | Grayed placeholder card | Pulled from Priority ERP; warnings + order block when exceeded |
| **Drafts (save cart)** | In-memory only — lost on page refresh | Persisted per user in backend database |
| **Company admin portal** | Hardcoded 3 mock sites with static order data | Dynamic per-company accounts; sites and users managed by Termodan in the backend |
| **Documents (delivery notes)** | Stub "הורדה" button — no real file | Real PDFs generated from Priority ERP delivery data |
| **Contact info** | Placeholder: 08-600-1000, service@termodan.co.il | Real Termodan contact details per client relationship |
| **Analytics** | Deferred — not in mockup | Pulled from order history and Priority data; spend summaries, product rankings |
| **Alternative / complementary products** | Hardcoded in App.tsx (`COMPLEMENTARY_MAP` + `alternativeProductId` fields) | Must be managed in Priority — define related/substitute item links per SKU so Termodan controls what's suggested, not random similar-looking products |

---

## Phase 1 — Core Portal ✅ (complete)

- ✅ Login screen — branded split layout, Termodan logo + background image
- ✅ Register flow — form + success state
- ✅ Dashboard — stats cards, quick actions, order history table, drafts section
- ✅ Catalog — 3-level accordion sidebar (Main → Sub → Products), product grid, starred/my products
- ✅ Product detail — specs, variant selectors (color + finish + pallet size), quantity stepper, pallet calculator, complementary products strip
- ✅ Cart — item rows, pallet fill progress, truck utilization bar, upsell nudge, cost summary
- ✅ Checkout — delivery form, saved addresses, zone detection + delivery cost, date picker, cost breakdown, confirmation checkbox
- ✅ Confirmation screen — success state, WhatsApp/SMS note
- ✅ Order detail — status timeline, items table, contact card, download stub
- ✅ Profile screen — personal info

---

## Phase 2 — Intelligent Features

### 2.1 ✅ מחשבון כמויות — Quantity Calculator
- Embedded in product page below the main product card
- Input: measurement value + unit selector (מ"ר / מטר קוב / מטר אורך — varies by product type)
- Output: required units, pallets, estimated price, "הוסף לעגלה" button
- 10% reserve applied automatically, displayed transparently
- Hidden for materials (חומרים ועזרים) — no area/volume conversion applicable
- Float precision fix: `Math.ceil(Math.round(val * rate * 1.10 * 10000) / 10000)`

### 2.2 ✅ בוט מידע חכם — Smart Info Chat Widget 🔁
- Floating chat bubble (bottom-left corner, always visible in portal screens)
- Pre-canned Q&A: adhesive recommendations, block specs, delivery questions
- Escalation path: "העבר לנציג" button when question is complex
- Suggested questions shown on open (3-4 quick chips)
- Simulated "typing" delay for realism

**🔁 Mockup vs. real:** Mockup uses keyword-matching to hardcoded answers. Real product should be backed by an AI model (RAG over Termodan's product catalog and FAQ). Escalation should route to a human agent via WhatsApp or CRM. Strong candidate for AI investment — reduces support load significantly.

### 2.3 ✅ הזמן שוב — Reorder (Duplicate Order)
- "שכפל" on each order row and in order detail — loads that order's items into the cart, replacing current contents
- "שכפל הזמנה אחרונה" quick action on dashboard

**Product decisions:**
- Duplicate replaces current cart (not merge) — avoids confusion with mixed-order carts
- Scheduled recurring orders dropped entirely — in this industry, exact repeat orders on a fixed schedule almost never happen

### 2.4 ✅ Smart Order Optimization Notifications
- Cart: truck utilization bar with smart truck type calculation (דאבל/פול)
- Cart: "יש מקום ל-X משטחים נוספים ב[truck type] ללא עלות הובלה נוספת" — fill-up nudge
- Cart: multi-truck info banner when order requires more than one truck
- Checkout: "עלות הובלה למשטח ≈ ₪X" — cost-per-pallet transparency line

**Product decisions:**
- Truck types: דאבל = up to 12 pallets / 26 tons; פול = up to 24 pallets / 36 tons
- Truck selection: greedy פול-first, then דאבל for remainder
- Binding constraint: whichever limit (pallets or weight) is hit first per order
- Materials (barrels, adhesive bags) excluded from truck calculation — accessories, not palletized cargo
- No portal discounts — all pricing from Priority ERP per customer

**🔁 Mockup vs. real:** Truck calculation is frontend-only for display. Real product runs this server-side and feeds into the logistics/dispatch system.

### 2.5 ⬜ אנליטיקס אישי — Personal Analytics (deferred)
- Spend summary banner, month-over-month chart, top products, Excel export stub
- Deferred — not a flow decision for Mosh sign-off

---

## Phase 3 — Ordering Flow Extensions

### 3.1 ✅ Preferred Delivery Date Selection 🔁
- Full calendar in checkout below the address section
- "בהקדם האפשרי" default option
- 48h lead time enforced — earlier dates and Saturdays disabled
- Selected date shown as confirmation line; note that date is not guaranteed

**Product decisions:**
- No Saturday delivery; Friday is okay
- 48h lead time; if 48h lands on Saturday, snaps to Sunday
- Requested date is a preference — rep confirms actual date

**🔁 Mockup vs. real:** Date is captured in UI only — not submitted anywhere. Real product sends it with the order to Priority for logistics planning.

### 3.2 ✅ Draft Orders / Save Cart 🔁
- "שמור טיוטה" button in cart — saves current items with timestamp
- Drafts section in dashboard above order history — shows summary, total, resume + delete
- Resuming a draft replaces current cart

**🔁 Mockup vs. real:** Drafts are in-memory only — lost on page refresh. Real product persists drafts per user in the backend.

### 3.3 ✅ Contact Section (replaces Order Cancellation)
- Sidebar footer: persistent phone + WhatsApp block
- Order detail screen: "שאלה על ההזמנה?" card with WhatsApp, phone, email buttons

**Product decisions:**
- Order cancellation and modification dropped entirely — Priority ERP is the system of record; the portal cannot modify submitted orders
- Each client has a dedicated Termodan sales agent as primary contact
- Placeholder contact info (08-600-1000, service@termodan.co.il) — replace before demo

### 3.4 ⬜ Digital Signature Flow (deferred — legal/ops detail)

### 3.5 ⬜ Delivery Confirmation + Rating (deferred — secondary flow)

### 3.6 ⬜ Delivery Document Download 🔁
- Stub "הורדה" button exists in order detail
- **🔁 Real product:** Real PDFs generated from Priority ERP delivery data

---

## Phase 4 — Company Admin Portal ✅ (redefined)

> **Original spec** described a Termodan back-office admin panel. **Redefined:** the admin portal is for company-level clients (large contractors with multiple construction sites) who manage all their sites' activity in one place. Separate login, controlled by Termodan.

### 4.1 ✅ Admin Login (via "כניסה כמנהל חברה" panel on login screen)
- Second login form on the login screen — any credentials trigger admin mode
- Completely separate portal: different layout, different sidebar, no cart/catalog

### 4.2 ✅ Company Admin Dashboard
- 4 stat cards: total spend, active orders, number of sites, shipments in transit
- Sites table: name, manager, last order, active orders, total spend, "כניסה לאתר" button

### 4.3 ✅ Site Detail View
- Orders tab: full order history for that site with status badges
- Documents tab: stub delivery notes list with "הורדה" buttons

**🔁 Mockup vs. real:**
- Mockup: 3 hardcoded mock sites (גבעת שמואל, ירושלים, תל אביב) with static order data
- Real: dynamic per-company configuration; sites, users, and permissions managed by Termodan in the backend
- Real: documents are actual PDFs from Priority ERP
- Real: admin should be able to drill into a site and see live status updates

### 4.4 ⬜ Termodan Back-Office Admin (separate scope — not in mockup)
- Original 4.x features (orders management, customer management, product/catalog management) are Termodan-internal tooling
- Out of scope for the customer-facing portal mockup

---

## Phase 5 — ERP & Business Integration (all deferred — real build only)

### 5.1 ⬜ Live Inventory from Priority
### 5.2 ⬜ Per-Customer Pricing from Priority
### 5.3 ⬜ Order Submission to Priority API
### 5.4 ⬜ Status Sync from Priority
### 5.5 ⬜ Obligo / Credit Limit
### 5.6 ✅ Company Admin Login — partially delivered as the admin portal (Phase 4)

---

## Phase 6 — Multi-User / Org Structure (deferred — real build only)

### 6.1 ⬜ Sub-Users Under One Company (invite, roles, addresses)
### 6.2 ⬜ Role-Based Permissions (view / order / approve)
### 6.3 🔁 Per-Site Visibility — partially demonstrated via company admin portal (Phase 4): admin sees all sites, site buyer sees only their own

---

## Approved Implementation Plan — Status Summary

| Step | Feature | Status |
|------|---------|--------|
| 0 | Real product data (21 SKUs from Priority ERP) | ✅ |
| 1 | Chat widget (2.2) | ✅ |
| 2 | Smart order notifications (2.4) | ✅ |
| 3 | Reorder / duplicate (2.3) | ✅ |
| 4 | Delivery date picker (3.1) | ✅ |
| 5 | Contact section — replaces cancellation (3.3) | ✅ |
| 6 | Draft orders / save cart (3.2) | ✅ |
| 7–9 | Company admin portal (4.1–4.3) | ✅ |
| — | Termodan back-office admin (original 4.x) | Dropped — out of portal scope |

**All planned mockup features are complete. Ready for Mosh review.**

---

## Notes & Ideas (running list)

Free-form notes from working sessions — things to revisit, open questions, and ideas that came up but aren't scheduled yet.

- **Alternative/complementary items managed in Priority** — the portal currently hardcodes related SKUs. In production, these relationships should be defined inside Priority ERP per item (substitute items, complementary items) so Termodan's team can maintain them without touching code. The portal fetches them via API at runtime.

---

## Before Demo — Checklist

- [ ] Replace placeholder contact info: 08-600-1000 → real number; service@termodan.co.il → real email
- [ ] Confirm WhatsApp number in sidebar and order detail card
- [ ] Review mock product prices — confirm they match current pricelists
- [ ] Confirm admin mock company name (כהן בנייה בע"מ) and site names are appropriate for demo context
