# OWBR

A Chromium-based desktop browser, built on Electron (Chromium + Node).
Working name: **OWBR** (rename later in `package.json` and `src/renderer/*`).

## Features

**Window** — frameless, no OS menu bar; custom minimize/maximize/close controls
built into the tab strip (native traffic lights on macOS).

**Browsing**
- Multi-tab: favicons, spinners, audio-muted badge, middle-click close, `Ctrl+1…9`
- Omnibox with live suggestions from history and bookmarks, arrow-key selection
- Back / Forward / Reload+Stop / Home, per-site zoom, find-in-page
- Custom error page, view-source, print, save page as

**Settings** (`owbr://settings`, `Ctrl+,`)
- Startup behaviour: new tab / homepage / restore last session
- 6 search engines (DuckDuckGo, Google, Bing, Brave, Startpage, Ecosia)
- Dark & light themes plus a custom accent colour applied across every surface
- Privacy: ad & tracker blocking, Do Not Track / Sec-GPC, third-party cookies
- Clear browsing data, download preferences, shortcut reference, about page

**Library** (`owbr://history`, `Ctrl+H`) — history grouped by day, bookmarks and
downloads with progress bars, all searchable.

**Privacy** — built-in blocklist covering ~35 ad/tracking networks, with a live
blocked counter in the menu and on the new tab page. Toggle from the 🛡 button.

**Security** — context isolation and sandbox on, no Node in web pages,
permissions denied by default, Electron hidden from the user agent.

## Privacy

OWBR is built to be private by default — every protection below is **on** out of
the box, no configuration needed.

**Tracker blocking** — 262 known ad, analytics, social, fingerprinting,
cryptomining and session-replay hosts across 6 independently toggleable
categories. Strict mode additionally kills beacon/telemetry requests to any
third party. Per-site exceptions from the 🛡 shield menu.

**Cookies** — third-party cookies blocked by default (this is *the* mechanism
that follows you between sites). Also available: session-only cookies that die
on exit, or blocking cookies entirely. Enforced on both the request `Cookie`
header and the response `Set-Cookie`, so a tracker can't set one at all.

**Cookie banners** — refused automatically. OWBR pre-writes "everything off"
consent cookies for 26 CMP platforms *before* the page loads, so on most sites
the banner never appears at all. If one does appear, an in-page script finds and
clicks the real **Reject all** control — for OneTrust, Cookiebot, Usercentrics,
Didomi, Sourcepoint, Quantcast, TrustArc, Osano, Borlabs, Complianz,
consentmanager.net, CookieYes and more — plus text matching in English, German,
French, Spanish, Italian, Dutch, Danish, Swedish and Polish, including inside
shadow DOM. Anything left over gets hidden and your scrolling restored.

**Sites that force you into a settings panel** are handled too: where there is no
"Reject all", OWBR clicks *Cookie settings* / *Einstellungen* / *Manage preferences*,
switches off every non-essential purpose (checkboxes, `role="switch"` buttons and
`aria-pressed` toggles alike), leaves *strictly necessary* enabled because it is
required, then clicks *Save* / *Speichern* / *Confirm my choices*. Purpose labels
are matched individually, so one row saying "essential" cannot protect the rest.

It **never accepts on your behalf**: sites offering only "accept or pay" (the
German *Pur* model) are left untouched, because there is nothing to refuse.

### Interface style

**Settings → Appearance → Interface style** switches between **Default** and
**Soft**. Soft is a frosted skin: translucent surfaces over a slow colour wash
tinted by your accent, fully rounded tabs and controls, no drop shadows, and
hover feedback from fill rather than lift. It lives in its own stylesheet
(`src/renderer/soft.css`) with every rule scoped to `body.ui-soft`, so choosing
Default leaves the original chrome byte-for-byte unchanged. It covers the internal pages (new tab, settings,
history, diagnostics) as well as the chrome, and honours
`prefers-reduced-motion` and degrades to opaque surfaces where `backdrop-filter`
is unavailable.

