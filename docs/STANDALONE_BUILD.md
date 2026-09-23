# Standalone Windows Build

This repository now includes a packaged Windows executable build for the main CLI.

## Output

After a successful build, the executable is written to:

```text
dist\hiroglyph.exe
```

## What It Bundles

The executable is built from `hiro.py` and includes:

- the current encode/decode CLI
- the `run` command for executing `.glyph` files
- bundled addons from `addons\`
- bundled glyph core resources from `glyph_core.glyph`
- bundled fallback source from `depo\glyph_core_backup_depo.py`

That means the `.exe` can encode, decode, inspect addon support, and run glyphs without requiring a separate Python install on the target machine.

## Build Command

From the repo root:

```powershell
powershell -ExecutionPolicy Bypass -File .\build_standalone.ps1 -Clean
```

Or directly:

```powershell
python -m PyInstaller --clean --noconfirm hiroglyph.spec
```

## Why the Spec Exists

`hiroglyph.spec` does a few important things:

- includes non-Python runtime files like `glyph_core.glyph`
- bundles the `addons\` and `depo\` folders
- excludes unrelated machine-wide packages that can confuse PyInstaller
- keeps the build focused on the actual runtime dependencies for this repo

## Smoke Test

These commands verify the packaged app end to end:

```powershell
.\dist\hiroglyph.exe run --list-addons
.\dist\hiroglyph.exe encode "print('exe smoke test')" --file-type python --key-type password --key-source secret --output exe_smoke.glyph
.\dist\hiroglyph.exe run exe_smoke.glyph --key-type password --key secret
```

Expected result:

- addon types are listed successfully
- `exe_smoke.glyph` is created
- the executable prints `exe smoke test`

## Entry Point Guidance

Use `hiro.py` and `hiroglyph.exe` as the primary entry points going forward.

`glyph_runner.py` remains in the repo as a legacy runner, but the packaged app avoids depending on that older execution path by running glyphs through the current `hiro.py` decoder and the shared addon system.
