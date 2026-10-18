# Taxx Savage System Blueprint

**Status:** living document. Update the Change Log and the "Open Items" list every session.
**Last updated:** 2026-10-18 (v0.6)
**Build target:** HighLevel AI Studio (with Cloud: database, auth, file storage). This document is the spec; the app is built in AI Studio. The built app is in the private repo `angeliepesquira-spec/taxx-savage-2` (see section 16, As Built).

---

## 1. Overview
Taxx Savage is a white-label tax software company. The system has a public website and three portals:
1. **Public website** (separate app, not the hub): marketing site and packages page. Each package's **Start Now** button links to the hub's signup page. Built later, outside this AI Studio project.
   **Hub app** (this build): signup, payment, contract, document upload, and the three portals.
2. **Client portal** (the "preparer" who signed up).
3. **Admin portal**.
4. **Review team portal**.

All pages must be **mobile optimized** (most clients sign up on phones).

## 2. Roles
| Role | Can do |
|---|---|
| Client (preparer) | Complete onboarding, pay for and submit returns, view own billing and data |
| Admin | Everything: users, products, contracts, menus, PTINs, notifications, reports, review team |
| Reviewer | See the paid returns list, search and filter, export; no user or billing management |

## 3. Packages
| Tier | Price | Notes |
|---|---|---|
| Collab | $89 | Product shown on client portal as "Taxx Collaboration" |
| Pro | $599 | Confirm price (earlier stated as $5.99, then $599) |
| Third tier | TBD | **Open item** |

Reference: https://taxxsavage.com/tax-software-packages/ (3 tiers, "Start Now" button opens the signup form).

## 4. Client onboarding flow
1. Public packages page (separate app) > **Start Now** link > hub signup page. The link carries the chosen package (for example `?package=collab`) so the hub preselects it.
2. **Stripe payment** (card is saved on file for later per-return charges).
3. **Contract signing** (embedded template, chosen by package; mobile friendly). Fields: company name, print name, authorized signature, date.
4. **Document upload**: ID (license/passport/govt ID), W9, Social Security card, PTIN (IRS document), EFIN summary (IRS document), anticipated bank product returns, EIN (optional).
5. **Thank-you page** with next steps, prompting account creation (login and password).
6. **Experience questionnaire**.
7. **Review** by Taxx Savage (1-3 business days).
8. **Portal access granted** (portal is locked until approval).

Rules:
- The signup form does **not** ask for a PTIN. The client uploads the PTIN document; the **admin verifies it and types the PTIN into the client record**. Client cannot edit it. Log who entered it and when.
- Recommended: block approval until a PTIN is entered (confirm).

## 5. Client portal
Header: logo, user email, package label ("Your Package: ...").

Menus are managed by the admin (see Menu Manager). Starting layout, copied from the existing hub:

- **Main:** Home, Profile & Billing, Fee Schedule, Log Out
- **Tools:** Tax Software, Request Total Audit Protection, Submit Amended Return, Submit Corporate Return, Submit State Return, Submit Non-Bank Product Return, File Extensions, Request Refund Cards, Request Check Reset, Messaging Center
- **Reports:** Payouts Dashboard, Advances, Prep Fees, Client Disbursement List, Mailing Fees, Rework Fees (phase 2)
- **Premium Tools:** Taxx Academy (courses)
- **Resources:** Mastermind Community (and more, TBD)

Home page: welcome with name, "Your Profile Has Been Approved" status, season renewal banner (Renew button), announcements panel (title, post date, text, optional button), walkthrough video.

## 6. Return request forms (one per return type)
Types: Amended Return, Corporate Return, State Return, Non-Bank Product Return, Audit Protection, Refund Cards, Check Reset, File Extensions.

Flow for each:
1. Page shows price per return and the card on file (brand, last 4, expiry).
2. Client picks **applicable tax year** (radio buttons, e.g. 2022-2025).
3. Client enters **number of returns**.
4. Page shows **Your Cost Today** = count x product price.
5. **Confirm Count** charges the card on file via Stripe.
6. **Decided (as built):** the client enters details **for each return** first (taxpayer first initial, last name, SSN last 4 only), then pays. A return becomes visible to admin and reviewers only after payment succeeds.
7. **If payment fails or is cancelled:** nothing is recorded in the admin or review portals.
8. Refunds are done by the admin inside Stripe; a refunded return is flagged.

Pricing is fixed per return type (no tier discounts). Each return type is a **Product** (name, price, Stripe price ID, active flag), created in Stripe or the CRM and pulled in.

Open: exact fields for each form other than Amended. Draft default fields: taxpayer name, SSN, tax year, file upload, notes.

## 7. Paid returns record (feeds admin and review portals)
One row per paid return, created automatically when payment succeeds. Replaces the review Google Sheet.

| Field | Source |
|---|---|
| Date/time | Payment success timestamp |
| Company name | Client profile |
| Preparer first name, last name | Client profile |
| Email | Client email |
| Review name | Taxpayer first initial + last name (e.g. "T FREEMAN") |
| SSN last 4 | Last 4 only shown by default; full SSN encrypted |
| Tax year | Form |
| Type | Product |
| Stripe confirmation | Stripe charge ID (ch_...); shared by returns paid together |

