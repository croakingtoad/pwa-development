---
name: pwa-development
description: >
  Implement Progressive Web App features including service workers, caching
  strategies, offline support, and installation prompts. Use when the user asks
  to "make a PWA", "add offline support", "create a service worker", "fix
  caching issues", or wants installable web apps. Keywords: PWA, service worker,
  manifest, offline, installable, add to home screen, push notifications,
  Android, iOS.
license: MIT
paths: ["**/sw.*", "**/service-worker.*", "**/workbox-config.*", "**/manifest.json", "**/manifest.webmanifest"]
---

# PWA Development

Build Progressive Web Apps that work offline, install like native apps, and
deliver fast, reliable experiences across all devices.

## Core Principle

**PWAs fail when offline behavior is an afterthought.** A PWA is not "add
service worker to existing app." It's a fundamental architectural decision
about data flow, caching, and connectivity failure.

Default to balanced Android/iOS coverage; only weight toward one platform if
the user says to.

## The Three Pillars

1. **HTTPS** — Required for service workers. localhost for dev.
2. **Service Worker** — Background JS enabling offline, caching, push.
3. **Web App Manifest** — JSON metadata enabling installation.

**Installability (Chrome):** HTTPS + SW with fetch handler + manifest with
`name`, `icons` (192 + 512px), `start_url`, `display` (standalone/fullscreen/minimal-ui).

## Diagnostic States

Use these to identify where an app stands and what to do next.

### P0: No PWA Setup

**Symptoms:** No manifest.json, no service worker, online-only

**Interventions:**
- Create manifest (see below; `reference/manifest-advanced.md` for full features)
- Add essential HTML head tags (below)
- Add basic service worker (below)

### P1: Basic Manifest Only

**Symptoms:** Manifest exists but SW missing, breaks offline

**Interventions:**
- Implement basic service worker (below)
- Choose caching strategies per decision table (below)
- Add offline fallback page
- See `reference/workbox-and-caching.md` for Workbox setup

### P2: Caching Issues

**Symptoms:** Stale content, unexpected caching behavior

**Interventions:**
- Map resources to strategies (decision table below)
- Add cache expiration and cleanup
- See `reference/workbox-and-caching.md`

### P3: Update Problems

**Symptoms:** Users stuck on old versions, multiple refreshes needed

**Interventions:**
- Implement skipWaiting/clients.claim with update notification UI
- See `reference/testing-and-debugging.md` for update-testing pattern

### P4: Offline Data Gaps

**Symptoms:** User actions lost offline, no sync indicator

**Interventions:**
- Implement IndexedDB for offline storage + Background Sync API
- See `reference/workbox-and-caching.md` (background sync section)

### P5: Platform-Specific Issues

**Symptoms:** Works on one platform, breaks on the other (Android or iOS)

**Interventions:**
- Review `reference/platform-quirks.md` for iOS Safari and Android Chrome specifics
- Apply `reference/mobile-native-ux.md` for safe areas and touch targets
- Handle storage eviction gracefully on iOS
- Handle back-button/history navigation on Android

### P6: Production Ready

**Indicators:** Lighthouse PWA 100, works offline, updates cleanly

**Validation:**
- Run the audit checklist in `reference/testing-and-debugging.md`

## Essential HTML Head

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<meta name="theme-color" content="#000000">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="App Name">
<link rel="manifest" href="/manifest.webmanifest">
<link rel="apple-touch-icon" href="/apple-touch-icon-180x180.png">
```

## Minimal Manifest

```json
{
  "name": "My Progressive Web App",
  "short_name": "MyPWA",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icons/icon-512-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

See `reference/manifest-advanced.md` for screenshots, shortcuts, share_target, and more.

## Basic Service Worker

```javascript
const CACHE_NAME = 'app-cache-v1';
const STATIC_ASSETS = ['/', '/index.html', '/styles/main.css', '/scripts/app.js', '/offline.html'];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(STATIC_ASSETS)).then(() => self.skipWaiting())
  );
});

self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((keys) => Promise.all(
      keys.filter((key) => key !== CACHE_NAME).map((key) => caches.delete(key))
    )).then(() => self.clients.claim())
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => cached || fetch(event.request))
      .catch(() => caches.match('/offline.html'))
  );
});
```

### Registration

```javascript
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js', { scope: '/' });
}
```

## Caching Strategy Decision Table

| Strategy | Use Case | Behavior |
|----------|----------|----------|
| **Cache First** | Static assets (CSS, JS, images, fonts) | Cache → network fallback |
| **Network First** | API responses, dynamic content | Network → cache fallback |
| **Stale While Revalidate** | Semi-static (avatars, articles) | Serve cache, update in background |
| **Network Only** | Auth, real-time data, analytics | No caching |
| **Cache Only** | Versioned/immutable assets | Cache only, never updates |

Per-strategy code: `reference/workbox-and-caching.md`.

## Install Prompt

```javascript
let deferredPrompt;

window.addEventListener('beforeinstallprompt', (e) => {
  e.preventDefault();
  deferredPrompt = e;
  showInstallButton();
});

async function installApp() {
  if (!deferredPrompt) return;
  deferredPrompt.prompt();
  const { outcome } = await deferredPrompt.userChoice;
  deferredPrompt = null;
  hideInstallButton();
}

window.addEventListener('appinstalled', () => { deferredPrompt = null; });
```

## Launch Checklist

### Required for Installation

- [ ] HTTPS (localhost allowed for dev)
- [ ] Valid manifest with `name`, `icons`, `start_url`, `display`
- [ ] 192x192 PNG icon
- [ ] 512x512 PNG icon
- [ ] Service worker with fetch handler

### Recommended

- [ ] `viewport-fit=cover` meta tag
- [ ] Safe area inset handling
- [ ] `theme_color` in manifest and meta tag
- [ ] Maskable icon (512x512 with 20% safe zone)
- [ ] Apple touch icon (180x180)
- [ ] `apple-mobile-web-app-status-bar-style` meta tag
- [ ] Offline fallback page
- [ ] Install prompt UI
- [ ] Cache strategies defined for all resource types
- [ ] Lighthouse PWA audit passes

## Reference Files

Detailed content for specific tasks — only pull in what you need:

| File | When to use |
|------|-------------|
| `reference/manifest-advanced.md` | Rich manifest fields (screenshots, shortcuts, share_target), full icon matrix |
| `reference/workbox-and-caching.md` | Workbox config, per-strategy code, background sync |
| `reference/mobile-native-ux.md` | Safe areas, touch targets, pull-to-refresh, native feel |
| `reference/platform-quirks.md` | iOS Safari and Android Chrome specifics, side by side |
| `reference/push-notifications.md` | Subscribe/server/show-notification flow |
| `reference/framework-integration.md` | Next.js, CRA, Vite, SvelteKit setup |
| `reference/testing-and-debugging.md` | Lighthouse, DevTools checklist, manual test snippets, performance targets |
| `reference/anti-patterns.md` | Common mistakes and named anti-patterns |
