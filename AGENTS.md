# Luau Agent Guidelines

This document contains Luau-specific guidelines for agents working on Luau and Roblox
projects.

Always follow these guidelines unless otherwise specified. Before completing any task,
verify that all changes adhere to these rules.

## Terminology

- Always use "Luau" (not "Lua") when referring to the Roblox programming language.
- Say "Roblox Luau", never "Roblox Lua".
- Use `luau` for markdown code block syntax hints, never `lua`.
- Refer to "Luau code", "Luau files", "Luau scripts", "Luau modules", and the "Luau
  runtime".
- The only exceptions are pre-existing `.lua` file extensions and tool names like
  `stylua`.

## Code Style

### Formatting

- Run `stylua .` often to ensure that Luau code is formatted properly.
- Prefer string interpolation using backticks over the `..` concatenation operator (e.g.,
  `` `Here is a string with an interpolated {value}.` ``).
- Prefer double quotes over single quotes.
- Indent multi-line strings using `[[...]]` with the rest of the code, even if this adds
  leading whitespace to the string content.
- When any key in a table literal requires bracket-quote syntax (e.g., `["or"]` for Luau
  keywords), use `["key"]` syntax for **all** keys in that table for consistency.

### Declarations

Declare every binding with `const` unless it's reassigned later, and only then use
`local`. This includes functions (`const function helper()`), requires, constants, and
module tables. `const` locks the binding, not the value, so tables declared with `const`
can still be changed. Bindings declared without a value (e.g., `local result`) also need
`local`, since `const` requires an initial value.

`const` needs Luau 0.711 or newer, which means Lune `0.11.1-horse.16.0` or newer and
StyLua `2.6.0` or newer. Forks of upstream projects keep `local` in upstream code, since
rewriting it would make every upstream sync conflict, and only use `const` in files the
fork added.

### Requires

Always use require-by-string (e.g., `require("@packages/Fusion")`,
`require("@self/Module")`, `require("./Sibling")`) instead of instance-based requires
(e.g., `require(script.Parent.Module)`). Require-by-string is superior in every way.

### File Headers

Almost every Luau file should start with this header...

```luau
--[[

<File name without .luau extension>

<Description in a few sentences, wrapping by word at column 90>

--]]
```

The name in the header is the full file name with `.luau` removed. For files with
sub-extensions like `.spec.luau`, `.story.luau`, or `.storybook.luau`, keep the
sub-extension in the header (e.g., `MyModule.spec`, `PromptInput.story`).

For `init.luau` files (and Rojo variants like `init.server.luau`, `init.client.luau`,
`init.plugin.luau`), use the parent folder name instead of "init".

The exception is the root-level `init.luau` file in packages, which is just a re-export
for package consumers and does not need a header.

### Ordering

- Place constants (e.g., `const THIS_IS_A_CONSTANT`) above the module definition (e.g.,
  `const MyModule = {}`).

### Runtime Typechecking

- Always add runtime typechecking to function parameters using the `t` library (if it's a
  project dependency) with `assert(t.type(param))`. For projects without `t`, use
  `assert(type(param) == "string")` style checks instead.
- Add `t` checks to all public API functions and key internal functions that accept
  parameters from external callers.
- Use `t.interface()` for validating table and config parameters with known shapes.
- Use `t.optional()` for nullable parameters.
- Use `t.array()` for array parameters with element type validation.
- Use Roblox-specific type checkers for Roblox datatypes (`t.Font`, `t.Color3`,
  `t.Instance`, `t.Vector3`, `t.UDim2`, `t.CFrame`, `t.Enum`, `t.EnumItem`, etc.). Do
  **not** use `t.table` or `t.any` for these, since they are userdata, not tables.
- Keep `t` checks at the top of functions, before any logic.

### Code Reuse

- Never duplicate function definitions across files. If the same logic is needed in
  multiple places (e.g., a script and a command, or a command and a demo), extract it into
  a shared helper module and import it everywhere. This applies to utility functions,
  formatting logic, rendering code, color calculations, etc.
