# Clerk Authentication Setup Guide

## ✅ Completed Steps

The following has been set up for you:

1. ✅ Installed `@clerk/nextjs` package
2. ✅ Created `src/middleware.ts` with `clerkMiddleware()`
3. ✅ Updated `src/app/layout.tsx` with `ClerkProvider` and auth components

## 🔑 Environment Variables Setup

**You need to create a `.env.local` file** in the `my-nextjs-app` directory with your Clerk API keys.

### Steps:

1. Go to [Clerk Dashboard](https://dashboard.clerk.com/last-active?path=api-keys)
2. Copy your **Publishable Key** and **Secret Key**
3. Create a file named `.env.local` in the `my-nextjs-app` directory
4. Add the following content (replace with your actual keys):

```bash
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CLERK_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**⚠️ IMPORTANT:** 
- Never commit `.env.local` to git (it's already in `.gitignore`)
- Keep your Secret Key private and secure
- Use test keys for development, production keys for deployment

## 🚀 Running the Application

Once you've added your Clerk keys to `.env.local`:

```bash
npm run dev
```

Visit `http://localhost:3000` and you should see:
- **Sign In** and **Sign Up** buttons when not authenticated
- **User Button** (avatar) when authenticated

## 📝 What Was Integrated

### 1. Middleware (`src/middleware.ts`)
- Uses `clerkMiddleware()` from `@clerk/nextjs/server`
- Configured to run on all routes except static files and Next.js internals
- Handles authentication state automatically

### 2. Root Layout (`src/app/layout.tsx`)
- Wrapped with `<ClerkProvider>`
- Added header with authentication UI:
  - `<SignInButton>` - Opens sign-in modal
  - `<SignUpButton>` - Opens sign-up modal  
  - `<UserButton>` - Shows user avatar and account menu
  - `<SignedIn>` and `<SignedOut>` - Conditional rendering based on auth state

## 🔐 Using Authentication in Your App

### In Server Components

```typescript
import { auth } from "@clerk/nextjs/server";

export default async function Page() {
  const { userId } = await auth();
  
  if (!userId) {
    return <div>Not authenticated</div>;
  }
  
  return <div>User ID: {userId}</div>;
}
```

### In Client Components

```typescript
"use client";

import { useUser } from "@clerk/nextjs";

export default function ClientComponent() {
  const { isLoaded, isSignedIn, user } = useUser();
  
  if (!isLoaded) return <div>Loading...</div>;
  if (!isSignedIn) return <div>Not signed in</div>;
  
  return <div>Hello, {user.firstName}!</div>;
}
```

### Protecting Routes

You can make routes require authentication by using `clerkMiddleware` with custom logic:

```typescript
// src/middleware.ts
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isProtectedRoute = createRouteMatcher([
  '/dashboard(.*)',
  '/admin(.*)',
]);

export default clerkMiddleware(async (auth, req) => {
  if (isProtectedRoute(req)) {
    await auth.protect();
  }
});

export const config = {
  matcher: [
    "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
    "/(api|trpc)(.*)",
  ],
};
```

## 📚 Additional Resources

- [Clerk Next.js Documentation](https://clerk.com/docs/quickstarts/nextjs)
- [Clerk React Components](https://clerk.com/docs/components/overview)
- [Clerk Authentication Hooks](https://clerk.com/docs/references/react/use-user)

## ✨ Next Steps

1. Create your `.env.local` file with Clerk keys
2. Run `npm run dev`
3. Test sign up and sign in flows
4. Customize the authentication UI as needed
5. Add protected routes for authenticated users only

