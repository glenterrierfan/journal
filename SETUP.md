# Setup Guide — Training Log

This is a dog training journal app built with React, Supabase, and Tailwind CSS. It runs entirely in the browser as a single HTML file.

## Quick Start

1. **Create a Supabase project** at [supabase.com](https://supabase.com)
2. **Copy your project credentials:**
   - Go to Settings → API
   - Copy your Project URL and Anon Public Key
3. **Paste them into `index.html`:**
   ```javascript
   window.SUPABASE_URL = "your-project-url";
   window.SUPABASE_ANON_KEY = "your-anon-key";
   ```
4. **Set up your database** (see Database Setup below)
5. **Open `index.html`** in a browser — that's it

---

## Database Setup

### 1. Create the `planner_state` table

In your Supabase SQL Editor, run:

```sql
CREATE TABLE planner_state (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  data JSONB NOT NULL DEFAULT '{"dogs":[],"plans":[],"goals":[],"sessions":[],"maintenance":[]}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id)
);
```

### 2. Enable Row-Level Security (RLS)

**Enable RLS on the table:**
```sql
ALTER TABLE planner_state ENABLE ROW LEVEL SECURITY;
```

**Create policy — users can only read/write their own data:**
```sql
CREATE POLICY "Users can only access their own planner state"
ON planner_state
FOR ALL
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);
```

**Verify the policy is active:**
- Go to Supabase Dashboard → Authentication → Policies
- You should see the policy listed for `planner_state`
- Status should show "Enabled"

### 3. Configure Authentication

In your Supabase project:

**Settings → Auth → Providers:**
- Email is enabled by default ✓

**Settings → Auth → Email Templates:**
- Confirm Email: enabled (users must verify email before signup completes) — recommended but optional
- Set your site URL under Settings → General → Site URL

---

## Security Notes

⚠️ **The anon key is intentionally public** — it's used for client-side authentication. Supabase secures data via:

1. **Row-Level Security (RLS)** — the database only returns rows where `user_id = auth.uid()`
2. **Auth state** — users can only write if they're signed in
3. **Unique constraint** — each user has exactly one `planner_state` row

**After setup, rotate your keys if they're exposed:**
- Supabase Dashboard → Settings → API → Rotate Keys

**Test RLS is working:**
1. Sign up with `alice@example.com` / `password123`
2. Add a dog and create a session
3. Open a private browser tab, sign up with `bob@example.com` / `password123`
4. Bob should see an empty journal — he can't see Alice's data

---

## File Structure

```
journal/
├── index.html          # Single-file app (React, auth, UI, all logic)
├── SETUP.md            # This file
└── README.md           # Project overview
```

---

## Deployment

### Option 1: GitHub Pages

1. Push `index.html` to your repo
2. Go to Settings → Pages → Deploy from branch (main)
3. Site will be live at `https://glenterrierfan.github.io/journal/`

**Note:** Update your Supabase Site URL to include this address.

### Option 2: Other hosts

- Netlify: Drag and drop `index.html`
- Vercel: Upload as static file
- Any web server: Serve `index.html` as-is

---

## Troubleshooting

### "Couldn't save changes — check your connection"

- Check your Supabase URL and Anon Key are correct
- Verify RLS policies are enabled
- Open browser DevTools → Console for error details

### Sign-up fails or takes forever

- Check Supabase email confirmation settings (Settings → Auth → Email)
- If confirmation is required, check your email for the confirmation link
- In development, you can disable confirmation (not recommended for production)

### "Access denied" when saving

- RLS policy is not active or missing
- Run the RLS commands above
- Verify `user_id` column exists in `planner_state`

---

## Data Structure

The app stores everything in a single `data` JSONB column:

```json
{
  "dogs": [
    { "id": "abc123", "name": "Wanda", "breed": "Border Collie", "accent": "#B5592E", "photoDataUrl": "...", "dob": "2021-03-15", "microchip": "...", "vetName": "...", "vetPhone": "...", "vetAddress": "..." }
  ],
  "plans": [
    { "id": "def456", "dogId": "abc123", "title": "Crate settling", "status": "in_progress", "createdAt": 1234567890 }
  ],
  "goals": [
    { "id": "ghi789", "dogId": "abc123", "planId": "def456", "term": "long", "title": "Settle for 30 mins", "detail": "...", "done": false, "createdAt": 1234567890 }
  ],
  "sessions": [
    { "id": "jkl012", "dogId": "abc123", "date": "2024-08-04", "goalId": "ghi789", "notes": "...", "successCriteria": "...", "videoUrl": "...", "videoNote": "...", "takeaways": "...", "distance": "10m", "duration": "5", "distraction": "Medium", "reps": "12", "reinforcements": "8", "createdAt": 1234567890 }
  ],
  "maintenance": [
    { "id": "mno345", "dogId": "abc123", "planId": "def456", "title": "Crate settling", "createdAt": 1234567890 }
  ]
}
```

Photos are stored as data URLs in the browser (up to 2MB). Videos are links only — point to Google Photos, Drive, or YouTube.

---

## Support

- Supabase docs: https://supabase.com/docs
- React docs: https://react.dev
- Tailwind CSS: https://tailwindcss.com
