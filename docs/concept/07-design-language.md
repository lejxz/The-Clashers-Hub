# 07 — Design Language

The shared visual system. Its job: make 10+ screens built by 4 people look
like one product, and make dense data feel calm. Decisions here are defaults,
not suggestions — a screen that deviates files a PR that changes *this* doc
too.

## 1. Feel

**A dark scoreboard.** Clash of Clans is a game of gold and trophies, but a
data tool should be quiet: a near-black canvas, panels that float on it, one
gold accent used sparingly for the things that matter (numbers, verdicts,
focus). Dense but never noisy — micro-labels whisper, values speak.

## 2. Tokens (the only allowed values)

| Token | Value | Used for |
|---|---|---|
| `color/bg` | `#0C1017` | app background |
| `color/panel` | `#151B26` | cards, tables, sheets |
| `color/line` | `#232B3A` | hairline borders (1px) |
| `color/text` | `#E7ECF4` | primary text |
| `color/muted` | `#8B94A7` | secondary text, micro-labels |
| `color/accent` | `#E4B34A` | gold — key values, focus ring, links |
| `color/good` | `#3FB984` | positive deltas, "at target", win chips |
| `color/warn` | `#E0653A` | flags, negative deltas, loss chips |
| `color/info` | `#5AA7E8` | neutral highlights, tie chips, war state |
| spacing scale | `4 8 12 16 24 32` | the only gaps |
| radius | `10` cards / `999` chips & pills | |
| font | Inter (system fallback) + a mono for numerics | `font-variant: tabular-nums` everywhere numbers align |

Type scale: display 28/700 · title 18/600 · body 14/400 · micro-label 11 mono
UPPERCASE +tracking (used for column headers, chip labels, stat captions).

All tokens live in one `tokens.ts` exported object; no literal colors in
components (CI greps for hex literals in `src/components`).

## 3. Layout

- Centered content column, max width 1280, side padding from the spacing scale.
- **Roster table ↔ cards:** ≥ 768px a real sortable table; below, one card per
  member with the same columns stacked. Same data, two presentations.
- **Player detail:** ≥ 1024px two columns (identity + analysis); stacked below.
- Breakpoints come from `useWindowDimensions` into a shared `breakpoints`
  helper — no CSS media queries scattered through components; RN paradigm is
  JS-driven layout.
- Every list that can exceed ~30 rows is virtualized (FlatList), because
  50-member rosters and paginated search results are the *normal* case.

## 4. Component inventory (shared kit)

| Component | Purpose |
|---|---|
| `AppShell` | top bar (logo, tag-lookup shortcut) + content + footer |
| `SearchBar` + `FilterRow` | home screen query controls |
| `TagInput` | the ONE owner of tag normalization (03 §2) |
| `ClanCard` | search result row: badge, level, members, location |
| `StatChip` | label + value pair used on every header |
| `RatioFlag` | donation-balance chip (good/warn/dash states) |
| `RosterTable` / `RosterCard` | the clan roster, table & card presentations |
| `ProgressRail` | achievement value/target bar with star markers |
| `CategoryBars` | rushed per-category horizontal bars |
| `Gauge` | overall rushed % dial (the flagship number) |
| `WarResultChip` | win/loss/tie pill |
| `CompareRow` | metric row with both values + delta arrow |
| `Legend` | the ONE chart legend pattern — all charts use it, no exceptions |
| `Skeleton`, `EmptyState`, `ErrorState` | the state matrix (06 §3), shared |

## 5. Charts

- Built on **react-native-svg**: bar (donation balance), line (war trend),
  dual-line (compare trend), radar (P2 compare garnish). Four types, maximum.
- One `Legend` component for every chart — swatch + label + optional footnote,
  laid out in a row. A chart adding its own legend is a PR that gets sent back.
- Numerics: tabular figures, no decimals unless the metric is inherently
  fractional (stars per attack → 1 decimal max).
- No chart junk: no gradients, no 3D, no animations on data changes — data
  tools answer questions, they don't perform.

## 6. Accessibility

- Full keyboard path: search → results → roster rows (as links) → player
  detail, all reachable and visible via a 2px gold focus ring.
- RN accessibility props map to ARIA on web: `accessibilityRole` on every
  interactive element; the roster table uses row/cell/grid roles; charts carry
  text alternatives (the legend already lists the series — numbers too).
- Contrast ≥ 4.5:1 checked against the token pairs during M0 review; muted
  text on panels is the pair most likely to fail, so it's verified first.
- Touch targets ≥ 44px on the card presentation (phone web is a real target).

## 7. Imagery

Clan badges and league icons come from the game API's own `iconUrls` (public
CDN). They are rendered with RN `Image` (an `<img>` on web) with fixed layout
slots (badge 48, league icon 24) so slow CDN loads never reflow the layout.
No game art is bundled with the app — nothing to license, nothing to resize.
