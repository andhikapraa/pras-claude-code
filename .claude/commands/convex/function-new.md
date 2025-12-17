---
description: Create a new Convex function (query, mutation, or action)
model: claude-opus-4-5
---

> **Reference:** See `.claude/convex-rules.md` for official Convex coding guidelines
> **Official Rules:** https://convex.link/convex_rules.txt

Create a new Convex function with proper TypeScript typing and best practices.

## Function Specification

$ARGUMENTS

## Convex Function Types

### Overview
| Type | Purpose | Database Access | External APIs | Caching |
|------|---------|-----------------|---------------|---------|
| **Query** | Read data | Read-only | No | Yes (reactive) |
| **Mutation** | Write data | Read/Write | No | No |
| **Action** | Side effects | Via scheduler | Yes | No |

## Query Functions (Read-only)

### Basic Query
```typescript
// convex/posts.ts
import { query } from "./_generated/server";
import { v } from "convex/values";

export const list = query({
  args: {},
  handler: async (ctx) => {
    return await ctx.db.query("posts").collect();
  },
});
```

### Query with Arguments
```typescript
export const getById = query({
  args: { id: v.id("posts") },
  handler: async (ctx, args) => {
    return await ctx.db.get(args.id);
  },
});
```

### Query with Index
```typescript
export const getByAuthor = query({
  args: { authorId: v.id("users") },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("posts")
      .withIndex("by_author", (q) => q.eq("authorId", args.authorId))
      .collect();
  },
});
```

### Query with Filtering & Pagination
```typescript
export const listPublished = query({
  args: {
    limit: v.optional(v.number()),
    cursor: v.optional(v.string()),
  },
  handler: async (ctx, args) => {
    const results = await ctx.db
      .query("posts")
      .withIndex("by_published", (q) => q.eq("published", true))
      .order("desc")
      .paginate({ numItems: args.limit ?? 10, cursor: args.cursor ?? null });

    return results;
  },
});
```

## Mutation Functions (Write)

### Basic Mutation
```typescript
// convex/posts.ts
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const create = mutation({
  args: {
    title: v.string(),
    content: v.string(),
  },
  handler: async (ctx, args) => {
    const id = await ctx.db.insert("posts", {
      ...args,
      published: false,
      createdAt: Date.now(),
    });
    return id;
  },
});
```

### Mutation with Authentication
```typescript
export const create = mutation({
  args: {
    title: v.string(),
    content: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) {
      throw new Error("Unauthorized");
    }

    return await ctx.db.insert("posts", {
      ...args,
      authorId: identity.subject,
      published: false,
      createdAt: Date.now(),
    });
  },
});
```

### Update Mutation
```typescript
export const update = mutation({
  args: {
    id: v.id("posts"),
    title: v.optional(v.string()),
    content: v.optional(v.string()),
  },
  handler: async (ctx, args) => {
    const { id, ...updates } = args;

    // Remove undefined values
    const cleanUpdates = Object.fromEntries(
      Object.entries(updates).filter(([_, v]) => v !== undefined)
    );

    await ctx.db.patch(id, cleanUpdates);
  },
});
```

### Delete Mutation
```typescript
export const remove = mutation({
  args: { id: v.id("posts") },
  handler: async (ctx, args) => {
    // Optional: verify ownership first
    const post = await ctx.db.get(args.id);
    if (!post) {
      throw new Error("Post not found");
    }

    await ctx.db.delete(args.id);
  },
});
```

## Action Functions (Side Effects)

### Basic Action (External API)
```typescript
// convex/actions.ts
import { action } from "./_generated/server";
import { v } from "convex/values";

export const sendEmail = action({
  args: {
    to: v.string(),
    subject: v.string(),
    body: v.string(),
  },
  handler: async (ctx, args) => {
    const response = await fetch("https://api.resend.com/emails", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${process.env.RESEND_API_KEY}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        from: "noreply@example.com",
        to: args.to,
        subject: args.subject,
        html: args.body,
      }),
    });

    return response.ok;
  },
});
```

