# Implementation plan

Companion to `docs/payment-system-design.md`, which holds the decisions and the reasons. This document says how the system gets built: technology, structure, the order of work, and how each piece is proven to work.

Status: plan only, 2026-11-25. Nothing here is built yet.

## 0. Confirmed decisions and how the build is run

### Confirmed on 2026-11-25

| Decision | Affects |
|---|---|
| Technology and hosting as in section 1 | Everything |
| One-time codes are sent by email only, not SMS | Phase 3; adding SMS later is one provider and one setting |
| Staff can edit text and images on brand pages, not rearrange layout | Phase 1 Content screen |
| A review flagged for health-claim language does not auto-publish; it waits for staff | Phase 5 |

### Who does the building

The build is carried out by AI coding sessions, organised as follows:

- **Orchestrator: Fable 5.2 at high effort.** It owns the plan, breaks each phase into tasks, writes the briefs, reviews every piece of returned work, runs the tests, and is the only one that merges and reports to you.
- **Workers: Opus 5.5 at high effort**, started by the orchestrator as subagents, one per self-contained task (a provider module, a set of admin screens, a rule and its tests).

Working rules:

- The orchestrator's own model and effort level are whatever the session is started with, so each build session should be started on Fable 5.2 at high effort. Workers are started by the orchestrator with the model and effort set explicitly.
- A worker gets a written brief: the goal, the files it owns, the interfaces it must match, the tests that must pass, and what it must not touch. Workers on parallel tasks never share files.
- The pure business rules in section 3 and the database schema are written or reviewed by the orchestrator itself before workers build on them, because every other part depends on them.
- No worker's output is accepted on its own report. The orchestrator reads the code and runs the checks and tests before merging.

### Build first, ask at the end

**No outside account is needed to build.** The work runs against a local database and stand-in versions of every provider, so development proceeds through Phases 1 to 5 without waiting on the company. Everything that needs an account, a key, a number or a sign-off is collected into one list (section 7) and requested once, when the build is otherwise complete.

What this costs, stated plainly:

- **You cannot click through a live test site until the hosting account exists.** Until then, each phase is reported with screenshots and test results from the build environment.
- **Provider connections are unproven until real test accounts exist.** They are written from the providers' documentation and tested against stand-ins that imitate them. First contact with the real services happens in Phase 6 and will find differences, so that phase includes time to fix them.
- Placeholders are used for anything unknown (shipping fee, company details, final product data) and listed so none are forgotten.

Where a provider offers a public test environment that needs no account, the orchestrator may ask you to approve network access to it. That is a one-click approval in the session, not something you have to set up.

Provider details (ECPay, LINE Pay, LINE) are written from general knowledge of their interfaces. **Each phase starts by reading the provider's current documentation and correcting this plan where it differs.**

Effort figures are rough engineer-weeks for one experienced human developer, given so the phases can be compared. They are not a schedule.

---

## 1. Technology

| Layer | Choice | Reason |
|---|---|---|
| Language | TypeScript throughout | One language for storefront, admin and server logic |
| Application | Next.js (App Router), one deployable application | Storefront pages render on the server for speed and search; admin and server endpoints live in the same codebase |
| Database | PostgreSQL (managed: Google Cloud SQL) | Transactions and constraints are what keep money and points correct |
| Database access | Prisma, with versioned migrations checked into the repository | Every schema change is reviewable and repeatable |
| Styling | Tailwind CSS with the brand's colours, type and spacing defined once as tokens | Reproduces the current design and keeps new screens consistent |
| Input validation | Zod schemas shared by forms and server | One definition of what valid input is |
| Customer sign-in | Auth.js with LINE, Google and email + password; sessions stored in the database | Standard, maintained library for exactly these providers |
| Staff sign-in | Separate login with password plus two-step code (TOTP), separate session cookie | Staff and customer access never mix |
| Background work | A `jobs` table, run by a worker endpoint that Google Cloud Scheduler calls every minute | No extra infrastructure; jobs survive restarts and are visible to staff |
| Hosting | Google Cloud Run, Taiwan region (asia-east1), with a fixed outbound address | Managed, scales with traffic, data stays in Taiwan |
| Files | Google Cloud Storage for product and content images | |
| Secrets | Google Secret Manager | Provider keys never sit in the code |
| Email | A transactional email service (to be chosen in Phase 2) with a verified sending domain | Order mail must not land in spam |
| Monitoring | Error tracking with alerts, uptime check, structured logs | The technical owner hears about failures before customers report them |
| Tests | Vitest for logic, Playwright for full customer and staff journeys | |
| Delivery | GitHub Actions: check, test, build, deploy | |

