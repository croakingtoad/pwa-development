# Mobile-Native UX

Make a PWA feel like a native app rather than a website in a wrapper.

## Safe Area Handling

**Required:** `viewport-fit=cover` in the viewport meta tag.

`env(safe-area-inset-*)` is a standard CSS API — it covers notches, Dynamic
Island, and rounded corners on iOS, **and** display cutouts (notches, punch-holes)
and gesture-navigation bars on Android. The CSS below is cross-platform.

```css
:root {
  --safe-top: env(safe-area-inset-top, 0px);
  --safe-right: env(safe-area-inset-right, 0px);
  --safe-bottom: env(safe-area-inset-bottom, 0px);
  --safe-left: env(safe-area-inset-left, 0px);
}

body {
  padding: var(--safe-top) var(--safe-right) var(--safe-bottom) var(--safe-left);
}

/* Fixed header */
.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  padding: calc(1rem + var(--safe-top)) calc(1rem + var(--safe-right)) 1rem calc(1rem + var(--safe-left));
}

/* Fixed bottom navigation */
.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 0.5rem var(--safe-right) calc(0.5rem + var(--safe-bottom)) var(--safe-left);
}

/* Landscape notch handling */
@media (orientation: landscape) {
  .content {
    padding-left: max(1rem, var(--safe-left));
    padding-right: max(1rem, var(--safe-right));
  }
}
```

## Display Mode Detection

```css
@media (display-mode: standalone) {
  .browser-nav { display: none; }
}
```

```javascript
const isInstalled = window.matchMedia('(display-mode: standalone)').matches
  || window.navigator.standalone; // iOS Safari fallback

// Listen for display mode changes
window.matchMedia('(display-mode: standalone)')
  .addEventListener('change', (e) => {
    console.log('Display mode:', e.matches ? 'standalone' : 'browser');
  });
```

## Touch Targets

```css
/* Apple HIG: minimum 44x44px */
/* Material Design: 48x48dp (Android recommendation) */
/* Use at least 44px for cross-platform, 48px if Android-primary */
button, a, [role="button"] {
  min-width: 44px;
  min-height: 44px;
}
```

## Disable Pull-to-Refresh

```css
html {
  overscroll-behavior-y: contain;
}
```

## Native-Like Touch Feedback

```css
button, a {
  -webkit-tap-highlight-color: transparent;
  touch-action: manipulation; /* Disable double-tap zoom */
}

/* Disable text selection on UI chrome (nav, toolbars) */
.nav, .toolbar {
  -webkit-user-select: none;
  user-select: none;
}

/* Disable callout on long press (images, links) */
img, a {
  -webkit-touch-callout: none;
}
```

## Smooth Scrolling

```css
.scroll-container {
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  overscroll-behavior: contain;
}
```

## Prevent iOS Input Zoom (iOS only)

iOS Safari zooms in on form inputs with font-size below 16px. Android does not
have this behavior.

```css
input, select, textarea {
  font-size: 16px;
  -webkit-appearance: none; /* Remove iOS default styling */
  border-radius: 0; /* Remove iOS rounded inputs */
}
```

## Android Back Button / Gesture Navigation

Android PWAs running in standalone mode rely on the browser's history stack for
back navigation. In a SPA, pressing the system back button or swiping back
will pop the browser history — if the stack is empty, it exits the app entirely.

Handle this by ensuring your router pushes real history entries and intercepting
the back gesture:

```javascript
// Push state for SPA navigation so back-button has somewhere to go
window.addEventListener('popstate', (event) => {
  // Handle back navigation within your app
  // e.g., close a modal, go to previous view, etc.
  if (isModalOpen()) {
    closeModal();
    // Re-push state so back doesn't exit the app
    history.pushState(null, '', window.location.href);
  }
});

// On app load, push an initial state so the first back press
// doesn't immediately exit
if (window.matchMedia('(display-mode: standalone)').matches) {
  history.pushState(null, '', window.location.href);
}
```

<!-- NEW: not in original source repos -->
## Window Controls Overlay (Desktop PWA)

On desktop, PWAs can use the title bar area for app content:

```json
{
  "display_override": ["window-controls-overlay"],
  "display": "standalone"
}
```

```css
.titlebar {
  position: fixed;
  top: 0;
  left: env(titlebar-area-x, 0);
  width: env(titlebar-area-width, 100%);
  height: env(titlebar-area-height, 40px);
  -webkit-app-region: drag;
}

.titlebar button {
  -webkit-app-region: no-drag;
}
```
