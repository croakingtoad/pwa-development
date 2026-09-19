# Advanced Manifest Configuration

## Enhanced Manifest (Full Features)

```json
{
  "name": "My Progressive Web App",
  "short_name": "MyPWA",
  "description": "A full-featured PWA",
  "start_url": "/?source=pwa",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait-primary",
  "background_color": "#ffffff",
  "theme_color": "#3367D6",
  "dir": "ltr",
  "lang": "en",
  "categories": ["productivity", "utilities"],

  "icons": [
    { "src": "/icons/icon-72.png", "sizes": "72x72", "type": "image/png" },
    { "src": "/icons/icon-96.png", "sizes": "96x96", "type": "image/png" },
    { "src": "/icons/icon-128.png", "sizes": "128x128", "type": "image/png" },
    { "src": "/icons/icon-144.png", "sizes": "144x144", "type": "image/png" },
    { "src": "/icons/icon-152.png", "sizes": "152x152", "type": "image/png" },
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-384.png", "sizes": "384x384", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icons/icon-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ],

  "screenshots": [
    {
      "src": "/screenshots/desktop.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    },
    {
      "src": "/screenshots/mobile.png",
      "sizes": "750x1334",
      "type": "image/png",
      "form_factor": "narrow"
    }
  ],

  "shortcuts": [
    {
      "name": "New Item",
      "short_name": "New",
      "description": "Create a new item",
      "url": "/new?source=shortcut",
      "icons": [{ "src": "/icons/shortcut-new.png", "sizes": "192x192" }]
    }
  ],

  "share_target": {
    "action": "/share",
    "method": "POST",
    "enctype": "multipart/form-data",
    "params": {
      "title": "title",
      "text": "text",
      "url": "url",
      "files": [{ "name": "files", "accept": ["image/*"] }]
    }
  },

  "protocol_handlers": [
    {
      "protocol": "web+myapp",
      "url": "/handle?url=%s"
    }
  ],

  "file_handlers": [
    {
      "action": "/open-file",
      "accept": {
        "text/plain": [".txt"]
      }
    }
  ]
}
```

## Icon Size Matrix

| Size | Purpose |
|------|---------|
| 72x72 | Older Android devices |
| 96x96 | Android notification icon |
| 128x128 | Chrome Web Store |
| 144x144 | IE/Edge pinned site |
| 152x152 | iPad (iOS) |
| 180x180 | Apple touch icon (`<link rel="apple-touch-icon">`) |
| 192x192 | Android home screen (required for installability) |
| 384x384 | Android splash screen (2x) |
| 512x512 | Android splash screen (required for installability) |
| 512x512 maskable | Android adaptive icon |

### Maskable Icons

Icons with `"purpose": "maskable"` are used by Android for adaptive icon shapes
(circles, squircles, rounded squares). The icon content must stay within the
center 80% of the image — the outer 20% is the **safe zone** that may be clipped
by the OS. Place your logo/artwork within this inner area, and fill the safe zone
with background color.

A maskable icon should be a separate asset from your `"purpose": "any"` icon.
Do not combine purposes (`"purpose": "any maskable"`) — this forces the
any-purpose icon into the maskable safe zone, making it appear too small on
platforms that use the any-purpose version.

## Display Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `standalone` | Native app look, no browser UI | Most apps (recommended) |
| `fullscreen` | Entire screen, no status bar | Games, immersive, VR/AR |
| `minimal-ui` | Minimal browser controls | Content needing browser navigation |
| `browser` | Standard browser tab | Not recommended for PWAs |

## Manifest Field Checklist

- [ ] `name` and `short_name` defined
- [ ] `start_url` set (use query param for analytics, e.g. `/?source=pwa`)
- [ ] `display` set to `standalone` or `fullscreen`
- [ ] Icons: 192x192 and 512x512 minimum
- [ ] Maskable icon included for Android adaptive icons (separate asset, 20% safe zone)
- [ ] `theme_color` matches app design
- [ ] `background_color` for splash screen
- [ ] `scope` set to control navigation boundary
- [ ] Screenshots for richer install UI (optional, `form_factor`: `wide` and `narrow`)
- [ ] Shortcuts for quick actions (optional)
- [ ] `categories` for app store classification (optional)
- [ ] `dir` and `lang` for localization (optional)
