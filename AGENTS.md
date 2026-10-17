# AGENTS.md

## What this repo is

Not code. It holds a single machine-generated keymap export for a **Keychron B1 Pro (ANSI)** keyboard:

- `Keymap-Keychron B1 Pro ANSI-17-18-28.json` — the only tracked file. Filename contains spaces; quote it in shell commands.

There is no build, test, lint, CI, or package config. There is nothing to run.

## Editing the keymap

- The file is an export from the Keychron Launcher web app (launcher.keychron.com). The authoritative edit flow is: edit in the Launcher, re-export, replace this file. Hand-edits are possible but must preserve the exact structure below or the Launcher will reject the import.
- Structure invariants (verified against the current file):
  - Top level: `id`, `keymap`, `version` (`"1.0.0"`), `MD5`.
  - `keymap` is an array of exactly **4 layers** (0–3).
  - Each layer is a flat list of exactly **154** objects of shape `{"col": int, "row": int, "val": int}` — this is rows 0–7 × cols 0–17 plus row 8 cols 0–9. Do not add, drop, or reorder entries; positions are identified by `col`/`row`, and every layer must contain the same 154 positions.
- `val` is a Keychron/HID keycode, not a character. Layer 0 uses raw HID usage IDs (e.g. `4`=A, `225`=Left Ctrl); layers 1–3 additionally use Launcher-encoded values in the 32xxx range (e.g. `32256`, `32279`). `val: 0` marks an unused position. Consult the Launcher keycode table rather than guessing values.
- `MD5` is `md5(JSON.stringify(keymap))` — the MD5 of the `keymap` array serialized as compact JSON (no spaces, each object's keys in `col`,`row`,`val` order). Verified against the current file. After any hand-edit, recompute it or the Launcher may reject the import:
  - `jq -cj '.keymap' "<file>" | md5sum` — update the `MD5` field with this value.
- After any edit, validate syntax and checksum:
  - `jq . "<file>" > /dev/null`
  - `test "$(jq -cj '.keymap' "<file>" | md5sum | cut -d' ' -f1)" = "$(jq -r '.MD5' "<file>")" && echo OK`