### Design

Every icon in the chrome is a single stroked SVG set drawn at one weight and
size. The old build mixed emoji (lock, shield, star, broom) with text glyphs
(chevrons, refresh, ellipsis) — they never matched in colour, weight or
baseline, and emoji render differently on every OS. Address-bar security state
is now colour, not a coloured picture: green padlock, amber warning, grey for
browser pages.

**Tabs stop shrinking at 76px and the strip scrolls instead.** Below roughly
63px a tab has no room for its close button, so shrinking past that leaves tabs
you cannot close — the exact failure Chrome shipped when it removed tab
scrolling. Firefox's `browser.tabs.tabMinWidth` defaults to a similar floor for
the same reason. Scroll buttons appear only when needed and grey out at each
end, the edges fade to signal more tabs, the mouse wheel scrolls the strip, and
the active tab is always scrolled into view.

Tabs sit at the top, as they should. The chrome is drawn as **one continuous
surface** with a faint accent wash falling off toward the toolbar, rather than
three separately-coloured bars stacked on each other — that stacking was what
made it look unfinished. Inactive tabs are fully transparent and only tint on
hover; the active tab is the one lit surface and **merges into the toolbar
below it** via masked corner connectors, so tab and page read as one object.
Close buttons fade in on hover instead of always sitting there. The address bar
is a full pill with a focus ring, and pinned tabs mark themselves on the favicon.

Every surface — browser chrome, new tab, settings, history, diagnostics and the
error page — is styled from one shared stylesheet, `src/renderer/theme.css`.
Before, each page carried its own copy of the palette and they had already
drifted (the chrome was `#1b1d21` while internal pages were `#17191d`).

It defines a six-step neutral ramp, semantic colours, three elevation levels,
four radii and shared motion easings, in both dark and light. Chrome-specific
touches: the active tab now merges into the toolbar with rounded connectors
instead of floating above it, all five popovers share one shell and animation,
and every interactive element has a hover, focus and active state. Scrollbars
are styled once globally, keyboard focus rings only appear for keyboard users,
and `prefers-reduced-motion` disables animation.

A test fails the build if any page reintroduces a private palette, uses an
undefined variable, or repeats a raw hex colour across three or more pages.

### Start page

Three columns: pinned bookmarks on the left under a striped wordmark, a
time-aware greeting and search field in the centre, and recently visited sites
on the right with relative timestamps. Workspace cards sit below. Both lists are
real data — bookmarks and history — with duplicate hosts collapsed and empty
states handled; clicking a row navigates.

### Old start page notes

The new-tab page is built around **workspaces, not website shortcuts**. A grid
of generic links to DuckDuckGo and Reddit is noise — you already have an address
bar. Instead each workspace is a card showing its colour, icon, and whether it is
open now, empty, ephemeral, or holding *n* saved tabs. Click one to switch, or
press its **number key** (1–9). A dashed card adds a new workspace.

Above it: a time-aware greeting naming the workspace you are in, the date, and a
search field. Below it: live counts of trackers blocked, banners refused, links
cleaned and the total blocklist size.

### Tabs

- **Pin a tab** (right-click → Pin, or the tab menu). Pinned tabs shrink to their
  favicon, sit at the front of the strip, and ignore Ctrl+W and middle-click so
  you cannot lose them by reflex — the ✕ still closes them deliberately.
- **Reopen a closed tab** with **Ctrl+Shift+T**, up to 25 deep. It comes back in
  the workspace it came from, still pinned if it was. Private tabs are never
  recorded.
- **Drag tabs to reorder** them. Pinned and unpinned form separate zones, so a
  drag cannot accidentally unpin something.
- **Right-click a tab** for a real menu: reload, duplicate, pin, mute, copy
  address, move to another workspace, close others, close to the right, reopen
  last closed. (It used to just duplicate the tab with no menu at all.)
- **Ctrl+Shift+Tab** steps backwards through tabs.

