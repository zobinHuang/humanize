# TypeScript Coding Conventions

## 0. Scope and General Principles

These conventions govern all first-party TypeScript in the codebase (application, libraries, tools, scripts and tests). One codebase follows one style, even when it spans several repositories and languages; where a rule has a Python or C++ counterpart, the two read the same (see [Python Coding Conventions](python-coding-conventions.md) and [C++ Coding Conventions](cpp-coding-conventions.md)).

- **Third-party code is out of scope.** Vendored packages and generated code (API clients, schema bindings) must not be reformatted, so they can be regenerated or re-synced with upstream.
- **These rules win over local habit.** Where an older file's own style disagrees, follow this document (see [Section 7](#7-changing-code-that-predates-these-rules)).
- **The compiler runs in `strict` mode.** `strict`, `noUncheckedIndexedAccess` and `noImplicitOverride` are on; a rule below never stands in for a compiler check that could enforce it.
- Names such as `ProjectError` and `Registry` are placeholders; substitute the project's own types.

## 1. Comments and Doc Comments

A comment states business or algorithm semantics, never language mechanics.

- **Domain semantics only.** Say what the code does in domain terms. Do not explain language mechanics ("spread first so the caller's array isn't mutated"), a style rule, or why you chose a construct.
- **No multi-line prose inside function bodies.** A one-line `//` label for a step is fine; a run of two or three `//` lines explaining a branch chain is not. When in doubt, delete it: self-explanatory code needs no prose.
- **No file header comment.** A file begins directly with its imports: no license banner, file description or `@file` block.
- **The doc comment is the declaration.** Every exported class, function, method and type carries a TSDoc comment (`@param`, `@returns`, `@throws`); it may span several lines. Private helpers carry a one-line comment or none.
- **Types live in the signature.** Every parameter and every exported function's return type is annotated, so the doc comment does not repeat types (`@param {string} name` is redundant).

```ts
/**
 * Look up a loaded definition by its name.
 *
 * @param name - the definition name
 * @returns the definition, or undefined if absent
 */
get_definition_by_name(name: string): Definition | undefined {
```

## 2. Error Handling: Exceptions and a Single Cleanup Path

Failure is signalled by throwing, and every resource is released on one path.

- **Throw, do not return status values.** A function that can fail throws a project error (a subclass of `ProjectError`); it never returns `false`, `-1` or `null` to mean "failed". `undefined` means "absent", not "error".
- **Throw `Error` subclasses only.** Never throw a string or a plain object; the stack trace is lost.
- **Convert at the boundary.** Where an external API reports failure through a status field or an error-first callback, the wrapper checks it once and throws; callers above the wrapper never see the raw status.
- **One cleanup path: `using` or `finally`.** Resources (files, handles, locks, subscriptions) are acquired with `using` / `await using`; cleanup that has no disposer goes in a single `finally`. Never duplicate cleanup before several `return`s.
- **Catch narrowly.** The caught value is `unknown`; narrow it with `instanceof` and either recover meaningfully or rethrow with `{ cause: err }` so the cause is kept. No empty `catch {}` and no catch-all that logs and continues.

```ts
await using trace = await TraceReader.open(path);
try {
    list_records = await trace.read_all();
} catch (err: unknown) {
    if (err instanceof TraceFormatError) {
        throw new ProjectError(`malformed trace: ${path}`, { cause: err });
    }
    throw err;
}
return list_records;
```

## 3. Types and Variables

A type states what a value is; the code never lies to the compiler about it.

- **No `any`.** Untrusted or unknown data is `unknown` and is narrowed before use.
- **No casts to silence the compiler.** `as T` appears only right after a runtime check that proves `T`, typically inside a type guard at an input boundary. Never `as unknown as T`.
- **No non-null assertions.** `value!` is replaced by an explicit check that throws or handles the absent case.
- **`const` by default, `let` only when reassigned, never `var`.**
- **Records are `interface`s or `type`s, never bare tuples.** A function returning two values returns an object with named fields.
- **Model states with discriminated unions,** not with several optional fields or boolean flags that may contradict each other.
- **A binding must not shadow an outer local.** `for (const [orig_index, new_index] of map_remap)` in a function that already holds `orig_index` silently hides it; name the binding for what it holds.
- **Never read a loop variable after the loop.** Compute what you need inside the loop and store it in a variable declared before it.

## 4. Multi-line Formatting

Any multi-argument call, long signature, object or array literal, or multi-condition test is fully expanded, one item per line, with a trailing comma so the formatter (prettier / biome) keeps it expanded.

**Calls:** one argument per line. A function that takes more than two parameters takes a single options object instead, so every call site names its values.

```ts
const state = this._model().get_state_at({
    position,
    num_items,
    map_live,
    list_free,
});
```

**Long signatures:** one parameter per line.

```ts
private _record(
    name: string,
    subtype: string,
    is_virtual: boolean,
    record_id: number,
): number {
```

**Multi-field construction:** one field per line, always by name.

```ts
list_window.push({
    producer: node_index,
    start_time: list_start_time[node_index],
    end_time,
});
```

**Functions passed as arguments:** a one-expression arrow function stays inline; anything longer becomes a named method.

```ts
list_window.sort(
    (a, b) => a.start_time - b.start_time,
);
```

**Imports:** one contiguous block at the top of the file, with no blank lines between the built-in, third-party and first-party groups. Type-only imports use `import type`.

