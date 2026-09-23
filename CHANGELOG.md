# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- **Cross-System Compatibility**: Export/import system variables for sharing encrypted files between systems
- **Enhanced System Fingerprinting**: SHA256-hashed hardware attributes for improved obfuscation
- **Export System Variables**: `export-system-vars` command to save system fingerprints to JSON
- **System Variables Import**: `--system-vars-file` option for cross-system decryption and execution
- **Transsystem Key Support**: System-key encrypted files can be shared using exported fingerprints
- **Addon System**: Extensible plugin architecture for supporting multiple file types
- **HTML Addon**: Built-in support for encoding and displaying HTML files in browsers
- **--html Flag**: Convenient command-line option for HTML file encoding
- **Addon Management**: CLI commands for listing and inspecting loaded addons
- **Usage Cheatsheet**: Comprehensive command reference and examples in `USAGE_CHEATSHEET.md`
- **Secure Notes Application**: Password-protected note-taking app that can be encrypted and run as a glyph file (`notes_app.py`, `NOTES_README.md`)
- Initial release of Crypto-Hieroglyphic Encoder
- System-bound encryption using SHA3-512
- Visual hieroglyphic encoding with 8 Unicode symbols
- Memory-only execution capabilities
- Comprehensive brute force demonstration
- CLI interface for encoding/decoding
- Glyph runner for direct execution
- White paper documentation
- Security analysis and proofs

### Features
- 256-bit equivalent cryptographic strength
- Error correction with parity symbols
- Integrity verification with SHA3-512
- Multiple key types (system, password)
- Interactive terminal UI with Rich library
- Educational attack demonstrations
- Cross-platform compatibility
- Extensible addon system for new file types

## [0.1.0] - 2025-11-25

### Added
- Core cryptographic engine
- Hieroglyphic encoding/decoding
- System fingerprinting module
- Basic CLI interface
- File I/O operations
- Error handling and validation
- Initial test suite

### Security
- SHA3-512 key derivation
- Custom stream cipher implementation
- Random IV generation
- Integrity hash verification
- System binding for authentication

### Documentation
- White paper with technical details
- Brute force demonstration guide
- API documentation
- Security analysis

---

## Types of Changes
- `Added` for new features
- `Changed` for changes in existing functionality
- `Deprecated` for soon-to-be removed features
- `Removed` for now removed features
- `Fixed` for any bug fixes
- `Security` in case of vulnerabilities</content>
<parameter name="filePath">c:\Users\b0052\Desktop\hiroglyph\CHANGELOG.md