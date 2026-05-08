# Architecture & Flow Notes

---

## 🏛️ Clean Architecture Layers — What Goes Where

```
UI (View)
  ↓ reads from
Controller (ChangeNotifier)
  ↓ calls
UseCase
  ↓ calls
Repo (abstract)
  ↓ calls
DataSource (abstract)
  ↓ calls
Supabase / External API
```

## 🔐 Auth Flow — After Sign In

### What happens step by step:

```
1. User signs in (email + password)
2. Supabase fires an auth state change event
3. AuthController picks it up via Stream
4. AuthController fetches current user → gets userId
5. AuthController fetches profile using userId
6. AuthController notifies listeners (UI reacts)
```

### Layer breakdown:

**DataSource** — only talks to Supabase

```dart
// Single record by primary key
Future<Map<String, dynamic>> fetchSingleById({
  required String tableName,
  required String id,
});
```

**Repo** — only talks to DataSource, returns `Either<Failure, Entity>`
```dart
// ProfileRepoImpl
Future<Either<Failure, ProfileEntity>> getProfile(String userId) async {
  final data = await supabaseDataSource.fetchSingleById(
    tableName: kSupaProfilesTable,
    id: userId,
  );
  return Right(ProfileEntity.fromMap(data));
}
```

**UseCase** — wraps repo, exposes clean interface to controller
```dart
// CheckAuthStatusUseCase
Stream<AuthStatus> execute() {
  return authRepo.authStateChanges().map((user) =>
    user != null ? AuthStatus.authenticated : AuthStatus.unauthenticated
  );
}
```

**Controller** — listens to stream, orchestrates fetching, holds state
```dart
class AuthController extends ChangeNotifier {
  AuthStatus _status = AuthStatus.unknown;
  ProfileEntity? _currentProfile;
  String? _userId;

  void _listenToAuthChanges() {
    _checkAuthStatusUseCase.execute().listen((status) async {
      _status = status;

      if (status == AuthStatus.authenticated) {
        await _onAuthenticated();
      } else {
        _onUnauthenticated();
      }

      notifyListeners(); // always once, at the end
    });
  }

  Future<void> _onAuthenticated() async {
    final user = await _checkAuthStatusUseCase.authRepo.getCurrentUser();
    if (user == null) return;

    _userId = user.uId;

    final result = await _profileRepo.getProfile(user.uId);
    result.fold(
      (failure) => debugPrint('Profile load failed: $failure'),
      (profile) => _currentProfile = profile,
    );
  }

  void _onUnauthenticated() {
    _userId = null;
    _currentProfile = null;
  }
}
```

**View** — reads from controller only, never calls repos or usecases directly
```dart
final controller = context.watch<AuthController>();

if (controller.isUnknown) return SplashScreen();
if (controller.isAuthenticated) return HomeScreen();
return LoginScreen();
```

---

## ❌ Mistake: Using globals when a top-level Controller exists
**Date:** 2026-05-07

### What I did wrong
Used `globalUserId` and `globalProfile` variables to share auth state across the app.

```dart
// ❌ Wrong
globalUserId = currentUser.uId;
globalProfile = _currentProfile;
```

### Why it's wrong
`AuthController` is already provided at the top of the widget tree. It IS the global state. Duplicating it into plain globals creates two sources of truth that can go out of sync.

### ✅ Correct way
Access the controller anywhere in the widget tree:
```dart
final userId = context.read<AuthController>().userId;
final profile = context.read<AuthController>().currentProfile;
```

Or if you need it outside the widget tree (e.g. inside a repo), pass the userId as a parameter instead of using a global.

### 💡 Rule to remember
> If you have a top-level ChangeNotifier, it is your global state. No separate global variables needed.

---