### Workspaces

Not tab groups — **separate browsing contexts**. Each workspace owns its own
Electron session partition, which means its own cookies, localStorage, cache,
IndexedDB and service workers. Practically: *Work* and *Personal* can both be
signed into Gmail on different accounts, at the same time, with no logging out.

- Each workspace remembers its own tabs and restores them when you switch back.
- Switching **tears down** the old workspace's tabs rather than hiding them, so
  a big workspace does not keep dozens of renderer processes alive.
- Deleting a workspace **erases its storage** — logins do not linger on disk.
- Mark a workspace **ephemeral** and everything it stored is wiped on quit.
- Tracker blocking, the storage jail and permission rules apply to every
  workspace; the filters are installed per partition, once.

**Adding one:** click the coloured pill at the top-left of the tab strip and
choose **＋ New workspace**, or press **Ctrl+Shift+D**. Pick a name, colour and
icon, then hit Create (or just press Enter). The new workspace opens immediately
with a fresh, empty cookie jar.

**Switching:** click the pill and pick one, or press **Ctrl+Shift+E** to cycle.
Rename and delete are in the same menu. Up to 12 workspaces; two (*Personal*,
*Work*) ship by default.

### Security

- **Certificate errors are never bypassable.** There is no "proceed anyway" —
  clicking through is exactly how interception attacks succeed. You get a page
  explaining which check failed instead.
- **Look-alike domain guard.** `аpple.com` with a Cyrillic *а* is a different
  site from `apple.com`. Any label mixing Latin with a confusable script, or
  containing zero-width or right-to-left-override characters, is flagged.
- **`data:` / `javascript:` top-level navigation is blocked** — a classic
  phishing and XSS vector, since the address bar shows nothing useful.
- **Insecure downloads blocked.** A file fetched over plain http can be swapped
  in transit; that is how installers get trojaned on public wifi.
- **Executable downloads flagged** (.exe, .msi, .scr, .ps1, .dmg, …).
- **Renderer lockdown:** context isolation on, node integration off, sandbox on,
  `webviewTag` off, web security never disabled, popups get no opener handle.
- All six permissions (camera, mic, location, notifications, …) default to
  **denied**, not "ask".

### Performance

- Blocklist lookups are **memoised and split**: the ~50 entries needing path
  matching no longer force every request to scan the whole list. Measured
  **73 ms for 60,000 non-matching lookups, down from ~1,950 ms** (~26× faster) —
  this runs on every single request, so it dominates browsing latency.
- Tab-state IPC is **coalesced to one send per frame**. Page loads fire
  title/favicon/loading/navigate events in bursts that previously each
  serialised and shipped the whole tab list.
- Downloaded blocklists are capped and loaded once at startup, not per request.
- Download progress updates are throttled to ~3/second.

### Blocklists

905 ad/tracker domains are built in and always work offline. On top of that OWBR
downloads full public blocklists on first run — StevenBlack, AdGuard DNS,
EasyPrivacy, EasyList, Prigent, Admiral, NoTracking and AdGuard's CNAME
first-party tracker list — which together add **hundreds of thousands of
domains**, including the tiny obscure ones. They refresh automatically every few
days and are cached on disk, so the browser still blocks with no network.

A built-in allowlist means a blocklist can never break sign-in: Google accounts,
PayPal, Stripe, GitHub, CDNs and `googlevideo.com` are dropped from any list that
includes them. Downloaded lists also never block first-party requests.

Manage them in **Settings → Privacy → Blocklists** ("Update now" shows the live
domain count).

### YouTube ads

YouTube ads **cannot** be blocked by domain — the ad stream comes from the same
`googlevideo.com` server and connection as the video you asked for, so blocking
it blocks all playback, and blocking the ad-metadata endpoints just yields "an
error occurred". OWBR handles them in the page instead: it presses **Skip** the
instant it appears, **fast-forwards unskippable pre-rolls** to their end (muted),
and removes banner/overlay/sponsored units. You may still glimpse a fraction of a
second before the skip lands. Optional: hide promoted items in the feed.

