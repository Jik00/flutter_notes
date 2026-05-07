# Supabase Flutter Notes

---

## ❌ Mistake: Conditional query chaining after await
**Date:** 2026-05-07

### What I did wrong
Tried to conditionally add `.eq()` filters **after** the query was already awaited, meaning the response was already a `List<Map<String, dynamic>>` — a plain Dart list, not a Supabase query builder.

```dart
// ❌ Wrong
var response = await supabase
    .from(tableName)
    .select()
    .eq(query, value)
    .order('created_at', ascending: true);

if (query2 != null && value2 != null) {
  response = response.eq(query2, value2); // ❌ List has no .eq()
}
```

### Why it's wrong
`await` executes the query immediately and returns a `List`. You can no longer chain Supabase builder methods on a `List`.

### ✅ Correct way
Build the full query **before** awaiting it. Store the builder in a variable, chain conditionally, then await at the end.

```dart
// ✅ Correct
var dbQuery = supabase
    .from(tableName)
    .select()
    .eq(query, value);

if (query2 != null && value2 != null) {
  dbQuery = dbQuery.eq(query2, value2); // ✅ Still a builder, not a List
}

final response = await dbQuery.order('created_at', ascending: true);
```

### 💡 Rule to remember
> **Build first, await last.** Never chain Supabase query methods after `await` — always construct the full query before executing it.

---
