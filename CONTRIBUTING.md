# Contributing

The plugin runs inside Claude Code. There is no npm runtime dependency or separate translation server. The optional `codex-cli` backend requires a locally installed and authenticated Codex CLI; API remains the default backend.

Use Claude Code 2.1.316 for the current bidirectional translation validation baseline:

```bash
claude plugin validate . --strict
claude plugin validate plugins/prompt-translator --strict
claude plugin test plugins/prompt-translator
```

Tests mock HTTP and process calls and do not require an API key, Codex installation, or Claude session authentication. Cover CLI invocation and failures without calling a real model in the automated suite.

For a live check, load the source plugin and configure a translation provider:

```bash
claude --plugin-dir ./plugins/prompt-translator
```

Do not load another prompt-translation plugin in the same test session. Verify the rewritten model-turn input separately from the translation API response.

For the CLI backend, choose `codex-cli` in **Translation backend**. Keep **Codex executable** on `跟随本地配置` (PATH), or choose `手动输入` and fill the separate executable row without arguments. Verify that the progress card names Codex CLI, a protected path survives translation, and a failed Codex process stops submission without an API fallback. Real CLI runs use the user's configured provider and quota; keep them separate from mock tests.

`codex_model_selection` offers inheritance, official presets (checked 2026-11-01), and manual input; `codex_model` remains the manual/legacy value. `codex_reasoning_effort` also offers inheritance and a separate manual value. Resolve these before invoking either translation direction. Inheritance must omit overrides, and UI labels must never reach argv. Keep API model/effort independent, with its existing `low` default. Test argument boundaries without starting a real model.

`codex_fast_mode` is independent: `跟随本地配置` (legacy `default`) omits overrides; `on` passes `service_tier="fast"` and `features.fast_mode=true`; `off` passes `service_tier="default"` and `features.fast_mode=false` to replace inherited Fast preferences. Both input and output use it. Keep API requests and main Claude model/effort unchanged. Offline loopback probes may inspect real Codex request shaping with a dummy provider/key; they do not establish paid-tier availability or performance.

Native text rows do not display unsaved schema defaults. Keep primary controls as explicit pickers and manual values in separately labeled rows. Preserve old text keys: changing them directly to enums discards custom IDs. Use the unique provider of this plugin's `config.list` rows as its exact configuration ID; the native provider includes `@marketplace` or `@inline`. Do not infer identity from filesystem paths: local marketplace installs can use the same root as inline plugins. Only user, flag and policy layers configure plugin options; project/local settings must not participate. Saved-field presence reads are lazy and shared per activation. Opening this plugin's native configuration initializes missing user selectors through `config.set`, skipping fields owned by flag/policy or locked rows. Both locked legacy fields and explicitly locked selectors remain authoritative. Never rewrite raw values or secrets or choose another installation's settings. API Key retains native masking/storage; manual credential mode must not fall back to environment credentials. Cover upgrade preservation, explicit reset, active-backend-only validation, and preset/manual switching.

The CLI invocation is verified with Codex CLI 0.166.0 and Claude Code 2.1.316. Its read-only sandbox and disabled shell tool do not disable every user-configured MCP server or hook.

Language routing defaults to Chinese input, English prompts, inherited English Claude replies, and Chinese output. Keep `language` as the prompt-language key; configure input, Claude replies, and output independently. `same-as-claude` skips outbound translation.

The `turn.step` hook rewrites both displayed text and the transcript, so subsequent Claude requests read translated history. Buffer complete response text to protect split code fences; preserve non-text chunks and references. A failed translation must not reveal untranslated source as if translation succeeded or issue another main-model request. Cover concurrency, retry boundaries, cancellation while reading credentials, truncation, and bounded buffering in tests. Opaque engine refs must come from the host in native tests; use pure shaping tests for fabricated protocol fixtures.

The shared feedback lifecycle uses top-level named helpers so Claude Code can statically trace host capabilities. Do not pass `on` to a controller factory or `$` to object methods or nested helpers: the native loader rejects those shapes even when TypeScript passes. Test both feedback stages, output-only timing, cancellation and ownership after asynchronous clock reads. Terminal rendering also identifies an interactive session after plugin reload; headless runs must not create UI timers.

After Claude generates the development-only type declarations, run strict checking with `npx --yes --package typescript@5.9.3 tsc --noEmit -p plugins/prompt-translator`. OpenSpec stays local under the ignored `openspec/` directory and must not be committed.

Changes should preserve commands, source origin, paths, URLs, inline code, and fenced code. Never submit untranslated source text after a translation failure. Keep credentials, session transcripts, personal directories, and provider response logs out of Git.

The marketplace entry and plugin manifest must use the same name and version. Update both when publishing a release. Contributions are licensed under MIT.
