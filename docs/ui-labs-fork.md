# UI Labs Fork

Our fork of UI Labs exposes a `shared.UILabs` API that allows programmatic story
management from `execute_luau` through the Studio MCP (see
[`roblox-studio-mcp.md`](roblox-studio-mcp.md)).

## Shared API

Here's how to list, mount, and unmount stories...

```luau
-- List all available stories
local stories = shared.UILabs.listStories()
-- Returns: { { name: string, path: string, module: ModuleScript }, ... }

-- Mount a story by name (blocks until rendered, up to 5s)
local holder = shared.UILabs.mountStory("SetKeyPrompt")
-- Returns: Frame (the rendered story container) or nil

-- Get the holder of an already-mounted story
local holder = shared.UILabs.getStoryHolder("SetKeyPrompt")

-- List currently mounted stories
local mounted = shared.UILabs.getMountedStories()

-- Unmount a story
shared.UILabs.unmountStory("SetKeyPrompt")
```

UI Labs must be open in Studio for the API to be available. The stories are discovered
from Rojo serve sessions (e.g., `rojo serve storybook.project.json`).

## Screenshotting Stories

To screenshot a story, render it with `renderStory` first...

```luau
-- Renders story into CoreGui with dotted background (auto-cleans
-- after 30s via Debris)
shared.UILabs.renderStory("SetKeyPrompt")
-- Optional: pass width, height overrides
shared.UILabs.renderStory("SetKeyPrompt", 400, 350)
```

Then call `mcp__RobloxStudio__screen_capture` to capture the viewport (see
[Capturing Screenshots](roblox-studio-mcp.md#capturing-screenshots)). The render
auto-destroys after 30 seconds, or you can manually remove it...

```luau
local gui = game:GetService("CoreGui"):FindFirstChild("UILabsRender")
if gui then gui:Destroy() end
```

## Building & Installing

To rebuild our fork, run
`pnpm install && npx rbxtsc && rojo build default.project.json --output UILabs.rbxm` from
the `ui-labs` repo. Then install it into the Studio plugins folder under a fresh name, as
described in [Restarting Plugins](roblox-studio-mcp.md#restarting-plugins).
