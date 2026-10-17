# AGENTS.md

## What this repo is

Not code. It holds a single machine-generated keymap export for a **Keychron B1 Pro (ANSI)** keyboard:

- `keymap.json` — the only tracked file. Compact single-line JSON (minified `JSON.stringify` style).

There is no build, test, lint, CI, or package config. There is nothing to run.

## Editing the keymap

- The data originates from the Keychron Launcher web app (launcher.keychron.com). The authoritative edit flow is: edit in the Launcher, re-export, replace this file. Hand-edits are possible but must preserve the exact structure below or the Launcher will reject the import. A fresh Launcher export is usually pretty-printed under a different filename (`Keymap-Keychron B1 Pro ANSI-<date>.json`) — when replacing `keymap.json`, minify it and recompute the checksum (see below) so the raw-bytes property holds.
- Structure invariants (verified against the current file):
  - Top level: `id`, `keymap`, `version` (`"1.0.0"`), `MD5`.
  - `keymap` is an array of exactly **4 layers** (0–3).
  - Each layer is a flat list of exactly **154** objects of shape `{"col": int, "row": int, "val": int}` — this is rows 0–7 × cols 0–17 plus row 8 cols 0–9. Do not add, drop, or reorder entries; positions are identified by `col`/`row`, and every layer must contain the same 154 positions.
- `val` is a Keychron/HID keycode, not a character. Layer 0 uses raw HID usage IDs (e.g. `4`=A, `225`=Left Ctrl); layers 1–3 additionally use Launcher-encoded values in the 32xxx range (e.g. `32256`, `32279`). `val: 0` marks an unused position. Consult the Launcher keycode table rather than guessing values.
- `MD5` is `md5(JSON.stringify(keymap))` — the MD5 of the `keymap` array serialized as compact JSON (no spaces, keys in `col`,`row`,`val` order). Verified against the current file. After any hand-edit, recompute it or the Launcher may reject the import:
  - `jq -cj '.keymap' keymap.json | md5sum` — update the `MD5` field with this value.
- Because `keymap.json` is minified in exactly `JSON.stringify` style, the raw byte-slice of the `"keymap":[...]` array in the file hashes directly to the stored `MD5` with no transformation. Pretty-printed exports break this: re-minify before hashing.
- After any edit, validate syntax and checksum:
  - `jq . keymap.json > /dev/null`
  - `test "$(jq -cj '.keymap' keymap.json | md5sum | cut -d' ' -f1)" = "$(jq -r '.MD5' keymap.json)" && echo OK`
