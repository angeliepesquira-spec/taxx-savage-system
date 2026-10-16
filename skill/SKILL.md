---
name: taxx-savage-system
description: Use when working on the Taxx Savage white-label tax software system (website, client portal, admin portal, review team portal, Stripe return payments, HighLevel AI Studio build). Loads the blueprint and rules so every session starts from the current spec and keeps it updated.
---

# Taxx Savage System Skill

## Purpose
Keep one source of truth for the Taxx Savage build. The spec lives in `BLUEPRINT.md` (repo root). The system is built by the owner in **HighLevel AI Studio** (Cloud: database, auth, file storage); Claude produces blueprints, database designs and build prompts, not the app itself.

## Every session
1. Read `BLUEPRINT.md` first, including **Open Items** and **Change Log**.
2. Do the task using the blueprint as the source of truth. Do not re-ask answered questions.
3. When a decision is made or a screenshot adds detail, **update `BLUEPRINT.md`** in the right section, tick or add Open Items, and append a dated Change Log line.
4. Bump the version and "Last updated" date at the top.
5. If this skill's rules change, update this file too.

## Fixed decisions (do not change without the owner saying so)
- Three portals: client (preparer), admin, review team, plus the public site. All mobile optimized.
- Build target: HighLevel AI Studio Cloud. Email via HighLevel CRM workflows. Forms are created in AI Studio.
- Packages: Collab $89, Pro $599, third tier TBD.
- Return forms are per return type; price per return is fixed per product; card on file via Stripe; payment must succeed before a return is recorded.
- Failed or cancelled payment = no record in admin or review portals. Refunds are done in Stripe.
- Each paid return is its own row; one list for all returns. Portal replaces the review Google Sheet.
- Review team v1 = paid returns list only (date, company, preparer name, email, review name, SSN last 4, tax year, type, Stripe charge ID).
- PTIN is not collected at signup; admin enters it manually after verifying the uploaded PTIN document.
- Menus are admin-editable (Menu Manager); never hardcode menus.
- Contract templates are managed in the admin portal and tied to packages.
- Brand: logo placeholder, gold/navy/lime-green colors, a professional sans-serif font (not Cormorant Garamond).

## Style for owner-facing replies
- Plain, clear language; short sections; end with specific questions or a next step.
- Never print full SSNs, bank numbers or secrets in chat or files; mask them.
- Ask only questions the blueprint does not already answer.

## Outputs this skill can produce
- Updated `BLUEPRINT.md`
- HighLevel AI Studio build prompts (one portal or feature at a time, in the blueprint's build order)
- Database table designs, form field lists, workflow plans for CRM emails
- Test checklists per portal
