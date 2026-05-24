---
name: valibot-zod-translator
description: Translates validation schemas between Valibot and Zod libraries. Handles schema conversion, type inference, error handling, and API mapping. Use when migrating validation code between libraries or refactoring schemas across repositories.
---

# Valibot ↔ Zod Translator Skill

Converts validation schemas and patterns between Valibot's functional composition model and Zod's method-chaining API. Works bidirectionally for single schemas, nested objects, custom validators, and error handling.

## Quick Start

### Zod to Valibot Pattern

**Zod (method chaining):**
```typescript
const schema = z.object({
  email: z.string().email(),
  age: z.number().min(18).max(120)
});
```

**Valibot (functional pipe):**
```typescript
const schema = v.pipe(
  v.object({
    email: v.pipe(v.string(), v.email()),
    age: v.pipe(v.number(), v.minValue(18), v.maxValue(120))
  })
);
```

### Valibot to Zod Pattern

**Valibot:**
```typescript
const schema = v.pipe(
  v.string(),
  v.minLength(5),
  v.regex(/^[a-z]+$/)
);
```

**Zod:**
```typescript
const schema = z.string().min(5).regex(/^[a-z]+$/);
```

## Workflows

### 1. Converting Zod Object to Valibot

**Steps:**
1. Identify Zod's `z.object()` structure
2. Convert to Valibot's `v.object()` with pipe-wrapped validators
3. Replace chained methods with `v.pipe()` wrapping
4. Update `z.infer<typeof schema>` to `v.infer<typeof schema>`

**Checklist:**
- [ ] Object fields wrapped with `v.pipe()`
- [ ] All validators use Valibot naming (e.g., `minLength` not `min`)
- [ ] Type inference updated from `z.infer` to `v.infer`
- [ ] Custom validators converted to Valibot's functional style

### 2. Converting Valibot Pipe to Zod

**Steps:**
1. Extract base schema from Valibot's `v.pipe()` first argument
2. Convert remaining pipe arguments to chained Zod methods
3. Unwrap nested `v.pipe()` calls into method chains
4. Update `v.infer` to `z.infer`

**Checklist:**
- [ ] Base type correctly identified and initialized
- [ ] All pipe validators converted to Zod methods
- [ ] Method chain order preserved
- [ ] Type inference updated

### 3. Handling Nested/Complex Schemas

**For nested objects:**
- Valibot: Each field gets its own `v.pipe()` wrapper
- Zod: Fields use chained methods directly
- Both maintain identical nesting structure inside `object()`

**For arrays:**
- Valibot: `v.array(v.pipe(/* base */, /* validators */))`
- Zod: `z.array(/* schema */)`

**For unions/discriminated unions:**
- Valibot: `v.union([schema1, schema2])`
- Zod: `z.union([schema1, schema2])` or `.discriminatedUnion()`

### 4. Converting Error Handling

**Zod error handling:**
```typescript
const result = schema.safeParse(data);
if (!result.success) {
  console.error(result.error.flatten());
}
```

**Valibot error handling:**
```typescript
const result = v.safeParse(schema, data);
if (!result.success) {
  console.error(result.issues);
}
```

## Key API Mappings

### String Validators
| Zod | Valibot |
|-----|---------|
| `.string()` | `v.string()` |
| `.email()` | `v.email()` |
| `.url()` | `v.url()` |
| `.min(n)` | `v.minLength(n)` |
| `.max(n)` | `v.maxLength(n)` |
| `.regex(pattern)` | `v.regex(pattern)` |
| `.transform(fn)` | `v.transform(fn)` |

### Number Validators
| Zod | Valibot |
|-----|---------|
| `.number()` | `v.number()` |
| `.min(n)` | `v.minValue(n)` |
| `.max(n)` | `v.maxValue(n)` |
| `.int()` | `v.integer()` |
| `.positive()` | `v.minValue(0)` (or custom) |

### Object & Array
| Zod | Valibot |
|-----|---------|
| `.object({...})` | `v.object({...})` |
| `.array(schema)` | `v.array(schema)` |
| `.optional()` | `v.optional(schema)` |
| `.nullable()` | `v.nullable(schema)` |
| `.or(schema)` | wrapped in `v.union()` |

## Advanced Features

See [EXAMPLES.md](EXAMPLES.md) for detailed conversion examples including:
- Custom validators and refinements
- Conditional validation (.refine, .superRefine)
- Async validators
- Form validation patterns
- API route validation

See [REFERENCE.md](REFERENCE.md) for complete API reference and implementation notes.

## Translation Tips

1. **Bundle size aware**: Remember Valibot is ~90% smaller—consider switching for frontend schemas
2. **Performance**: Both similar in v4; Valibot ~2x faster than Zod v3
3. **DX tradeoff**: Zod's chaining is more intuitive; Valibot's composition is more composable
4. **Ecosystem**: Zod has deeper integration (tRPC, Astro, etc.); check before migrating
5. **Type inference**: Both use similar patterns (`z.infer` ↔ `v.infer`)—update these consistently

## Common Pitfalls

- ❌ Forgetting to wrap Valibot validators in `v.pipe()`
- ❌ Using Zod method names in Valibot (`.min()` instead of `.minLength()`)
- ❌ Not updating error handling from `result.error` to `result.issues`
- ❌ Assuming validation ordering is identical (always test after conversion)
- ❌ Forgetting imports: `import * as v from 'valibot'` or `import { z } from 'zod'`

---

**Last updated:** 2026-05-25  
**Version:** 1.0
