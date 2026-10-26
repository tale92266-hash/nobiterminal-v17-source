# NobiTerminal v17 — full source snapshot

This repo is the **complete patched app source tree for NobiTerminal v16**
(the version shipped as the v17 APK on 2026-10-26). It exists so the app's
source can be browsed, understood, and edited by hand in one place.

> This snapshot is for viewing and editing. It is **not** the build route.
> The real build route is the generator repo
> (`tale92266-hash/nobiterminal`): it clones upstream, applies the Nobi
> patches, renames the package, builds the bootstrap and the app in GitHub
> Actions, and produces the APK. Edits made here do **not** flow into
> generator builds automatically — to ship an edit, it must become a patch
> in the generator repo.

## What this snapshot was made from (NEW pattern — plugin + base patches)

- **Base:** upstream `termux/termux-app`
  `@8629e632fcb95da272221be327db653fb24befe9` (2026-09-26).
- **Patches, in generator order:** first all 33 Nobi plugin patches from
  generator branch `v17-nobicode-ui`
  `@815a495d2beaf5a0b4956bfe7fb1de49977d7596`
  (built on v16-minimal-batch `@dd8584cf`; v17 adds exactly four new
  app patches, w20, w22, w23 and w24 — there is NO w21 patch, see
  below), then the **base patches** from the generator repo's
  `f-droid-patches/app-patches/` dir — the same order the generator
  uses (`install_plugin` first, then `patch_apps`). All applied with GNU
  `patch -p1`. Plugin phase: 0 `.rej`, 0 `.orig`. Base phase: 0 `.rej`;
  `.orig` files from offset/fuzz were logged and deleted, exactly like
  the generator's `apply_patches` does.
- **Shipped v17 APK for cross-reference:** 35,490,345 bytes,
  SHA-256 `bd761b26c05fa23754fa9755ddf7003e57168444ed59d7cfbcb73d47af4f13a2`,
  generator run `40365941867` (cache HIT build on the already-seeded
  bootstrap — no new seed was published),
  signer
  SHA-256 fingerprint
  `B6:DA:01:48:0E:EF:D5:FB:F2:CD:37:71:B8:D1:02:1E:C7:91:30:4B:DD:6C:4B:F4:1D:3F:AA:BA:D4:8E:E5:E1`.

### The 33 plugin patches, in apply order

1. `nobiterminal-app-name.patch`
2. `nobiterminal-drawer-default.patch`
3. `nobiterminal-neon-default.patch`
4. `nobiterminal-settings-suite.patch`
5. `nobiterminal-tabs-bar.patch`
6. `nobiterminal-v5-core.patch`
7. `nobiterminal-v5-drawer.patch`
8. `nobiterminal-v5-icon-splash.patch`
9. `nobiterminal-v5-nobicode-icons-a.patch`
10. `nobiterminal-v5-nobicode-icons-b.patch`
11. `nobiterminal-v5-nobicode-res.patch`
12. `nobiterminal-v5-nobicode-src.patch`
13. `nobiterminal-v5-tabs-rename.patch`
14. `nobiterminal-v6-batch.patch`
15. `nobiterminal-v7-batch.patch`
16. `nobiterminal-v7-fix.patch`
17. `nobiterminal-v7-fix2.patch`
18. `nobiterminal-v8-extrakeys.patch`
19. `nobiterminal-v9-drawer-notify.patch`
20. `nobiterminal-w10-editor-permissions.patch`
21. `nobiterminal-w11-followup.patch`
22. `nobiterminal-w12-fix-duplicate-permissions.patch`
23. `nobiterminal-w13-fix-vectors-kotlin.patch`
24. `nobiterminal-w14-autorepo.patch`
25. `nobiterminal-w15-keys-settings-theme.patch`
26. `nobiterminal-w16-settings-follow-terminal.patch`
27. `nobiterminal-w17-settings-toolbar-identity.patch`
28. `nobiterminal-w18-drawer-hides-keyboard.patch` (2,377 bytes)
29. `nobiterminal-w19-dialog-hides-keyboard.patch` (5,902 bytes)
30. `nobiterminal-w20-gutter-tokens.patch` (1,434 bytes)
31. `nobiterminal-w22-file-breadcrumb.patch` (3,672 bytes)
32. `nobiterminal-w23-run-output-errors.patch` (2,218 bytes)
33. `nobiterminal-w24-secondary-cyan.patch` (1,589 bytes)

### The 7 base patches — 5 applied, 2 out of scope (read this)

The generator applies 7 base patches after the plugin patches. Five of
them touch files inside this `termux-app` tree and ARE applied here, in
lexical order:

1. `auto-hide-extra-keys.patch` — extra-keys toolbar hides when a
   hardware keyboard is connected (new preference
   `shouldExtraKeysBeVisible` / `isExtraKeysOnlyIfNoHardware`).
2. `increase-gradle-memory.patch` — `gradle.properties` JVM heap
   2048M → 8192M (build-only setting).
