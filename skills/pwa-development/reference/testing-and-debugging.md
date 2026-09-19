# Testing & Debugging

## DevTools Debugging Checklist

1. **DevTools > Application > Manifest** — Valid? All fields present?
2. **DevTools > Application > Service Workers** — Registered? Status active?
3. **DevTools > Application > Cache Storage** — What's cached? Expected entries?
4. **DevTools > Network > Offline** — Works offline?
5. **DevTools > Lighthouse > PWA** — Score and failures?
6. **Real device for each platform you support** — Test on actual Android and iOS hardware, not just desktop emulation

## Lighthouse CLI

```bash
# Run Lighthouse from CLI
npx lighthouse https://your-app.com --view

# Key metrics to check:
# - PWA badge (installable, offline-ready)
# - Performance score
# - Best practices
# - Accessibility
```

## Manual JS Test Snippets

### Check if Installed

```javascript
window.matchMedia('(display-mode: standalone)').matches
```

### List Service Worker Registrations

```javascript
navigator.serviceWorker.getRegistrations()
  .then(regs => console.log('SW:', regs));
```

### List All Caches

```javascript
caches.keys().then(names => console.log('Caches:', names));
```

### Force Unregister All Service Workers

```javascript
navigator.serviceWorker.getRegistrations()
  .then(regs => regs.forEach(r => r.unregister()));
```

### Clear All Caches

```javascript
caches.keys().then(names => names.forEach(n => caches.delete(n)));
```

## Performance Targets

| Metric | Target |
|--------|--------|
| First Contentful Paint (FCP) | < 1.8s |
| Largest Contentful Paint (LCP) | < 2.5s |
| Time to Interactive (TTI) | < 3.8s |
| Cumulative Layout Shift (CLS) | < 0.1 |

## Service Worker Update Testing

```javascript
// Force update check
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.ready.then((registration) => {
    registration.update();
  });
}

// Listen for controller change (new SW activated)
navigator.serviceWorker.addEventListener('controllerchange', () => {
  // New service worker activated
  window.location.reload();
});
```

## Manual Testing Checklist

### Installability
- [ ] Install prompt appears on desktop Chrome
- [ ] Can be added to home screen on mobile
- [ ] App opens in standalone mode after install

### Offline Support
- [ ] App loads when offline (airplane mode)
- [ ] Cached pages display correctly
- [ ] Offline fallback page shows for uncached routes
- [ ] Background sync works when coming back online

### Service Worker
- [ ] SW registers successfully
- [ ] Static assets cached on install
- [ ] SW updates correctly (new version)
- [ ] No stale cache issues

### Manifest
- [ ] All required fields present
- [ ] Icons display correctly on home screen
- [ ] Theme color applied to browser chrome
- [ ] Splash screen shows on launch (Android)

### Cross-Platform
- [ ] Test on Android Chrome (install, offline, notifications)
- [ ] Test on iOS Safari (Add to Home Screen, offline, safe areas)
- [ ] Test on desktop Chrome/Edge (install, offline)
