# better_native_video_player — Fork

This fork contains multiple Android PiP and fullscreen behavior fixes tailored for Flutter applications with custom inline video players.

## Changes Compared to the Original Package

### Android PiP Improvements

#### Added `isPipAllowedWhileNotInFullscreen`

New controller parameter:

```dart
isPipAllowedWhileNotInFullscreen
```

This allows controlling whether Picture-in-Picture can be entered while the player is not in fullscreen mode.

Example:

```dart
NativeVideoPlayerController(
  isPipAllowedWhileNotInFullscreen: !Platform.isAndroid,
)
```

Main use case:

* Disable inline PiP on Android because Android PiP snapshots the entire FlutterActivity instead of only the video surface.
* Keep inline PiP enabled on iOS.

---

### Fullscreen Orientation Fixes

Fixed Android fullscreen orientation behavior.

Original package behavior:

* Fullscreen mode was effectively landscape-only on some devices.

Fork behavior:

* Full portrait + landscape support in fullscreen mode.

Android implementation changed from:

```kotlin
SCREEN_ORIENTATION_SENSOR
```

to:

```kotlin
SCREEN_ORIENTATION_FULL_SENSOR
```

This allows:

* portrait fullscreen
* portrait upside-down
* landscape left
* landscape right

depending on device orientation.

---

### Android PiP Stability Improvements

Improved Android PiP lifecycle handling.

Changes include:

* Proper cleanup of native PiP state during controller disposal.
* Proper disabling of automatic PiP when a controller is released.
* Better handling of controller-scoped PiP behavior.

This prevents issues where:

* PiP could still activate after leaving the video page.
* Background playback continued after controller disposal.
* Android kept stale native player references alive.

---

### Fixed `MissingPluginException` During Dispose

Fixed:

```text
MissingPluginException(No implementation found for method cancel on channel native_video_player_controller_xxx)
```

Cause:

* EventChannel cleanup race conditions during platform view disposal.

Fix:

* Safe native event channel disposal handling.
* Better controller lifecycle cleanup.

---

### Android Inline PiP Behavior

This fork keeps the original inline Android PiP implementation as the most stable approach currently available for Flutter.

However, Android FlutterActivity limitations still apply:

* Android PiP snapshots the entire Activity surface.
* Because of this, inline PiP on Android may still visually include Flutter UI around the player.

Recommended setup:

```dart
isPipAllowedWhileNotInFullscreen: false
```

and use PiP only from fullscreen mode on Android.

---

### Recommended Android Controller Configuration

```dart
final controller = NativeVideoPlayerController(
  id: _buildPlayerId(widget.params.flowId),
  showNativeControls: false,
  canStartPictureInPictureAutomatically: true,
  isPipAllowedWhileNotInFullscreen: !Platform.isAndroid,
  preferredOrientations: const <DeviceOrientation>[
    DeviceOrientation.portraitUp,
  ],
  mediaInfo: NativeVideoPlayerMediaInfo(
    title: title,
    artworkUrl: imageUrl,
  ),
);
```

---

## Notes

This fork was primarily created to improve:

* Android PiP behavior
* fullscreen orientation handling
* controller disposal stability
* fullscreen + PiP interaction in Flutter apps

while keeping the package compatible with existing APIs.
