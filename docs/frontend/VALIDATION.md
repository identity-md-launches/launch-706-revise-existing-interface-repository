# Terminal themes and charts — worker validation

Date: 2026-10-04. This is a worker report, not independent certification. The task revises the existing terminal; all seven original panes and contract controls remain. The static production export is `dist/`, served during tests from `/preview/` with relative assets. The source and existing manifest/lockfile are in `web/`. No backend, indexer, subgraph, swap or trading pool was added.

## Verification performed

| Check | Command / evidence | Result |
| --- | --- | --- |
| Locked dependency install | `npm ci --cache /tmp/comp-npm-cache` inside `web/` | Passed; manifest and lockfile unchanged; no dependency added |
| Typecheck | `npm run typecheck --prefix web` (also run by build) | Passed |
| Production build | `npm run build --prefix web` | Passed; relative Vite base and original build configuration retained |
| Manifest and ABI integrity | `npm run check:export --prefix web` | Passed; four canonical pinned ABI hashes and all 16 asset hashes verified |
| Amount, history and theme tests | `node --test web/tests/math.test.mjs web/tests/history.test.mjs web/tests/theme.test.mjs` | 12 passed |
| Production browser interactions | `PLAYWRIGHT_BROWSERS_PATH=/tmp/floppy-pepe-playwright-browsers npm run test:browser --prefix web` | 34 checks passed in Chromium 153.0.8010.12; no mocked-flow console errors or uncaught exceptions |
| Live public chain/code | `node web/scripts/live-check.mjs` | All three configured RPCs returned Sepolia chain ID 11155111 and nonempty code for all four handoff contracts |
| Live browser reads | `PLAYWRIGHT_BROWSERS_PATH=/tmp/floppy-pepe-playwright-browsers node web/scripts/live-charts.mjs` | Read-only production export; evidence in `live-charts.json` and `live-charts.png`; no wallet or broadcasts |
| Complete candidate package | `node web/scripts/check-package.mjs` | Full snapshot Git bundle measured in a temporary bare repository; below 8 MiB; conservative inclusive bound and export sizes in `packaging.json` |

Vite emits its existing-size advisory because the principal JS chunk is approximately 573 kB before gzip (approximately 175 kB compressed). The export remains under 1 MiB. No existing dependency or build setting was changed to suppress the warning.

`browser-results.json` contains the individual test names, dimensions and measured contrasts. The browser script owns and closes its local HTTP server and browser. Unit-history tests resolve TypeScript module paths in `test/scratch/`; scratch is not part of the delivery.

## Interaction and history coverage

The preserved action suite covers missing/disconnected wallets, rejection recovery, wrong chain, the exact unknown-chain add/switch sequence, disabled prerequisites, exact allowance approval, rejected signing, receipt/refetch locking, deposit/borrow/repay/withdraw controls, work issuance, keeper inspection/mark/clear/liquidation, backing sync, governance proposal/apply and reporter permission/report. Redemption coverage includes reserve, position and mixed routes, invalidated quotes, nonzero minimum output, derived eligibility ceiling, translated simulation errors, account/chain gates, keyboard review/cancel/focus return and a mocked successful receipt. No real transaction was sent.

New checks establish:

- Contiguous inclusive log chunks of at most 2,000 blocks; a silent RPC `[]` triggers paginated Blockscout fallback. Empty fallback, missing pagination metadata, cursor loops and address mismatches fail closed.
- Discovery from actual deployment height, duplicate depositor elimination, same-block owner reads, zero-debt exclusion and whole-book rejection on any failed owner. A failed book removes marks and reports unknown coverage; retry restores successful reads.
- Pairing each accepted attestation with its preceding value log in the same transaction. Reporter-only values are excluded; missing paired values fail. Four deliberately irregular fixture timestamps retain their unequal horizontal gaps; a fifth point appends.
- Deterministic address labels, a 4:1 debt/area example, full hover addresses and keyboard-selectable details. Narrow-width label rectangles do not intersect.
- Live minCR changes move bands and reclassify marks, and changing prices move existing marks. The axis preserves its upper bound during a visit rather than shrinking whenever health changes.
- Supply segments, backing/par markers, work headroom, live work-ceiling changes, live divergence-limit changes and explicit breach labels.
- OS default and live OS changes, explicit persistence across reloads, matching theme-color, complete paired color-token definitions, exact seven-hex light palette and unchanged six dark base values.
- Reduced-motion removal of chart transitions, sparkline drawing and point entrances. Numeric interpolation reads the same motion preference and preserves exact final text.

