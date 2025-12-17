---
description: Add Convex authentication and authorization to API endpoints
model: claude-opus-4-5
---

> **Reference:** See `.claude/convex-rules.md` for official Convex coding guidelines
> **Official Rules:** https://convex.link/convex_rules.txt

Add Convex-based authentication, authorization, and security to the specified function or API route.

## Target Function/Route

$ARGUMENTS

## Convex Security Model

Convex provides built-in security through:
1. **Authentication** - Identity verification via auth providers
2. **Authorization** - Function-level access control
3. **Validation** - Argument validators prevent invalid data
4. **Type Safety** - TypeScript ensures correct usage

## Authentication Patterns

### 1. **Get Current User Identity**
```typescript
import { query, mutation } from "./_generated/server";

export const getMyProfile = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) {
      throw new Error("Unauthorized");
    }

    // identity contains:
    // - subject: unique user ID from auth provider
    // - tokenIdentifier: full token identifier
    // - email, name, pictureUrl (if provided by auth)

    return await ctx.db
      .query("users")
      .withIndex("by_token", (q) =>
        q.eq("tokenIdentifier", identity.tokenIdentifier)
      )
      .unique();
  },
});
```

### 2. **Auth Helper Function**
```typescript
// convex/lib/auth.ts
import { QueryCtx, MutationCtx } from "./_generated/server";

export async function requireAuth(ctx: QueryCtx | MutationCtx) {
  const identity = await ctx.auth.getUserIdentity();
  if (!identity) {
    throw new Error("Unauthorized");
  }
  return identity;
}

export async function requireUser(ctx: QueryCtx | MutationCtx) {
  const identity = await requireAuth(ctx);

  const user = await ctx.db
    .query("users")
    .withIndex("by_token", (q) =>
      q.eq("tokenIdentifier", identity.tokenIdentifier)
    )
    .unique();

  if (!user) {
    throw new Error("User not found");
  }

  return user;
}
```

### 3. **Using Auth Helpers**
```typescript
import { mutation } from "./_generated/server";
import { v } from "convex/values";
import { requireUser } from "./lib/auth";

export const createPost = mutation({
  args: {
    title: v.string(),
    content: v.string(),
  },
  handler: async (ctx, args) => {
    const user = await requireUser(ctx);

    return await ctx.db.insert("posts", {
      ...args,
      authorId: user._id,
      createdAt: Date.now(),
    });
  },
});
```

## Authorization Patterns

### 1. **Document Ownership**
```typescript
export const updatePost = mutation({
  args: {
    id: v.id("posts"),
    title: v.string(),
  },
  handler: async (ctx, args) => {
    const user = await requireUser(ctx);
    const post = await ctx.db.get(args.id);

    if (!post) {
      throw new Error("Not found");
    }

    if (post.authorId !== user._id) {
      throw new Error("Forbidden");
    }

    await ctx.db.patch(args.id, { title: args.title });
  },
});
```

### 2. **Role-Based Access Control**
```typescript
// convex/lib/auth.ts
export async function requireRole(
  ctx: QueryCtx | MutationCtx,
  allowedRoles: string[]
) {
  const user = await requireUser(ctx);

  if (!allowedRoles.includes(user.role)) {
    throw new Error("Forbidden: insufficient permissions");
  }

  return user;
}

// Usage
export const deleteAnyPost = mutation({
  args: { id: v.id("posts") },
  handler: async (ctx, args) => {
    await requireRole(ctx, ["admin", "moderator"]);
    await ctx.db.delete(args.id);
  },
});
```

### 3. **Resource-Level Permissions**
```typescript
// Check if user has access to a workspace
async function requireWorkspaceAccess(
  ctx: QueryCtx | MutationCtx,
  workspaceId: Id<"workspaces">,
  requiredRole?: "owner" | "admin" | "member"
) {
  const user = await requireUser(ctx);

  const membership = await ctx.db
    .query("workspaceMembers")
    .withIndex("by_workspace_user", (q) =>
      q.eq("workspaceId", workspaceId).eq("userId", user._id)
    )
    .unique();

  if (!membership) {
    throw new Error("Forbidden: not a workspace member");
  }

  if (requiredRole) {
    const roleHierarchy = { owner: 3, admin: 2, member: 1 };
    if (roleHierarchy[membership.role] < roleHierarchy[requiredRole]) {
      throw new Error(`Forbidden: requires ${requiredRole} role`);
    }
  }

  return { user, membership };
}
```

