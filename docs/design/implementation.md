# Bracket Command Implementation Notes

These notes describe the current implementation shape. They are not a
separate user-facing design; the behavior is defined in `intro.md`.

## Execution Path

Bracket command expansion happens in `execute`, only for null-selection
middle-click execution.

The order is:

1. If the click is inside an explicit selection, execute the selection.
2. Try Level 1 inline bracket expansion on the current line.
3. Try Level 2 multiline bracket expansion near the click.
4. Fall back to Acme's existing executable-word expansion.

This preserves Acme's existing selection behavior and makes bracket
commands a convenience layer rather than a replacement execution model.

## Level 1

Level 1 scans only the current line.

The implementation ignores bracket openers immediately preceded by `<`
or `>` so common rc file descriptor redirection such as `>[2]` does not
become an accidental command object.

## Level 2

Level 2 recognizes standalone delimiter lines:

```text
[
command
]
```

The opening and closing delimiter lines may have leading or trailing
whitespace, but no other text.

Multiline search is bounded. A click can be near the opening delimiter,
near the closing delimiter, or inside a small number of body lines near
either delimiter. The implementation also has an internal maximum scan
distance so malformed buffers do not trigger unbounded searches.

When searching down from an opening delimiter, the search stops if it
finds another standalone opening delimiter before finding a closing
delimiter. When searching up from a closing delimiter, the search stops
if it finds another standalone closing delimiter before finding an
opening delimiter. This avoids accidentally pairing delimiters across
unrelated malformed blocks.

## Testing

The current verification path is manual:

* `demo/level1.txt`
* `demo/level2.txt`

A future refactor could move the expansion logic behind a text-buffer
and cursor-position interface so these cases can become automated tests.
