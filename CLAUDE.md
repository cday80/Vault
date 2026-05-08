# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is an **Obsidian vault** — a directory of Markdown notes managed by the Obsidian app — not a software project. There is no build system, no tests, and no source code to compile. The "code" here is configuration for Obsidian itself under `.obsidian/`, plus the user's notes as `.md` files at the root and in subfolders.

Remote: https://github.com/cday80/Vault.git

## Backup automation (important context for any commit you see)

The `obsidian-git` community plugin (`.obsidian/plugins/obsidian-git/data.json`) auto-commits **every 1 minute** when files change, using the message template `vault backup: {{date}}`. It also auto-pulls on boot and pulls before pushing (merge strategy). Implications:

- Most commits in `git log` are automated backups, not meaningful change history. Don't try to derive intent from them.
- Manual commits should use a descriptive message so they stand out from `vault backup: …` noise.
- Before doing any work that touches files, expect that uncommitted changes may appear within ~60 seconds from the running Obsidian instance. Avoid long-lived branches or rebases against this branch — coordinate with the user first.
- `.obsidian/workspace.json` and similar UI-state files churn constantly. Don't treat their diffs as signal.

## Enabled community plugins

Listed in `.obsidian/community-plugins.json`:
- `obsidian-git` — the backup automation above.
- `terminal` — embedded terminal inside Obsidian.

## Working in this repo

- Treat note edits as content changes, not code changes. Preserve the user's voice, formatting, and any wikilinks (`[[…]]`) or tags (`#…`).
- Don't reorganize, rename, or "clean up" notes unless explicitly asked — links between notes break silently when files move.
- New notes go at the vault root or in an existing folder the user points to; don't invent a folder hierarchy.
