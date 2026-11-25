# Rust Coding Conventions

## 0. Scope and General Principles

These conventions govern all first-party Rust in the codebase (libraries, binaries, FFI and WebAssembly shims, build scripts and tests). One codebase follows one style, even when it spans several repositories and languages; where a rule has a C++, Python, Swift or TypeScript counterpart, the two read the same (see [C++ Coding Conventions](cpp-coding-conventions.md), [Python Coding Conventions](python-coding-conventions.md), [Swift Coding Conventions](swift-coding-conventions.md) and [TypeScript Coding Conventions](typescript-coding-conventions.md)).

- **Third-party code is out of scope.** Vendored crates and generated code (bindgen output, protocol bindings) must not be reformatted, so they can be regenerated or re-synced with upstream.
- **These rules win over local habit.** Where an older file's own style disagrees, follow this document (see [Section 7](#7-changing-code-that-predates-these-rules)).
- **The compiler and Clippy are the first reviewers.** Every crate builds on stable Rust with `cargo clippy --all-targets -- -D warnings` clean, and the lints that enforce rules below are switched on in the workspace's `[workspace.lints]` table rather than left to review; a rule below never stands in for a lint that could enforce it.
- **Naming follows the Rust API Guidelines** (`snake_case` items, `UpperCamelCase` types, `as_` / `to_` / `into_` conversions), with the house additions in [Section 5](#5-naming).
- Names such as `ProjectError` and `Registry` are placeholders; substitute the project's own types.

```toml
[workspace.lints.rust]
unsafe_code = "deny"

[workspace.lints.clippy]
unwrap_used = "deny"
expect_used = "deny"
panic = "deny"
```

Tests may unwrap, expect and panic: say so once in `clippy.toml` (`allow-unwrap-in-tests`, `allow-expect-in-tests`, `allow-panic-in-tests`), and with a file-level `#![allow(...)]` in integration test files, whose helpers are not `#[test]` functions.

## 1. Comments and Doc Comments

A comment states business or algorithm semantics, never language mechanics.

- **Domain semantics only.** Say what the code does in domain terms. Do not explain language mechanics ("borrow here so the vector is not moved"), a style rule, or why you chose a construct.
- **No multi-line prose inside function bodies.** A one-line `//` label for a step is fine; a run of two or three `//` lines explaining a branch chain is not. When in doubt, delete it: self-explanatory code needs no prose.
- **No file header comment.** A module file begins directly with its `use` items: no license banner, `// file.rs` line or module description. The crate root (`lib.rs`, `main.rs`) is the one exception: it may open with a short `//!` crate doc, which rustdoc renders as the crate's front page.
- **The doc comment is the declaration.** Every `pub` and `pub(crate)` item (type, function, method, field, variant, constant) carries a `///` doc comment; it may span several lines. Use rustdoc's headings: `# Arguments` when a parameter's meaning is not plain from its name and type, `# Returns` when the result needs saying, and always `# Errors` on a function returning `Result`, `# Panics` on one that can panic, `# Safety` on an `unsafe fn`. Private helpers carry a one-line comment or none.
- **Types live in the signature.** The doc comment does not repeat a parameter's type.
- **No banner comments.** No `// ---- helpers ----` or `// MARK:` scaffolding; a file whose structure needs signposts wants splitting into modules.

```rust
/// Look up a tracked window by its window-server identifier.
///
/// # Returns
///
/// The window, or `None` if it is not tracked.
pub fn window(&self, window_id: WindowId) -> Option<&TrackedWindow> {
```

## 2. Error Handling: Result and a Single Cleanup Path

Failure is signalled by returning `Err`, and every resource is released on one path.

- **Return `Result`, not status values.** A function that can fail returns `Result<T, E>`; it never returns `false`, `-1` or `None` to mean "failed". `Option` means "absent", not "error".
- **Errors are typed enums.** Each module that can fail declares its error as an `enum` deriving `Debug` and `thiserror::Error`, with fields carrying the context (`AxCallFailed { attribute: String, status: i32 }`). A wrapped cause is kept with `#[source]` or `#[from]`, never flattened into a string. Libraries never return `Box<dyn Error>` or `anyhow::Error`; a binary's `main` may collect errors into one report type at the very top.
- **Propagate with `?`.** Convert between error types with `From` impls or `map_err` at the call that crosses a module boundary, adding the context the caller lacks; do not match and re-wrap by hand where `?` does it.
- **Convert at the boundary.** Where a system or C API reports failure through a status code, an errno or an out-parameter, the wrapper checks it once and returns `Err`; callers above the wrapper never see the raw status.
- **No `unwrap`, no `expect` and no `panic!` outside tests.** Replace them with `?`, an explicit `Err`, or a handled `None`. The one exception is an invariant the code has just proven (an index taken from the same slice, a regex literal compiled once): there `expect("...")` is allowed with a message stating the invariant, and the `#[allow(clippy::expect_used)]` sits on that line alone with a one-line comment naming the proof.
- **Never swallow a failure.** `let _ = fallible();`, `.ok()` and `.unwrap_or_default()` that discard an `Err` are allowed only where "failed" and "absent" genuinely mean the same thing to the caller, and the call site makes that obvious. A `match` on an error handles specific variants and either recovers meaningfully or returns it with context; no arm that logs and carries on.
- **One cleanup path: `Drop`.** A resource is owned by a value whose `Drop` releases it (a file, a lock guard, a temporary directory, an FFI handle wrapped in its own type). Cleanup that has no owning type gets a small guard type with a `Drop` impl, acquired right after the resource; never duplicate cleanup before several `return`s or `?`s.

```rust
let file = File::open(&path).map_err(|source| TraceError::Open { path: path.clone(), source })?;
let list_records = TraceReader::new(file)
    .read_all()
    .map_err(|source| TraceError::Malformed { path, source })?;
Ok(list_records)
```

## 3. Types and Variables

- **`let` by default, `let mut` only when the binding is mutated.**
- **Value types first.** Records are `struct`s with named fields; shared ownership (`Rc`, `Arc`) is used only where several owners are real, not to quiet the borrow checker.
- **Records are named types, never tuples.** A function returning two values returns a `struct` with named fields; tuples are acceptable only as transient locals inside one expression or as the item of an iterator chain.
- **Model states with enums with fields,** not with several `Option`s or boolean flags that may contradict each other. Match them exhaustively; a `_ =>` arm on a first-party enum hides the next variant someone adds.
- **A binding must not shadow an outer local.** `for (window_id, frame) in &map_frames` in a function that already holds `window_id` silently hides it; name the binding for what it holds. Re-binding the same value under the same name as it changes form (`let path = path?;`, `let count: usize = count.parse()?;`, `if let Some(value) = value`) is the one allowed form.
- **No lossy `as` casts.** Convert numbers with `From` / `TryFrom` (`u32::try_from(length)?`); `as` is allowed only where the conversion cannot lose a value and no `From` impl exists.
- **`usize` for indices and counts.** Use fixed-width integers (`u32`, `i64`) only where an external ABI or wire format dictates them, and wrap such values in a newtype at the boundary (`struct WindowId(u32)`).
- **Borrow in parameters.** Take `&str`, `&[T]` and `&T` rather than `String`, `Vec<T>` or `T` unless the function keeps the value; `.clone()` is a decision about ownership, not a fix for a borrow error.
- **`unsafe` is fenced and justified.** A crate that needs no `unsafe` keeps the workspace's `unsafe_code = "deny"`; one that does (an FFI shim) allows it for that crate alone. Every `unsafe` block is as small as the operation it covers and is preceded by a `// SAFETY:` comment naming the invariant that makes it sound; every `unsafe fn` documents it under `# Safety`.

## 4. Multi-line Formatting

`rustfmt`, with its default settings, is the formatter, and `cargo fmt --check` passes on every commit. Do not fight it with `#[rustfmt::skip]`; shape the code so its output reads well.

**Calls:** a call that fits on one line stays there; once rustfmt expands it, each argument sits on its own line with a trailing comma. Rust has no argument labels, so **a function that takes more than two parameters besides `self` takes a single options struct instead**, and every call site names its values.

```rust
let state = model.state_at(StateQuery {
    position,
    num_items,
    map_live: &map_live,
    list_free: &list_free,
});
```

**Positional calls into external APIs:** where a callee the codebase does not own takes positional arguments (C functions over FFI, a foreign crate's constructor), each argument carries a leading `/* parameter-name */` label naming the parameter it binds. rustfmt keeps the labels but not alignment, so they are not aligned.

```rust
let error = AXUIElementCopyAttributeValue(
    /* element */ element,
    /* attribute */ attribute.as_concrete_TypeRef(),
    /* value */ &mut value,
);
```

**Long signatures:** rustfmt puts one parameter per line with a trailing comma; past two parameters besides `self`, take an options struct (above).

**Multi-field construction:** one field per line, by field name, using field-init shorthand where the local has the field's name.

```rust
list_windows.push(WindowSpan {
    producer: node_index,
    start_time: list_start_times[node_index],
    end_time,
});
```

**Closures passed as arguments:** a one-expression closure stays inline; anything longer becomes a named method or associated function.

```rust
list_windows.sort_by_key(|window| window.start_time);
```

**Imports:** one contiguous block of `use` items at the top of the file, with no blank lines between `std`, third-party and first-party paths, so rustfmt sorts it as one block. Merge paths of one crate into one `use` with braces. No glob imports (`use module::*`) outside a test module's `use super::*`.

```rust
use crate::diagnostic::Diagnostic;
use serde::Serialize;
use std::collections::BTreeMap;
```

**Multi-condition `if` / `while`:** when rustfmt breaks a condition, the operator leads each continuation line; do not hand-wrap it otherwise. A condition with more than three terms becomes a named predicate method.

```rust
if list_nodes[node_index].is_variable
    && !list_nodes[node_index].list_successors.is_empty()
{
```

## 5. Naming

A name tells the reader its type, visibility and purpose at a glance.

| Item | Rule | Example |
|---|---|---|
| `HashMap` / `BTreeMap` / its iterator | prefix `map_` / suffix `_iter` | `map_free_slots`, `map_by_index_iter` |
| `Vec` / slice / array / `VecDeque` | prefix `list_` | `list_nodes` |
| `HashSet` / `BTreeSet` | prefix `set_` | `set_live_items` |
| Private members and helpers | no `pub`, no prefix | `fn emit_header(&self)` |
| Constants | `<CATEGORY>_<ROLE>`, upper snake, associated `const` on the owning type | `Register::REGISTER_ZERO`, `Buffer::BUFFER_MAX_SIZE` |
| Booleans | read as predicates | `is_virtual`, `should_write` |
| Indices / IDs | `usize`, absent is `None` (`Option<usize>`) | `node_index: usize` |
| Types, traits, enum variants | `UpperCamelCase`, no `I` / `T` prefix | `TraceReader`, `WindowSource` |
| Lifetimes | short and lower case; named for what they borrow once there are two | `'a`, `'src` |
| Everything else | `snake_case` | `state_at` |

- **The container prefix is mandatory on locals, fields and parameters alike.** `new_list`, `by_index` and `inserted` are containers that read as scalars; write `list_new_items`, `map_by_index`, `set_inserted`.
- **No leading underscore on used names.** In Rust `_name` tells the compiler and the reader the binding is unused; privacy is expressed by leaving out `pub`.
- **No first-party method starts with `set_`.** The prefix means a `HashSet` here; a method that writes state is named for its effect (`apply_frame`, `move_to`), or the field is made visible to the module that writes it.
- **Getters take no `get_` prefix,** per the API Guidelines: `fn name(&self) -> &str`. Fallible lookups that might be confused with the getter read as questions or sources (`find_task`, `task_named`).
- **Acronyms are one word in `UpperCamelCase`** (`HttpServer`, `RookbinFormat`, `AxElement`) and lower case in `snake_case` (`window_id`, `ax_element`).
- **Descriptive full words, not terse abbreviations.** `def_node_index`, not `def`; `instruction_def_name`, not `def_name`.
- **No magic numbers or strings;** every literal with meaning becomes a named associated constant.
- **Names that an external API dictates** (C symbols, `serde` keys of a third-party schema, `#[wasm_bindgen]` exports) keep their spelling at that boundary, through `#[serde(rename = "...")]` or `#[no_mangle]`, and are mapped to house names right behind it.

## 6. Structure and Placement

Every concept has one canonical home, on the type that owns its data.

- **No free helper functions.** Put a helper in the `impl` of the type that owns its data (an associated function, without `self`, if it needs no instance). Free functions are acceptable only for `main`, tests, and exports whose ABI requires them (`extern "C"` and `#[wasm_bindgen]` entry points), which stay thin and call into a type.
- **One home per concept; consolidate duplicates.** When the same logic exists as several copied functions across modules or crates, collapse it into one method on the owning type, in the crate that owns the concept.
- **Reuse existing APIs; do not reinvent.** Know the wider codebase and the standard library before adding code: do not build a new wrapper layer when an existing crate or client already does the job, or a new helper when an iterator adaptor already exists.
- **No ad-hoc helper types.** Do not declare a private `struct` just to bundle a few values inside one module. A record either names a real concept, and then it is a type in the module that owns the concept, reused wherever that concept appears; or it does not, and the code keeps plain locals or an existing type instead.
- **Modules follow ownership.** One module per concept, named for it (`lexer.rs`, `diagnostic.rs`); a module's main type shares its name. Platform-agnostic logic sits behind a trait; platform-specific logic lives in the implementing type; domain constants live on the type that defines that domain.
- **Visibility is a decision.** Items are private by default; `pub(crate)` for what other modules of the crate use; `pub` only for the crate's API. No `pub use` re-export chains or prelude modules; the crate root re-exports its API by decision, and nothing else does.
- **Traits add capabilities, not homes.** An `impl Trait for Type` implements one trait; an extension trait on a foreign type (`trait StrExt { ... } impl StrExt for str`) is not a place to park unrelated helpers.
- **Enums are append-only where their value is persisted.** When a variant's discriminant or serialized name is an ID (a `#[repr(u8)]` enum, a `serde` tag written to disk or sent over the wire), give every variant an explicit discriminant or `#[serde(rename = "...")]` and extend it only at the end; never renumber or rename. Mark a public enum `#[non_exhaustive]` when downstream crates must not match it exhaustively.
- **No work at load time.** A `static` or `const` holds a value, not a side effect; a `LazyLock` or `OnceLock` may compute a value, but does not read files, spawn threads or register handlers.

## 7. Changing Code That Predates These Rules

When you touch a function, bring that function into line.

- **The house rules win; local consistency does not excuse a violation.** A file that names seventeen vectors without `list_` has seventeen violations, not a style of its own.
- **Fix neighbours that share the same violation.** Leaving a fixed function beside unfixed ones in the same file leaves the file with two styles instead of one.
- **A rename is not a `sed`.** Container prefixes collide with English: `windows` → `list_windows` also rewrites ordinary words in comments, doc comments and strings. After a rename sweep, re-read every comment and string literal it touched and restore the English.
- **A rename crosses into strings the compiler cannot see.** Rename with rust-analyzer, not text replacement, then search for the old name in `serde` keys, `#[serde(rename)]` attributes, JSON fixtures and golden files, C headers and exported symbol names; none of these fail to compile, they fail late or quietly stop covering what they named.
- **Establish the baseline before you blame yourself.** Stash the change, run `cargo build`, `cargo test` and `cargo clippy --all-targets -- -D warnings`, and diff the pass/fail lists. Failures that already exist on a clean tree are not yours; the evidence is the diff, not inspection.

## 8. Project-Specific Constraints

Each project records its hard constraints and how changes are verified here.

- **State what must not be done.** For example: "the compiler never runs a program's `sh` commands; it only parses and checks them."
- **State how output is verified.** For example: "`cargo test`, `cargo clippy --all-targets -- -D warnings` and `cargo fmt --check` pass; golden files change only with the change that explains them."
- **Link to companion documents** for the domain model and the repository layout / build flow rather than repeating them here.

## 9. Concurrency and Async

Every piece of shared mutable state has exactly one owner.

- **Let the type system carry thread safety.** Shared state is `Send` / `Sync` because its types are; an `unsafe impl Send` or `unsafe impl Sync` carries a `// SAFETY:` comment naming the invariant that makes it sound.
- **Shared mutable state has one owner.** It lives either behind one `Mutex` / `RwLock` held inside a single type that exposes methods, or in one task that owns it and is reached through a channel; never behind locks spread across several types.
- **No lock across an `.await`.** A `std::sync::Mutex` guard is dropped before the next `.await`; where state must be held across one, the owning task holds it instead.
- **Blocking calls never run on an async executor.** File I/O, subprocesses and synchronous FFI go through `spawn_blocking` (or the runtime's async equivalents), each with a timeout.
- **Structured concurrency first.** Prefer `join!`, `select!` and a `JoinSet` over a detached `spawn`; every spawned task's `JoinHandle` is stored by its owner and awaited or aborted when the owner shuts down.
- **Cancellation goes through one token.** A long-running async operation takes a cancellation token (or is dropped as a future) and passes it down; it does not invent its own cancel flag.
- **Threads, tasks and subscriptions are released.** Every thread joined, task aborted, file watcher or signal handler an object starts is stopped by that object's `Drop` or `shutdown` method.
- **Cover it with a regression test** that exercises the failure directly: cancel the outer task and assert the inner work stops and its resources are released.
