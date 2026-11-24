# Changelog

All notable changes to this project are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/) and the project uses
[SemVer](https://semver.org/).

## [0.2.4-tools.8] - 2026-11-24 (local fork patch)

- For auto models, drop the client's native `generateImage` tool from the
  forwarded tool list. aicode's native image tool expects an images-API
  reply; this chain answers with a chat completion, so the tool failed
  with "drew nothing" (image URL truncated) and the model retried it in a
  loop, even improvising calls to third-party image sites. In auto mode
  the aiProxy state machine and its server-side `generate_image` tool
  cover image generation end-to-end. Other models are unaffected.

## [0.2.4-tools.7] - 2026-11-24 (local fork patch)

- Stream early: for streaming requests the SSE response now opens
  immediately (role frame + first reasoning delta) and the function is
  invoked while the stream is held open, instead of synthesizing the whole
  stream after the function returns. Clients that time the thinking phase
  from the first reasoning delta previously measured ~0.1 s regardless of
  the real wait; now the measured thinking time tracks the actual
  function duration. Non-streaming requests are unchanged.

## [0.2.4-tools.6] - 2026-11-24 (local fork patch)

- Declare `reasoning: true` on every catalog model (config hook and the
  ping-refreshed dynamic list). tools.5 streamed reasoning_content but the
  client did not render a thinking block: without the capability flag the
  opencode-family client does not expect reasoning from this provider.

## [0.2.4-tools.5] - 2026-11-24 (local fork patch)

- Forward `reasoning_content` from aiProxy v2.2.7+: normalizeCompletion keeps
  it, the JSON response carries it, and the synthesized SSE stream emits
  reasoning deltas before content deltas, so clients can render the thinking
  block and its elapsed time.

## [0.2.4] - 2026-11-23

- Model refresh (2026-11-23, from my environment's current list): add
  `hy4-flash` and enable `hy4` — the catalog is now six text models
  (`hy4-flash`, `hy4`, `hy4-preview`, `hy3`, `deepseek-v4.1-flash`,
  `deepseek-v4.1-flash-1017`), all enabled by default. aiProxy's default
  allowlist matches. `hy-image-v3.5-preview` (image generation) is out of
  scope for this text-chat bridge.
- Overall review pass: the model catalog now matches the README table —
  `hy4` and `deepseek-v4.1-flash-1017` context documented as 1M in code
  too (was a conservative 128K placeholder); a unit test locks the
  catalog to the documented table.
- Project logo added to the top of both READMEs and to
  `examples/registry-entry.json` (`icon` field) for the community
  registry listing; local `cloudbaserc.json` is now gitignored.

## [0.2.3] - 2026-11-12

- Replace the shields.io image badge with a plain text link to the
  repository — Magpie's detail page does not reliably render remote
  images, so the GitHub link was invisible there.

## [0.2.2] - 2026-11-12

- Add a GitHub badge to both READMEs (renders as a button in Magpie's plugin
  detail page and on npm, linking to the repository).

## [0.2.1] - 2026-11-10

- Replace the plugin icon with the project logo (sprout + network) in
  `magpie.icon` — used by Magpie's discovery cards and installed list
  (GitHub-topic and npm sources alike).

## [0.2.0] - 2026-11-09

- **Enable `deepseek-v4.1-flash-1017` by default.** I verified it
  end-to-end through aiProxy in my own CloudBase environment (2026-11).
  It is not part of the official growth-plan notice; availability may differ
  per environment — CloudBase answers `AI_MODEL_NOT_SUPPORTED` where it is
  unavailable. `hy4` remains an unverified, disabled placeholder.
- `provider.models` no longer leaks a stale `disabled: true` for models the
  deployed aiProxy allowlist explicitly returns in its `ping` reply (the
  flag is stripped for every ping-allowed id).
- The request payload size limit now measures real UTF-8 bytes
  (`new TextEncoder().encode(payload).byteLength`) instead of JavaScript
  string length, which under-counted CJK content (each CJK char is typically
  3 UTF-8 bytes but 1 UTF-16 code unit). The 413 error message and the
  debug log's `bytes=` field now report accurate byte counts.
- Tests: new cases for the 1017 default-enabled state, disabled-flag
  stripping on ping, UTF-8 byte accounting for CJK payloads, `hy4` staying
  disabled, and the aiProxy allowlist containing `deepseek-v4.1-flash-1017`.

## [0.1.0] - 2026-11-09

Initial release.

- Magpie/OpenCode provider plugin (`cloudbase` provider id) that invokes the
  user-deployed `aiProxy` CloudBase cloud function through the official
  CloudBase HTTP function-invocation endpoint
  (`POST https://{envId}.api.tcloudbasegateway.com/v1/functions/{name}`).
- Adapts the one-shot cloud-function JSON reply into an OpenAI Chat
  Completions response and synthesizes SSE frames (role chunk, content
  chunks, final chunk, `[DONE]`) when the agent requests `stream: true`.
- Login flow collects environment ID, region (domestic/international) and
  cloud-function name; the CloudBase server-side API Key is stored only in
  Magpie's local auth store (`plugin-auth.json`, mode 600).
- Model catalog (official growth-plan notice, verified 2026-11-09):
  `hy3`, `hy4-preview`, `deepseek-v4.1-flash` enabled;
  `hy4`, `deepseek-v4.1-flash-1017` shipped as disabled placeholders.
- Cloud function `aiProxy` (Event Function, `index.main`) using
  `@cloudbase/node-sdk` `ai.createModel("cloudbase")` + `generateText`,
  env ID read from `TCB_ENV`, model allowlist via `AI_PROXY_ALLOWED_MODELS`.
- Unit tests (`node --test`), plugin load check script and a local mock
  aiProxy server for sandbox testing.
