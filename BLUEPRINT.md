# Taxx Savage System Blueprint

**Status:** living document. Update the Change Log and the "Open Items" list every session.
**Last updated:** 2026-10-16 (v0.1)
**Build target:** HighLevel AI Studio (with Cloud: database, auth, file storage). This document is the spec; nothing is built from here.

---

## 1. Overview
Taxx Savage is a white-label tax software company. The system has a public website and three portals:
1. **Public website**: packages page, signup, payment, contract, document upload.
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
1. Packages page, then **Start Now** and the signup form.
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
6. **If payment succeeds:** client enters details **for each return** (client name, SSN, files, other info; different for every return).
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
- SSN and ID documents encrypted at rest, private file storage, access limited to admin and reviewers.
- SSN shown masked (last 4); reveal is logged.
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

## 15. Change Log
- v0.1 (2026-10-16): Initial blueprint from discovery conversation and screenshots (signup, client portal, return form, admin menu, review sheets).
