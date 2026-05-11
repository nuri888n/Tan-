# Agency Finance Tool by Safari Studios
## Complete Build Guide for Agencies — v13.9

## QUICK START — HOW TO USE THIS KIT

1. Open a new Claude chat at **claude.ai**
2. Upload **both files** as attachments:
   - `AGENCY_FINANCE_BUILD_GUIDE.md` ← this file
   - `AGENCY_FINANCE_SOURCE_TEMPLATE.html` ← the source code
3. Type: **"Please read the build guide and start the build process."**
4. Answer the 11 questions — type `skip` for anything you don't have yet
5. Download the customized `index.html` the AI returns
6. Follow the Supabase + Vercel setup in PART 2 of this guide

> **Tip:** You can skip any question and fill in the details later directly inside the app under Settings → Company Profile — no need to touch the code again.

---

> **Powered by:** Agency Finance Tool by Safari Studios  
> **Stack:** Single-page HTML app · Supabase (PostgreSQL + Auth) · Vercel (hosting) · Chart.js  
> **Target audience:** Any talent/creator agency that wants their own branded finance dashboard

---

## This Build Kit Contains Two Files

| File | Purpose |
|------|---------|
| `AGENCY_FINANCE_BUILD_GUIDE.md` | This guide — human setup instructions + AI Build Prompt |
| `AGENCY_FINANCE_SOURCE_TEMPLATE.html` | Complete cleaned source code — all private data removed, placeholders in place |

**Send both files together.** The AI Build Prompt (Part 4) instructs the AI to read the source template and apply your company details to it.

---

## How to Use This Guide

1. **Read PART 1** — Choose which features you want
2. **Read PART 2** — Set up Supabase and Vercel yourself (step by step, assumes zero prior knowledge)
3. **Read PART 3** — Apply the code to your repository
4. **Copy PART 4** — Paste the complete AI Build Prompt to Claude, ChatGPT, or any AI, then also paste/upload the `AGENCY_FINANCE_SOURCE_TEMPLATE.html` file so the AI has the full source code to work from
5. **Use PART 5** — Verify everything works

---

## PART 1: Feature Options

Before setting up anything, decide which features you need.

### Option 1 — Full Build (Recommended)
Includes ALL features out of the box:

| Feature | What it does |
|---------|-------------|
| **Dashboard** | Monthly KPIs, creator breakdown table, income vs. expenses bar chart, expense payer donuts |
| **Creators** | Add/edit creators with commission splits (flat % or tiered), currency, termination system |
| **Income** | Log income by creator and source, multi-currency, repeat entries, date display |
| **Expenses** | Log expenses by category and payer, link to staff, repeat entries, date display |
| **Staff** | Staff members with fixed/hourly pay, creator assignments, payment tracking |
| **Invoices** | Create branded PDF invoices with line items, send by email, track paid/unpaid |
| **Contracts** | Create and sign service agreements (PDF), send via email |
| **Analysis** | Per-creator performance charts, period comparisons |
| **Archive** | Month snapshots with search, terminated creator history |
| **Users & Permissions** | Role-based access (admin/user/guest), per-module permission control |
| **Activity Log** | Track all income/expense/invoice actions with timestamps and user info |
| **Settings** | Expense categories, invoice categories, company profile, exchange rates |
| **Export Backup** | Download all data as timestamped JSON |
| **Monthly Summary PDF** | Print-ready monthly finance report |
| **Multi-currency** | USD, EUR, GBP, CHF, USDT, USDC, BTC with live exchange rates |

**→ If you want Option 1:** Continue to PART 2. The AI build prompt in PART 4 uses the full feature set.

### Option 2 — Custom Feature Selection
You want most features but want to skip a few. Tell the AI in PART 4 which modules to remove when you paste the prompt.

Example: "Remove the Contracts tab and the Archive tab."

### Option 3 — Describe Your Use Case
You have a different type of agency (not OnlyFans). Describe it to the AI in PART 4 and the AI will adapt the contract template, terminology, and feature set accordingly.

Example: "We are a music management agency. Replace 'OnlyFans' with 'streaming/tour revenue'. We don't need the contract module."

---

## PART 2: Human Setup (Do This First — No Coding Required)

### Step 1: Create a GitHub Account and Repository

**GitHub** is where your code lives. It also connects to Vercel for auto-deployment.

1. Go to **https://github.com** and click **Sign up**
2. Create an account with your email, a username, and a password
3. Verify your email address
4. Click the **+** button in the top right → **New repository**
5. Name it something like `agency-finance-tool`
6. Set it to **Private** (so your code is not publicly visible)
7. Check **Add a README file**
8. Click **Create repository**

You now have an empty repository. You will upload your customized `index.html` here after PART 4.

---

### Step 2: Create a Supabase Project (Complete Beginner Guide)

**Supabase** is your database and authentication system. It's free to start.

#### 2.1 — Create a Supabase Account

1. Go to **https://supabase.com** and click **Start your project**
2. Sign up with GitHub or email
3. Verify your email if prompted

#### 2.2 — Create a New Project

1. After logging in, click **New project**
2. Choose or create an **organization** (your agency name is fine)
3. Fill in:
   - **Project name:** `agency-finance` (or anything you like)
   - **Database password:** Create a strong password — **save it somewhere safe!**
   - **Region:** Choose the region closest to you (e.g., EU West for Europe, US East for USA)
4. Click **Create new project**
5. Wait 1–2 minutes for the project to initialize. You will see a green "Project is ready" status.

#### 2.3 — Get Your API Credentials

1. In your Supabase project, click **Project Settings** (gear icon) in the left sidebar
2. Click **API** under Project Settings
3. You will see:
   - **Project URL** — looks like `https://abcdefghijklmno.supabase.co`
   - **Project API keys → anon / public** — a long string starting with `eyJ`
4. **Copy both values** and save them. You will need them in PART 3.

⚠️ **Never share your `service_role` key. Only use the `anon` key in your app.**

#### 2.4 — Run the SQL Migrations (Set Up Your Database Tables)

Your database needs tables before it can store any data. You will run 10 SQL scripts in order.

1. In your Supabase project, click **SQL Editor** in the left sidebar
2. Click **New query**
3. **Copy each SQL block below** (in order, one at a time), paste it into the editor, and click **Run**
4. After each one, you should see: `Success. No rows returned.`

**Run these in order: Migration 0 → 005 → 006 → 007 → 008 → 009 → 010 → 011 → 012 → 013**

---

#### MIGRATION 0 — Core Tables (Run this first)

