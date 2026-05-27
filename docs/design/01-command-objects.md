# Bracketed Command Objects

## Overview

Bracketed command objects allow multi-word and multi-line commands to
become first-class executable objects without requiring manual text
selection.

Levels 1 and 2 cover direct mouse execution:

```text
Level 1: Single-line bracketed command objects
Level 2: Multi-line bracketed command objects
```

## Delimiter Choice

The selected delimiter is:

```text
[ ... ]
```

Square brackets were selected because:

* They are visually lightweight.
* They are easy to identify.
* They naturally group commands.
* They avoid some of the stronger semantic associations of `{}` and `()`
  in rc.
* They visually distinguish executable objects from ordinary text.

Square brackets do have meaning in rc redirection syntax, but the
bracket interpretation only applies during Acme command-object expansion
and does not alter rc parsing behavior.

## Execution Semantics

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

Leading and trailing whitespace inside brackets is trimmed before
execution.

Example:

```text
[   mk install   ]
```

executes:

```text
mk install
```

## Level 1: Single-Line Bracketed Command Objects

Level 1 introduces executable bracketed regions constrained to a single
line.

Examples:

```text
[mk]
[mk install]
[grep TODO *.c]
[plumb docs/foo.txt:37]
```

### Behavior

Middle-clicking anywhere inside the bracketed region:

1. Expands to the enclosing bracketed region.
2. Extracts the interior text.
3. Executes the extracted command.

Clicking on either bracket character is treated as clicking in the
bracketed region.

### Selection Precedence

An explicit text selection takes precedence over bracket expansion.

If the middle-click occurs inside the current selection, Acme executes
the selected text using its existing behavior. Bracket expansion is only
a convenience for a null-selection middle-click.

### Whitespace

Leading and trailing whitespace inside the brackets is ignored.

Example:

```text
[  date  ]
```

executes:

```text
date
```

Empty or whitespace-only bracketed regions are ignored.

### rc Redirection Brackets

rc uses square brackets in file descriptor redirection syntax.

Example:

```text
>[2]/dev/null
```

Level 1 does not treat brackets immediately preceded by `<` or `>` as
command objects. Clicking inside `[2]` in the example above should not
execute `2`.

This exception is intentionally narrow. Level 1 does not otherwise parse
rc syntax; it only prevents common redirection syntax from becoming an
accidental bracketed command object.

### Scope

Level 1 applies to:

* Tags
* Window bodies

### Initial Constraints

Bracket matching is restricted to a single line.

Nested bracketed expressions are not supported.

Example:

```text
[a [b] c]
```

is undefined behavior in Level 1.

Unmatched brackets and empty brackets are ignored. No warning is required
in the initial implementation.

### Success Criteria

Level 1 succeeds when:

```text
[mk]
[mk install]
```

can be middle-clicked and executed as first-class commands without
requiring manual selection.

## Level 2: Multi-Line Bracketed Command Objects

Level 2 extends bracketed command objects to support multi-line regions.

Example:

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

### Behavior

Middle-clicking anywhere inside the bracketed region executes the entire
enclosed text.

### Delimiter Lines

Level 2 command objects use standalone bracket delimiter lines.

The opening delimiter line must contain only optional whitespace, `[`,
and optional whitespace.

The closing delimiter line must contain only optional whitespace, `]`,
and optional whitespace.

Valid:

```text
[
date
]
```

Also valid:

```text
    [
        date
    ]
```

Not a Level 2 delimiter:

```text
prefix [
date
]
```

This keeps multi-line command objects visually distinct from prose and
inline commands.

### Body Text

The command body is the text between the delimiter lines.

Leading and trailing blank lines inside the body are ignored. Internal
blank lines and indentation are preserved. Level 2 does not auto-dedent
the body.

Empty or whitespace-only bodies are ignored.

### Click Area

Level 2 uses bounded searching.

A click may occur on:

* the opening delimiter line
* the closing delimiter line
* one of the first three body lines after the opening delimiter
* one of the last three body lines before the closing delimiter

This allows convenient clicking near either end of the block without
requiring unbounded searches through large buffers.

Level 1 single-line bracket expansion is attempted first. If Level 1
matches, Level 2 is not considered.

### Parsing Rules

Nested bracket regions remain unsupported.

Implementations should define bounded scanning behavior to avoid
expensive searches through extremely large buffers.

Suggested approach:

* Scan outward from click position.
* Stop after configurable limits.
* Abort gracefully if no valid enclosure is found.

### Success Criteria

Level 2 succeeds when a multi-line bracketed region can be executed
without requiring manual text selection.
