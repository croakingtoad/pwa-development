# Platform Quirks — iOS Safari & Android Chrome

Side-by-side reference for platform-specific behavior. The core PWA APIs (service
workers, Cache API, manifest) are the same everywhere — this file covers where
platforms diverge.

---

## iOS Safari

### Status Bar Styles

Set via `<meta name="apple-mobile-web-app-status-bar-style">`:

| Value | Effect |
|-------|--------|
| `default` | White bar, black text |
| `black` | Black bar, white text |
| `black-translucent` | Transparent bar, content flows behind it (use with `viewport-fit=cover` and safe-area insets) |

### Apple-Specific Meta Tags

```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="App Name">
<link rel="apple-touch-icon" href="/apple-touch-icon-180x180.png">
```

The `apple-touch-icon` should be 180x180px. Unlike the manifest icons, iOS
ignores the manifest's icon list for the home screen icon and uses this link
tag instead.

### Known Gaps and Limitations

- **No `beforeinstallprompt`**: iOS does not fire the `beforeinstallprompt`
  event. Users install via Safari's Share > "Add to Home Screen" menu. You
  cannot programmatically trigger or detect the install prompt — show manual
  instructions instead.

- **Push notifications**: Supported from iOS/iPadOS 16.4+ (March 2023) for
  home-screen-installed PWAs only. Requires user to add the app to their home
  screen first. Not supported in Safari browser tabs.

- **Storage eviction**: iOS can evict Service Worker caches and IndexedDB data
  after roughly 7 days of inactivity (no user visit). Plan for data loss —
  treat local storage as a cache, not a database. Critical data should sync
  to a server.

- **No background sync**: The Background Sync API is not supported on iOS.
  Offline-queued actions need a different strategy — retry on the next
  foreground visit.

- **Scope limitations**: iOS ignores the manifest `scope` field in some
  contexts. Navigation outside the start_url's origin opens in Safari rather
  than staying in the PWA shell.

- **Audio/video**: Autoplay is restricted. Background audio stops when the
  PWA is backgrounded.

---

## Android Chrome / WebView

### Install Prompt (`beforeinstallprompt`)

Android is the primary platform for PWA installation UX. Chrome fires the
`beforeinstallprompt` event when the installability criteria are met, allowing
you to show a custom install button. See the install prompt code in `SKILL.md`.

The mini-infobar (Android's default install banner) appears automatically.
Call `e.preventDefault()` in the `beforeinstallprompt` handler to suppress it
and show your own UI instead.

### WebAPK

When a user installs a PWA on Android via Chrome, the browser generates a
**WebAPK** — a real Android APK wrapper that gives the PWA a presence in the
app drawer, settings, and recent apps, just like a native app. This happens
transparently; no action needed from the developer beyond meeting
installability criteria.

WebAPK updates are handled automatically by Chrome when it detects manifest
changes.

### Notification Channels (Android 8+)

On Android 8.0 (Oreo) and above, notifications are organized into **channels**.
PWA push notifications go into a default channel created by the browser.
Users can control notification behavior per-channel in Android Settings.

The `showNotification()` options (`badge`, `icon`, `vibrate`, `actions`) are
well-supported on Android. `badge` is particularly useful — it sets the small
monochrome icon in the status bar.

### Auto-Generated Splash Screen

Android generates a splash screen automatically from the manifest's
`background_color`, `theme_color`, `name`, and the largest icon. You cannot
customize the splash layout — only these four inputs. Ensure:

- `background_color` is set and matches your app's initial background
- The 512x512 icon looks good centered on that background
- `name` is the text shown below the icon on the splash

### Maskable / Adaptive Icons

Android's adaptive icon system clips icons into the device's preferred shape
(circle, squircle, rounded square). Use `"purpose": "maskable"` in the manifest
to provide an icon that accounts for this clipping. See `manifest-advanced.md`
for safe-zone details.

### Display Cutout (Notch / Punch-Hole)

Android devices with notches or camera punch-holes use the same
`env(safe-area-inset-*)` CSS API as iOS. With `viewport-fit=cover`, content
extends into the cutout area and you control padding with the safe-area
variables. See `mobile-native-ux.md` for the CSS patterns.

### Back Button / History

Android's system back button (or gesture swipe) pops the browser history stack.
In standalone mode, an empty history stack exits the app. See
`mobile-native-ux.md` for the `popstate` handling pattern.

### TWA (Trusted Web Activity)

For Play Store distribution, a PWA can be wrapped as a Trusted Web Activity.
This is a packaging/distribution concern beyond this skill's scope — it uses
the same underlying PWA but runs inside a thin native shell verified via
Digital Asset Links.
