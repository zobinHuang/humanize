# Python Coding Conventions

## 0. Scope and General Principles

These conventions govern all first-party Python in the codebase (application, tools, language bindings' Python side, and tests). One codebase follows one style, even when it spans several repositories and languages; where a rule has a C++ counterpart, the two read the same (see [C++ Coding Conventions](cpp-coding-conventions.md)).

- **Third-party code is out of scope.** Vendored packages and samples carried verbatim must not be reformatted, so they can be re-synced with upstream.
- **These rules win over local habit.** Where an older file's own style disagrees, follow this document (see [Section 7](#7-changing-code-that-predates-these-rules)).
- Names such as `ProjectError` and `Registry` are placeholders; substitute the project's own types.

## 1. Comments and Docstrings

A comment states business or algorithm semantics, never language mechanics.

- **Domain semantics only.** Say what the code does in domain terms. Do not explain language mechanics ("copy first so the caller's list isn't mutated"), a style rule, or why you chose a construct.
- **No multi-line prose inside function bodies.** A one-line `#` label for a step is fine; a run of two or three `#` lines explaining a branch chain is not. When in doubt, delete it: self-explanatory code needs no prose.
- **No file header comment.** A file begins directly with code: no module docstring, license banner or leading `#` lines.
- **The docstring is the declaration.** Every public class, function and method carries a docstring; it may span several lines. Use Google style (`Args:`, `Returns:`, `Raises:`). Private helpers carry a one-line docstring or none.
- **Type hints are part of the declaration.** Every function signature is fully annotated, so the docstring does not repeat types.

```python
def get_definition_by_name(self, name: str) -> Definition | None:
    """Look up a loaded definition by its name.

    Args:
        name: the definition name

    Returns:
        the definition, or None if absent
    """
```

## 2. Error Handling: Exceptions and a Single Cleanup Path

Failure is signalled by raising, and every resource is released on one path.

- **Raise, do not return status codes.** A function that can fail raises a project exception (a subclass of `ProjectError`); it never returns `False`, `-1` or `None` to mean "failed". `None` means "absent", not "error".
- **Convert at the boundary.** Where a C++ binding returns a status code, the Python wrapper checks it once and raises; callers above the wrapper never see status codes.
- **One cleanup path: `with` or `finally`.** Resources (files, handles, locks, temp dirs) are acquired in a `with` block; cleanup that has no context manager goes in a single `finally`. Never duplicate cleanup before several `return`s.
- **Catch narrowly.** No bare `except:` and no `except Exception:` that swallows. Catch the specific type, and either recover meaningfully or re-raise with `raise ... from err` so the cause is kept.
- **No `assert` for runtime validation.** Asserts vanish under `-O`; validate inputs with an explicit check that raises.

```python
with self._open_trace(path) as trace:
    try:
        list_records = trace.read_all()
    except TraceFormatError as err:
        raise ProjectError(f"malformed trace: {path}") from err
return list_records
```

## 3. Variables and Scope

Python has no block scope, so a name lives from its first assignment to the end of the function.

- **Bind before branching.** A local assigned inside an `if` / `try` / loop and read after it is first bound before that block, so every path defines it.
- **Never read a loop variable after the loop ends.** Its value is whatever the last iteration left, or unbound if the iterable was empty.
- **A loop or comprehension variable must not shadow an outer local.** `for orig_idx, new_idx in map_remap.items()` in a function that already holds `orig_idx` silently overwrites it; name the binding for what it holds.
- **Never shadow builtins.** `list`, `dict`, `set`, `map`, `id`, `type`, `input`, `filter` are not variable names; the container prefixes in Section 5 exist partly to avoid this.
- **No mutable default arguments.** Write `list_items: list[Item] | None = None` and create the list in the body.

## 4. Multi-line Formatting

Any multi-argument call, long signature, collection literal or multi-condition test is fully expanded, one item per line, with a trailing comma so formatters (black / ruff format) keep it expanded.

**Calls:** one argument per line, keyword arguments wherever the callee accepts them.

```python
state = self._model().get_state_at(
    position=position,
    num_items=num_items,
    map_live=map_live,
    list_free=list_free,
)
```

**Positional-only calls:** when the callee does not take keywords (a foreign function loaded through `ctypes`, a builtin), each argument ends with a trailing `# parameter-name` comment naming the callee's parameter it binds. The comments are aligned, and name the parameter even when the argument is a variable of the same name. Try keywords first: many bindings, Cython ones included, do accept them, and then the call takes keywords instead.

```python
libcuda.cuMemcpyHtoD_v2(
    device_pointer,  # dstDevice
    host_buffer,     # srcHost
    nb_bytes,        # ByteCount
)
```

**Long signatures:** one parameter per line.

```python
def _record(
    self,
    name: str,
    subtype: str,
    is_virtual: bool,
    record_id: int,
) -> int:
```

**Multi-field construction:** one field per line, always by keyword, so values stay readable without positional labels. Records are `@dataclass`es, never bare tuples or dicts.

```python
list_window.append(
    Window(
        producer=node_index,
        start_time=list_start_time[node_index],
        end_time=end_time,
    )
)
```

**Functions passed as arguments:** a one-expression `lambda` stays inline; anything longer becomes a named method.

```python
list_window.sort(
    key=lambda window: window.start_time,
)
```

**Imports:** one contiguous block at the top of the file, with no blank lines between the standard-library, third-party and first-party groups.

```python
import math
import torch
from project.error import ProjectError
```

**Multi-condition `if` / `while`:** parenthesised, one condition per line, the operator leading each continuation line.

```python
if (
    list_nodes[node_index].is_variable
    and list_nodes[node_index].list_succs
):
```

## 5. Naming

A name tells the reader its type, visibility and purpose at a glance.

| Item | Rule | Example |
|---|---|---|
| `dict` / its iterator | prefix `map_` / suffix `_iter` | `map_free_slots`, `map_by_idx_iter` |
| `list` / `tuple` used as a sequence | prefix `list_` | `list_nodes` |
| `set` / `frozenset` | prefix `set_` | `set_live_items` |
| Private members and helpers | prefix `_` | `_model()`, `_emit_header()` |
| Constants | `<CATEGORY>_<ROLE>`, upper snake, on the owning class | `Register.REGISTER_ZERO`, `Buffer.BUFFER_MAX_SIZE` |
| Booleans | read as predicates | `do_write`, `is_virtual` |
| Indices / IDs | `int`, absent is `None` (`int \| None`) | `node_index: int` |
| Classes | `PascalCase` | `TraceReader` |
| Everything else | `snake_case` | `get_state_at` |

- **The container prefix is mandatory on locals, attributes and dataclass fields alike.** `new_list`, `by_idx` and `inserted` are containers that read as scalars; write `list_new_items`, `map_by_idx`, `set_inserted`.
- **Single underscore, not double.** `__name` triggers name mangling and breaks access from subclasses and tests; private means `_`.
- **Use built-in generics in hints.** `list[int]`, `dict[str, Node]`, `X | None`; not `typing.List`, `typing.Optional`.
- **Descriptive full words, not terse abbreviations.** `def_node_index`, not `def`; `instruction_def_name`, not `def_name`.
- **No magic numbers;** every literal with meaning becomes a named constant.
- **`snake_case`, never `camelCase`,** including inside lambdas and comprehensions.

## 6. Structure and Placement

Every concept has one canonical home, on the class that owns its data.

- **No module-level helper functions.** Put a helper on the class that owns its data (a `@staticmethod` if it needs no instance). Module-level functions are acceptable only for the entry point (`main`) and in test files.
- **One home per concept; consolidate duplicates.** When the same logic exists as several copied functions across modules, collapse it into one method on the owning class.
- **Reuse existing APIs; do not reinvent.** Know the wider codebase before adding code: do not build a new wrapper layer when an existing binding already does the job, or a new helper when an equivalent query already exists.
- **No ad-hoc helper types.** Do not define a private class or dataclass (such as `_Range` or `_Pair`) just to bundle a few values inside one module. A record either names a real concept, and then it is a public type in the module that owns the concept, reused wherever that concept appears; or it does not, and the code keeps plain locals or an existing type instead.
- **Place by ownership.** Backend-agnostic data and logic live in the base class; backend-specific logic lives in the matching subclass; domain constants live on the class that defines that domain.
- **Enums are append-only.** When an enum's value serves as an ID (`IntEnum` with explicit values), extend it only at the end; never insert in the middle or renumber.
- **No logic at import time.** A module body defines names; it does not open files, start processes or mutate global state. Executable modules guard with `if __name__ == "__main__":`.

## 7. Changing Code That Predates These Rules

When you touch a function, bring that function into line.

- **The house rules win; local consistency does not excuse a violation.** A file that names seventeen dicts without `map_` has seventeen violations, not a style of its own.
- **Fix neighbours that share the same violation.** Leaving a fixed function beside unfixed ones in the same file leaves the file with two styles instead of one.
- **A rename is not a `sed`.** Container prefixes collide with English: `kernels` → `list_kernels` also rewrites ordinary words in comments, docstrings and strings. After a rename sweep, re-read every comment and quoted string it touched and restore the English.
- **A rename crosses into the tests and into keyword arguments.** Tests often read attributes directly, and every keyword-argument call site names the parameter; Python reports neither until the line runs, so a missed site fails late or, worse, quietly stops covering what it named.
- **Establish the baseline before you blame yourself.** Stash the change, run the suite, and diff the pass/fail lists. Tests that already fail on a clean tree are not yours; the evidence is the diff, not inspection.

## 8. Project-Specific Constraints

Each project records its hard constraints and how changes are verified here.

- **State what must not be done.** For example: "development is fully static; never run on real hardware and never profile."
- **State how output is verified.** For example: "verify generated output by parse → serialize → disassembler."
- **Link to companion documents** for the domain model and the repository layout / build flow rather than repeating them here.

## 9. Ownership Across the Python / C++ Boundary

The C++ side owns lifetimes through `shared_ptr` (see the C++ conventions, Section 9); the Python side must not undermine it.

- **Hold the object, never a handle.** Keep the bound object itself; do not stash raw addresses, `id()` values or capsule pointers to look it up later.
- **Release explicitly, not through `__del__`.** Objects that own native resources expose a context manager or `close()`; finalizers run at an unpredictable time, or not at all at interpreter exit.
- **Do not touch an object after its owner is closed.** Once a parent's `close()` has run, its children are dead even if Python still holds them; drop the references.
- **Cover it with a regression test** that exercises the lifetime directly: release the parent, `del` it, run `gc.collect()`, then access the child.