Bulk purchases: each return is its own row; rows from one purchase share the Stripe charge ID.
Requests with no taxpayer (Check Reset, Refund Cards, etc.): separate requests list (default; confirm).

## 8. Admin portal
Starter menu (editable): Dashboard, User Management, Review Requests, Reviewer Activity, Review Team, Registered PTINs, Preparer Completeness, Products & Pricing, Contract Templates, Record Payments, Client Notifications, Client Messaging, Menu Manager, Settings.

Later (from existing admin): Preparer Ledger, Source File Import History, 10X Offer Manager, Preparer Payouts View, Academy Subuser Monitor, Academy Course Manager, Preparer Request Manager.

Key features:
- **User Management:** full data on every client who signed up (profile, package, contract, documents, status, PTIN).
- **Registered PTINs:** list; admin enters PTIN manually.
- **Preparer Completeness:** which onboarding items are done or missing.
- **Review Requests / Reviewer Activity:** counts of returns reviewed per reviewer, filter by date and status.
- **Review Team:** admin adds and removes reviewer users. Admin returns count as reviewed work too.
- **Contract Templates:** paste or embed 2-3 templates; each tied to a package; signing step uses the matching one.
- **Menu Manager:** admin creates, orders, shows and hides menu links for all three portals (label, section, icon, internal or external URL, role and package restrictions). Menus are a work in progress, so nothing is hardcoded.
- **Client Notifications:** posts shown on the client Home page.

## 9. Review team portal (version 1)
Paid returns list only (the columns in section 7), with search and filters (date, tax year, type, company), CSV export. Reviewers have their own logins.

Phase 2 (from the existing review sheet "Reviews Team A"): Reviewed?, Reviewer initials, Transmitted?, O/S Issue (None / Concerns / Missing Doc / Possible Fraud), Notes with dates, Audit Protection, Household Employee Income, row colors by issue.

## 10. Data model (HighLevel Cloud tables)
- `users` (auth) with `role`: client, admin, reviewer
- `clients`: user_id, company_name, first_name, last_name, email, phone, package_id, status (signed_up, contract_signed, docs_submitted, approved), ptin (admin only), stripe_customer_id, approved_at
- `packages`: name, price, stripe_price_id, contract_template_id, active
- `contract_templates`: name, body/embed, package_id
- `contracts_signed`: client_id, template_id, signature, signed_at, pdf_file
- `documents`: client_id, type, file, uploaded_at
- `products`: name, price, stripe_price_id, active, form_type
- `orders`: client_id, product_id, tax_year, quantity, amount, stripe_charge_id, status (paid, failed, refunded)
- `returns`: order_id, client_id, product_id, tax_year, taxpayer_first_initial, taxpayer_last_name, ssn_encrypted, ssn_last4, files, created_at, status
- `requests`: non-taxpayer requests (check reset, refund cards, etc.)
- `announcements`: title, body, button_label, button_url, posted_at
- `menu_items`: portal, section, label, icon, url, order, visible, roles
- `messages`: client_id, direction, body, sent_at
- `audit_log`: user_id, action, record, at (SSN reveals, PTIN entries, refunds)

## 11. Integrations
- **Stripe:** package payment at signup; saved card; per-return charges; webhook on success or failure creates `orders` and `returns`; refunds handled in Stripe.
- **CRM (HighLevel):** sends email notifications (account created, payment receipt, approval). Workflows triggered from system events.
- **Email/password reset:** through HighLevel Cloud auth (confirm email capability in AI Studio docs).

## 12. Security
- Full SSN is never collected (last 4 only). ID and onboarding documents live in private storage; access limited to the client and admin (reviewers cannot see onboarding documents).
- Consider IRS Pub 4557 / WISP requirements (confirm with the owner).

## 13. Build order
1. Foundation: roles, login, Menu Manager, database
2. Public website, packages, signup
3. Stripe payment, contract, document upload
4. Admin and review portals (paid returns list first)
5. Client portal and return forms
6. Reports, Academy, ledger and payouts (phase 2)

## 14. Open Items
- [ ] Third package tier: name, price, inclusions
- [ ] Confirm Pro price is $599
- [ ] Full contract template(s) text
- [ ] Fields for each return form except Amended
- [ ] Stripe direct vs HighLevel payments
- [ ] Is AI Studio project created and on TanStack Start? (Cloud requires it)
- [ ] Email support inside AI Studio apps (or CRM workflows only)
- [ ] Rest of admin menu below the screenshot cut-off
- [ ] Reports/payout logic (Ledger, Advances, Prep Fees, etc.)
- [ ] Logo, final brand colors, font (placeholder for now)
- [ ] Domain
- [ ] Test pay-then-record: do failed or cancelled payments stay out of admin and review lists?
- [x] Decided 2026-10-18: keep as built (details first, then pay); unpaid returns must stay hidden from admin and review lists (Prompt 02 Part A)
- [x] Decided 2026-10-18: SSN last 4 only; never collect the full SSN
- [ ] Confirm AI Studio Payments can charge a saved card per return
- [ ] Hub signup URL (for the Start Now links) and the exact link format for each package
- [ ] Public website: separate app, to plan later

