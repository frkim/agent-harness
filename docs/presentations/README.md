# Presentations

Slide decks for this guide are written as [Marp](https://marp.app/) Markdown so
they diff in review and render on GitHub.

| Deck | Length | Source |
| --- | --- | --- |
| Designing an Agent Harness | 45 minutes | [`agent-harness-45min.md`](agent-harness-45min.md) |

The published HTML and PDF are built by the
[`publish-presentations`](../../.github/workflows/publish-presentations.yml)
workflow and served from GitHub Pages at
`https://frkim.github.io/agent-harness/`.

## Preview locally

Run the live preview server and open the printed URL:

```bash
npx --yes @marp-team/marp-cli@4.5.1 --server docs/presentations
```

## Build locally

Export the same artifacts the workflow publishes:

```bash
npx --yes @marp-team/marp-cli@4.5.1 docs/presentations/agent-harness-45min.md \
  --html --output tmp/site/index.html
npx --yes @marp-team/marp-cli@4.5.1 docs/presentations/agent-harness-45min.md \
  --pdf --pdf-notes --output tmp/site/agent-harness-45min.pdf
```

PDF export needs a local Chrome or Chromium installation. `tmp/` is
git-ignored, so the exported files stay out of the repository.

## Presenting

Speaker notes carry the timing plan for each part. Open the deck in presenter
mode with <kbd>P</kbd> after starting `--server`, or export the notes into the
PDF with `--pdf-notes` as shown above.

## Writing slides

- Keep one idea per slide and prefer lists or short tables over paragraphs.
- Use `---` to separate slides and `<!-- comments -->` for speaker notes.
- Use `<!-- _class: lead -->` for section title slides.
- Marp does not render Mermaid diagrams; keep diagrams in the chapters under
  [`docs/`](../) and use a plain-text sketch on the slide.
- Check the deck locally before opening a pull request: the workflow builds
  every deck on pull requests and fails the check if a deck does not render.