## 2. Repository and environments

### Layout

```
/legacy-site/          the current static site, moved here unchanged (still deployed to GitHub Pages until launch)
/web/                  the new application
  app/(store)/         storefront pages: home, team, classroom, shop, product, cart, checkout, order, account
  app/admin/           admin screens
  app/api/             server endpoints: provider callbacks, job worker, health
  lib/domain/          pure business rules: pricing, points, promo, refunds, review routing (no database, fully unit-tested)
  lib/services/        operations that change data: place order, mark paid, refund, award points
  lib/providers/       one folder each: ecpay-payment, ecpay-invoice, ecpay-logistics, linepay, line-login, line-messaging, email
  lib/jobs/            job handlers
  prisma/              schema and migrations
  tests/               unit, integration, end-to-end
/docs/                 design and plan
```

Two rules keep this maintainable:

- **Business rules live in `lib/domain` as plain functions.** "What does this cart cost", "how many points does this refund cancel" take data in and return data out, so they can be tested exhaustively without a database or a provider.
- **Each provider sits behind an interface with a fake implementation.** Tests and local development run against fakes; staging runs against the providers' test environments; only production uses real keys.

The GitHub Pages workflow is changed in Phase 1 to publish only `legacy-site/`, so the old site stays live and the new source is not published by accident.

### Environments

| Environment | Purpose | Providers | Data |
|---|---|---|---|
| Local | Development | Fakes | Seeded sample data |
| Staging (from Phase 6) | Review and staff testing, at a temporary address | Provider test environments | Test data, reset freely |
| Production | The real shop | Live merchant accounts | Real, backed up daily, restore tested before launch |

Every change goes: branch, automatic checks and tests, review, merge. Once staging exists in Phase 6, merges deploy to it automatically. Production deploys are a deliberate manual step.

---

## 3. Core rules, specified

These are the calculations the whole system depends on. They are written and tested first within their phase.

### Pricing (Phase 2, extended in Phase 4)

Input: cart lines, optional promo code, points to spend, delivery method, settings. Output: every amount on the order. All amounts are whole NT$.

1. `items_total` = sum of unit price × quantity.
2. `promo_discount` from the code (percentage rounds down; fixed amount is capped at `items_total`). A free-shipping code sets a flag instead.
3. Free shipping if the flag is set or `items_total − promo_discount` ≥ the threshold.
4. `points_used` = requested points, capped at the member's balance and at `items_total − promo_discount`.
5. `shipping_fee` = 0 if free, otherwise the flat fee.
6. `amount_due` = `items_total − promo_discount − points_used + shipping_fee`.
7. `promo_discount` and `points_used` are split across order lines in proportion to line value, using largest-remainder rounding so the shares add up exactly. These shares are stored on each line.

### Points (Phase 4)

- Earned at completion: `floor(paid_for_products / 100)`, where `paid_for_products = items_total − promo_discount − points_used`, less anything refunded before completion.
- Refund of specific lines after completion:
  - Points returned = the `points_share` of the refunded quantity.
  - Points cancelled = original earn − `floor(remaining paid_for_products / 100)`, limited to the current balance so it never goes negative.
- Every change is a new ledger row. The database refuses a second earn row for the same order.

### Refund amount (Phase 2, extended in Phase 4)

Refund for a line = (unit price × quantity refunded) − its promo share − its points share. Shipping is refunded only on a full refund before shipment, or at staff discretion with a reason.

An order can be returned or refunded once. That one refund can cover all items or some of them; the database refuses a second refund for the same order.

### Review routing (Phase 5)

On submission:

1. Stars are recorded and count in the average immediately.
2. Text is checked against the health-claim word list and the spam rules (links, phone numbers, profanity list, duplicate text).
3. 4–5 stars and no flags: published now.
4. Otherwise: queued, with `auto_publish_at` = submission + 3 working days (Monday to Friday, skipping dates on an admin-editable holiday list).
5. A job publishes queued reviews when that time passes, unless staff rejected them with a listed reason.

A review flagged for a health claim is the one exception to auto-publish: publishing it is the legal risk, so it stays queued and shows as overdue on the Home screen until staff act. *This exception is new in this plan and needs your agreement.*

### Order states

| Payment status | Moves to |
|---|---|
| pending | paid, expired, cancelled |
| paid | partially refunded, refunded |
| partially refunded | refunded |

