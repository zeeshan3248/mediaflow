# MediaFlow

**MediaFlow** is an Android-first, local-media player built in Flutter. It uses a deep violet accent (`#AA7DFF`) on a dark, minimal Material 3 interface and is designed as a polished foundation for a PLAYit-style player without copying its branding or UI.

The project scans the Android MediaStore for device audio and video, plays audio in the background with a media notification, and uses `media_kit` for broad video playback. It intentionally uses content URIs instead of unrestricted file paths, so it works with Android scoped storage.

> **Privacy model:** MediaFlow reads local media metadata and plays media stored on the device. It does not include analytics, advertising, accounts, cloud upload, or a network media catalog.

## Included screens and interaction design

| Screen | Layout and behavior |
|---|---|
| **Library home** | A searchable header and four compact tabs: **Videos**, **Music**, **Folders**, and **Recent**. Video and music appear as artwork-led rows; folders group the indexed content; recent sorts by MediaStore date added. Pull to refresh or use the toolbar refresh control to rescan. |
| **Video player** | Immersive landscape playback with a low-distraction gradient control layer. It includes play/pause, 10-second seek, timeline seek, playback speeds from **0.5×–2×**, subtitle selection/loading, aspect fit/fill/stretch choices, app-level brightness/volume swipes, double-tap seek, pinch zoom, screen lock, and Picture-in-Picture. |
| **Music player** | Large album art, title/artist/album metadata, a precise seek bar, shuffle, previous/next, play/pause, repeat-off/all/one, a favorites affordance, and an up-next queue sheet. |
| **Mini player** | Persistent above the bottom navigation while an audio item is active. It shows progress, item metadata, play/pause, next, and expands into the full music player. |
| **Collections** | A Favorites collection plus persisted custom playlists. Long-press a playlist to delete it; use an item overflow menu to add library items to an existing playlist. |
| **Settings** | Dark, light, or system theme; resume preference; skip-silence preference surface; native Android equalizer hand-off when available; rescan and media permission controls. |

## Feature coverage

| Requirement | Implementation |
|---|---|
| Local video and audio scan | `on_audio_query` reads indexed music; a Kotlin `MethodChannel` performs a MediaStore video query. |
| Common file formats | Android MediaStore indexes local media; `media_kit` is configured for video playback and `just_audio` handles music. Typical MP4, MKV, AVI, MP3, M4A, FLAC, AAC, and WAV files are supported to the extent their codec is available on the device. |
| Background playback | `just_audio_background` supplies a media session and Android notification/lock-screen/headset controls. |
| Picture-in-Picture | Android activity declares PiP support; the video controls invoke `enterPictureInPictureMode`. |
| Resume from last position | Position and duration are stored in `SharedPreferences` for both audio and video and restored when the item is reopened. |
| Subtitle support | Embedded subtitle tracks can be selected or disabled. External **SRT**, **WebVTT**, **ASS**, and **SSA** files can be selected from the system file picker. |
| Search | The home search field filters title, artist, and folder in every library tab. |
| Folders, favorites, playlists | Folder grouping comes from MediaStore metadata; favorites and playlist memberships persist locally. |

## Project structure

```text
mediaflow_player/
├── android/
│   ├── app/
│   │   ├── build.gradle                       # SDK levels and Android app configuration
│   │   └── src/main/
│   │       ├── AndroidManifest.xml            # media, notification, foreground-service, PiP permissions
│   │       └── kotlin/com/mediaflow/player/
│   │           └── MainActivity.kt            # MediaStore, brightness, PiP, and equalizer native bridge
│   ├── gradle/wrapper/                         # included Gradle 8.7 wrapper
│   ├── gradlew / gradlew.bat
│   └── settings.gradle
├── lib/
│   ├── core/app_theme.dart                     # dark violet Material 3 visual system
│   ├── models/media_item.dart                  # unified audio/video model and formatters
│   ├── providers/
│   │   ├── library_provider.dart               # library, favorites, folders, and playlist state
│   │   ├── playback_provider.dart              # background audio queue and controls
│   │   └── settings_provider.dart              # persisted application preferences
│   ├── services/
│   │   ├── media_library_service.dart          # permission requests and MediaStore access
│   │   └── preferences_service.dart            # resume, favorites, playlist persistence
│   ├── screens/
│   │   ├── home_screen.dart                    # Videos/Music/Folders/Recent tabs and search
│   │   ├── video_player_screen.dart            # immersive gesture player
│   │   ├── music_player_screen.dart            # full music UI and queue
│   │   ├── playlists_screen.dart               # favorites and custom playlists
│   │   └── settings_screen.dart                # preferences and media controls
│   ├── widgets/                                # reusable artwork, media rows, mini player, empty state
│   └── main.dart                               # initialization, providers, root navigation
├── pubspec.yaml                                # all package declarations
├── pubspec.lock                                # resolved dependency set
├── analysis_options.yaml
└── README.md
```

