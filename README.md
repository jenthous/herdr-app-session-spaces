# Session spaces for the Herdr app

A spaces sidebar for the Herdr app, cloned from Herdr's built-in `spaces` mod (0.1.11). Each space shows what its agents are doing before its name, its Linear issue if it has one, and a color only if you give it one.

![Session spaces](preview.png)

## Features

- **Space state.** Where the avatar was: working spins, blocked (or failed) is a red dot, done is a check, idle an empty ring. Each a different shape, not just a color.
- **Colors when you want them.** Right-click a space for Herdr's items plus a row of six colors and `×` for none. A colored space gets a thin stripe on its left edge; new spaces start with none.
- **Linear issues (optional).** A space whose name (or branch) has an identifier like `ENG-142` shows it as a chip with the issue's state icon in the state's color, and the name without the identifier. Click it to open the issue: in the [Linear mod](https://github.com/jenthous/herdr-app-linear)'s window when it's installed, else in the browser. Needs a Linear API key; without one, nothing is asked or shown.
- **One session or all.** Next to the [machine-switch](https://github.com/jenthous/herdr-machine-rail) mod in a layout, it shows only the session that mod chose. Without it, every machine's spaces under a header per machine (fold it, `+` for a new space there), as Herdr's own list.
- Kept from Herdr's list: worktrees under their checkout; after a name, the repo, branch and ahead/behind of its first terminal when unusual; click to switch, drag to reorder, a compact rail when the sidebar is narrow.

## Install

1. Copy `mods/session-spaces` and `mods/space-menu` into `~/.config/herdr-app/mods/` (the right-click menu is `space-menu`).
2. In your layout, use `session-spaces` in place of `spaces` (clone a shipped layout first: `herdr-app do layout.clone '{"name":"main","as":"my-main"}'`).
3. `herdr-app check ~/.config/herdr-app/mods/session-spaces` and the same for `space-menu`.

Or ask the agent in your Herdr terminal: *"add herdr.dev/mods/jenthous/herdr-app-session-spaces"*.

It needs nothing else: the Linear state icons are in `mods/session-spaces/icons`, and the other icons come from Herdr's shipped `workspace-chrome`.

### Linear key (optional)

`LINEAR_API_KEY` in your login environment, else the first of `~/.config/herdr/plugins/config/jh.linear/credentials` and `~/.config/herdr-linear/credentials` that has one (the files [herdr-linear](https://github.com/jenthous/herdr-linear) and the [Linear mod](https://github.com/jenthous/herdr-app-linear) save).

## What it touches

- **Programs:** `git status` and `git rev-parse` in each space's folder, on that space's machine (`hc exec` for another machine), when its answer could have changed and every 60 s while the window is in front (as Herdr's `spaces`). `hc machine ls` every 15 s. Once, when a space is named after an issue: `sh -lc` to read `LINEAR_API_KEY`.
- **Network:** with a Linear key, `api.linear.app` (one GraphQL request for every space's issue, every 90 s or when the set changes). None otherwise.
- **Files:** reads the Linear credential files above, and whether `mods/linear/mod.json` exists. Writes nothing outside the app's state.
- **Shared state:** `space.colors` (a space's color; `#7c818b` means none, so Herdr's picker doesn't give it one), `linear.states` (the issues' states, for other mods). Reads `machine.view` when machine-switch is in the layout.

## License

MIT for the changes here. `session-spaces` is a clone of Herdr's `spaces` mod and `space-menu` of its `menu` mod; that code stays under Herdr's terms.
