# Valibot ↔ Zod Complete API Reference

## Fundamental Differences

| Aspect | Zod | Valibot |
|--------|-----|---------|
| **API Style** | Method chaining | Functional composition via pipes |
| **Bundle Size** | 17.7 KB (standard), 9.5 KB (mini) | 1.37 KB (tree-shakeable) |
| **Performance** | Similar to Valibot v4, slower than v3 | ~2x faster than Zod v3, equal to v4 |
| **Type Inference** | `z.infer<typeof schema>` | `v.infer<typeof schema>` |
| **Error Format** | `.error.flatten()` / `.error.errors` | `.issues` (array of ValiError) |
| **Parse Functions** | `schema.safeParse(data)` | `v.safeParse(schema, data)` |
| **Custom Validation** | `.refine()` / `.superRefine()` | `v.custom()` / `v.checkAsync()` |
| **Async Support** | Built into `.refine(async)` | Separate `v.checkAsync()` |
| **Ecosystem** | tRPC, Astro, React Hook Form | SvelteKit, Superforms, Fastify |

---

## String Validators - Complete Mapping

### Basic
```typescript
// Zod
z.string()

// Valibot
v.string()
```

### Email
```typescript
// Zod
z.string().email()
z.string().email('Custom message')

// Valibot
v.email()
v.email('Custom message')

// In context
z.string().email() 
  ↓
v.pipe(v.string(), v.email())
```

### URL
```typescript
// Zod
z.string().url()

// Valibot
v.url()

// In context
z.string().url()
  ↓
v.pipe(v.string(), v.url())
```

### Length Constraints
```typescript
// Zod
z.string().min(5)        // min length
z.string().max(20)       // max length
z.string().length(10)    // exact length

// Valibot
v.minLength(5)
v.maxLength(20)
v.length(10)

// In context
z.string().min(5).max(20)
  ↓
v.pipe(v.string(), v.minLength(5), v.maxLength(20))
```

### Regex/Pattern
```typescript
// Zod
z.string().regex(/^[a-z]+$/)
z.string().regex(/^[a-z]+$/, 'Custom message')

// Valibot
v.regex(/^[a-z]+$/)
v.regex(/^[a-z]+$/, 'Custom message')

// In context
z.string().regex(/^[a-z]+$/)
  ↓
v.pipe(v.string(), v.regex(/^[a-z]+$/))
```

### Case
```typescript
// Zod
z.string().toLowerCase()
z.string().toUpperCase()

// Valibot (use transform instead)
v.transform(str => str.toLowerCase())
v.transform(str => str.toUpperCase())

// In context
z.string().email().toLowerCase()
  ↓
v.pipe(
  v.string(),
  v.email(),
  v.transform(str => str.toLowerCase())
)
```

### Trim
```typescript
// Zod
z.string().trim()

// Valibot
v.transform(str => str.trim())
```

### Starts/Ends With
```typescript
// Zod
z.string().startsWith('prefix')
z.string().endsWith('suffix')

// Valibot
v.startsWith('prefix')
v.endsWith('suffix')
```

### Special Types
```typescript
// Zod
z.string().uuid()
z.string().cuid()
z.string().datetime()
z.string().date()
z.string().time()
z.string().duration()

// Valibot
v.uuid()
// v.cuid() - not built-in
v.datetime()
// v.date() - use custom validator
// v.time() - use custom validator
// v.duration() - use custom validator
```

---

## Number Validators - Complete Mapping

### Basic
```typescript
// Zod
z.number()

// Valibot
v.number()
```

### Range
```typescript
// Zod
z.number().min(0)        // ≥ min
z.number().max(100)      // ≤ max
z.number().gt(0)         // > min
z.number().lt(100)       // < max

// Valibot
v.minValue(0)
v.maxValue(100)
// v.gt() - use custom validator
// v.lt() - use custom validator

// In context
z.number().min(18).max(120)
  ↓
v.pipe(v.number(), v.minValue(18), v.maxValue(120))
```

### Precision
```typescript
// Zod
z.number().int()           // integer only
z.number().positive()      // > 0
z.number().nonnegative()   // ≥ 0
z.number().negative()      // < 0
z.number().nonpositive()   // ≤ 0

// Valibot
v.integer()
v.minValue(0.0000001)      // close to positive
v.minValue(0)              // close to nonnegative
v.maxValue(-0.0000001)     // close to negative
v.maxValue(0)              // close to nonpositive
```

### Safe Integers
```typescript
// Zod
z.number().safe()          // within Number.MAX_SAFE_INTEGER

// Valibot
// Not built-in; use custom validator
v.custom(n => Number.isSafeInteger(n), 'Must be safe integer')
```

