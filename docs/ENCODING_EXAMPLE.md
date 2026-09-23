# Encoding Example (Before vs After)

This document shows a concrete HIRO encoding example with:
- original plaintext (before)
- encoded glyph string (after)
- key details used
- decode verification result

## Example Setup

- **Plaintext input:** `Hello Hiroglyph!`
- **Key type:** `password`
- **Key source:** `DemoPass2026!`
- **File type embedded:** `python`
- **Integrity hash:** enabled
- **Error protection symbols:** enabled
- **Format version:** `1` (`HIRO` header)

> Note: This example was generated with a fixed IV for reproducibility (`0x01` repeated 16 times) and `use_per_file_salt=False`.
> In normal usage, output will be different each run because IV/salt are random.

---

## Before Encoding

### Human-readable input

`Hello Hiroglyph!`

### Embedded message bytes (hex)

`707974686f6e0a48656c6c6f204869726f676c79706821`

(That is `python\nHello Hiroglyph!` in UTF-8)

---

## Key + Crypto Metadata

- **key_id:** `cba1350b00b85a6c`
- **encrypted bytes prefix (hex):**

`4849524f010101010101010101010101010101016e2f138cd0a4048a08d18228`

- `4849524f` = ASCII `HIRO`

- **protected bytes prefix (hex):**

`36f24da265b986fe936b13c76378bbf042a9bde8b6a514e1397aca4ec2035ce5`

---

## After Encoding (Glyph Output)

### Glyph length

`357` symbols

### Full encoded glyph string

`\\•─△─│\\△\|╱•◇•─•/◇••\•△•─•/◇••\•△•─•/◇••\•△•─•/◇••\•△•─•/◇••\•△•││╱◇/|╱/◇/╱/─◇╱─/\○\••─○─\─•△─|\/◇─•─\△─\╲/□╱│\╱○╱/\╲△/╲|╱△─╲│\○/•││□\•\|△╱\││□╲/─\□╲\|╱◇─/•/○──╱•◇─|│╲○••\│△\|││○╱\|\□││•─△╱│•│◇│─\•○|─\│△|\╱─□╲|╱╱○││╲\△╱/│\△╱│╲•◇╱│•─□\╲/|□•╱|/△•/╲|◇\//╲○╲─╲─○/••─◇──╱\□╱─/╱□\•╱•□─|/─◇\╲╲\○╱─│\◇|││/◇╱\\/○\|//△||─/◇│|╲─○╲│•─△╱╱─/□\/\│□╱|│╲□••─/◇╱│|│□│•|│△╱•`

---

## Decode Check (Verification)

Using the same key settings:

- **decoded_success:** `true`
- **decoded_file_type:** `python`
- **integrity_verified:** `true`
- **decoded_message:** `Hello Hiroglyph!`

---

## Quick Summary

This confirms a full reversible pipeline:

1. plaintext (`Hello Hiroglyph!`)
2. key derivation (`password` + `DemoPass2026!`)
3. encryption + glyph conversion
4. successful decode back to original message with integrity pass