| Shipping status | Moves to |
|---|---|
| not ready (awaiting payment) | preparing, cancelled |
| preparing | shipped, cancelled |
| shipped | arrived at store, delivered, returned |
| arrived at store | picked up, returned |
| picked up / delivered | completed (after 7 days) |

Pay-at-pickup orders start at "preparing" with payment "pending", and become "paid" at pickup. Any move not listed is rejected by the code.

---

## 4. Phases

### Phase 0: nothing blocks the start

There is no setup step for the company before building begins. Phases 1 to 5 are built and tested in the build environment; hosting, live accounts and sign-offs are all gathered in Phase 6.

One optional head start: the ECPay and LINE Pay merchant applications take the longest to be approved. Starting them early shortens Phase 6, but the build does not depend on it.

### Phase 1: storefront and catalog (about 2–3 weeks)

**Build**
1. Move the current site to `legacy-site/`; narrow the Pages workflow.
2. Application skeleton in `web/`; local database; checks and tests in GitHub Actions.
3. Deployment configuration for Cloud Run, Cloud SQL, secrets and storage written and kept ready; it is first run in Phase 6, when the hosting account exists. Images are stored on local disk behind the same interface until then.
4. Database tables: products, variants, settings, content_pages, staff, audit_log.
5. Design tokens and shared components taken from `frontend-v1-reference`: navigation, footer, buttons, cards, accordion, toast.
6. Storefront pages at real addresses: `/`, `/team`, `/classroom`, `/classroom/[slug]`, `/shop`, `/shop/[category]`, `/products/[slug]`.
7. Cart: stored in the browser for guests and in the database for members; cart page; quantity limits; stock check.
8. Staff login with two-step code; roles; audit log.
9. Admin screens: Products (with image upload), Content, Settings (first fields), Staff.
10. Seed the three current products.

**Done when**
- Every current page exists at its own address and matches the reference design on phone and desktop, checked side by side.
- A staff member changes a price and a product photo in the admin and sees it on the site, with no developer involved.
- A product link pasted into LINE opens that product.
- Page speed on mobile is good (measured, target agreed at the time).

### Phase 2: guest checkout and money (about 5–7 weeks, the largest phase)

**Build**
1. Tables: customers, orders, order_items, payments, payment_events, shipments, invoices, invoice_allowances, refunds, refund_items, jobs, messages.
2. Pricing rules (section 3) with unit tests.
3. One-page checkout: contact details, delivery method, invoice choice, payment method, consent text; validation of phone, 手機條碼, 統編 and 愛心碼 formats.
4. Place-order service: one transaction that creates the order, reserves stock, and is safe against double submission.
5. **ECPay payments:**
   - Send the customer to ECPay's payment page for card and Apple Pay / Google Pay.
   - ATM: receive the virtual account number and show it on the order page and in email, with the 3-day deadline.
   - Receive the server-to-server result, verify its signature, store it in `payment_events`, mark the order paid exactly once.
   - Query a single order's status (used by the reconciliation job).
   - Refund a card payment in full or part.
6. **ECPay logistics:**
   - 超商 map for 7-11 and FamilyMart; store the chosen shop.
   - Create shipments for 超商 pickup, pay-at-pickup and home delivery; fetch printable labels.
   - Receive status updates and map them to shipping statuses.
7. **ECPay e-invoice:**
   - Issue with each carrier type, donation, or 統編.
   - Void and allowance.
   - Timing: at payment for prepaid, at shipment for pay-at-pickup.
8. Jobs and worker: retries with increasing delay; job kinds for invoice, shipment, email, reconciliation (every 15 minutes), ATM expiry, order completion (7 days after pickup), nightly settlement comparison.
9. Pay-at-pickup controls: order-value cap; count uncollected parcels per phone; block after two.
10. Order page for customers, reached by a private link in the confirmation email: status, ATM details, invoice number.
11. Email: order confirmed, ATM payment instructions, paid, shipped, arrived, refunded.
12. Admin screens: Home, Orders (with refund), Shipping (batch labels), Problems.

**Done when**, in the build environment against provider stand-ins (repeated against real test accounts in Phase 6)
- A guest completes an order with card, with ATM, and with pay-at-pickup, for both 超商 chains and home delivery.
- Each produces a correct invoice of each carrier type; a full refund voids it and a partial refund issues an allowance.
- Failure drills pass:
  - the payment result is deliberately blocked and the reconciliation job still marks the order paid;
  - the same result is delivered three times and the order is paid once;
  - the invoice service is made to fail, and the job retries, appears on Problems, and succeeds when retried by a staff member;
  - an unpaid ATM order expires and its stock returns.
