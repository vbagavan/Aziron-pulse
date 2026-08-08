# PR sidebar — design options

**File:** [`hybrid.html`](hybrid.html) — open it directly; it has state and density switchers and a dark-theme toggle.

## The question

The right rail on `pr-details.html` carries change readiness, reviewers, assignees, labels, change
and risk, linked findings, enforcement, post-merge delivery, and the merge control. Six visual
directions were drawn. This folder records how they rated and what the hybrid takes from each.

Two facts decided most of it, both measured on the live page at a 900px viewport rather than assumed:

- **The rail is 300px wide.** The mockups were drawn at roughly 660px. Any design whose core idea
  needs horizontal room — side-by-side cards, multi-column blocks — does not survive the real width.
- **The rail is 2,032px tall — 2.3 screens of scrolling.** Four sections are 76% of it:
  Merge 495 · Change 466 · Delivery 287 · Readiness 289. Length is the problem to solve first.

## Ratings

| Option | Rating | Verdict |
|---|---|---|
| 1 — donut + "Ready with blockers" | 7.0 | Strong headline, explicit blocker list, chevrons that read as navigable. But the merge button is red — that reads *destructive* when it is merely blocked; uppercase headers are shouty; and it does nothing about length. |
| 2 — tabs + sticky footer | **9.0** | **The only option that attacks the length problem.** Sticky action footer keeps the next step permanently in reach; "Why Medium?" is the best risk explainer in the set; collapsible sections; merging People & labels is a smart economy. The tabs are the weak part. |
| 3 — two-column cards | 6.0 | Its core idea — pairing Reviewers│Assignees, Labels│Change — **cannot fit 300px** (each column would get ~145px). Two typos ship in the mock ("Luthor familiarity", "readiness tus ×"). Promoting Risk to its own card is the keeper. |
| 4 — tabs, chevrons everywhere | 8.0 | Cleanest hierarchy, and the best cause→effect in the set: "Merge blocked: approvals incomplete" sits directly above the button. Undone by a **blue, enabled-looking** merge button while blocked — it invites a click that cannot succeed. |
| 5 — per-row icons | **8.5** | Icons make the eight-row Change block genuinely scannable — the best fix for the densest region, and it costs no width. Most humane tone ("Good progress — address blockers"). Clearest statement of the approval requirement. But it is the longest option: nothing collapses. |
| 6 — count chips, disabled button | 8.0 | **Only option with correct merge semantics** — properly greyed, `disabled` button. Count chips and reviewer timestamps are excellent. But its two signature ideas, the 2-column Change card and the 4-up readiness strip, both need width it lacks at 300px. |
| **7 — this hybrid** | **9.5** | Fits 300px, cuts the rail to 1,203px (−41%), and states the merge rule honestly in every state. |

## The hybrid

- **Structure — option 2.** Collapsible sections and a sticky action footer.
- **Merge semantics — option 6.** The button is `disabled` and grey when merge is blocked, and a real
  accent primary only when every gate is green. Red reads *destructive*; blue reads *clickable*.
  Neither is true of a blocked merge.
- **Density — option 5.** Row icons on Change & risk, the densest block.
- **Causality — option 4.** "Merge blocked: <reason>" immediately above the control.
- **Risk — option 2 + option 3's keeper.** A labelled sub-block inside Change (stacked, not a
  side-by-side card) with the "Why Medium?" explainer inline.

### Two deliberate departures

**No sidebar tabs.** Options 2 and 4 put Summary / Findings / Merge tabs inside the rail. A second tab
bar in 300px, on a page that already has five main tabs, is an IA smell — and a Findings tab would
duplicate the Files-changed workspace. The collapse pattern the 360° strip already uses does the same
job with no new navigation model.

**Strip-aware defaults.** The 360° strip above already answers Change, Risk, Enforcement (governance)
and Delivery with pills. Those sections therefore start collapsed, carrying their value on the header
row instead. Nothing is removed — one click opens any of them, and the choice persists across state
changes. This is where most of the 41% comes from; it is an *information-architecture* saving, not a
styling one.

## Measured

| Variant | Height | vs shipped |
|---|---|---|
| Shipped rail today | 2,032px | — |
| Hybrid, strip-aware defaults | **1,203px** | **−41%** |
| Hybrid, everything expanded | 1,826px | −10% |

Even fully expanded the hybrid is shorter than what ships today, so the saving is not purely from
hiding things.

## States to check

**Blocked** (today's state) — disabled merge button, inline reason, footer names the blocker.
Note the reason reads "1 approval outstanding", *not* the 3 findings: findings are advisory pressure
in this product, approvals actually gate. **Gate failed** — the SonarQube gate joins the blockers and
is named first as the harder one. **Ready to merge** — the only state where a live primary button is
honest; the footer flips to the merge control. **Merged** — the rail stops asking for anything and the
footer reports delivery.

## Adopted — shipped 2026-08-09

Live in `pr-details.html`. Real-world height came in at **1,434px (−29%)**, not the lab's
−41%: the product's Merge card is 566px against the lab's 388, because it also carries the
update-branch row, the merge-method menu, auto-merge and its notes. The lab under-modelled
that one card; everything else landed as measured.

Three things the port surfaced that the lab could not:

- **`mountMetricDefs` appends the "?" button into `.lbl`** — which would have nested a
  button inside the collapsible header button (invalid HTML). The fix is a zero-font-size
  `.lbl` anchor placed *outside* the toggle.
- **An existing `.sb-h h2{text-transform:uppercase}`** rule won on equal specificity and
  made every card title shout — the exact trait this review criticised in option 1.
- **A count chip that lied.** `#revCnt` shipped as static markup reading "1 of 2" while the
  note beside it said "2 of 2". Any chip added to a header has to be wired to the same
  state its section renders from; there is now a `sbCounts()` for exactly that.

## If this is adopted

Port into `pr-details.html` by wrapping each `.sb-sec` in the collapsible header pattern and replacing
the merge card's button block. The state reads (`gatesOk()`, `S.approvals`, `aiOpen()`, `SONAR.status`)
do not change — only the markup they produce. Two things to carry over exactly:

- The merge button must be a real `disabled` button, not a styled-disabled one, so it is
  keyboard- and screen-reader-correct.
- Collapsed-section open-state should persist the way the 360° strip's toggle already does, so the
  rail remembers how a given reviewer likes to work.
