# grist-cursor-persist

## What it does

Invisible 1-page widget whose only job is to remember which row was selected in a Grist table and re-select it on reload. The user binds it to whatever table they want stable cursor for (in this project, the Events table), enables `allowSelectBy` so linked widgets follow its selection, and the widget keeps a single `sessionStorage` entry (`persist_cursor_Event`) with the last `record.id`. On page load it calls `grist.setSelectedRows([rowId])` to restore it.

Used to work around Grist losing the cursor after iframe reloads, switching pages, or refreshing the browser tab — which is annoying when several linked widgets all depend on the same Events row.

## Grist surface

| Aspect | Detail |
|---|---|
| `grist.ready` | `{requiredAccess: 'full', columns: ['id'], allowSelectBy: true}` ([index.html:13](index.html#L13)) |
| Reads | `grist.onRecord` — only the `record.id` ([index.html:22](index.html#L22)) |
| Writes (Grist) | `grist.setSelectedRows([rowId])` on page load — no row data is mutated. |
| Writes (browser) | `sessionStorage['persist_cursor_Event']` — survives tab refresh, dies when the tab closes. |

## Tables/columns touched

**None directly.** The widget binds to whatever table the user attaches it to; it only reads `record.id`. The `id` of the bound row is the only data point persisted.

## Files

| File | Purpose |
|---|---|
| `index.html` (45 lines) | The entire widget — inline script. |
| `grist-plugin.json` | Manifest declaring `id: cursor-persist`, `requiredAccess: full`, `allowSelectBy: true`, and a `sections[0].mappings.id → table: "Event", column: "id"`. |

## Dependencies

- `grist-plugin-api.js` (from `docs.getgrist.com`). Nothing else.

## Build / deploy

- None. Static `index.html`. **No `_config.yml` yet → not actually published anywhere.** To put this in production the file needs to be hosted (GitHub Pages, `~/GristData/custom_static/cursor-persist/` on the VM, or similar) and the URL pasted into the Grist custom-widget dialog.

## Tests / lint / typecheck

- None.

## State of the code

**Prototype, production-quality but undeployed.** The code itself is tight (one effect each way, no edge cases). The blockers are the deployment story and a manifest issue:

- `grist-plugin.json` declares `"table": "Event"` (**singular**). Every doc in `vm-context/schema/` has `Events` (**plural**). The manifest path is only consulted when Grist is fed a `GRIST_WIDGET_LIST_URL` (which it currently isn't — see [vm-context/widgets.md](../vm-context/widgets.md)), so this doesn't break runtime, but it's misleading documentation.
- sessionStorage key is hard-coded `'persist_cursor_Event'` — if you bind the widget to *another* table (Periods, etc.) in some future doc, two instances will fight over the same key.

## Quick gotchas

- `sessionStorage` is **per-tab** by design. Reloading the tab restores the cursor; opening a new tab doesn't. If you want cross-tab persistence, switch to `localStorage`.
- A 200 ms `setTimeout` is used to let Grist finish loading before calling `setSelectedRows` ([index.html:31](index.html#L31)). Brittle; consider awaiting a Grist-ready signal instead.
