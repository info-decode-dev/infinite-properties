# Quick Start Guide - Supabase Deployment

This is a condensed guide to get your website hosted on Supabase quickly.

## Prerequisites Checklist

- [ ] GitHub account
- [ ] Supabase account
- [ ] Vercel account (for frontend)
- [ ] Railway account (for backend) - or Render/Fly.io

## 5-Minute Setup

### 1. Supabase Setup (2 minutes)

1. Create project at [supabase.com](https://supabase.com)
2. Go to **Settings** → **API** → Copy:
   - Project URL
   - Service Role Key
3. Go to **Settings** → **Database** → Copy connection string
4. Go to **Storage** → Create bucket `uploads` (public)

### 2. Install Dependencies (1 minute)

```bash
cd server
npm install @supabase/supabase-js
```

### 3. Deploy Backend (1 minute)

**Railway:**
1. Go to [railway.app](https://railway.app)
2. New Project → Deploy from GitHub
3. Select your repo
4. Add environment variables (see `DEPLOYMENT_ENV_VARS.md`)
5. Deploy

**Get your backend URL** (e.g., `https://your-app.railway.app`)

### 4. Deploy Frontend (1 minute)

**Vercel:**
1. Go to [vercel.com](https://vercel.com)
2. Import GitHub repo
3. Root Directory: `client`
4. Add environment variable:
   ```
   NEXT_PUBLIC_API_URL=https://your-backend-url.railway.app
   ```
5. Deploy

## Done! 🎉

Your website should now be live. Check both URLs and test functionality.

## Need Help?

- Full setup guide: `SUPABASE_SETUP.md`
- Environment variables: `DEPLOYMENT_ENV_VARS.md`
- Migration guide: `MIGRATION_GUIDE.md`
