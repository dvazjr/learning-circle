# Turning on sign-in and sync

About 15 minutes, all on the free tier. Until you finish this, the app still
works — it just saves on each person's own device, and no sign-in box appears.

## 1. Make the project

1. Go to supabase.com, sign up, click **New project**.
2. Name it `learning-circle`. Pick a region near Phoenix (US West). Set a
   database password and save it somewhere — you won't need it for this, but
   losing it is annoying later.
3. Wait about two minutes for it to finish building.

## 2. Make the table

Left sidebar → **SQL Editor** → **New query**. Paste all of this in and click
**Run**. It creates the table and locks it down so each person can only ever
see their own rows.

```sql
create table public.circles (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users on delete cascade,
  date text,
  who text,
  kairos text,
  answers jsonb not null default '[]'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.circles enable row level security;

create policy "read own circles" on public.circles
  for select using (auth.uid() = user_id);
create policy "add own circles" on public.circles
  for insert with check (auth.uid() = user_id);
create policy "edit own circles" on public.circles
  for update using (auth.uid() = user_id);
create policy "delete own circles" on public.circles
  for delete using (auth.uid() = user_id);

create index circles_user_idx on public.circles (user_id, updated_at desc);
```

## 3. Set up sign-in

Left sidebar → **Authentication** → **Sign In / Providers**.

- Email should already be on. Leave it on.
- Turn **Confirm email** OFF. This matters: Supabase's built-in email sender
  is rate limited to a couple of messages an hour, so confirmation emails will
  fail for a group. With it off, people create an account and are in.
- Leave every other provider off.

Then go to **Authentication → URL Configuration** and set Site URL to:

```
https://dvazjr.github.io/learning-circle/
```

## 4. Paste your two values into the app

Left sidebar → **Project Settings** → **API Keys** (or **Data API**). Copy:

- **Project URL** — looks like `https://abcdefgh.supabase.co`
- **anon public** key — a long string starting `eyJ...`

Open `index.html` in your GitHub repo, click the pencil, and find these lines
near the bottom, just under `<script>`:

```js
const SUPABASE_URL = "";
const SUPABASE_ANON_KEY = "";
```

Put your two values inside the quotes, commit, and wait a minute. The sign-in
box will appear on the site.

The anon key is meant to be public — it's safe in the file. The row-level
security you set up in step 2 is what actually protects the data, which is why
you shouldn't skip it.

## 5. Check it

1. Open the site. Create an account with your email and a password.
2. Fill in a circle and save it.
3. Open the site on a different device, sign in with the same email. Your
   circle should be there.

## What people will see

- **Create account** on their first visit, **Sign in** after that.
- If they already saved circles on that device, a button appears offering to
  move those into their account. One tap.
- **Just this device** skips the whole thing for anyone who doesn't want an
  account. They keep the old browser-only behavior.

## Things to know

- Free tier pauses a project after a week with zero activity. A weekly group
  keeps it awake; if it ever pauses, you restore it with one click in the
  dashboard.
- Forgotten passwords need email, which is the rate-limited part. For a small
  group, resetting someone's password from the Authentication dashboard is the
  practical fix. If this becomes a nuisance, connect a free Resend account
  under Project Settings → Authentication → SMTP.
- You can see that rows exist in the dashboard, and as project owner you could
  read them. Worth saying plainly to your leaders, since these entries are
  personal.
