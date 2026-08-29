# Changelog

All notable changes to Arch Updater are documented here. The panel's changelog
icon (the history icon next to Check) shows this same file.

## 2.5.0 - 2026-08-29

- Added: a maintenance section under the package list — package cache size with one-click pruning (paccache, configurable versions to keep), orphaned packages with confirmed removal (pacman -Rns), and a firmware-updates shortcut when fwupd is installed.
- Added: the Arch keyring is refreshed right before each update run (toggleable), so packages signed with new keys don't fail after a long gap between updates.
- Added: a free-disk-space check before starting an update; the run is refused with a clear message when the root filesystem is too full.
- Added: background update runs hold a systemd sleep/idle inhibitor, so a closed laptop lid can't interrupt a pacman transaction halfway.
- Added: the update log is scanned after every run for known failure signatures — a failed initramfs generation (critical: warns before you reboot), failed transactions, and package signature errors.
- Merged from upstream 2.0.1: the changelog view, terminal-mode Update closing the panel, and detection of a terminal window closed mid-run.

## 2.4.2 - 2026-08-18

- Fixed: an unreachable AUR or Flatpak no longer aborts the whole check — the failing source is reported in the panel and the rest of the check continues.

## 2.4.0 - 2026-08-17

- Added: update mode setting (terminal window by default, background as an option), equal citizens.
- Added: LC_ALL=C pinning for all parsers, so non-English systems count ignored packages and progress correctly.
- Fixed: the stop-timer no longer kills terminal runs sitting at a PKGBUILD prompt.

## 2.1.0 - 2.3.1 - 2026-08-17

- Added: an Ignored section and per-package ignore buttons, persisted in the plugin's own list.
- Added: a polkit rule (installable from the panel) so one password covers the whole background run.
- Added: update history with per-package and whole-run rollback resolved against the pacman/AUR cache.

## 2.0.0 - 2026-08-16 (fork baseline)

- Forked from yuuto/arch-updater 1.1.0: background update mode with a live log and progress, and the news-timer watchdog fix.

## 1.1.0 - 2026-08-08 (upstream)

- Added: per-source icons in the package list header; an activity graph of pending-update counts.
- Fixed: Dismiss actually clears the pending list; Update closes the panel.

## 1.0.0 - 2026-07-28 (upstream)

- Initial release: check pacman, AUR and Flatpak for updates from the panel and the bar widget.
