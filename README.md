# Flint

A typed, ergonomic CLI argument parser for [Luau](https://luau.org/).

Declare your command-line interface once — Flint handles parsing, validation, and `--help` for you.

```luau
local flint = require("@pkg/flint")
local command, opt, flag, arg = flint.command, flint.opt, flint.flag, flint.arg

local cli = command("greet", "Greet someone") {
    arg "name" { help = "who to greet" },
    opt "count" { short = "n", type = "number", default = 1, help = "how many times" },
    flag "loud" { short = "l", help = "shout it" },
}

local args = cli:parse()

for _ = 1, args.count do
    local message = `Hello, {args.name}!`
    print(if args.loud then message:upper() else message)
end
```

```
$ greet World -n 2 --loud
HELLO, WORLD!
HELLO, WORLD!
```

## Features

- Declarative, minimal syntax inspired by [Vide](https://github.com/centau/vide)
- Fully typed — parsed results carry their types, so `args.count` is a `number` and your LSP knows it
- Auto-generated `--help` and `--version`, colored via [Tint](https://github.com/omarcoaraujo/tint)
- Positional arguments, options with values, and boolean flags
- Nested subcommands (`git commit`, `git push`) with their own options
- Typed values (`string`, `number`, `boolean`) with validation and clear errors
- Required options, default values, and repeatable options
- Helpful errors — unknown flags suggest the closest match ("did you mean `--output`?")
- Runtime agnostic — works with Lute, Lune, and Zune

## Installation

Add Flint to your `pesde.toml`:

```toml
[dependencies]
flint = { name = "omarcoaraujo/flint", version = "^0.1.0" }
```

Then require it:

```luau
local flint = require("@pkg/flint")
```

## Usage

### The pieces

Flint gives you four building blocks, which you import and compose:

- `command(name, about)` — the root of your CLI (or a subcommand)
- `opt(name)` — an option that takes a value: `--output path`
- `flag(name)` — a boolean switch: `--verbose`
- `arg(name)` — a positional argument: the bare `file.txt`

Each piece takes a config table. The name becomes the long form automatically, so `opt "output"` is `--output`.

### Options

An option takes a value. Configure its short form, type, whether it's required, and a default:

```luau
local cli = command("build", "Build the project") {
    opt "output" { short = "o", help = "output directory" },
    opt "jobs" { short = "j", type = "number", default = 4, help = "parallel jobs" },
    opt "target" { required = true, help = "build target" },
}

local args = cli:parse()
print(args.output, args.jobs, args.target)
```

```
$ build --target release -o ./dist -j 8
./dist   8   release
```

If `--target` is missing, Flint stops and prints a clear error instead of handing you a `nil`.

### Flags

A flag is a boolean — present means `true`, absent means `false`:

```luau
local cli = command("run", "Run the app") {
    flag "verbose" { short = "v", help = "verbose output" },
    flag "dry-run" { help = "don't actually do anything" },
}

local args = cli:parse()
if args.verbose then
    print("running in verbose mode")
end
```

Flags can be combined in the usual short form: `-v` and `-l` become `-vl`.

### Positional arguments

Positional arguments are matched by order:

```luau
local cli = command("copy", "Copy a file") {
    arg "source" { help = "file to copy" },
    arg "dest" { help = "where to copy it" },
}

local args = cli:parse()
print(`copying {args.source} -> {args.dest}`)
```

```
$ copy a.txt b.txt
copying a.txt -> b.txt
```

Make one optional with `default`, or accept many with `many = true` (collected into a table).

### Typed values

By default, values are strings. Set `type` to have Flint parse and validate them:

```luau
opt "port" { type = "number", default = 8080 }
opt "name" { type = "string" }
flag "debug" {}  -- flags are always boolean
```

If the user passes `--port abc`, Flint reports the type mismatch rather than silently giving you a bad value. Parsed results are typed accordingly — `args.port` is a `number`.

### Subcommands

A subcommand is just a `command` nested inside another. Each has its own options, arguments, and help:

```luau
local cli = command("git", "Version control") {
    command("commit", "Record changes") {
        flag "amend" { help = "amend the previous commit" },
        opt "message" { short = "m", required = true, help = "commit message" },
    },
    command("push", "Upload changes") {
        arg "remote" { default = "origin", help = "remote name" },
        flag "force" { short = "f", help = "force push" },
    },
}

local args = cli:parse()

if args.command == "commit" then
    print("committing:", args.message)
elseif args.command == "push" then
    print("pushing to:", args.remote)
end
```

```
$ git commit -m "initial commit"
committing: initial commit
```

The invoked subcommand is reported in `args.command`, and its own parsed values sit alongside it.

### Parsing

Call `parse` with no arguments to read from the process arguments automatically, or pass a table to parse explicitly (handy for testing):

```luau
local args = cli:parse()                        -- reads process args
local args = cli:parse({ "--target", "release" }) -- explicit
```

On a parse error — unknown flag, missing required option, bad type — Flint prints a message and the relevant usage, then signals failure. You don't write any of that handling yourself.

## Help and version

Flint generates `--help` (and `-h`) for every command and subcommand from what you declared. No help text to write or keep in sync:

```
$ greet --help
greet — Greet someone

Usage: greet [options] <name>

Arguments:
  name           who to greet

Options:
  -n, --count    how many times (default: 1)
  -l, --loud     shout it
  -h, --help     show this help
```

Set a version to get `--version` for free:

```luau
local cli = command("greet", "Greet someone") {
    version = "1.0.0",
    -- ...
}
```

When output goes to a terminal, help is colored via Tint; when piped, it's plain. Color detection is delegated to Tint, so `NO_COLOR` and friends are respected automatically.

## License

MIT