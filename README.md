# HyperCast

Android TV receiver app. Current status: **milestones 1-3** (app shell, DLNA renderer, Cast discovery + connection probe).

## Build
1. Open this folder in Android Studio (Koala / 2024.1 or newer). Let it sync Gradle (uses Gradle 8.7, AGP 8.5.2).
2. Run on an Android TV device or the Android TV emulator (API 24+).
   For a real box: enable ADB debugging, then `adb connect <tv-ip>:5555` and press Run.

## Test DLNA
1. Open HyperCast once on the TV (it starts the background receiver service).
2. On a phone on the same Wi-Fi, open BubbleUPnP, VLC (Renderer menu) or any "cast to DLNA" app.
3. Pick "HyperCast (<model>)" as the renderer and play a video URL or a file from the phone.
Debug logs: `adb logcat -s HyperCast HyperCast.SSDP HyperCast.DLNA`

## Test Cast discovery (probe)
1. Open HyperCast on the TV, then run: `adb logcat -s HyperCast.Cast`
2. Expect `mDNS registered as HyperCast-...` and `Cast control channel listening on :8009`.
3. On a PC on the same network open Chrome -> menu -> Cast (or the Cast button in a sender app).
   HyperCast should appear in the device list. Try connecting and send the log output back.
4. Connecting is expected to stall at device authentication in this milestone. The log shows how far the sender gets.
If it doesn't appear: confirm phone/PC and TV are on the same subnet and the router isn't blocking mDNS (UDP 5353).

## Layout
- `PlayerController` - single Media3 pipeline every protocol feeds
- `ReceiverService` - foreground service owning the network listeners
- `dlna/SsdpServer` - discovery (M-SEARCH + NOTIFY)
- `dlna/DlnaServer` - device description + AVTransport / RenderingControl / ConnectionManager SOAP
- `cast/CastAdvertiser` - mDNS `_googlecast._tcp` registration
- `cast/CastServer` + `CastProto` - TLS :8009 control channel, protobuf framing, message logging
- `dlna/Services.kt` - service definitions, generates the SCPD XML

## Known limitations (this milestone)
- Not yet compiled or run by the author; expect a few small fixes on first build.
- UPnP eventing is stubbed (subscriptions accepted, no events sent). Controllers fall back to polling.
- Bringing the app to the front on a new cast is best effort; Android 10+ may block it when the app is in the background.
- Server restarts are not handled if the Wi-Fi IP changes while running.
