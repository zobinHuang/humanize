# C++ Coding Conventions

## 0. Scope and General Principles

These conventions govern all first-party C++ in the codebase (application, shared libraries, tools, language bindings and tests) and, where stated, the companion Python code. One codebase follows one style, even when it spans several repositories.

- **Third-party code is out of scope.** Vendored sources and SDK samples carried verbatim must not be reformatted, so they can be re-synced with upstream.
- **These rules win over local habit.** Where an older file's own style disagrees, follow this document (see [Section 7](#7-changing-code-that-predates-these-rules)).
- Names such as `status_t`, `STATUS_OK` and `IF_FAILED` are placeholders; substitute the project's own types and macros.

## 1. Comments

A comment states business or algorithm semantics, never language mechanics.

- **Domain semantics only.** Say what the code does in domain terms. Do not explain language mechanics ("declared up front so goto can't skip the init"), a style rule, or why you chose a construct.
- **No multi-line prose inside function bodies.** A one-line `//` label for a step is fine; a two- or three-line paragraph explaining a branch chain is not. When in doubt, delete it: self-explanatory code needs no prose.
- **Declarations carry doc comments; definitions do not.** Header declarations use Doxygen (`\brief`, `\param`, `\return`). Function definitions in `.cpp` files carry no function-header comment.
- **Python follows the same rules.** Comments and docstrings state domain semantics, and a function body has no run of `#` prose lines: one line, or the docstring. A docstring is the declaration and may span several lines.

```cpp
/*!
 *  \brief  look up a loaded definition by its name
 *  \param  name    the definition name
 *  \return the definition, or nullptr if absent
 */
Definition* get_definition_by_name(const std::string& name) const;
```

## 2. Error Handling: Single Exit (`goto exit`)

A function that returns a status code has exactly one exit, and every failure path jumps to it.

- Declare a single `status_t retval = STATUS_OK;` at the top. On failure, set `retval` and `goto exit;`.
- One `exit:` label at the end performs cleanup and `return retval;`.
- Every function-scope exit goes through `goto exit`, never a bare `return`.
- Wrap fallible calls in an `IF_FAILED`-style macro whose recovery block does `goto exit`.

```cpp
status_t retval = STATUS_OK;
...
if (bad) { retval = STATUS_FAILED; goto exit; }
IF_FAILED(this->do_thing(), retval, { goto exit; });
...
exit:
    return retval;
```

## 3. Declare Variables at the Top

`goto exit` cannot jump over the initialization of a variable still in scope, so every function-scope local is declared at the top, before the first `goto`.

- **Hoist:** every function-scope local, and explicit loop counters. Write `for (i = 0; ...)` with `i` declared at the top, not `for (uint64_t i = 0; ...)`.
- **Keep inline** (the language cannot hoist them): range-for iteration variables (`for (const auto& x : c)`) and references (`T& v = c[i];`).
- **A lambda body is a function.** Its own declarations go at its own top; the enclosing function's hoisted counters are not the lambda's.
- **One counter serves every loop in the function.** A function with seventeen loops declares `i` / `k` once. The counter's value is meaningless between loops; never read it after a loop ends.
- **A range-for binding must not shadow a hoisted variable.** `for (const auto& [oi, ni] : m)` in a function that hoisted `oi` compiles and reads wrong; name the binding for what it holds (`orig_idx`, `new_idx`).

## 4. Multi-line Formatting

Any multi-argument call, long signature, aggregate initialization or multi-condition test is fully expanded, one item per line.

**Calls wrapped in error handling:** the wrapped call and the recovery block are both expanded.

```cpp
IF_FAILED(
    this->__model()->get_state_at(
        position,
        num_items,
        map_live,
        list_free
    ),
    retval,
    {
        goto exit;
    }
);
```

**Long signatures (function or lambda):** one parameter per line.

```cpp
auto __record = [&](
    const std::string& name,
    const std::string& subtype,
    bool is_virtual,
    uint64_t id
) -> uint64_t {
```

**Multi-field aggregate initialization:** one field per line, each with a leading `/* field-name */` label so positional values stay readable.

```cpp
list_window.push_back(window_t{
    /* producer */     node_index,
    /* start_time */   list_start_time[node_index],
    /* end_time */     end_time
});
```

**A lambda passed as an argument:** expanded too, with its body on its own lines.

```cpp
std::sort(
    list_window.begin(),
    list_window.end(),
    [](const window_t& a, const window_t& b) {
        return a.start_time < b.start_time;
    }
);
```

**Multi-condition `if` / `while`:** `if(` on its own line, one condition per line, `){` on its own line.

```cpp
if(
    list_nodes[node_index].is_variable
    && !list_nodes[node_index].list_succs.empty()
){
```

## 5. Naming

A name tells the reader its type, visibility and purpose at a glance.