### Action Calling Mutations
```typescript
import { action } from "./_generated/server";
import { internal } from "./_generated/api";
import { v } from "convex/values";

export const processWebhook = action({
  args: { payload: v.any() },
  handler: async (ctx, args) => {
    // Process external data
    const processed = transformPayload(args.payload);

    // Call internal mutation to save
    await ctx.runMutation(internal.webhooks.save, {
      data: processed,
    });

    return { success: true };
  },
});
```

## Internal Functions

### Define Internal Functions
```typescript
// convex/internal.ts
import { internalMutation, internalQuery } from "./_generated/server";
import { v } from "convex/values";

// Only callable from other Convex functions, not from client
export const logEvent = internalMutation({
  args: {
    type: v.string(),
    data: v.any(),
  },
  handler: async (ctx, args) => {
    await ctx.db.insert("events", {
      ...args,
      timestamp: Date.now(),
    });
  },
});
```

### Scheduled Functions (Cron Jobs)
```typescript
// convex/crons.ts
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";

const crons = cronJobs();

// Run every hour
crons.interval(
  "cleanup old sessions",
  { hours: 1 },
  internal.maintenance.cleanupSessions
);

// Run daily at midnight UTC
crons.daily(
  "send daily digest",
  { hourUTC: 0, minuteUTC: 0 },
  internal.emails.sendDailyDigest
);

export default crons;
```

## Frontend Usage (React)

### useQuery Hook
```typescript
"use client";
import { useQuery } from "convex/react";
import { api } from "@/convex/_generated/api";

export function PostList() {
  const posts = useQuery(api.posts.list);

  if (posts === undefined) return <Loading />;
  if (posts.length === 0) return <Empty />;

  return (
    <ul>
      {posts.map((post) => (
        <li key={post._id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

### useMutation Hook
```typescript
"use client";
import { useMutation } from "convex/react";
import { api } from "@/convex/_generated/api";

export function CreatePostForm() {
  const createPost = useMutation(api.posts.create);

  const handleSubmit = async (data: FormData) => {
    await createPost({
      title: data.get("title") as string,
      content: data.get("content") as string,
    });
  };

  return <form action={handleSubmit}>...</form>;
}
```

### useAction Hook
```typescript
"use client";
import { useAction } from "convex/react";
import { api } from "@/convex/_generated/api";

export function SendEmailButton({ to }: { to: string }) {
  const sendEmail = useAction(api.actions.sendEmail);

  return (
    <button onClick={() => sendEmail({ to, subject: "Hello", body: "..." })}>
      Send Email
    </button>
  );
}
```

## Best Practices

### 1. **Function Organization**
```
convex/
├── schema.ts          # Schema definitions
├── users.ts           # User queries/mutations
├── posts.ts           # Post queries/mutations
├── actions.ts         # External API actions
├── internal.ts        # Internal-only functions
└── crons.ts           # Scheduled jobs
```

### 2. **Error Handling**
```typescript
export const update = mutation({
  args: { id: v.id("posts"), title: v.string() },
  handler: async (ctx, args) => {
    const post = await ctx.db.get(args.id);
    if (!post) {
      throw new Error("Post not found");
    }

    const identity = await ctx.auth.getUserIdentity();
    if (post.authorId !== identity?.subject) {
      throw new Error("Forbidden");
    }

    await ctx.db.patch(args.id, { title: args.title });
  },
});
```

### 3. **Type Safety**
```typescript
import { Doc, Id } from "./_generated/dataModel";

// Return type annotation
export const getUser = query({
  args: { id: v.id("users") },
  handler: async (ctx, args): Promise<Doc<"users"> | null> => {
    return await ctx.db.get(args.id);
  },
});
```

## What to Generate

1. **Function File** - Appropriate query/mutation/action
2. **Argument Validators** - Proper `v.*` types
3. **Error Handling** - Auth checks, not found, forbidden
4. **Frontend Hook Usage** - Example React component

Generate production-ready Convex functions with proper typing, validation, and error handling.
