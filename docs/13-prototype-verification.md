# Planned/observed prototype verification

Status: 20 in-memory scenario functions executed and their outputs inspected against the adopted rules. Browser verification is blocked, not passed. This is a design investigation, not application acceptance or synchronization validation.

## Finding and correction

The previous module counted 45 estimated morning-care minutes at 08:00 for an entry at 07:45 and a planned end at 08:30. This contradicted the newly confirmed in-progress rule.

The temporary module now exposes an expected duration while the interval is in progress and excludes it from completed totals. At 08:30, the 45 minutes enter the estimated total. A declared exit at 08:10 contributes 25 declared minutes immediately and remains 25 at 08:30. Annulling that exit restores the applicable in-progress or completed pattern-derived reading.

Additional fixture helpers explore date moves, slot removal, calendar precedence and preserved planning revisions. Concurrent-fact traversal now sorts input by identity so the inspected output does not depend on the order in which the same facts arrive. Calendar coverage that is not known open blocks automatic meal presumption while retaining explicit declarations with a visible warning in the scenario output.

## Executed scenarios and observed outcomes

| # | Scenario | Observed outcome |
| --- | --- | --- |
| 1 | Morning clock at 08:00, 08:29, 08:30 | 45 expected minutes and zero completed minutes before the boundary; 45 estimated minutes at the boundary |
| 2 | Early morning exit | 25 declared minutes at 08:10 and 08:30 |
| 3 | Annul early exit | In progress at 08:10; 45 estimated minutes at 08:30; history retained |
| 4 | No-action morning and afternoon | No care duration, no incomplete-data warning |
| 5 | Afternoon exit and correction | 15 estimated minutes becomes 20; original declaration retained |
| 6 | Duplicate morning entry | Duration excluded until explicit selection and annulment; then 45 estimated minutes |
| 7 | Lunch boundary, absence, annulment | Zero meals at 12:59; one presumed at 13:00; zero after absence; one presumed after annulment |
| 8 | Move one recurring session | Same identity and responsible adult; another Wednesday and the original input remain unchanged |
| 9 | Move onto another session | Separate identities with overlap flagged |
| 10 | Move a session with attendance | Move blocked; original date preserved |
| 11 | Move after mistaken-attendance annulment | Same session moves; original declaration metadata retained |
| 12 | Planning revision and existing exception | Past uses old revision; future uses new revision; exception remains 17:30; incompatible new start flagged |
| 13 | Remove a slot with an exception | To resolve, followed by explicit one-off retention or cancellation with history |
| 14 | Offline planning basis fixture | Original 16:30 basis retained; incompatible later revision flagged |
| 15 | Calendar priority | Closure plus exception flagged; explicit opening overrides general holiday; contradictory institution records conflict; independent nanny calendar stays open |
| 16 | Missing calendar coverage | No presumed meal; explicit attendance remains counted with a calendar warning |
| 17 | Concurrent corrections in reversed arrival order | Identical incomplete summaries with the affected duration excluded |
| 18 | Conflicting lunch presence and absence | No declared or presumed meal; incomplete summary |
| 19 | Operation retry | One fact after identical replay; different content under the same identity rejected |
| 20 | Exit before care start | Incomplete result with no counted duration |

All 20 scenario functions completed. Their inspected outputs match the expected outcomes above. No application test suite was introduced. The three script blocks embedded in the standalone HTML were parsed together successfully, including the updated guided morning flows and a selectable decision-scenario panel.

## What this does not verify

- The browser security policy rejected opening the local HTML. No browser interaction, visual rendering, responsive layout or end-user acceptance is claimed. The earlier timeout limitation is superseded by this concrete policy block for this attempt.
- Planning helpers operate on explicit fixtures. They do not implement an integrated production calendar importer, occurrence generator, revision store or historical correction workflow. Blocking a move with attendance does not itself validate creation of the later replacement appointment.
- The offline case checks preservation of a pinned planning basis in memory. No local persistence, outbox, server, two-phone synchronization, authorization or recovery behavior was executed.
- Competing corrections are detected. Selection across competing corrections/resolutions, subsequent new conflicting facts and concurrent planning writes are not fully modeled. Duplicate selection is exercised only for the supported simple case.
- Time zones, daylight-saving transitions, overnight sessions, device-clock drift and server reconciliation are specified but not implemented by the same-day wall-clock fixtures.
- The earlier schedule-expansion helper remains limited; the new calendar and planning helpers are separate explorations. Passing these examples is not evidence that a composed scheduling pipeline is correct.

## Local artifacts and reproduction

The standalone demonstration is `/tmp/foylo-planned-observed-prototype.html`. Its source module, scenario functions and captured output are bundled in `/tmp/foylo-prototype-review.zip`. Open the HTML directly to inspect the interactive walkthroughs manually. These files are temporary local artifacts, not repository implementation files or published links.

Replay the scenario functions with Node:

```sh
node - <<'JS'
require('/tmp/foylo-planned-observed-domain.js');
require('/tmp/foylo-planned-observed-scenarios.js');
console.log(JSON.stringify(FoyloScenarios.map(s => ({name: s.name, expected: s.expected, result: s.run()})), null, 2));
JS
```

The module was also loaded from its TypeScript source under Node 24.14.0. The browser JS was regenerated using Node's type stripping. Hashes identify the reviewed artifact contents:

| Artifact | SHA-256 |
| --- | --- |
| `foylo-planned-observed-prototype.html` | `1ae56a50cc2196e624b735ff09af2756ff93140edd322f20595e9ef007bf00a2` |
| `foylo-planned-observed-domain.ts` | `9575b24ca6466a958b016dc6d5d282086be904709120d6e874aef7ad4aff171c` |
| `foylo-planned-observed-domain.js` | `20369d2cd235950ba1ca0bd0d34c3799254818f98729551375e87890f7a22545` |
| `foylo-planned-observed-scenarios.js` | `fb3f260e10029553fb8a101e1c1b7e0a3c17078a05a5937db40c06777ca014ce` |
| `foylo-planned-observed-results.json` | `cc37cf78d0cea59ec540ebc2705d1d360466a9b6fcc3d53aa8daaac28247d9ed` |

The repository keeps the conclusions and links only. No branch, commit, push, publication or issue closure was performed. The subsequent [conceptual model](02-domain-model.md) is now frozen for V1. The unverified integration cases remain acceptance requirements; the model freeze does not add runtime evidence.