```ts
import fs from "node:fs";
import { z } from "zod";
import type { Definition } from "./definition.js";
import { ProjectError } from "./error.js";
```

**Multi-condition `if` / `while`:** one condition per line, the operator leading each continuation line.

```ts
if (
    list_nodes[node_index].is_variable
    && list_nodes[node_index].list_succs.length > 0
) {
```

## 5. Naming

A name tells the reader its type, visibility and purpose at a glance.

| Item | Rule | Example |
|---|---|---|
| `Map` / `Record` used as a map / its iterator | prefix `map_` / suffix `_iter` | `map_free_slots`, `map_by_index_iter` |
| `Array` / `ReadonlyArray` | prefix `list_` | `list_nodes` |
| `Set` | prefix `set_` | `set_live_items` |
| Private members and helpers | `private` with prefix `_` | `_model()`, `_emit_header()` |
| Constants | `<CATEGORY>_<ROLE>`, upper snake, `static readonly` on the owning class | `Register.REGISTER_ZERO`, `Buffer.BUFFER_MAX_SIZE` |
| Booleans | read as predicates | `do_write`, `is_virtual` |
| Indices / IDs | `number`, absent is `undefined` | `node_index: number` |
| Classes, interfaces, types | `PascalCase`, no `I` / `T` prefix | `TraceReader`, `Definition` |
| Everything else | `snake_case` | `get_state_at` |

- **The container prefix is mandatory on locals, members and interface fields alike.** `new_list`, `by_index` and `inserted` are containers that read as scalars; write `list_new_items`, `map_by_index`, `set_inserted`.
- **`snake_case`, never `camelCase`, for first-party names,** including inside arrow functions and destructuring. Names that an external API dictates (DOM properties, library options, JSON fields of a third-party schema) keep their spelling at that boundary and are mapped to house names right behind it.
- **Descriptive full words, not terse abbreviations.** `def_node_index`, not `def`; `instruction_def_name`, not `def_name`.
- **No magic numbers or strings;** every literal with meaning becomes a named constant.

## 6. Structure and Placement

Every concept has one canonical home, on the class that owns its data.

- **No module-level helper functions.** Put a helper on the class that owns its data (a `static` method if it needs no instance). Module-level functions are acceptable only for the entry point (`main`) and in test files.
- **One home per concept; consolidate duplicates.** When the same logic exists as several copied functions across modules, collapse it into one method on the owning class.
- **Reuse existing APIs; do not reinvent.** Know the wider codebase before adding code: do not build a new wrapper layer when an existing client already does the job, or a new helper when an equivalent query already exists.
- **Place by ownership.** Backend-agnostic data and logic live in the base class; backend-specific logic lives in the matching subclass; domain constants live on the class that defines that domain.
- **Named exports only.** No `export default`, and no barrel files that re-export a directory; import from the module that defines the name.
- **Enums are append-only.** When an enum's value serves as an ID, give every member an explicit value and extend it only at the end; never insert in the middle or renumber. Prefer string unions where the value is not an ID.
- **No logic at import time.** A module body defines names; it does not read files, open connections, start timers or mutate global state.

## 7. Changing Code That Predates These Rules

When you touch a function, bring that function into line.

- **The house rules win; local consistency does not excuse a violation.** A file that names seventeen arrays without `list_` has seventeen violations, not a style of its own.
- **Fix neighbours that share the same violation.** Leaving a fixed function beside unfixed ones in the same file leaves the file with two styles instead of one.
- **A rename is not a `sed`.** Container prefixes collide with English: `kernels` → `list_kernels` also rewrites ordinary words in comments, doc comments and strings. After a rename sweep, re-read every comment and string literal it touched and restore the English.
- **A rename crosses into strings the compiler cannot see.** Use the editor's rename, not text replacement, then search for the old name in string-keyed access (`obj["name"]`), serialized JSON, test fixtures and snapshots; none of these fail to compile, they fail late or quietly stop covering what they named.
- **Establish the baseline before you blame yourself.** Stash the change, run the type check and the suite, and diff the pass/fail lists. Errors that already exist on a clean tree are not yours; the evidence is the diff, not inspection.

## 8. Project-Specific Constraints

Each project records its hard constraints and how changes are verified here.

- **State what must not be done.** For example: "never call the production API from tests; use the recorded fixtures."
- **State how output is verified.** For example: "`tsc --noEmit`, the linter and the test suite pass with zero warnings."
- **Link to companion documents** for the domain model and the repository layout / build flow rather than repeating them here.

## 9. Asynchrony

Every promise has an owner that awaits it or returns it.

- **No floating promises.** A call that returns a promise is `await`ed, returned, or collected into a `Promise.all`; fire-and-forget is not allowed (enable `no-floating-promises`).
- **`async` / `await` only.** Do not mix `.then()` chains with `await` in the same codebase, and never pass an `async` function to `forEach`; use `for ... of` for sequential work and `Promise.all` for independent work.
- **Cancellation goes through `AbortSignal`.** A long-running async function takes an optional `signal` in its options object and passes it down; it does not invent its own cancel flag.
- **Timers and listeners are disposed.** Every `setInterval`, event listener or subscription registered by an object is released by that object's `[Symbol.dispose]()` / `close()`.
- **Cover it with a regression test** that exercises the failure directly: reject the inner promise and assert the outer call rejects with the original error as its `cause`.
