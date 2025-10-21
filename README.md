# dragdrop

A small SvelteKit app that demonstrates drag & drop UI patterns and includes Progressive Web App (PWA) support.

## Features

- Built with SvelteKit.
- Drag & drop UI for organizing items.
- PWA-ready: includes a web app manifest and a service worker to enable offline caching and installation.
- Minimal, client-first architecture (static assets served from `static/`).

## Project layout (key files)

- src/app.html — HTML template (includes manifest & theme-color).
- src/routes/+page.svelte — app entry page (may contain service worker registration).
- static/manifest.json — web app manifest (icons, name, theme).
- static/service-worker.js — service worker for caching and offline support.
- package.json — scripts for dev/build/preview.

## Getting started

1. Install dependencies

   npm install

2. Run development server

   npm run dev

3. Build for production

   npm run build

4. Preview production build

   npm run preview

## PWA notes

- Manifest: static/manifest.json — confirm icons and metadata are correct.
- Service worker: static/service-worker.js — register/replace as needed. After building, verify Service Worker and Manifest in browser DevTools → Application.
- To test offline behavior: build the app, open in a secure context (localhost or HTTPS), then in DevTools disable network and reload.

## Contributing

- Report issues or open pull requests.
- Keep changes small and focused; run the dev server to verify UI and PWA behavior.

## License

Add a license (e.g., MIT) or other terms here.
