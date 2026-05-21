# UNIVEN Library Smart Seat Reservation System
### Group 12 — Supabase Edition v2 (Separate Pages + Approval Workflow)

---

## File Structure

```
univen-v2/
├── index.html          ← Login / Register page
├── student.html        ← Student portal (book seats, view requests)
├── admin.html          ← Admin portal (approve/reject, manage zones)
├── css/style.css
├── js/
│   ├── config.js       ← ⚠️  Add your Supabase URL + key here
│   ├── student.js
│   └── admin.js
├── sql/schema.sql      ← Full DB schema (run once in SQL Editor)
└── supabase/functions/notify-booking/index.ts  ← Email notifications
```

---

## Page Flow

```
index.html (login)
    ├── role = student  →  student.html
    └── role = admin    →  admin.html
```

Students cannot access `admin.html` — they are redirected back to `student.html` automatically. Admins who open `student.html` are redirected to `admin.html`.

---

## Setup

### Step 1 — Run the SQL schema
Supabase Dashboard → SQL Editor → paste `sql/schema.sql` → Run

### Step 2 — Add your keys to js/config.js
```js
export const SUPABASE_URL  = 'https://xxxx.supabase.co';
export const SUPABASE_ANON = 'eyJ...';
```

### Step 3 — Serve the app
```bash
python -m http.server 3000
# open http://localhost:3000
```

### Step 4 — Register, then make yourself admin
Register at index.html, then in Supabase → Table Editor → profiles → change your role to 'admin'.

Or run in SQL Editor:
```sql
update public.profiles set role = 'admin' where id = 'YOUR-UUID';
```

---

## Email Notifications (optional)

The app uses a Supabase Edge Function to send emails via Resend (free: 3000 emails/month).

### Setup emails:
1. Create a free account at https://resend.com → get your API key
2. Install Supabase CLI: `npm install -g supabase`
3. Login: `supabase login`
4. Deploy the function:
   ```bash
   supabase functions deploy notify-booking --project-ref YOUR_PROJECT_ID
   ```
5. Set secrets in Supabase Dashboard → Edge Functions → notify-booking → Secrets:
   - `RESEND_API_KEY` = your Resend key
   - `ADMIN_EMAIL` = admin's email address
6. Copy the function URL and paste into `js/config.js`:
   ```js
   export const NOTIFY_FN_URL = 'https://xxxx.supabase.co/functions/v1/notify-booking';
   ```

Without this setup the app works fine — emails are just skipped silently.

---

## How the Approval Flow Works

1. Student opens `student.html` → selects a zone and seat → submits request
2. Booking is saved with `status = 'pending'`
3. Admin gets an email (if configured) and sees the request on `admin.html`
4. Admin checks occupancy and either **Approves** or **Rejects** (with a reason)
5. Student's page updates in real time via Supabase Realtime WebSocket
6. Student gets an email notification (if configured)
7. If approved: seat is now marked as taken for that date
8. If rejected: student can submit a new request for a different seat/date

---

## Troubleshooting

**Stuck on loading / blank page** → Open DevTools (F12) → Console tab for errors

**"Tracking Prevention blocked..."** in Edge → Click the shield icon in the address bar → turn off tracking prevention for localhost

**Roles not redirecting** → Make sure you ran the SQL schema and your profile row has the correct role

**Emails not sending** → Check the Edge Function logs in Supabase Dashboard → Edge Functions → Logs
