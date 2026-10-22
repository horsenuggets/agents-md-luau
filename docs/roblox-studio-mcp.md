# Roblox Studio MCP

When working on Roblox projects, you can interact with a running Roblox Studio instance
via the RobloxStudio MCP server. This is useful for visually verifying UI changes,
inspecting the game tree, reading scripts, and more.

For our forks of Studio plugins with programmatic APIs, see these docs...

- [`rojo-fork.md`](rojo-fork.md) → Connecting and syncing Rojo from `execute_luau`.
- [`ui-labs-fork.md`](ui-labs-fork.md) → Mounting, rendering, and screenshotting UI Labs
  stories.

## Plugins Folder

To install or update a Studio plugin, copy its built `.rbxm` file into the Studio plugins
folder. Studio watches that folder and reloads plugins automatically, so there's no need
to restart Studio. The folder's location depends on the machine, so check the device
guidelines for it.

## Restarting Plugins

Studio treats each file in the plugins folder as its own plugin. To restart a plugin
instantly, remove the old file and install the new build under a fresh name with a random
8-character hexadecimal suffix...

```sh
rm -f <plugins-folder>/UILabs.rbxm <plugins-folder>/UILabs_*.rbxm
cp UILabs.rbxm "<plugins-folder>/UILabs_$(openssl rand -hex 4).rbxm"
```

Since the new file has a different name, Studio unloads the old plugin and loads the new
one right away, with no need to sleep between the removal and the copy. This works for any
plugin (Rojo, UI Labs, etc.). Always remove both the unsuffixed file and any suffixed
copies first, so stale copies don't pile up and load side by side.

## Capturing Screenshots

Use `mcp__RobloxStudio__screen_capture` with a `capture_id` parameter (e.g.,
`"ScreenCapture_1"`) to capture the current Studio viewport. This returns an image you can
view directly, so use it to verify UI components, plugin widgets, and visual changes. The
tool doesn't always show up in the list of available MCP tools, but it's there and works,
so call it anyway.

## Other Useful Tools

- `mcp__RobloxStudio__list_roblox_studios` → List connected Studio instances.
- `mcp__RobloxStudio__set_active_studio` → Switch the active Studio instance.
- `mcp__RobloxStudio__search_game_tree` → Search the Explorer tree.
- `mcp__RobloxStudio__inspect_instance` → Inspect instance properties.
- `mcp__RobloxStudio__script_read` → Read a script's source.
- `mcp__RobloxStudio__script_grep` → Search across scripts.
- `mcp__RobloxStudio__execute_luau` → Run Luau code in Studio.
- `mcp__RobloxStudio__get_console_output` → Read the output console.

These tools are deferred, so use `ToolSearch` to fetch their schemas before calling them.

## Clearing Studio Output

Use `LogService:ClearOutput()` to clear the output log before checking for new errors.
This avoids confusion from stale log entries...

```luau
local LogService = game:GetService("LogService")
LogService:ClearOutput()
```

## Reading Studio Output

The `get_console_output` tool often returns empty. If it does, use `execute_luau` with
`LogService` to read the output history directly...

```luau
local LogService = game:GetService("LogService")
local history = LogService:GetLogHistory()
local results = {}
for i = math.max(1, #history - 50), #history do
    local entry = history[i]
    if entry then
        table.insert(results, `[{entry.messageType}] {entry.message}`)
    end
end
if #results == 0 then
    return "No log entries found."
end
return table.concat(results, "\n")
```
