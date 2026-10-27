# PulseSync

PulseSync combines a Pomodoro timer and a local-network media command center for **Android** and **Windows desktop**. The timer can be synchronized between paired devices, while the media panel can show and control an active player on either side of the connection.

This README describes the features present in the project. The comparison below is category-level rather than a claim about any named competitor; other Pomodoro apps vary in their feature sets.

## Feature overview

### Pomodoro timer

- **Three phases:** Focus, Short break, and Long break.
- **Current durations:** 25 minutes for Focus, 5 minutes for Short break, and 15 minutes for Long break. These values are currently defined by the app; users can change the displayed timer size, but not the phase durations in Settings.
- **Timer controls:** Start, pause, resume, reset, and select a phase.
- **Automatic cycle progression:** Completing a Focus phase increments the completed-focus count and selects a short break, or a long break after every fourth completed focus. Completing a break returns to Focus.
- **Seven timer faces:**
  1. **Polygon** — a rotating, morphing polygon with a progress accent.
  2. **Kinetic** — oversized minutes and seconds typography.
  3. **Wavy** — a circular wave display that responds to playback activity.
  4. **Bento** — split, proportionally animated time cards.
  5. **Orbit** — a radial progress dial with a moving orbit marker.
  6. **Flip** — separate animated minute and second blocks.
  7. **Ripple** — concentric radial progress rings.
- **Timer size setting:** Adjust the timer face from 100% to 250%; the selection is saved on the device and applies to portrait, landscape, and desktop layouts. Mini mode uses a compact scale.
- **Motion and refresh:** Compose animations run on the platform frame clock. Android requests the display’s highest supported refresh mode when available.

### Cross-device timer sync

- Either Android or desktop can host or connect to a peer.
- Countdown values are calculated locally from a replicated timer snapshot. PulseSync sends timer state on transitions rather than sending a network message for every displayed second or frame.
- A four-timestamp NTP-style exchange estimates clock offset when connected and is repeated every 30 seconds. The offset is used when a receiving device projects the shared countdown.
- Session IDs and increasing revisions allow peers to reject stale or duplicate timer snapshots.
- The connection state and estimated clock offset are visible in the pairing area.

### Pairing and local-network discovery

- **Nearby pairing:** Android advertises and discovers peers with Android NSD; Windows desktop uses JmDNS. Select a nearby peer and enter the four-digit pairing code shown on that device.
- **QR fallback:** A pairing QR can carry the host address, port, and session token. Android can scan a desktop QR when discovery is unavailable.
- **Advanced manual pairing:** Enter the peer’s private-network IP address, port, and session token.
- **Reconnect support:** A successfully paired peer can be saved for reconnection.
- **Port:** The app’s default pairing/WebSocket port is `43821`.

### Media playback and remote control

- **Android:** Reads active media sessions through Android’s notification-listener/media-session APIs and sends transport commands through the session controller. Notification access must be enabled for the app in Android Settings.
- **Windows:** Reads Windows Global System Media Transport Controls sessions through the bundled PowerShell WinRT bridge. A player must expose a Windows media session for its track and controls to be available.
- **Track information:** Title, artist, album, player/source, playback state, track position, duration, and album artwork when the player supplies them.
- **Playback controls:** Play, pause, toggle, previous, next, and seek.
- **Track timeline:** The media card displays elapsed time, remaining time, and total duration alongside its seek control.
- **Album art:** Artwork is read from the platform media session where available, resized/compressed for Android, and normalized or passed through for desktop rendering.
- **Target selection:**
  - **Auto follow** selects the device that most recently started playback.
  - **Manual target** lets the user choose This device or Remote peer.
- **Media access UI:** Android shows an enable-access action when notification access is not enabled. The access-enabled status text is hidden.

### Appearance and window modes

- Material 3 surfaces, typography, animated controls, phase chips, and spring-based transitions.
- The active album artwork can seed the app’s dynamic color palette; phase colors provide a fallback when there is no artwork.
- **Dark mode** uses a dark Material color scheme.
- **Emulate Dark Mode** uses a fully black background with the dark scheme’s foreground and control colors.
- **Fullscreen:** Android hides the system bars and the app header. Tap anywhere to reveal the header temporarily; it hides again after a short delay. Desktop uses a fullscreen window placement.
- **Windows mini mode:** Shrinks PulseSync into a compact always-on-top window with timer and media skip controls. The window size transitions with a spring animation.
- **Adaptive layout:** The UI rearranges for compact and expanded windows, including phone orientation changes and desktop two-pane layouts.

## How PulseSync compares with a conventional single-device Pomodoro timer

