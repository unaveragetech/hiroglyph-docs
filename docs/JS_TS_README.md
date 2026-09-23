# JavaScript & TypeScript Addons for Glyph Runner

[![JavaScript Test](https://img.shields.io/badge/JavaScript-PASSED-green)](testing/test_js_addon.py)
[![TypeScript Test](https://img.shields.io/badge/TypeScript-PASSED-green)](testing/test_ts_addon.py)

This document provides comprehensive documentation for the JavaScript and TypeScript addons that enable secure execution of encrypted JavaScript and TypeScript code within the Glyph Runner system.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Security](#security)
- [Testing](#testing)
- [API Reference](#api-reference)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Overview

The JavaScript and TypeScript addons extend Glyph Runner's capabilities to handle encrypted `.glyph` files containing JavaScript and TypeScript code. These addons provide:

- **JavaScript Addon**: Direct execution of JavaScript code via Node.js stdin (in-memory)
- **TypeScript Addon**: Compilation of TypeScript to JavaScript using the TypeScript compiler, followed by execution

Both addons maintain Glyph Runner's security principles by avoiding persistent decrypted files and using secure temporary file handling when compilation is required.

## Features

### JavaScript Addon (`javascript.py`)

- ✅ **In-Memory Execution**: JavaScript code executes via Node.js stdin without creating persistent files
- ✅ **ES6+ Support**: Full support for modern JavaScript features including async/await, arrow functions, and modules
- ✅ **Syntax Validation**: Pattern-based validation to ensure content appears to be valid JavaScript
- ✅ **Error Handling**: Comprehensive error catching with detailed error messages
- ✅ **Cross-Platform**: Works on Windows, macOS, and Linux

### TypeScript Addon (`typescript.py`)

- ✅ **TypeScript Compilation**: Uses official TypeScript compiler (tsc) to compile TS to JS
- ✅ **Type Safety**: Leverages TypeScript's static type checking during compilation
- ✅ **Advanced Features**: Support for interfaces, generics, classes, and modern TypeScript syntax
- ✅ **Secure Compilation**: Temporary files are securely created and cleaned up
- ✅ **Cross-Platform**: Automatic detection and use of platform-specific executable paths

### Common Features

- 🔒 **Security First**: No persistent decrypted files, secure temp file cleanup
- 🛡️ **Validation**: Syntax validation before execution
- 📊 **Logging**: Detailed execution logs and error reporting
- 🔧 **Configurable**: Customizable Node.js and TypeScript compiler paths
- 🧪 **Tested**: Comprehensive test suites with 100% pass rates

## Requirements

### System Requirements

- **Python**: 3.7+ (same as Glyph Runner)
- **Node.js**: 14.0+ (for JavaScript execution)
- **TypeScript Compiler**: 4.0+ (for TypeScript compilation)

### Dependencies

The addons use only standard library modules plus:
- `subprocess` (standard library)
- `tempfile` (standard library)
- `shutil` (standard library)
- `re` (standard library)

No external Python packages are required.

## Installation

### 1. Install Node.js

Download and install Node.js from [nodejs.org](https://nodejs.org/) (version 14.0 or higher recommended).

**Windows/macOS**: Use the installer
**Linux**: Use your package manager or the official installer

Verify installation:
```bash
node --version
# Should show: v22.14.0 or similar
```

### 2. Install TypeScript Compiler (for TypeScript addon)

```bash
npm install -g typescript
```

Verify installation:
```bash
tsc --version
# Should show: Version 5.8.3 or similar
```

### 3. Verify Addon Installation

The addons are automatically discovered by Glyph Runner. Check that they're loaded:

```bash
python glyph_runner.py --list-addons
```

You should see:
```
Available Addons:
- Windows Batch Runner v1.0.0
- HTML Viewer v1.0.0
- Java Runner v1.0.0
- JavaScript Runner v1.0.0
- TypeScript Runner v1.0.0
```

## Usage

### Basic Usage

#### 1. Create JavaScript/TypeScript Files

**hello.js**:
```javascript
console.log("Hello from JavaScript addon!");
console.log("Current timestamp:", new Date().toISOString());

const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);

console.log("Original numbers:", numbers);
console.log("Doubled numbers:", doubled);
```

**hello.ts**:
```typescript
interface User {
    name: string;
    age: number;
}

const user: User = {
    name: "TypeScript User",
    age: 25
};

function greetUser(user: User): string {
    return `Hello, ${user.name}! You are ${user.age} years old.`;
}

console.log(greetUser(user));

// Generic function
function identity<T>(arg: T): T {
    return arg;
}

const result: string = identity("TypeScript generics work!");
console.log(result);
```

#### 2. Encode Files

**JavaScript**:
```bash
python hiro.py encode --input hello.js --output hello.js.glyph --file-type javascript --key-type system
```

**TypeScript**:
```bash
python hiro.py encode --input hello.ts --output hello.ts.glyph --file-type typescript --key-type system
```

#### 3. Execute Encrypted Files

**JavaScript**:
```bash
python glyph_runner.py hello.js.glyph --key-type system
```

**TypeScript**:
```bash
python glyph_runner.py hello.ts.glyph --key-type system
```

### Advanced Usage

#### Custom Node.js Path

```bash
python glyph_runner.py hello.js.glyph --key-type system --node_cmd /custom/path/to/node
```

#### Custom TypeScript Compiler Path

```bash
python glyph_runner.py hello.ts.glyph --key-type system --tsc_cmd /custom/path/to/tsc
```

#### Keep Temporary Files (for debugging)

```bash
python glyph_runner.py hello.ts.glyph --key-type system --keep_temp
```

#### Disable Temporary Files (TypeScript only)

```bash
python glyph_runner.py hello.ts.glyph --key-type system --no_temp
```

## Security

### Security Measures

1. **No Persistent Decrypted Files**: JavaScript executes in-memory via stdin. TypeScript compiles to temporary files that are immediately deleted.

2. **Secure File Cleanup**: Temporary files are overwritten with null bytes before deletion to prevent forensic recovery.

3. **Input Validation**: Content is validated for syntax patterns before execution.

4. **Error Sanitization**: Error messages are sanitized to prevent information leakage.

5. **Timeout Protection**: Execution is limited to prevent infinite loops or hanging processes.

### Security Considerations

- **Untrusted Code**: Treat encrypted content as potentially malicious
- **Network Access**: JavaScript/TypeScript can make network requests
- **File System Access**: Code can read/write files if permissions allow
- **System Commands**: Can execute system commands via child_process (Node.js) or process spawning

## Testing

### Run Test Suites

#### JavaScript Addon Tests

```bash
python testing/test_js_addon.py
```

Expected output:
```
============================================================
TESTING JAVASCRIPT ADDON
============================================================
...
✅ Encoding successful
✅ Glyph file created
✅ Glyph runner completed
✅ Output verification passed (6/6)
✅ JavaScript addon discovered
🧹 Cleaned up glyph file
🎉 JAVASCRIPT ADDON TEST PASSED!
```

#### TypeScript Addon Tests

```bash
python testing/test_ts_addon.py
```

Expected output:
```
============================================================
TESTING TYPESCRIPT ADDON
============================================================
...
✅ Encoding successful
✅ Glyph file created
✅ Glyph runner completed
✅ Output verification passed (7/7)
✅ TypeScript addon discovered
🧹 Cleaned up glyph file
🎉 TYPESCRIPT ADDON TEST PASSED!
```

### Test Files

- `testing/hello.js` - JavaScript test program
- `testing/hello.ts` - TypeScript test program
- `testing/test_js_addon.py` - JavaScript addon test suite
- `testing/test_ts_addon.py` - TypeScript addon test suite

### Manual Testing

1. Create test files with your own JavaScript/TypeScript code
2. Encode them using hiro.py
3. Execute using glyph_runner.py
4. Verify output matches expectations

## API Reference

### JavaScriptAddon Class

```python
class JavaScriptAddon:
    @property
    def name(self) -> str
        """Returns: 'JavaScript Runner'"""

    @property
    def version(self) -> str
        """Returns: '1.0.0'"""

    @property
    def supported_file_types(self) -> List[str]
        """Returns: ['javascript', 'js']"""

    def validate_content(self, content: str) -> bool
        """Validates JavaScript syntax patterns"""

    def execute(self, content: str, filepath: str, **kwargs) -> int
        """Executes JavaScript content via Node.js"""

    def get_description(self) -> str
        """Returns addon description"""
```

### TypeScriptAddon Class

```python
class TypeScriptAddon:
    @property
    def name(self) -> str
        """Returns: 'TypeScript Runner'"""

    @property
    def version(self) -> str
        """Returns: '1.0.0'"""

    @property
    def supported_file_types(self) -> List[str]
        """Returns: ['typescript', 'ts']"""

    def validate_content(self, content: str) -> bool
        """Validates TypeScript syntax patterns"""

    def execute(self, content: str, filepath: str, **kwargs) -> int
        """Compiles and executes TypeScript content"""

    def get_description(self) -> str
        """Returns addon description"""
```

### Execution Parameters

Both addons accept these `**kwargs` parameters:

- `node_cmd`: Custom Node.js executable path (default: 'node')
- `tsc_cmd`: Custom TypeScript compiler path (default: 'tsc') - TypeScript addon only
- `timeout`: Execution timeout in seconds (default: 30)
- `keep_temp`: Keep temporary files after execution (default: False) - TypeScript addon only
- `no_temp`: Skip temporary file creation (default: False) - TypeScript addon only

## Troubleshooting

### Common Issues

#### "Node.js not found" Error

**Solution**: Install Node.js and ensure it's in your PATH
```bash
# Check if Node.js is installed
node --version

# If not found, install from https://nodejs.org/
# Or specify custom path
python glyph_runner.py file.glyph --node_cmd /path/to/node
```

#### "TypeScript compiler (tsc) not found" Error

**Solution**: Install TypeScript compiler globally
```bash
npm install -g typescript
tsc --version

# Or specify custom path
python glyph_runner.py file.glyph --tsc_cmd /path/to/tsc
```

#### "TypeScript compilation failed" Error

**Common causes**:
- Syntax errors in TypeScript code
- Using incompatible module system
- Missing type definitions

**Solution**: Check TypeScript compilation manually
```bash
tsc --noEmit --target ES2020 yourfile.ts
```

#### Permission Errors

**Solution**: Ensure proper permissions for Node.js and TypeScript executables

#### Windows Path Issues

**Solution**: The addons automatically resolve Windows executable paths. If issues persist, specify full paths manually.

### Debug Mode

Enable detailed logging by checking the stdout/stderr output from glyph_runner.py. The addons provide comprehensive error messages and execution details.

### Getting Help

1. Check the test outputs for expected behavior
2. Review the addon source code for implementation details
3. Check Glyph Runner logs for addon loading issues
4. Verify Node.js and TypeScript installations

## Contributing

### Development Setup

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Ensure all tests pass
6. Submit a pull request

### Code Style

- Follow existing code patterns in the addons
- Use type hints for Python code
- Include comprehensive docstrings
- Add error handling for edge cases
- Maintain security best practices

### Testing Guidelines

- Add test cases for new features
- Test on multiple platforms (Windows, macOS, Linux)
- Verify security measures are maintained
- Test error conditions and edge cases

### Documentation

- Update this README for new features
- Include code examples
- Document any new configuration options
- Update troubleshooting section as needed

## License

This addon follows the same license as the main Glyph Runner project.

## Changelog

### Version 1.0.0
- Initial release
- JavaScript addon with in-memory execution
- TypeScript addon with compilation support
- Comprehensive test suites
- Cross-platform compatibility
- Security-focused design

---

For more information about Glyph Runner, see the main [README.md](../MAIN/README.md) file.
