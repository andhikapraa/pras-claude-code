---
description: Generate Convex schema definitions and TypeScript types
model: claude-opus-4-5
---

> **Reference:** See `.claude/convex-rules.md` for official Convex coding guidelines
> **Official Rules:** https://convex.link/convex_rules.txt

Generate a Convex schema definition with proper TypeScript types.

## Schema Specification

$ARGUMENTS

## Convex Schema Fundamentals

### 1. **Schema File Location**
All schemas are defined in `convex/schema.ts` - Convex automatically generates types from this file.

### 2. **Basic Schema Structure**
```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  // Define tables here
});
```

## Validator Types

### Primitive Validators
```typescript
v.string()        // string
v.number()        // number (int or float)
v.boolean()       // boolean
v.int64()         // 64-bit integer
v.bytes()         // ArrayBuffer
v.null()          // null
```

### Complex Validators
```typescript
v.id("tableName")           // Reference to another table
v.array(v.string())         // Array of strings
v.object({ ... })           // Nested object
v.union(v.literal("a"), v.literal("b"))  // Union/enum type
v.optional(v.string())      // Optional field
v.any()                     // Any type (avoid if possible)
```

## Schema Patterns

### 1. **Basic Table**
```typescript
export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
    avatarUrl: v.optional(v.string()),
    createdAt: v.number(),
  }),
});
```

### 2. **Table with Indexes**
```typescript
export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
    role: v.union(v.literal("admin"), v.literal("user")),
  })
    .index("by_email", ["email"])
    .index("by_role", ["role"]),
});
```

### 3. **Table with References**
```typescript
export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
  }),

  posts: defineTable({
    title: v.string(),
    content: v.string(),
    authorId: v.id("users"),  // Reference to users table
    published: v.boolean(),
  })
    .index("by_author", ["authorId"]),
});
```

### 4. **Enum/Union Types**
```typescript
export default defineSchema({
  tasks: defineTable({
    title: v.string(),
    status: v.union(
      v.literal("pending"),
      v.literal("in_progress"),
      v.literal("completed"),
      v.literal("cancelled")
    ),
    priority: v.union(
      v.literal("low"),
      v.literal("medium"),
      v.literal("high")
    ),
  }),
});
```

### 5. **Nested Objects**
```typescript
export default defineSchema({
  profiles: defineTable({
    userId: v.id("users"),
    settings: v.object({
      theme: v.union(v.literal("light"), v.literal("dark")),
      notifications: v.object({
        email: v.boolean(),
        push: v.boolean(),
      }),
    }),
    socialLinks: v.optional(v.object({
      twitter: v.optional(v.string()),
      github: v.optional(v.string()),
    })),
  }),
});
```

## Using Generated Types

### Import Types
```typescript
import { Doc, Id } from "./_generated/dataModel";

// Full document type (includes _id and _creationTime)
type User = Doc<"users">;

// Just the ID type
type UserId = Id<"users">;

// In function signatures
export const getUser = query({
  args: { userId: v.id("users") },
  handler: async (ctx, args): Promise<Doc<"users"> | null> => {
    return await ctx.db.get(args.userId);
  },
});
```

### Type Inference in Functions
```typescript
// Types are automatically inferred from validators
export const createUser = mutation({
  args: {
    name: v.string(),
    email: v.string(),
  },
  // args is typed as { name: string; email: string }
  handler: async (ctx, args) => {
    return await ctx.db.insert("users", args);
  },
});
```

## Index Best Practices

### When to Add Indexes
- Fields used in `.filter()` or `.withIndex()` queries
- Foreign key fields (e.g., `authorId`, `userId`)
- Fields used for sorting or pagination
- Unique lookup fields (email, slug, etc.)

### Index Patterns
```typescript
defineTable({ ... })
  // Single field index
  .index("by_email", ["email"])

  // Compound index (for queries filtering on multiple fields)
  .index("by_author_and_status", ["authorId", "status"])

  // Search index (for full-text search)
  .searchIndex("search_content", {
    searchField: "content",
    filterFields: ["authorId"],
  })
```

## What to Generate

1. **Schema Definition** - Complete `convex/schema.ts` file
2. **Type Exports** - Helper types if needed
3. **Index Strategy** - Appropriate indexes for query patterns
4. **Migration Notes** - If modifying existing schema

## Schema Change Workflow

1. Update `convex/schema.ts`
2. Run `npx convex dev` (types auto-regenerate)
3. Update functions to use new fields
4. Test queries and mutations

Generate a well-structured Convex schema with appropriate validators, indexes, and TypeScript types.
