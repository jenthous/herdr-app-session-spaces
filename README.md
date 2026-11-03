# Session spaces for the Herdr app

A spaces sidebar for the [Herdr app](https://herdr.dev) that shows one session (machine) at a time, cloned from Herdr's built-in `spaces` mod (0.1.11).

## Features

- **One session.** Only the spaces of the machine chosen in the `machine-switch` mod (shared `machine.view`; none means this computer).
- **Space state.** Before each name, whether its agents are working, blocked, done or idle; after it, the repo, branch and ahead/behind of its first terminal when unusual.
- **Colors and worktrees.** A stable color stripe per space (shared `space.colors`), worktrees under their checkout, grouped by machine when several are known.
- **Linear issues.** A space whose name (or branch) has a Linear identifier like `ENG-142` shows it as a chip with the issue's state icon in the state's color, and the name without the identifier. States are fetched every 90 s, or when the set of identifiers changes, for spaces on every machine; they're shared as `linear.states`.
- Click to switch, right-click for the space menu, drag to reorder.

## Install

Copy `mods/session-spaces` into `~/.config/herdr-app/mods/` and use it in a layout in place of `spaces`.

## Needs

- The [Linear mod](https://github.com/jenthous/herdr-app-linear) installed as `mods/linear`: the state icons are drawn from `mods/linear/icons`. Without it the chips have no icon.
- A Linear API key for the chips: `LINEAR_API_KEY` in the login environment, or the credentials file the Linear mod saves (`~/.config/herdr-linear/credentials`).
- Optional: the `machine-switch` mod (which session to show) and `workspace-chrome` (the + icon).

## License

MIT for the changes here. It is a clone of Herdr's `spaces` mod; that code stays under Herdr's terms.
