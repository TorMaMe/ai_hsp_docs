# High Speed Projection

**Design v0.1 · 18 November 2026 · Windows 10/11**

HSP listens to a speaker, recognizes a Bible passage, and directs **VideoPsalm** to display it. VideoPsalm owns the Bible library, slide formatting, and audience output. HSP owns speech capture, command interpretation, session context, validation, and controlled automation.

!!! info "Design status"
    This site describes the **planned Rust product**, not completed Rust functionality. The Python proof of concept supplies integration evidence; each Rust milestone needs its own acceptance test.

## First milestone

Prove this complete path on a Windows computer with an external display:

**Microphone → transcript → structured scripture intent → VideoPsalm search → live projection.**

A **Type a scripture command** field exercises the same intent and projection pipeline without audio. Build the VideoPsalm text-entry path first, then add inference and voice.

## Technology choices

| Layer | Choice | Status |
| --- | --- | --- |
| Desktop | Tauri 2 + React/TypeScript | Planned |
| Orchestration | Rust + Tokio | Planned |
| Audio | Rust `cpal` | Planned |
| Streaming recognition | ASR adapter; sherpa-onnx is the initial candidate | Model/binding selection pending |
| Language interpretation | Qwen3-1.7B, non-thinking, on Modal | Initial candidate; benchmark required |
| Inference server | vLLM; evaluate SGLang after measurement | Initial serving choice |
| Desktop control | Windows UI Automation / Win32 | Port and verify the prototype flow |
| Settings and secrets | Local Rust persistence + Windows Credential Manager | Planned |

**The Rust design is network-dependent for language interpretation.** Audio/ASR run locally; stable transcript text and bounded context go to Modal. This differs from the Python prototype's local rule-based interpretation. See [inference and privacy](inference.md#network-and-data-boundary).

## Read next

- [Architecture](architecture.md): ownership, modules, concurrency, and lifecycle.
- [Contracts](contracts.md): domain types, actions, and the proposed HTTP interface.
- [VideoPsalm](videopsalm.md): verified prototype observations and the Rust transaction.
- [Interface](interface.md): simple operator flow, modes, and setup.
- [Delivery](delivery.md): milestones, benchmarks, and release gates.
- [Decisions](decisions.md): unresolved choices and primary references.
