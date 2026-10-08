# Rust checklist

Applies to changed `*.rs` files. The repository's rules win (for example Stella Rain `core`'s
determinism rules); clippy and fmt are gates, so do not repeat what they enforce.

## CRITICAL

- `unwrap()`, `expect()`, `panic!`, `todo!`, `unreachable!` on a path reachable from input,
  files, network or another process. Fine in tests, build scripts and proven invariants with a comment.
- `unsafe` without a `// SAFETY:` comment stating the invariant, or a comment the code does not uphold.
- `std::process::Command` or a shell built from input; file paths from input without
  canonicalising and checking the prefix.
- Deserialising untrusted data (save files, shared stages, network) without size or depth limits.
- `let _ = <Result>` or `.ok()` that silently drops an error the caller needed.

## HIGH

- Blocking calls (`std::thread::sleep`, `std::fs`, a blocking lock held across `.await`) inside async code.
- Unbounded channels or queues fed by input, without a stated reason.
- Lock order that differs between two call sites (deadlock); `.lock().unwrap()` where a poisoned
  lock must be survivable.
- `_ =>` on an enum the project owns, hiding variants added later.
- Error returned without context (`map_err`, `context`), so the log cannot say which file or input failed.
- Iteration over `HashMap`/`HashSet` where the order reaches output, saved data or a replay.

## MEDIUM

- `.clone()` added only to satisfy the borrow checker, on a hot path or a large value.
- Allocating inside a loop that runs per frame or per item when it could be hoisted.
- `#[allow(...)]` without a comment saying why.
- `pub` item added without `///` docs in a library crate.

Report these only with a cited line and a concrete cost; style alone is not a finding.
