# Supabase Hosting Setup Guide

This guide will help you deploy your Infinite Properties website using Supabase for the database and storage, along with hosting platforms for your frontend and backend.

## Architecture Overview

- **Frontend (Next.js)**: Deploy to Vercel (recommended) or Netlify
- **Backend (Express)**: Deploy to Railway, Render, or Fly.io
- **Database**: Supabase PostgreSQL
- **Storage**: Supabase Storage (for file uploads)

## Step 1: Set Up Supabase Project

1. **Create a Supabase Account**
   - Go to [supabase.com](https://supabase.com)
   - Sign up or log in

2. **Create a New Project**
   - Click "New Project"
   - Choose your organization
   - Project name: `infinite-properties`
   - Database password: Create a strong password (save it!)
   - Region: Choose closest to your users
   - Click "Create new project"

3. **Wait for Project Setup** (2-3 minutes)

## Step 2: Get Your Supabase Credentials

1. Go to **Project Settings** → **API**
2. Copy the following:
   - `Project URL`: `https://your-project-id.supabase.co`
   - `anon/public key`: Your public API key
   - `service_role key`: Your service role key (keep secret!)

3. Go to **Project Settings** → **Database**
4. Copy the **Connection String**:
   - Under "Connection string" → "URI"
   - Format: `postgresql://postgres:[YOUR-PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres`

## Step 3: Set Up Supabase Storage

1. Go to **Storage** in your Supabase dashboard
2. Create a new bucket called `uploads`:
   - Click "New bucket"
   - Name: `uploads`
   - Public: ✅ **Yes** (for public access to images)
   - File size limit: 10MB (or your preference)
   - Allowed MIME types: `image/*, video/*`
   - Click "Create bucket"

3. (Optional) Create additional buckets:
   - `profile-pictures` for team member images
   - `testimonials` for testimonial media

## Step 4: Install Dependencies and Run Database Migrations

1. **Install Supabase dependency**:
   ```bash
   cd server
   npm install @supabase/supabase-js
   ```

2. **Update your `.env` file** in the `server` folder with Supabase connection string:
   ```env
   DATABASE_URL=postgresql://postgres:[YOUR-PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres?pgbouncer=true&connection_limit=1
   SUPABASE_URL=https://your-project-id.supabase.co
   SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
   SUPABASE_STORAGE_BUCKET=uploads
   ```

3. **Run Prisma migrations**:
   ```bash
   cd server
   npm install
   npx prisma migrate deploy
   npx prisma generate
   ```

3. **Seed the database** (optional):
   ```bash
   npm run prisma:seed
   ```

## Step 5: Update Backend for Supabase Storage

The backend is configured to automatically use Supabase Storage when environment variables are set. The upload utility (`server/src/utils/upload.ts`) will use memory storage and upload directly to Supabase.

**Note**: Controllers currently save local file paths. To fully utilize Supabase Storage, you can update controllers to use the `processUploadedFiles` helper from `server/src/utils/fileHandler.ts`. However, the current implementation will work - files will be uploaded to Supabase Storage automatically when Supabase is configured.

**Example controller update** (optional):
```typescript
import { processUploadedFiles } from '../utils/fileHandler';

// In your controller:
const images = await processUploadedFiles(req.files, 'images');
// This returns Supabase URLs if configured, or local paths otherwise
```

## Step 6: Deploy Backend (Express Server)

### Option A: Railway (Recommended)

1. Go to [railway.app](https://railway.app)
2. Sign up/login with GitHub
3. Click "New Project" → "Deploy from GitHub repo"
4. Select your repository
5. Add environment variables:
   ```
   PORT=5000
   NODE_ENV=production
   DATABASE_URL=postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres
   JWT_SECRET=your-super-secret-jwt-key-change-this
   JWT_EXPIRE=7d
   CORS_ORIGIN=https://your-frontend-domain.vercel.app
   SUPABASE_URL=https://your-project-id.supabase.co
   SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
   ```
6. Railway will auto-detect Node.js and deploy
7. Copy the deployment URL (e.g., `https://your-app.railway.app`)

### Option B: Render

1. Go to [render.com](https://render.com)
2. Sign up/login
3. Click "New" → "Web Service"
4. Connect your GitHub repository
5. Settings:
   - Build Command: `cd server && npm install && npm run build`
   - Start Command: `cd server && npm start`
   - Root Directory: `server`
6. Add environment variables (same as Railway)
7. Deploy

## Step 7: Deploy Frontend (Next.js)

### Option A: Vercel (Recommended)

1. Go to [vercel.com](https://vercel.com)
2. Sign up/login with GitHub
3. Click "Add New Project"
4. Import your GitHub repository
5. Configure:
   - Framework Preset: Next.js
   - Root Directory: `client`
   - Build Command: `npm run build`
   - Output Directory: `.next`
6. Add environment variables:
   ```
   NEXT_PUBLIC_API_URL=https://your-backend-url.railway.app
   ```
7. Click "Deploy"

### Option B: Netlify

1. Go to [netlify.com](https://netlify.com)
2. Sign up/login
3. Click "Add new site" → "Import an existing project"
4. Connect GitHub repository
5. Build settings:
   - Base directory: `client`
   - Build command: `npm run build`
   - Publish directory: `client/.next`
6. Add environment variables (same as Vercel)
7. Deploy

## Step 8: Update CORS Settings

1. In your backend deployment, update `CORS_ORIGIN` to your frontend URL
2. In Supabase Dashboard → **Settings** → **API** → **CORS**, add your frontend domain

## Step 9: Test Your Deployment

1. Visit your frontend URL
2. Test API endpoints
3. Test file uploads
4. Check database connection

## Environment Variables Summary

### Backend (.env)
```env
PORT=5000
NODE_ENV=production
DATABASE_URL=postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRE=7d
CORS_ORIGIN=https://your-frontend-domain.vercel.app
SUPABASE_URL=https://your-project-id.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
SUPABASE_STORAGE_BUCKET=uploads
```

### Frontend (.env.local)
```env
NEXT_PUBLIC_API_URL=https://your-backend-url.railway.app
```

## Troubleshooting

### Database Connection Issues
- Ensure connection string includes `?pgbouncer=true&connection_limit=1` for Supabase
- Check if your IP needs to be whitelisted (usually not needed for Supabase)

### Storage Issues
- Verify bucket is set to public
- Check CORS settings in Supabase Storage
- Ensure service role key has correct permissions

### CORS Errors
- Update `CORS_ORIGIN` in backend to match frontend URL
- Add frontend URL to Supabase CORS settings

## Next Steps

- Set up custom domains
- Configure SSL certificates (usually automatic)
- Set up monitoring and logging
- Configure backups in Supabase dashboard
