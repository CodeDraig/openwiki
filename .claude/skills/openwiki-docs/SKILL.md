---
name: openwiki-docs
description: Generate or maintain an OpenWiki-style documentation set under openwiki/ for the current repository. Use when asked to write initial repo documentation, refresh docs after code changes, or otherwise keep openwiki/ current with the codebase.
---

# OpenWiki documentation skill

You are acting as an expert technical writer, software architect, and product analyst. Your job is to inspect the current repository and produce documentation under `openwiki/` that is excellent for both humans and future coding agents.

Ground every important claim in source files, existing docs, or git evidence you have actually inspected. Do not invent files, modules, APIs, business rules, or behavior.

## Determine the run type

Before doing anything else, decide which mode applies:

- **Init** — `openwiki/quickstart.md` does not exist, or `openwiki/` has no useful documentation yet.
- **Update** — `openwiki/quickstart.md` already exists and contains real content.

This determines the workflow sections below.

## Run discipline

- Prefer targeted filesystem discovery (list directories, grep, targeted reads) over exhaustively reading every file.
- Inspect the repository tree, package/config files, README-style files, entrypoints, routing files, database/schema files, and representative files for each major domain.
- Do not glob or list recursively from the repository root without excludes. Use targeted discovery by directory and extension, or a files-listing command that excludes `.git`, `node_modules`, `dist`, `build`, cache directories, and the existing `openwiki/` output.
- Prefer grep/glob and short targeted reads over full-file reads when files are large.
- Create a strong first-pass wiki that is accurate and navigable, then stop. It can be refined in later runs.
- Keep the documentation set focused: quickstart plus the smallest set of section pages needed to explain the repo clearly.
- Do not run commands that search or write outside the target repository.

## Subagent discipline

- If your environment supports parallel subagents/tasks, use them to parallelize read-only research on init and update runs when the repository has multiple substantial domains.
- Default to 1-2 subagents for large or unfamiliar repositories. Use 3-4 only when the repository is clearly small/medium, the domains are naturally independent, or the user explicitly asks for deeper research.
- Subagents must only inspect and summarize. They must not create, edit, delete, or move files, and they must not write to `openwiki/`.
- Give each subagent a narrow brief such as existing docs, runtime architecture, data/storage, UI/API surface, integrations, tests/evals, or business workflows.
- Ask each subagent to return concise findings with source paths and open questions. You are responsible for synthesizing the final docs and for all writes.
- Treat subagent reports as internal discovery notes, not final content.
- If your environment has no subagent/task capability, perform discovery sequentially instead.

## Planning discipline

- After discovery and before writing final documentation, create a temporary `openwiki/_plan.md` that lists the intended wiki pages, source evidence for each page, and remaining questions.
- Before finishing the run, delete `openwiki/_plan.md`. Do not leave it in the final wiki.

## Git discipline

- Use git heavily to explain *why* code exists, not just what it contains. Run `git log`, `git show`, `git blame`, `git status`, and `git diff` yourself as needed — nothing is pre-gathered for you.
- **Init**: inspect recent commit history (e.g. `git log --max-count=20 --name-status --oneline`) and use `git show`/`git blame` selectively on important files to understand how major workflows, entrypoints, and business rules evolved. Do not over-index on ancient history — focus on recent, high-signal commits.
- **Update**: read `openwiki/.last-update.json` if it exists. Prefer its `gitHead` field to scope `git log <gitHead>..HEAD --name-status --oneline`; fall back to its `updatedAt` timestamp with `git log --since <updatedAt>` if there is no `gitHead`. If there is no metadata file at all, treat it like init discovery for git history. Also check `git status --short` and `git diff --name-status HEAD` for uncommitted local changes that touch docs or important source files.

## Existing documentation discipline

- Treat existing README files, `docs/` trees, root documentation files, runbooks, and `SKILL.md` files (including this one) as primary source material.
- Summarize and link to existing docs when they are still useful instead of duplicating them wholesale.
- If existing docs conflict with source code or git history, call out the likely stale documentation and prefer current source evidence.

## Root agent instruction files

Unless explicitly told not to, make sure the repository's top-level agent instruction files reference the OpenWiki quickstart:

- Only consider top-level `/AGENTS.md` and `/CLAUDE.md`. Do not edit nested `AGENTS.md`/`CLAUDE.md` files.
- If either file exists, add or update the OpenWiki reference section in it. If both exist, add the same section to both.
- If neither exists, create a top-level `/AGENTS.md` containing only the OpenWiki reference section.
- On update runs, refresh the existing section only if it is missing or semantically stale — do this check even when the wiki itself is otherwise current.
- Preserve surrounding instructions in existing files. Replace/update an existing OpenWiki section instead of duplicating it. Do not touch the file just to normalize formatting if the section is already semantically correct.
- Use this exact section structure every time:

```markdown
## OpenWiki

This repository has documentation located in the /openwiki directory.

Start here:
- [OpenWiki quickstart](openwiki/quickstart.md)

OpenWiki includes repository overview, architecture notes, workflows, domain concepts, operations, integrations, testing guidance, and source maps.

When working in this repository, read the OpenWiki quickstart first, then follow its links to the relevant architecture, workflow, domain, operation, and testing notes.
```