```sql
-- Agency Finance Tool — Core Tables
-- Run this first before any other migration.

create extension if not exists pgcrypto;

-- Creators table
create table if not exists creators (
  id text primary key,
  name text not null,
  currency text default 'USD',
  commission numeric default 0,
  tiers jsonb default null,
  fixed_costs jsonb default null,
  active boolean default true,
  terminated boolean default false,
  terminated_month text,
  gender text,
  created_at timestamptz not null default now()
);

-- Monthly revenue/cost data per creator per month
create table if not exists monthly_data (
  id text primary key,
  month_key text not null,
  creator_id text not null references creators(id) on delete cascade,
  revenue numeric default 0,
  variable_costs numeric default 0,
  created_at timestamptz not null default now(),
  unique (month_key, creator_id)
);

-- Month snapshots for the Archive tab
create table if not exists month_snapshots (
  id text primary key,
  month_key text not null,
  data jsonb,
  created_at timestamptz not null default now()
);

-- Notifications / alerts
create table if not exists notifications (
  id text primary key,
  message text,
  type text default 'info',
  read boolean default false,
  created_at timestamptz not null default now()
);

-- Expenses
create table if not exists expenses (
  id text primary key,
  name text,
  amount numeric default 0,
  currency text default 'USD',
  category text,
  month_key text,
  creator_id text,
  paid_by text default 'Bank',
  repeat boolean default false,
  recurring boolean default false,
  notes text,
  staff_member_id text,
  expense_date text,
  created_at timestamptz not null default now()
);

-- Income entries
create table if not exists income_entries (
  id text primary key,
  note text,
  source text,
  amount numeric default 0,
  currency text default 'USD',
  creator_id text,
  month_key text,
  revenue_type text default 'gross',
  repeat boolean default false,
  income_date text,
  created_at timestamptz not null default now()
);

-- Staff members
create table if not exists staff_members (
  id text primary key,
  name text not null,
  role text,
  payment_type text default 'fixed',
  fixed_monthly numeric default 0,
  hourly_rate numeric default 0,
  currency text default 'USD',
  active boolean default true,
  created_at timestamptz not null default now()
);

-- Staff creator assignments
create table if not exists staff_assignments (
  id text primary key,
  staff_id text not null references staff_members(id) on delete cascade,
  creator_id text not null references creators(id) on delete cascade,
  split_percent numeric default 0,
  active boolean default true,
  created_at timestamptz not null default now()
);

-- Staff payment records
create table if not exists staff_payments (
  id text primary key,
  staff_id text not null references staff_members(id) on delete cascade,
  month_key text not null,
  base_amount numeric default 0,
  hours_worked numeric default 0,
  commission_amount numeric default 0,
  currency text default 'USD',
  paid boolean default false,
  created_at timestamptz not null default now()
);

-- Enable RLS on all tables (open policies for now — locked down in Migration 007)
alter table creators enable row level security;
alter table monthly_data enable row level security;
alter table month_snapshots enable row level security;
alter table notifications enable row level security;
alter table expenses enable row level security;
alter table income_entries enable row level security;
alter table staff_members enable row level security;
alter table staff_assignments enable row level security;
alter table staff_payments enable row level security;

create policy "allow all creators" on creators for all using (true) with check (true);
create policy "allow all monthly_data" on monthly_data for all using (true) with check (true);
create policy "allow all month_snapshots" on month_snapshots for all using (true) with check (true);
create policy "allow all notifications" on notifications for all using (true) with check (true);
create policy "allow all expenses" on expenses for all using (true) with check (true);
create policy "allow all income_entries" on income_entries for all using (true) with check (true);
create policy "allow all staff_members" on staff_members for all using (true) with check (true);
create policy "allow all staff_assignments" on staff_assignments for all using (true) with check (true);
create policy "allow all staff_payments" on staff_payments for all using (true) with check (true);

-- Indexes
create index if not exists creators_active_idx on creators(active);
create index if not exists monthly_data_month_key_idx on monthly_data(month_key);
create index if not exists monthly_data_creator_id_idx on monthly_data(creator_id);
create index if not exists expenses_month_key_idx on expenses(month_key);
create index if not exists expenses_creator_id_idx on expenses(creator_id);
create index if not exists income_entries_month_key_idx on income_entries(month_key);
create index if not exists income_entries_creator_id_idx on income_entries(creator_id);

notify pgrst, 'reload schema';
```

✅ **After running:** You should see "Success. No rows returned."

---

#### MIGRATION 005 — Invoices

```sql
create table if not exists invoices (
  id text primary key,
  invoice_number text,
  client_id text,
  client_name text not null,
  client_email text,
  client_phone text,
  client_address text,
  date text,
  due_date text,
  currency text not null default 'USD',
  status text not null default 'unpaid',
  items jsonb default '[]'::jsonb,
  payment_instructions text,
  notes text,
  total numeric default 0,
  sent_at timestamptz,
  sent_to text,
  paid_at timestamptz,
  created_at timestamptz not null default now()
);

alter table invoices add column if not exists client_id text;
alter table invoices add column if not exists client_phone text;
alter table invoices add column if not exists client_address text;
alter table invoices add column if not exists sent_at timestamptz;
alter table invoices add column if not exists sent_to text;
alter table invoices add column if not exists paid_at timestamptz;
alter table invoices add column if not exists items jsonb default '[]'::jsonb;

create table if not exists invoice_clients (
  id text primary key,
  name text not null,
  email text,
  phone text,
  company text,
  address text,
  notes text,
  created_at timestamptz not null default now()
);

alter table invoices enable row level security;
alter table invoice_clients enable row level security;

drop policy if exists "allow all invoices" on invoices;
drop policy if exists "allow all invoice_clients" on invoice_clients;

create policy "allow all invoices" on invoices for all using (true) with check (true);
create policy "allow all invoice_clients" on invoice_clients for all using (true) with check (true);

create index if not exists invoices_status_idx on invoices(status);
create index if not exists invoices_client_id_idx on invoices(client_id);
create index if not exists invoices_created_at_idx on invoices(created_at);
create index if not exists invoice_clients_name_idx on invoice_clients(name);

do $$
begin
  begin alter publication supabase_realtime add table invoices; exception when duplicate_object then null; end;
  begin alter publication supabase_realtime add table invoice_clients; exception when duplicate_object then null; end;
end $$;

notify pgrst, 'reload schema';
```

---

#### MIGRATION 006 — User Roles & Permissions

```sql
create extension if not exists pgcrypto;

create table if not exists user_profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  email text,
  full_name text,
  role text not null default 'guest' check (role in ('admin', 'user', 'guest')),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table if not exists user_permissions (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  module text not null check (module in (
    'dashboard','creators','expenses','income','invoices','staff',
    'analysis','archive','alerts','settings','users'
  )),
  action text not null check (action in ('view','create','edit','delete')),
  allowed boolean not null default true,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  unique (user_id, module, action)
);

create index if not exists user_profiles_role_idx on user_profiles(role);
create index if not exists user_profiles_email_idx on user_profiles(email);
create index if not exists user_permissions_user_id_idx on user_permissions(user_id);
create index if not exists user_permissions_module_action_idx on user_permissions(module, action);

create or replace function has_permission(p_user_id uuid, module_name text, action_name text)
returns boolean language sql security definer set search_path = public stable
as $$
  select exists (
    select 1 from user_profiles up where up.id = p_user_id and up.role = 'admin'
  )
  or exists (
    select 1 from user_permissions perm
    where perm.user_id = p_user_id and perm.module = module_name
      and perm.action = action_name and perm.allowed = true
  );
$$;

grant execute on function has_permission(uuid, text, text) to authenticated;

alter table user_profiles enable row level security;
alter table user_permissions enable row level security;

drop policy if exists "users can read own profile" on user_profiles;
drop policy if exists "admins can manage profiles" on user_profiles;
drop policy if exists "users can read own permissions" on user_permissions;
drop policy if exists "admins can manage permissions" on user_permissions;

create policy "users can read own profile" on user_profiles for select to authenticated using (id = auth.uid());
create policy "admins can manage profiles" on user_profiles for all to authenticated
  using (has_permission(auth.uid(), 'users', 'edit'))
  with check (has_permission(auth.uid(), 'users', 'edit'));
create policy "users can read own permissions" on user_permissions for select to authenticated using (user_id = auth.uid());
create policy "admins can manage permissions" on user_permissions for all to authenticated
  using (has_permission(auth.uid(), 'users', 'edit'))
  with check (has_permission(auth.uid(), 'users', 'edit'));

notify pgrst, 'reload schema';
```

