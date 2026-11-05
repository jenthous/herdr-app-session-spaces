# Session spaces

A spaces sidebar in place of Herdr's `spaces`: each row shows the space's agent state where the avatar was, its Linear issue as a chip when it's named after one, then its name and the git of its first terminal. A space has a color only when you give it one (right-click), drawn as a thin stripe. Next to the machine-switch mod it shows one session; without it, every machine's spaces under machine headers.

![Session spaces](preview.png)

## What you see

- **Header (31 px, 16 px left, 10 px right):** "Spaces", 11.5 px `dim` (hidden when the sidebar is narrower than 160 px); a 26 px `+` on the right (radius 6, `hover` background) that makes a space on the shown machine.
- **List (8 px sides, 14 px bottom, always-visible scrollbar).** One row per space:
  - 30 px high (28 for a worktree), 8 px sides, radius 8; `selected` background when active, `hover` otherwise on hover. Children 9 px apart, vertically centered.
  - **Color stripe:** only when the space has a color: 3 px wide, 2 px from the left edge, 8 px from top and bottom, radius 2. Not on worktree rows.
  - **State slot (18 px square, centered):** working, the ring spinner 11 px in `working`, turning every 900 ms; blocked or error, a 7 px `blocked` dot; done, a 12 px check in `done`; idle (or anything else), a 7 px ring, 1.5 px `idle` (else `dim`) border.
  - Worktree rows are indented 22 px before the slot.
  - **Linear chip,** when the space's issue state is known: 18 px high, 5 px sides, radius 5, 4 px gap; border the state's color at `55` alpha, background at `1f` (else `border` / `raised`); the state icon 11 px in the state's color, then the identifier 11 px semibold `foreground`.
  - **Name:** `foreground`, truncated at 150 px (120 with a chip). With a chip, the name without the identifier and the separators after it (`ENG-142: fix` → `fix`).
  - **Git tail,** right-aligned, 11.5 px `dim`, 5 px gap: the repo (not on worktrees, not when it equals the name) and the branch (not when it equals the name, not `main`/`master`), joined by ` · `; `↑n` in `ansi2`, `↓n` in `ansi1`.
- **Without machine-switch, more than one machine known:** a 26 px header per machine (this computer first, then by name), 8 px sides, 7 px gap, 11 px `dim`, `hover` background: a 13 px `monitor`/`server` icon, the name ("This computer"), its state when not connected, a `+` (new space there), a 6 px dot (`ansi2` connected, else `dim`) and an 11 px chevron (down, right when folded).
- **No spaces there:** one 28 px row, 12 px `dim`: a 12 px `plus` and "New space"; `hover` turns it `foreground`.
- **Narrow sidebar (< 160 px):** 26 px rows of the space's number and `●` (blocked, done) or `·` in the status color; active on `raised`.
- **Right-click menu (`space-menu`, popup 210 × up to 440):** Herdr's space items (Rename, New/Open/Remove worktree, Close) with their icons and keys, a divider, then a row 10 px sides, 4 px top, 7 px bottom, 7 px gap: six 16 px round swatches (`#22a99a`, `#e3814f`, `#5b8def`, `#8b7cf6`, `#3fae6a`, `#d9649a`) and a `×` circle (1 px `input` border, 10 px `dim`). The space's color is ringed with a 2 px `foreground` border; `×` is ringed when it has no color. Last a `+` circle (11 px `dim`) that opens the picker under the row; filled with the space's color, and ringed, when that color came from the picker.
- **Picker** (10 px sides, 8 px bottom, 8 px gap): 24 swatches of 16 px, 6 px apart (hues 0, 30, 50, 140, 175, 210, 255, 300 at 65 % saturation, lightness 72, 58, 44 %), then a 16 px preview dot and a 26 px `#rrggbb` field (radius 6, `input` border, `accent` while typing). A swatch writes its hex; the field writes a full hex as it's typed, Enter closes the menu. While typing, ↑/↓/Enter don't move the menu.

## How it behaves

