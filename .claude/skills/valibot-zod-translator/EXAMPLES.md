# Valibot ↔ Zod Translation Examples

## Example 1: Simple Form Validation

### Zod Version
```typescript
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email('Invalid email').min(1, 'Email required'),
  password: z.string().min(8, 'Password must be 8+ chars').max(100),
  rememberMe: z.boolean().default(false)
});

type LoginForm = z.infer<typeof loginSchema>;
```

### Valibot Version
```typescript
import * as v from 'valibot';

const loginSchema = v.object({
  email: v.pipe(
    v.string('Email is required'),
    v.minLength(1, 'Email required'),
    v.email('Invalid email')
  ),
  password: v.pipe(
    v.string(),
    v.minLength(8, 'Password must be 8+ chars'),
    v.maxLength(100)
  ),
  rememberMe: v.optional(v.boolean(), false)
});

type LoginForm = v.infer<typeof loginSchema>;
```

**Key Changes:**
- Zod chains → Valibot pipes
- `.default()` → `v.optional(schema, defaultValue)`
- `.infer<typeof>` syntax identical, just different import
- Error messages passed inline in each validator

---

## Example 2: Nested Objects with Arrays

### Zod Version
```typescript
const userSchema = z.object({
  id: z.string().uuid(),
  name: z.string(),
  email: z.string().email(),
  profile: z.object({
    bio: z.string().max(500),
    avatar: z.string().url()
  }),
  tags: z.array(z.string()),
  metadata: z.record(z.any())
});
```

### Valibot Version
```typescript
const userSchema = v.object({
  id: v.pipe(v.string(), v.uuid()),
  name: v.string(),
  email: v.pipe(v.string(), v.email()),
  profile: v.object({
    bio: v.pipe(v.string(), v.maxLength(500)),
    avatar: v.pipe(v.string(), v.url())
  }),
  tags: v.array(v.string()),
  metadata: v.record(v.string(), v.any())
});
```

**Key Changes:**
- Nested objects use `v.object()` (same syntax)
- Each field wraps validators in `v.pipe()` if multiple validations
- `z.array()` ↔ `v.array()` (same concept)
- `z.record()` ↔ `v.record()` with explicit key/value types

---

## Example 3: Discriminated Union

### Zod Version
```typescript
const eventSchema = z.discriminatedUnion('type', [
  z.object({
    type: z.literal('click'),
    x: z.number(),
    y: z.number()
  }),
  z.object({
    type: z.literal('scroll'),
    amount: z.number()
  })
]);
```

### Valibot Version
```typescript
const eventSchema = v.union([
  v.object({
    type: v.literal('click'),
    x: v.number(),
    y: v.number()
  }),
  v.object({
    type: v.literal('scroll'),
    amount: v.number()
  })
]);
```

**Key Changes:**
- Zod's `.discriminatedUnion()` → Valibot's `v.union()` (less optimized but functionally identical)
- Valibot doesn't have built-in discriminated union optimization
- Both work the same at runtime

---

## Example 4: Custom Validator (Refine)

### Zod Version
```typescript
const passwordSchema = z.object({
  password: z.string().min(8),
  confirm: z.string()
}).refine(data => data.password === data.confirm, {
  message: "Passwords don't match",
  path: ["confirm"]
});
```

### Valibot Version
```typescript
const passwordSchema = v.object({
  password: v.pipe(v.string(), v.minLength(8)),
  confirm: v.string()
});

// Option 1: Custom validation
const passwordSchemaWithValidation = v.pipe(
  passwordSchema,
  v.custom(data => data.password === data.confirm, 'Passwords must match')
);

// Option 2: More explicit with issue
const passwordSchemaWithValidation = v.pipe(
  passwordSchema,
  v.check(
    data => data.password === data.confirm,
    'Passwords must match'
  )
);
```

**Key Changes:**
- Zod's `.refine()` → Valibot's `v.custom()` or `v.check()`
- Valibot wraps the entire object in an additional `v.pipe()`
- Different approach to cross-field validation

---

## Example 5: Async Validation

### Zod Version
```typescript
const emailSchema = z.string().email().refine(
  async (email) => {
    const exists = await checkEmailExists(email);
    return !exists;
  },
  { message: 'Email already registered' }
);
```

### Valibot Version
```typescript
const emailSchema = v.pipe(
  v.string(),
  v.email(),
  v.checkAsync(
    async (email) => {
      const exists = await checkEmailExists(email);
      return !exists;
    },
    'Email already registered'
  )
);
```

**Key Changes:**
- Zod's `.refine(async)` → Valibot's `v.checkAsync()`
- Valibot distinguishes sync (`v.check`) from async (`v.checkAsync`)
- Error handling in async validation slightly different