---

#### MIGRATION 007 — Secure RLS (Permission-Based Policies)

```sql
-- Replace open "allow all" policies with permission-based policies on all business tables.

alter table creators enable row level security;
drop policy if exists "allow all creators" on creators;
drop policy if exists "Allow all creators" on creators;
drop policy if exists "creators_select" on creators;
drop policy if exists "creators_insert" on creators;
drop policy if exists "creators_update" on creators;
drop policy if exists "creators_delete" on creators;
create policy "creators_select" on creators for select to authenticated using (has_permission(auth.uid(), 'creators', 'view'));
create policy "creators_insert" on creators for insert to authenticated with check (has_permission(auth.uid(), 'creators', 'create'));
create policy "creators_update" on creators for update to authenticated using (has_permission(auth.uid(), 'creators', 'edit')) with check (has_permission(auth.uid(), 'creators', 'edit'));
create policy "creators_delete" on creators for delete to authenticated using (has_permission(auth.uid(), 'creators', 'delete'));

alter table monthly_data enable row level security;
drop policy if exists "allow all monthly_data" on monthly_data;
drop policy if exists "monthly_data_select" on monthly_data;
drop policy if exists "monthly_data_insert" on monthly_data;
drop policy if exists "monthly_data_update" on monthly_data;
drop policy if exists "monthly_data_delete" on monthly_data;
create policy "monthly_data_select" on monthly_data for select to authenticated using (has_permission(auth.uid(), 'dashboard', 'view'));
create policy "monthly_data_insert" on monthly_data for insert to authenticated with check (has_permission(auth.uid(), 'dashboard', 'create'));
create policy "monthly_data_update" on monthly_data for update to authenticated using (has_permission(auth.uid(), 'dashboard', 'edit')) with check (has_permission(auth.uid(), 'dashboard', 'edit'));
create policy "monthly_data_delete" on monthly_data for delete to authenticated using (has_permission(auth.uid(), 'dashboard', 'delete'));

alter table month_snapshots enable row level security;
drop policy if exists "allow all month_snapshots" on month_snapshots;
drop policy if exists "month_snapshots_select" on month_snapshots;
drop policy if exists "month_snapshots_insert" on month_snapshots;
drop policy if exists "month_snapshots_update" on month_snapshots;
drop policy if exists "month_snapshots_delete" on month_snapshots;
create policy "month_snapshots_select" on month_snapshots for select to authenticated using (has_permission(auth.uid(), 'dashboard', 'view') or has_permission(auth.uid(), 'archive', 'view'));
create policy "month_snapshots_insert" on month_snapshots for insert to authenticated with check (has_permission(auth.uid(), 'dashboard', 'create') or has_permission(auth.uid(), 'archive', 'create'));
create policy "month_snapshots_update" on month_snapshots for update to authenticated using (has_permission(auth.uid(), 'dashboard', 'edit') or has_permission(auth.uid(), 'archive', 'edit')) with check (has_permission(auth.uid(), 'dashboard', 'edit') or has_permission(auth.uid(), 'archive', 'edit'));
create policy "month_snapshots_delete" on month_snapshots for delete to authenticated using (has_permission(auth.uid(), 'dashboard', 'delete') or has_permission(auth.uid(), 'archive', 'delete'));

alter table notifications enable row level security;
drop policy if exists "allow all notifications" on notifications;
drop policy if exists "notifications_select" on notifications;
drop policy if exists "notifications_insert" on notifications;
drop policy if exists "notifications_update" on notifications;
drop policy if exists "notifications_delete" on notifications;
create policy "notifications_select" on notifications for select to authenticated using (has_permission(auth.uid(), 'alerts', 'view'));
create policy "notifications_insert" on notifications for insert to authenticated with check (has_permission(auth.uid(), 'alerts', 'create'));
create policy "notifications_update" on notifications for update to authenticated using (has_permission(auth.uid(), 'alerts', 'edit')) with check (has_permission(auth.uid(), 'alerts', 'edit'));
create policy "notifications_delete" on notifications for delete to authenticated using (has_permission(auth.uid(), 'alerts', 'delete'));

alter table expenses enable row level security;
drop policy if exists "allow all expenses" on expenses;
drop policy if exists "expenses_select" on expenses;
drop policy if exists "expenses_insert" on expenses;
drop policy if exists "expenses_update" on expenses;
drop policy if exists "expenses_delete" on expenses;
create policy "expenses_select" on expenses for select to authenticated using (has_permission(auth.uid(), 'expenses', 'view'));
create policy "expenses_insert" on expenses for insert to authenticated with check (has_permission(auth.uid(), 'expenses', 'create'));
create policy "expenses_update" on expenses for update to authenticated using (has_permission(auth.uid(), 'expenses', 'edit')) with check (has_permission(auth.uid(), 'expenses', 'edit'));
create policy "expenses_delete" on expenses for delete to authenticated using (has_permission(auth.uid(), 'expenses', 'delete'));

alter table income_entries enable row level security;
drop policy if exists "allow all income_entries" on income_entries;
drop policy if exists "income_entries_select" on income_entries;
drop policy if exists "income_entries_insert" on income_entries;
drop policy if exists "income_entries_update" on income_entries;
drop policy if exists "income_entries_delete" on income_entries;
create policy "income_entries_select" on income_entries for select to authenticated using (has_permission(auth.uid(), 'income', 'view'));
create policy "income_entries_insert" on income_entries for insert to authenticated with check (has_permission(auth.uid(), 'income', 'create'));
create policy "income_entries_update" on income_entries for update to authenticated using (has_permission(auth.uid(), 'income', 'edit')) with check (has_permission(auth.uid(), 'income', 'edit'));
create policy "income_entries_delete" on income_entries for delete to authenticated using (has_permission(auth.uid(), 'income', 'delete'));

alter table staff_members enable row level security;
drop policy if exists "allow all staff_members" on staff_members;
drop policy if exists "staff_members_select" on staff_members;
drop policy if exists "staff_members_insert" on staff_members;
drop policy if exists "staff_members_update" on staff_members;
drop policy if exists "staff_members_delete" on staff_members;
create policy "staff_members_select" on staff_members for select to authenticated using (has_permission(auth.uid(), 'staff', 'view'));
create policy "staff_members_insert" on staff_members for insert to authenticated with check (has_permission(auth.uid(), 'staff', 'create'));
create policy "staff_members_update" on staff_members for update to authenticated using (has_permission(auth.uid(), 'staff', 'edit')) with check (has_permission(auth.uid(), 'staff', 'edit'));
create policy "staff_members_delete" on staff_members for delete to authenticated using (has_permission(auth.uid(), 'staff', 'delete'));

alter table staff_assignments enable row level security;
drop policy if exists "allow all staff_assignments" on staff_assignments;
drop policy if exists "staff_assignments_select" on staff_assignments;
drop policy if exists "staff_assignments_insert" on staff_assignments;
drop policy if exists "staff_assignments_update" on staff_assignments;
drop policy if exists "staff_assignments_delete" on staff_assignments;
create policy "staff_assignments_select" on staff_assignments for select to authenticated using (has_permission(auth.uid(), 'staff', 'view'));
create policy "staff_assignments_insert" on staff_assignments for insert to authenticated with check (has_permission(auth.uid(), 'staff', 'create'));
create policy "staff_assignments_update" on staff_assignments for update to authenticated using (has_permission(auth.uid(), 'staff', 'edit')) with check (has_permission(auth.uid(), 'staff', 'edit'));
create policy "staff_assignments_delete" on staff_assignments for delete to authenticated using (has_permission(auth.uid(), 'staff', 'delete'));

alter table staff_payments enable row level security;
drop policy if exists "allow all staff_payments" on staff_payments;
drop policy if exists "staff_payments_select" on staff_payments;
drop policy if exists "staff_payments_insert" on staff_payments;
drop policy if exists "staff_payments_update" on staff_payments;
drop policy if exists "staff_payments_delete" on staff_payments;
create policy "staff_payments_select" on staff_payments for select to authenticated using (has_permission(auth.uid(), 'staff', 'view'));
create policy "staff_payments_insert" on staff_payments for insert to authenticated with check (has_permission(auth.uid(), 'staff', 'create'));
create policy "staff_payments_update" on staff_payments for update to authenticated using (has_permission(auth.uid(), 'staff', 'edit')) with check (has_permission(auth.uid(), 'staff', 'edit'));
create policy "staff_payments_delete" on staff_payments for delete to authenticated using (has_permission(auth.uid(), 'staff', 'delete'));

alter table invoices enable row level security;
drop policy if exists "allow all invoices" on invoices;
drop policy if exists "invoices_select" on invoices;
drop policy if exists "invoices_insert" on invoices;
drop policy if exists "invoices_update" on invoices;
drop policy if exists "invoices_delete" on invoices;
create policy "invoices_select" on invoices for select to authenticated using (has_permission(auth.uid(), 'invoices', 'view'));
create policy "invoices_insert" on invoices for insert to authenticated with check (has_permission(auth.uid(), 'invoices', 'create'));
create policy "invoices_update" on invoices for update to authenticated using (has_permission(auth.uid(), 'invoices', 'edit')) with check (has_permission(auth.uid(), 'invoices', 'edit'));
create policy "invoices_delete" on invoices for delete to authenticated using (has_permission(auth.uid(), 'invoices', 'delete'));

alter table invoice_clients enable row level security;
drop policy if exists "allow all invoice_clients" on invoice_clients;
drop policy if exists "invoice_clients_select" on invoice_clients;
drop policy if exists "invoice_clients_insert" on invoice_clients;
drop policy if exists "invoice_clients_update" on invoice_clients;
drop policy if exists "invoice_clients_delete" on invoice_clients;
create policy "invoice_clients_select" on invoice_clients for select to authenticated using (has_permission(auth.uid(), 'invoices', 'view'));
create policy "invoice_clients_insert" on invoice_clients for insert to authenticated with check (has_permission(auth.uid(), 'invoices', 'create'));
create policy "invoice_clients_update" on invoice_clients for update to authenticated using (has_permission(auth.uid(), 'invoices', 'edit')) with check (has_permission(auth.uid(), 'invoices', 'edit'));
create policy "invoice_clients_delete" on invoice_clients for delete to authenticated using (has_permission(auth.uid(), 'invoices', 'delete'));

notify pgrst, 'reload schema';
```