## Better Interface consolidated review

The supplied Better Interface workflow, six core domains, documentation method, and Ethereum frontend UX reference were read and applied during implementation. The pinned files are assignment inputs and are not included in the deliverable. Attribution and license texts remain in `GUIDE-LICENSES.txt`. Root `DESIGN.md` describes the final source, replacing the old design document with a pointer.

| Domain | Coverage | Evidence / limitations |
| --- | --- | --- |
| Accessibility | Checked | Native toggle/mark buttons, named figures, complete value disclosures, ledger headers, address details, visible focus, action gating, native modal keyboard flow, reduced motion. Axe reported no tested A/AA violations on both desktop themes and the existing 320px redemption flow. No screen-reader session, physical touch device or exhaustive forced-colors session. |
| Layout | Checked | Both themes at 1440×900, 1280×800, 900×900, 390×844 and 320×740: document width/height equal viewport dimensions, eight phone panes reachable, labels do not overlap. Representative screenshots were opened and inspected. Native 200% browser zoom, RTL and pseudo-localization not performed. |
| Writing | Checked | Read failures name unknown coverage and a recovery action. Net-work accounting and stale USD are explained; no fabricated empty book or reporter attestation. Names are described as deterministic public address labels. Existing transaction review language preserved. |
| Typography | Checked | Existing monospace stack and density retained; tabular digits, stable animated number wrappers, wrapping addresses and 16px mobile fields reviewed in source and screenshots. System font availability varies; exact installed font faces were not certified. |
| Colors | Checked | Both complete theme blocks, exact light palette and original dark base tested. Actual text/surface and green mark/band pairs measured in browser; no literal component color, gradient or undefined custom property in the CSS source checks. Not every disabled/forced-color/native-widget combination was manually measured. |
| UI | Checked | Loading, ready, unavailable, retry, disconnected, wallet error, breach, hover/title, selected and focus states exercised; theme changes suppress paint transitions. Existing 150ms cubic-bezier curve retained. Real-time geometry and append behavior tested; 10%-speed DevTools animation inspection not performed. |

### Findings, fixes and rechecks

| Severity | Source location | Evidence, impact, correction and recheck |
| --- | --- | --- |
| High | `web/src/history.ts:169` | Public providers can silently return empty logs. Implemented contiguous chunk reads, whole-range Blockscout fallback and conservative empty/partial failure. Tests verify that unknown discovery never appears as zero positions. |
| High | `web/src/history.ts:259` | Partial owner reads would conceal positions. The entire book now fails on one owner read failure, deduplicates deposits and removes debt-free positions only after successful reads. Unit and browser failure/recovery checks pass. |
| Medium | `web/src/Charts.tsx:89` | An axis recomputed from every ceiling change could cancel the visible band movement. Preserved a growing axis upper bound and rechecked a 150→200 minCR update, classification and price movement. |
| Medium | `web/src/Charts.tsx:90` | Initial visual phone review showed overlapping labels: ResizeObserver had run before the plot existed. Observe when loaded data mounts the plot; both-theme viewport tests now assert nonintersecting label rectangles. |
| Medium | `web/src/style.css:779` | Initial browser check measured document height 1303px at a 900px viewport because offscreen accessible number text escaped pane positioning. Added the positioned `Ticker` wrapper; all five viewport dimensions now pass. |
| Medium | `web/src/history.ts:302` | Accepted events carry no numeric value themselves. Paired preceding `ValueUpdated` events by transaction/log order, preserving signed timestamp spacing and excluding reporter-only updates. Tests verify gaps and append behavior. |
| Medium | `web/src/Charts.tsx:510` | Cumulative work does not equal the circulating work slice after non-principal burns. Added the accounting identity read, net-work split, reconciliation failure, bad-debt explanation and guarded backing calculation. Exported chart and read tests pass. |
| Medium | `web/src/style.css:1`, `web/src/theme.tsx:15` | The original global dark palette and literal theme-color could leak across themes. Added paired roles, early OS resolution, explicit persistence and token-derived meta/favicon. Both-theme screenshots, color source tests and persistence tests pass. |
| Medium | `web/src/Charts.tsx:332` | Visual review of the phone cadence chart showed future expiry compressing recent updates into a narrow cluster. The plot now ends at now and marks off-axis future expiry with an arrow and UTC label. Irregular-spacing and append checks pass; refreshed cadence screenshots inspected. |
| Low | `web/src/Charts.tsx:18` | A brief pre-effect chart state could say thresholds failed before history started. Initial history state now starts loading; the live browser waited for and recorded the final coverage result. |

