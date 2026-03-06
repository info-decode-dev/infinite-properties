# Environment Variables for Deployment

## Backend Environment Variables

Add these to your backend hosting platform (Railway, Render, etc.):

```env
PORT=5000
NODE_ENV=production

# Supabase Database Connection
DATABASE_URL=postgresql://postgres:[YOUR-PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres?pgbouncer=true&connection_limit=1

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
JWT_EXPIRE=7d

# CORS - Update with your frontend URL
CORS_ORIGIN=https://your-frontend-domain.vercel.app

# Supabase Storage Configuration
SUPABASE_URL=https://your-project-id.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here
SUPABASE_STORAGE_BUCKET=uploads

# Optional: Local file storage (if not using Supabase Storage)
UPLOAD_DIR=./public/uploads
MAX_FILE_SIZE=10485760
```

## Frontend Environment Variables

Add these to your frontend hosting platform (Vercel, Netlify, etc.):

```env
NEXT_PUBLIC_API_URL=https://your-backend-url.railway.app
```

## How to Get Supabase Credentials

1. **Database URL**:
   - Go to Supabase Dashboard → Project Settings → Database
   - Copy the "Connection string" → "URI"
   - Replace `[YOUR-PASSWORD]` with your database password
   - Replace `[PROJECT-REF]` with your project reference

2. **Supabase URL**:
   - Go to Supabase Dashboard → Project Settings → API
   - Copy the "Project URL"

3. **Service Role Key**:
   - Go to Supabase Dashboard → Project Settings → API
   - Copy the "service_role" key (keep this secret!)

## Important Notes

- Never commit `.env` files to git
- Use environment variables in your hosting platform's dashboard
- Update `CORS_ORIGIN` to match your frontend URL
- The `DATABASE_URL` for Supabase should include `?pgbouncer=true&connection_limit=1` for connection pooling
