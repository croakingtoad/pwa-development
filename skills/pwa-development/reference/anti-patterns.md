# Anti-Patterns & Common Mistakes

## Named Anti-Patterns

### The Everything Cache

Precaching every asset — massive initial download, wasted bandwidth.

**Fix:** Precache only the critical app shell (HTML, core CSS/JS, offline page).
Use runtime caching for everything else, with appropriate strategies per
resource type.

### The Immortal Cache

Never expiring caches — stale content forever, growing storage usage.

**Fix:** Cache versioning with `CACHE_NAME` increments, delete old caches on
activate, set `maxEntries` and `maxAgeSeconds` (via Workbox `ExpirationPlugin`
or manual cleanup).

### The Silent Update

Forcing updates via `skipWaiting()` + `clients.claim()` without telling the
user — content changes mid-session, state is lost, UI breaks.

**Fix:** Notify the user that an update is available. Let them choose when to
refresh. Show a toast/banner with a "Refresh" button rather than auto-reloading.

### The Single-Platform Afterthought

Building for one platform, testing the other last. Both Android and iOS have
meaningful differences in PWA support (see `platform-quirks.md`).

**Fix:** Test on both platforms early in development. Accept each platform's
limitations and design for them rather than discovering breakage late.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Missing maskable icon | Add a separate icon with `"purpose": "maskable"` and 20% safe zone |
| No offline fallback | Create `offline.html`, precache it, serve it on navigation failure |
| SW caches too aggressively | Use appropriate strategies per resource type — not everything is Cache First |
| Broken install prompt | Ensure manifest meets all installability criteria (name, icons, start_url, display, SW with fetch) |
| No HTTPS in production | Configure SSL certificate — required for service workers |
| Large cache size | Set `maxEntries` and `maxAgeSeconds` to bound cache growth |
| Stale API responses | Use Network First for dynamic data, not Cache First |
| Missing `start_url` tracking | Add query param: `/?source=pwa` to distinguish installed-app visits |
| No update mechanism | Implement update detection + reload prompt (see `testing-and-debugging.md`) |
| Combining `any` and `maskable` icon purpose | Use `"purpose": "maskable"` on a separate asset — combining them forces the any icon into the maskable safe zone, making it too small |

## Deduplication Notes

"Cache never expires" (common mistake) and "The Immortal Cache" (named
anti-pattern) are the same issue. The fix: use Workbox `ExpirationPlugin` with
`maxEntries` and `maxAgeSeconds`, or manually version caches and clean up old
ones in the `activate` event.

"SW caches too aggressively" and "The Everything Cache" are the same issue.
The fix: precache only the app shell, runtime-cache everything else with the
right strategy.