No unresolved applicable high/medium finding was observed in the reviewed states. This does not establish untested assistive-technology or live transaction behavior.

### Measured rendered contrast

| Foreground / background | Light | Dark |
| --- | ---: | ---: |
| Main text / pane surface | 16.14:1 | 15.60:1 |
| Muted text / pane surface | 5.90:1 | 7.61:1 |
| Green position / redeemable band | 5.02:1 | 5.08:1 |
| Green position / pane surface | 7.37:1 | 8.09:1 |

Text thresholds were 4.5:1 and mark thresholds 3:1. Exact RGB pairs and unrounded results are in `browser-results.json`. Separate text/shapes/zone placement carry the statuses as well as hue.

## Visual evidence

The final mocked export screenshots are `charts-light-1440.png`, `charts-light-1280.png`, `charts-light-390.png` and the corresponding `charts-dark-*` images. They show fixture positions, not claimed live balances. `cadence-light-390.png` and `cadence-dark-390.png` show the oracle charts after their load animation. `terminal-1440.png`, `terminal-1280.png`, `terminal-900.png`, `terminal-390.png`, `terminal-320.png` and `keyboard-focus.png` record the preserved interaction suite. Representative desktop, phone and focus screenshots were viewed; dimensions and pane reachability were also checked programmatically.

The new live browser session read fresh primary/NHI/spot/USD values and zero COMP supply. Archive state probes encountered pruned-state RPC errors, so deployment discovery used the public Blockscout creation metadata. Short-range logs were then read from RPC. Primary had an accepted attestation; NHI and spot had reporter updates only. The vault returned its constructor event but no deposit events: the loan pane reported **could not read / position count unknown**, which is the required conservative result. The timestamp and block in `live-charts.json` identify this observation. No recorded position or balance was invented to fill the chart. Older `live-state.json` remains dated evidence from the previous revision and is not the basis for this report.

## Configuration, packaging and limitations

No new deployment or network handoff was supplied under `.imd/reads/` in this revision. The existing `web/deployment-source.json` remains the manifest build input; the application loads only the generated `dist/imd-deployment.json` at runtime and verifies the ABI bindings. Contract addresses, ABI loading, chain IDs, RPC endpoints and history policy are centralized through `web/src/config.ts` and that runtime manifest. No private credential, guessed contract address, extra manifest key or Uniswap flow was introduced. The pre-existing add-chain fallback remains tested against its supplied configuration.

The production inventory contains every exported file other than the manifest itself, with SHA-256 hashes; four contract ABI Keccak hashes remain pinned to the deployed source. Dependency/cache directories remain excluded by the existing ignore file. No ignore file was changed. Temporary debug screenshots/reports were removed; source, existing lockfile, required ABIs and all runtime assets remain complete. Packaging measurement includes the complete candidate tree and the packaging report; no Git bundle or dependency archive is left in the repository.

Live funded actions, long production loan books, extended provider outages, deep reorganizations, native mobile wallets, physical-device touch, screen readers, native zoom, localized/RTL layout, absolute social metadata, pinning and publication are not verified. Historical reads depend on public RPC/Blockscout availability and are conservative about empty results. The cached 12-block overlap is not a guarantee against deeper reorganizations. Asset and source checks do not independently certify contract safety.

Implementation and local validation are complete for the stated frontend scope. **No repository commit was created:** the task explicitly forbids writes to `.git/`. The source, existing lockfile, static export and documentation are present for submission. Bundle-size checks use only a disposable temporary bare repository and leave the working repository’s Git metadata untouched.
