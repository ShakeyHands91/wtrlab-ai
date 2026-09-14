# WTR-LAB (AI) — personal LNReader plugin repo

A modified build of the official LNReader WTR-LAB plugin that reads the **AI** translation
section instead of falling back to the machine-translated Web version — and can sign in to a
wtr-lab account, which is what unlocks AI past the free preview.

Everything here is prebuilt. No Node, no npm, no build step.

---

## Installing

LNReader → **Plugins** → **Add repository**:

```
https://raw.githubusercontent.com/ShakeyHands91/wtrlab-ai/main/plugins.min.json
```

Then install **WTR-LAB (AI)**.

It keeps the official plugin's id (`WTRLAB`), so existing library entries carry over, and its
version is ahead of the official `1.1.6` so LNReader treats it as the newer build.

## Signing in

wtr-lab gives guests AI translation for **the first 10 chapters of any novel**. Past chapter 10
it needs an account — that, and not the novel, is why some chapters used to fall back to Web.

Signing in through LNReader's built-in WebView does not work: Google blocks OAuth in embedded
WebViews, and the app never flushes WebView cookies to disk. Pasting a `Cookie` header does not
work either — on Android the system cookie store replaces that header whenever it holds cookies
for the domain. So the plugin redeems an email sign-in link itself:

1. On wtr-lab (any browser), choose **Continue with Email** and request a link
2. In the email, **copy the link address** — do not tap it, or the session lands in that browser
3. Paste it into the plugin's **Sign-in link** setting
4. Open any chapter — the notice line should read `signed in as …`
5. **Clear the Sign-in link field.** The links are single-use; left in place, each app launch
   retries a dead token and prints a failure line above your chapters

The plugin pulls the `token=` value out of the link and calls
`/api/auth/magic-link/verify` directly. The emailed URL is a *page* whose JavaScript would
normally do this; nothing runs that script inside the app, which is why opening the page itself
achieves nothing.

The session then lives in the app's own cookie store, where it is not subject to the header
problem above. Sessions run about 30 days and refresh with use, so this is rarely repeated.

**Per device.** Sessions are per-device and links are single-use, so each device needs its own.
A fresh link is also needed after Settings → Advanced → Clear cookies, clearing app data, or
reinstalling.

## Settings

Long-press the plugin in the Plugins list.

| Setting | What it does |
| --- | --- |
| **Sign-in link** | Paste the emailed link here to sign in. Clear it once AI chapters load. |
| **Session cookie** | Fallback only; normally leave empty. Android overrides this header when it has its own cookies. |
| **Preferred translation** | `AI`, `Web+`, `Web`, or `Custom`. Default AI. |
| **Custom translation id** | Only used when Preferred is Custom — the raw value sent as `translate`. |
| **Fall back to Web…** | On by default. Off means an error instead of a silent downgrade — useful when diagnosing. |
| **Show which translation was used** | Prints the translation and sign-in state at the top of each chapter. |

The notice line looks like `Translation: ai — signed in as Shakey`, with a second line listing
anything that was skipped and why.

## Repo layout

| Path | What it is |
| --- | --- |
| `plugins.min.json` | The repository manifest — the URL LNReader points at. |
| `plugins.json` | Same content, readable. Not used by the app. |
| `wtrlab.js` | The compiled, minified plugin — what LNReader downloads and runs. |
| `wtrlab.ts` | Readable TypeScript source, for future edits. |
| `public/static/src/en/wtrlab/icon.png` | Plugin icon. |

Editing `wtrlab.ts` alone changes nothing — `wtrlab.js` must be rebuilt from it, compiled
against the upstream [lnreader-plugins](https://github.com/LNReader/lnreader-plugins) repo
which supplies the `@libs/*` modules:

```
npx tsc --project tsconfig.production.json --rootDir plugins
```

Bump the `version` in both the source and the two manifests, or LNReader won't offer the update.

## What was changed

Against the official `plugins/english/wtrlab.ts` (v1.1.6):

- **Sign-in link setting** that extracts the magic-link token and redeems it via the auth API.
  Only ever tried once per app run, so a single-use token is not burned twice, and refused
  outright unless the link is an `https://…wtr-lab.com/` address.
- **`webplus`** added as a selectable mode — the site's Web+ tab, not available upstream.
- **Explicit mode choice** rather than the hardcoded `['ai', 'web']`, with an optional fallback.
- **Session check** against `/api/auth/get-session` so the notice reports what the *server*
  thinks, rather than what the plugin assumes.
- **Real diagnostics**: each failed attempt reports its HTTP status and where it ended up, and
  responses are read as text before parsing so an HTML error page is reported rather than
  throwing.
- Optional per-chapter notice of the translation actually used.

## Known limitations

- **No reading-progress sync.** A source plugin gets five read-only methods and no hook for
  finished chapters, and LNReader's tracker list is a hardcoded union of four services. Syncing
  progress to a wtr-lab account would mean forking the app itself.
- **WebView sign-in is unusable**, for the reasons above.
