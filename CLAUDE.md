# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A deliberately minimal to-do app: plain HTML/CSS/JS, **no frameworks, no build step, no package manager, no tests**. Keep it that way — don't introduce dependencies or tooling.

Note: the parent folder name is `CCC ` (trailing space), so quote the path in shell commands: `/Users/wenqi/Documents/CCC /To Do/`.

## Running

Open `index.html` directly in a browser (double-click or `open index.html`). There is nothing to build or lint. Node and python3 are not installed; ruby is (`/usr/bin/ruby`).

## Architecture

- `index.html` — the entire app (markup, `<style>`, `<script>` in one file).
  - State is a single array `todos = [{text, done, category, due}]`, persisted to `localStorage` under `todos`; the theme is stored under `theme`. All storage access is wrapped in try/catch so the app still works when storage is blocked.
  - `render()` rebuilds the whole list DOM from state: groups by the `CATEGORIES` array order (`Work`, `Personal`, `Others`; empty groups skipped), sorting each group by due date with undated items last. Items are referenced by object identity (the `editing` variable holds the todo being edited), not by index. Text is always set via `textContent`.
  - On load, todos whose category isn't in `CATEGORIES` are coerced to `Others` and a missing `due` becomes `''`. If you change the category list, keep this migration in mind.
  - Theming: CSS variables on `:root` and `:root[data-theme="dark"]`; defaults to `prefers-color-scheme`, toggled by the icon button (which shows the mode you would switch *to*).
  - Layout must hold at ~360px width (list rows don't wrap; `.main` / `.actions` wrappers in `li` exist for that).
- `mockup.html` — **generated** side-by-side iPhone / Galaxy S25 preview that embeds `index.html` in iframes (`srcdoc`) with a fake `localStorage` seeded with demo data (so the real list is never touched). Don't hand-edit it; it is a snapshot and goes stale whenever `index.html` changes.
  - The generator is a ruby script that lived in the session scratchpad and may no longer exist. If missing, recreate it: read `index.html`, inject a seeded fake `localStorage` script right after `<head>`, HTML-escape the result (`CGI.escapeHTML`) into each iframe's `srcdoc`, and write `mockup.html`.

## Testing notes

The Claude desktop app's browser pane loads local files as `data:` snapshots, where `localStorage` throws a SecurityError — persistence can't be verified there. Check persistence by opening the file in a normal browser.