- Two simultaneous checkouts for the last box in stock result in exactly one order.
- Staff ship, refund and resolve a problem using only the admin.

This phase is a usable shop for guests. A limited launch after it is possible if the company wants revenue sooner.

### Phase 3: members and LINE (about 3–4 weeks)

**Build**
1. Tables: login_identities, one_time_codes, sessions; consent fields.
2. Sign-in with LINE (requesting the Official Account be added as a friend, stated on screen), Google, and email + password with reset.
3. Account creation at the thank-you page with details pre-filled.
4. Claiming guest orders: one-time code to the order's email, then orders attach to the member. Merge rule: verified match only.
5. Member area: orders, order detail, addresses and favourite pickup store, login methods, marketing consent, delete account.
6. Checkout for members: pre-filled; "用 LINE 快速結帳" shortcut.
7. **LINE Pay:** request, customer approval, confirm, refund, status check for the reconciliation job. Fixed outbound address registered with LINE Pay if required.
8. **LINE Official Account:**
   - Rich menu (商城 / 我的訂單 / 會員點數 / 客服) created through the Messaging interface, opening the shop inside LINE already signed in.
   - Message templates for order confirmed, shipped, arrived at store, refunded, review invitation; sent to bound members, email otherwise.
   - Auto-reply and greeting text.
9. Channel tagging: a visit that begins inside LINE marks the cart, and the order, as LINE.
10. Admin screens: Customers (profile, history, merge, pay-at-pickup block, delete), message templates in Settings, role permissions and action log in Staff.

**Done when**
- A new customer goes from the rich menu to a paid LINE Pay order without typing a password.
- A guest with two past orders signs up and sees both after verifying, and a different person entering the same email address without verifying sees nothing.
- One member signs in by LINE on a phone and Google on a laptop and sees the same account.
- A bound member receives each order message in LINE; an unbound one receives email.
- Deleting a member removes personal details and keeps the order and invoice records.

### Phase 4: points, store credit, promo codes (about 3 weeks)

**Build**
1. Tables: points_ledger, store_credit_ledger, promo_codes, promo_redemptions.
2. Points and promo rules (section 3) with unit tests covering every combination: code only, points only, both, order fully paid by points, each with full and partial refunds.
3. Checkout: promo code field (members only, with a sign-in prompt for guests); points slider; store-credit use; live totals.
4. Zero-amount orders: marked paid without a provider or invoice.
5. Completion job awards points. Cancel, expiry and refund services write the matching ledger rows and release code uses.
6. Refund screen gains: store credit as a refund method for members; points effects shown before staff confirm.
7. Member area: points balance and history; store credit balance and history; request cash-out of store credit.
8. Admin screens: Promo codes; points adjustment with required reason; store-credit cash-out handling; points rate in Settings.
9. Reports: points outstanding (for the accountant), promo code usage by campaign and channel.

**Done when**
- The unit tests for the rule combinations pass, and the same cases walked through in the running application match to the dollar and the point.
- Two browsers spending the same member's points at the same moment cannot overspend.
- A code at its usage limit is refused; a cancelled order gives its use back.
- A guest cannot apply a code; a member can combine one code with points.

### Phase 5: reviews and counts (about 2 weeks)

**Build**
1. Tables: reviews, review_replies, review_tokens, product_stats, holidays.
2. Review invitation at completion: LINE or email for members; email with a private link for guests.
3. Review form: stars, text, a notice of what is not allowed (health claims) and that the name is masked.
4. Routing rules (section 3), word list and spam rules.
5. Product page: average, star breakdown, verified badge, masked name, pack bought, brand replies, newest first, roll-up across pack sizes of one formula. Copy changed to "實際購買的顧客".
6. Nightly job: units sold (paid orders, not reduced by refunds) and star statistics into product_stats. Display rounded down with "+", hidden under the minimum.
7. Admin screens: Reviews queue with time remaining, approve, reject with listed reason (reviewer notified, can resubmit), reply; word list, rounding step, minimum and holiday list in Settings.

**Done when**
- A 5-star clean review appears at once; a 5-star review containing a listed health word is queued; a 2-star review is queued and appears by itself after 3 working days.
- The average changes the moment any review is submitted.
- A guest reviews through the emailed link, and the link cannot be reused or used for another order.
- The sold count matches a hand count from the Orders screen, rounded as configured.