- Never manually copy data structures (command trees, config tables, option lists, etc.)
  that already exist in the codebase. Always import from the single source of truth. If a
  script needs a command tree, require the actual module instead of reconstructing it by
  hand.
- Scripts in `Scripts/` should import from `Source/` helpers, not copy their
  implementations.

## Naming Conventions

### File Names

- `MyModule.luau` → A module.
- `MyServerScript.server.luau` → A server script.
- `MyClientScript.client.luau` → A client script.
- `MyLegacyScript.legacy.luau` → A legacy script.
- `MyPluginScript.plugin.luau` → A plugin script.
- `init.luau` → An initialization file for a folder (use the parent folder name in the
  header).
- `MyFunction.func.luau` → A module that returns a function.
- `MyExecutableScript.exec.luau` → An executable Lune script.

### Executable Files

Every executable Lune script ends in `.exec.luau`. Declare a module with the file name in
PascalCase, add an `.exec()` function, and call it at the bottom. Helper modules that
scripts require (e.g., `Scripts/Helpers/`) are ordinary modules without the suffix.

Executable Lune scripts must have the executable bit set and start with a shebang line
(`#!/usr/bin/env -S lune run`). Always run them directly (e.g.,
`./Scripts/RunTests.exec.luau`) instead of using `lune run`. The file header names the
script with its sub-extension (e.g., `RunTests.exec`).

```luau
#!/usr/bin/env -S lune run

--[[

MyExecutableScript.exec

Describes what the script does.

[Usage] ./Scripts/MyExecutableScript.exec.luau

--]]

const MyExecutableScript = {}

function MyExecutableScript.exec()
    -- implementation
end

MyExecutableScript.exec()
```

### Windows Setup

On Windows, there is no shebang support. To make `./Scripts/Foo.exec.luau` work in
PowerShell, register a file association for `.luau` files by running this once in an
elevated PowerShell session...

```powershell
cmd /c 'assoc .luau=LuauScript'
cmd /c 'ftype LuauScript=lune run "%1" %*'
```

This tells Windows to run `.luau` files through `lune run`, which is equivalent to what
the shebang does on macOS and Linux. After this, `./Scripts/Foo.exec.luau` works in
PowerShell the same way it does in zsh or bash.

### Modules

For standard modules (PascalCase file names), use these conventions...

- **Functions** → camelCase (e.g., `Module.doSomething()`).
- **Properties** → PascalCase (e.g., `Module.SomeValue`).
- **Constructors** → camelCase (e.g., `Module.new()`, `Module.create()`).

### Classes

For classes (modules using metatables to instantiate), use these conventions...

- **Methods** → PascalCase with an explicit first `self` parameter (explained below).
- **Static Methods & Functions** → camelCase (e.g., `Class.new()`, `Class.fromData()`).
- **Instance & Static Properties** → PascalCase if public, _underscoredCamelCase if
  private.

Define methods using dot syntax with an explicit `self` parameter...

```luau
function Class.DoSomething(self: Class, value: number)
    -- implementation
end
```

Prefer an explicit `export type` for the class type, unioning static and instance members,
as well as public and private members. Sort alphabetically, but put constructors at the
top and private values at the bottom...

```luau
export type Class<T> = {
    new: () -> Class<T>,
    fromData: (data: T) -> Class<T>,
    PublicMethod: (self: Class<T>, value: T) -> (),
    PublicValue: T,
    Value: T,
    _privateMethod: (self: Class<T>, value: T) -> (),
    _privateValue: number,
}
```

### Table Keys & Type Fields

All keys in dictionary-style table literals and fields in type definitions must use
PascalCase. This applies to config tables, data structures, return values, and any table
with named keys...

```luau
export type Config = {
    ApiKeyEnvName: string?,
    Assets: string,
    Creator: CreatorConfig,
}

const state = {
    CurrentPhase = "lobby",
    RoundNumber = 0,
}
```