## Input Validation

### Convex Validators (Built-in)
```typescript
import { v } from "convex/values";

export const createUser = mutation({
  args: {
    // Required string
    email: v.string(),

    // Optional field
    name: v.optional(v.string()),

    // Enum/union type
    role: v.union(v.literal("user"), v.literal("admin")),

    // Constrained number (validate in handler)
    age: v.number(),

    // Reference to another table
    teamId: v.optional(v.id("teams")),
  },
  handler: async (ctx, args) => {
    // Additional validation
    if (!args.email.includes("@")) {
      throw new Error("Invalid email format");
    }

    if (args.age < 0 || args.age > 150) {
      throw new Error("Invalid age");
    }

    // Validators guarantee type safety
    return await ctx.db.insert("users", args);
  },
});
```

### Custom Validation Helpers
```typescript
// convex/lib/validators.ts
export function validateEmail(email: string): void {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    throw new Error("Invalid email format");
  }
}

export function validateUrl(url: string): void {
  try {
    new URL(url);
  } catch {
    throw new Error("Invalid URL format");
  }
}

export function validateLength(
  value: string,
  min: number,
  max: number,
  fieldName: string
): void {
  if (value.length < min || value.length > max) {
    throw new Error(`${fieldName} must be between ${min} and ${max} characters`);
  }
}
```

## Next.js API Route Integration

### Server-Side Convex Calls
```typescript
// app/api/posts/route.ts
import { fetchQuery, fetchMutation } from "convex/nextjs";
import { api } from "@/convex/_generated/api";
import { auth } from "@clerk/nextjs/server";

export async function GET() {
  const posts = await fetchQuery(api.posts.listPublished);
  return Response.json(posts);
}

export async function POST(request: Request) {
  const { userId, getToken } = await auth();

  if (!userId) {
    return Response.json({ error: "Unauthorized" }, { status: 401 });
  }

  const token = await getToken({ template: "convex" });
  const body = await request.json();

  try {
    const id = await fetchMutation(
      api.posts.create,
      { title: body.title, content: body.content },
      { token }
    );
    return Response.json({ id }, { status: 201 });
  } catch (error) {
    return Response.json(
      { error: error instanceof Error ? error.message : "Unknown error" },
      { status: 400 }
    );
  }
}
```

### HTTP Actions (Webhooks)
```typescript
// convex/http.ts
import { httpRouter } from "convex/server";
import { httpAction } from "./_generated/server";
import { internal } from "./_generated/api";

const http = httpRouter();

http.route({
  path: "/webhooks/stripe",
  method: "POST",
  handler: httpAction(async (ctx, request) => {
    // Verify webhook signature
    const signature = request.headers.get("stripe-signature");
    const body = await request.text();

    // Validate signature (implement based on provider)
    if (!verifyStripeSignature(body, signature)) {
      return new Response("Invalid signature", { status: 401 });
    }

    const event = JSON.parse(body);

    // Process webhook
    await ctx.runMutation(internal.payments.handleWebhook, { event });

    return new Response("OK", { status: 200 });
  }),
});

export default http;
```

## Security Checklist

**Authentication**
-  Verify identity in all protected functions
-  Use auth helper functions for consistency
-  Handle missing identity (return 401)
-  Store tokenIdentifier for user lookup

**Authorization**
-  Check document ownership before mutations
-  Implement role-based access where needed
-  Verify resource-level permissions
-  Use least privilege principle

**Input Validation**
-  Use Convex validators for type safety
-  Add custom validation for business rules
-  Validate string lengths and formats
-  Sanitize user input

**Error Handling**
-  Return appropriate error messages
-  Don't expose internal details
-  Log errors server-side
-  Use consistent error format

## What to Generate

1. **Auth Helpers** - Reusable authentication utilities
2. **Protected Functions** - Secured queries/mutations
3. **Authorization Logic** - Ownership and role checks
4. **Validation** - Input validators and sanitization
5. **Error Handling** - Consistent error responses

Generate production-ready, secure Convex functions following the principle of least privilege.
