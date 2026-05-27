# Keyboard Bindings

## Overview

Keyboard bindings extend the visible command-object model to keypresses.

Levels 3 and 4 cover compiled keyboard bindings:

```text
Level 3: Keyboard-triggered command objects
Level 4: Scoped keyboard bindings
```

## Level 3: Keyboard-Triggered Command Objects

Level 3 introduces keyboard-triggered command objects.

### Syntax

```text
[F8: mk]
[Ctrl-R: ./run]
[Shift-F5: test]
```

### Semantics

The text before the colon defines the trigger.

The text after the colon defines the command.

For example:

```text
[F8: mk]
```

binds `F8` to:

```text
mk
```

### Compilation Model

Key bindings are not dynamically scanned on every keypress.

Instead:

```text
tag text
    |
    v
compile
    |
    v
in-memory keymap
```

Bindings activate only after an explicit compile action.

Suggested commands:

```text
KeyPut
Keys
```

`KeyPut` is the compile action: it reads visible key binding declarations
and updates the in-memory keymap.

`Keys` is the inspection action: it shows the currently compiled bindings.

`KeyPut` compiles bindings from the window tag only. Body text is not
scanned.

If the tag contains no key bindings, `KeyPut` clears the window's current
compiled bindings.

If `KeyPut` finds an invalid key binding declaration, it reports the
error and leaves the previous compiled bindings unchanged.

### Current Window

Acme does not use a traditional toolkit-level "active window" model.

For Level 3, the current window means the window containing the `Text`
that would receive the keypress.

In the current acme code, keyboard handling resolves this through:

```c
typetext = rowtype(&row, r, mouse->xy);
```

So the window-local binding scope is the window associated with
`typetext`.

If `typetext` is a window body, the binding scope is `typetext->w`.

If `typetext` is a window tag, the binding scope is also `typetext->w`.

Row tags and column tags are deferred to Level 4.

### Initial Scope

Level 3 initially supports:

* Window-local bindings only
* Function keys `F1` through `F12`

### Initial Constraints

Column/global lookup is deferred to Level 4.

Bindings are recompiled only on explicit user action.

Nested command objects remain unsupported.

Modifier syntax such as `Ctrl-R` and `Shift-F5` is deferred.

### Success Criteria

Level 3 succeeds when:

```text
[F8: mk]
```

can be compiled and invoked using the corresponding keypress.

## Level 4: Scoped Keyboard Bindings

Level 4 introduces hierarchical keyboard-binding scopes.

### Scope Hierarchy

```text
Window tag
Column tag
Global tag
```

### Lookup Order

Bindings resolve using:

```text
window > column > global
```

A window-local F8 overrides a column-local F8.

A column-local F8 overrides a global F8.

### Runtime Model

Compiled keymaps are stored independently per scope.

Keypress handling performs lightweight in-memory lookup only.

No tag scanning occurs during normal keypress handling.

### Success Criteria

Level 4 succeeds when identical bindings resolve correctly according to
scope precedence.