### Multiple
```typescript
// Zod
z.number().multipleOf(5)

// Valibot
// Not built-in; use custom validator
v.custom(n => n % 5 === 0, 'Must be multiple of 5')
```

---

## Boolean, Literal, Enum Validators

### Boolean
```typescript
// Zod
z.boolean()

// Valibot
v.boolean()
```

### Literal
```typescript
// Zod
z.literal('active')
z.literal(42)

// Valibot
v.literal('active')
v.literal(42)
```

### Enum (Picklist)
```typescript
// Zod
z.enum(['small', 'medium', 'large'])

// Valibot
v.picklist(['small', 'medium', 'large'])

// Native TypeScript enum
enum Status { Active = 'active', Inactive = 'inactive' }

// Zod
z.nativeEnum(Status)

// Valibot
v.picklist(Object.values(Status))
```

---

## Collections: Object, Array, Record, Tuple

### Object
```typescript
// Zod
z.object({ name: z.string(), age: z.number() })

// Valibot
v.object({ name: v.string(), age: v.number() })

// With nested pipes
z.object({ 
  email: z.string().email(),
  age: z.number().min(18)
})
  ↓
v.object({
  email: v.pipe(v.string(), v.email()),
  age: v.pipe(v.number(), v.minValue(18))
})
```

### Array
```typescript
// Zod
z.array(z.string())
z.array(z.string()).min(1).max(10)

// Valibot
v.array(v.string())
v.pipe(v.array(v.string()), v.minLength(1), v.maxLength(10))
```

### Record (Key-Value)
```typescript
// Zod
z.record(z.string())                    // Record<string, string>
z.record(z.enum(['a', 'b']), z.number()) // Record<'a'|'b', number>

// Valibot
v.record(v.string())
v.record(v.picklist(['a', 'b']), v.number())
```

### Tuple
```typescript
// Zod
z.tuple([z.string(), z.number()])

// Valibot
v.tuple([v.string(), v.number()])
```

### Set
```typescript
// Zod
z.set(z.string())

// Valibot
// Not built-in; use custom validator or transform
v.transform(arr => new Set(arr))
```

### Map
```typescript
// Zod
z.map(z.string(), z.number())

// Valibot
// Not built-in; use custom validator or transform
```

---

## Optional, Nullable, Default

### Optional (undefined allowed)
```typescript
// Zod
z.string().optional()        // string | undefined

// Valibot
v.optional(v.string())

// In pipes
z.string().email().optional()
  ↓
v.optional(v.pipe(v.string(), v.email()))
```

### Nullable (null allowed)
```typescript
// Zod
z.string().nullable()        // string | null

// Valibot
v.nullable(v.string())
```

### Default Value
```typescript
// Zod
z.string().default('active')

// Valibot
v.optional(v.string(), 'active')  // Note: provides default + optional

// For just default without optional
v.pipe(
  v.string(),
  v.transform(val => val || 'default')
)
```

### Combined (optional AND default)
```typescript
// Zod
z.string().optional().default('unknown')

// Valibot
v.optional(v.string(), 'unknown')
```

### Catch (fallback on error)
```typescript
// Zod
z.string().catch('fallback')   // On parse error, use fallback

// Valibot
// Not built-in; use custom validator or post-parse handling
```

---

## Union, Intersection, Discriminated Union

### Union (OR logic)
```typescript
// Zod
z.union([z.string(), z.number()])
z.string().or(z.number())      // shorthand

// Valibot
v.union([v.string(), v.number()])

// Type inference
z.infer<typeof schema>  // string | number
  ↓
v.infer<typeof schema>  // string | number
```

### Intersection (AND logic)
```typescript
// Zod
z.intersection(schemaA, schemaB)
schemaA.and(schemaB)

// Valibot
// Not built-in; merge objects manually or use custom validator
```

### Discriminated Union
```typescript
// Zod
z.discriminatedUnion('type', [
  z.object({ type: z.literal('a'), valA: z.string() }),
  z.object({ type: z.literal('b'), valB: z.number() })
])

// Valibot (regular union, no optimization)
v.union([
  v.object({ type: v.literal('a'), valA: v.string() }),
  v.object({ type: v.literal('b'), valB: v.number() })
])
```

---

## Custom Validation & Refinement

### Simple Custom Validation
```typescript
// Zod
z.string().refine(val => val.length > 5, {
  message: "String must be > 5 chars"
})

// Valibot
v.pipe(
  v.string(),
  v.check(val => val.length > 5, 'String must be > 5 chars')
)
```