- **States:** from each space's `status` (Herdr sums its agents): working, blocked, done, idle as above; `error` draws as blocked. The spinner is native (`svg().spin(900)`), no timer.
- **Click** a row: select the space. **Drag** onto another row of the same machine: a 2 px `accent` line shows where (below when moving down, above when moving up), the dragged row dims to 50%, release moves it (`space.move`). **Right-click:** the space menu. **Click the Linear chip:** the issue opens in the Linear mod's window (`mod.open {name: "linear"}`, then `linear.open:<ID>` 400 ms later) when `mods/linear` is installed, else in the browser (the issue's `url`); the row isn't selected.
- **Colors:** shared `"space.colors"` = `{ spaceId: name | "#rrggbb" }`, shared with Herdr's picker and spaces. A space without an entry gets `#7c818b` (no color) on the next render, so Herdr's picker (which gives a color to any space without one) leaves it alone; the picker draws `#7c818b` as a grey avatar. This list draws no stripe for it. An entry is never dropped because its space isn't listed: right after start a remote machine's spaces aren't there until it connects, and dropping them lost their colors on every restart. Only `#7c818b` entries of unlisted spaces are dropped. Swatches write the color name and the menu stays open; `×` writes `#7c818b`.
- **One session or all:** when a block of the layout shows `machine-switch`, the list shows only spaces whose `machine` equals shared `"machine.view"` (null: this computer); an unreachable one shows its state instead of rows. Otherwise every machine, grouped as above; a header click folds it (kept in `localStorage` `machines.folded`; folded machines aren't refreshed by the 60 s fallback).
- **Git:** a space's place is its first terminal's live folder, else where it was made. `git status --porcelain=v2 --branch` (no optional locks, fsmonitor off) and `git rev-parse` there, through `exec {cwd, on}`: when a place is new or a space moved there, when a command or agent turn in it ends, when its machine comes back, on local repos' `.git` changes (`watchFolder`), and every 60 s while the window is active. One at a time. A worktree is listed under the first space on its main checkout. A new space waits up to 1.5 s for its git so it doesn't jump.
- **Linear:** identifiers (`[A-Za-z][A-Za-z0-9]{0,9}-\d{1,6}`, upper-cased, leading zeros dropped) in each space's name, else its branch. When there are any: the key is looked up once (`LINEAR_API_KEY` from `sh -lc`, then the two credential files); none found stops it until restart. Then one GraphQL request for all of them, grouped by team, again when the set changes or after 90 s; a 401 stops it. Unknown identifiers (`UTF-8`) get no chip. States are shared as `"linear.states"` (`{ "ENG-142": { name, type, color, url } }`) and read from there at start.
- **State icons:** by state type: backlog, triage, unstarted, started, completed, canceled (duplicate draws as canceled, unknown as unstarted); a started state whose name has "review" or "리뷰" draws `state-review`.

## Built from

- `mods/session-spaces`: a `mod.clone` of Herdr's `spaces` (0.1.11). `main.js` (the list), `linear.js` (key lookup, identifiers, the request, state icons), `icons/` (`spinner`, `check`, `state-*` copied from the Linear mod).
- `mods/space-menu`: a `mod.clone` of Herdr's `menu` (0.1.12) with the swatch row under a space's items, opened by `session-spaces` as `openOverlay({ mod: "space-menu", kind: "popup", width: 200, height: 300, props: { tab: "space", target: { id } } })`.
- Shipped mod used: `workspace-chrome` (`plus`, `monitor`, `server`, `chevron-*` icons).
- `herdr`: `spaces`, `machines`, `panes`, `layout`, `window`, `exec`, `hc` (`machine ls --json` every 15 s), `watchFolder`, `run` (`space.select`, `space.move`, `space.new {machine}`), `openOverlay`, `shared`/`setShared`, `readTextFile`, `home`, `theme`; `fetch` to `api.linear.app`.
- `gpui`: `svg().spin()`, `overflow_y_scrollbar().mode("always")`.

## Watch out

- Never store a color name Herdr's mods don't know (`"none"`): the picker turns it into an undefined background and fails to render. Use a `#rrggbb`.
- A space without a `space.colors` entry gets one from Herdr's picker the next time it opens; that's why "no color" is written down.
- Mods can't import each other's files: the Linear code and its icons are copies. Icon paths are `mods/session-spaces/...`; a clone under another name has to rewrite them (`mod.clone` does).
- Detect machine-switch from the layout (`content.mod` of any block), not from `machine.view`: that key may be left over from a layout that had it.
- The status of a space is Herdr's, not this list's: a space with no agents is idle.

## Done when

- `herdr-app check` passes for both mods.
- A space whose agent is working shows the spinner; when it asks for permission, a red dot; when it finishes, a check; with no agent, an empty ring.
- Right-click → a swatch: the stripe appears at once and the swatch is ringed; `×`: the stripe goes. After a restart the colors are still there, also on a remote machine's spaces. Herdr's picker (`⌘P`) still opens and shows that space with a grey avatar.
- Without a Linear key, no chips and no request (and nothing breaks); with one, a space named `ENG-142 …` shows the chip and the name without `ENG-142`.
- In a layout without machine-switch: a header per machine once a remote machine is known, folding works, `+` makes a space there. With it: only the chosen session's spaces.