---

#### MIGRATION 008 — Creator Termination & Gender Fields

```sql
alter table creators add column if not exists terminated boolean default false;
alter table creators add column if not exists terminated_month text;
alter table creators add column if not exists gender text;
notify pgrst, 'reload schema';
```

---

#### MIGRATION 009 — Staff/Expense Link

```sql
alter table expenses add column if not exists staff_member_id text;
notify pgrst, 'reload schema';
```

---

#### MIGRATION 010 — Expense Date Column

```sql
alter table expenses add column if not exists expense_date text;
notify pgrst, 'reload schema';
```

---

#### MIGRATION 011 — Income Date Column

```sql
alter table income_entries add column if not exists income_date text;
notify pgrst, 'reload schema';
```

---

#### MIGRATION 012 — Auto-Create User Profile on Invite/Signup

```sql
create or replace function public.handle_new_user()
returns trigger language plpgsql security definer set search_path = public
as $$
begin
  insert into public.user_profiles (id, email, full_name, role)
  values (new.id, new.email, coalesce(new.raw_user_meta_data->>'full_name', ''), 'guest')
  on conflict (id) do nothing;
  return new;
end;
$$;

drop trigger if exists on_auth_user_created on auth.users;
create trigger on_auth_user_created
  after insert on auth.users
  for each row execute procedure public.handle_new_user();

insert into public.user_profiles (id, email, full_name, role)
select u.id, u.email, coalesce(u.raw_user_meta_data->>'full_name', ''), 'guest'
from auth.users u
left join public.user_profiles p on p.id = u.id
where p.id is null
on conflict (id) do nothing;

notify pgrst, 'reload schema';
```

---

#### MIGRATION 013 — Contracts Table

```sql
create table if not exists contracts (
  id               uuid default gen_random_uuid() primary key,
  client_name      text not null,
  client_email     text,
  client_address   text,
  commission       text,
  tiers_text       text,
  extra_clauses    text,
  start_date       text,
  signing_date     text,
  location         text,
  status           text default 'draft',
  contractor_signature text,
  created_at       timestamptz default now()
);

alter table contracts enable row level security;

create policy "Authenticated users full access on contracts"
  on contracts for all
  using (auth.uid() is not null)
  with check (auth.uid() is not null);

notify pgrst, 'reload schema';
```

✅ **After running all 10 migrations in order**, your database is fully set up.

---

### Step 3: Create Your Admin User in Supabase

1. In your Supabase project, click **Authentication** in the left sidebar
2. Click **Users** → **Invite user**
3. Enter your email address and click **Send invite**
4. Check your email and click the invite link
5. Set a password when prompted
6. Go back to Supabase → **Authentication → Users** — you should see your user listed
7. Now go to **Table Editor** → **user_profiles** table
8. Find your user and change their **role** from `guest` to `admin`
9. Click **Save**

You are now the admin user.

---

### Step 4: Upload Your Code to GitHub

1. Go to your GitHub repository
2. Click **Add file** → **Upload files**
3. Drag and drop your customized `index.html` file (from PART 3 or PART 4)
4. Also upload the `assets/` folder if it exists (it contains the logo)
5. Write a commit message like "Initial upload"
6. Click **Commit changes**

---

