# Contributing

The plugin runs inside Claude Code. There is no npm runtime dependency or separate translation server. The optional `codex-cli` backend requires a locally installed and authenticated Codex CLI; API remains the default backend.

Use Claude Code 2.1.291 for the current validation baseline:

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

The CLI invocation is verified with Codex CLI 0.166.0 and Claude Code 2.1.316. Its read-only sandbox and disabled shell tool do not disable every user-configured MCP server or hook.

Changes should preserve commands, source origin, paths, URLs, inline code, and fenced code. Never submit untranslated source text after a translation failure. Keep credentials, session transcripts, personal directories, and provider response logs out of Git.

The marketplace entry and plugin manifest must use the same name and version. Update both when publishing a release. Contributions are licensed under MIT.