### Phase 6: connect, verify, launch (about 3–5 weeks, plus account approval time)

**First: the one request to the company.** Everything in section 7 is asked for here, in a single list.

**Then: connect the real services.**
- Deploy to staging on the company's hosting account.
- Replace each provider stand-in with the real test account, rerun every "done when" list from Phases 2 to 5 on staging, and fix what differs.
- Replace placeholders with the company's real values.

**Build and do**
1. Reports and exports: sales by day, product and channel; reconciliation; customer, order, points and review exports (logged, restricted by role).
2. Policy pages drafted in Chinese for the lawyer; registration details in the footer.
3. Tracking: Google Analytics and LINE Tag, disclosed in the privacy policy.
4. Production environment: live keys, backups with a tested restore, alerts, uptime check.
5. Hardening: rate limits, security headers, dependency audit, load test at several times expected peak.
6. Independent security review; fix findings.
7. Staff handbook in Chinese: daily routine, shipping, refunds, review moderation, what each Problems entry means, when to call the technical owner.
8. Soft launch on a temporary address: real orders with every payment and delivery method, a real refund, real invoices.
9. Domain switch; old site turned off; `legacy-site/` kept in the repository.

**Done when**
- Lawyer and accountant have signed off the items in the design document's legal section.
- Security review findings rated high are fixed.
- Staff complete a full day of real test orders with no developer help.
- A database restore has been performed successfully.

### Total

About 18–24 engineer-weeks across the six build phases, plus waiting time for merchant accounts and reviews by others.

---

## 5. Testing

| Kind | What it covers | When it runs |
|---|---|---|
| Unit | Every rule in section 3, with tables of cases including edge amounts | Every change |
| Integration | Services against a real test database with fake providers: place order, mark paid, refund, expire, award | Every change |
| Provider contract | Our provider code against the providers' test environments: signatures, field formats, error cases | Nightly and before each release |
| End to end | Full customer and staff journeys in a browser, on phone and desktop sizes | Before each release |
| Failure drills | Lost, duplicated and delayed provider messages; provider outage; simultaneous purchases | End of Phases 2 and 4, and before launch |
| Phase report | Screenshots and test results for each "done when" list, from the build environment | End of Phases 1 to 5 |
| Manual acceptance | The "done when" lists, by you and company staff on staging | Phase 6 |

## 6. Risks

| Risk | Mitigation |
|---|---|
| Provider interfaces differ from this plan | Documentation read at the start of each phase; providers isolated behind interfaces |
| Merchant account approval is slow | Applications start in Phase 0; building uses test environments |
| LINE Pay or ECPay needs a registered server address | Fixed outbound address set up in Phase 1 |
| No technical owner after launch | Named before Phase 6; handbook and Problems screen reduce, but do not remove, the need |
| Money or points error in production | Rules isolated and exhaustively tested; append-only ledgers; nightly reconciliation; every state change logged |
| Personal data exposure | Minimal collection, role-limited admin, two-step login, logged exports, security review |
| Scope growth mid-build | Deferred items stay in the design document's "Later" list until launch |
| Legal review changes review moderation or the sold count | Both are settings and rules in one place; the fallback for reviews is already designed |

## 7. The one request to the company, made at the start of Phase 6

Nothing below is needed before then. Keys and passwords are entered through the session's secure credential form or the hosting account's secret store, never pasted into chat or committed to the repository.

**Accounts and access**
- Google Cloud account with billing, in the company's name, with developer access granted.
- ECPay merchant account covering payments, e-invoice and logistics, with its test and live keys.
- LINE Pay merchant account, with sandbox and live keys.
- LINE Official Account (verified), and a LINE Developers provider owned by the company with a Login channel and a Messaging channel.
- Google sign-in credentials for the site.
- The domain, and access to its DNS settings.
- An email-sending service account, and a sending address on the company domain.

**Real values to replace placeholders**
- Final product names, prices, photos, ingredient lists, stock.
- Shipping fee, free-shipping threshold, pay-at-pickup order-value cap.
- Warehouse sender details for shipping labels.
- Company legal name, 統編, address and contact; 食品業者登錄 number.
- Wording approval for LINE messages, emails and the rich menu image.

**People and sign-offs**
- Named technical owner after launch, and who holds the hosting account.
- Staff list with roles; who answers LINE support; who moderates reviews.
- Lawyer: policy pages, review moderation, the health-claim word list, the sold count, returns of opened boxes.
- Accountant: invoice timing and the points liability.
- Staff time for the soft launch.
