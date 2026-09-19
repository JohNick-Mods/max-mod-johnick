# MAX Mod 26.31.0 RS V8

**Date:** 2026-09-19 | **Tag:** v26.31.0.8

## Changes

- **Fix 2631-E7**: In strict E2E mode (`strictActiveForChat`), messages no longer leak as plaintext while Signal session is not yet established. Real text is replaced with `E2E_STRICT_PENDING_SENTINEL`; key exchange control tokens continue as before. Non-strict E2E is unchanged.

## Previous (V7)

- Removed RuStore independent self-update path (`vt2.smali`)

## Previous (V6)

- Scroll-to-pinned fix, mod update card, incoming call disable, playback speed threshold, Share Logs from mod settings

## Previous (V5)

- RedCode crash fix (A9), anti-delete after restart, transparent theme, KeepAliveWorker, QR scanner, video_toggle E2E calls