### Step 5: Deploy to Vercel (Complete Beginner Guide)

**Vercel** hosts your app and auto-deploys whenever you push changes to GitHub.

1. Go to **https://vercel.com** and click **Sign Up**
2. Choose **Continue with GitHub** — this links your Vercel and GitHub accounts
3. Authorize Vercel to access your GitHub
4. Click **Add New** → **Project**
5. Find your repository in the list and click **Import**
6. Leave all settings as default (Framework: Other, Root directory: ./)
7. Click **Deploy**
8. Wait 1–2 minutes
9. Vercel gives you a URL like `https://agency-finance-tool-xyz.vercel.app` — **this is your app!**

**Custom Domain (optional):**
1. In your Vercel project, click **Domains**
2. Add your own domain (e.g., `finance.youragency.com`)
3. Follow the DNS instructions Vercel provides

**Auto-deployment:** Every time you update `index.html` in GitHub, Vercel automatically redeploys within 1–2 minutes. No manual steps needed.

---

## PART 3: Applying the Source Code

The source code for this app is a single HTML file (`index.html`). It contains all HTML, CSS, and JavaScript in one file — no build process needed.

To configure it for your agency, you need to make these specific replacements in `index.html`:

### Required Replacements

| What to find | What to replace with |
|---|---|
| `YOUR_SUPABASE_URL` | Your Supabase Project URL (from Step 2.3) |
| `YOUR_SUPABASE_ANON_KEY` | Your Supabase anon/public key (from Step 2.3) |
| `[YOUR_PAYER_1]` | First payer name (e.g., the name of a partner/employee who pays expenses) |
| `[YOUR_PAYER_2]` | Second payer name |
| `[YOUR_ENTITY_NAME]` | Your legal entity name (e.g., "ABC Management LLC") |
| `[YOUR_ADDRESS_1]` | Address line 1 |
| `[YOUR_ADDRESS_2]` | Address line 2 (suite, etc.) |
| `[YOUR_CITY_STATE_ZIP]` | City, State, ZIP |
| `[YOUR_PHONE]` | Phone number |
| `[YOUR_REP_1] & [YOUR_REP_2]` | Names of your representatives |
| `[YOUR_ACH_ROUTING]` | ACH routing number |
| `[YOUR_ACCOUNT_NUMBER]` | Bank account number |
| `[YOUR_BANK_ADDRESS_LINE_1]` | Bank address line 1 |
| `[YOUR_BANK_ADDRESS_LINE_2]` | Bank address line 2 |
| `[YOUR_BANK_COUNTRY]` | Bank country |

**Or:** Use the AI Build Prompt in PART 4. The AI will ask for all these values and apply them automatically.

---

## PART 4: AI Build Prompt

**How to use this prompt:**
1. Copy everything between `=== START OF AI PROMPT ===` and `=== END OF AI PROMPT ===`
2. Paste it into your AI assistant (Claude, ChatGPT, etc.)
3. Also paste or upload the **`AGENCY_FINANCE_SOURCE_TEMPLATE.html`** file — the AI needs this as the base source code to customize
4. Answer the AI's questions and it will return your complete customized `index.html`

---

=== START OF AI PROMPT ===

# Agency Finance Tool — Build & Customization Prompt

You are helping set up the **Agency Finance Tool by Safari Studios** — a single-file finance dashboard for talent/creator agencies. I am providing you with:
- This prompt (the instructions + questions)
- The file `AGENCY_FINANCE_SOURCE_TEMPLATE.html` — the complete source code with `[YOUR_X]` placeholders

Your job is to:

1. Ask me the clarifying questions below
2. Take my answers and replace every `[YOUR_X]` placeholder in the source template with the real values I provide
3. Also apply the dynamic code improvements listed in the Key Code Sections (these turn hardcoded names into values read from Settings)
4. Give me the complete, customized `index.html` file ready to deploy
5. Self-verify all changes before returning the file

## Step 1: Ask Me These Questions First

Before writing any code, ask me ALL of the following questions. Wait for my answers before proceeding.

> **For every question:** I can answer with the real value, or type **"skip"** to leave a placeholder that I'll fill in later directly in the Settings page inside the app. Skipped values will remain as `[YOUR_X]` in the code — I can update them anytime after deployment without touching the file.

---