The only exception is tables whose keys must match an external API's JSON field names
(e.g., Open Cloud request and response bodies). Even so, we should do our best to abstract
these keys behind PascalCase fields in our own code.

## Module Structure & `@self`

In Roblox, scripts can have other scripts as children. For example, a `Script` can contain
a `ModuleScript`, and you can access it via `script.ModuleScript`. This parent-child
relationship is a fundamental pattern in Roblox development.

When syncing to the filesystem, the OS doesn't support putting a file as a child of
another file. Instead, the `dir/init.luau` pattern mimics this behavior...

- A directory with an `init.luau` file is treated as a single module.
- The directory name becomes the module name.
- Files inside the directory are "child scripts" of that module.
- `@self` refers to the directory containing the `init.luau`.

### Example Structure

Here's how a module with children looks on disk...

```text
MyModule/
├── init.luau       -- The main module (like Script in Roblox)
├── Helper.luau     -- Child module (like script.Helper)
└── Utils/
    └── init.luau   -- Nested child module (like script.Utils)
```

### Using `@self`

Only use the `@self` alias inside `init.luau` files. It refers to the directory containing
the `init.luau`, allowing you to require sibling files...

```luau
-- Inside MyModule/init.luau
const Helper = require("@self/Helper") -- MyModule/Helper.luau
const Utils = require("@self/Utils") -- MyModule/Utils/init.luau
```

This is equivalent to `script.Helper` and `script.Utils` in Roblox. For files that are not
`init.luau`, **never** use `@self`. Use relative paths (`./Sibling`) instead.

### Relative Requires From `init.luau`

In an `init.luau` file, relative requires like `./foo` resolve relative to the parent
directory (where the module would be imported from), not the directory containing the
`init.luau`. Use `@self/foo` to require files in the same directory as the `init.luau`.

## Configuration Files

- In `rokit.toml` and `wally.toml`, repository names using the `username/project` format
  should always be lowercase (e.g., `johnnymorganz/stylua`, not `JohnnyMorganz/StyLua`).
- In `.luaurc`, do not add `@self` as an alias. It is already built in and refers to the
  current module.

## Testing

- Always add tests for new behavior, especially when modifying analysis, linting, or
  type-checking logic. If you add a feature, add a test that verifies it works and a
  negative test that verifies it doesn't false-trigger.
- For TestEZ-style tests, do not wrap everything in a `describe` block with just the file
  name. The file name is already used as the test name, so a wrapping `describe` block is
  redundant.

## Static Analysis With `luau-lsp`

Run `luau-lsp analyze` frequently while working, after making changes, before committing,
and especially before pushing. Treat it like running tests, and do not wait until CI to
discover type errors.

Most of our projects have a script that runs linting or static analysis for you. It's
often named something like `Scripts/Lint.exec.luau`, but the name varies, so look through
`Scripts/` for it. If a project has one, use it. Otherwise, run `luau-lsp analyze`
directly with the appropriate platform flag. The commands below also show how to call the
binary when writing one of these scripts in the first place.

**Warnings Are Errors**

CI rejects any diagnostic output, including warnings like `ImportUnused` and
`LocalUnused`. Fix all warnings before committing, and do not leave unused imports, unused
variables, or any other warnings in the code.

Most projects have both Lune code (scripts in `Scripts/`) and Roblox code (source in
`Source/`). These require different platforms because they have different globals and type
definitions.

### Lune Projects & Scripts

Code that runs on Lune uses `--platform=lune`, which provides built-in globals like
`process`, `fs`, `net`, `task`, `stdio`, etc. This is the right platform for anything in
`Scripts/` and for standalone Lune packages that don't use Roblox APIs...

```sh
luau-lsp analyze \
  --platform=lune \
  --no-flags-enabled \
  --ignore "Packages/**" \
  --ignore "DevPackages/**" \
  --ignore "Submodules/**" \
  Scripts/
```

