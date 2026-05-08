# Async Dart Notes

_Mistakes and lessons related to Future, async/await, Stream, and concurrency._

---

## 📖 Concept: Completer
**Date:** 2026-05-07

A `Future` that you complete manually — you decide when it's done.

```dart
// Normal Future — completes on its own
Future.delayed(Duration(seconds: 2));

// Completer — YOU decide when it completes
final completer = Completer<void>();
completer.future; // waiting...
completer.complete(); // ← you fire this, now it's done
```

### Real use case — wait for a stream to emit a non-unknown value:
```dart
Future<void> _waitForAuthResolution() async {
  final completer = Completer<void>();

  void listener() {
    if (!_authController.isUnknown) {
      _authController.removeListener(listener);
      completer.complete(); // auth is ready, stop waiting
    }
  }

  _authController.addListener(listener);
  return completer.future; // await here until complete() is called
}
```

### 💡 Rule to remember
> Use `Completer` when you need to wait for something that isn't naturally a `Future` — like a listener, a callback, or an external event.

---

<!-- New entries go here -->