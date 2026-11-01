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

For the CLI backend, choose `codex-cli` in the loaded plugin's **Translation backend** configuration. Leave **Codex executable** as `codex` or use an executable path without arguments. Verify that the progress card names Codex CLI, a protected path survives translation, and a failed Codex process stops submission without an API fallback. Real CLI runs use the user's configured provider and quota; keep them separate from mock tests.

`codex_model` and `codex_reasoning_effort` override only translator subprocesses, for both directions. Blank model and `default` effort must omit their CLI arguments. Keep API model/effort independent, with its existing `low` default. Test argument boundaries and inheritance without starting a real model.

The CLI invocation is verified with Codex CLI 0.166.0 and Claude Code 2.1.316. Its read-only sandbox and disabled shell tool do not disable every user-configured MCP server or hook.

Language routing defaults to Chinese input, English prompts, inherited English Claude replies, and Chinese output. Keep `language` as the prompt-language key; configure input, Claude replies, and output independently. `same-as-claude` skips outbound translation.

The `turn.step` hook rewrites both displayed text and the transcript, so subsequent Claude requests read translated history. Buffer complete response text to protect split code fences; preserve non-text chunks and references. A failed translation must not reveal untranslated source as if translation succeeded or issue another main-model request. Cover concurrency, retry boundaries, cancellation while reading credentials, truncation, and bounded buffering in tests. Opaque engine refs must come from the host in native tests; use pure shaping tests for fabricated protocol fixtures.

The shared feedback lifecycle uses top-level named helpers so Claude Code can statically trace host capabilities. Do not pass `on` to a controller factory or `$` to object methods or nested helpers: the native loader rejects those shapes even when TypeScript passes. Test both feedback stages, output-only timing, cancellation and ownership after asynchronous clock reads. Terminal rendering also identifies an interactive session after plugin reload; headless runs must not create UI timers.

After Claude generates the development-only type declarations, run strict checking with `npx --yes --package typescript@5.9.3 tsc --noEmit -p plugins/prompt-translator`. OpenSpec stays local under the ignored `openspec/` directory and must not be committed.

Changes should preserve commands, source origin, paths, URLs, inline code, and fenced code. Never submit untranslated source text after a translation failure. Keep credentials, session transcripts, personal directories, and provider response logs out of Git.

The marketplace entry and plugin manifest must use the same name and version. Update both when publishing a release. Contributions are licensed under MIT.
