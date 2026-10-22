# Roblox Open Cloud » Creating & Publishing Places

This reference covers creating universes and places, publishing place files, and running
Luau in published places through the Roblox web APIs.

## Creating a New Universe & Place

There is no Open Cloud (API key) endpoint for creating universes. Use the undocumented
legacy endpoint with `.ROBLOSECURITY` cookie auth instead.

### Step 1 » Get a CSRF Token

Fetch a CSRF token by sending a request to the logout endpoint...

```sh
CSRF=$(curl -s -I -X POST 'https://auth.roblox.com/v2/logout' \
  --header "Cookie: .ROBLOSECURITY=$ROBLOX_COOKIE" \
  | rg -i 'x-csrf-token:' | sed 's/.*: //' | tr -d '\r\n')
```

### Step 2 » Create the Universe

Pass the token along with the cookie to create the universe...

```sh
curl -s -X POST 'https://apis.roblox.com/universes/v1/universes/create' \
  --header "Cookie: .ROBLOSECURITY=$ROBLOX_COOKIE" \
  --header "X-CSRF-TOKEN: $CSRF" \
  --header 'Content-Type: application/json' \
  --data '{ "templatePlaceId": 95206881 }'
```

The response looks like `{ "universeId": 123, "rootPlaceId": 456 }`. Template `95206881`
is the standard baseplate.

### Step 3 » Create Additional Places (Optional)

Add more places to the universe with the same token and cookie...

```sh
curl -s -X POST \
  "https://apis.roblox.com/universes/v1/user/universes/$UNIVERSE_ID/places" \
  --header "Cookie: .ROBLOSECURITY=$ROBLOX_COOKIE" \
  --header "X-CSRF-TOKEN: $CSRF" \
  --header 'Content-Type: application/json' \
  --data '{ "templatePlaceId": 95206881 }'
```

The response looks like `{ "placeId": 789 }`.

## Publishing a Place File

Use the Open Cloud API with an API key. The key needs `universe-places:write` scoped to
the target universe.

**Upload Caveat**

`curl --data-binary` does **not** work reliably for publishing place files. Use Lune's
`net.request` instead...

```luau
local fs = require("@lune/fs")
local net = require("@lune/net")
local serde = require("@lune/serde")

local placeData = fs.readFile("place.rbxl")

local response = net.request({
    url = `https://apis.roblox.com/universes/v1/{UNIVERSE_ID}/places/{PLACE_ID}/versions?versionType=Published`,
    method = "POST",
    headers = {
        ["Content-Type"] = "application/octet-stream",
        ["x-api-key"] = API_KEY,
    },
    body = placeData,
})

local result = serde.decode("json", response.body)
print(`Published version {result.versionNumber}.`)
```

Use `versionType=Published` to publish live, or `versionType=Saved` for a draft.

## Running Luau in a Published Place

Use the Luau Execution API to run scripts in a published place's server environment...

```luau
-- Create execution task
local createResponse = net.request({
    url = `https://apis.roblox.com/cloud/v2/universes/{UNIVERSE_ID}/places/{PLACE_ID}/luau-execution-session-tasks`,
    method = "POST",
    headers = {
        ["Content-Type"] = "application/json",
        ["x-api-key"] = API_KEY,
    },
    body = serde.encode("json", {
        script = scriptSource,
        universe = `universes/{UNIVERSE_ID}`,
    }),
})

local taskData = serde.decode("json", createResponse.body)
local taskPath = taskData.path

-- Poll for completion
local pollResponse = net.request({
    url = `https://apis.roblox.com/cloud/v2/{taskPath}`,
    method = "GET",
    headers = { ["x-api-key"] = API_KEY },
})

local pollData = serde.decode("json", pollResponse.body)
-- pollData.state: "PROCESSING" | "COMPLETE" | "FAILED"
-- pollData.output.results: array of return values from the script
```

The script runs in a real Roblox server with access to `game`, `workspace`,
`Instance.new`, etc. Scripts can return tables, which are serialized as JSON in the
results.

## API Key Scopes

Each API key needs the scopes for the operations it performs...

| Scope                                         | Required For             |
| --------------------------------------------- | ------------------------ |
| `universe-places:write`                       | Publishing place files   |
| `universe.place.luau-execution-session:write` | Running Luau scripts     |
| `universe:read`                               | Reading universe details |

Keys are created at
[create.roblox.com/dashboard/credentials](https://create.roblox.com/dashboard/credentials)
and must be scoped to specific universes.

## Key Locations

API keys and cookies are stored in `.env` files, never in code. Look for
`ROBLOX_OPEN_CLOUD_API_KEY` and `ROBLOX_COOKIE` in the project's `.env`. Individual repos
may also define their own project-scoped keys (e.g., `ROBLOX_E2E_API_KEY`).