| Area | Common single-device timer pattern | PulseSync capability |
| --- | --- | --- |
| Timer state | Countdown owned by one screen | Timer phase and transitions can replicate between paired Android and Windows devices |
| Synchronization | No peer sync, or a continuously updated shared timer | Peers derive remaining time from a shared transition snapshot and calibrated clocks; no per-second timer traffic |
| Pairing | Account/cloud pairing or no pairing | LAN discovery with a four-digit confirmation code, plus QR and manual IP/token fallbacks |
| Media | Often separate from the timer | A single panel can display and command local or remote Android/Windows media sessions |
| Playback destination | Usually controls the current device | Auto-follow or explicit local/remote target selection |
| Timer presentation | One or a small number of clock styles | Seven animated timer faces and an adjustable face size |
| Window use | Standard app window | Android fullscreen and Windows fullscreen/always-on-top mini mode |
| Network model | May rely on account services depending on the product | PulseSync peer traffic is designed for a private local network and does not require a PulseSync cloud account |

This table compares product approaches, not every feature available in other apps. PulseSync’s phase durations are fixed in the current implementation, and it does not currently provide cloud accounts, cross-network relay, or a general task/project planner.

## Pairing steps

### Nearby pairing with a code

1. Put both devices on the same local network and open PulseSync on each.
2. Open **Settings → Pair device** on either device.
3. Select the other device from the nearby list.
4. Enter the four-digit code displayed by the selected device, then confirm the connection.

### QR fallback

1. On desktop, open Settings and display its pairing QR.
2. On Android, open the pairing area and choose **Scan pairing QR**.
3. After scanning, choose **Connect**. If the network or firewall prevents the connection, allow PulseSync on the Windows private network and retry.

### Advanced manual pairing

Open the **Advanced** pairing tab on the connecting device and enter the peer’s LAN IP address, port (normally `43821`), and session token. Keep this option for networks where automatic discovery is blocked or unavailable.

## Privacy and network security

- Timer and media synchronization are direct peer-to-peer WebSocket messages on the local network; the application has no cloud relay or account service.
- The app uses `ws://` on LAN, so WebSocket contents are **not encrypted in transit**. Use pairing only with devices and networks you trust; do not expose the port to the public internet.
- Android cleartext networking is enabled for the requested private-LAN WebSocket connection.
- mDNS discovery advertises the device name, address, and port, not the 256-bit session token. The token is used in QR/manual pairing and the WebSocket authentication exchange. Four-digit code pairing confirms peer intent before the host’s token is accepted.
- The WebSocket client restricts addresses to private IPv4 ranges (`10/8`, `172.16/12`, `192.168/16`) or `.local` names.
- Windows Firewall may need to allow PulseSync on **Private networks** for incoming connections and discovery.

## Platform requirements and permissions

### Android

- Android notification-listener access is required to read/control active media sessions. The app provides a shortcut to the relevant Android settings page.
- Camera permission is used for the QR scanner. The camera is optional for installing and using the timer or manual pairing.
- LAN discovery and WebSocket connections use Android internet/network and Wi-Fi multicast permissions.

### Windows desktop

- Windows media information and playback commands require a player that publishes a GSMTC session. Some apps may expose incomplete track details or no artwork.
- Desktop media access is implemented with a PowerShell WinRT sidecar to avoid requiring a native C++/Rust compiler toolchain.
- Windows Firewall can block inbound LAN connections or multicast discovery until PulseSync is allowed on a private network.

## Build and run

Open the `outputs/PulseSync` project in Android Studio with JDK 17. The Gradle tasks are:

```text
:composeApp:assembleDebug       Build the Android debug APK
:composeApp:compileKotlinDesktop  Compile the desktop target
:composeApp:run                Run the desktop app
:composeApp:packageMsi         Package the Windows desktop app as an MSI
```

Desktop packaging is configured for Windows MSI. The project uses Kotlin Multiplatform, Compose Multiplatform, Kotlin serialization, Ktor CIO/WebSockets, Android NSD, JmDNS, and the Windows GSMTC bridge.

## Architecture notes

- `commonMain/core/Timer.kt` contains the timer state model, reducer, completion progression, revision handling, and clock-offset math.
- `commonMain/lan/PeerTransport.kt` contains the shared WebSocket protocol, pairing authentication, periodic clock calibration, and timer/media replication.
- `commonMain/media/Media.kt` defines media state, commands, and target-selection models.
- `commonMain/ui/PulseSyncApp.kt` contains the shared timer, media, settings, pairing, and adaptive Compose UI.
- `androidMain` provides Android media-session, notification-listener, NSD, QR scan, and platform persistence implementations.
- `desktopMain` provides the Compose Desktop window, JmDNS discovery, Windows GSMTC bridge, and desktop platform persistence.

## Protocol contract

`TimerSyncState` is the replicated timer snapshot. `revision` increases on state transitions. `ClockCalibration.offsetMillis` is remote epoch time minus local epoch time. `remainingAt()` projects the countdown locally using the calibrated clock. NTP samples use four timestamps and calculate round-trip time. A new `sessionId` starts a new replication stream; stale or duplicate revisions in the same session are ignored.
