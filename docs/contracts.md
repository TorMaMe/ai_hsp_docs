# Data and API contracts

The shapes below are a **proposed v1 implementation contract** derived from the design. They are not evidence of a deployed endpoint. Share matching wire types and fixtures between Rust and the Modal service.

## Domain types

| Type | Essential fields |
| --- | --- |
| `AudioFrame` | Samples, channels, sample rate, monotonic capture timestamp |
| `TranscriptSegment` | Segment ID, revision, text, start/end, stable/final flags |
| `IntentRequest` | Request/session IDs, stable text, current passage/translation, recent context |
| `ProjectionIntent` | Action, reference, translation, range, quote, confidence, evidence span, request ID |
| `BibleReference` | Canonical book, chapter, start verse, optional end verse |
| `ProjectionCommand` | Command/request/session IDs, resolved reference/version, mode, creation/deadline, cancellation generation |
| `ProjectionResult` | Command ID, outcome, message, timings, observed selection/live state when available |

Use monotonic clocks for durations/deadlines and wall-clock timestamps for human-readable history. Preserve the model response for validation diagnostics only when the operator's privacy policy allows it.

## Interpret endpoint

`POST /v1/interpret` over HTTPS, authenticated with a bearer credential held by the Rust client. The service validates authentication before inference. The endpoint URL is configured during setup; none is deployed by this documentation.

```http
Content-Type: application/json
Authorization: Bearer <credential>
```

Request example ([JSON](examples/interpret-request.json)):

```json
{
  "request_id": "req-0001",
  "session_id": "session-0001",
  "stable_transcript": "Turn to Romans chapter eight verse twenty-eight",
  "current_reference": null,
  "current_translation": "KJV",
  "recent_context": []
}
```

Response example ([JSON](examples/interpret-response.json)):

```json
{
  "request_id": "req-0001",
  "action": "SHOW_REFERENCE",
  "reference": {"book": "Romans", "chapter": 8, "verse": 28},
  "translation": "KJV",
  "verse_range": null,
  "quote": null,
  "confidence": 0.98,
  "evidence_span": "Romans chapter eight verse twenty-eight"
}
```

`reference` is an object or `null`; `chapter` and `verse` are positive integers. A same-chapter `verse_range` is `{"start": 28, "end": 30}` or `null`; its start must equal `reference.verse`. Normalize into the domain reference only after checking consistency and canonical boundaries. Cross-chapter range support must be explicitly added and tested, not inferred from this shape.

`translation`, `quote`, and `evidence_span` are strings or `null`. Codes are normalized by Rust and mapped to installed translations; missing versions are errors, never silent substitutions. A null translation retains the configured/current version. Confidence is a number in `[0,1]`: it is **model-reported evidence, not a calibrated probability or permission to project**.

## Action rules

| Action | Required interpretation | Rust behavior |
| --- | --- | --- |
| `SHOW_REFERENCE` | Explicit complete reference; optional same-chapter range/version | Normalize, validate, resolve translation, create a command |
| `NEXT_VERSE` | Recent usable reference context | Compute the next canonical verse, including chapter/book boundaries |
| `PREVIOUS_VERSE` | Recent usable reference context | Compute the previous canonical verse; reject before the Bible's beginning |
| `SET_TRANSLATION` | Recognized translation and recent passage context | Resolve an installed version for the current passage |
| `SEARCH_QUOTE` | Non-empty quote; no guessed reference | Search the licensed index, show ranked candidates, require candidate selection |
| `NONE` | No actionable scripture request | Update transcript/status only |

For relative actions, Rust resolves context itself and rejects any conflicting model-supplied reference. Missing/ambiguous context goes to operator review. An incomplete book/chapter mention must not acquire an invented verse.

## Validation before execution

1. Require a matching request ID and an active, unexpired session/generation.
2. Deserialize the constrained schema; reject unknown actions/fields, invalid types, missing action-specific data, or malformed references.
3. Check evidence against the transcript, canonical chapter/verse metadata, range order, context freshness, and installed translation mapping.
4. Apply deduplication and uncertainty/multiple-reference checks. One model response never authorizes rapid presentation of several passages.
5. Enforce the selected mode and operator overrides, then enqueue one supported command.

No model field may specify a control selector, executable, keystroke sequence, file path, or arbitrary desktop action.

## Errors and results

Proposed HTTP errors use `{"request_id":"req-0001","error":{"code":"…","message":"…"}}`: `401/403` authentication, `422` bad input, `429` saturation, and `503/504` unavailable/timed-out inference. A local network error is also visible in HSP. Retry only the latest still-valid interpretation request within its deadline; never blindly retry a presentation whose outcome is unknown.

| Projection outcome | Meaning |
| --- | --- |
| `CONFIRMED` | Supported observation verifies the intended live reference and translation |
| `SENT_UNVERIFIED` | Input was sent, but live output could not be independently confirmed |
| `FAILED` | A known step failed; include the step and whether any input was sent |
| `CANCELED` | Work was canceled; report partial dispatch separately if it occurred |
| `UNSUPPORTED` | Required capability, translation, or verified workflow is unavailable |
| `SIMULATED` | Mock execution only; no audience output changed |

Emit UI events for connection, audio/ASR state, transcript, candidate, command step, result, and recoverable error. Every command result remains correlated with its IDs and stage timings.
