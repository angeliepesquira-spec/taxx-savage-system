# Taxx Savage: Launch Checklist

**Rule: nothing here is optional for "real clients". Do not launch until every item under 1 and 2 is checked.**
The product is still in the building stage. This list is the "before launch" gate. Whoever works on the project (human or AI) should bring it up whenever launching, going live, or onboarding real offices is mentioned.

Last updated: 2026-10-24

## 1. Must be done before any real client uses it (security and money)
- [ ] **Verify payments on the server.** Today the browser tells the server "I paid", and the server believes it. The server must confirm each payment with the payment provider before marking an order or package as paid.
- [ ] **Connect Stripe directly** (needed to show the real card on file, charge a saved card, and keep products in sync). Refunds are made in Stripe, not in the app.
- [ ] **Turn payment test mode OFF** (Admin > Settings). Check that the "continue without paying (test)" and "Mark as paid (test)" buttons are gone.
- [ ] **Remove the Quick sign-in box** from the login page (`quickAccounts` in `src/routes/login.tsx`). It has test passwords in the code. Delete the three test accounts (`admin@`, `reviewer@`, `client@taxxsavage.com`) or give them new passwords. Make a real admin account with a strong password.
- [ ] **2-factor sign-in for admin accounts.** Stronger password rules (today almost any password is accepted). Limit repeated sign-in attempts.
- [ ] **Independent security review** of the code and the database rules. Fix what it finds. Re-test that one office can never see another office's data.
- [ ] **Backups confirmed and restore tested.** Know the backup plan, how long copies are kept, and do one practice restore.
- [ ] **Legal duties confirmed** with an accountant or lawyer (for example IRS Publication 4557, the FTC Safeguards Rule, a written information security plan, state data-breach rules). Privacy policy and terms published.

## 2. Must be done before many offices (hundreds) use it (scale and reliability)
- [ ] Ask AI Studio: what are the backend plan limits (database size, file storage, active users), what backups exist, and what happens in an outage? Upgrade the plan if needed.
- [ ] **Paging and server-side search** for long lists: Returns, Review Requests, User Management, Reviewer Activity, Check Requests, Refund Card Requests, Registered PTINs. (They load every row today.)
- [ ] **Database indexes** for the columns lists filter and sort on.
- [ ] **Error alerts and uptime monitoring**, plus a status page or a plan for telling offices about downtime.
- [ ] Written **backup and recovery plan** (who does what if the server has a problem).
- [ ] **Load test** with sample data for about 500 offices.
- [ ] **Real-device test** on an iPhone, an Android phone and a tablet, including "Add to Home Screen".

## 3. Setup and content to finish
- [ ] Every database step applied in the live backend (menus and tables). Check that the menus show: Products & Pricing, Review Requests, Reviewer Activity, Questionnaire (URL must be `/admin/questionnaire`), FAQ, Check Requests, Refund Card Requests, Resources items.
- [ ] Prices set in **Products & Pricing** and the same prices in Payments: Collab ($89), Pro ($599), 10x (price still to decide), return types, Total Audit Protection ($59.99), Corporate Return ($99), Refund Cards flat fee ($27).
- [ ] Which menu items each package gets (Collab, Pro, 10x).
- [ ] Mastermind Community: link or content.
- [ ] Training videos: the `YOUTUBE_API_KEY` secret if there are more than 15 videos; check the order and titles.
- [ ] FAQ text reviewed. Two different text-line numbers appear: (267) 209-9329 and (267) 209-8329. Check every phone number, here and in the Check Reset note.
- [ ] Final contract template text, and the signed-copy email (CRM email).
- [ ] Brand set in Settings: company name, logo, three colors. Looked at on client, admin and review team screens.
- [ ] Domain and email sending set up and tested (needed for email-change confirmation links and password resets).
- [ ] Questionnaire questions reviewed and the starter sample questions replaced.

## 4. Moving from the old Bubble app
- [ ] Export clients and returns from Bubble; build and test an import into this app.
- [ ] Pilot with 2 or 3 friendly offices on their phones.
- [ ] Keep Bubble running until the new app has been stable for a while; then move the rest a few offices at a time.

## 5. If this will be sold to other companies (white label)
- [ ] White-label pass: move the company-specific items into Settings (training channel, support and IRS links, phone numbers) so a copy needs no code edits.
- [ ] One-page setup guide with one combined database message, so a new company can start in about 30 minutes.
- [ ] Each copy gets its own GitHub repository and its own backend (never share the original's).

## After launch (keep doing)
- Update packages and security patches regularly. Re-run the security review after big features.
- Check that backups still work, every few months.