3. `local-bootstraps.patch` — removes the `app` module's
   `externalNativeBuild` ndkBuild block (and APK splits) from
   `app/build.gradle`, plus a `TermuxInstaller.java` change. This is why
   the shipped APK contains NO `libtermux-bootstrap.so`: the CI tree
   does not build it either. (This resolves the old "app module wires
   ndkBuild but the APK has no bootstrap .so" wrinkle from v11–v15
   snapshots — those trees lacked this base patch, so they still showed
   the app ndk wiring that CI removes.)
4. `local-termux-am-library.patch` — `termux-shared/build.gradle` uses
   `project(":termux-am-library")` and `settings.gradle` includes
   `:termux-am-library`. NOTE: the module's sources themselves are NOT
   in this tree — CI moves them into `termux-app/` during its download
   step, before patching. So `settings.gradle` references a module
   whose sources arrive via CI, not via this snapshot.
5. `multi-window.patch` — multi-window support: `TerminalView`
   `attachSession` now returns the previous session (was boolean),
   `detachSession()` added, manifest + activity/service/session
   changes. NOTE: this patch is a concatenation of several upstream
   commits, including an add-then-remove pair for a `mAttached` field
   in `TerminalSession.java` — net effect on that file is zero, which
   is the correct CI result, not a missing change.

Two base patches are NOT applied here, and that is deliberate, not an
oversight — they contain ZERO `termux-app/` targets, so there is nothing
in this tree for them to patch:

6. `local-maven.patch` — touches only `termux-tasker/`, `termux-float/`,
   `termux-api/`, `termux-widget/` build files (sibling apps in the
   generator's `termux-apps-main` dir). Those apps are not built in
   NobiTerminal builds (all disabled) and are not part of this
   `termux-app`-only snapshot.
7. `local-termux-gui.patch` — touches only `termux-gui/app/build.gradle`
   (sibling app, also not built here).

So this tree is the full CI tree **for the `termux-app` module**: every
base-patch effect that lands inside `termux-app` is present; the two
sibling-app-only patches have no effect on this module by construction.

## What's new in v17 (vs v16)

- **w20 — gutter tokens (NobiCode):** editor gutter/line-number colors
  move to cyan tokens in `Themes.kt`, matching the Nobi navy + cyan
  identity instead of default greys.
- **w22 — file breadcrumb (NobiCode):** the editor header shows a
  file breadcrumb refreshed by `refreshFileBreadcrumb`
  (`MainActivity.kt` + `activity_main.xml` layout).
- **w23 — run output errors (NobiCode):** run output surfaces an
  "Errors detected" hint so a failed run is visible at a glance
  (`MainActivity.kt`).
- **w24 — secondary cyan (NobiCode):** secondary text/tints use
  `#CC22D3EE` cyan (`MainActivity.kt` + `values/colors.xml`).
- **NO w21 patch:** the save/dirty indicator chain (the would-be w21
  item) was verified to already exist, so no patch was made for it —
  the numbering skips w21 deliberately, it is not a missing file.
- **Base patches (above)** — same 5 applied, same 2 sibling-only
  patches out of scope, as in v16.

## Dead PNG nuance (same as v11–v16)

The 11 legacy launcher PNGs (`mipmap-*/ic_launcher.png`,
`mipmap-*/ic_launcher_round.png`) plus `drawable/banner.png` are
**present in this tree** (upstream files, 11/11 counted) but **absent
from the shipped v17 APK** — the generator deletes them at build time
(`remove_dead_nobi_assets()`, before Gradle). Do NOT claim tree == APK
for these 11 files.

## Honest limitations (same as v11–v16)

- **Package name:** the tree is `com.termux`, not `com.nobiterminal`.
  The rename is a generator build-time step (after patching). Edit
  `com.termux` paths. Building this tree directly produces a `com.termux`
  app.
- **No bootstrap:** `app/src/main/assets/` is absent. CI builds the
  bootstrap from `termux-packages` (v17 build used HEAD `157e965980b7`,
  cache HIT — no new seed was published).
- **Signing:** `app/testkey_untrusted.jks` is in the tree; its cert
  fingerprint matches the shipped v17 APK signer exactly, so a debug
  build from this tree installs over the shipped v17.
- **ACS:** `main` will NOT build directly in a phone IDE. With the base
  patches, TWO modules still wire `externalNativeBuild` ndkBuild
  (`terminal-emulator` → `libtermux`, `termux-shared` →
  `liblocal-socket`) — the `app` module no longer does (see
  `local-bootstraps` above). The phone IDE cannot run the inner
  `ndk-build` (v4 failed at `:termux-shared:configureNdkBuildDebug`,
  exit 126). No `acs-prebuilt` branch exists for v17. The shipped v17
  APK contains exactly `libtermux.so` + `liblocal-socket.so`
  (× 4 ABIs), matching the two wired modules. Gradle wrapper jar IS in
  the tree, so ACS import does not hang on the wrapper.
- **Upstream README:** upstream's `README.md` was renamed to
  `README_UPSTREAM.md` (content byte-identical); this file is the
  snapshot doc.
- **Binary hunks:** GNU `patch` strips git binary hunks, exactly like
  CI — patch-intended binary files are absent here, as in the CI tree.
- **Edits here don't ship:** to ship an edit, it must become a patch
  in the generator repo.