### Cross-Field Validation
```typescript
// Zod
z.object({ password: z.string(), confirm: z.string() })
  .refine(data => data.password === data.confirm, {
    message: "Passwords don't match",
    path: ["confirm"]
  })

// Valibot
v.pipe(
  v.object({ password: v.string(), confirm: v.string() }),
  v.check(
    data => data.password === data.confirm,
    'Passwords must match'
  )
)

// Note: Valibot doesn't support path specification in simple check
```

### Async Custom Validation
```typescript
// Zod
z.string().email().refine(
  async email => !(await emailExists(email)),
  { message: 'Email already registered' }
)

// Valibot
v.pipe(
  v.string(),
  v.email(),
  v.checkAsync(
    async email => !(await emailExists(email)),
    'Email already registered'
  )
)
```

### Transform During Validation
```typescript
// Zod
z.string().transform(val => val.toUpperCase())

// Valibot
v.transform(val => val.toUpperCase())

// In context
z.object({
  name: z.string().transform(s => s.trim().toUpperCase())
})
  ↓
v.object({
  name: v.pipe(
    v.string(),
    v.transform(s => s.trim().toUpperCase())
  )
})
```

---

## Error Handling & Parsing

### Safe Parse (Non-throwing)
```typescript
// Zod
const result = schema.safeParse(data);
if (!result.success) {
  console.log(result.error.errors);
  console.log(result.error.flatten());
}

// Valibot
const result = v.safeParse(schema, data);
if (!result.success) {
  console.log(result.issues);
  // ValiError[] with structure:
  // { code: string, message: string, input: unknown, path?: (string|number)[] }
}
```

### Direct Parse (Throwing)
```typescript
// Zod
try {
  const data = schema.parse(input);
} catch (error) {
  console.log(error.errors);
}

// Valibot
try {
  const data = v.parse(schema, input);
} catch (error) {
  console.log(error.issues);
}
```

### Error Structure Comparison

**Zod Error:**
```typescript
{
  code: 'invalid_type',
  expected: 'string',
  received: 'number',
  path: ['email'],
  message: 'Expected string, received number'
}
```

**Valibot Error:**
```typescript
{
  code: 'type',
  message: 'Invalid type. Expected string.',
  input: 123,
  path: [{ key: 'email', schema: StringSchema }]
}
```

### Error Formatting

**Zod:**
```typescript
const formatted = result.error.flatten();
// Output: { fieldErrors: { email: [...] }, formErrors: [...] }
```

**Valibot:**
```typescript
const formatted = result.issues.reduce((acc, issue) => {
  const path = issue.path?.map(p => p.key).join('.') || 'root';
  (acc[path] ||= []).push(issue.message);
  return acc;
}, {});
```

---

## Performance & Bundle Size Comparison

### Bundle Size Impact (Single Schema Example)
```typescript
// Login form schema

Zod standard:     17.7 KB
Zod mini:          9.5 KB  (-46%)
Valibot:            1.37 KB (-92% vs standard, -86% vs mini)

// Tree-shaking effectiveness
Valibot: Removes unused validators (90%+ reduction possible)
Zod: Limited tree-shaking (uses static properties)
```

### Runtime Performance (1M iterations, complex nested schema)
```
Valibot (latest):  ~150ms
Zod v4:            ~160ms
Zod v3:            ~300ms
```

### Recommendations
- **Bundle-critical (frontend, edge):** Valibot
- **Performance-critical (API, bulk ops):** Valibot or Zod v4
- **Developer experience:** Zod (method chaining more intuitive)
- **Ecosystem integration:** Zod (tRPC, Astro, Next.js)

---

## Migration Checklist

When converting a schema:

- [ ] Replace `z.` with `v.` for types
- [ ] Wrap chained validators in `v.pipe()`
- [ ] Update string methods: `.min()` → `v.minLength()`
- [ ] Update number methods: `.min()` → `v.minValue()`
- [ ] Update `.enum()` → `v.picklist()`
- [ ] Update `.default()` → `v.optional(schema, default)`
- [ ] Update type inference: `z.infer` → `v.infer`
- [ ] Update parse calls: `schema.safeParse()` → `v.safeParse(schema, data)`
- [ ] Update error handling: `.error` → `.issues`
- [ ] Update `.refine()` → `v.check()` or `v.checkAsync()`
- [ ] Update `.transform()` inline in pipes
- [ ] Test all validation paths
- [ ] Check ecosystem compatibility

---

## Useful Resources

- [Valibot Official Comparison](https://valibot.dev/guides/comparison/)
- [Valibot Documentation](https://valibot.dev/)
- [Zod Documentation](https://zod.dev/)
- [Valibot GitHub](https://github.com/fabian-hiller/valibot)
- [Zod GitHub](https://github.com/colinhacks/zod)

---

**Last Updated:** 2026-05-25  
**Valibot Version:** 1.3.1  
**Zod Version:** 3.22.x (latest)
