# Kotlin and Android checklist

Applies to changed `*.kt` and `*.kts` files. The repository's rules win; architecture is whatever
the repository chose, so flag a layering problem only against its own stated layout.

## CRITICAL

- Exported Activity, Service or Receiver, or a deep link or intent filter, that acts on intent
  data without checking it.
- Tokens, keys or credentials in source, `SharedPreferences`, logs or `BuildConfig`.
- WebView with JavaScript enabled loading content that is not fully trusted; cleartext traffic
  allowed outside debug builds.
- Catching `CancellationException` (directly or through `Exception`/`Throwable`) without rethrowing.

## HIGH

- `GlobalScope` or an unscoped `CoroutineScope`; work that outlives its screen.
- Disk, database or network on `Dispatchers.Main`.
- Flows collected in an Activity or Fragment without `repeatOnLifecycle`.
- A mutable collection inside `StateFlow`/`MutableState` mutated in place (no new value, no update).
- Side effects (network, database, navigation) in a composable body instead of `LaunchedEffect`
  or the ViewModel.
- `remember` missing a key the computation depends on; `LazyColumn` items without stable keys
  when the list reorders.
- An `Activity`, `Fragment` or `View` held by a singleton or ViewModel.

## MEDIUM

- `!!` on a value that can be null at runtime; prefer `?:`, `requireNotNull` with a message.
- `when` over a sealed type with an `else` branch that hides new subtypes.
- `MutableList`/`MutableMap` returned from a public API.
- User-facing strings hardcoded instead of resources.
- Serialized classes missing keep rules for release builds with minification.