## Packages

| Package | Purpose |
|---|---|
| [`media_kit`](https://pub.dev/packages/media_kit), `media_kit_video`, `media_kit_libs_video` | Video rendering, video/audio/subtitle tracks, playback speed, and broad codec support. |
| [`just_audio`](https://pub.dev/packages/just_audio), [`just_audio_background`](https://pub.dev/packages/just_audio_background) | Audio queue, media notification, lock-screen controls, and background playback. |
| [`on_audio_query`](https://pub.dev/packages/on_audio_query) | Indexed song metadata and album artwork from Android MediaStore. |
| [`permission_handler`](https://pub.dev/packages/permission_handler) | Android granular audio/video and notification permission requests. |
| [`file_picker`](https://pub.dev/packages/file_picker) | External subtitle file selection. |
| [`provider`](https://pub.dev/packages/provider) | Lightweight, testable application state. |
| [`shared_preferences`](https://pub.dev/packages/shared_preferences) | Resume points, theme, favorites, and playlists. |

## Android permissions and platform configuration

The app declares the following in `android/app/src/main/AndroidManifest.xml`.

| Declaration | Why it is required |
|---|---|
| `READ_MEDIA_AUDIO` | Read local audio on Android 13+ (API 33+). |
| `READ_MEDIA_VIDEO` | Read local video on Android 13+ (API 33+). |
| `READ_EXTERNAL_STORAGE` (max SDK 32) | Read local media on Android 12 and below. |
| `POST_NOTIFICATIONS` | Show the playback notification on Android 13+. Declining it does not prevent in-app playback. |
| `FOREGROUND_SERVICE` and `FOREGROUND_SERVICE_MEDIA_PLAYBACK` | Keep music playing after the UI leaves the foreground. |
| `WAKE_LOCK` | Keep audio playback stable when the screen locks. |
| `android:supportsPictureInPicture="true"` | Enables Android Picture-in-Picture from the video player. |

The project uses **minSdk 23**, **compileSdk 35**, and **targetSdk 35**. The runtime permission request follows Android 13 granular media permissions; it does not request `MANAGE_EXTERNAL_STORAGE`.

## Build and run

### Prerequisites

1. Install the current [Flutter stable SDK](https://docs.flutter.dev/get-started/install) and run `flutter doctor` until Flutter and the Android toolchain are green.
2. Install Android Studio or Android command-line tools with Android SDK Platform 35, Build Tools, and a suitable emulator/device image.
3. Connect a physical Android device with USB debugging enabled, or start an Android emulator.
4. Use JDK 17 or newer. The included Gradle wrapper is 8.7 and Android Gradle Plugin is 8.5.2.

### Debug build

```bash
cd mediaflow_player
flutter pub get
flutter analyze
flutter devices
flutter run
```

On first run, grant **Music and audio** and/or **Photos and videos** access. Add local media to the emulator/device if its library is empty, then use the refresh button in the Library screen.

### Create an APK

```bash
flutter build apk --debug
# Output: build/app/outputs/flutter-apk/app-debug.apk

# Release APK (uses the debug signing configuration until you add release signing)
flutter build apk --release
# Output: build/app/outputs/flutter-apk/app-release.apk
```

### Create an Android App Bundle for Play distribution

Before publishing, configure a release keystore and the `signingConfigs.release` section in `android/app/build.gradle`, then run:

```bash
flutter build appbundle --release
# Output: build/app/outputs/bundle/release/app-release.aab
```

For Play Store release review, accurately complete the Google Play Data safety form and foreground-service declarations. Do **not** add broad file-system permission unless a release policy review specifically justifies it.

## Important implementation notes

- `MainActivity` extends `AudioServiceActivity`, which is required for the background audio service and media-session integration.
- The video scanner returns `content://` URIs rather than raw paths. This is deliberate: it works with scoped storage and avoids broad storage access.
- App-level brightness changes from the left-hand video swipe. Right-hand vertical swipes change video volume. On devices where system media volume is controlled separately, this remains scoped to video playback.
- Codec support ultimately depends on the device/runtime decoder. Test target devices with representative MKV/AVI, subtitle, and high-bitrate files before production release.
- `skip silence` is persisted as a playback preference. Turning it into a destructive audio transformation is intentionally avoided; apply it only if the selected audio engine/device supports it.

## Validation performed

`flutter pub get` completed successfully and `flutter analyze` completed with **No issues found** using Flutter 3.24.5 / Dart 3.5.4. A debug APK was not produced in this environment because it does not have an Android SDK installed; the source includes the Gradle wrapper and Android project configuration needed to build on a machine with the Android toolchain.
