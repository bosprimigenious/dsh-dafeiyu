# Mac acceptance evidence · 2026-11-02

**PR #97 remains NOT READY.** The official-package #83 checks below passed;
PR no-client reason text has not been captured, its closed-session race has not
been reproduced, and its cross-display regression awaits an external monitor.

Environment: macOS 27.0.1 (26A434), Apple M5 / arm64, DSH Desktop 0.2.0-rc.2,
`bubbleScale=0.8`, character scale 0.85, `webOverlay=false`. Screenshots are
unmodified application-window captures, not composited desktop captures.
Use the raw [position/window log](native-boundary-log.jsonl) to establish
offscreen placement and screen/full-screen transitions; screenshots alone do
not establish those facts. [SHA-256 manifest](sha256.json).

## Official 0.1.23 / #83

Official package SHA-256:
`c1a3798a65952540220eadeff3291ee38036c99a877624b9c0281bb54117cdf9`.
This package has no task-row navigation.

All times below are UTC+8; coordinates are AppKit bottom-left pet anchors.

| Operation | Actual result |
| --- | --- |
| Nov 1, 17:29–17:32: partial-right placement, window/content changes | Anchor `(1464,470)` unchanged. |
| Nov 1, 17:41:33→35→37: built-in→SSN-24→built-in real drag | `(1514,430)`→`(724,1480)`→`(1514,430)`. External CG bounds `(-103,-1080,1920,1080)`, built-in `1710×1112`. |
| Nov 2, 00:40:21: drag to partial-right position | Anchor `(1510,506)`, Helper CG window `(1485,220,401,394)`, beyond built-in right edge. |
| 00:41:13→31→42: complete DSH quit/restart | Same `(1510,506)` before quit, after quit and after restart. Helper PID changed 95043→46712. The operator separately checked that DSH/Host/Helper exited; the raw log records windows, not all processes. |
| 00:42:21: create real C waiting state | Same anchor/window; [waiting card after restart](official-20261102-after-restart-waiting.png). |
| 00:42:31→52: enter/leave full screen | Same anchor/window; DSH `(215,82,1280,820)`→`(0,38,1710,1074)`→original. [DSH page](official-20261102-fullscreen-dsh.png), [native card](official-20261102-fullscreen-pet.png). |

Earlier unclear fullscreen/restart attempts remain in the raw log: Nov 1
anchors changed during those attempts, so they were not counted as passes.
The controlled Nov 2 sequence above is the restart/full-screen evidence.
The first CUA input method returned `AXError.notImplemented`; subsequent
real mouse operations used user-authorized CGEvent input.

## PR #97 local arm64 build

Source head `b927ec34a09b53a91a43e66152cccc684ecaf647`, implementation `1dae5c7`,
based on `0843238`. Package version is also 0.1.23; distinguish it by hash:
`175ffc9e3760fdc1b436ce84f9c14519d898bc775a4eb89af26b74d95ede6f7a`.
Installed Helper SHA-256:
`224d73542cbadbdbe1627c07fdf0148294198fa518e3aa7cded50014520dd6c2`.

| Operation | Actual result / screenshot |
| --- | --- |
| Create real A/B/C waiting sessions | [Three visible rows](pr-20261102-three-tasks.png). No synthetic task events. |
| Click visible rows 2, 3, 1 | Actual DSH destinations [B](pr-20261102-row2-B-dsh.png), [C](pr-20261102-row3-C-dsh.png), [A](pr-20261102-row1-A-dsh.png). |
| Drag from row 2 by 40pt while displaying A | Anchor X1174→1134; [page remains A](pr-20261102-drag-row2-stays-A.png). A subsequent normal row-2 click opened B. |
| Open/close context menu | [Native menu](pr-20261102-right-menu.png); later navigation still works. Not every menu command was exercised. |
| Enter full screen and click row 1 | [Actual A destination](pr-20261102-fullscreen-row1-A.png); exit restored DSH size, pet anchor unchanged. |
| Place transparent card/character gap over DSH question-collapse button and click | [Underlying A question collapsed](pr-20261102-transparent-click-through.png); compare expanded A above. This tests the transparent gap, not every transparent sprite pixel. |
| Finish and archive C | [C removed; A/B remain](pr-20261102-C-archived-two-rows.png). Click in the current third-row area did not reopen C or misnavigate. Archiving while C was waiting initially failed in DSH; finishing then archiving succeeded. |

No-client testing blocked only the real navigation SSE in DevTools. The error
reason was not captured: CUA capture waited about 5s, while feedback lasts 4s.
Those unsuccessful screenshots are not included as pass evidence. The added
rule was removed, blocking disabled, the stream restored to HTTP 200, and a
real A click successfully navigated again. A system screenshot-mode residue
temporarily intercepted input; Escape restored it. These are tool limitations,
not demonstrated plugin defects.

The normal C removal does not cover the `session-unavailable` race. PR
cross-display is **untested**: only the built-in display was connected in this
PR run. The official-package cross-display result cannot substitute for it.
Windows/Linux manual acceptance was not performed.
