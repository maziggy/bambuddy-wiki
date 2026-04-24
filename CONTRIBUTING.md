# Contributing to the Bambuddy Wiki

Thanks for helping improve the docs! This repo is the source for
<https://wiki.bambuddy.cool> — content is Markdown, rendered with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Quick edit (no local clone needed)

Both of these work directly in your browser:

- **GitHub pencil icon** — click any file under `docs/`, click the
  pencil in the top-right of the file view, and edit inline. Best for
  small changes (one or two files).
- **github.dev** — press `.` (period) anywhere in this repo to open a
  full VS Code session in your browser with the entire file tree,
  syntax highlighting, multi-file edits, and find-and-replace. Best for
  anything larger than a typo.

### Submitting your change

1. After editing, scroll to the bottom of the page.
2. Select **"Create a new branch for this commit and start a pull
   request"** (direct commits to `main` are blocked — this option is
   the only one you'll see).
3. Give the branch a short, descriptive name — e.g.
   `fix/typo-installation`, `docs/add-ams-example`,
   `update/firmware-table`.
4. On the pull request page, fill in the template (Summary + Preview
   checklist) and click **"Create pull request"**.

## What happens after you submit

A maintainer will review your PR and merge it. If something needs
adjustment, you'll get feedback on the PR.

### Check your edit before requesting review

If you want to preview locally before opening the PR, clone the repo
and run `mkdocs serve` — that gives you the real Material theme with
the sidebar, search, and live reload.

Things to check:

- Does your edit render correctly? (headings, lists, code blocks,
  admonitions)
- Do the sidebar navigation and TOC still work?
- Are internal links (`[see here](../features/ams.md)`) still
  resolving?
- If you uploaded an image, does it load?
- On mobile width, does the layout still work?

## Writing style

- **Headings**: start at `##` — the page title comes from the file
  path via MkDocs, so don't open a file with `#`.
- **Code blocks**: always specify a language —
  ` ```bash`, ` ```yaml`, ` ```python`, etc.
- **Admonitions**: use MkDocs Material syntax —
  `!!! tip`, `!!! warning`, `!!! note`.
- **Internal links**: use relative paths
  (`../features/ams.md`), not absolute URLs.
- **Images**: commit to `docs/assets/`, reference as
  `![alt text](/assets/yourfile.png)`.

## Reporting issues or requesting new docs

If you'd rather flag a missing or outdated topic without writing it
yourself, open an issue at
[maziggy/bambuddy/issues](https://github.com/maziggy/bambuddy/issues)
and apply the `documentation` label. We use the main repo's issue
tracker for everything — this repo doesn't track its own issues.

## Docs tied to code changes

If you're adding documentation for a new feature that's also being
added to the main [bambuddy](https://github.com/maziggy/bambuddy) repo,
open both PRs in parallel and link them to each other. See the main
repo's [CONTRIBUTING.md](https://github.com/maziggy/bambuddy/blob/main/CONTRIBUTING.md#documentation-requirements)
for the full rules.