**QUESTION 1 — Operating System**
Are you on Windows or Mac? This affects which copy/paste instructions I give you for GitHub and Vercel.
*(Can't be skipped — needed for setup instructions)*

**QUESTION 2 — Company Name**
What is your company/agency name? (Appears in the app header and auth screen. Example: "Skyline Talent Agency")
*→ Skip option: type "skip" — will show "[YOUR_COMPANY_NAME]" on the login screen until updated in Settings*

**QUESTION 3 — Legal Entity Name**
What is your legal entity name (LLC, GmbH, Ltd, etc.)? Used on invoices and contracts. (Example: "Skyline Management LLC")
*→ Skip option: type "skip" — will show "[YOUR_ENTITY_NAME]" on PDFs until you fill it in under Settings → Company Profile*

**QUESTION 4 — Address**
Business address:
- Address line 1 (street + number)
- Address line 2 (suite, floor — optional)
- City, State/Country, ZIP/Postal code

*→ Skip option: type "skip" — address will show as placeholder on invoice/contract PDFs until updated in Settings → Company Profile*

**QUESTION 5 — Phone Number**
Business phone number?
*→ Skip option: type "skip" — can be added later in Settings → Company Profile*

**QUESTION 6 — Representatives**
Names of representatives/signatories on contracts and invoices? (Example: "Jane Smith & John Doe")
*→ Skip option: type "skip" — will show "[YOUR_REP_1] & [YOUR_REP_2]" on PDFs until updated in Settings → Company Profile*

**QUESTION 7 — Bank Details for Invoices and Contracts**
Banking details for receiving payments:
- Account holder name
- ACH/Wire routing number (or IBAN/BIC for Europe)
- Account number
- Account type (Checking/Savings)
- Bank name and address line 1
- Bank address line 2
- Country

*→ Skip option: type "skip" — bank details will show as placeholders on invoice and contract PDFs until added in Settings → Company Profile*

**QUESTION 8 — Expense Payer Names**
Two names for the people who pay business expenses (shown as colored pills on each expense row). Third option is always "Open" for unassigned expenses. (Example: "Alice" and "Bob")
*→ Skip option: type "skip" — will show "[YOUR_PAYER_1]" and "[YOUR_PAYER_2]" as payer options*

**QUESTION 9 — Features**
Which features do you want?
- **Option A:** All features (full build — recommended)
- **Option B:** Tell me which modules to remove (e.g. "remove Contracts and Archive")
- **Option C:** Describe what you need in plain language — I'll decide which modules to keep or remove based on your description

  *Example sentences for Option C:*
  - "I just need to track my expenses and income, nothing else."
  - "I manage 5 creators on OnlyFans and want to see their revenue and my costs per month."
  - "I need invoices and contracts for clients, but I don't have a team so no Staff tab."
  - "I run a music agency, I need income/expense tracking and a way to send invoices."

*→ Skip option: type "skip" — defaults to Option A (full build)*

**QUESTION 10 — Contract Template**
Do you already have your own contract text that you want to use in the app?

- **Yes:** Paste your contract text here (plain text or Word copy-paste is fine) — I will build the contract PDF around your exact wording
- **No:** The app ships with a default service agreement template. You can edit contracts field by field inside the app, or skip this and update it later

*→ Skip option: type "skip" — the default contract template stays in place; you can replace it later by providing your text*

**QUESTION 11 — Logo**
Logo filename for the watermark on contracts and invoice PDFs? (Default: `safari-studios-logo.png`)
*→ Skip option: type "skip" — uses the default filename; replace the file in the `assets/` folder anytime*

---

## Step 2: After Getting My Answers

Once I answer all questions, do the following:

1. Take the complete source code template below
2. Apply all my answers as replacements for the `[PLACEHOLDER]` values
3. If I chose **Option C** for features: read my description, decide which modules to keep, briefly tell me what you're removing and why, then apply the changes
4. Return the complete, ready-to-deploy `index.html`
5. After each major section, verify the changes are correct

## Step 3: Self-Check After Each Section

After applying changes to each section, confirm:
- ✅ All `[YOUR_X]` placeholders have been replaced with real values
- ✅ No private data from the template remains (no "Kaneki", "Tan", "Furkan", etc.)
- ✅ The JavaScript syntax is valid (no missing quotes, brackets, or commas)
- ✅ The Supabase URL and key have been inserted correctly
- ✅ The payer names appear consistently throughout the file

---

## Complete Source Code Template

The following is the complete, ready-to-customize source code. Apply the replacements from my answers.

**IMPORTANT NOTES for the AI:**
- The branding "Agency Finance Tool by Safari Studios" must remain in the footer and visible in the app — this is the tool creator's branding and cannot be removed
- Replace only the `[YOUR_X]` placeholders and the `YOUR_SUPABASE_URL` / `YOUR_SUPABASE_ANON_KEY` values
- Do NOT change any JavaScript logic, CSS, or HTML structure
- The `EXPENSE_PAYERS` array uses index-based color coding: index 1 = purple pill, index 2 = blue pill, Bank = green, Open = amber
- The contract PDF and invoice PDF read company info dynamically from `getInvoiceCompanySettings()` — so the Settings → Company Profile section in the app controls the company details shown on PDFs

---

### Key Code Sections to Customize

Below are the EXACT sections of the source file that contain placeholders. Apply the user's answers to these sections:

#### Section A — Page Title (near line 6)
```html
<title>Agency Finance Tool by Safari Studios</title>
```
→ Keep exactly as shown. Do NOT change this.

#### Section B — Auth Screen (near lines 491–502)
```html
<div class="auth-brand">Agency Finance Tool by Safari Studios</div>
<div class="auth-version">Finance Tracker</div>
<div class="auth-title" id="authTitle">Sign in</div>
<div class="auth-copy" id="authCopy">Use your [YOUR_COMPANY_NAME] account to access the finance dashboard.</div>
```
→ Replace `[YOUR_COMPANY_NAME]` with the company name from Question 2.

#### Section C — Sidebar Brand (near lines 520–525)
```html
<div class="brand">Agency Finance Tool</div>
...
<div class="sub">Finance Tracker</div>
```
→ Keep exactly as shown.

#### Section D — Supabase Credentials (near lines 1128–1129)
```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_KEY = 'YOUR_SUPABASE_ANON_KEY';
```
→ Replace with values from Supabase Project Settings → API.

#### Section E — Expense Payers (near line 1621)
```javascript
const EXPENSE_PAYERS = ['Bank','[YOUR_PAYER_1]','[YOUR_PAYER_2]','Open'];
```
→ Replace `[YOUR_PAYER_1]` and `[YOUR_PAYER_2]` with names from Question 8.

#### Section F — Expense Payer Pill Function (near lines 1627–1631)
```javascript
function expensePayerPill(value) {
  const v = normalizeExpensePayer(value);
  if (v === 'Open') return `<span class="pill pill-amber">! Open</span>`;
  const idx = EXPENSE_PAYERS.indexOf(v);
  const cls = idx === 1 ? 'pill-purple' : idx === 2 ? 'pill-blue' : 'pill-green';
  return `<span class="pill ${cls}">${v}</span>`;
}
```
→ Keep exactly as shown (it's already dynamic based on array index).

#### Section G — Expense Payer Breakdown (near lines 2023–2035)
```javascript
function getExpensePayerBreakdownForMonths(months) {
  const selected = state.selectedCreators;
  const breakdown = {};
  EXPENSE_PAYERS.forEach(p => { breakdown[p] = 0; });
  state.expenses.forEach(e => {
    if (selected && selected.length && e.creator_id && !selected.includes(e.creator_id)) return;
    const amount = toUSDMulti(parseFloat(e.amount)||0, e.currency||'USD');
    let appliedMonths = 0;
    months.forEach(mk => { if (entryAppliesToMonth(e, mk)) appliedMonths++; });
    if (!appliedMonths) return;
    const payer = normalizeExpensePayer(e.paid_by);
    breakdown[payer] = (breakdown[payer] || 0) + amount * appliedMonths;
  });
  return breakdown;
}
```
→ Keep exactly as shown (already dynamic).

#### Section H — Invoice Company & Payment Instructions (near lines 3835–3854)
```javascript
const INVOICE_COMPANY = {
  entityLine: 'represented by [YOUR_ENTITY_NAME]',
  name: '[YOUR_ENTITY_NAME]',
  addressLines: [
    '[YOUR_ADDRESS_1]',
    '[YOUR_ADDRESS_2]',
    '[YOUR_CITY_STATE_ZIP]'
  ],
  phone: '[YOUR_PHONE]',
  representatives: '[YOUR_REP_1] & [YOUR_REP_2]'
};

const INVOICE_PAYMENT_INSTRUCTIONS = [
  ['Account holder', '[YOUR_ENTITY_NAME]'],
  ['ACH and Wire routing number', '[YOUR_ACH_ROUTING]'],
  ['Account number', '[YOUR_ACCOUNT_NUMBER]'],
  ['Account type', 'Checking'],
  ['Bank address', '[YOUR_BANK_ADDRESS_LINE_1]'],
  ['', '[YOUR_BANK_ADDRESS_LINE_2]'],
  ['', '[YOUR_BANK_COUNTRY]']
];
```
→ Replace ALL `[YOUR_X]` placeholders with values from Questions 3–7.

#### Section I — Invoice Email Functions (near lines 4104–4105)
```javascript
function invoiceEmailSubject(inv){
  const cfg = getInvoiceCompanySettings();
  return `Invoice ${inv.invoice_number || ''} from ${cfg.name || '[YOUR_COMPANY_NAME]'}`.trim();
}
function invoiceEmailBody(inv){
  const cfg = getInvoiceCompanySettings();
  return `Hi ${inv.client_name || ''},\n\nplease find attached invoice ${inv.invoice_number || ''} from ${cfg.name || '[YOUR_COMPANY_NAME]'}.\n\nTotal: ${fmt(parseFloat(inv.total)||0)} ${inv.currency||'USD'}\nDue date: ${inv.due_date || '-'}\n\nBest regards,\n${cfg.name || '[YOUR_COMPANY_NAME]'}`;
}
```
→ Keep as shown (reads from company settings dynamically).

#### Section J — Contract Email (near lines 4875–4882)
```javascript
function sendContractEmail(id) {
  const c = (state.contracts || []).find(x => x.id === id);
  if (!c?.client_email) return;
  const cfg = getInvoiceCompanySettings();
  const entityName = cfg.name || '[YOUR_ENTITY_NAME]';
  const repName = cfg.representatives?.split('&')[0]?.trim() || cfg.representatives || '[YOUR_NAME]';
  const subject = encodeURIComponent('Service Agreement – ' + entityName);
  const body = encodeURIComponent(`Dear ${c.client_name},\n\nPlease find attached your Service Agreement with ${entityName}.\n\nKindly review, sign, and return a copy at your earliest convenience.\n\nBest regards,\n${repName}\n${entityName}`);
  window.open(`mailto:${c.client_email}?subject=${subject}&body=${body}`, '_blank');
  if (c.status === 'draft') cycleContractStatus(id);
}
```
→ Keep as shown (reads from company settings dynamically).

#### Section K — Contract Print — Contractor Party (near line 4943 in the HTML string)
```javascript
// At the top of openContractPrint(), add these lines:
const cfg = getInvoiceCompanySettings();
const contractorEntity = cfg.name || '[YOUR_ENTITY_NAME]';
const contractorReps = cfg.representatives || '[YOUR_REPRESENTATIVES]';
const contractorAddr1 = cfg.addressLine1 || '[YOUR_ADDRESS_1]';
const contractorAddr2 = cfg.addressLine2 ? `${cfg.addressLine2}<br>` : '';
const contractorCityStateZip = cfg.cityStateZip || '[YOUR_CITY_STATE_ZIP]';

// In the HTML string, replace the contractor party div:
// FROM:
<div class="parties"><b>Kaneki Consulting LLC</b><br>represented by Furkan Kocak &amp; Tan Phi Nguyen<br>2880 W OAKLAND PARK BLVD, SUITE 225C<br>OAKLAND PARK, FL 33311<br>– Following, referred to as the <b>"Contractor"</b> –</div>
// TO:
<div class="parties"><b>${h(contractorEntity)}</b><br>represented by ${h(contractorReps)}<br>${h(contractorAddr1)}<br>${contractorAddr2}${h(contractorCityStateZip)}<br>– Following, referred to as the <b>"Contractor"</b> –</div>
```

#### Section L — Contract PDF Bank Details (§5.x, near line 4911)
```javascript
// After extracting cfg at the top of openContractPrint(), also extract:
const payInstr = INVOICE_PAYMENT_INSTRUCTIONS;
const bankHolder = payInstr[0]?.[1] || '[YOUR_ENTITY_NAME]';
const bankACH = payInstr[1]?.[1] || '[YOUR_ACH_ROUTING]';
const bankAcct = payInstr[2]?.[1] || '[YOUR_ACCOUNT_NUMBER]';

// Replace the bank details block in p5 from:
<div style="margin:8px 0 12px 40px;font-weight:600;line-height:1.9;">Kaneki Consulting LLC<br>ACH and wire routing number: 026073150<br>Account number: 822000706135<br>Account type: Checking</div>
// TO:
<div style="margin:8px 0 12px 40px;font-weight:600;line-height:1.9;">${h(bankHolder)}<br>ACH and wire routing number: ${h(bankACH)}<br>Account number: ${h(bankAcct)}<br>Account type: Checking</div>
```

#### Section M — Contract PDF Signature Block (near lines 5052–5053)
```html
<!-- FROM: -->
<strong>Contractor</strong><br>
Kaneki Consulting LLC<br>
Furkan Kocak &amp; Tan Phi Nguyen
<!-- TO: -->
<strong>Contractor</strong><br>
${h(contractorEntity)}<br>
${h(contractorReps)}
```

#### Section N — Monthly Summary Print (near lines 2349 and 2417)
```javascript
// At the top of openMonthlySummaryPrint(), add:
const cfg = getInvoiceCompanySettings();
const companyName = cfg.name || 'Agency Finance Tool';

// Then in the HTML string:
// Line 2349: change from "Safari Studios — Finance Report" to "${companyName} — Finance Report"
// Line 2417: change from "Safari Studios Finance Tool" to "${companyName}"
```

#### Section O — Auth Login Copy Text (near line 1341)
```javascript
// In showLogin() function, change:
document.getElementById('authCopy').textContent = 'Use your [YOUR_COMPANY_NAME] account to access the finance dashboard.';
// → Replace [YOUR_COMPANY_NAME] with the company name from Question 2
```

#### Section P — Backup File Name (near line 2263)
```javascript
// Change:
a.download = 'safari-backup-' + new Date().toISOString().slice(0, 10) + '.json';
// TO:
a.download = 'agency-backup-' + new Date().toISOString().slice(0, 10) + '.json';
```

#### Section Q — Settings Description Text (near line 1013)
```html
<!-- Change: -->
<div class="settings-section-copy">Company details used in invoice Preview and PDF/Print. Defaults preserve the current Safari Studios invoice output.</div>
<!-- TO: -->
<div class="settings-section-copy">Company details used in invoice Preview, PDF/Print, and contract PDFs.</div>
```

---

### Self-Verification Checklist for the AI

After applying ALL changes, verify:

- [ ] `SUPABASE_URL` replaced with real Supabase project URL (format: `https://xxxxx.supabase.co`)
- [ ] `SUPABASE_KEY` replaced with real anon key (long JWT string starting with `eyJ`)
- [ ] `[YOUR_PAYER_1]` and `[YOUR_PAYER_2]` both replaced in `EXPENSE_PAYERS` array
- [ ] `INVOICE_COMPANY.entityLine` contains real entity name (no `[YOUR_` prefix)
- [ ] `INVOICE_COMPANY.name` contains real entity name
- [ ] All 3 `addressLines` filled in
- [ ] `INVOICE_COMPANY.phone` filled in
- [ ] `INVOICE_COMPANY.representatives` filled in
- [ ] All 7 rows of `INVOICE_PAYMENT_INSTRUCTIONS` filled in with real banking data
- [ ] `invoiceEmailSubject` uses `cfg.name` (dynamic — no hardcoded name)
- [ ] `invoiceEmailBody` uses `cfg.name` (dynamic — no hardcoded name)
- [ ] `sendContractEmail` uses `cfg.name` / `cfg.representatives` (dynamic)
- [ ] `openContractPrint` contractor party HTML uses `contractorEntity` and `contractorReps` variables
- [ ] `openContractPrint` §5.x bank details use `bankHolder`, `bankACH`, `bankAcct` variables
- [ ] Sig block in contract uses `contractorEntity` and `contractorReps` variables
- [ ] `openMonthlySummaryPrint` uses `companyName` variable in header and footer
- [ ] `showLogin` auth copy text contains the real company name
- [ ] Backup filename changed to `agency-backup-`
- [ ] Settings description updated
- [ ] No occurrences of "Kaneki", "Tan Phi", "Furkan", "826073150", "822000706135", "pcanznhpduspumxgwqyb", or the old Supabase key remain in the file
- [ ] Branding "Agency Finance Tool by Safari Studios" remains in the title, auth screen, and footer

---

### Architecture Overview (So You Understand What You're Building)

**App Structure:** Single HTML file (`index.html`) — all CSS, HTML, and JavaScript in one file. No build step needed. Drop the file into any static host and it works.

**Authentication:** Supabase Auth. Users sign in with email/password. On first load, the app checks for an active session. Admins can invite new users via Supabase dashboard; invited users auto-get a `guest` profile (Migration 012 trigger).

**Database:** Supabase PostgreSQL. 11 tables total. Row-Level Security (RLS) on all tables — only authenticated users with the correct permissions can read/write data.

**Permissions:** Three roles (admin, user, guest). Admins have full access. Other roles get per-module permissions set by the admin in the Users tab.

**State Management:** All data lives in a global `state` object. On startup, `loadAll()` fetches everything from Supabase into state. UI renders from state. Changes write to Supabase then reload state.

**Real-time:** Supabase real-time channel (`safari-realtime`) broadcasts changes so multiple users see updates without refreshing.

**Charts:** Chart.js (CDN). Three chart types: doughnut (expense payer breakdown), bar (income vs. expenses per month), line (analysis).

**Modules (sidebar tabs):**
- `dashboard` — KPI cards, creator breakdown, charts, month picker
- `creators` — creator CRUD with commission splits and termination
- `income` — income entry CRUD with date, source, creator, currency
- `expenses` — expense entry CRUD with payer, category, staff link
- `staff` — staff member CRUD with assignments and payment tracking
- `invoices` — invoice CRUD with PDF preview and email
- `contracts` — service agreement CRUD with signature canvas, PDF, email
- `analysis` — charts and breakdowns for analysis
- `archive` — month snapshots, closed months
- `settings` — categories, company profile, exchange rates, users
- `users` — role and permission management
- `activitylog` — action history (localStorage)

**Key Functions to Know:**
- `calcMonth(monthKey, creators)` — calculates all KPIs for a month
- `calcStaffCosts(monthKey)` — calculates staff costs (never double-counted with expenses)
- `getInvoiceCompanySettings()` — reads company profile from localStorage
- `openInvoicePreview(inv)` — renders full invoice HTML for preview/print
- `openContractPrint(id)` — renders full contract PDF HTML
- `exportBackup()` — downloads all Supabase data as JSON
- `loadAll()` — fetches all Supabase data into state
- `renderAll()` — re-renders current page

---

=== END OF AI PROMPT ===

---

## PART 5: First-Login Setup (In-App Steps)

After deploying, do these steps in the app itself:

1. **Sign in** with your admin email and password
2. Go to **Settings** (gear icon in the sidebar)
3. Click **Company Profile**
4. Fill in your company details (name, address, representatives, phone)
5. Fill in your bank details (account holder, ACH routing, account number, bank address)
6. Click **Save company profile**
7. Go to **Settings → Income & Expense Categories** and add/remove categories as needed
8. Go to **Settings → Invoice Settings** and add invoice line item categories
9. Go to **Users** tab and invite your team members (they get an email with a login link)
10. For each new team member, set their role and permissions in the Users tab

---

## PART 6: Verification Checklist

Run through this checklist before going live:

### Database
- [ ] All 10 migrations ran successfully ("Success. No rows returned." for each)
- [ ] Supabase → Table Editor shows tables: creators, monthly_data, month_snapshots, notifications, expenses, income_entries, staff_members, staff_assignments, staff_payments, invoices, invoice_clients, user_profiles, user_permissions, contracts
- [ ] Your admin user appears in user_profiles with role = 'admin'

### App — Authentication
- [ ] App loads at your Vercel URL without errors
- [ ] Login page shows your company name in the auth screen
- [ ] Signing in with your admin email and password works
- [ ] The sidebar appears after login

### App — Core Features
- [ ] Dashboard loads with "This month" period selector
- [ ] Can add a creator (Creators tab → + Add Creator)
- [ ] Can add income entry (Income tab → + Add Income)
- [ ] Can add expense entry (Expenses tab → + Add Expense)
- [ ] Expense payer pills show your custom payer names (not "Tan"/"Furkan")
- [ ] Dashboard KPI cards update when income/expense is added

### App — Invoices
- [ ] Can create an invoice (Invoices tab → + New Invoice)
- [ ] Invoice preview shows YOUR company name (not "Kaneki Consulting")
- [ ] Invoice preview shows YOUR address and banking details
- [ ] PDF/Print opens in a new window

### App — Contracts
- [ ] Contracts tab visible in sidebar
- [ ] Can create a contract
- [ ] Contract PDF shows YOUR entity name in the contractor party block
- [ ] Contract PDF §5.x bank details show YOUR banking info
- [ ] Signature block shows YOUR entity name

### App — Settings
- [ ] Company Profile shows YOUR pre-filled details
- [ ] "Save company profile" button works

### App — Users
- [ ] Users tab shows your admin account
- [ ] Can invite a new user (they receive an email)
- [ ] New user appears in Users tab after accepting invite

---

## PART 7: Troubleshooting

### "Supabase RLS blocked this action"
The permissions system blocked the request. Check:
- Your user has role = 'admin' in the user_profiles table
- Migration 007 ran successfully

### "Schema cache" error about `paid_by`
Migration 0 (core tables) did not run or did not apply the `paid_by` column. Re-run Migration 0 in Supabase SQL Editor.

### Invited user can't log in / doesn't appear in Users tab
- Check Migration 012 ran successfully (the auto-profile trigger)
- If they accepted the invite, check Supabase → Authentication → Users to confirm the account exists
- If user_profiles row is missing, run this SQL manually:
  ```sql
  insert into user_profiles (id, email, full_name, role)
  select id, email, '', 'guest' from auth.users
  where id not in (select id from user_profiles)
  on conflict do nothing;
  ```

### Invoices/Contracts don't show my company name
- Go to Settings → Company Profile and fill in all fields
- Click "Save company profile"
- The PDFs read from localStorage — data saved in Settings applies immediately

### App doesn't load (blank white page)
- Open browser console (F12) and check for errors
- Most common: Supabase URL or key is wrong. Double-check the values in `index.html` lines 1128–1129

### Real-time updates not working
- Check Supabase → Database → Replication and confirm the `invoices` and `invoice_clients` tables are enabled for realtime
- If tables are missing from replication, run:
  ```sql
  alter publication supabase_realtime add table invoices;
  alter publication supabase_realtime add table invoice_clients;
  ```

---

## Notes on Branding

The label **"Agency Finance Tool by Safari Studios"** remains in the app footer and title. This identifies the tool builder and cannot be removed. Your agency's name appears in:
- The auth screen login copy ("Use your [Company] account to access...")
- Invoice PDFs (company name, address, representatives)
- Contract PDFs (contractor party block, bank details, signature block)
- Email subjects and bodies (invoice and contract emails)
- Monthly summary PDF header

All of these are controlled by the **Settings → Company Profile** section inside the app. Update them there and all PDFs and emails update automatically.

---

*Agency Finance Tool by Safari Studios — v13.9*  
*For questions or support, contact the Safari Studios team.*
