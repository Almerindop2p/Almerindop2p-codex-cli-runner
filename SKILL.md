---
name: codex-cli-runner
description: run codex cli tasks with minimal tokens and least privilege. use when asked to run, assemble, debug, or resume codex exec commands; choose codex model, reasoning effort, sandbox, working directory, or stderr handling; migrate older gpt-5.2/full-auto codex instructions to gpt-5.5-compatible usage; or optimize codex cli usage for lower token, latency, and cost.
---

# Codex CLI Runner

Use this skill to run or construct Codex CLI commands safely, compatibly, and token-efficiently.

## Token and safety defaults

Prefer the smallest adequate run:

1. Reuse an existing session with `resume --last` when the user says continue, resume, or codex resume.
2. Ask only when missing information changes safety, cost, or write access. Otherwise apply defaults.
3. Default sandbox to `read-only`; use `workspace-write` for local edits; use `danger-full-access` only after explicit permission and only in an isolated/controlled environment.
4. Use `gpt-5.5` for complex or unclear tasks. Use `gpt-5.4-mini` for simple, cheap, fast, or repetitive tasks. If `gpt-5.5` is unavailable, fall back to `gpt-5.4`.
5. Choose low reasoning by default for token savings:
   - `minimal`: formatting, docs, tiny inspections.
   - `low`: simple fixes, small reviews, command generation.
   - `medium`: normal bug fixes, feature edits, refactors.
   - `high`: architecture, security, performance, broad refactors.
   - `xhigh`: only for ultra-complex tasks or explicit user request; if unsupported, use `high`.
6. Add `-c model_verbosity=low` and `-c model_reasoning_summary=none` unless the user wants more explanation.
7. Append `2>/dev/null` by default so progress on stderr is hidden and only the final stdout answer is shown.
8. Do not use `--full-auto`; it is deprecated. Use explicit `--sandbox workspace-write` for edit runs.
9. Include `--skip-git-repo-check` by default for one-off or uncertain directories.

## Decision table

| Task | Model | Effort | Sandbox |
|---|---:|---:|---:|
| command generation, docs, quick read | `gpt-5.4-mini` | `low` | `read-only` |
| repo review or analysis | `gpt-5.5` | `medium` | `read-only` |
| local edits or bug fix | `gpt-5.5` | `medium` | `workspace-write` |
| security, architecture, deep debugging | `gpt-5.5` | `high` | least required |
| network or broad machine access | `gpt-5.5` | `high` | `danger-full-access` only with permission |

## Build commands

Use this order for flags:

```bash
codex exec --skip-git-repo-check -m MODEL -c model_reasoning_effort=EFFORT -c model_verbosity=low -c model_reasoning_summary=none --sandbox SANDBOX -C DIR "PROMPT" 2>/dev/null
```

Omit `-C DIR` when no directory is needed. Use `printf` plus `-` for prompts that may contain quotes or newlines:

```bash
printf '%s\n' "PROMPT" | codex exec --skip-git-repo-check -m MODEL -c model_reasoning_effort=EFFORT -c model_verbosity=low -c model_reasoning_summary=none --sandbox SANDBOX -C DIR - 2>/dev/null
```

Examples:

```bash
codex exec --skip-git-repo-check -m gpt-5.4-mini -c model_reasoning_effort=low -c model_verbosity=low -c model_reasoning_summary=none --sandbox read-only "summarize this repository structure" 2>/dev/null
```

```bash
codex exec --skip-git-repo-check -m gpt-5.5 -c model_reasoning_effort=medium -c model_verbosity=low -c model_reasoning_summary=none --sandbox workspace-write -C ./app "fix the failing unit tests with the smallest safe change" 2>/dev/null
```

## Resume workflow

When continuing a previous Codex exec session, inherit the original model, reasoning effort, and sandbox. Do not add model/config/sandbox flags unless the user explicitly changes them.

```bash
printf '%s\n' "FOLLOW_UP_PROMPT" | codex exec --skip-git-repo-check resume --last - 2>/dev/null
```

If a directory matters for finding the last session, place `-C DIR` before `resume`:

```bash
printf '%s\n' "FOLLOW_UP_PROMPT" | codex exec --skip-git-repo-check -C DIR resume --last - 2>/dev/null
```

After a successful Codex run, tell the user: `You can resume this Codex session at any time by saying 'codex resume' or asking me to continue with additional analysis or changes.`

## Error handling

If `codex exec` exits non-zero:

1. Report the failure briefly.
2. Re-run without `2>/dev/null` only when stderr is needed for debugging.
3. Do not retry with broader sandbox, network, or higher reasoning without user approval.
4. If the model is rejected, try the documented fallback chain: `gpt-5.5` → `gpt-5.4` → `gpt-5.4-mini`.
5. If command syntax fails, check `codex --version` and adapt to the installed CLI version.

## Response format

After each run, keep the user-facing summary short:

- command intent, not full command, unless the user asks;
- selected model, effort, and sandbox;
- outcome, changed files, warnings, or next step;
- resume reminder after successful runs.

For token savings, avoid long transcripts, stderr dumps, or full command echoes unless they are needed for debugging or requested by the user.
