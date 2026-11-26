# CYBERBIZ: what is confirmed and what is not

Private working list. Updated 2026-11-26 after two reviews of a live CYBERBIZ shop run by an unrelated business: the public storefront as a signed-out visitor, and the admin panel through a read-only session. That shop is not named here, and details specific to it are left out.

The comparison PDFs contain none of the open points below.

## Limits of the admin review

- **The signed-in account has the Co-managers role.** These screens returned "no permission": Checkout Page & Logistics, Payment Settings, Delivery Settings, Convenience Store Logistics, Reconciliation Center, All Customers, VIP Settings.
- **A second, complete report covered all eleven areas,** including reviews, sold count, design and staff records, which the first one left out.
- **The shop is on the 企業版 plan.** What it shows may not be available on other plans.

## Confirmed on the live shop

| Topic | What was seen |
|---|---|
| Plan | 企業版. API and webhooks are available to custom apps, with scopes for customers, orders and products. |
| Guest checkout | Two registration modes only: "fast checkout with registration" and "registration required". Checkout needs no sign-in or password, but a consent box, ticked by default and required, registers the buyer as a member. |
| Sign-in | LINE, Google, Facebook, and email or mobile with password. |
| Registration fields | Email, mobile, birthday and gender can each be set required or optional and shown or hidden per page. This shop has mobile required and email optional. |
| Verification | Email or SMS activation link after sign-up. Not a one-time code. SMS costs 1 coin (NT$ 1) per domestic message. |
| Checkout | One page. Promo code and referral code fields are shown to signed-out buyers. |
| Marketing consent | A separate tick box, ticked by default. |
| Payment | Card, Apple Pay, Google Pay, LINE Pay, virtual ATM, 街口 and 銀聯, through CYBERBIZ Payments. |
| Delivery | 7-11 and 全家 pickup (prepaid and pay-at-pickup methods both exist), 中華郵政, 黑貓, store pickup, DHL. Fee tiers by order amount, with a free-shipping threshold. |
| Invoices | 會員載具, 統編, 手機載具, 自然人憑證 and 捐贈碼 offered at checkout. |
| Order statuses | Payment, shipping and order status are separate lists. Partial return and partial refund statuses exist. |
| Points | 1 point = NT$ 1 (editable). Expiry can be never. Per-order limit can be "no limit". Sign-up, birthday and review bonuses each have their own field. Points are credited when the order is closed. |
| Points limit per product | Each variant has a "Credit(s)" field capping points on that item. The shop reviewed uses it, capping points at about a quarter of the price. |
| Store credit | No 購物金, 儲值金, 禮物卡 or customer wallet in any menu. |
| Promo codes | Types: fixed amount, percentage, free shipping, gift. Minimum spend, dates, total-use and per-customer limits. No members-only option. No tag or campaign field; a per-code report exists. Code format: Latin letters, digits and hyphen. Checkout here allows 3 codes per order. |
| LINE | Official Account connected. Message templates for new order, payment, cancel, shipping, arrived at store, pay-at-pickup, returns and refunds, each switchable. Email templates for the same events. |
| Order completion message | No email or LINE template is sent when an order completes, and none asks for a review. A webhook event for order completion exists. |
| Order source | A "Source" column (CYBERBIZ / Merchant / POS / Warehouse). No LINE-versus-website label or report. |
| Reviews | Sign-in required; no way to review without it. One moderation switch (approval on or off), shop-wide and per product. Staff can approve or delete only: no reason field, no notice to the reviewer, no public reply. No keyword filter and no timed publishing. Shown with average, star breakdown, masked name, date and photos; no sort option. Reviews attach to the product, so pack sizes share them. Reward can be zero. reCAPTCHA and an export exist. |
| Sold count | One on/off switch in the theme's site settings. No rounding, no minimum, no manual starting number. |
| Design | The no-code editor covers browsing pages only: home, product, collection, category, blog, contact, search, 404. The code editor lists template files for every page type, including product, cart (which is also the checkout) and the sign-in and member pages. Opening it requires agreeing that CYBERBIZ gives no free code help and that edited templates stop receiving theme updates. |
| Content | Custom pages and blog built from blocks, including free HTML. Product descriptions are rich text with an HTML mode; no structured product fields. Preset fonts and colour settings; no font upload outside code. Own domain with automatic SSL; site password for a private launch. |
| Staff | One account per person with fine-grained permissions, including separate customer-export and order-export rights. Two-step sign-in with an authenticator app, set per account. |
| Tracking | LINE Tag, Google Analytics 4, Google Tag Manager, Facebook Pixel. |

