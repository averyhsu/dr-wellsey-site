# CYBERBIZ: points not confirmed

Private working list. These are things CYBERBIZ's published documentation does not state, or states unclearly, as read on 2026-11-25. None of them appear in the comparison PDFs, which contain confirmed points only.

"Not found" means the help centre says nothing about it. It does not mean the feature is missing. Each item is a question for CYBERBIZ sales or something to test in the 30-day trial.

Numbers are kept from the first version of this list. Points 2, 3, 6 and 8 were moved into the comparison on 2026-11-25 at the owner's decision (see the last section).

## Would change the comparison if answered

| # | Question | Why it matters | What the docs do say |
|---|---|---|---|
| 1 | Is there a store credit (購物金) balance separate from points, usable for refunds? | We planned store credit as a refund method for ATM and pay-at-pickup orders. | Not found. Points are the only balance described. |
| 4 | Can the brand reply publicly to a review? | Our answer to negative reviews is a public reply. | Not found. Staff can approve, delete or edit. |
| 5 | Is showing review stars on the storefront available on PLUS plans, or only 企業版? | Decides whether reviews are usable at all on the plan we would pick. | Two of their own pages disagree. |
| 7 | Is there a message inviting a review after an order completes? | Our review flow starts from that message. | Not found. |
| 9 | Does a guest who registers later keep their earlier orders? | Our design lets them claim past orders after verifying. | Not found for the website. Accounts with the same email merge. |
| 10 | Is every staff action logged, or only some? | Needed for 個資法 accountability. | Order changes, logins and exports are recorded. |

## Would not change the comparison, but needed before setup

| # | Question | What the docs do say |
|---|---|---|
| 11 | Do buyers who check out without registering earn points? | Not found. |
| 12 | Can buyers who check out without registering use a promo code? | "Any member" with the code can use it. |
| 13 | Is the free-shipping threshold really judged after points, and can that be changed? | Stated as a general rule: each step uses what is left after the previous one, and shipping comes after points. Not stated for free shipping specifically. |
| 14 | Can 超商取貨付款 be capped by order value? | Amount limits per payment method exist on some plans; not stated for store pickup specifically. |
| 15 | What happens to payment and invoice when an order is paid entirely with points? | Not found. |
| 16 | Are points spent on an order excluded before new points are calculated? | Earning is on the checkout amount excluding shipping. |
| 17 | Is a promo code's total-use count restored when an order is cancelled? | Coupons are returned on cancellation; the count is not mentioned. |
| 18 | With 綠界 as the invoice provider, are 手機條碼, donation and 統編 offered at checkout? | Listed only for invoices CYBERBIZ issues itself (企業版). |
| 19 | How is the marketing-consent tick box worded, and is it unticked by default? | Members can set their own subscription flags. |
| 20 | Is checkout a single page? | Not found. |
| 21 | How far are theme code edits honoured on drag-and-drop themes? Can login, member centre and order pages be edited? | "Only a few" edits supported; some may not take effect. |
| 22 | Can member points be written through their developer interface? | Reading is possible. Writing not found; the full reference could not be opened. |
| 23 | What is the payout cycle? | Not found. |
| 24 | Are LINE notification messages by phone number (通知型訊息) supported? | Not found. |

## Pricing points not confirmed

| # | Question | What the docs do say |
|---|---|---|
| 25 | Does 專業版+ carry a cut of sales, and how much? | It is missing from the plan comparison table. |
| 26 | Which listed plan prices are current? | Different pages on their site show different figures. |
| 27 | SMS price per message; price of the CHAT BOX inbox; price of extra feature modules beyond the two included. | Not found. |

## Moved into the comparison, still not confirmed

These now appear in the comparison PDFs as plain statements. The documentation supports them only as far as shown here, so they remain worth confirming.

| # | Stated in the comparison | What the docs do say |
|---|---|---|
| 2 | Reviewers must be signed-in members. | Stated directly. Whether reviewing is limited to buyers of the product is not found. |
| 3 | No moderation rules: approve every review by hand, or publish all. | One switch is documented. No rule by star rating, keyword or timer is described. |
| 6 | Shows the running total of units sold, as is. | A switch shows cumulative units sold. Rounding, a minimum, and how returns are counted are not described. |
| 8 | No built-in repurchase reminder. | None is described. Their automation tool can send scheduled messages to saved member groups, which might be set up to do a similar job. |

## Related lists

- Items needed from the company: `docs/company-checklist-zh.pdf` and `docs/company-checklist-en.pdf`.
- Open design questions and legal points: `docs/payment-system-design.md`, sections 9 and 11.
