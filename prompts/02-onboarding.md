# Prompt 02: Onboarding (paid-only returns, contract, uploads, approval)

**Project:** Taxx Savage System
**Status:** ready to paste · v1 · 2026-10-18
**Depends on:** Prompt 01 and the existing app (repo `taxx-savage-2`)
**Source of truth:** `BLUEPRINT.md` (sections 4, 7, 8, 16)

**Decisions this prompt follows**
- Return flow stays as built: client enters return details, then pays.
- SSN stays last 4 only. Do not collect full SSNs anywhere.
- Signup stays on the hub app. The public website is a separate app that links to the hub signup.

**Before pasting:** this prompt edits the existing app. Tell AI Studio to inspect the current code first (the prompt says so). If you have a real contract, keep it handy to paste into the admin Contract Templates page after the build.

---

## Paste this into AI Studio

```
This app already exists (Taxx Savage). Do NOT rebuild or rename anything that already works. First inspect the current signup page, checkout page, returns tables, menu_items, clients table, storage bucket "client-files", and the admin/client/review portals, then add the following. Reuse existing components, styles and naming.

PART A. ONLY PAID RETURNS COUNT (fix first)
1. A return must appear in the admin returns list and the reviewer Paid Returns list ONLY after its payment succeeds. Orders and returns created before payment must be marked "pending" and hidden from admin and reviewer lists (and from the client's "paid" views, though the client may see their own pending items marked "Awaiting payment").
2. If payment fails or is cancelled, the order and its returns stay "pending" or become "cancelled" and never appear to admin or reviewers. Add a "Resume payment" action on the client's pending items.
3. If a payment is refunded, mark the order and its returns "refunded" and show a "Refunded" badge in the admin list. Refunds themselves are done in the payment provider, not in this app.
4. Reviewers and admin see these fields per paid return: date and time, company name, preparer first and last name, email, review name (taxpayer first initial + last name), SSN last 4, tax year, type (product), payment confirmation ID. Add any that are missing.
5. Never collect or store a full SSN anywhere. Keep last 4 only. Remove or ignore the unused ssn_encrypted column in the UI.
6. Show me a short test note: how to verify a failed payment leaves nothing in the admin and reviewer lists.

PART B. ONBOARDING STATUS AND STEPS
The client journey after signup has these steps, in order:
 1) Package payment   2) Sign contract   3) Upload documents   4) Thank-you / next steps   5) Experience questionnaire   6) Under review   7) Approved
Add to the clients table (new migration, do not break existing rows): onboarding_step (text), package_paid_at, contract_signed_at, docs_submitted_at, questionnaire_submitted_at, approved_at, approved_by, rejection_note. Use the existing status values (signed_up, contract_signed, docs_submitted, approved) and add "questionnaire_done" and "needs_changes" if useful.
- Keep the existing signup and account creation. After signup, send the client to the package payment for the package chosen at signup (a package can be preselected from a link such as /signup?package=collab or pro; keep this working). Use the existing Payments integration for package prices.
- A client who is not "approved" can only see a simple onboarding checklist page at /client with their current step and a button to continue. All other client menu pages show "Your profile is under review" until approved. After approval, the normal client portal opens with the message "Your Profile Has Been Approved".
- The thank-you page after the document step explains next steps: questionnaire, then review (1 to 3 business days), then portal access.

PART C. CONTRACT TEMPLATES AND SIGNING
1. New table contract_templates: id, name, body (rich text or HTML, so I can paste the full contract), package_id (nullable; null means default), active, version, updated_at. Seed one placeholder template named "Default Agreement" with a few paragraphs of placeholder text and the line "Replace this with your real contract."
2. Admin page /admin/contracts ("Contract Templates"): list, create, edit, activate/deactivate, and assign each template to a package. Support at least 3 templates. Add it to the admin menu through menu_items.
3. Client signing page /client/contract (mobile friendly):
   - Shows the template for the client's package (fall back to the default one). Show it in a scrollable area with the full text.
   - Fields: company name (prefilled from the client), print name, signature (draw with finger/mouse AND a typed option), date (auto), and a checkbox "I agree to this agreement".
   - On submit: store the signed record in a new table contracts_signed (client_id, template_id, template_version, template_body_snapshot, company_name, print_name, signature_image or typed text, signed_at, ip_address if available). Set contract_signed_at on the client.
4. Admin can open any client's signed contract (read-only) and download it as a PDF or printable page.
5. Do NOT add bank routing or account numbers to the contract page for now.

PART D. DOCUMENT UPLOADS
1. New table client_documents: id, client_id, doc_type, file_path, file_name, uploaded_at, status (uploaded, accepted, rejected), reviewer_note.
2. Client page /client/documents (mobile friendly, camera or file upload) with these upload slots, shown as labeled rows with status:
   - Photo ID (driver's license, passport or government ID)  [required]
   - W-9  [required]
   - Social Security card  [required]
   - PTIN (IRS document)  [required]
   - EFIN summary (IRS document)  [required]
   - Anticipated bank product returns  [required]
   - EIN (IRS document)  [optional]
   Accept PDF, JPG, PNG, HEIC up to 15 MB. Allow replacing a file. A "Submit Documents" button is enabled only when all required slots have a file. On submit set docs_submitted_at.
3. Files go ONLY to the private "client-files" bucket under the client's own folder. Never public URLs; use short-lived signed links for viewing. The client can see their own files. Admin can see all. Reviewers cannot see clients' onboarding documents.
4. The signup form does NOT ask for a PTIN. Admin types the PTIN in the client record after verifying the uploaded PTIN document (the existing admin-only PTIN field and audit log stay as they are).

PART E. EXPERIENCE QUESTIONNAIRE
1. New tables questionnaire_questions (id, question, type: short_text | long_text | single_choice | multi_choice | yes_no, options, required, sort_order, active) and questionnaire_answers (client_id, question_id, answer, submitted_at).
2. Admin page /admin/questionnaire to add, edit, reorder and deactivate questions. Seed 5 placeholder questions: years of tax preparation experience, software used before, number of returns prepared last season, do you have an EFIN (yes/no), how did you hear about us.
3. Client page /client/questionnaire shows the active questions; submitting sets questionnaire_submitted_at and moves the client to "Under review".

PART F. ADMIN REVIEW AND APPROVAL
1. Admin page User Management (/admin/users, add to menu): table of ALL clients with columns: name, company, email, phone, package, onboarding step, status, signed up date, PTIN. Search and filter by status and package. Row click opens a client detail page.
2. Client detail page (admin only): profile, package, payment status, signed contract (view/download), uploaded documents (open each, mark accepted or rejected with a note), questionnaire answers, PTIN field (admin edits), audit trail.
3. Buttons: Approve (sets approved_at, approved_by, status approved), Request Changes (sets needs_changes with a note the client sees on their onboarding page), and Reject.
4. Admin page "Preparer Completeness" (/admin/completeness, add to menu): table with one row per client and check marks for: package paid, contract signed, all documents uploaded, PTIN entered, questionnaire done, approved. Filter for "missing PTIN" and "waiting for review".
5. Admin page "Registered PTINs": make the existing /admin/ptins list real: client name, company, PTIN, entered by, entered at. Admin can add or edit.
6. Block approval if the PTIN is empty, with a clear message.

PART G. NOTIFICATIONS (placeholders only)
Create an events table (or use audit_log) that records: signup_completed, package_paid, contract_signed, documents_submitted, questionnaire_submitted, approved, changes_requested. I will connect my CRM to send emails from these later. Do not build email sending now.

GENERAL RULES
- Mobile first on every new page. Keep brand colors and the existing font.
- All new pages go in the correct menu through the menu_items table (admin and client). Never hardcode menu links. Mark anything unfinished as "coming soon".
- Security: clients only read/write their own rows; only admins edit templates, questions, PTINs and approvals; reviewers cannot see onboarding documents or contracts; all uploads private.
- Add a migration file for every database change. Do not edit old migrations.

When finished: summarize what you built and changed, list anything you could not do, and give me a step-by-step test script for: a failed payment, a new client from signup to approved, and an admin Request Changes round trip.
```

---

## After it builds: test checklist
- [ ] A cancelled or failed payment leaves nothing in the admin and reviewer lists; "Resume payment" works
- [ ] A paid return shows all fields (date, company, preparer, email, review name, SSN last 4, tax year, type, payment ID)
- [ ] New client: signup (package preselected) > package payment > contract > documents > thank-you > questionnaire > "under review"
- [ ] Unapproved client sees only the onboarding checklist
- [ ] Contract works on a phone (draw signature); signed copy opens in admin
- [ ] Submit Documents stays disabled until all 6 required files are uploaded
- [ ] Admin: User Management, client detail, documents accept/reject, Request Changes shows a note to the client
- [ ] Approve is blocked until the PTIN is entered
- [ ] Preparer Completeness and Registered PTINs pages show real data
- [ ] Reviewer cannot open onboarding documents or contracts
- [ ] No page asks for a full SSN

## Notes to bring back to Claude
- Result:
- Problems:
- Changes to the blueprint:

## Next
Prompt 03: remaining return forms (Audit Protection, Refund Cards, Check Reset), Client Notifications and announcements, and CRM email events.
