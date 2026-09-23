# What Is a Glyph?

A `.glyph` file is an encrypted, visually-encoded blob of data that looks like a stream of Unicode art symbols but secretly holds executable code, scripts, or any other content. To the naked eye it is indistinguishable from a decorative pattern; to the system that created it, it is a fully runnable program.

---

## Table of Contents

- [The Core Idea](#the-core-idea)
- [The Symbol Set](#the-symbol-set)
- [File Format Anatomy](#file-format-anatomy)
- [Annotated Example — Hello World](#annotated-example--hello-world)
- [How Encoding Works Step by Step](#how-encoding-works-step-by-step)
- [How Decoding Works Step by Step](#how-decoding-works-step-by-step)
- [What Gets Encrypted](#what-gets-encrypted)
- [Key Types and How They Affect Glyphs](#key-types-and-how-they-affect-glyphs)
- [Per-File Salt (v2 / HIR2 Format)](#per-file-salt-v2--hir2-format)
- [Error Correction Symbols](#error-correction-symbols)
- [A Short Python Script as a Glyph](#a-short-python-script-as-a-glyph)
- [A Batch Script as a Glyph](#a-batch-script-as-a-glyph)
- [Why It Looks Like Art](#why-it-looks-like-art)
- [Glyph vs Plain Encryption](#glyph-vs-plain-encryption)

---

## The Core Idea

Traditional encryption stores ciphertext as binary blobs or Base64 strings that are immediately recognisable as encrypted data. A glyph file instead maps every byte of ciphertext to a sequence of hand-picked Unicode symbols that humans associate with calligraphy, hieroglyphics, or decorative borders.

There are two separate layers:

1. **Cryptographic layer** — AES-based stream cipher, PBKDF2 key derivation, SHA3-512 integrity hash.
2. **Visual encoding layer** — each 3 bits of ciphertext becomes one of 8 hieroglyphic symbols.

These two layers are completely independent. The visual encoding does not add security on its own; the security comes entirely from the cryptographic layer. The visual encoding adds *plausible deniability* — an encrypted file can be passed off as an artistic text snippet without obviously looking like ciphertext.

---

## The Symbol Set

Eight **data symbols** encode 3 bits each:

```
Symbol │ Binary │ Decimal
───────┼────────┼────────
  •    │  000   │   0
  /    │  001   │   1
  \    │  010   │   2
  |    │  011   │   3
  ─    │  100   │   4
  │    │  101   │   5
  ╱    │  110   │   6
  ╲    │  111   │   7
```

Four **error-correction symbols** carry parity data (they never encode payload bits):

```
  ○   □   △   ◇
```

All 12 symbols are chosen to look visually consistent — thin strokes, no letters, no digits — so any mixture of them looks like an intentional Unicode art piece.

---

## File Format Anatomy

A `.glyph` file is a single line of symbols with no filename, no extension hint, and no human-readable header.

### Version 1 (HIR1 — legacy)

```
[data symbols and error-correction symbols interleaved]
```

The IV and a fixed salt are prepended as part of the encrypted payload. All files encoded with the same key on the same system produce the same salt — making v1 files theoretically susceptible to rainbow table pre-computation.

### Version 2 (HIR2 — current default)

```
[magic 4 bytes: HIR2][16-byte random salt][data symbols and error-correction symbols interleaved]
```

The `HIR2` magic header and the 16-byte random salt are themselves hieroglyph-encoded before being written, so the on-disk representation is still a pure symbol stream. The first ~26 symbols of any v2 glyph file decode to:

```
bytes 0–3   → ASCII "HIR2"   (4 bytes, 16 symbols + 2 parity)
bytes 4–19  → random salt    (16 bytes, 64 symbols + 8 parity)
bytes 20+   → encrypted payload (variable length)
```

Because each file uses a freshly generated salt, two glyphs encoded from the same plaintext with the same key look completely different:

```
Encode "hello" with password "test"  →  ╲•│─╱\○│•╲/│□─\╱◇...
Encode "hello" with password "test"  →  /│╲─•│△╱\│╲/─○│╲...
```
*(Two different glyphs for the same input — the random salt makes them unique every time.)*

---

## Annotated Example — Hello World

Source Python script (`hello.py`):

```python
print("Hello, Glyph World!")
```

After encoding with a password key (`python hiro.py encode --input hello.py --key-type password --key-source "demo"`), the `.glyph` file might look like:

```
──/•□|╲──△─|╲\△╲─\│○─|│•○\─/\◇\••|○\||╱□/•\|□\//╱◇│•╲─△\╲─│
○|╲|/□╲╱•╲◇│•─╱□/╱/|◇│─╲\○\/│╲◇╱\|•◇•││•○╱•─╲□••╲─△\//•○|│•
─○╱\│/△•╱|╱○||─\○╲•│╲○│─•╲△╱••\□/─|╱○╱╲|─◇╲//╲△╱/│\△/•|╱△••
\╱□─|•│○|\||○|╱╲─◇|\\│△│─│─○╱─/\○•─|/□•─|│○|╲|╱◇─/│|□│•/•□\
```

No one looking at that file can tell what language it encodes, how long the original content was, or even that it is an encrypted file and not a font sample.

---

## How Encoding Works Step by Step

```
┌─────────────────────────────────────────────────────────────────────┐
│  Source content  (e.g. "print('Hello!')")                           │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ 1. METADATA WRAPPING
                            │    {file_type: "python", content: "print(...)"}
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Plaintext payload  = file_type + separator + raw content           │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ 2. INTEGRITY HASH
                            │    SHA3-512(payload) prepended
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Signed payload  = SHA3-512 hash (64 bytes) + plaintext payload     │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ 3. KEY DERIVATION
                            │    PBKDF2-HMAC-SHA256 (100,000 rounds)
                            │    Key material = password / system FP / AES key
                            │    Salt = per-file random 16 bytes (v2)
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  AES key (32 bytes)  +  IV (16 bytes, random)                       │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ 4. STREAM CIPHER ENCRYPTION
                            │    IV prepended → XOR with SHA3-512 keystream
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Ciphertext bytes                                                   │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ 5. HIERARCHIC ENCODING
                            │    Every 3 bits → 1 data symbol from {•/\|─│╱╲}
                            │    Every 8 data symbols → 1 parity symbol from {○□△◇}
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  .glyph file  (pure Unicode symbol stream, no binary, no Base64)    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## How Decoding Works Step by Step

```
.glyph file
     │
     │ 1. Strip v2 header (HIR2 magic + 16-byte salt) if present
     │ 2. Separate data symbols from parity symbols
     │ 3. Verify parity (error detection)
     │ 4. Map data symbols → 3-bit groups → bytes (ciphertext)
     │ 5. Derive key using same parameters (password / FP / AES key + salt)
     │ 6. Decrypt: XOR ciphertext with SHA3-512 keystream seeded by (key + IV)
     │ 7. Split first 64 bytes as expected hash; remainder = plaintext payload
     │ 8. Recompute SHA3-512(plaintext payload); assert == stored hash
     │ 9. Parse file_type from metadata prefix
     │10. Dispatch to glyph_runner / addon for execution
     ▼
Executed content
```

If the key is wrong, step 8 fails (hash mismatch) and execution is refused. No information about the plaintext leaks.

---

## What Gets Encrypted

The encrypted payload contains:

```
[file_type string] [separator] [original file content]
```

For example, a Python file produces:

```
python|||print("Hello!")
```

And a JavaScript file:

```
javascript|||console.log("Hello!");
```

The `file_type` tells `glyph_runner` which addon to dispatch the content to once decrypted. This means the `.glyph` file itself gives no hint about what language or content type it holds.

---

## Key Types and How They Affect Glyphs

Different key types affect how the AES key is derived but not the visual format of the glyph file:

### System key (`--key-type system`)

The key is derived from a hardware fingerprint:

```python
fingerprint = {
    's1': sha256(cpu_model),    # CPU model string
    'm2': sha256(total_ram),    # Total physical RAM in bytes
    'h3': sha256(hostname),     # Machine hostname
    'o4': sha256(os_name),      # Operating system name
}
```

The glyph can only be decoded on the machine that encoded it (or on a machine that imports the exported fingerprint JSON). Anyone else who obtains the file cannot decrypt it because they cannot reproduce the hardware fingerprint.

### Password key (`--key-type password`)

PBKDF2-HMAC-SHA256 with 100,000 iterations converts the password into an AES-256 key. The glyph can be decoded by anyone who knows the password, on any machine.

### AES key (`--key-type aes`)

The supplied string is used directly as (or hashed into) the AES key. Useful for scripted automation where a fixed key is distributed securely by other means.

---

## Per-File Salt (v2 / HIR2 Format)

Without a per-file salt, two different glyphs encoded from the same password produce the same ciphertext (given the same IV). This allows pre-computed rainbow tables to attack the key offline.

The v2 format solves this by embedding a freshly generated 16-byte cryptographic random salt in every glyph file. The salt is fed into PBKDF2 as part of the key derivation, so:

```
password "abc"  +  salt 0xA1B2...  →  key_1
password "abc"  +  salt 0xC3D4...  →  key_2
```

`key_1 ≠ key_2` even though the password is the same. A rainbow table built against one glyph file is useless against any other glyph file.

---

## Error Correction Symbols

Every 8 data symbols are followed by 1 parity symbol chosen from `{○ □ △ ◇}`. The parity symbol encodes a 2-bit checksum over the preceding 8 symbols. This allows single-symbol corruption to be detected. It also means the parity symbols appear roughly once every 9 characters, giving glyph files a characteristic rhythm when read visually.

```
╲•│─╱\○│•╲/│□─\╱◇•─│╲\╱│○╲│─•/△
                ^         ^       ^
           parity (○)  parity (□)  parity (△)
```

---

## A Short Python Script as a Glyph

**Original `hello.py`:**

```python
import sys
print(f"Running on Python {sys.version}")
print("Hello from a glyph!")
```

**Encoded with `--key-type password --key-source demo`:**

```
\\|╱□╱\\\□╲••\△╲\─╱◇/|╱│△│\─|△\╱|╲◇╲│/│□/╱╲|△|•│\○/─\│□╲•/╱○
╱/╲•○││•|△╱╱│\○\│||△•╱/│○•│//△•/╱─△╱•••◇╲\•/◇─•/╲◇\|•│○//╲•△
|/\╲□|•─\△╲||/□─/││◇/─|•□|─│|△//─•○─│─|△╲│|╱□•\─│○╱│─╱○│•│•○
╱/│╲○│••/□─•╱│□│/╲╱○•/\╲◇•╱|╱○│╱─╱○|╱|─□||│─△│•╱│◇│/╱\△/╱\/◇
•|│/◇\╲|\△|\╲─△╲│─╲△│╱╱─○
```

Run it with:

```bash
python glyph_runner.py hello.glyph --key-type password --key "demo"
```

---

## A Batch Script as a Glyph

**Original `greet.bat`:**

```bat
@echo off
echo Hello from a glyph batch file!
```

**Encode:**

```bash
python hiro.py encode --input greet.bat --file-type bat --key-type password --key-source "demo" --output greet.glyph
```

**Run (Windows, in-memory — no .bat file written to disk):**

```bash
python glyph_runner.py greet.glyph --key-type password --key "demo" --no-temp
```

The `--no-temp` flag instructs the batch addon to pipe the decrypted script directly to `cmd.exe` stdin without ever writing a `.bat` file to disk.

---

## Why It Looks Like Art

The 8 data symbols (`• / \ | ─ │ ╱ ╲`) and 4 parity symbols (`○ □ △ ◇`) were deliberately chosen from the Unicode box-drawing and geometric shapes blocks. They:

- Are all thin, single-stroke characters
- Contain no letters or digits
- Mix to produce something that looks like a grid of graphical lines
- Are present in almost every Unicode-capable font

The encoder optionally emits the glyph in fixed-width rows (configurable with `--width`) so the output resembles a grid or tiled artwork rather than a single long line.

---

## Glyph vs Plain Encryption

| Property                    | .glyph file         | Base64 ciphertext   | PGP armoured block |
|-----------------------------|---------------------|---------------------|--------------------|
| Visually recognisable as encrypted | No          | Yes (Base64 chars)  | Yes (header text)  |
| Plausible deniability       | High                | None                | None               |
| Execution metadata embedded | Yes (file_type)     | No                  | No                 |
| Integrity check built in    | Yes (SHA3-512)      | Depends             | Yes (signatures)   |
| System-binding possible     | Yes                 | No                  | No                 |
| Per-file salt (rainbow-resistant) | Yes (v2)    | Depends on tool     | Yes                |
| Human-readable format       | Symbol stream       | ASCII               | ASCII              |
| Requires special runtime    | `glyph_runner.py`   | Any AES library     | GPG                |