### Is it working? (`owbr://diagnostics`)

🛡 shield → **"Why am I still seeing this?"** shows, for the site in your previous
tab: whether it is jailed and in which mode, every cookie it currently has with
its real expiry date, and which protections are active. If a banner still shows,
this tells you whether it actually matters.

**Clicking "Accept" on a jailed site is safe.** The consent itself is stored in a
cookie — and on a jailed site that cookie is either blocked outright or stripped
of its expiry so it dies with the visit. The site forgets you either way.

### Sites that give you no choice

Pay-or-consent walls can't be argued with — so OWBR stops asking and takes their
storage away instead. Detection is automatic (a consent dialog with no refuse
button) and 36 known walls (FAZ, Spiegel, Zeit, Welt, NYT, WSJ, Le Monde…) are
covered from the very first visit.

| Mode | What happens |
|---|---|
| **Delete on leave** *(default)* | The site works normally while you read. The moment you navigate away, every cookie, localStorage entry, IndexedDB database, cache and service worker for that domain is erased — including cookies set on the parent domain. It cannot recognise you tomorrow. |
| **Block entirely** | The page gets in-memory shims instead of real storage. `localStorage`, `sessionStorage`, `document.cookie`, IndexedDB, the Cache API and service workers all appear to work but forget instantly. Strongest; may break logins. |
| **Off** | Normal behaviour. |

Pay-or-consent walls specifically (FAZ, NYT, Spiegel, Zeit, WSJ…) default to
**block**, because delete-on-leave still lets them store an ID while you read.
For those, `Set-Cookie` is stripped at the network layer and no `Cookie` header
is ever sent back, so nothing is stored in the first place.

All scripts run in **every frame, including cross-origin iframes** — Sourcepoint
(FAZ, Spiegel) renders its banner in an iframe from its own CDN, which a
top-frame-only injection can never reach.

Nothing is wiped while another tab is still open on that site; closing the tab
or quitting wipes too. Tabs show ⏱ or 🔒 so you can see the mode at a
glance, and the 🛡 shield menu lets you cycle it per site or wipe a site now.

**Tracking links** — `utm_*`, `fbclid`, `gclid`, `msclkid`, `igshid` and ~40 more
parameters are stripped from URLs before the request is even sent.

