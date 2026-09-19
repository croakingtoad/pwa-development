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
