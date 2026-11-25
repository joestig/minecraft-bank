# Bank

A private Minecraft-player currency website. The frontend is a static site for GitHub Pages; Supabase provides authentication, PostgreSQL storage, row-level security, and server-side RPCs.

## Included

- Email/password accounts with public Minecraft usernames (email addresses are never shown in the public ledger).
- Player balances calculated from active note records.
- Admin-only issuance of notes.
- Individual UUID notes, split into 1-notes, and merge back into a larger note.
- Amount-based transfers. The database automatically breaks a larger denomination when an exact amount is needed.
- Public ledger for mints, transfers, splits, merges, and casino rounds.
- House-funded dice, roulette, slots, and quick simulated blackjack.
- Admin overview of player currency supply, registered players, and house reserve.

## 1. Create the Supabase backend

1. Create a project at [supabase.com](https://supabase.com/).
2. Open **SQL Editor** in the Supabase dashboard.
3. Paste and run the complete `supabase/schema.sql` file.
4. Open **Project Settings → API** (or **Connect**) and copy the Project URL and the public `anon` key. Never use the `service_role` key in this website.
5. Open `app.js` and replace:
   - `YOUR_SUPABASE_PROJECT_URL`
   - `YOUR_SUPABASE_ANON_KEY`
6. In Supabase **Authentication → URL Configuration**, add your GitHub Pages URL to the Site URL and Redirect URLs. For example: `https://YOUR_GITHUB_NAME.github.io/YOUR_REPOSITORY/`.
7. In **Authentication → Providers → Email**, choose whether email confirmation is required. If it is enabled, players must confirm their email before logging in.

## 2. Create the administrator

1. Open the site and create the account that should be the administrator.
2. In Supabase SQL Editor, run this statement, replacing the username with the exact username you registered:

```sql
update public.profiles
set is_admin = true
where lower(username) = lower('YOUR_ADMIN_USERNAME')
  and is_system = false;
```

3. Log out and back in. The Administration page will appear for that account.
4. Other accounts are ordinary players. They cannot promote themselves or mint currency.

## 3. Publish on GitHub Pages

1. Create a GitHub repository.
2. Upload `index.html`, `style.css`, `app.js`, `README.md`, and the `supabase` folder.
3. In the repository, open **Settings → Pages**.
4. Set the source to **Deploy from a branch**, select `main` and `/ (root)`, then save.
5. Wait for GitHub Pages to publish the site. Add the resulting URL to Supabase Auth's allowed redirect URLs.

No Node server is required. GitHub Pages hosts the frontend; Supabase is the backend.

## 4. Economy rules

- One note represents one diamond.
- Only an administrator can mint new notes after verifying a diamond deposit in Minecraft.
- Splitting replaces one active note with child notes of value 1. The parent is permanently marked `split`.
- Merging replaces multiple active notes with one new note. The source notes are permanently marked `merged`.
- Transfers move ownership of active notes. If a denomination is too large, the backend splits it as needed and sends the exact amount.
- Old notes and note relationships are retained for audit/history; they are not deleted or reactivated.
- The public ledger records the message for every player action.
- The casino uses the same notes as the rest of the economy. It is funded by a separate 100,000-note house reserve (seeded as 1,000 notes of 100) created by the SQL setup. It cannot pay a winning round if the reserve cannot cover the maximum possible payout for that wager.

## 5. Security notes

- The Supabase anon key is designed to be public in a browser app. Database RLS and the RPC functions enforce access. **Never put a Supabase service-role key in `app.js` or commit it to GitHub.**
- Users cannot directly insert, update, or delete balances, notes, ledger rows, or casino rounds through the public API.
- Every balance-changing operation runs in a database transaction through a restricted function.
- For a real server, enable Supabase Auth email confirmation, use strong passwords, and consider rate limits / CAPTCHA for sign-up and login.
- The casino is a simulated, friends-only game feature. It has no real-money deposits, withdrawals, or cash-outs. Quick Blackjack is a single-round simplified simulation, not a full interactive card table.
- The SQL setup seeds the house with a 100,000-note reserve. That reserve is separate from player currency supply. Admin issuance increases player supply; casino wins and losses move existing notes between players and the house.

## 6. Notes on lineage

Each split child records its parent UUID in `notes.parent_id`; `note_relations` records split and merge edges. The original and intermediate records remain in the database, so note ancestry can be audited. A merge can have multiple source notes, represented by multiple `note_relations` rows pointing to the resulting note.

## Troubleshooting

- **“Setup required”**: the Supabase URL and anon key have not been added to `app.js`.
- **Username already taken**: choose another username; usernames are unique, case-insensitively checked when transferring and minting.
- **Admin page missing**: promote the account using the SQL command above, then log out and back in.
- **Email confirmation**: check the confirmation email, or adjust the Supabase Email provider settings for a private test server.
- **Casino reserve too low**: the house has insufficient funds to cover the maximum payout for the requested wager. An administrator can replenish the house reserve only by following a deliberate audited database operation; do not casually alter note ownership from the browser.
