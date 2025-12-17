# Convex Coding Guidelines

**Official Reference:** https://convex.link/convex_rules.txt

This file summarizes the official Convex coding rules. Always refer to the official link above for the most up-to-date guidelines.

---

## Function Syntax

### Always use the new function syntax:
```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

export const myFunction = query({
  args: { name: v.string() },
  returns: v.string(),
  handler: async (ctx, args) => {
    return "Hello " + args.name;
  },
});
```

### Always include return validators:
- If a function doesn't return anything, use `returns: v.null()`
- JavaScript functions that don't return implicitly return `null`

---

## Function Registration

| Type | Use For | Import From |
|------|---------|-------------|
| `query` | Public read operations | `./_generated/server` |
| `mutation` | Public write operations | `./_generated/server` |
| `action` | Public external API calls | `./_generated/server` |
| `internalQuery` | Private read operations | `./_generated/server` |
| `internalMutation` | Private write operations | `./_generated/server` |
| `internalAction` | Private external API calls | `./_generated/server` |

**Important:** Use `internal*` functions for sensitive operations that should not be exposed publicly.

---

## Function References

- Use `api` object for public functions: `api.example.myFunction`
- Use `internal` object for private functions: `internal.example.myFunction`
- File-based routing: `convex/messages/send.ts` → `api.messages.send.functionName`

---

## Validators

| Convex Type | TypeScript | Validator |
|-------------|------------|-----------|
| Id | string | `v.id("tableName")` |
| Null | null | `v.null()` |
| Int64 | bigint | `v.int64()` |
| Float64 | number | `v.number()` |
| Boolean | boolean | `v.boolean()` |
| String | string | `v.string()` |
| Bytes | ArrayBuffer | `v.bytes()` |
| Array | Array | `v.array(v.string())` |
| Object | Object | `v.object({ field: v.string() })` |
| Record | Record | `v.record(v.string(), v.number())` |
| Union | union | `v.union(v.string(), v.number())` |
| Literal | literal | `v.literal("value")` |
| Optional | optional | `v.optional(v.string())` |

**Note:** `v.bigint()` is deprecated - use `v.int64()` instead.

---

## Schema Guidelines

- Always define schema in `convex/schema.ts`
- System fields `_id` and `_creationTime` are added automatically
- Include all index fields in index name: `by_field1_and_field2`
- Index fields must be queried in order they are defined

```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
  }).index("by_email", ["email"]),
});
```

---

## Function Calling

- `ctx.runQuery` - Call query from query/mutation/action
- `ctx.runMutation` - Call mutation from mutation/action
- `ctx.runAction` - Call action from action only

**Important:** Minimize calls from actions to queries/mutations to avoid race conditions.

---

## HTTP Endpoints

Define in `convex/http.ts`:
```typescript
import { httpRouter } from "convex/server";
import { httpAction } from "./_generated/server";

const http = httpRouter();

http.route({
  path: "/api/webhook",
  method: "POST",
  handler: httpAction(async (ctx, req) => {
    const body = await req.json();
    return new Response(JSON.stringify({ ok: true }), { status: 200 });
  }),
});

export default http;
```

---

## TypeScript Types

```typescript
import { Doc, Id } from "./_generated/dataModel";

// Document type
type User = Doc<"users">;

// ID type
type UserId = Id<"users">;
```

---

*Source: [Convex Official Guidelines](https://convex.link/convex_rules.txt)*
