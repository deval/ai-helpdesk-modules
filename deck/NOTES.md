# Building this deck — lessons learned

Notes from building the workshop slide deck with reveal.js, kept here so the
next round of edits (or the next deck built this way) doesn't re-discover the
same bugs from scratch.

## Library choice: remark.js → reveal.js

Started with **remark.js** for the "simple, offline, single file" requirement.
Switched to **reveal.js** once richer animation was wanted — specifically
Auto-Animate (elements morph between two adjacent slides automatically) and
per-fragment builds, which remark.js doesn't have. reveal.js needs more setup
(separate JS/CSS files, a plugin ecosystem) but is worth it once the deck
needs more than fade transitions.

## Bug 1: inlining a minified library into `<script>` breaks it

Original remark.js version concatenated the library's minified JS directly
into one `<script>...</script>` block (`cat head.html lib.js tail.html >
deck.html`) for a true single-file deck.

**Symptom:** page showed raw JS source as plain text instead of rendering
slides at all.

**Root cause:** the minified bundle contained bundled syntax-highlighting
language grammars (highlight.js definitions for asm, HSP, etc.) whose string
data happened to contain the literal substring `</script>`. HTML's parser
doesn't care that it's inside a JS string — it terminates the `<script>`
block right there, and everything after gets parsed as literal page text.

**Fix:** never inline a third-party minified bundle into a `<script>` tag.
Always load it as a separate external `.js` file (`<script src="lib.js">`).
Doesn't cost you "offline, no server" — both files just need to sit in the
same folder. This is why every library here (`reveal.js`, `menu.js`, theme
CSS) is vendored as its own file under `reveal-dist/`, never inlined.

## Bug 2: reveal.js-menu plugin silently does nothing

Added the plugin as `plugins: [ RevealMenu ]`. No JS errors, `Reveal.
initialize()` resolved fine, but the hamburger button never appeared.

**Root cause:** `RevealMenu` (as loaded via `<script src="menu.js">`) is a
**factory function**, not a ready plugin object. reveal.js's plugin API wants
`{id, init}` objects in the `plugins` array. Passing the bare function
reference meant `.init` was `undefined` on it, so nothing ran — silently,
no error, because reveal.js just doesn't find `.init` and moves on.

**Fix:** call it — `plugins: [ RevealMenu() ]`, not `plugins: [ RevealMenu ]`.

**How this was actually diagnosed:** no browser access in this environment
(no MCP browser tool, no working Chrome extension bridge from this session).
Built a temporary on-page debug banner (`window.onerror` handler + a visible
`<div>` logging each step: library loaded? plugin loaded? did `initialize()`
resolve? does the button exist in the DOM? what's its computed style/rect?)
instead of asking for DevTools console output blind. `typeof RevealMenu ===
'function'` (not `'object'`) was the concrete tell that pointed straight at
the factory-function issue. Worth reaching for this pattern again anytime a
plugin "loads fine but does nothing" and there's no direct browser access.

## Bug 3: Font Awesome dependency, silently missing

reveal.js-menu's hamburger/close/tab icons are hardcoded as `<i class="fas
fa-bars">` etc. — they need Font Awesome's CSS+webfont to render a glyph.
Only `menu.js` and `menu.css` were vendored, not the `font-awesome/` folder
that ships alongside them in the npm package.

**Symptom:** button existed in the DOM, correctly positioned, `display:
block`, `visibility: visible` — and still invisible, because its entire
visible content came from an icon font that was never loaded. (`.slide-menu-
button` itself has no background/border in the plugin's CSS — it's 100%
reliant on the icon glyph for any visible pixels.)

**Fix:** rather than vendor the whole Font Awesome package for three icons,
override with plain CSS `content: "☰"` / `"✕"` (unicode characters) on the
`.fas.fa-*::before` selectors, and set `loadIcons: false` in the plugin
config so it doesn't try to fetch a `font-awesome/` path that doesn't exist.

**Lesson:** "button exists, correctly styled, still invisible" is a strong
signal to check whether its content depends on an icon font that wasn't
actually loaded — check computed content/glyph, not just position/display.

## Bug 4: two fixed-position buttons in the same corner

Added a custom "Overview" button at `bottom:16px; left:16px`. The menu
plugin's own button defaults to `bottom:30px; left:30px` — same corner.
Equal z-index, added later in the DOM → the custom button sat directly on
top of the hamburger, hiding it completely. Looked exactly like "the plugin
isn't rendering" but was actually a plain CSS collision.

**Lesson:** when adding custom fixed-position UI to a page that already has
a plugin injecting its own fixed-position UI, check the plugin's default
CSS for its corner/position *before* picking a spot for anything new.

(This button was later removed entirely — overview mode has no touch-drag
support in reveal.js, per an open upstream feature request, so swipe-to-pan
in overview mode feels stepped/choppy on touch devices by design, not bug.
The menu plugin's tap-to-jump list is the better touch-native option and was
kept as the only navigation-jump mechanism.)

## Environment: serving over Tailscale

- `python3 -m http.server 8888` in the deck folder, backgrounded.
- Two ways to expose it on the tailnet: `tailscale serve --bg 8888` (adds a
  reverse-proxy HTTPS layer at `https://device.tailnet.ts.net/`) or just hit
  the plain Tailscale IP directly (`http://100.x.x.x:8888/...`) since
  Python's http.server already binds all interfaces by default — no proxy
  needed for casual same-tailnet viewing. Went with the plain-IP route since
  it's one less moving part.
- `tailscale serve --https=443 off` tears the proxy down cleanly if it was
  used; the plain python server needs its own `kill` when done.

## File structure reference

```
deck/
├── workshop-deck.html       — the real deck (42 slides, all 3 days/11 modules)
├── sample-deck-reveal.html  — original 7-slide reveal.js proof-of-concept
├── sample-deck.html         — superseded remark.js version (kept for reference)
├── remark.min.js            — remark.js lib, only used by the superseded file above
└── reveal-dist/
    ├── reveal.js / reveal.css
    ├── theme/black.css      — base theme; navy/maroon is CSS overrides on top, inline in workshop-deck.html
    └── plugin/menu/menu.js / menu.css
```
