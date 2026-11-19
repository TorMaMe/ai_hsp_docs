# Interface and settings

Keep the main window focused on a service operator's task. Detailed setup and diagnostics belong on a separate setup page, with scrolling on small screens.

## Main window

| Area | Content |
| --- | --- |
| Connection | Connect to VideoPsalm; Connected/Disconnected and capability/setup status |
| Listening | Microphone selector, Start Listening, Stop, clear audio/ASR state |
| Speech | Latest transcript; distinguish unstable partial text from stable text |
| Scripture | Detected passage, active/requested translation, review state |
| Projection | Project when a candidate is ready; last command/result and retry guidance |
| Testing | One **Type a scripture command** field using the same intent pipeline |

Show **Detected/Connected**, **Selection ready**, **Command sent**, and **Projection confirmed** as distinct states. Do not imply successful projection from process detection alone. Errors should identify the failed stage and the next useful action.

## Operator flow

1. Open VideoPsalm and select the intended Bible translation and display setup.
2. Connect HSP; confirm the selected instance and available capabilities.
3. Choose the microphone. Complete ASR/model and Modal setup if needed.
4. Press Start Listening. In the default Approval mode, review the detected passage and press Project.
5. Confirm the audience output during rehearsal. Stop cancels listening and unexecuted automation; it does not clear VideoPsalm's display.

For the observed 1.29 workflow, projection must already be enabled in VideoPsalm. Future profiles may expose a separately verified presentation action; do not assume a universal toggle or shortcut.

## Modes

| Mode | Detection and execution |
| --- | --- |
| **Approval — initial default** | Detect and queue; the operator authorizes presentation with Project |
| Manual | Show detections for review; only an explicit operator action drives automation |
| Automatic — opt-in | Execute only a fresh, validated, unambiguous intent through a supported profile/version mapping |

Start Listening does **not** silently switch Approval into Automatic. Confidence alone never enables Automatic execution. Keep Stop, cancel-pending/pause controls, and operator override accessible. Do not replay old automatic commands after recovery.

## Setup and persistence

| Setting | Storage/behavior |
| --- | --- |
| Microphone | Device identity and capture settings; handle disappearance/replacement |
| ASR | Provider/model location and compatibility; model provisioning is explicit |
| Modal | Endpoint URL and service configuration in local settings |
| Credential | Windows Credential Manager; accessed only by Rust |
| Translation | Preference plus confirmed installed/displayed mapping |
| Automation | Versioned VideoPsalm profile and calibrated capabilities |
| Session policy | Mode, context expiry, dedupe, inference/command deadlines |
| Diagnostics | Retention and optional transcript logging; redacted exports |

Store non-secret configuration under the user's application-data directory. No secrets go into source control or frontend persistence. Show permission, model-loading, network/authentication, and automation errors with appropriate retry/setup actions.
