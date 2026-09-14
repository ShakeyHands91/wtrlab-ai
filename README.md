# WTR-LAB (AI) — personal LNReader plugin repo

A modified build of the official LNReader WTR-LAB plugin that lets you choose which
translation section to read — **AI**, **Web+**, or **Web** — instead of taking whatever
the site serves by default.

Everything here is prebuilt. You do not need Node, npm, or any build step.

---

## 1. Create the repo

1. On GitHub, create a **new public repository** (any name — e.g. `wtrlab-ai`).
   It must be public: LNReader fetches the files with no authentication.
2. Upload the contents of this folder to it, keeping the folder structure exactly as-is.
   The fastest way is GitHub's "uploading an existing file" link, then drag the whole folder in.

> **Do not add a Node `.gitignore`.** The `.dist/` and `.js/` folders here are the actual
> published output — a standard Node gitignore would exclude them and the repo would serve nothing.

## 2. Point the manifest at your repo

Open `.dist/plugins.min.json` (and `.dist/plugins.json`) in GitHub's web editor and replace
both occurrences of `__BASE__` with:

```
https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main
```

So `"url": "__BASE__/.js/src/plugins/english/wtrlab.js"` becomes
`"url": "https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/.js/src/plugins/english/wtrlab.js"`.

If your default branch is `master` rather than `main`, use that instead.

## 3. Add it in LNReader

Settings → **Plugins** → **Add repository**, and paste:

```
https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/.dist/plugins.min.json
```

Then install **WTR-LAB (AI)** from that repo.

It uses the same plugin id (`WTRLAB`) as the official plugin, so your existing library entries
keep working. It is version `1.2.0` against the official `1.1.6`, so LNReader treats it as the
newer one.

---

## Plugin settings

Long-press the plugin in the Plugins list to open its settings.

| Setting | What it does |
| --- | --- |
| **Session cookie** | Optional. Only needed for chapters that require a signed-in account. See below. |
| **Preferred translation** | `AI`, `Web+`, `Web`, or `Custom`. Default is AI. |
| **Custom translation id** | Only used when Preferred is set to Custom. The raw value wtr-lab sends as `translate`. |
| **Fall back to Web…** | On by default. Turn it **off** if you would rather see an error than silently get the Web version. |
| **Show which translation was used** | Prints `Translation: ai` (or `webplus`, `web`, `google (on-device)`) at the top of each chapter. |

### About the session cookie

Testing showed the AI section answers fine **without** being signed in, so leave this blank
to start. It exists for chapters that are account-gated.

If you do need it: sign in to wtr-lab in a browser, open DevTools → Network, click any request
to wtr-lab.com, and copy the entire value of the **Cookie** request header. Paste that whole
string into the setting. (Copying it from the Console with `document.cookie` will not work —
the session cookie is httpOnly and does not appear there.)

Treat that value like a password. It is stored only on your device, and because it lives in a
settings field rather than in the code, it is never part of anything you publish.

---

## What was changed

Against the official `plugins/english/wtrlab.ts` (v1.1.6):

- Added `pluginSettings` — session cookie, preferred mode, custom mode, fallback toggle, notice toggle.
- The chapter fetch loop now takes its mode list from settings instead of the hardcoded `['ai', 'web']`.
- `webplus` added as a selectable mode (the Web+ tab on the site) — not available upstream.
- The session cookie, when set, is sent as a `Cookie` header on the reader API and chapter-list calls.
- Clearer errors: which mode failed and why, plus a hint when no cookie is set.
- Optional per-chapter notice of which translation you actually got.

The readable source is in `src/wtrlab.ts`. The file LNReader actually loads is the compiled,
minified `.js/src/plugins/english/wtrlab.js` — if you edit the source, the JS has to be
rebuilt from it; editing the `.ts` alone changes nothing.

## Updating later

If the official plugin gets fixes you want, the changes above are small and self-contained —
they can be reapplied to a newer upstream version.
