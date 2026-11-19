# Modal inference

The Scripture Intent Service converts stable transcript text plus session context into one constrained [projection intent](contracts.md). It does not operate VideoPsalm or return Bible verse text for rendering.

## Initial configuration

| Setting | Initial design choice |
| --- | --- |
| Model | `Qwen/Qwen3-1.7B` |
| Execution | Non-thinking mode |
| Serving | vLLM; compare SGLang after measurements |
| GPU | Start with an L4-class candidate; benchmark alternatives |
| Placement | Region close to the actual client/church network |
| Warm capacity | `min_containers=1` during live-service windows |
| Request shape | Short prompt, bounded recent context, short output limit |
| Output | Schema-constrained JSON; one action |
| Concurrency | Low serving concurrency, selected from load tests |

The model card documents a hard non-thinking switch through `enable_thinking=False` in the chat template. Configure the selected server to honor that switch; verify the response does not contain reasoning markup. Pin the model revision and serving/tokenizer versions after benchmarking. [Qwen model card](https://huggingface.co/Qwen/Qwen3-1.7B).

The design calls for deterministic inference settings; the exact sampling configuration remains a measured decision. Compare Qwen3-0.6B, 1.7B, and a larger baseline for extraction accuracy, context handling, and p50/p95 latency before finalizing the model.

## Serving lifecycle

Preload weights and initialize the inference engine before a service window. Keep connections warm, restrict output length, and measure both warm and cold requests. Keeping a minimum container count trades reduced cold-start exposure for idle resource cost; it is not a latency guarantee. [Modal cold-start guidance](https://modal.com/docs/guide/cold-start).

Implement an authenticated HTTPS handler for `/v1/interpret`. Reject unauthenticated or oversized requests before inference, constrain decoding, validate the response, and return correlated errors. A Modal web function/ASGI app is the hosting mechanism; the HSP bearer authentication contract must be implemented explicitly. [Modal web functions](https://modal.com/docs/guide/webhooks).

Deployment source, service-window scheduling, region, model revision, concurrency, token limits, and timeouts are configuration to finalize in the implementation repository. No endpoint or service deployment exists in this docs repository.

## Streaming policy

1. ASR displays partial hypotheses locally.
2. Send stable incremental spans after revision debounce; do not repeatedly submit the growing sermon.
3. Include current reference/translation and bounded recent context. Assign a request ID tied to the session and segment revision.
4. Discard superseded or late results even if HTTP cancellation did not stop server computation.
5. Route low-confidence, ambiguous, comparison, and multi-reference utterances to review. Thresholds need labeled benchmark calibration.

A newer inference response never bypasses Rust validation or the serialized projection queue.

## Network and data boundary

Microphone capture and the proposed ASR adapter run on Windows. Stable transcript text, current passage/translation, and recent text context leave the machine for Modal over authenticated HTTPS. Raw microphone frames are not part of the interpret API.

The setup UI must explain this boundary. Store the credential in Windows Credential Manager, call the service from Rust, and never expose the secret in React state, frontend storage, logs, or diagnostic exports. Do not retain raw audio or full sermon transcripts by default. Define client and service log retention before deployment.

If the network or inference service is unavailable, show a recoverable error and stop new voice-driven projection. Retain local operator control; do not silently switch models or replay a backlog after reconnection. Typed natural-language commands also use Modal. A separately implemented normalized-reference bypass must be explicit and validated, not assumed to exist.

## Quote lookup

`SEARCH_QUOTE` sends the spoken phrase to `VerseSearchService`, not directly to projection. Search an indexed, license-checked corpus and present ranked references. The operator chooses a candidate, which becomes a normal `SHOW_REFERENCE` command. An LLM's guessed verse is not a search result. This is Milestone D work, not a dependency for first-reference projection.
