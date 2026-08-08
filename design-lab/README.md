# Design lab

Explorations that are **not wired into the product**. Each subfolder is one design question,
holding the option(s) under consideration so they can be opened, compared and argued about
before anything lands in the prototype pages.

## Convention

```
design-lab/
  <topic>/            one folder per design question
    hybrid.html       the proposal (self-contained, opens with no build step)
    NOTES.md          what was compared, how it rated, what was decided
```

- **Self-contained.** Every file opens directly in a browser — inline CSS/JS, no imports,
  no fonts, no build. Same rule as the prototype pages.
- **Tokens are copied, never forked.** Colour/type/spacing variables are pasted verbatim from
  the page the design targets, so a lab file renders pixel-identical to the product. If the
  app's tokens change, re-copy them.
- **Live state, not static mockups.** Where a component reacts to state, the lab file renders
  it from a small state object with a switcher, so every variant is inspectable rather than
  described.
- **Nothing here is loaded by the product.** Promoting a design means porting it into the
  target page; the lab file then stays as the record of why.

## Contents

| Topic | Question | Status |
|---|---|---|
| [`360-strip/`](360-strip/) | How should the PR's 360° overview strip look? | **Adopted** — option 7 (Grouped) shipped in `pr-details.html`, 2026-08-08 |
