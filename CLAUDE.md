# Deconstruct

A first-principles knowledge base hosted on GitHub Pages. It's a static site with no build step: `.github/workflows/static.yml` uploads the repo as-is on every push to `main`.

## Writing entries

The voice, rules, and required 5-section schema for every entry live in `initial-build-prompt.md`. Follow it exactly: elevator-pitch blockquote, then Pain Point → Aha → Step-by-Step → Trade-offs → Speak Like an Expert.

- Only claim what the source says. If you add outside knowledge (later critiques, follow-up work), mark it inline as coming from outside the source.
- End each entry with a `<p class="source">` line citing the source.

## Adding an entry

1. Write `entries/<slug>.md`. The slug is lowercase and hyphenated (`[a-z0-9-]+`).
2. Add an object to `entries.json` with `slug`, `title`, `summary` (one line), `tags`, `source`, `date` (YYYY-MM-DD). The home page sorts newest first.

## How rendering works

- `index.html` reads `entries.json` and renders a searchable list.
- `entry.html?e=<slug>` fetches `entries/<slug>.md` and renders it with marked (pinned on jsDelivr).
- A blockquote that starts with "The 10-Second Elevator Pitch" gets the `pitch` style. One that starts with "Note:" gets the `note` style.
- Pages use `fetch`, so preview locally with `python3 -m http.server`, not `file://`.
