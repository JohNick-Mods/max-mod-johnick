# MAX Mod 26.31.0 RS V9

**Date:** 2026-09-20 | **Tag:** v26.31.0.9

## Changes

- **Real fix for 2631-E7** (V8 did not fix it): `MIRROR_VERSION_TAG` was not bumped when building V8, so the mod kept announcing its old version via the E2E CAP protocol — this caused the stuck "peer has outdated mod version" banner and asymmetric encryption. Bumped to 26.31.0.9, statically verified inside classes.dex. Also added `clearE2EIssue` on the incoming CAP path (the banner used to clear only on the next outgoing send attempt).
- **Fix for 2631-U8**: race between `onKey`/`onCancel` in `SubMenuBackKeyListener` could trigger `reopenRootMenu()` twice for one Back press, resetting caps/emoji/switch-tint styling in the mod settings screen. Added a single-fire guard.

## Verification

- Static: single `26.31.0.9` version string in classes.dex (no stale `.6`/`.7`/`.8`).
- Device (Redmi): app launches without crash, 4 subsection→Back cycles with no style regression.
- Not verified this session: live 2-device E2E round-trip (needs a second SIM).
