# ForkPlayer PWA — Alpha 0.2 compatibility audit (2026-10-22)

## Confirmed baseline
- Alpha 0.1 on GitHub Pages: iPhone Safari interface and direct HTTPS MP4 playback work, including rotation/fullscreen (device observation).
- Preserve the existing `pwa/` player and deployment until a separate Alpha 0.2 candidate passes device testing.

## Source trace (branch pwa-prototype)
- `app/src/main/java/aforkplayer/aforkplayer/FullscreenActivity.kt`: Android WebView loads `file:///android_asset/index.html` and injects JavaScript object `andr`.
- `app/src/main/assets/index.html`: loads local `js/load.js` and CSS; has no standalone application implementation.
- `app/src/main/assets/js/load.js`: fetches remote `getsettings.js` from two HTTP proxies and a GitHub raw HTTPS fallback, then loads further remote JavaScript. Also uses localStorage to cache settings/code.
- Android bridge handles `popUP`, `curl`, `waitcurl`/`asyncurl`, `exit`, `video`, and local filesystem/remotefo operations. Video dispatch can open MX Player or Vimu via Android Intent.

## Browser compatibility / security blockers
1. Browser cannot expose the Android `andr` object. Feature calls must be inventoried and replaced with explicit, minimal browser APIs; do not expose an unrestricted command bridge.
2. HTTP script/settings sources are mixed content under HTTPS Pages; browser CORS and content security constraints may block remote requests.
3. Dynamically loaded remote JavaScript is a code-execution trust boundary. Do not import or execute it automatically in the PWA. First review license, integrity, provenance, and update mechanism.
4. Android external player Intents, arbitrary local file reads, and arbitrary HTTP proxy requests cannot be mapped directly to Safari.
5. Media compatibility: Safari supports a subset of browser-playable formats; direct HTTPS MP4 is verified; HLS and playlist flows remain unverified.

## Next bounded implementation
- Build an isolated **local** navigation/catalog UI in `pwa/` with a static test entry and direct media URLs, using the already verified HTML5 video element.
- Add an explicit compatibility adapter for safe browser-only actions, with unsupported legacy commands reported rather than silently executed.
- Audit original UI JS and external dependency licensing before any reuse or publication of that code.
- Keep no accounts, analytics, or server-side user data. No unnecessary build/test loops.
- After local navigation works, test HLS separately and then explore vetted catalog integration with browser-safe network access.

## Not claimed
This is **not** a working full ForkPlayer catalog, nor evidence that remote ForkPlayer servers can be accessed from iOS.
