# 360° strip — design options

**File:** [`hybrid.html`](hybrid.html) — open it directly; it has a state switcher and a dark-theme toggle.

## The question

The 360° strip sits under the PR header in `pr-details.html` and is visible from every tab. It has
to carry the whole lifecycle *and* nine review dimensions in roughly 90px of vertical space, on a
page where the reviewer's next action is the thing that matters. Six visual directions were drawn;
this folder records how they rated and what the hybrid takes from each.

## Ratings

| Option | Rating | Verdict |
|---|---|---|
| 1 — boxed current step, two-row chip cards | 7.5 | Strong current-step containment, but done steps carry *two* indicators (big check + numbered chip) and the two-row wrap reads noisy. |
| 2 — minimal inline strip | 8.0 | Cheapest vertically and calmest — a real virtue for an always-visible strip — but weak current-step emphasis and uniform pill tone, so nothing pops on scan. |
| 3 — two-tier track + "CURRENT STEP" badge | 7.0 | The badge is unambiguous, but the step is double-encoded (numbered track *and* icon row) and values are coloured decoratively (indigo numbers, purple coverage) without carrying meaning. |
| 4 — grouped pills: Blockers / Quality / Context | **8.5** | **Best information architecture** — grouping turns nine items into three questions. Held back by a semantic error (see below) and the tallest pill region. |
| 5 — card spine + spinner | 6.5 | Six bordered step-cards read as content rather than chrome, and the spinner implies something is *running* when the PR is waiting on humans — motion that misleads. |
| 6 — filled dots, ringed current, tinted icon chips | **8.5** | **Best single-tier execution** — confident filled-green done states, unmistakable ringed current step, calm pills with tone carried by the icon chip. Only gap: nine flat items, no grouping. |
| **7 — hybrid (this folder)** | **9.5** | Keeps every value, cuts scan cost to three questions, and states the merge rule instead of implying it. |

## The hybrid

- **Spine — option 6.** Solid green disc + white check for done, filled indigo number with a soft
  ring for current, muted outline ahead. Connectors: solid green behind, solid accent through the
  current step, dotted ahead.
- **Pill grouping — option 4.** Three labelled groups with hairline dividers: *Needs attention*,
  *Quality & compliance*, *Context & delivery*.
- **Pill styling — option 6.** A soft-tinted circular icon chip carries the tone; label quiet above,
  value in mono below. No decorative colour on values — only danger and attention tint text.
- **Copy — option 4's** "waiting for review completion" for the Gates sub-line, which is clearer than
  what shipped.

## The corrections the mockups needed

1. **"Blockers" is wrong for this product.** Option 4 files Security and Risk under a red BLOCKERS
   header. In Pulse, findings are *advisory pressure* — merge is gated by the attested verdict,
   approvals and the SonarQube gate. Calling a P1 finding a blocker misstates the model in the most
   prominent pixels on the page. Renamed **Needs attention**.
2. **A `BLOCKS MERGE` tag that means it.** Only a pill that genuinely gates carries the tag — visible
   on the quality-gate pill in the *Quality gate failed* state, never on a finding. This makes the
   rule legible rather than assumed.
3. **Dynamic membership.** A pill moves into *Needs attention* when its own tone turns bad or warn and
   returns home when it clears; if nothing needs attention the group disappears rather than sitting
   empty (see the *All settled* state).
4. **Coverage as an arrow.** `86.0% → 98.7%` — all six mockups converged on this independently, and it
   beats the shipped `86.0 B ↗98.7`. The grade band moves to the tooltip and aria-label.
5. **"3 rules · Enforce", not "3 rules enforced".** Enforce is the repository's mode, not a property of
   the rules.
6. **No spinner.** These states change on events, not on a timer; an animated current step both
   misleads and fights the page's zero-idle-repaint discipline.

## States to check

The switcher covers **Default**, **Quality gate failed** (the only state with a `BLOCKS MERGE` tag —
and note that coverage *and* the gate both promote into Needs attention, while only the gate is
tagged), **Verdict analyzing** (amber and still), **All settled** (attention group disappears
entirely) and **Merged** (Deliver becomes current — the half of the lifecycle a PR page usually
drops).

## The height cost — measured, not estimated

The strip is visible on every tab, so vertical space is a permanent tax. All three measured at a
1440px viewport with the component at its production width (~1312px):

| Variant | Height | Pill rows |
|---|---|---|
| Shipped strip (`pr-details.html` today) | **117px** | 2, ungrouped wrap |
| Hybrid — **Compact** density | **147px** | 2, zone-ordered with dividers |
| Hybrid — **Grouped** density | **180px** | 3, one labelled row per group |
| Grouped, all findings settled | 146px | 2 — attention row is gone |

Roughly 30px of the compact/shipped gap is the richer pills: a 22px tinted icon chip (which is what
carries tone) makes a pill 30px instead of 24px, and option 6's 26px spine discs cost the rest.
The remaining ~33px buys the group labels.

**Use the `Density` toggle in the lab to judge this directly** — that trade is the one real decision
left. My recommendation is **Grouped**: the labels are what convert nine values into three questions,
and the row-per-group layout means membership never depends on where text happens to wrap. Compact
keeps the same pills, order and dividers but drops the labels that say *why* a group exists — worth
taking only if 63px on every tab proves too expensive in situ.

One layout note if Grouped is adopted: the groups are laid out as **one row each with the label
inline on the left**, not as option 4's side-by-side blocks. Blocks re-wrap unpredictably as pill
membership changes (a group can jump rows mid-review) and measured 211px — taller than the version
here for strictly less stability.

## Adopted — shipped 2026-08-08

The **Grouped** variant is now live in `pr-details.html`. The port replaced the `.p360*` CSS cluster
and `render360()`'s markup strings; the state reads (`covPct()`, `secOpenAll()`, `SONAR`,
`S.approvals`, `aiOpen()`) were unchanged, as were all nine `data-p360` keys and the click routing.
The icon map went into the page's top state block, not next to `render360()` — that file's render
graph runs mid-script during composer mounting, so a late `const` is a temporal-dead-zone crash.

Two things this lab caught that the product port then inherited:

- **Side-by-side group blocks (option 4's literal layout) measured 211px** and re-wrapped
  unpredictably as pill membership changed. One row per group with an inline label is both shorter
  and stable — that is why the shipped version looks the way it does.
- **`min-width:0` on a spine step lets it shrink below its own label**, so at 375px the step text
  overlapped the next step instead of the spine scrolling. Both this file and the product use
  `min-width:auto`. Worth remembering for any future flex spine.

The **Compact** density (147px) remains in this file as the fallback if 180px on every tab proves too
expensive in real use — switching is a CSS-only change plus dropping the group labels.
