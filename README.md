# Design Atlas

A growing reference library of how design actually gets made — the working vocabulary,
the governing numbers, and the visual tells — assembled discipline by discipline.

It begins with **Industrial Design**, whose first topic is **Surface & Substance**: an
atlas of surfacing, manufacture, automotive, appliance, furniture, CMF, human factors,
process and sustainability, with computed geometry figures and an interactive continuity lab.

## Live site

Served by GitHub Pages from this repo:

```
https://<username>.github.io/DesignAtlas/
```

## Structure

```
index.html                     the hub — lists disciplines and topics
topics/
  surface-and-substance.html   the first topic (a self-contained atlas)
CLAUDE.md                      how the atlas is built and maintained
README.md                      this file
```

Every page is a single self-contained HTML file. No build step, no dependencies to ship —
open any file in a browser and it works.

## How it grows

New topics are added **one at a time, by conversation**. A topic is proposed, researched,
its figures drawn (geometric ones computed for correctness), built as a new page in
`topics/`, and linked from the hub. The full recipe and conventions live in
[`CLAUDE.md`](./CLAUDE.md), which is written for [Claude Code](https://claude.com/claude-code)
to follow — open Claude Code in this repo and ask it to add a topic.

## Local preview

Just open `index.html` in a browser. For link-relative correctness you can optionally serve
the folder:

```
python3 -m http.server
# then visit http://localhost:8000
```

## Maintenance & QA

Contributors (human or Claude Code) should read `CLAUDE.md` before editing. It defines the
design tokens, the entry schema, the figure house-style and overflow budget, the image and
source policy, and the QA checks that must pass before every commit. The only dev dependency
is `jsdom`, used for the smoke tests:

```
npm i -D jsdom
```

## Credits

Photographs are hotlinked from Wikimedia Commons under their respective PD / CC-BY / CC-BY-SA
licences and credited on each figure. Source links point to standards bodies, regulators and
reference works. Everything else — text, diagrams, interactive figures — is original to the atlas.
