# Migration Guide: Local to Supabase

This guide will help you migrate your existing local setup to Supabase.

## Prerequisites

- Supabase account and project created
- Backend and frontend code updated with Supabase configuration
- Supabase Storage bucket created

## Step 1: Install Dependencies

In the `server` directory:

```bash
cd server
npm install @supabase/supabase-js
```

## Step 2: Set Up Supabase Database

1. **Get your Supabase database connection string**:
   - Go to Supabase Dashboard → Project Settings → Database
   - Copy the "Connection string" → "URI"
   - Format: `postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres`

2. **Update your server `.env` file**:
   ```env
   DATABASE_URL=postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres?pgbouncer=true&connection_limit=1
   ```

3. **Run migrations**:
   ```bash
   cd server
   npx prisma migrate deploy
   npx prisma generate
   ```

4. **Seed the database** (optional):
   ```bash
   npm run prisma:seed
   ```

## Step 3: Set Up Supabase Storage

1. **Create storage buckets** in Supabase Dashboard:
   - Go to Storage → New bucket
   - Create buckets: `uploads` (public), `images`, `videos`, `logos`, `profiles`

2. **Update your server `.env` file**:
   ```env
   SUPABASE_URL=https://your-project-id.supabase.co
   SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
   SUPABASE_STORAGE_BUCKET=uploads
   ```

## Step 4: Migrate Existing Files (Optional)

If you have existing files in `server/public/uploads`, you can migrate them to Supabase Storage:

1. **Manual upload**:
   - Go to Supabase Dashboard → Storage
   - Upload files to appropriate buckets

2. **Or use a migration script** (create `server/scripts/migrate-files.ts`):
   ```typescript
   import { supabase } from '../src/config/supabase';
   import fs from 'fs';
   import path from 'path';

   async function migrateFiles() {
     const uploadDir = './public/uploads';
     const folders = ['images', 'videos', 'logos', 'profiles'];

     for (const folder of folders) {
       const folderPath = path.join(uploadDir, folder);
       if (!fs.existsSync(folderPath)) continue;

       const files = fs.readdirSync(folderPath);
       for (const file of files) {
         const filePath = path.join(folderPath, file);
         const fileBuffer = fs.readFileSync(filePath);
         const { error } = await supabase.storage
           .from('uploads')
           .upload(`${folder}/${file}`, fileBuffer);
         
         if (error) {
           console.error(`Error uploading ${file}:`, error);
         } else {
           console.log(`Uploaded ${file}`);
         }
       }
     }
   }

   migrateFiles();
   ```

## Step 5: Update Controllers (If Needed)

The upload utility now automatically uses Supabase Storage if configured. However, you may need to update controllers to handle Supabase URLs properly.

### Example: Property Controller

The controller should now use Supabase URLs when files are uploaded. The `uploadToSupabase` function returns a public URL that should be stored in the database.

## Step 6: Update Frontend Image URLs

If you're using Supabase Storage, image URLs will be different. Update your frontend to handle both local and Supabase URLs:

```typescript
const getImageUrl = (imagePath?: string) => {
  if (!imagePath) return "";
  // If it's already a full URL (Supabase), return as-is
  if (imagePath.startsWith("http://") || imagePath.startsWith("https://"))
    return imagePath;
  // Otherwise, prepend API URL (for local storage)
  const apiUrl = process.env.NEXT_PUBLIC_API_URL || "http://localhost:5000";
  return `${apiUrl}${imagePath}`;
};
```

## Step 7: Test Your Migration

1. **Test database connection**:
   ```bash
   cd server
   npm run dev
   # Check health endpoint: http://localhost:5000/health
   ```

2. **Test file uploads**:
   - Upload a file through your admin panel
   - Verify it appears in Supabase Storage
   - Check the URL in your database

3. **Test file retrieval**:
   - View uploaded images on your frontend
   - Verify they load correctly

## Step 8: Deploy

Follow the deployment guide in `SUPABASE_SETUP.md` to deploy your application.

## Troubleshooting

### Database Connection Issues
- Ensure connection string includes `?pgbouncer=true&connection_limit=1`
- Check if your IP needs to be whitelisted (usually not needed)

### Storage Upload Issues
- Verify bucket is set to public
- Check service role key has correct permissions
- Ensure file size limits are configured correctly

### URL Issues
- Supabase URLs are public URLs, not relative paths
- Update your frontend to handle full URLs
- Check CORS settings if images don't load

## Rollback Plan

If you need to rollback to local storage:

1. Remove Supabase environment variables from `.env`
2. Restart your server
3. Files will automatically use local storage again
