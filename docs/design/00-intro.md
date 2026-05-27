# Acme Bracketed Command Objects and Keyboard Bindings

## Design Proposal

This project explores a staged enhancement to the Acme interaction model.

The core idea is to introduce bracketed command objects and keyboard
binding declarations while preserving Acme's visible, textual, editable
workflow.

The proposal evolves through four implementation levels:

```text
Level 1: Single-line bracketed command objects
Level 2: Multi-line bracketed command objects
Level 3: Keyboard-triggered command objects
Level 4: Scoped keyboard bindings
```

## Reading Order

* `00-intro.md`: project goals, design overview, and compatibility philosophy
* `01-command-objects.md`: Level 1 and Level 2 bracketed command objects
* `02-key-bindings.md`: Level 3 and Level 4 keyboard binding model
* `implementation.md`: implementation notes for the current code

## Goals

### Primary Goals

1. Make multi-word commands first-class executable objects.
2. Reduce dependence on manual text selection before middle-click execution.
3. Preserve Acme's textual interaction model.
4. Avoid introducing hidden configuration systems.
5. Keep commands visible and editable directly within the interface.
6. Support both mouse-driven and keyboard-driven workflows.

### Non-Goals

1. Replacing Acme's existing execution model.
2. Introducing a modal editor paradigm.
3. Introducing a separate scripting language.
4. Supporting nested command objects in initial implementations.
5. Replacing existing tag commands.

## Core Concept

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

Keyboard binding declarations extend that same visible-text model:

```text
[F8: mk]
```

After explicit compilation, pressing `F8` in the relevant scope runs:

```text
mk
```

## Parsing Model

Bracketed command objects are parsed as lightweight structural regions.

The parser is not intended to become a full programming-language parser.

Early implementations prioritize:

* simplicity
* predictability
* low runtime cost
* incremental experimentation

## Compatibility Philosophy

This proposal attempts to extend Acme while preserving its core
philosophy:

* Commands remain textual.
* Commands remain visible.
* Commands remain editable.
* Behavior emerges directly from interface text.
* No hidden menus or configuration dialogs are required.

## Future Possibilities

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

## Initial Implementation Strategy

The recommended first implementation target was:

```text
Level 1 only
```

Specifically:

* Single-line bracketed command objects
* No multiline support yet
* No keyboard bindings yet
* No scoped lookup yet

This provided:

* a small implementation surface
* immediate usability improvement
* low conceptual risk
* easy testing and iteration

Subsequent levels build incrementally from that foundation.
