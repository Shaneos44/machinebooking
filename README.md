# Machine Booking System

Static frontend (deploys to Vercel) + Supabase backend (project **machine-booking**, Sydney).

- **index.html** — public: pick a machine, weekly calendar, click a free slot to book (name, email, start and finish time). After booking, users get a secret cancellation link on screen and by email.
- **cancel.html** — users open their cancellation link here to cancel their own booking.
- **admin.html** — admin login, add/edit/delete machines, view and cancel any booking.
- Double bookings are impossible at the database level, even with simultaneous submissions. Back-to-back bookings are allowed.

## Admin access

Admin = a Supabase auth user whose email is in the `app_admins` table.
`shaneosullivan@vitaltrace.com.au` is on the list. To add another admin, run in the SQL Editor:

```sql
insert into app_admins (email) values ('another@email.com');
```

and create that user under **Authentication → Users**. Recommended: disable public sign-ups under **Authentication → Sign In / Providers → Email**.

## Confirmation emails (one-time setup, ~5 min)

Emails are sent by the `send-confirmation` Edge Function (already deployed). It needs a free [Resend](https://resend.com) account:

1. Sign up at resend.com and create an API key.
2. In the [Supabase dashboard](https://supabase.com/dashboard/project/orulmqcxdviwxtybqqoy) go to **Edge Functions → Secrets** and add:
   - `RESEND_API_KEY` — your Resend key (required)
   - `SITE_URL` — your Vercel URL, e.g. `https://your-app.vercel.app` (recommended, used for the cancel link in emails)
   - `FROM_EMAIL` — e.g. `Bookings <bookings@yourdomain.com>` (optional)

Note: until you verify your own domain in Resend, the default `onboarding@resend.dev` sender can only deliver to your own Resend account email — verify a domain to email everyone.

Until the key is set, the site still works: the cancellation link is always shown on screen after booking, and the page tells the user the email couldn't be sent.

## Deploy to Vercel

1. Go to https://vercel.com/new
2. Drag and drop this folder (or import a Git repo). Framework preset: **Other**, no build command, no env vars.

Or `npx vercel` from inside the folder. The Supabase URL/key in `config.js` are safe to expose — everything is protected by Row Level Security.

## Configuration

`config.js`: `DAY_START_HOUR` / `DAY_END_HOUR` set the calendar's visible hours (default 06:00–22:00).

## Who can do what

| Action | Who |
|---|---|
| View machines and the calendar | Everyone |
| Create a booking (max 12 h, must be upcoming) | Everyone |
| Cancel their own booking | Anyone with the secret cancel link |
| See bookers' email addresses | Admin only |
| Add / edit / delete machines, cancel any booking | Admin only |
