# GetIt Notes

_Patterns and lessons related to GetIt service locator and injectable._

---

## 📖 Concept: factoryParam — injecting runtime values into GetIt
**Date:** 2026-05-07

Use `factoryParam` when a class needs both GetIt-registered dependencies **and** a runtime value (like `userId`) that's only available later (e.g. after auth).

### Registration
```dart
// GetIt resolves the repo, you pass userId at runtime
getIt.factoryParam<SomeCubit, String, dynamic>(
  (userId, _) => SomeCubit(
    userId: userId,
    someRepo: GetIt.I.get<SomeRepo>(), // resolved internally by GetIt
  ),
);
```

### Usage in widget tree
```dart
BlocProvider(
  create: (context) => GetIt.I.get<SomeCubit>(
    param1: context.read<AuthController>().userId,
  ),
  child: SomeView(),
)
```

### How params work
| Param | What it is |
|---|---|
| `param1` | First runtime value you pass (e.g. userId) |
| `param2` | Second runtime value if needed, otherwise pass `dynamic` and ignore with `_` |

### 💡 Rule to remember
> Use `factoryParam` when GetIt can't know a value at registration time. GetIt handles the deps it knows about, the widget tree provides the rest via `context`.

---

<!-- New entries go here -->
