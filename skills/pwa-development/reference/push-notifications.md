# Push Notifications

Android is the primary, fully-supported platform for PWA push notifications.
iOS 16.4+ supports push for home-screen-installed PWAs only — see
`platform-quirks.md` for iOS limitations.

The code below uses the standard Push API and works on both platforms where
supported.

## Request Permission

```javascript
async function requestNotificationPermission() {
  const permission = await Notification.requestPermission();
  if (permission === 'granted') {
    await subscribeToPush();
  }
  return permission;
}
```

Best practice: don't request permission on page load. Wait until the user takes
an action that makes notifications relevant (e.g., enabling alerts for a
specific event), then explain what they'll receive before prompting.

## Subscribe to Push

```javascript
async function subscribeToPush() {
  const registration = await navigator.serviceWorker.ready;
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY)
  });

  // Send subscription to server
  await fetch('/api/push/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(subscription)
  });
}
```

### VAPID Key Helper

```javascript
function urlBase64ToUint8Array(base64String) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/-/g, '+')
    .replace(/_/g, '/');
  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);
  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}
```

## Handle Push Events (Service Worker)

```javascript
// sw.js
self.addEventListener('push', (event) => {
  const data = event.data.json();
  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: '/icons/icon-192.png',
      badge: '/icons/badge-72.png',
      data: { url: data.url }
    })
  );
});
```

### Notification Options

| Option | Description |
|--------|-------------|
| `body` | Main notification text |
| `icon` | Large icon (192px recommended) |
| `badge` | Small monochrome status-bar icon (Android, 72px) |
| `image` | Large image shown in expanded notification |
| `vibrate` | Vibration pattern, e.g. `[200, 100, 200]` |
| `actions` | Up to 2 action buttons with `action` and `title` |
| `tag` | Group/replace notifications with the same tag |
| `renotify` | Re-alert for updated notification with same tag |
| `data` | Arbitrary data passed to notification click handler |

### Android Notification Channels

On Android 8+, notifications are organized into channels. The browser creates
a default channel for PWA notifications. Users can control notification behavior
per-channel in Android Settings > Apps > [Your App] > Notifications.

## Handle Notification Click

```javascript
// sw.js
self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  event.waitUntil(
    clients.openWindow(event.notification.data.url)
  );
});
```

### Handle Action Buttons

```javascript
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  if (event.action === 'reply') {
    event.waitUntil(clients.openWindow('/reply'));
  } else if (event.action === 'dismiss') {
    // Just close, already handled above
  } else {
    // Default click (not an action button)
    event.waitUntil(clients.openWindow(event.notification.data.url));
  }
});
```