## Security and privacy rules

- Do not read or document secret values, credentials, private keys, tokens, `.env` files, or other sensitive material. `.env.example` and other sample configs may be read only if they contain placeholders, not live secrets.
- If a secret-bearing file is relevant, document only that such configuration exists and where non-sensitive setup should be described.
- Keep all documentation under `openwiki/`.
- Do not modify source code outside `openwiki/`. The only allowed exceptions are top-level `/AGENTS.md` and `/CLAUDE.md`, and only for the OpenWiki reference section above.

## Documentation goals

- Someone with zero knowledge of the repository should be able to start at `openwiki/quickstart.md` and understand what the project is, how it is organized, what it does, and where to go next.
- A future agent should be able to use the docs to make high-quality code changes with less source exploration.
- Capture both technical details and business/product logic. Explain why important code exists, not only what files contain.
- Prefer clear Markdown with stable links between pages.
- Organize the docs like human documentation, not a raw file inventory.
- Include change-oriented guidance for future agents: where to start, what to watch out for, and which tests or checks are relevant when changing each major area.
- Keep docs concise enough to maintain. Give each concept one canonical home and link to it from other pages instead of repeating it.
- Use git history for discovery, but do not include persistent commit hash lists in documentation unless a specific historical decision is important for future work.

## Section quality rules

- Do not create a directory unless it represents a real documentation area.
- A section directory should usually contain multiple substantive pages. A single-file directory is acceptable only when that page is substantial, has a clear domain boundary, and is likely to grow.
- Avoid thin pages. If a page would mostly be a stub, source map, or short note, merge it into `openwiki/quickstart.md` or a broader section page instead.
- Each page should provide real explanatory value: what the area does, why it exists, where to start, what to watch out for, and key source references.
- Before finishing, review the `openwiki/` tree and merge, move, or remove low-value single-file directories and stub pages.
- For small repositories (about 10 or fewer primary source files), prefer `openwiki/quickstart.md` plus at most 1-2 supporting pages.

## Required documentation structure

- `openwiki/quickstart.md` must be the entrypoint, with a high-level repository overview and links to every major section.
- When the repository is large enough to need section directories, use one directory per major section — e.g. `architecture/`, `workflows/`, `domain/`, `api/`, `data-models/`, `operations/`, `integrations/`, `testing/`, or names that fit the repo.
- Each section directory should contain focused Markdown pages; if a directory would contain only one short page, prefer a broader page or a heading in `openwiki/quickstart.md` instead.
- Include source-file references inline where they help readers verify or continue exploring. Source Map sections are optional — add one only when it materially improves navigation.

## Workflow: init

- Assume `openwiki/` does not yet contain useful documentation. Build the structure from scratch.
- First build a repository inventory: existing docs, entrypoints, package/config files, major domain folders, tests/evals, data/schema files, skill/playbook files, and operational scripts.
- If the repo already has substantial docs, create a wiki that functions as an opinionated map and synthesis layer over those docs rather than duplicating them.
- Create `openwiki/quickstart.md` first, then the linked section pages.
- Use at most 8 documentation pages on the initial run unless the repository is clearly large.
- Do not try to document every source file — document the main architecture, workflows, domain concepts, data models, integrations, operations, tests, and known extension points at the right level of detail.

## Workflow: update

- Inspect the existing `openwiki/` documentation before editing.
- Before editing, build a docs impact plan from the changed source files: source change → docs affected → edit needed → why. If a page cannot be tied to a relevant source, workflow, product, or existing-doc change, do not edit it.
- Be surgical. Preserve useful existing structure and wording when it remains accurate. Prefer replacing one stale sentence over adding new paragraphs. Only edit pages whose current content is inaccurate, incomplete, or misleading because of recent changes — do not refresh every page.
- Keep each concept in one canonical page. If the same detail appears in multiple pages, keep the detailed explanation in the canonical page and make other mentions brief or link-only.
- Do not make formatting-only edits (reformatting tables, normalizing blank lines, reordering source lists, polishing wording) unless the surrounding content is already being changed for accuracy.
- Do not update Source Map sections, git evidence lists, or generic "things to watch" sections unless they are materially wrong because of the source changes.
- Use a soft diff budget: if fewer than about 5 source files changed, update at most 1-2 wiki pages. Avoid touching quickstart unless top-level product behavior, setup, or navigation changed. If more than 3 wiki pages seem to need edits, stop and reconsider why before making broad changes.
- Updates may be a no-op. If there are no relevant changes since the previous run and the wiki is already accurate, do not edit files — say so.

## Update metadata

After finishing an init or update run **where you actually changed content under `openwiki/`**, write or overwrite `openwiki/.last-update.json` with:

```json
{
  "updatedAt": "<current UTC ISO-8601 timestamp>",
  "command": "init | update",
  "gitHead": "<current `git rev-parse HEAD`>",
  "model": "<your model identifier, if known>"
}
```

If you made no changes to `openwiki/` content during this run (a true no-op update), leave `openwiki/.last-update.json` untouched — do not update the timestamp just because the run happened. This keeps scheduled/CI-triggered runs from producing metadata-only churn.
