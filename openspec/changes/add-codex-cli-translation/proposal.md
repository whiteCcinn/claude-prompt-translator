## Why

Users with a working local Codex CLI should be able to translate prompts through its saved authentication and configuration without configuring a separate translation API key or HTTP gateway.

## What Changes

- Add an optional `codex-cli` translation backend; retain `api` as the default.
- Invoke Codex non-interactively without a shell, passing protected text through stdin and consuming its final stdout.
- Reuse fragment restoration, submission validation, reply-language context, and progress feedback.
- Stop submission on process failure, timeout, cancellation, empty/truncated output, or invalid protected markers. Never fall back to the API.

## Capabilities

### New Capabilities
- `codex-cli-translation`: local CLI invocation and configuration.

### Modified Capabilities
- None.

## Impact

Plugin hook, user configuration, tests, README, and integration guide. No new runtime dependencies or server; no changes to the user's Codex credentials or configuration.
