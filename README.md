# WTR-LAB (AI) — personal LNReader plugin repo

A modified build of the official LNReader WTR-LAB plugin that lets you choose which
translation section to read — **AI**, **Web+**, or **Web** — instead of taking whatever
the site serves by default.

Everything here is prebuilt. No Node, no npm, no build step.

---

## Installing

Add this URL in LNReader → **Plugins** → **Add repository**:

```
https://raw.githubusercontent.com/ShakeyHands91/wtrlab-ai/main/plugins.min.json
```

Then install **WTR-LAB (AI)** from that repo.

It uses the same plugin id (`WTRLAB`) as the official plugin, so existing library entries keep
working, and it is version `1.2.0` against the official `1.1.6`, so LNReader treats it as newer.

## Plugin settings

Long-press the plugin in the Plugins list to open its settings.

| Setting | What it does |
| --- | --- |
| **Session cookie** | Optional — only for chapters that require a signed-in account. See below. |
| **Preferred translation** | `AI`, `Web+`, `Web`, or `Custom`. Default is AI. |
| **Custom translation id** | Only used when Preferred is Custom. The raw value wtr-lab sends as `translate`. |
| **Fall back to Web…** | On by default. Turn it **off** to get an error instead of silently reading the Web version. |
| **Show which translation was used** | Prints `Translation: ai` (or `webplus`, `web`, `google (on-device)`) at the top of each chapter. |

### About the session cookie

Testing showed the AI section answers fine **without** being signed in, so leave this blank to
start. It exists for chapters that turn out to be account-gated.

If you do need it: sign in to wtr-lab in a browser, open DevTools → Network, click any request
to wtr-lab.com, and copy the entire value of the **Cookie** request header. Paste that whole
string in. (`document.cookie` in the Console will not show it — the session cookie is httpOnly.)

Treat that value like a password. It is stored only on your device, and because it lives in a
settings field rather than in the code, it is never part of anything published here.

---

## Repo layout

| Path | What it is |
| --- | --- |
| `plugins.min.json` | The repository manifest — this is the URL LNReader points at. |
| `plugins.json` | Same content, readable. Not used by the app. |
| `dist/wtrlab.js` | The compiled, minified plugin — what LNReader actually downloads and runs. |
| `src/wtrlab.ts` | Readable TypeScript source, for future edits. |
| `public/static/src/en/wtrlab/icon.png` | Plugin icon. |

Editing `src/wtrlab.ts` alone changes nothing — `dist/wtrlab.js` has to be rebuilt from it
(`npx tsc` against the upstream lnreader-plugins repo, which supplies the `@libs/*` types).

## What was changed

Against the official `plugins/english/wtrlab.ts` (v1.1.6):

- Added `pluginSettings` — session cookie, preferred mode, custom mode, fallback toggle, notice toggle.
- The chapter fetch loop takes its mode list from settings instead of the hardcoded `['ai', 'web']`.
- `webplus` added as a selectable mode (the Web+ tab on the site) — not supported upstream.
- The session cookie, when set, is sent as a `Cookie` header on the reader API and chapter-list calls.
- Clearer errors: which mode failed and why, plus a hint when no cookie is set.
- Optional per-chapter notice of which translation you actually got.

The upstream project is [LNReader/lnreader-plugins](https://github.com/LNReader/lnreader-plugins).
If it gets fixes worth having, the changes above are small and self-contained enough to reapply
to a newer version.
