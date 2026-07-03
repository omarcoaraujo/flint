# Reveal

Pretty-printing and value inspection for [Luau](https://luau.org/).

Turns opaque tables into readable, colored output — the debug view Luau doesn't give you out of the box.

```luau
print(reveal({ name = "Marco", tags = { "dev", "luau" }, active = true }))
```

```
{
  name = "Marco",
  active = true,
  tags = { "dev", "luau" },
}
```

## Features

- Readable, indented output for tables of any depth
- Cyclic and shared references detected and labelled — never loops forever
- Colored output via [Tint](https://github.com/omarcoaraujo/tint), with automatic terminal capability detection (respects `NO_COLOR`, non-TTY piping, etc.)
- Deterministic key ordering — same value always prints the same way
- Handles non-string keys, functions, and userdata gracefully
- Configurable depth, indentation, and color
- Runtime agnostic — works with Zune, Lune, and Lute

## Installation

Add Reveal to your `pesde.toml`:

```toml
[dependencies]
reveal = { name = "omarcoaraujo/reveal", version = "^0.1.0" }
```

Then require it:

```luau
local reveal = require("@pkg/reveal")
```

## Usage

### Basic

Pass any value and get back a human-readable string:

```luau
print(reveal({ 1, 2, 3 }))
print(reveal({ x = 10, y = 20 }))
print(reveal("just a string"))
print(reveal(42))
```

### Nested tables

Depth is handled automatically, with indentation:

```luau
local config = {
  server = { host = "localhost", port = 8080 },
  retries = 3,
  tags = { "a", "b", "c" },
}

print(reveal(config))
```

```
{
  retries = 3,
  server = {
    host = "localhost",
    port = 8080,
  },
  tags = { "a", "b", "c" },
}
```

### Cyclic references

Self-references and shared tables are detected and labelled instead of looping forever:

```luau
local a = { name = "a" }
a.self = a

print(reveal(a))
```

```
<ref 1> {
  name = "a",
  self = <cycle to ref 1>,
}
```

### Non-string keys

Numeric, boolean, and table keys are formatted correctly:

```luau
print(reveal({ [1] = "one", [true] = "yes", ["has space"] = "ok" }))
```

```
{
  [1] = "one",
  [true] = "yes",
  ["has space"] = "ok",
}
```

### Options

Pass a table of options as the second argument:

```luau
print(reveal(value, {
  depth = 3,        -- max nesting depth before truncating (default: unlimited)
  indent = 2,       -- spaces per level (default: 2)
  color = true,     -- colored output (default: auto, based on terminal)
  sort_keys = true, -- deterministic key ordering (default: true)
}))
```

### Color

By default, Reveal colors its output when writing to a terminal that supports it, and prints plain text otherwise. Strings, numbers, booleans, and keys each get their own color.

Color detection is delegated to [Tint](https://github.com/omarcoaraujo/tint), so `NO_COLOR`, `FORCE_COLOR`, and TTY detection are all respected automatically. When output is piped to a file, colors are stripped.

Force it on or off explicitly:

```luau
print(reveal(value, { color = false })) -- always plain
print(reveal(value, { color = true }))  -- always colored
```

## License

MIT
