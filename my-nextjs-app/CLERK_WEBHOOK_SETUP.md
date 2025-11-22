# Clerk Webhook Setup Instructions

## 🎯 Purpose
This guide will help you sync users from Clerk to your database automatically using webhooks.

## ✅ Step 1: Add Webhook Secret to Environment Variables

Add this to your `.env.local` file:

```env
# Clerk Webhook Secret (get this from Clerk Dashboard)
CLERK_WEBHOOK_SECRET=whsec_...
```

## ✅ Step 2: Push Database Schema

The users table schema has been added. Push it to your database:

```bash
npx drizzle-kit push
```

When prompted about column creation:
- Select **"+ firstName create column"** (first option)
- Select **"+ lastName create column"**  
- Select **"+ imageUrl create column"**

Press Enter for each to confirm they are new columns.

## ✅ Step 3: Configure Webhook in Clerk Dashboard

### 3.1 Start Your Dev Server (if not running)
```bash
npm run dev
```

### 3.2 Expose Your Local Server (for development)
You need to expose your local server so Clerk can send webhooks to it. Use one of these tools:

**Option A: Using ngrok (Recommended)**
```bash
ngrok http 3000
```

**Option B: Using Clerk's built-in tunneling**
```bash
npx clerk-tunnel
```

This will give you a public URL like: `https://xxxx-xx-xx-xx-xxx.ngrok-free.app`

### 3.3 Add Webhook Endpoint in Clerk Dashboard

1. Go to [Clerk Dashboard](https://dashboard.clerk.com/)
2. Select your application
3. Navigate to **Webhooks** in the left sidebar
4. Click **"+ Add Endpoint"**
5. Enter your webhook URL:
   - For ngrok: `https://your-ngrok-url.ngrok-free.app/api/webhooks/clerk`
   - For production: `https://yourdomain.com/api/webhooks/clerk`
6. Subscribe to these events:
   - ✅ `user.created`
   - ✅ `user.updated`
   - ✅ `user.deleted`
7. Click **"Create"**

### 3.4 Copy the Webhook Secret

After creating the webhook endpoint:
1. Click on the endpoint you just created
2. Copy the **Signing Secret** (starts with `whsec_`)
3. Add it to your `.env.local` file as `CLERK_WEBHOOK_SECRET`
4. **Restart your dev server** after adding the secret

## ✅ Step 4: Test the Webhook

### 4.1 Test with Existing User
If you already have a user signed up in Clerk:
1. Go to **Clerk Dashboard → Users**
2. Click on a user
3. Click **"..."** (three dots menu)
4. Select **"Send webhook"**
5. Choose `user.created` event
6. Click **"Send"**

### 4.2 Create a New Test User
1. Go to your app's sign-up page
2. Create a new account
3. Check your database - the user should now appear in the `users` table!

### 4.3 Verify in Database
You can verify users are being synced by checking your Neon database:
1. Go to [Neon Console](https://console.neon.tech/)
2. Select your project
3. Go to **SQL Editor**
4. Run: `SELECT * FROM users;`

You should see your Clerk users there!

## 📊 What the Webhook Does

The webhook at `/api/webhooks/clerk` handles three events:

1. **user.created** - When a new user signs up
   - Creates a new record in the `users` table
   - Stores: id, email, firstName, lastName, imageUrl

2. **user.updated** - When a user updates their profile
   - Updates the corresponding record in the `users` table

3. **user.deleted** - When a user is deleted
   - Removes the user from the `users` table

## 🔒 Security

The webhook is secured using Svix signatures. Clerk signs each webhook request, and our endpoint verifies the signature before processing. This ensures only legitimate requests from Clerk are accepted.

## ❌ Troubleshooting

### Webhook returns 400 error
- Check that `CLERK_WEBHOOK_SECRET` is set in `.env.local`
- Make sure you restarted the dev server after adding the secret

### User not appearing in database
- Check the console logs for error messages
- Verify the webhook URL is correct in Clerk Dashboard
- Make sure the database schema was pushed successfully
- Check that the `users` table exists in your database

### "Error occured -- no svix headers"
- The webhook URL might be wrong
- Make sure the request is coming from Clerk (not testing directly in browser)

## 📝 For Production

When deploying to production:

1. Update the webhook URL in Clerk Dashboard to your production URL
   - Example: `https://yourdomain.com/api/webhooks/clerk`
2. Make sure `CLERK_WEBHOOK_SECRET` is set in your production environment variables
3. The same webhook endpoint will work without any code changes!

## ✨ Next Steps

After setup is complete:
1. All new Clerk sign-ups will automatically create users in your database
2. You can now create relationships between users and other tables (like decks and cards)
3. You can query user data from your database instead of always calling Clerk API

---

## Current Database Schema

```typescript
// Users table - synced from Clerk
export const usersTable = pgTable("users", {
  id: varchar({ length: 255 }).primaryKey(), // Clerk user ID
  email: varchar({ length: 255 }).notNull(),
  firstName: varchar({ length: 255 }),
  lastName: varchar({ length: 255 }),
  imageUrl: text(),
  createdAt: timestamp().notNull().defaultNow(),
  updatedAt: timestamp().notNull().defaultNow(),
});

// Decks table references userId from users table
export const decksTable = pgTable("decks", {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  userId: varchar({ length: 255 }).notNull(), // References users.id (Clerk ID)
  // ... other fields
});
```

The `decksTable.userId` now properly references `usersTable.id`, creating a relationship between users and their decks.

