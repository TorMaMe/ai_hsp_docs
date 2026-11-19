# Decisions and references

## Design baseline

This documentation follows **HSP Rust Low-Level Design v0.1, 2026-11-18**. The Rust/Tauri desktop and Modal-hosted language interpretation are the planned product architecture. The existing [Python proof of concept](https://github.com/TorMaMe/ai_hsp_python) is a separate implementation and source of observed integration behavior.

The added HTTP field shapes, result names, and lifecycle rules make the design implementable; they are **proposed contracts**, not a claim that code or an endpoint already exists.

## Open decisions

| Decision | Evidence needed |
| --- | --- |
| ASR model and Rust binding | Streaming English scripture tests, Windows packaging, CPU/latency measurements, licensing |
| Qwen model size and sampling | Compare 0.6B, 1.7B, and a larger baseline on labeled church speech |
| vLLM vs SGLang | Accuracy, schema validity, warm/cold latency, and concurrency under the same workload |
| GPU/region/warm schedule | Actual client-network p50/p95, service windows, and operating cost |
| Confidence/stability/debounce thresholds | Calibration and false-projection tests; no arbitrary probability claims |
| Context and queue limits | Session tests, stale-response rejection, and measured backlog behavior |
| Canonical metadata and quote corpus | Chapter/verse coverage, translation differences, source provenance, and redistribution/indexing rights |
| VideoPsalm support matrix | Rust-side two-monitor tests per release, profile, translation, and action |
| Unresponsive automation recovery | COM/thread behavior, deadlines, cancellation, and worker/process recovery tests |
| Deployment and retention | Actual app/service build scripts, credential lifecycle, transcript/log policy |

## Primary references

These links support technology and product behavior; they do not replace testing on the installed VideoPsalm release. Pin implementation dependencies after verification rather than treating a `latest` documentation URL as a version lock.

- [Python prototype — private repository](https://github.com/TorMaMe/ai_hsp_python)
- [VideoPsalm Bible search and presentation guide](https://myvideopsalm.weebly.com/how-to-display-song-lyrics-and-bible-verses.html)
- [VideoPsalm keyboard controls](https://myvideopsalm.weebly.com/blog/how-to-control-videopsalm-with-a-keyboard-without-a-mouse)
- [Rust uiautomation crate](https://docs.rs/uiautomation/latest/uiautomation/)
- [Tauri 2 prerequisites](https://v2.tauri.app/start/prerequisites/)
- [sherpa-onnx](https://k2-fsa.github.io/sherpa/onnx/)
- [Qwen3-1.7B model card](https://huggingface.co/Qwen/Qwen3-1.7B)
- [Modal low-latency inference example](https://modal.com/docs/examples/trtllm_latency)
- [Modal cold-start guidance](https://modal.com/docs/guide/cold-start)
- [Modal web functions](https://modal.com/docs/guide/webhooks)
- [Material for MkDocs diagrams](https://squidfunk.github.io/mkdocs-material/reference/diagrams/)
