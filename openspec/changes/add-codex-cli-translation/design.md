## Context

The plugin's `prompt.submit` hook currently calls an OpenAI-compatible HTTP endpoint. Claude Code exposes `$.process.run(argv, { stdin, timeoutMs })`, which runs without a shell and reports stdout separately from stderr. Local Codex CLI 0.166.0 supports `exec`, stdin, ephemeral sessions, read-only sandboxing, and plain final-message stdout.

## Goals / Non-Goals

Goals: optional local CLI translation, saved CLI authentication, unchanged API defaults, common validation, and observable failures.

Non-goals: a local HTTP adapter, persistent Codex sessions, automatic login, editing Codex configuration, provider fallback, and offline model inference.

## Decisions

1. Add `backend` (`api` or `codex-cli`) and `codex_path` (default `codex`). A direct CLI invocation is simpler than maintaining an adapter server; a persistent app-server would add lifecycle and protocol complexity without a demonstrated need.
   Expose these through native `userConfig`, placing common backend/language choices before backend-specific fields. Use the same configuration in the installation flow, `/plugin configure`, and `/config`; retain saved settings when switching. Document `plugin install --config` for hosts that support it. Source loading with `--plugin-dir` is a developer verification path.
2. Use an argv array with `codex exec --ephemeral --skip-git-repo-check --sandbox read-only --color never -`. Supply translation-only developer instructions and disable repository instruction loading for this invocation. Send the protected input exclusively through stdin, preserving multiline text without shell interpolation.
3. Inherit Codex authentication, provider, and model selection. Do not forward API-backend configuration or read its key in CLI mode. This is a local executable calling its configured model provider, not a guarantee of offline inference.
4. Set a 120-second process timeout. Reject nonzero exit status, truncated stdout, and empty output. Use only stdout, never stderr, as the translation. Do not expose raw process error output, which can contain private configuration.
5. Check cancellation before starting and after finishing translation. Existing marker and command-prefix checks run for both backends. Keep feedback state local to the submission and show the selected backend in the progress card.

## Risks / Trade-offs

CLI startup adds latency and each request starts a fresh session. Local configuration can include MCP servers, hooks, or model instructions that affect behavior or startup time. Read-only sandboxing does not mean that the CLI is offline or that every external integration is disabled. Document these limits without claiming tool isolation.

## Validation

Write failing mocked hook tests first, then implement the backend and configuration. Cover command construction, stdin, language routing, fragment restoration, failure cases, API bypass, and the progress card. Run strict plugin/marketplace validation, existing and new tests, TypeScript checking when available, and an adversarial code review. Run a small real translation separately to distinguish local process integration from mocked behavior.
