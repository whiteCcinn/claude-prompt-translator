## 1. Specify and test
- [x] 1.1 Inspect the existing hook and verify local Codex and Claude process APIs.
- [x] 1.2 Document the backend design and acceptance scenarios.
- [x] 1.3 Write CLI hook tests and observe the expected failures before implementation.

## 2. Implement
- [x] 2.1 Add CLI configuration, safe process invocation, and shared validation.
- [x] 2.2 Update progress feedback and user/developer documentation.

## 3. Verify
- [x] 3.1 Pass all mocked tests and strict plugin/marketplace validation.
- [x] 3.2 Validate the OpenSpec change and review the diff adversarially.
- [x] 3.3 Check a real Codex translation and report its exact verification scope.
- [x] 3.4 Verify normal installation configuration and later backend switching in an isolated Claude Code configuration directory.

## Evidence

- TDD red: 15 passing / 10 failing tests before implementation; failures were missing CLI translation behavior.
- TDD green: `claude plugin test plugins/prompt-translator`: 25 passing / 0 failing.
- `claude plugin validate plugins/prompt-translator --strict` and `claude plugin validate . --strict`: exit 0. The plugin validator reports a catalog icon warning, unrelated to this feature.
- `openspec validate add-codex-cli-translation --strict`: valid.
- Independent TypeScript adversarial review: no confirmed new blocking issue. `tsc` and `eslint` are unavailable locally; no claim of a completed typecheck/lint run.
- Live check: Node 24 imported the actual `register.ts`, invoked its SDK-origin submit handler, and adapted the process API to a real shell-free Codex child process. Input ``请检查 `/tmp/demo.go` 的并发问题。`` produced ``Please check `/tmp/demo.go` for concurrency issues.`` plus the English reply context. Codex CLI 0.166.0; Claude Code process interface checked against generated 2.1.316 types.
- Live verification does not include the Claude model response or a complete interactive UI session. Cancellation blocks later submission; immediate process termination is not promised beyond host behavior and the 120-second timeout.
- Native installation/configuration check on Claude Code 2.1.316: an isolated `CLAUDE_CONFIG_DIR` installed a copied local marketplace with `--config backend=codex-cli --config language=jp` and a spaced `codex_path`; configuration inspection confirmed Codex selected without an API key. Later switches through `configure --values-stdin` retained the CLI path, language, API URL, and API model. The isolated installation was removed afterward. This confirms native schema/configuration persistence; it does not claim a visual wizard inspection or a published marketplace release.
