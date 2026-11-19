# Delivery and acceptance

Build in this order. Each milestone needs executable evidence, not only a design or successful input call.

## Milestones

| Milestone | Deliverable | Exit criterion |
| --- | --- | --- |
| **A — VideoPsalm parity** | Rust Windows adapter + typed-reference harness | Reproduce the verified Python search/select/present flow and confirm audience output on two monitors |
| **B — Modal interpretation** | Authenticated endpoint, Qwen3-1.7B serving candidate, shared contracts/fixtures | Realistic text becomes valid, contextual intents; accuracy and latency benchmarked |
| **C — Voice end to end** | Microphone + streaming ASR + full pipeline | Clear spoken references reach VideoPsalm with Approval; duplicates, lag, and session stop are exercised |
| **D — Contextual actions** | Navigation, translation changes, quote-search candidates | Next/previous, range, installed-version switching, and reviewed quote lookup reuse the same command system |

## Development prerequisites

The application targets Windows 10/11; verify architecture and native-library compatibility before pinning releases. Rust development needs the Windows MSVC toolchain, Microsoft C++ build tools, and WebView2; the React frontend also needs a compatible Node.js toolchain. Follow the [official Tauri prerequisites](https://v2.tauri.app/start/prerequisites/) and pin the app toolchains/lockfiles when scaffolding it.

Evaluate the streaming ASR Rust/native binding and package its runtime/model assets. sherpa-onnx is a candidate with local recognition support, not yet a selected HSP model or measured latency result. [sherpa-onnx documentation](https://k2-fsa.github.io/sherpa/onnx/).

The Modal service additionally needs its own Python environment, deployment configuration, model weights, GPU allocation, and authentication secret. App build/run/deploy commands belong in the implementation repo once its actual scripts exist. The [documentation README](https://github.com/TorMaMe/ai_hsp_docs) provides working MkDocs commands for this repo.

## Test coverage

| Layer | Required cases |
| --- | --- |
| Audio/ASR | Device selection/unplugging, resampling, bounded buffers, revision stability, silence/noise, long sessions |
| Interpretation | Spoken numbers, numbered books, aliases, ranges, incomplete mentions, ordinary numbers, comparisons, quote vs explicit reference |
| Session/navigation | Fresh/stale context, chapter/book boundaries, previous at the beginning, unavailable translations |
| HTTP | Authentication, malformed output, unknown actions, timeout, saturation, retries, out-of-order and superseded responses |
| Coordinator | All modes, dedupe, one serialized transaction, cancel/pause, stale queue replacement, sent-but-unverified outcomes |
| Windows adapter | Wrong focus, missing controls, changed handles/version/DPI, minimization, restart, slow UI, multiple instances, privilege mismatch |
| UI/package | Small-screen scrolling, visible errors, operator overrides, clean-machine startup and model/runtime provisioning |

Use deterministic fake ASR/intent/adapter providers in automated tests. Keep labeled utterances and expected actions in fixtures. Hardware tests remain separate from mock success.

## Measurement

Record per-request timestamps, settings, model revisions, hardware, network region, and VideoPsalm version. Report p50/p95 for:

- **ASR stabilization:** end of the spoken reference to a stable usable segment.
- **Modal round-trip:** client send to validated response, including network and serving time.
- **Search/selection:** adapter start through matching VideoPsalm selection.
- **End to end:** end of speech to independently observed live output.

Include queue time and cold/warm state. If live output cannot be observed, measure **command sent** latency separately; do not call it confirmed projection latency.

Design targets under tested clear-speech conditions are **p50 ≤ 2 s**, **p95 ≤ 4 s**, and **≥ 98% correct reference extraction** on a representative labeled benchmark. They are not measured results or guarantees. Define exact-match scoring, ambiguous/no-action negatives, dataset composition, and held-out evaluation before using the accuracy figure as a release claim.

## Rehearsal and release gate

On a non-live two-display setup, verify a typed passage, a spoken passage, installed translation handling, preview/live separation, duplicate suppression, Stop/override, and recovery from lost focus/network/app restart. Record audience observations independently of input success. Repeat after relevant profile, model, or application changes.

Before live use, require correct observable projection confirmation for the claimed supported path, operator override, and safe error recovery. Any unobservable outcome remains labeled `SENT_UNVERIFIED` and does not satisfy a confirmed-projection acceptance check. Package the Windows app, native ASR dependencies/models, and operator setup instructions; validate the distribution on a clean target machine.
