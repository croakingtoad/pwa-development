# Framework Integration

## Next.js

```bash
npm install next-pwa
```

```javascript
// next.config.js
const withPWA = require('next-pwa')({
  dest: 'public',
  disable: process.env.NODE_ENV === 'development'
});

module.exports = withPWA({
  // Your Next.js config
});
```

This generates a service worker in `public/` at build time. Place your
`manifest.json` in the `public/` directory and add the `<link rel="manifest">`
tag to your `_document.js` or root layout.

## Create React App

```bash
npx create-react-app my-pwa --template cra-template-pwa
```

CRA 4+ includes PWA support via Workbox out of the box with this template. The
generated `src/service-worker.js` uses `workbox-precaching` and can be extended
with custom routes.

To enable the service worker, change `serviceWorkerRegistration.unregister()`
to `serviceWorkerRegistration.register()` in `src/index.js`.

## Vite (Any Framework)

```bash
npm install vite-plugin-pwa -D
```

See `workbox-and-caching.md` for the full `VitePWA({...})` configuration
including manifest and runtime caching rules.

The plugin handles:
- Generating the service worker from Workbox config
- Injecting the manifest link into HTML
- Auto-registration or prompt-based registration (`registerType: 'autoUpdate'` vs `'prompt'`)

<!-- NEW: not in original source repos -->
### Update Prompt (React + Vite)

```typescript
import { useRegisterSW } from 'virtual:pwa-register/react';

function App() {
  const { needRefresh: [needRefresh], updateServiceWorker } = useRegisterSW();
  return needRefresh && (
    <button onClick={() => updateServiceWorker(true)}>Update available</button>
  );
}
```

<!-- NEW: not in original source repos -->
## Webpack

```bash
npm install -D workbox-webpack-plugin
```

```javascript
// webpack.config.js
const { GenerateSW } = require('workbox-webpack-plugin');

module.exports = {
  plugins: [
    new GenerateSW({
      clientsClaim: true,
      skipWaiting: true,
      runtimeCaching: [
        {
          urlPattern: /^https:\/\/api\./,
          handler: 'NetworkFirst'
        },
        {
          urlPattern: /\.(?:png|jpg|jpeg|svg)$/,
          handler: 'CacheFirst',
          options: {
            cacheName: 'images',
            expiration: { maxEntries: 50 }
          }
        }
      ]
    })
  ]
};
```

<!-- NEW: not in original source repos -->
## Nuxt 3

```bash
npm install -D @vite-pwa/nuxt
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@vite-pwa/nuxt'],
  pwa: {
    manifest: {
      name: 'My App',
      short_name: 'App',
      theme_color: '#000000'
    }
  }
});
```

<!-- NEW: not in original source repos -->
## SvelteKit

```bash
npm i -D @vite-pwa/sveltekit
```

```javascript
// svelte.config.js
import { sveltekit } from '@sveltejs/kit/vite';
import { SvelteKitPWA } from '@vite-pwa/sveltekit';

/** @type {import('vite').UserConfig} */
const config = {
  plugins: [
    sveltekit(),
    SvelteKitPWA({
      registerType: 'autoUpdate',
      manifest: {
        name: 'My SvelteKit App',
        short_name: 'MyApp',
        theme_color: '#ffffff',
        icons: [
          { src: 'pwa-192x192.png', sizes: '192x192', type: 'image/png' },
          { src: 'pwa-512x512.png', sizes: '512x512', type: 'image/png' }
        ]
      },
      workbox: {
        globPatterns: ['client/**/*.{js,css,html,ico,png,svg,webp}']
      }
    })
  ]
};

export default config;
```

`@vite-pwa/sveltekit` is built on the same `vite-plugin-pwa` core — the
Workbox runtime caching config from `workbox-and-caching.md` applies here too.

## General Notes

- **Static export** (`output: 'export'` in Next.js, `adapter-static` in
  SvelteKit): works well with PWA since all assets are static and predictable
  for precaching.

- **SSR apps**: the service worker only caches client-side assets and navigation
  responses. Server-rendered HTML can be cached with a network-first strategy
  so offline fallback works.

- **Manifest location**: most frameworks expect `manifest.json` or
  `manifest.webmanifest` in the public/static directory. The `<link rel="manifest">`
  tag should be in the HTML `<head>`.

<!-- NEW: not in original source repos -->
## Icon Generation

```bash
npm install -D @vite-pwa/assets-generator
```

```typescript
// pwa-assets.config.ts
import { defineConfig, minimal2023Preset } from '@vite-pwa/assets-generator/config';

export default defineConfig({
  preset: minimal2023Preset,
  images: ['public/logo.svg']
});
```

```bash
npx pwa-assets-generator
```

Generates all required icon sizes from a single SVG source. See
`manifest-advanced.md` for the full icon size matrix.

<!-- NEW: not in original source repos -->
## Offline Fallback Page

Configure Workbox to serve a fallback for failed navigations:

```javascript
workbox: {
  navigateFallback: '/offline.html',
  navigateFallbackDenylist: [/^\/api/]
}
```

Create `public/offline.html` with a simple "You're offline" message and a
retry button. Precache it so it's always available.
