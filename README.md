# Where Is Zorda? 📍

A live sightings board. The feed is public (anyone with the link can see where Zorda is); only a logged-in admin (Chris) can post a selfie + location.

- **Hosting:** Netlify (static, auto-deploys from this repo's `main` branch)
- **Backend:** Supabase (database + photo storage + login) — free tier
- **No build step:** it's a single `index.html`. Edit it right in the GitHub mobile app.

It runs in **demo mode** with sample posts until you add your Supabase keys.

---

## 1. Create a Supabase project (one time, ~3 min)

1. Go to supabase.com, sign in, **New project**. Pick a name + password, any region.
2. When it's ready, open **Project Settings → API** and copy two things:
   - **Project URL**
   - **anon public** key

## 2. Create the database table

In Supabase, open **SQL Editor**, paste this, and run it:

```sql
create table posts (
  id bigint generated always as identity primary key,
  created_at timestamptz default now(),
  location text not null,
  message text,
  image_url text not null
);

alter table posts enable row level security;

-- anyone can read the feed
create policy "public read" on posts for select using (true);

-- only logged-in users can post
create policy "auth insert" on posts for insert to authenticated with check (true);
```

## 3. Create the photo storage bucket

1. **Storage → New bucket**, name it exactly `selfies`, and tick **Public bucket**.
2. That's it — public bucket means the selfies are viewable by everyone, uploads are restricted to logged-in users by default.

## 4. Create Chris's login

**Authentication → Users → Add user** → enter Chris's email + a password.
(Optional: under Authentication → Providers, turn off "Confirm email" so it works instantly.)
This email + password is what he types into the **Admin login** on the site.

## 5. Plug the keys into the site

Open `index.html`, find the **CONFIG** block near the top, and replace the two placeholders:

```js
window.ZORDA_CONFIG = {
  SUPABASE_URL: "https://xxxx.supabase.co",
  SUPABASE_ANON_KEY: "eyJhbGci...your-anon-key..."
};
```

Save / commit. (The anon key is meant to be public — it's safe in client-side code. Your security comes from the row-level policies above, not from hiding the key.)

## 6. Connect Netlify (one time)

1. netlify.com → **Add new site → Import from Git** → pick this repo.
2. No build command needed; **publish directory = `/`** (root). Deploy.
3. Every push to `main` now auto-deploys.

## 7. Point your GoDaddy domain at Netlify

In Netlify: **Domain settings → Add a domain** → enter your domain → follow its instructions (either switch GoDaddy's nameservers to Netlify's, or add the records Netlify shows). HTTPS is automatic.

---

That's the whole thing. After step 5, the yellow demo banner disappears, the login works, and Chris can post real sightings from his phone.
