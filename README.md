# PulseSync

Cross-device Pomodoro and media command center for Android and Windows desktop.

## Implemented modules

- Shared MVI timer reducer with transition revisions and locally derived countdown time.
- Ktor CIO WebSocket host/client at `/pulsesync`, JSON envelopes, bearer-token pairing, and four-timestamp NTP calibration on connect and every 30 seconds.
- Android NSD and desktop JmDNS peer discovery, ZXing QR pairing, and manual IP/token entry.
- Android notification-listener media sessions and a Windows GSMTC PowerShell bridge.
- Four animated timer faces, responsive layouts, auto-follow/manual media routing, and album-art seeded MaterialKolor expressive theme.

The 256-bit token is not published over mDNS; mDNS exposes only service name, address, and port. The token is included in the QR/manual pairing flow and checked during the WebSocket hello exchange.

Android cleartext is enabled through the manifest and network security config for the requested `ws://` transport. The WebSocket client only accepts private IPv4 peer addresses (`10/8`, `172.16/12`, `192.168/16`) or `.local` mDNS names. The desktop app has a compact always-on-top mini window.

## Build

Open this directory in Android Studio with JDK 17 and use `:composeApp:assembleDebug` (Android) or `:composeApp:run` (desktop). Desktop packaging targets Windows MSI. Android notification access must be enabled in Settings before media-session control is available.

## Protocol contract

`TimerSyncState` is the replicated snapshot. `revision` is monotonically increased on state transitions. `ClockCalibration.offsetMillis` is remote epoch minus local epoch, and `remainingAt()` projects the countdown on the receiving device. NTP samples use all four timestamps and include measured round-trip time. A different `sessionId` begins a new replication stream; stale/duplicate revisions in the same session are discarded.