### Roblox Projects

Code that runs on Roblox uses `--platform=roblox`, which auto-loads PluginSecurity-level
type definitions (`Instance`, `UDim2`, `Color3`, `Enum`, `game`, `workspace`, etc.). No
`--definitions` flag is needed, since the fork embeds them at build time.

Roblox projects also need a Rojo sourcemap for proper module resolution. Generate one
first, then pass it in...

```sh
rojo sourcemap default.project.json --output sourcemap.json

luau-lsp analyze \
  --platform=roblox \
  --sourcemap=sourcemap.json \
  --no-flags-enabled \
  --ignore "Packages/**" \
  --ignore "DevPackages/**" \
  --ignore "Submodules/**" \
  Source/
```

The sourcemap enables DataModel-aware require resolution, including Wally packages that
use `default.project.json` with `$path` redirects (e.g., Fusion).

### Mixed Projects

Many projects have both Lune scripts and Roblox source. Run analyze twice with different
platforms targeting different directories...

```sh
# Analyze Lune scripts
luau-lsp analyze --platform=lune --no-flags-enabled \
  --ignore "Packages/**" --ignore "DevPackages/**" \
  Scripts/

# Analyze Roblox source
luau-lsp analyze --platform=roblox --no-flags-enabled \
  --sourcemap=sourcemap.json \
  --ignore "Packages/**" --ignore "DevPackages/**" \
  Source/
```

### Common Flags

- `--platform=standard|roblox|lune` → Load platform-specific type definitions.
- `--sourcemap=sourcemap.json` → Use a Rojo sourcemap for DataModel-aware resolution.
- `--ignore "glob"` → Ignore glob patterns (repeat for multiple).
- `--no-flags-enabled` → Disable all Luau FFlags (useful for deterministic CI).
- `--definitions=path.d.luau` → Load custom type definitions (overrides built-in ones).
- `--formatter=gnu` → Use GNU-style error format for editor integration.

### Exit Codes

The exit code is non-zero if any errors are found. Warnings alone do not cause failure
unless the project's `.luaurc` promotes them to errors.

## Reference Docs

The `docs/` directory contains reference data for Roblox and Luau development. Refer to
these files when you need detailed information about specific APIs or assets...

- `docs/roblox-fonts.md` → Complete list of all Roblox fonts with asset IDs.
- `docs/roblox-open-cloud-places.md` → Creating, publishing, and testing Roblox places via
  the API.
- `docs/roblox-studio-mcp.md` → Working with a running Roblox Studio through the Studio
  MCP (installing and restarting plugins, screenshots, and output logs). Read this before
  verifying UI or plugin changes in Studio.
- `docs/rojo-fork.md` → Our fork of Rojo and its `shared.Rojo` API for connecting and
  syncing from `execute_luau`.
- `docs/ui-labs-fork.md` → Our fork of UI Labs and its `shared.UILabs` API for mounting,
  rendering, and screenshotting stories.

Whenever you add, remove, rename, or change the purpose of a file in `docs/`, update this
list so it always matches the actual files.

## Lune Documentation

Read the Lune documentation as needed to understand the Lune code you're writing...

- [datetime](https://lune-org.github.io/docs/api-reference/datetime/)
- [fs](https://lune-org.github.io/docs/api-reference/fs/)
- [luau](https://lune-org.github.io/docs/api-reference/luau/)
- [net](https://lune-org.github.io/docs/api-reference/net/)
- [process](https://lune-org.github.io/docs/api-reference/process/)
- [regex](https://lune-org.github.io/docs/api-reference/regex/)
- [roblox](https://lune-org.github.io/docs/api-reference/roblox/)
- [serde](https://lune-org.github.io/docs/api-reference/serde/)
- [stdio](https://lune-org.github.io/docs/api-reference/stdio/)
- [task](https://lune-org.github.io/docs/api-reference/task/)
