---
name: llm-tldr
description: summarize and navigate codebases using llm-tldr structural context and semantic search. use when an assistant needs to understand an unfamiliar repository, summarize a file, module, or function, find code by behavior when exact symbol names are unknown, trace control flow or data flow for debugging, estimate change impact before editing or refactoring, or reduce token usage by reading structural summaries instead of raw files. prefer ripgrep for exact string, symbol, filename, or regex search.
version: 1.0.0
platforms: [macos, linux]
metadata:
  hermes:
    tags: [code-analysis, semantic-search, llm-tldr, ripgrep, token-efficiency]
    category: developer-tools
    requires_toolsets: [terminal]
---

# Codebase Understanding with llm-tldr

## When to Use
Use this skill to understand a codebase efficiently without reading large amounts of raw code.

Typical triggers:
- The user asks how a feature works in a repository.
- The user wants a file, module, or function summarized before making edits.
- The user describes behavior but does not know the exact symbol or filename.
- The user wants to trace control flow, data flow, or related logic while debugging.
- The user wants to estimate what code may be affected by a change.
- Structural summaries would be more efficient than reading large raw file dumps.

Prefer `rg` when the task is:
- finding an exact string
- locating a known symbol or filename
- matching a regex
- checking for literal occurrences quickly

Prefer `llm-tldr` when the task is:
- understanding structure
- finding semantically related logic
- tracing relationships between functions or files
- tracing control flow or data flow for debugging
- summarizing context before reasoning or editing
- estimating impact of a change

## Procedure
1. Confirm that the task is code-oriented and benefits from structure-aware summarization.
2. Check whether `llm-tldr` is available through `uvx` or as an installed tool.
3. Avoid modifying the project environment unless explicitly requested.
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
   - For debugging-oriented questions, use `tldr semantic`, `tldr context`, or narrower follow-up commands to trace likely control flow, data flow, and related call paths.
7. If needed, combine with `rg`:
   - use `rg` first to narrow candidate files
   - then use `tldr` on the best candidates to build a concise map
8. Return a compact answer that includes:
   - what was analyzed
   - a concise structural summary
   - important files, functions, or symbols
   - likely next inspection targets
   - any uncertainty or gaps
9. Only read full files when `llm-tldr` output is insufficient or exact source-level confirmation is needed.

## Pitfalls
- `llm-tldr` is not a replacement for `rg`. Do not use it for simple literal or regex lookup when `rg` is faster and clearer.
- Do not assume the repo is already indexed. If results look incomplete, run `tldr warm .` first.
- Some AST-based commands (`context`, sometimes `semantic`) can fail or degrade when unrelated files in the repo have syntax errors. If that happens, fall back to `tldr structure`, `tldr extract`, or `tldr search` on narrower paths instead of treating the whole tool as broken.
- Do not over-read raw files after getting a good structural summary; that defeats the token-saving goal.
- `uvx --from llm-tldr tldr ...` working does not mean bare `tldr` is installed in PATH. In ephemeral shells, keep using the full `uvx --from llm-tldr tldr ...` form for every command unless `tldr --help` succeeds directly.
- If the CLI is unavailable, fall back to normal file inspection and available file-search tools rather than failing silently. Use `rg` only when it actually exists in the environment.
- For highly precise edits, verify critical details against the actual source before making claims about exact behavior.
- This skill is optimized for code and structured repositories, not generic prose documents.

## Verification
Confirm the skill worked if:
- `tldr --help` or `uvx --from llm-tldr tldr --help` succeeds
- `tldr warm .` completes successfully on the repository
- `tldr context`, `tldr semantic`, or `tldr impact` returns a useful structural summary
- the final answer is shorter and more targeted than reading equivalent raw code
- relevant files, symbols, and likely next steps can be identified with less context usage than direct file dumping