## Corrections to earlier statements

| Earlier statement | What the live shop shows |
|---|---|
| "Customers can check out without creating an account" (listed as the same on both) | Checkout registers the buyer as a member. Now a difference in the comparison. |
| "Codes are fixed at 8 characters" | Wrong for ordinary codes, which run 4 to 12 characters here. The 8-character rule applies only to member coupons imported by spreadsheet. Removed. |

## Still open

### Would change the comparison

| # | Question | Status |
|---|---|---|
| 1 | Can the "register as a member" box at checkout be made optional or removed? | No switch on the screens the account could open. One may exist on the blocked checkout settings screen. The comparison no longer has a guest checkout row. |
| 2 | Can reviews be limited to buyers? | No such option seen. The review list has a "verified" filter that nothing explains. |
| 3 | Does the AUTOMATION app offer an "order completed" or "N days after purchase" trigger? | Not opened; it needs a separate sign-in. The comparison states there is no built-in review invitation or repurchase reminder. |
| 4 | How far can template code change the cart and checkout page? | The template files are listed, but their contents were not read, and the help centre warns that drag-and-drop themes honour only a few code changes. |
| 5 | Is every staff action logged? | No log found in the menus by a Co-manager account. The help centre describes per-order history and login and export records. |
| 6 | Can two-step sign-in be required for all staff? | The Co-manager account saw no such option. The help centre says the owner can require it. |
| 7 | Is showing review stars available below 企業版? | Shown on this 企業版 shop. Unknown for other plans. |
| 8 | How is the sold count calculated, and does it include orders from LINE? | Not stated on screen. |

### Needed before setup

| # | Question | Status |
|---|---|---|
| 9 | Can the marketing consent box start unticked? | Ticked by default; no setting found on reachable screens. |
| 10 | Which amount are points earned on: before or after promo code and points? | The screen does not say. |
| 11 | How do points behave on a return? | No setting on the points screen; an email template for returned points exists. |
| 12 | What do an empty and a zero value mean in the per-variant points limit? | Not stated on screen. The help centre says zero means points cannot be used on that item. |
| 13 | Is the free-shipping threshold judged after points, and can that be changed? | Help centre states it as a general rule. No setting seen. |
| 14 | Unpaid-order cancellation and order auto-close: ranges and what they release. | On blocked screens. |
| 15 | Pay-at-pickup: order-value cap and automatic blocking after uncollected parcels. | On blocked screens. |
| 16 | Can the shop use its own 綠界 or 藍新 account? Invoice provider and issuing-time options. | On blocked screens. |
| 17 | Refund methods for ATM and pay-at-pickup orders; one refund per order. | Needs an order screen. |
| 18 | Does a cancelled order return the promo code use? | Not found. |
| 19 | Zero-amount orders: payment status and invoice. | Not found. |
| 20 | Can members delete their account? | Customer screens blocked. The help centre says accounts can only be disabled. |
| 21 | Can a guest look up an order without signing in? | No lookup link on the storefront; order details go by email. |
| 22 | Staff chat in LINE alongside automated messages; what Chat Box provides. | Chat Box is set up on this shop; not opened. |

### Pricing

| # | Question | Status |
|---|---|---|
| 23 | What does this shop pay as a cut of sales? | Not visible to this account. The public price table lists 5% for 企業版. |
| 24 | Does 專業版+ carry a cut of sales? | Missing from the public table. |
| 25 | Which listed plan prices are current? | Pages on their site disagree. |

## Tests that need a trial shop

These need a real order, registration or message, so they were not run on the live shop.

- Spend points on an order near the free-shipping threshold: is shipping then charged?
- Pay an order entirely with points: payment status and invoice.
- Refund part of an order: points, promo code and invoice.
- Let an ATM order go unpaid: are stock, code and points released?
- Submit a review: who can, when, and what the customer sees after rejection.
- Which sold count is shown after a refund.

## Related files

- Exploration brief used for the admin review: `docs/cyberbiz-exploration-plan.md`.
- Items needed from the company: `docs/company-checklist-zh.pdf` and `docs/company-checklist-en.pdf`.
- Open design questions and legal points: `docs/payment-system-design.md`, sections 9 and 11.
