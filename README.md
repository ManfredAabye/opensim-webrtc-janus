# opensim-webrtc-janus

Das ist mein Test Repository für das interne os-webrtc-janus im OpenSimulator.

This is my test repository for the internal os-webrtc-janus in OpenSimulator.

## Fazit Conclusion

Stand 07.03.2026 22:24 keinerlei abstürze oder Fehler mehr festzustellen.

As of March 7, 2026 at 10:24 PM, no more crashes or errors have been detected.

---

⚠️ This overview is AI-generated because I was unable to document the 230 working hours I invested.

---

Description - March 14, 2026

Treat "Janus plugin events" without a transaction as normal asynchronous flows instead of errors.

Log events without a transaction only as debug (with _MessageDetails enabled).

Log events with an existing but unknown transaction as a warning (instead of an error) so that genuine anomalies remain visible.

Leave event processing unchanged (OnEvent is still called).

Result

No more misleading "event no outstanding request error" during normal "Janus reconnect/push event flows".

Audio/signaling behavior remains functionally unchanged (verified).

.

Test 4 regions combined as a cube.

Test voice on Region A

Test voice on Region B

Test voice on Region C

Test voice on Region D

Test voice while moving between Regions A -> B -> C -> D -> A ->

Status: No problems detected.

---

## os-webrtc-janus Delta Summary vs upstream opensim/opensim

## Scope

This summary is based on the local `opensim` worktree relative to upstream `origin/master` (`https://github.com/opensim/opensim`).
It covers `OpenSim/Addons/os-webrtc-janus` and directly related config/docs changes.

## Change size

- 18 tracked files changed in addon/config/docs scope
- Approx. `+1366 / -429` line delta
- 1 new helper file: `OpenSim/Addons/os-webrtc-janus/Janus/OSDOps.cs`

## High-level changes

- Reliability hardening for Janus session/message flow (transaction matching, async response handling, safer cleanup)
- Reconnect/session lifecycle improvements across viewer-session handling
- Better room join/leave resilience in AudioBridge edge cases
- Provision/signaling flow cleanup in service module and connectors
- New signaling throttle control (`MaxSignalingCandidatesPerRequest`)
- Better diagnostics and console observability (`janus info`, `janus info json`, `janus show`, `janus list rooms`, `janus list sessions`, `janus room <roomId>`)
- README/config updates aligned with runtime behavior

## Detailed delta by area

## 1) Janus protocol/session reliability

Files:

- `OpenSim/Addons/os-webrtc-janus/Janus/JanusSession.cs`
- `OpenSim/Addons/os-webrtc-janus/Janus/JanusMessages.cs`
- `OpenSim/Addons/os-webrtc-janus/Janus/BHasher.cs`

Key changes:

- More explicit transaction ID handling for request/response matching
- Better handling of `ack` + deferred event completion in async request flow
- Safer outstanding-request bookkeeping and cleanup paths
- Improved trickle signaling handling (`candidate`, `candidates`, completion marker)

## 2) Room lifecycle and race-condition handling

Files:

- `OpenSim/Addons/os-webrtc-janus/Janus/JanusRoom.cs`
- `OpenSim/Addons/os-webrtc-janus/Janus/JanusAudioBridge.cs`

Key changes:

- Join recovery for AudioBridge `Already in a room` (`errorCode=491`)
- Stale-membership recovery by listing rooms/participants and forcing leave, then retrying join
- Leave-path hardening for benign "already left" races (`errorCode=487`) and `ack/event` outcomes

## 3) Viewer session robustness

Files:

- `OpenSim/Addons/os-webrtc-janus/Janus/JanusViewerSession.cs`
- `OpenSim/Addons/os-webrtc-janus/WebRtcVoice/VoiceViewerSession.cs`
- `OpenSim/Addons/os-webrtc-janus/WebRtcVoice/IVoiceViewerSession.cs`

Key changes:

- Duplicate-disconnect suppression and disconnect reason tracking
- Serialized provisioning via lock to avoid overlapping provision races
- Reuse/cleanup helpers for stale or duplicate viewer sessions
- Safer shutdown ordering (leave room, detach plugin, destroy session)

## 4) Service request flow and fallback behavior

Files:

