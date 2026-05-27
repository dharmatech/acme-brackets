# Acme Bracketed Command Objects and Keyboard Bindings

## Design Proposal

### Overview

This document proposes a staged enhancement to the Acme interaction model.

The core idea is to introduce **bracketed command objects** that allow multi-word and multi-line commands to become first-class executable objects without requiring manual text selection.

The proposal evolves incrementally through four implementation levels:

```text
Level 1: Single-line bracketed command objects
Level 2: Multi-line bracketed command objects
Level 3: Keyboard-triggered command objects
Level 4: Scoped keyboard bindings
```

The design preserves Acme’s philosophy that commands are visible, textual, editable, and user-controlled.

---

# Goals

## Primary Goals

1. Make multi-word commands first-class executable objects.
2. Reduce dependence on manual text selection before middle-click execution.
3. Preserve Acme’s textual interaction model.
4. Avoid introducing hidden configuration systems.
5. Keep commands visible and editable directly within the interface.
6. Support both mouse-driven and keyboard-driven workflows.

## Non-Goals

1. Replacing Acme’s existing execution model.
2. Introducing a modal editor paradigm.
3. Introducing a separate scripting language.
4. Supporting nested command objects in initial implementations.
5. Replacing existing tag commands.

---

# Core Concept

A bracketed region becomes an executable command object.

Example:

```text
[mk install]
```

Middle-clicking anywhere inside the brackets executes:

```text
mk install
```

without requiring manual selection.

---

# Delimiter Choice

## Selected Delimiter

```text
[ ... ]
```

## Rationale

Square brackets were selected because:

* They are visually lightweight.
* They are easy to identify.
* They naturally group commands.
* They avoid some of the stronger semantic associations of:

  * `{}` in rc
  * `()` in rc
* They visually distinguish executable objects from ordinary text.

## Notes

Square brackets do have meaning in rc redirection syntax, but the bracket interpretation only applies during Acme command-object expansion and does not alter rc parsing behavior.

---

# Execution Semantics

## Execution Rule

The contents inside the brackets are executed.

Example:

```text
[mk install]
```

executes:

```text
mk install
```

The brackets themselves are not included.

## Whitespace

Leading and trailing whitespace inside brackets is trimmed before execution.

Example:

```text
[   mk install   ]
```

executes:

```text
mk install
```

---

# Level 1

# Single-Line Bracketed Command Objects

## Overview

Level 1 introduces executable bracketed regions constrained to a single line.

## Examples

```text
[mk]
[mk install]
[grep TODO *.c]
[plumb docs/foo.txt:37]
```

## Behavior

Middle-clicking anywhere inside the bracketed region:

1. Expands to the enclosing bracketed region.
2. Extracts the interior text.
3. Executes the extracted command.

## Scope

Level 1 applies to:

* Tags
* Window bodies

## Initial Constraints

### Same-Line Only

Bracket matching is restricted to a single line.

### No Nesting

Nested bracketed expressions are not supported.

Example:

```text
[a [b] c]
```

is undefined behavior in Level 1.

### Unmatched Brackets

Unmatched brackets are ignored.

No warning is required in the initial implementation.

## Success Criteria

Level 1 succeeds when:

```text
[mk]
[mk install]
```

can be middle-clicked and executed as first-class commands without requiring manual selection.

---

# Level 2

# Multi-Line Bracketed Command Objects

## Overview

Level 2 extends bracketed command objects to support multi-line regions.

## Example

```text
[
mk
install
test
]
```

or:

```text
[
rc -c '
    for(i in *.c)
        echo $i
'
]
```

## Behavior

Middle-clicking anywhere inside the bracketed region executes the entire enclosed text.

## Parsing Rules

### No Nesting

Nested bracket regions remain unsupported.

### Search Boundaries

Implementations should define bounded scanning behavior to avoid expensive searches through extremely large buffers.

Suggested approach:

* Scan outward from click position.
* Stop after configurable limits.
* Abort gracefully if no valid enclosure is found.

## Success Criteria

Level 2 succeeds when a multi-line bracketed region can be executed without requiring manual text selection.

---

# Level 3

# Keyboard-Triggered Command Objects

## Overview

Level 3 introduces keyboard-triggered command objects.

## Syntax

```text
[F8: mk]
[Ctrl-R: ./run]
[Shift-F5: test]
```

## Semantics

The text before the colon defines the trigger.

The text after the colon defines the command.

## Compilation Model

Key bindings are not dynamically scanned on every keypress.

Instead:

```text
tag text
    ↓
compile
    ↓
in-memory keymap
```

## Explicit Compilation

Bindings activate only after an explicit compile action.

Suggested commands:

```text
KeyPut
Keys
```

Exact naming remains open for experimentation.

## Initial Scope

Level 3 initially supports:

* Window-local bindings only

## Initial Constraints

### No Scoped Lookup Yet

Column/global lookup is deferred to Level 4.

### No Live Incremental Parsing

Bindings are recompiled only on explicit user action.

### No Nested Bracket Parsing

Nested command objects remain unsupported.

## Success Criteria

Level 3 succeeds when:

```text
[F8: mk]
```

can be compiled and invoked using the corresponding keypress.

---

# Level 4

# Scoped Keyboard Bindings

## Overview

Level 4 introduces hierarchical keyboard-binding scopes.

## Scope Hierarchy

```text
Window tag
Column tag
Global tag
```

## Lookup Order

Bindings resolve using:

```text
window > column > global
```

## Example

A window-local F8 overrides a column-local F8.

A column-local F8 overrides a global F8.

## Runtime Model

Compiled keymaps are stored independently per scope.

Keypress handling performs lightweight in-memory lookup only.

No tag scanning occurs during normal keypress handling.

## Success Criteria

Level 4 succeeds when identical bindings resolve correctly according to scope precedence.

---

# Parsing Model

## Conceptual Model

Bracketed command objects are parsed as lightweight structural regions.

The parser is not intended to become a full programming-language parser.

## Initial Simplicity

Early implementations prioritize:

* Simplicity
* Predictability
* Low runtime cost
* Incremental experimentation

---

# Compatibility Philosophy

This proposal attempts to extend Acme while preserving its core philosophy:

* Commands remain textual.
* Commands remain visible.
* Commands remain editable.
* Behavior emerges directly from interface text.
* No hidden menus or configuration dialogs are required.

---

# Future Possibilities

Potential future work may include:

* Nested bracket support
* Escaping mechanisms
* Persistent keymap serialization
* Visual highlighting of executable command objects
* Command-object discovery tooling
* Dynamic keymap inspection
* TUI-native Acme variants
* Integration with plumber semantics
* Alternative delimiter experimentation

These are intentionally deferred from the initial implementation stages.

---

# Recommended Initial Implementation Strategy

Recommended first implementation target:

```text
Level 1 only
```

Specifically:

* Single-line bracketed command objects
* No multiline support yet
* No keyboard bindings yet
* No scoped lookup yet

This provides:

* A small implementation surface
* Immediate usability improvement
* Low conceptual risk
* Easy testing and iteration

Subsequent levels can build incrementally from this foundation.