| Item | Rule | Example |
|---|---|---|
| `std::map` / its iterator | prefix `map_` / suffix `_iter` | `map_free_slots`, `map_by_idx_iter` |
| `std::vector` | prefix `list_` | `list_nodes` |
| `std::set` | prefix `set_` | `set_live_items` |
| Private members and helpers | prefix `__` | `__model()`, `__emit_header()` |
| Constants | `k<Category>_<Role>` | `kRegister_Zero`, `kBuffer_MaxSize` |
| Booleans | read as predicates | `do_write`, `is_virtual` |
| Indices / IDs | `uint64_t`, sentinel `UINT64_MAX` | `uint64_t node_index` |

- **The container prefix is mandatory on locals and struct fields alike.** `new_list`, `by_idx` and `inserted` are containers that read as scalars; write `list_new_items`, `map_by_idx`, `set_inserted`. Iterators are declared at the function top like any other local.
- **`uint64_t`, never `size_t`.** The one exception is a type an external ABI dictates: a third-party callback signature, an out-parameter the library fills, or a field whose address is handed to a driver. Keep `size_t` exactly at that boundary with a comment saying why, and use `uint64_t` for every local, counter and constant on our side of it.
- **Watch where `size_t` hides:** `.find()` / `.size()` results, `constexpr` limits, lambda parameters, and a container's element type (`std::vector<size_t>`). Retype the container, not just the loop.
- **Descriptive full words, not terse abbreviations.** `def_node_index`, not `def`; `instruction_def_name`, not `def_name`.
- **No magic numbers;** every literal with meaning becomes a named constant.
- **`snake_case`, never `camelCase`,** including inside lambdas.

## 6. Structure and Placement

Every concept has one canonical home, on the class that owns its data.

- **No free functions outside a class.** Put a helper on the class that owns its data; a file-local `static` helper is acceptable only in test files.
- **One home per concept; consolidate duplicates.** When the same logic exists as several copied free functions, collapse it into one method on the owning class.
- **Reuse existing APIs; do not reinvent.** Know the wider codebase before adding code: do not build a new abstraction layer when an existing identifier already does the job, or a new helper when an equivalent query already exists.
- **Place by ownership.** Backend-agnostic data and logic live in the base class; backend-specific logic lives in the matching derived class; domain constants live on the class that defines that domain.
- **Enums are append-only.** When an enum's numeric value serves as an ID, extend it only at the end; never insert in the middle.

## 7. Changing Code That Predates These Rules

When you touch a function, bring that function into line.

- **The house rules win; local consistency does not excuse a violation.** A file that declares `for (size_t k = 0; ...)` seventeen times has seventeen violations, not a style of its own.
- **Fix neighbours that share the same violation.** Leaving a fixed function beside unfixed ones in the same file leaves the file with two styles instead of one.
- **A rename is not a `sed`.** Container prefixes collide with English: `kernels` → `list_kernels` also rewrites ordinary words in comments and strings. After a rename sweep, re-read every comment and quoted string it touched and restore the English.
- **A rename crosses into the tests.** Tests often read struct fields directly; a rename that skips them fails at link time or, worse, quietly stops covering what it named.
- **Establish the baseline before you blame yourself.** Stash the change, rebuild, run the suite, and diff the verdict lists. Tests that already fail on a clean tree are not yours; the evidence is the diff, not inspection.

## 8. Project-Specific Constraints

Each project records its hard constraints and how changes are verified here.

- **State what must not be done.** For example: "development is fully static; never run on real hardware and never profile."
- **State how output is verified.** For example: "verify generated output by parse → serialize → disassembler."
- **Link to companion documents** for the domain model and the repository layout / build flow rather than repeating them here.

## 9. Ownership Across the Python / C++ Boundary

Any C++ object that Python may still hold must be owned by a `shared_ptr`, never addressed by a raw pointer. Otherwise a child object outliving its parent surfaces as a segfault deep inside pybind11's cast, far from the cause.

- **Create with `make_shared`.** Factory methods such as `create()` and `clone()` return `std::shared_ptr`, never `new`.
- **Retain with `shared_ptr`.** Caches, `thread_local` caches, registration APIs, and every getter that hands an object across the boundary carry `shared_ptr`. A container that outlives one call owns its contents.
- **Back-references are `weak_ptr`.** A child points back at its parent with `std::weak_ptr` so the two do not keep each other alive; `.lock()` at the point of use makes the null check real. A class that hands out shares of itself derives from `std::enable_shared_from_this`.
- **`.get()` only at a non-retaining boundary.** Passing a raw pointer to a callee that reads and returns is fine; if the callee stores it, it takes a `shared_ptr`.
- **Cast with `std::static_pointer_cast`.** Under multiple inheritance the views sit at different offsets, so cross the base / extension boundary with the aliasing constructor.
- **The pybind11 holder must match.** A class registered as `pybind11::class_<T, std::shared_ptr<T>>` must never be returned as a raw pointer: pybind builds a second control block and the object is destroyed twice. `return_value_policy::reference` does not prevent this; only a `shared_ptr` return type does.
- **An opaque C-ABI handle points at the share, not the object.** When public headers cannot name an internal type, `void*` holds a heap `std::shared_ptr<T>` that the wrapper deletes in its destructor. An owning wrapper is neither copyable nor movable, and whoever caches wrappers deletes them.
- **Cover it with a regression test** that exercises the use-after-free directly: access a child after its parent has been released.