- `OpenSim/Addons/os-webrtc-janus/Janus/WebRtcJanusService.cs`
- `OpenSim/Addons/os-webrtc-janus/WebRtcVoiceServiceModule/WebRtcVoiceServiceModule.cs`
- `OpenSim/Addons/os-webrtc-janus/WebRtcVoice/WebRtcVoiceServiceConnector.cs`
- `OpenSim/Addons/os-webrtc-janus/WebRtcVoice/WebRtcVoiceServerConnector.cs`
- `OpenSim/Addons/os-webrtc-janus/WebRtcVoiceRegionModule/WebRtcVoiceRegionModule.cs`
- `OpenSim/Addons/os-webrtc-janus/WebRtcVoice/IWebRtcVoiceService.cs`

Key changes:

- Unified async call paths for `ProvisionVoiceAccountRequest` and `VoiceSignalingRequest`
- Better reconnect behavior when `viewer_session` is missing/stale
- Fallback reuse by agent+scene when `channel_type` is missing but reusable session exists
- Explicit error responses for invalid/missing session context instead of silent null pass-through
- Rejoin cooldown support to reduce rapid disconnect/rejoin churn

## 5) Signaling controls

Files:

- `OpenSim/Addons/os-webrtc-janus/Janus/WebRtcJanusService.cs`
- `bin/config/os-webrtc-janus.ini.example`
- `OpenSim/Addons/os-webrtc-janus/README.md`

Key changes:

- New `MaxSignalingCandidatesPerRequest` setting
- Default `20` candidates per `VoiceSignalingRequest` call
- `<= 0` means unlimited behavior
- Logging when candidate lists are capped

## 6) Observability and operator tooling

Files:

- `OpenSim/Addons/os-webrtc-janus/Janus/WebRtcJanusService.cs`
- `OpenSim/Addons/os-webrtc-janus/README.md`

Key changes:

- Improved `janus info` output in human-readable form
- Added/clarified commands: `janus info json`, `janus show`, `janus list rooms`, `janus list sessions`, `janus room <roomId>`

## 7) New OSD helper utilities

File:

- `OpenSim/Addons/os-webrtc-janus/Janus/OSDOps.cs` (new)

Key changes:

- Added helper extensions for safer OSD extraction (`TryGetString`, `TryGetOSDMap`, value helpers)
- Reduces repetitive map/value parsing code and null-conversion hazards

## Related docs touched outside addon scope

- `README.md` (top-level): includes a WebRTC/Janus update block summarizing the refresh

## Upstream packaging recommendation

Given scope and mixed concerns, upstream review will be easier if split into focused PRs:

1. Protocol/session reliability internals (`JanusSession`, `JanusMessages`, helpers)
2. Room lifecycle hardening (`JanusRoom`, `JanusAudioBridge`)
3. Service/request flow changes (`WebRtcJanusService`, service module/connectors)
4. Config/docs/console command updates

## Ready-to-send text for OpenSim DEV group

Subject: os-webrtc-janus delta vs upstream opensim/opensim

We prepared an os-webrtc-janus refresh on top of upstream `opensim/opensim` (`origin/master`). Main deltas are:

1. Reliability hardening in Janus request/response flow (transaction matching, ack/event handling, safer cleanup).
2. Room lifecycle fixes for known Janus race/error patterns, including join recovery for `Already in a room` (`491`) and benign leave handling for already-left race (`487`).
3. Viewer-session robustness improvements (duplicate-disconnect suppression, stale session cleanup/reuse, safer shutdown ordering).
4. Unified async service flow for `ProvisionVoiceAccountRequest` and `VoiceSignalingRequest`, with clearer error responses on missing session context.
5. New signaling control: `MaxSignalingCandidatesPerRequest` (default `20`, `<=0` unlimited), including cap logging.
6. Better observability and console diagnostics (`janus info`, `janus info json`, `janus show`, `janus list rooms`, `janus list sessions`, `janus room <roomId>`).
7. Added OSD helper utilities (`OSDOps`) to improve parser safety and reduce repetitive conversion code.

Overall delta in addon/config/docs scope is roughly `+1366/-429` across 18 tracked files, plus one new helper file. Recommended for upstreaming as multiple focused PRs due to feature breadth.
