# Arch Updater (background edition)

Fork of [yuuto/arch-updater](https://github.com/noctalia-dev/noctalia-plugins) for Noctalia 5.
Checks pacman, AUR and Flatpak updates and installs them in one of two modes
(the **Update mode** setting):

- **In a terminal window** (default, like the original plugin): *Update*
  opens a terminal running the usual interactive upgrade — prompts and the
  PKGBUILD review work as normal.
- **In the background**: one click, a polkit password dialog, and the run is
  fully non-interactive. The panel shows a live tail of the update log and a
  progress bar instead of a terminal window.

Either way the output lands in the same log, so the panel's live tail,
progress and the update history work in both modes.

## How the background update runs

- Repo packages: `pacman -Syu --noconfirm` through `pkexec` (Noctalia's own
  polkit agent shows the password dialog).
- AUR: the helper (paru/yay) runs as your user with `--sudo pkexec`, review
  prompts disabled (`--skipreview` / `--answer* None`); every remaining
  question gets its default answer. polkit may ask for the password once more
  when the built packages are installed.
- The runner is spawned fully detached and logs to
  `<plugin data dir>/update.log`, so a shell restart does not interrupt the
  update — the engine re-attaches to the log on startup.
- A background run that stops growing its log for 30 minutes is marked as
  stuck; terminal runs are exempt (log silence there may just be you reading
  a PKGBUILD diff).
- If a background run cannot proceed non-interactively it stops cleanly with
  a non-zero exit code before touching the system; the panel then offers
  **Retry in terminal**, where prompts and the PKGBUILD review work as usual.

## Differences from upstream 1.1.0

- Optional background update mode with live log + progress (panel and bar
  widget percent); the default stays a terminal window, like upstream.
- Fixed the news timer bug (upstream refetches the Arch news feed every 10
  seconds, which trips Noctalia's plugin health watchdog and gets the plugin
  auto-disabled ~50 s after start).
- Ignore management: `pacman.conf`'s `IgnorePkg` packages no longer count as
  pending; an expandable **Ignored** section shows all ignore sources, and
  every package row has an ignore button (panel-managed, removable).
- One polkit password per run: the panel offers to install an
  `auth_admin_keep` rule for `pkexec` + `/usr/bin/pacman` (also in
  `polkit/49-arch-updater-pacman.rules` for manual install). **Scope**: after
  each authentication the rule keeps the authorization for ~5 minutes, and it
  covers *any* `pkexec pacman …` call made from an active local session of a
  `wheel` member — not just this plugin's calls (comparable to a NOPASSWD
  sudoers line with sudo's timeout). It is opt-in, disclosed in the install
  button's tooltip, and removable with
  `sudo rm /etc/polkit-1/rules.d/49-arch-updater-pacman.rules`.
- Update history strip + rollback (see below).
- An AUR or Flatpak check that cannot complete (unreachable AUR, remote
  down) fails only itself: the remaining sources are still checked, and the
  panel reports the failed source under the headline instead of counting it
  as up to date. Only a failure of the pacman/checkupdates check stops the
  run.
- The activity graph is opt-in now (**Show activity graph**, off by default);
  when enabled it keeps its upstream behavior and data file and sits right
  above the update history strip. The duplicated footer *Check Updates*
  button is removed; the freed space hosts the log view. Download size,
  reboot hint and Arch news stay.
- Russian translation.

## Update history and rollback

The strip at the bottom of the panel has one segment per recorded run
(last 15), oldest on the left; rollbacks get their own segments. Hover shows
the date and package count, click opens the run's package list. Before a
run is recorded, `pacman -Q` confirms which packages actually changed:
anything declined during an interactive terminal run (its installed version
never moved) is left out of the entry. Each package
row has a rollback button (a second click confirms), the header rolls back
the whole run in one transaction.

Rollback installs the old package files straight from the caches
(`/var/cache/pacman/pkg`, paru's / yay's build dirs) with
`pkexec pacman -U`: the chosen version is installed directly, no need to
step through intermediate upgrades. Dependencies updated in the same run
ride along in the same transaction. `--nodeps` is never used, so a downgrade
that would break another package's versioned dependencies makes pacman
refuse the whole transaction before anything changes. Versions whose files
are gone from the cache are greyed out. The **Ignore packages after a
rollback** setting (off by default) keeps the next check from immediately
offering the rolled-back packages again.

**A quirk with `-git` (devel) packages:** after you roll one back, the next
check will usually *not* list it as an update. paru/yay track devel packages
by the last **built commit** (`~/.local/state/paru/devel.toml`), not by the
installed version — and that commit still matches the upstream repo, so the
check sees nothing new. The package reappears once upstream gets a new
commit. To move forward before that, open the rollback's own segment on the
strip and roll it back (installing the newer version from the cache again),
or run `paru -S <package>` by hand. Regular repo packages are unaffected: a
rolled-back version shows up as a pending update on the very next check.

## Requirements

`pacman-contrib` (checkupdates, pactree), `polkit` (pkexec) + a running
polkit agent (Noctalia 5 has one built in), optionally `paru`/`yay` and
`flatpak`. Shelled-out helpers, all in a base Arch install: `coreutils`
(`date`, `head`, `install`, `rm`, `tail`, `tee`, `test`, `uname`, `wc`),
`awk`, `grep`, `less`, `sed`, `sh`, `xdg-open`.
