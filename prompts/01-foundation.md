# Prompt 01: Foundation (HighLevel AI Studio)

**Project:** Taxx Savage System
**Status:** ready to paste · v1 · 2026-10-16
**Depends on:** nothing. Run this first.
**Source of truth:** `BLUEPRINT.md` (sections 2, 8, 10, 12)

**Before pasting:** create a new AI Studio project on **TanStack Start** and enable **Cloud** (database, authentication, file storage) from the builder's More menu. Cloud needs TanStack Start.

---

## Paste this into AI Studio

```
Build the foundation for "Taxx Savage", a white-label tax software platform. This step creates login, roles, the database, and an editable menu system. Do NOT build the public website, Stripe payments, contracts, or return forms yet; those come in later steps.

USE CLOUD for the database, user authentication (email + password with password reset), and private file storage.

1) BRANDING
- App name: Taxx Savage.
- Logo: placeholder image area top-left (I will upload the real logo later).
- Colors: deep navy for headings and primary buttons, gold as accent, lime green (#d4f07a style) for highlights and banners, white cards on a very light background.
- Font: a clean professional sans-serif (for example Inter). Do not use a decorative serif.
- Mobile first. Every page must work well on a phone, with a hamburger button that opens the sidebar menu on small screens.

2) ROLES (stored on the user record)
- client: a tax preparer who signed up.
- admin: full access.
- reviewer: review team member.
After login, redirect by role: client -> /client, admin -> /admin, reviewer -> /review. A user can only open pages for their own role. Block everything else and show a "no access" page.

3) PAGES (simple shells for now)
- /login and /reset-password.
- /client: sidebar layout, welcome message "Welcome, [first name]", and a status line.
- /admin: sidebar layout with a dashboard shell.
- /review: sidebar layout with a dashboard shell.
- Each layout: header with logo, user email, and package label area (can be blank for now), left sidebar built from the menu table (see 5), and a Log Out link.

4) DATABASE TABLES (create with sensible types; use ids and timestamps on all)
- clients: user_id, company_name, first_name, last_name, email, phone, package_id, status (signed_up, contract_signed, docs_submitted, approved), ptin (admin can edit, client cannot), stripe_customer_id, approved_at.
- packages: name, price, stripe_price_id, contract_template_id, active. Seed with: Collab = 89, Pro = 599. (A third tier will be added later.)
- products: name, price, stripe_price_id, form_type, active. Seed with these names and no prices yet: Amended Return, Corporate Return, State Return, Non-Bank Product Return, Audit Protection, Refund Cards, Check Reset, File Extensions.
- orders: client_id, product_id, tax_year, quantity, amount, stripe_charge_id, status (paid, failed, refunded), created_at.
- returns: order_id, client_id, product_id, tax_year, taxpayer_first_initial, taxpayer_last_name, ssn_encrypted, ssn_last4, files, status, created_at.
- announcements: title, body, button_label, button_url, posted_at, active.
- menu_items: portal (client, admin, reviewer), section (text, can be empty), label, icon, url, link_type (internal, external, coming_soon), sort_order, visible (true/false), allowed_roles, allowed_package_ids (empty = all).
- audit_log: user_id, action, record, created_at.

5) MENU MANAGER (admin only, page /admin/menus)
- Table of all menu items grouped by portal, with section headers.
- Add, edit, delete, show/hide, and reorder (drag handle or up/down arrows) menu items.
- Fields: portal, section, label, icon, link type, URL, roles, packages.
- Sidebars in all three portals are built ONLY from the menu_items table. Never hardcode menu links.
- Seed starter items:
  ADMIN: Dashboard, User Management, Review Requests, Reviewer Activity, Review Team, Registered PTINs, Preparer Completeness, Products & Pricing, Contract Templates, Record Payments, Client Notifications, Client Messaging, Menu Manager, Settings. Mark all except Dashboard, Menu Manager, and Review Team as link type "coming_soon".
  CLIENT: Home, Profile & Billing, Fee Schedule, Log Out. Section "Tools": Tax Software, Request Total Audit Protection, Submit Amended Return, Submit Corporate Return, Submit State Return, Submit Non-Bank Product Return, File Extensions, Request Refund Cards, Request Check Reset, Messaging Center. Section "Reports": Payouts Dashboard, Advances, Prep Fees, Client Disbursement List, Mailing Fees, Rework Fees. Section "Premium Tools": Taxx Academy. Section "Resources": Mastermind Community. All except Home and Log Out start as "coming_soon".
  REVIEWER: Dashboard, Paid Returns, Messages, Profile. Mark all except Dashboard as "coming_soon".
- A "coming_soon" item shows a simple "This page is coming soon" screen.

6) REVIEW TEAM MANAGEMENT (admin only, page /admin/review-team)
- List reviewers (users with role reviewer), with name, email, status.
- Admin can add a reviewer (name + email, sends an invite or sets a temporary password) and deactivate one.
- Reviewers cannot see or change anything outside /review.

7) SECURITY RULES
- Clients can only read and write their own records.
- Only admin can edit packages, products, menu items, PTIN, and review team.
- Reviewers can read returns and orders, but cannot edit them, and cannot see clients' full SSN or documents.
- Uploaded files go to PRIVATE storage, never public.
- Log PTIN changes and any reveal of SSN in audit_log.

8) SEED USERS FOR TESTING
Create one test user for each role: admin, reviewer, and client. Show me the logins so I can test.

When finished, summarize what was built, list anything you could not do, and tell me how to test each role.
```

---

## After it builds: test checklist
- [ ] Admin, reviewer and client test users can log in; password reset email works
- [ ] Each role lands on the right portal and cannot open the others
- [ ] Sidebar menus appear in all three portals on desktop and phone (hamburger works)
- [ ] Menu Manager: add, edit, hide, reorder; changes show immediately
- [ ] "Coming soon" links show the placeholder page
- [ ] Admin can add and deactivate a reviewer
- [ ] Client cannot edit the PTIN field
- [ ] Packages seeded: Collab $89, Pro $599

## Notes to bring back to Claude
Record here what AI Studio built, what failed, and anything it changed. Then ask Claude to update `BLUEPRINT.md` and write Prompt 02.

- Result:
- Problems:
- Changes to the blueprint:

## Next
Prompt 02: public website, packages page and signup form.
