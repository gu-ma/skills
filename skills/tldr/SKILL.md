---
name: tldr
description: Use llm-tldr to summarize and navigate codebases with structural context and semantic search. Trigger this skill when Hermes needs to understand an unfamiliar repository, trace behavior when exact symbol names are unknown, estimate change impact before editing, or reduce token usage by reading structural summaries instead of raw code. Prefer ripgrep for exact string, filename, symbol, or regex searches.
version: 1.0.0
platforms: [macos, linux]
metadata:
  hermes:
    tags: [code-analysis, semantic-search, llm-tldr, ripgrep, token-efficiency]
    category: developer-tools
    requires_toolsets: [terminal]
---

# Code Context Map

## When to Use
Use this skill when Hermes needs to understand a codebase efficiently without reading large amounts of raw code.

Typical triggers:
- The user asks how a feature works in a repository.
- The user wants a module, file, or subsystem summarized before making edits.
- The user describes behavior but does not know the exact symbol or filename.
- The user wants to estimate what code may be affected by a change.
- Hermes should reduce token usage by using structural summaries instead of large raw file dumps.

Prefer `rg` when the task is:
- finding an exact string
- locating a known symbol or filename
- matching a regex
- checking for literal occurrences quickly

Prefer `llm-tldr` when the task is:
- understanding structure
- finding semantically related logic
- tracing relationships between functions or files
- summarizing context before reasoning or editing
- estimating impact of a change

## Procedure
1. Confirm that the task is code-oriented and benefits from structure-aware summarization.
2. Check whether `llm-tldr` is available through `uvx` or as an installed tool.
3. Avoid modifying Hermes' own project environment unless the user explicitly asks for that.
4. Prefer one of these invocation styles:
   - `uvx --from llm-tldr tldr --help`
   - `uv tool install llm-tldr`
   - `tldr --help`
5. If the repository has not been indexed yet, warm the index from the project root:
   - `uvx --from llm-tldr tldr warm .`
   - or `tldr warm .`
6. Choose the lightest command that answers the question:
   - Use `tldr context <path-or-symbol>` for a structural summary of a file, function, or module.
   - Use `tldr semantic "<natural language query>"` when the user describes behavior rather than exact names.
   - Use `tldr impact <path-or-symbol>` before edits, refactors, or dependency-sensitive changes.
7. If needed, combine with `rg`:
   - use `rg` first to narrow candidate files
   - then use `tldr` on the best candidates to build a concise map
8. Return a compact answer that includes:
   - what was analyzed
   - a concise structural summary
   - important files, functions, or symbols
   - likely next inspection targets
   - any uncertainty or gaps
9. Only read full files when `llm-tldr` output is insufficient or the user explicitly asks for exact source-level confirmation.

## Pitfalls
- `llm-tldr` is not a replacement for `rg`. Do not use it for simple literal or regex lookup when `rg` is faster and clearer.
- Do not assume the repo is already indexed. If results look incomplete, run `tldr warm .` first.
- Some AST-based commands (`context`, sometimes `semantic`) can fail or degrade when unrelated files in the repo have syntax errors. If that happens, fall back to `tldr structure`, `tldr extract`, or `tldr search` on narrower paths instead of treating the whole tool as broken.
- Do not over-read raw files after getting a good structural summary; that defeats the token-saving goal.
- `uvx --from llm-tldr tldr ...` working does not mean bare `tldr` is installed in PATH. In ephemeral shells, keep using the full `uvx --from llm-tldr tldr ...` form for every command unless `tldr --help` succeeds directly.
- If the CLI is unavailable, fall back to normal file inspection and Hermes file-search tools rather than failing silently. Use `rg` only when it actually exists in the environment.
- For highly precise edits, verify critical details against the actual source before making claims about exact behavior.
- This skill is optimized for code and structured repositories, not generic prose documents.

## Verification
Confirm the skill worked if:
- `tldr --help` or `uvx --from llm-tldr tldr --help` succeeds
- `tldr warm .` completes successfully on the repository
- `tldr context`, `tldr semantic`, or `tldr impact` returns a useful structural summary
- the final answer is shorter and more targeted than reading equivalent raw code
- Hermes can identify relevant files, symbols, and likely next steps with less context usage than direct file dumping