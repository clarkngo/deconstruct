# Deconstruct

A first-principles knowledge base hosted on GitHub Pages. It's a static site with no build step: `.github/workflows/static.yml` uploads the repo as-is on every push to `main`.

## Writing entries

The voice, rules, and required 5-section schema for every entry live in `initial-build-prompt.md`. Follow it exactly: elevator-pitch blockquote, then Pain Point → Aha → Step-by-Step → Trade-offs → Speak Like an Expert.

- Only claim what the source says. If you add outside knowledge (later critiques, follow-up work), mark it inline as coming from outside the source.
- End each entry with a `<p class="source">` line citing the source.
- When an entry mentions another topic by name (a predecessor, a rival method), link it inline: `[LOF](entry.html?e=local-outlier-factor)`. Link even if that entry isn't written yet.

## Researching the source

Sometimes the user names a topic instead of supplying a PDF. Then:

1. Find the seminal paper (the one that introduced the idea, not a survey or blog post) and confirm it with the user before writing: title, authors, venue, year, and a link.
2. Read the paper itself, from an open copy (arXiv, the authors' page, or the publisher's open version). If you can only get the abstract or secondary sources, say so, and don't write numbers or claims you couldn't check against the paper.
3. Write the entry from the paper as usual. The paper is the source; anything taken from later work or other sites follows the outside-knowledge rule above.

## Adding an entry

1. Write `entries/<slug>.md`. The slug is lowercase and hyphenated (`[a-z0-9-]+`).
2. Add an object to `entries.json` with `slug`, `title`, `summary` (one line), `tags`, `source`, `date` (YYYY-MM-DD), and `related`. The home page sorts newest first.
3. Reuse existing tags before inventing new ones (check `entries.json`). One spelling per idea, lowercase, e.g. always "machine learning", never "ML". Tags filter by exact match.

### Connections (`related`)

`related` is a list of `{ "slug": "...", "rel": "..." }`. `rel` is one of:

- `builds-on`: this entry depends on or extends the other one (a prerequisite to read first).
- `contrasts-with`: a competing or alternative approach to the same problem.
- `used-in`: this idea is a component of the other one.

Record each link once, on the entry that declares it. The other entry shows the reverse link automatically ("Built on by", "Uses", "Contrasts with"), so don't add it there too. Slugs for entries that don't exist yet are fine: they show greyed out as "not written yet", and an optional `"title"` sets their display name. When you add a new entry, check whether existing entries already point at its slug.

## How rendering works

- `index.html` reads `entries.json` and renders a searchable list.
- `entry.html?e=<slug>` fetches `entries/<slug>.md` and renders it with marked (pinned on jsDelivr).
- `entry.html` also reads `entries.json` and renders a Connections section (outgoing links, computed backlinks, tags) below the entry.
- Tag chips link to `./?tag=<tag>`, which filters the home page.
- A blockquote that starts with "The 10-Second Elevator Pitch" gets the `pitch` style. One that starts with "Note:" gets the `note` style.
- Pages use `fetch`, so preview locally with `python3 -m http.server`, not `file://`.
