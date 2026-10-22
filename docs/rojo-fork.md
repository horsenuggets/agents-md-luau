# Rojo Fork

Our fork of Rojo exposes a `shared.Rojo` API that allows programmatic plugin control from
`execute_luau` through the Studio MCP (see
[`roblox-studio-mcp.md`](roblox-studio-mcp.md)). This is essential for automated
workflows, since it eliminates the need to manually click Connect and Accept in the Rojo
plugin UI.

## Shared API

Here's the full API...

```luau
-- Connect and auto-accept in one call (recommended)
local result = shared.Rojo.connectAndAccept()
-- Optional: specify host, port, timeout
local result = shared.Rojo.connectAndAccept("localhost", "34872", 10)
-- Returns: { success: bool, projectName?: string, address?: string,
--            error?: string }

-- Connect without auto-accept (will pause at Confirming state)
shared.Rojo.connect("localhost", "34872")

-- Accept/abort a pending sync confirmation
shared.Rojo.accept()
shared.Rojo.abort()

-- Disconnect from the current session
shared.Rojo.disconnect()

-- Check connection status
local status = shared.Rojo.getStatus()
-- Returns: { status: string, projectName?: string, address?: string,
--            error?: string }
-- Status values: "NotConnected", "Connecting", "Confirming",
--                "Connected", "Error", "Unloaded"

-- Get current host and port
local addr = shared.Rojo.getAddress()
-- Returns: { host: string, port: string }

-- Clear known projects cache (forces confirmation dialog to
-- appear even with "Initial" confirmation behavior)
shared.Rojo.clearKnownProjects()
```

The API is only available in edit mode, not during play tests. It registers automatically
when the plugin loads and cleans up on unload.

## HTTP Requirement

Rojo projects that need the shared API must have HTTP enabled. Add this to the project
JSON...

```json
"HttpService": {
    "$className": "HttpService",
    "$properties": {
        "HttpEnabled": true
    }
}
```

## Building & Installing

To rebuild our fork, run `rojo build plugin.project.json --output Rojo.rbxm` from the
`rojo` repo. Then install it into the Studio plugins folder under a fresh name, as
described in [Restarting Plugins](roblox-studio-mcp.md#restarting-plugins).
