# Arch Updater (background edition)

Fork of [yuuto/arch-updater](https://github.com/noctalia-dev/noctalia-plugins) for Noctalia 5.
Checks pacman, AUR and Flatpak updates — and, unlike the original, **updates in
the background**: one click on *Update*, a polkit password dialog, and the run
is fully non-interactive. The panel shows a live tail of the update log and a
progress bar instead of a terminal window.

## How the update runs

- Repo packages: `pacman -Syu --noconfirm` through `pkexec` (Noctalia's own
  polkit agent shows the password dialog).
- AUR: the helper (paru/yay) runs as your user with `--sudo pkexec`, review
  prompts disabled (`--skipreview` / `--answer* None`); every remaining
  question gets its default answer. polkit may ask for the password once more
  when the built packages are installed.
- The runner is spawned fully detached and logs to
  `<plugin data dir>/update.log`, so a shell restart does not interrupt the
  update — the engine re-attaches to the log on startup.
- If a run cannot proceed non-interactively it stops cleanly with a non-zero
  exit code before touching the system; the panel then offers **Retry in
  terminal**, where prompts and the PKGBUILD review work as usual.

## Differences from upstream 1.1.0

- Background update with live log + progress (panel and bar widget percent).
- Fixed the news timer bug (upstream refetches the Arch news feed every 10
  seconds, which trips Noctalia's plugin health watchdog and gets the plugin
  auto-disabled ~50 s after start).
- Ignore management: `pacman.conf`'s `IgnorePkg` packages no longer count as
  pending; an expandable **Ignored** section shows all ignore sources, and
  every package row has an ignore button (panel-managed, removable).
- One polkit password per run: the panel offers to install a narrow
  `auth_admin_keep` rule for `pkexec` + `/usr/bin/pacman` (also in
  `polkit/49-arch-updater-pacman.rules` for manual install).
- Update history strip + rollback (see below).
- Removed the activity graph and the duplicated footer *Check Updates* button;
  the freed space hosts the log view. Download size, reboot hint and Arch news
  stay.
- Russian translation.

## Update history and rollback

The strip at the bottom of the panel has one segment per recorded run
(last 15), oldest on the left; rollbacks get their own segments. Hover shows
the date and package count, click opens the run's package list. Each package
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

`pacman-contrib` (checkupdates), `polkit` (pkexec) + a running polkit agent
(Noctalia 5 has one built in), optionally `paru`/`yay` and `flatpak`.
