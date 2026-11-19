# VideoPsalm integration

Treat the [private Python prototype](https://github.com/TorMaMe/ai_hsp_python) as an integration reference, not a public control API. Port its observed behavior and re-test it in Rust on the installed release.

## Prototype evidence

On the operator's **VideoPsalm 1.29** installation, entering `Genesis 1:1` in Bible reference search and pressing **Enter** selected the verse. The operator confirmed that it appeared on the external monitor while VideoPsalm's **Project icon was already enabled**.

This confirms that interaction under those conditions; it does not prove the Rust adapter, other releases, translation switching, ranges, or every preview/live mode. Activating a window is different from giving keyboard focus to the reference Edit. The prototype encountered that distinction, and its focus guard prevented typing into the wrong control.

Observed identifiers to inspect—not universal selectors:

| Control | Observed identifier |
| --- | --- |
| Main form | `FormMain` |
| Reference entry | `buttonEditSearchBibleReference` |
| Active translation | `searchLookUpEditBibleList` |
| Book selection | `listBoxControlBibleBooks` |
| Chapter selection | `listBoxControlBibleChapters` |
| Verse selection | `listBoxControlBibleVerses` |
| Passage/slide sequence | `gridControlSequence` |

Reconfirm IDs, native handles, control roles, and keyboard behavior before using them. Custom DevExpress controls may expose native child Edits more reliably than a generic accessibility value setter. Never persist a native handle as permanent identity; rebind after restart and validate ownership before input.

## Adapter interface

| Operation | Contract |
| --- | --- |
| `connect()` | Locate the intended process/window and bind the profile |
| `inspect_capabilities()` | Discover Bible search, selection, preview/live controls, installed translations |
| `search_reference(reference)` | Focus search, enter the normalized reference, await matching results |
| `select_passage(reference, translation)` | Select the matching range/version; reject mismatches |
| `preview_selection()` | Prepare/verify a preview only if it is known not to affect live output |
| `present_selection()` | Execute the verified live-display action or supported existing-project-mode sequence |
| `read_status()` | Report connection, selected passage, and observable preview/live state |

Expose unsupported capabilities explicitly. Keep selectors, verified shortcuts, version information, and timeouts in a versioned automation profile, outside business logic.

## Transaction

1. Acquire the one command lock; check command age and cancellation generation.
2. Verify executable/process, window identity, profile compatibility, and control ownership.
3. Focus the required control and verify actual keyboard focus before text entry.
4. Clear stale input, enter the normalized reference, and wait for the result/selection change.
5. Resolve the installed translation and exact verse/range. Read back selection; fail if it differs.
6. Verify a safe preview if supported. Otherwise retain HSP's internal candidate until presentation is authorized.
7. Present using the calibrated workflow. For the observed 1.29 route, leave the operator's Project toggle unchanged and use reference entry + Enter.
8. Read any supported live observation, publish an honest result, and record each step's duration.

In Manual/Approval modes, do not call a search/preview operation automatically unless its effect on the audience display is verified safe. The selected passage or successful keypress alone is not proof of live output: report `SENT_UNVERIFIED` when necessary.

The vendor keyboard guide documents Ctrl+F search behavior. Treat it as a **calibration candidate**, not a verified shortcut for this integration. [VideoPsalm keyboard controls](https://myvideopsalm.weebly.com/blog/how-to-control-videopsalm-with-a-keyboard-without-a-mouse).

## Failure and recovery

Wrong focus, missing controls, unavailable translations, slow searches, app restart, privilege mismatch, and operator interaction must produce a visible step-specific error. Stop the transaction rather than guessing a coordinate, replacing a translation, or retrying a possibly live command. Reconnect/recalibrate after identity or layout changes.

## Integration fixtures

Keep sanitized fixtures under `tests/fixtures/` in the app repo: VideoPsalm version, Windows/DPI/display setup, process/window selectors, installed translation mapping, exact action sequence, step timings, expected/observed selection, and external-display verification. Capture real screenshots only from the tested installation. Include focus-loss, restart, and failure traces; exclude credentials, personal paths, and sermon recordings.
