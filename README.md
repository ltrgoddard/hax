# hax (fork) — server-side web search via `extra_tools`

This is a minimal fork of [OleksandrChekhovskyi/hax](https://github.com/OleksandrChekhovskyi/hax).
It adds one provider field, **`extra_tools`**, which lets a provider declare raw JSON tool
definitions that hax appends to the request's tool list *after* it builds its own function tools.

The motivating use case: DeepSeek's native server-side web search. DeepSeek only exposes it on
its [Responses API](https://api-docs.deepseek.com/guides/responses_api) via the `web_search` tool,
and hax's `extra_body` passthrough rejects `tools` as "protocol-owned". `extra_tools` appends it
instead of replacing hax's own tools.

## Usage

Point a provider at DeepSeek with the Responses dialect and add the tool:

```json
{
  "providers": {
    "deepseek": {
      "api": "openai-responses",
      "base_url": "https://api.deepseek.com",
      "api_key_env": "DEEPSEEK_API_KEY",
      "extra_tools": [ { "type": "web_search" } ]
    }
  }
}
```

The agent can then trigger native web search; hax tolerates the returned `web_search_call` item
and renders the searched answer.

## Keeping up with new releases

The code change lives as a self-contained patch in `patches/extra_tools_server_side.patch`.
`.github/workflows/rebuild.yml` fetches the **latest** upstream release, applies the patch,
builds, and publishes the patched binary as a release asset (weekly, on demand, or when the
patch changes) — so you never re-fork or manually rebase. If a future upstream release makes the
patch drift, the build fails loudly and you update the patch file.

## Build from source

```sh
git apply patches/extra_tools_server_side.patch
meson setup build
ninja -C build
```