---

## Example 6: Transform & Type Coercion

### Zod Version
```typescript
const userInput = z.object({
  age: z.string().transform(val => parseInt(val, 10)).pipe(z.number().min(0)),
  email: z.string().transform(val => val.toLowerCase()),
  active: z.enum(['yes', 'no']).transform(val => val === 'yes')
});
```

### Valibot Version
```typescript
const userInput = v.object({
  age: v.pipe(
    v.string(),
    v.transform(val => parseInt(val, 10)),
    v.number(),
    v.minValue(0)
  ),
  email: v.pipe(
    v.string(),
    v.transform(val => val.toLowerCase())
  ),
  active: v.pipe(
    v.picklist(['yes', 'no']),
    v.transform(val => val === 'yes')
  )
});
```

**Key Changes:**
- Zod's `.transform()` ↔ Valibot's `v.transform()` (same idea)
- Zod's `.enum()` ↔ Valibot's `v.picklist()`
- Valibot allows transforms inline in pipes
- Order matters: transform → validate converted type

---

## Example 7: Parsing & Error Handling

### Zod Version
```typescript
const schema = z.object({
  name: z.string(),
  age: z.number()
});

// Safe parse (doesn't throw)
const result = schema.safeParse(data);
if (!result.success) {
  console.log(result.error.flatten());
  // Output: { fieldErrors: { ... }, formErrors: [...] }
}

// Direct parse (throws)
try {
  const validated = schema.parse(data);
} catch (error) {
  console.log(error.errors);
}
```

### Valibot Version
```typescript
const schema = v.object({
  name: v.string(),
  age: v.number()
});

// Safe parse (doesn't throw)
const result = v.safeParse(schema, data);
if (!result.success) {
  console.log(result.issues);
  // Output: ValiError[] with path, message, code
}

// Direct parse (throws)
try {
  const validated = v.parse(schema, data);
} catch (error) {
  console.log(error.issues);
}
```

**Key Changes:**
- `schema.safeParse()` ↔ `v.safeParse(schema, data)` (reversed argument order!)
- Error structure: Zod's `error.flatten()` vs Valibot's `issues` array
- Different error metadata structure
- Valibot passes schema as first argument

---

## Example 8: Conditional Fields (Depends)

### Zod Version
```typescript
const formSchema = z.object({
  type: z.enum(['business', 'personal']),
  companyName: z.string().optional(),
}).refine(
  data => data.type !== 'business' || !!data.companyName,
  { message: 'Company name required for business type', path: ['companyName'] }
);
```

### Valibot Version
```typescript
const formSchema = v.pipe(
  v.object({
    type: v.picklist(['business', 'personal']),
    companyName: v.optional(v.string())
  }),
  v.check(
    data => data.type !== 'business' || !!data.companyName,
    'Company name required for business type'
  )
);
```

**Key Changes:**
- Both use post-validation checks for conditional logic
- Valibot doesn't have `path` specification in check messages
- Consider restructuring with discriminated unions for cleaner logic

---

## Bulk Migration Patterns

### Pattern 1: Replace All Chained Methods

**Find & Replace Regex (Zod → Valibot):**
```
z\.(\w+)\(\)\.(\w+)\(([^)]*)\)\.(\w+)\(([^)]*)\)
→
v\.pipe(v.$1(), v.$2($3), v.$4($5))
```

⚠️ This is a starting point; manual refinement needed.

### Pattern 2: Update Type Inference

**Find:**
```
z\.infer<typeof (\w+)>
```

**Replace:**
```
v.infer<typeof $1>
```

### Pattern 3: Update Parse Calls

**Find:**
```
(\w+)\.safeParse\((\w+)\)
```

**Replace:**
```
v.safeParse($1, $2)
```

---

## Testing After Migration

```typescript
// Create test cases for both before/after
const testData = [
  { valid: { email: 'test@example.com', age: 25 }, expect: 'pass' },
  { valid: { email: 'invalid', age: 17 }, expect: 'fail' }
];

const zodSchema = z.object({ email: z.string().email(), age: z.number().min(18) });
const valibotSchema = v.object({
  email: v.pipe(v.string(), v.email()),
  age: v.pipe(v.number(), v.minValue(18))
});

testData.forEach(({ valid, expect }) => {
  const zodResult = zodSchema.safeParse(valid);
  const valibotResult = v.safeParse(valibotSchema, valid);
  
  assert(zodResult.success === valibotResult.success, 
    `Mismatch for ${JSON.stringify(valid)}`);
});
```
