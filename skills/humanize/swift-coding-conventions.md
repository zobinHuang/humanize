# Swift Coding Conventions

## 0. Scope and General Principles

These conventions govern all first-party Swift in the codebase (application, packages, command-line tools and tests). One codebase follows one style, even when it spans several repositories and languages; where a rule has a C++ or TypeScript counterpart, the two read the same (see [C++ Coding Conventions](cpp-coding-conventions.md) and [TypeScript Coding Conventions](typescript-coding-conventions.md)).

- **Third-party code is out of scope.** Vendored packages and generated code must not be reformatted, so they can be re-synced with upstream.
- **These rules win over local habit.** Where an older file's own style disagrees, follow this document (see [Section 7](#7-changing-code-that-predates-these-rules)).
- **The compiler runs in Swift 6 language mode.** Strict concurrency checking is on and warnings are treated as errors; a rule below never stands in for a compiler check that could enforce it.
- **Naming follows the Swift API Design Guidelines** (`camelCase`, argument labels that read as phrases), with the house additions in [Section 5](#5-naming).
- Names such as `ProjectError` and `Registry` are placeholders; substitute the project's own types.

## 1. Comments and Doc Comments

A comment states business or algorithm semantics, never language mechanics.

- **Domain semantics only.** Say what the code does in domain terms. Do not explain language mechanics ("captured weakly so the closure doesn't retain self"), a style rule, or why you chose a construct.
- **No multi-line prose inside function bodies.** A one-line `//` label for a step is fine; a run of two or three `//` lines explaining a branch chain is not. When in doubt, delete it: self-explanatory code needs no prose.
- **No file header comment.** A file begins directly with its imports: no `//  File.swift` / `//  Created by` template banner, license block or file description. Delete the banner Xcode generates.
- **The doc comment is the declaration.** Every non-private type, function, method and property carries a `///` doc comment (`- Parameter`, `- Returns`, `- Throws`); it may span several lines. Private helpers carry a one-line comment or none.
- **No `// MARK:` scaffolding by default.** Use `// MARK: -` only in a type long enough that its sections are not obvious; a file whose structure needs many marks wants splitting.

```swift
/// Look up a tracked window by its window-server identifier.
///
/// - Parameter windowId: the window-server identifier
/// - Returns: the window, or nil if it is not tracked
func window(withId windowId: WindowId) -> TrackedWindow?
```

## 2. Error Handling: Throws and a Single Cleanup Path

Failure is signalled by throwing, and every resource is released on one path.

- **Throw, do not return status values.** A function that can fail is `throws`; it never returns `false`, `-1` or `nil` to mean "failed". `nil` means "absent", not "error".
- **Errors are typed enums.** Each module declares its error type as an `enum` conforming to `Error`, with associated values carrying the context (`case axCallFailed(AXError, attribute: String)`). Never throw a bare `NSError` built on the spot.
- **Convert at the boundary.** Where a system API reports failure through `OSStatus`, `AXError`, `CGError` or an out-parameter, the wrapper checks it once and throws; callers above the wrapper never see the raw status.
- **One cleanup path: `defer`.** Release a resource with a `defer` placed right after it is acquired. Never duplicate cleanup before several `return`s or `throw`s.
- **No `try!`, and no `try?` that swallows a failure.** `try?` is allowed only where "failed" and "absent" genuinely mean the same thing to the caller, and the call site makes that obvious. `do/catch` catches specific cases and either recovers meaningfully or rethrows wrapped with its cause; no empty `catch {}` and no catch-all that logs and continues.

```swift
let handle = try FileHandle(forReadingFrom: url)
defer { try? handle.close() }
do {
    listRecords = try TraceReader.decode(handle.readToEnd() ?? Data())
} catch let error as DecodingError {
    throw ProjectError.malformedTrace(url: url, cause: error)
}
return listRecords
```

## 3. Types and Variables

- **No force unwrap, no force cast.** `value!` and `as!` are replaced by `guard let` / `if let` with an explicit throw or handled absent case. Implicitly unwrapped optionals (`T!`) are not declared.
- **Core Foundation values are the one force-cast exception.** A CF type cannot be conditionally cast, so the wrapper that reads a `CFTypeRef` compares `CFGetTypeID` first and then uses `as!`, in exactly one place; callers above the wrapper never cast.
- **Unwrap early with `guard`.** Preconditions at the top of a function are `guard ... else { throw / return }`, so the main path stays unindented.
- **`let` by default, `var` only when mutated.**
- **Value types first.** Records are `struct`s; a `class` is used only for identity or shared mutable state, and is `final` unless it is designed for subclassing.
- **Records are named types, never tuples.** A function returning two values returns a `struct` with named fields; tuples are acceptable only as transient locals inside one expression.
- **Model states with enums with associated values,** not with several optionals or boolean flags that may contradict each other.
- **A binding must not shadow an outer local.** `for (windowId, frame) in mapFrames` in a function that already holds `windowId` silently hides it; name the binding for what it holds. Shadowing by `guard let value = value` / `if let value` unwrapping is the one allowed form.
- **Never read a loop variable after the loop.** Compute what you need inside the loop and store it in a variable declared before it.
- **`Int` for indices and counts.** Use fixed-width integers (`UInt32`, `Int64`) only where an external ABI dictates them (a `CGWindowID`, a wire format), and wrap such values in a named type at the boundary.

## 4. Multi-line Formatting

Any call, signature or literal that does not fit on one line is fully expanded, one item per line, with the closing delimiter on its own line.

**Calls:** one argument per line, each with its argument label; the closing parenthesis on its own line.

```swift
let state = model.state(
    at: position,
    count: numItems,
    live: mapLive,
    free: listFree
)
```

**Positional calls into external APIs:** where a callee the codebase does not own takes unlabelled arguments (C functions such as `AXUIElementCopyAttributeValue`), each argument carries a leading `/* parameter-name */` label naming the parameter it binds, aligned, as in C++.

```swift
let error = AXUIElementCopyAttributeValue(
    /* element */     element,
    /* attribute */   kAXWindowsAttribute as CFString,
    /* value */       &value
)
```

**Long signatures:** one parameter per line.

```swift
private func record(
    name: String,
    subtype: String,
    isVirtual: Bool,
    recordId: Int
) -> Int {
```

**Multi-field construction:** one field per line, always by label.

```swift
listWindows.append(WindowSpan(
    producer: nodeIndex,
    startTime: listStartTimes[nodeIndex],
    endTime: endTime
))
```

**Closures passed as arguments:** a one-expression closure stays inline, as a trailing closure; anything longer becomes a named method. Use `$0` only in one-expression closures.

```swift
listWindows.sort { $0.startTime < $1.startTime }
```

**Imports:** one contiguous block at the top of the file, sorted, with no blank lines between system and first-party modules. Import the narrowest module that provides the names (`import Foundation`, not `import Cocoa`, in code that does not touch AppKit).

**Multi-condition `if` / `guard` / `while`:** one condition per line, the operator leading each continuation line, and the opening brace on its own line.

```swift
if
    listNodes[nodeIndex].isVariable
    && !listNodes[nodeIndex].listSuccessors.isEmpty
{
```

## 5. Naming

A name tells the reader its type and purpose at a glance.

| Item | Rule | Example |
|---|---|---|
| `Dictionary` / its index | prefix `map` / suffix `Index` | `mapFreeSlots`, `mapByIdIndex` |
| `Array` / `ArraySlice` | prefix `list` | `listNodes` |
| `Set` | prefix `set` | `setLiveItems` |
| Private members and helpers | `private` / `fileprivate`, no prefix | `private func emitHeader()` |
| Constants | `static let` on the owning type, `camelCase` | `Layout.switcherCornerRadius` |
| Booleans | read as predicates | `isVirtual`, `shouldWrite` |
| Types and protocols | `UpperCamelCase`, no `I` / `T` prefix | `TraceReader`, `WindowSource` |
| Everything else | `lowerCamelCase` | `state(at:)` |

- **The container prefix is mandatory on locals, properties and struct fields alike.** `newList`, `byIndex` and `inserted` are containers that read as scalars; write `listNewItems`, `mapByIndex`, `setInserted`.
- **No first-party method starts with `set`.** Mutation is property assignment; a method that writes state is named for its effect (`applyFrame(_:)`, `move(to:)`), so a `set` prefix always means a `Set`.
- **Acronyms follow Swift casing:** uniformly upper or lower case as a unit (`windowId`, `axElement`, `URLSession`, `cgWindowId`).
- **Descriptive full words, not terse abbreviations.** `defNodeIndex`, not `def`; `instructionDefName`, not `defName`.
- **No magic numbers or strings;** every literal with meaning becomes a named constant.
- **Names that an external API dictates** (Objective-C selectors, `Codable` keys of a third-party schema) keep their spelling at that boundary and are mapped to house names right behind it.

## 6. Structure and Placement

Every concept has one canonical home, on the type that owns its data.

- **No top-level functions.** Put a helper on the type that owns its data (a `static` method if it needs no instance). Top-level code is acceptable only in the entry point (`main.swift` or the `@main` type) and in test files.
- **One home per concept; consolidate duplicates.** When the same logic exists as several copied functions across modules, collapse it into one method on the owning type.
- **Reuse existing APIs; do not reinvent.** Know the wider codebase before adding code: do not build a new wrapper layer when an existing client already does the job, or a new helper when an equivalent query already exists.
- **No ad-hoc helper types.** Do not declare a `fileprivate` or function-local `struct` just to bundle a few values inside one file. A record either names a real concept, and then it is a type in the module that owns the concept, reused wherever that concept appears; or it does not, and the code keeps plain locals or an existing type instead.
- **Place by ownership.** Platform-agnostic data and logic live behind a protocol; platform-specific logic lives in the conforming type; domain constants live on the type that defines that domain.
- **Extensions add conformances, not homes.** An `extension` implements one protocol conformance or splits a long type by concern; it is not a place to park unrelated helpers on a system type (`extension String { func ... }`).
- **Access control is explicit at module boundaries.** Everything a package exports is `public` by decision; everything else is `internal` or narrower. No `open` without a subclassing design.
- **Enums are append-only.** When an enum's raw value serves as an ID or is persisted, give every case an explicit raw value and extend it only at the end; never insert in the middle or renumber.
- **No work at load time.** A `static let` holds a value, not a side effect: it does not read files, start timers or register observers.

## 7. Changing Code That Predates These Rules

When you touch a function, bring that function into line.

- **The house rules win; local consistency does not excuse a violation.** A file that names seventeen arrays without `list` has seventeen violations, not a style of its own.
- **Fix neighbours that share the same violation.** Leaving a fixed function beside unfixed ones in the same file leaves the file with two styles instead of one.
- **A rename is not a `sed`.** Container prefixes collide with English: `windows` → `listWindows` also rewrites ordinary words in comments, doc comments and strings. After a rename sweep, re-read every comment and string literal it touched and restore the English.
- **A rename crosses into strings the compiler cannot see.** Use the editor's rename, not text replacement, then search for the old name in `Codable` keys, `UserDefaults` keys, `#selector` / key paths built from strings and test fixtures; none of these fail to compile, they fail late or quietly stop covering what they named.
- **Establish the baseline before you blame yourself.** Stash the change, run `swift build` and `swift test`, and diff the pass/fail lists. Failures that already exist on a clean tree are not yours; the evidence is the diff, not inspection.

## 8. Project-Specific Constraints

Each project records its hard constraints and how changes are verified here.

- **State what must not be done.** For example: "never call private SkyLight APIs that write window-server state."
- **State how output is verified.** For example: "`swift build`, `swift test` and `swift format lint --strict` pass with zero warnings."
- **Link to companion documents** for the domain model and the repository layout / build flow rather than repeating them here.

## 9. Concurrency

Every piece of shared mutable state has exactly one isolation domain.

- **UI state is `@MainActor`.** AppKit and SwiftUI types, and the models they observe, are main-actor isolated; nothing else runs on the main actor.
- **Shared mutable state lives in an `actor`** or behind a single serial queue owned by one type; never behind a hand-rolled lock spread across several types.
- **No escape hatches without a reason.** `@unchecked Sendable`, `nonisolated(unsafe)` and `@preconcurrency import` each carry a one-line comment naming the invariant that makes them safe.
- **Blocking calls never run on the main actor.** Synchronous IPC (Accessibility, Apple Events) and file I/O run on a background executor, each with a timeout.
- **Structured concurrency first.** Prefer `async let` and task groups over detached `Task {}`; every unstructured `Task` is stored by its owner and cancelled when the owner shuts down.
- **Observers and timers are released.** Every notification observer, `AXObserver`, run-loop source or timer an object registers is removed by that object when it shuts down.
- **Cover it with a regression test** that exercises the failure directly: cancel the outer task and assert the inner work stops and its resources are released.
