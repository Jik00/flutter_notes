# Dart Language Notes

_Mistakes and lessons related to Dart syntax, types, and language features._

---

Value Types vs Reference Types

## The core difference

Primitive | `int`, `double`, `bool`, `String` | Copied **by value** — independent copy

Object | Any class instance | Copied **by reference** — same object in memory 

---

## Primitives — copied by value

```dart
int x = 2;
final y = x; // y gets its OWN copy of 2
x = 3;
print(x + y); // 5 — y is still 2 ✅
```

`y` and `x` are completely independent. Changing one does **not** affect the other.

---

## Objects — copied by reference

```dart
AllCartEntity a = AllCartEntity(items: [apple]);
final b = a; // b points to the SAME object in memory

a.addItem(banana); // mutates the shared object
// b now also sees banana ❌
```

Both variables hold the **same memory address**, not a copy of the data.


## Why this matters for optimistic updates (rollback)

```dart
// ❌ Mutable — rollback is impossible
final previousCart = allCartEntity; // both point to 0x1A
allCartEntity.addCartItem(item);    // mutates 0x1A
// previousCart is now also changed — snapshot is lost!

// ✅ Immutable with copyWith — rollback works
final previousCart = allCartEntity;          // previousCart → 0x1A
allCartEntity = allCartEntity.addCartItem(item); // new object at 0x2B
// previousCart still → 0x1A (untouched) ✅
allCartEntity = previousCart; // rollback works perfectly
```

## Key rule

> `final` on a variable means the **reference** can't change — not the object it points to.
> Immutability (via `copyWith`) ensures any "change" produces a **new object** rather than mutating the existing one.

---

*Related: Equatable requires immutable fields. Optimistic update pattern requires immutable state for safe rollback.*


## Making a class immutable in Dart

```dart
class AllCartEntity {
  final List<CartItemEntity> cartItems;

  const AllCartEntity({required this.cartItems});

  // Returns a NEW object — never mutates self
  AllCartEntity addCartItem(CartItemEntity item) =>
      copyWith(cartItems: [...cartItems, item]);

  AllCartEntity removeCartItem(CartItemEntity item) =>
      copyWith(cartItems: cartItems.where((e) => e != item).toList());

  AllCartEntity copyWith({List<CartItemEntity>? cartItems}) =>
      AllCartEntity(cartItems: cartItems ?? this.cartItems);
}
```
