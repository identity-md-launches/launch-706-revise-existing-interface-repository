# COMP terminal design

## Overview

This revision extends the existing Vite, React and TypeScript terminal for borrowers, keepers and protocol operators. The seven action panes remain: position, redemption, work, oracle, keeper, backing and governance. The loan book is a full-width first pane. A compact monospace hierarchy, tabular figures, square controls and hairline divisions preserve the terminal’s character. Light mode uses the requested treasury-paper palette; dark mode preserves the original neutral surfaces.

The source of truth is `web/src/style.css`, with chart components in `web/src/Charts.tsx`, existing controls in `web/src/actions.tsx`, and theme behavior in `web/src/theme.tsx`. No UI framework, font service or chart dependency was added.

## Colors

All paint resolves through the same roles in both theme blocks. SVG uses these tokens or `currentColor`; `none` disables SVG paint. The light theme contains exactly seven distinct hex values.

| Role | Light | Dark | Use |
| --- | --- | --- | --- |
| `--bg` | `#F7F5EF` | `#111111` | Page, fields, inverse text |
| `--surface` | `#FFFDF8` | `#161616` | Panes, chart labels, dialogs |
| `--raised` | `#F7F5EF` | `#202020` | Hover, notices, chart tracks |
| `--text` | `#16202E` | `#EEEEEE` | Main text, neutral marks, primary actions |
| `--muted` | `#5A6472` | `#A8A8A8` | Secondary text, captions |
| `--rule` | `#D8D3C7` | `#3A3A3A` | Structural hairlines, redeemable band |
| `--control` | `#5A6472` | `#777777` | Control borders, chart limits |
| `--focus` | `#16202E` | `#EEEEEE` | Two-pixel keyboard outline |
| `--healthy` | `#2F5D50` | `#92B5A6` | Positions outside minCR, available backing/headroom |
| `--danger` | `#8C2F2F` | `#E19A9A` | Positions below minCR, breached limits |
| `--clear` | `transparent` | `transparent` | Unfilled controls |

Dark status colors increase lightness within the same green/red meanings for legibility on the preserved dark surfaces. Text, position coordinates and named zones supplement color. No gradients or glows are used. Forced-colors mode maps focus, control and status roles to system colors for both themes.

The OS preference is resolved before first paint. A visible light/dark toggle persists `comp-terminal-theme` in local storage; without an explicit choice, OS changes remain live. Storage failures leave the control usable for the current visit. The theme-color meta tag and favicon use the resolved tokens. Theme changes suppress paint transitions.

Measured text/surface contrast is 16.14:1 light and 15.60:1 dark; muted text/surface is 5.90:1 and 7.61:1. Green marks against the redeemable band measure 5.02:1 and 5.08:1. These are the tested rendered pairs, not a claim about every possible browser state; details are in `docs/frontend/VALIDATION.md`.

## Typography

The existing system stack is `SFMono-Regular`, Consolas, `Liberation Mono`, monospace. No font files are shipped. Actual available system fonts determine the face and weight substitution. Root text is 13px at 1.5 line height, with tabular numerals and root font smoothing. The 18px/600 page title, 13px/600 pane headings and 27px financial headline figures preserve the dense operator interface. Body descriptions use 11–12px; chart captions, labels and metadata use 10–11px. Small text remains regular or medium weight. Mobile headline figures become 32px and form inputs/selects 16px, avoiding small-input zoom.

Addresses and messages wrap; numeric rows align to the trailing edge. Full chart addresses are available in hover titles, accessible button names, selected-mark text and the position ledger. Keeper actions display the inspected full address. Financial text remains selectable. `Ticker` animates displayed numbers while exposing the final value once to assistive technology; its hidden text is positioned inside its own wrapper.

## Layout

`terminal` occupies `100dvh`; document and root overflow are disabled. Only pane bodies, tables, long notices and transaction status overflow locally. The page retains its header, status strip and footer. Desktop horizontal gutters are 20px, reducing to 12px below 1100px; pane content is inset 14px on desktop and 16px on phone.

Above 1100px, the grid has four equal columns. The loan book spans all four, followed by the original arrangement: position/keeper, two-row redemption, work/backing and oracle/governance. The loan row has a 245px minimum, with two flexible action rows. From 761–1100px, the original three-column arrangement remains below the loan book. At 760px and below, a native pane selector exposes one of all eight panes at a time, defaulting to loans. The header wraps. Scroll arrows in pane headers signal additional content.