**Fingerprinting** — the harder problem: even with cookies gone, sites identify
you by your canvas rendering, GPU string, audio stack and hardware counts. OWBR
injects per-session noise into all of them before any page script runs, reports
generic hardware values, blocks the battery-status identifier, empties the
plugin list, and coarsens `performance.now()` to defeat timing attacks. The
noise is stable within a session (so sites can't average it away) but changes
every launch (so you're not trackable across restarts).

**Network** — HTTPS-only upgrading, DNT + Global Privacy Control headers,
referrer trimmed to the origin cross-site, client-hint headers stripped
(they leak CPU, memory and screen), WebRTC restricted so it can't reveal your
real IP behind a VPN.

**Permissions** — camera, microphone, location, notifications, MIDI, USB, serial
and Bluetooth are *all denied by default*, and unknown permissions deny too.
A site cannot prompt its way in; you enable them in Settings.

**Private tabs** (`Ctrl+Shift+N`) — separate in-memory session with no cookies,
cache or storage persisted, and nothing written to history.

**Clear on exit** — optionally wipe cookies, cache, site storage and history
every time you close the browser.

Click the 🛡 shield in the toolbar to see exactly what was blocked on the page
you're looking at, broken down by category.

## Reliability

OWBR is built so that no single failure can take down the browser.

**Crash isolation** — each tab is its own OS process. If a page's renderer dies,
the tab shows a recovery page and every other tab keeps running. Hung pages get
a ⏳ badge instead of freezing the UI.

**Never lose data** — the store is written atomically (temp file → rename) with a
`.bak` kept alongside. On startup a corrupt file falls back to the backup, and
every field is type-checked and clamped, so truncated JSON, binary garbage or a
half-written file can't wipe your bookmarks. Pending writes flush on quit.

**Nothing throws** — global `uncaughtException` / `unhandledRejection` handlers
log to `userData/owbr.log` and keep going. Every `webContents` call checks the
target is alive first, so closing a tab mid-operation is harmless.

**Input validation** — everything crossing the IPC boundary is validated:
`javascript:` and `vbscript:` URLs are rejected, URL length is capped, tab IDs
must exist, overlay/height values are clamped, downloads can only be opened if
they're in your own download list and still on disk, and `openExternal` accepts
only `http(s)`.

**Hardening** — sandbox on, Node disabled in pages, context isolation, webview
attachment blocked, permissions denied by default, single-instance lock,
tab count capped.

### Tests

```bash
npm test          # 707 checks
npm run test:ui        # every button/menu fires the right IPC call (jsdom)
npm run test:settings  # every switch/dropdown maps to a real setting
npm run test:banners   # CMP layouts, two-step flows, storage jail, wall detection
npm run test:robust    # hostile input, corrupt stores, privacy engine, audit
```

The robustness suite feeds ~50 malformed values (null, NaN, huge strings,
prototype-pollution attempts, script URLs) into the URL and settings parsers,
then corrupts a real store file seven different ways to prove recovery works.

## Run

```bash
npm install     # downloads the Chromium/Electron runtime
npm start
```

> Note: this cloud sandbox blocks GitHub's release CDN, so the Electron binary
> could not be downloaded here (`npm install` was run with `--ignore-scripts`).
> On your own machine a plain `npm install` fetches it and `npm start` works.
> Behind a restrictive network you can set `ELECTRON_MIRROR` to a mirror.

## Package installers

```bash
npm run dist    # AppImage / NSIS / DMG via electron-builder
```

## Layout

```
src/main/main.js            app process: windows, tabs, navigation, storage, IPC
src/main/chrome-preload.js  safe bridge for the browser UI
src/main/page-preload.js    forwards shortcuts pressed inside web pages
src/renderer/chrome.html/js the browser chrome (tab strip + toolbar + menu)
src/renderer/newtab.html    new tab page
```

The window is a `BaseWindow`: the chrome UI is one `WebContentsView` pinned to the
top 88px, and every tab is its own `WebContentsView` below it — the same
architecture Chrome itself uses, so a hung page can never freeze the UI.

## Making it yours

- Rebrand: `productName`/`appId` in `package.json`, `<h1>` in `newtab.html`
- Default search: `SEARCH_URL` in `src/main/main.js`
- Theme: CSS variables at the top of `src/renderer/chrome.html`
- Next ideas: ad blocking via `session.webRequest`, downloads manager,
  find-in-page, private windows, extension support (`session.loadExtension`).

## Troubleshooting: "Electron failed to install correctly"

The `electron` npm package is just a stub; a postinstall script downloads the
real Chromium runtime. Newer npm versions block install scripts by default, so
that download may never run. Fix it with the bundled fallback installer:

```
npm run fetch-electron
npm start
```

Behind a firewall that blocks GitHub's release CDN, use a mirror:

```
set ELECTRON_MIRROR=https://npmmirror.com/mirrors/electron/
npm run fetch-electron
```

Fully manual alternative: download `electron-v<version>-win32-x64.zip` from the
Electron releases page, extract it to `node_modules/electron/dist/` (so
`dist/electron.exe` exists), and create `node_modules/electron/path.txt`
containing `electron.exe`.

## Building an installer

`npm run dist` produces a real installer — `.exe` on Windows, `.AppImage`/`.deb`
on Linux, `.dmg` on macOS. See **[BUILDING.md](BUILDING.md)** for what else
turning this into a distributable browser involves: code signing, auto-updates,
default-browser registration, and keeping Chromium current.
