# Employee Check-In Rewards

A production-oriented starter for a daily employee check-in and rewards system.

## Architecture

- **Frontend:** plain HTML/CSS/JavaScript
- **Hosting:** GitHub Pages
- **Backend:** Supabase Auth + PostgreSQL
- **Security:** PostgreSQL Row Level Security (RLS)
- **Daily rule:** one check-in per employee per calendar day
- **Reward:** 1 credit per successful daily check-in
- **Roles:** `employee` and `admin`

## 1. Create the Supabase project

Create a Supabase project at https://supabase.com/.

Open **SQL Editor** and run:

`supabase/schema.sql`

The SQL creates:
- `profiles`
- `checkins`
- automatic profile creation after signup
- atomic `record_daily_checkin()` RPC
- RLS policies
- admin access helper

## 2. Configure authentication

In Supabase, open **Authentication → Providers → Email** and enable Email.

For a simple deployment, you can use email/password authentication.

If email confirmation is enabled, employees must confirm their email before signing in.

## 3. Configure the frontend

Open `config.js` and replace:

```js
window.APP_CONFIG = {
  SUPABASE_URL: "https://YOUR-PROJECT.supabase.co",
  SUPABASE_ANON_KEY: "YOUR-PUBLISHABLE-OR-ANON-KEY"
};
```

Use the Supabase **publishable/anon client key**, never the `service_role`/secret key.

The client key is designed to be used by browser applications when RLS is configured correctly.

## 4. Create the first admin

1. Open the website.
2. Create an account using the intended admin email.
3. Confirm the email if confirmation is enabled.
4. Run this in Supabase SQL Editor:

```sql
update public.profiles
set role = 'admin'
where email = 'YOUR-ADMIN-EMAIL@example.com';
```

Log out and back in. The Admin Analytics dashboard will appear.

## 5. Deploy to GitHub Pages

Put these files in a GitHub repository:

- `index.html`
- `styles.css`
- `app.js`
- `config.js`
- `supabase/schema.sql`
- `README.md`

Then:

1. Push the repository to GitHub.
2. Open **Settings → Pages**.
3. Choose **Deploy from a branch**.
4. Select your main branch and `/ (root)`.
5. Save.
6. Wait for GitHub Pages to publish the site.

## Security model

The frontend does **not** directly modify credits.

When an employee checks in:

1. The browser calls `record_daily_checkin()`.
2. PostgreSQL verifies the authenticated user matches the employee ID.
3. PostgreSQL inserts today's check-in.
4. The unique `(employee_id, checkin_date)` constraint prevents duplicates.
5. PostgreSQL increments the employee's credits.
6. The result is returned to the browser.

Employees can read only their own profile/check-ins.

Admins can read employee profiles and check-ins.

There are deliberately no client-side insert/update/delete policies for check-ins.

## Changing the reward amount

The current reward is 1 credit per daily check-in.

To change it, edit the value in:

`supabase/schema.sql`

inside:

```sql
insert into public.checkins (..., credits_awarded)
values (..., 1)
```

If you need more complex rewards later, the RPC can be extended for:
- weekly bonuses
- streak bonuses
- late/early check-in rules
- department-specific rewards
- attendance milestones
- monthly leaderboards

## Time zone

`current_date` uses the PostgreSQL database/session timezone. If your organization must strictly operate on a specific local calendar (for example, Africa/Nairobi), set the database/session timezone strategy explicitly before production use.

## Production recommendations

Before a large deployment, add:
- password reset UI
- stronger admin management
- audit logs
- configurable reward rules
- CSV export
- pagination for large datasets
- server-side analytics views
- organization/department tenancy if multiple companies will use the same instance
- rate limiting and abuse monitoring
- a custom domain with HTTPS

## License

Use and modify this starter as needed.