## 15. Change Log
- v0.6 (2026-10-18): Prompt 02 implemented on branch `prompt-02-onboarding` of `taxx-savage-2` (section 17), including two security fixes and the payment-verification gap.
- v0.5 (2026-10-18): Decisions: keep details-then-pay; SSN last 4 only. Added `prompts/02-onboarding.md` (paid-only returns, contract templates and signing, document uploads, questionnaire, approval gate, admin user management).
- v0.4 (2026-10-18): Added section 16 (As Built) from review of `taxx-savage-2`. Listed differences from spec and what is still missing.
- v0.3 (2026-10-16): Decision: signup lives on the hub app. Public website and packages page are built separately in another app; Start Now links to the hub signup. Prompt 02 = hub signup page only.
- v0.2 (2026-10-16): Added `prompts/01-foundation.md` (first HighLevel AI Studio build prompt). Prompts live in `prompts/`, one per build step.
- v0.1 (2026-10-16): Initial blueprint from discovery conversation and screenshots (signup, client portal, return form, admin menu, review sheets).

## 16. As Built (repo `angeliepesquira-spec/taxx-savage-2`, reviewed 2026-10-18, code read only, not run)

Stack: TanStack Start, TypeScript, React, Tailwind, Supabase (AI Studio Cloud), AI Studio Payments. 164 files, 18 migrations, last commit 2026-10-16.

**Built**
- Database: user_roles, profiles, packages (Collab 89, Pro 599), products (8 return types, prices not set in DB), clients, orders, returns, announcements, menu_items, audit_log, preparers. Row-level security on tables. Private storage bucket `client-files` with owner/admin policies.
- PTIN: only admins can change it; every change goes to audit_log (trigger `guard_ptin`).
- Roles and portals: `/admin`, `/client`, `/review`, with login, logout, reset-password and a no-access page.
- Menu Manager (`/admin/menus`) and review team management (`/admin/review-team`).
- Admin pages: index, menus, PTINs, returns, review-team, profile, coming-soon.
- Client pages: index, amended-return, corporate-return, state-return, nonbank-return, extensions, add-preparer, checkout, returns, profile, coming-soon.
- Review pages: index, paid-returns, reviewed, profile, coming-soon.
- Public pages inside the app: home, packages, features, contact, signup.
- Reviewer column on returns tables (reviewer name stamped on the return when marked reviewed).

**Differences from this blueprint (decide and reconcile)**
1. Payments use AI Studio's Payments catalog (products matched by name), not direct Stripe. Card-on-file per-return charging is not confirmed.
2. Flow is submit return details first, then checkout. Blueprint says pay first, then enter details. Not verified whether unpaid or failed orders stay out of admin and review lists (must test).
3. SSN: forms collect only SSN last 4 plus taxpayer first initial and last name. `ssn_encrypted` exists but is unused. Decision proposed: keep last 4 only (matches the review sheet; safer).
4. A packages page exists inside the app although the public site is planned as a separate app.

**Not built yet**
- Contract signing and admin Contract Templates page
- Document uploads (ID, W9, SSN card, PTIN, EFIN, bank product, EIN)
- Experience questionnaire
- Admin approval gate before portal access
- Client Notifications (announcements UI), Client Messaging
- Audit Protection, Refund Cards and Check Reset forms
- Reports, Academy, ledger and payouts (phase 2)
- Third package tier

**Security notes**
- `.env` is committed (auto-generated public keys only; repo is private). Never add secret keys to it.

## 17. Prompt 02 built in code (branch `prompt-02-onboarding`, repo `taxx-savage-2`)

Built directly in the app repo on branch `prompt-02-onboarding` (commit `9a9071a`), NOT merged to `main` yet. Type-checks and builds; the SQL parses; **not run against a live database and not tested in a browser**.

**What it adds:** paid-only admin returns list with payment id, preparer, email and date columns, Refunded status and an admin "Mark refunded"; onboarding gate (package payment, contract, documents, thank-you, questionnaire, review) for unapproved clients; admin Contract Templates, User Management with client detail (documents, signed contract, answers, PTIN, Approve / Request changes / Reject), Preparer Completeness, Questionnaire manager; onboarding events in `audit_log`.

**Security fixes found in the earlier build (fixed on the branch):**
- Clients could update their own `orders` and `returns` rows directly, so they could mark returns paid without paying. Those policies are dropped; payment is confirmed in a server function.
- Clients could change their own `clients.status` (self-approve). A trigger now blocks clients from changing status, package, approval or onboarding fields.

**Known limits / open items:**
- Payment is still confirmed from the browser: the server records the payment provider's order id but does not verify it with the provider. Before launch, verify payments server-side (webhook or API) or an attacker could call the confirm function without paying.
- Contract text is plain text (no HTML) to avoid script injection.
- "Registered PTINs" menu still opens the existing preparer-PTIN page; client PTINs show in Preparer Completeness and on the client detail page.
- Existing non-approved clients will be gated until an admin approves them.
- The new migration must be applied to the database before the code runs, or pages will error.