Loan labels use measured plot width and separate vertical lanes to prevent collisions while preserving each exact ratio coordinate. A larger book adds pane-local vertical scrolling. The axis starts at zero and expands in 100-percentage-point steps when required; its upper bound does not shrink during a visit, so changing minCR does not continually rescale the strip. Saturated uint256 ratios use an overflow arrow and retain their exact value in the ledger.

Both themes were measured without page overflow at 1440×900, 1280×800, 900×900, 390×844 and 320×740. All mobile panes were reached, and position-label rectangles were checked for overlap. Physical devices, native browser zoom and localized/RTL layouts were not tested.

## Elevation & Depth

The system is flat. One-pixel rules communicate pane and field boundaries; surface changes distinguish tracks and notices. Dialogs use the existing native modal, a control-colored border, and an 85%-opaque page-token backdrop. There are no shadows. Marks sit above bands; visible label backplates use `--surface`.

## Shapes

Controls and panels remain square. Position marks are circles whose areas, not diameters, scale with accrued debt: `radius = 12 × sqrt(debt / largestDebt)`. Their separate 128×44px buttons keep small debts keyboard/touch accessible without inflating the represented debt area. Oracle primary price is a diamond and spot is a circle. Solid/dashed limit lines and the double-rule net-work segment remain distinguishable without hue.

## Components

- `Pane`, `Row`, `Action`, `ActionForm`, `AddressLink` in `web/src/actions.tsx` retain the existing scroll regions, transaction gating, validation, review and copy/explorer behavior. Primary emphasis stays on the wallet/review action. Native disabled states explain missing prerequisites nearby.
- `LoanBook` in `web/src/Charts.tsx` combines live minCR/redemption-ceiling bands, deterministic labels, proportional marks, selectable address details and a full ledger. Loading, coverage errors and a verified all-zero-debt result have distinct copy. Failed or incomplete discovery never renders zero open positions.
- `Sparkline` pairs `AttestationAccepted` with its preceding `ValueUpdated` from the same transaction. Each accepted update is a point on a linear signed-timestamp axis; gaps are not smoothed. The final accepted update’s expiry is marked using the live maxAge. The visible time domain ends at now; a right-edge arrow and exact UTC expiry label mark expiry beyond that domain without compressing historical gaps. UTC endpoints, units in the surrounding feed row and a values disclosure provide a textual alternative. Derived USD has no synthetic attestation history; a deployed attested work oracle receives its own sparkline.
- `SupplyChart` splits circulating supply into outstanding collateral principal and net work after non-principal burns. The accounting disclosure explains the identity and any bad debt. A separate backing-ratio scale overlays the bar with a solid backing marker and dashed par marker. Backing values secured collateral at the read USD price, caps it at `(principal − bad debt) × minCR`, then adds reserve value. Zero supply and stale USD are explicitly labeled.
- `DivergenceChart` derives limits from `maxDivergenceBps()` and locates primary/spot marks relative to primary. It labels remaining headroom, stale inputs and breaches.
- `WorkChart` compares cumulative `totalWorkMinted` to `workCeiling()`, including a labeled over-ceiling state. Burns do not restore cumulative issuance headroom.
- `ThemeToggle` uses native buttons with the destination theme as the accessible name. `Ticker` and chart geometry reuse `--fast: 150ms` and `--ease: cubic-bezier(0.2, 0, 0, 1)`. Loan bands and positions transition to new geometry; the sparkline draws on load and new points enter individually. Reduced motion turns these effects off, including numeric interpolation.

## Do’s and Don’ts

Reuse `Pane`, existing action components and paired semantic tokens when extending this terminal. Add a mobile selector option for a new pane and retain local overflow. Preserve the exact light palette and original dark surface tokens. Keep raw transaction addresses visible alongside readability labels.

Read thresholds and figures from the same contract snapshot; handle historical read failure as unknown coverage. Do not replace missing logs with zero values, plot reporter-only updates as accepted attestations, smooth irregular timestamps, or use cumulative work minted as the circulating work slice. Fixed vocabulary labels are deterministic and may collide; they do not change the public nature of addresses. Keep transaction prerequisites, simulation and review independent of chart loading.
