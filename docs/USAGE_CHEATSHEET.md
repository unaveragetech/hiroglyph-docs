# Crypto-Hieroglyphic Encoder - Usage Cheatsheet

## Preferred Entry Points

Use `hiro.py` for the full workflow:

```bash
python hiro.py encode ...
python hiro.py decode ...
python hiro.py run ...
```

The standalone Windows build exposes the same commands:

```powershell
.\dist\hiroglyph.exe <command> [options]
```

`glyph_runner.py` still exists as a legacy runner, but current docs and the packaged executable use `hiro.py run`.

## Encode

```bash
# System-bound glyph
python hiro.py encode --input script.py --key-type system --output script.glyph

# Password-protected glyph
python hiro.py encode --input script.py --key-type password --key-source "mypassword" --output script.glyph

# AES-key glyph
python hiro.py encode --input script.py --key-type aes --key-source "your_aes_key_here" --output script.glyph

# Encode inline text
python hiro.py encode "Hello World" --key-type system

# Encode HTML
python hiro.py encode --input page.html --html --key-type system --output page.glyph

# Encode with formatted art output
python hiro.py encode --input script.py --key-type system --output script.glyph --width 20 --art --verbose
```

## Decode

```bash
# Decode from a glyph file
python hiro.py decode --input script.glyph --key-type system

# Decode with password
python hiro.py decode --input script.glyph --key-type password --key-source "mypassword"

# Decode with AES key
python hiro.py decode --input script.glyph --key-type aes --key-source "your_aes_key_here"

# Decode to a file
python hiro.py decode --input script.glyph --key-type system --output decoded.txt

# Decode using exported system variables
python hiro.py decode --input script.glyph --key-type system --system-vars-file my_system.json
```

## Run

```bash
# Run with system key
python hiro.py run script.glyph

# Run with password
python hiro.py run script.glyph --key-type password --key "mypassword"

# Run with AES key
python hiro.py run script.glyph --key-type aes --key "your_aes_key_here"

# Prefer in-memory execution where supported
python hiro.py run script.glyph --key-type password --key "mypassword" --no-temp

# Use exported system variables on another machine
python hiro.py run script.glyph --system-vars-file my_system.json

# Inspect addon support
python hiro.py run --list-addons
python hiro.py run --addon-info
```

## Key Management

```bash
# Generate an AES key
python hiro.py generate-key

# Generate from a password
python hiro.py generate-key --password "MyStrongPassword123"

# Save generated key data
python hiro.py generate-key --password "securepass" --output mykey.json

# Show the current system fingerprint flow
python hiro.py show-system-key

# Export system variables for controlled cross-machine execution
python hiro.py export-system-vars --output system_vars.json
```

## Common Workflows

```bash
# Python script workflow
python hiro.py encode --input hello.py --key-type system --output hello.glyph
python hiro.py run hello.glyph

# HTML workflow
python hiro.py encode --input page.html --html --key-type system --output page.glyph
python hiro.py run page.glyph

# Cross-system workflow
python hiro.py export-system-vars --output my_system.json
python hiro.py encode --input script.py --key-type system --output script.glyph
python hiro.py run script.glyph --system-vars-file my_system.json

# Standalone executable workflow
.\dist\hiroglyph.exe encode --input hello.py --key-type password --key-source "secret" --output hello.glyph
.\dist\hiroglyph.exe run hello.glyph --key-type password --key "secret"
```

## Option Summary

`hiro.py` global:
- `--explain`, `-e`
- `--verbose`, `-v`

`encode`:
- `--input`, `-i`
- `--output`, `-o`
- `--key-type`, `-k`
- `--key-source`, `-s`
- `--width`, `-w`
- `--art`
- `--file-type`, `-t`
- `--html`

`decode`:
- `--input`, `-i`
- `--output`, `-o`
- `--key-type`, `-k`
- `--key-source`, `-s`
- `--system-vars-file`

`run`:
- `--key-type`
- `--key`
- `--system-vars-file`
- `--no-temp`
- `--cleanup-delay`
- `--keep-temp`
- `--list-addons`
- `--addon-info`

## Notes

- `--key-source` is used by `encode` and `decode`.
- `--key` is used by `run`.
- `--no-temp` depends on the addon and the local runtime for non-Python content.
- Keep exported system variable JSON files secure; they enable controlled cross-machine decryption and execution.
