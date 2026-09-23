# Windows Batch Addon for Hiroglyph

## Overview

The Windows Batch addon enables Glyph Runner to execute encrypted Windows batch scripts (`.bat`, `.cmd`) using the native Windows command processor (`cmd.exe`). **No downloads or external dependencies are required** - it uses 100% native Windows functionality.

## Features

✅ **Native Windows Execution** - Uses built-in `cmd.exe` (available on all Windows systems)  
✅ **No External Dependencies** - Works out of the box without any downloads  
✅ **Multiple File Types** - Supports `.bat`, `.cmd`, and `.batch` extensions  
✅ **Full Batch Capabilities** - All native batch commands and features work  
✅ **Secure Execution** - Scripts execute from encrypted `.glyph` files  
✅ **Automatic Cleanup** - Temporary files are automatically removed after execution  

## Supported File Types

- `bat` - Windows Batch Files
- `cmd` - Windows Command Scripts
- `batch` - Windows Batch (alternative extension)

## Quick Start

### 1. Encode a Batch Script

```batch
python hiro.py encode --input script.bat --output script.glyph --key-type system --file-type bat
```

### 2. Execute the Encrypted Script

```batch
python glyph_runner.py script.glyph
```

That's it! The script will be decrypted and executed using Windows native commands.

## Examples

### Example 1: Simple System Info Script

**Create `sysinfo.bat`:**
```batch
@echo off
echo System Information
echo ==================
echo Computer: %COMPUTERNAME%
echo User: %USERNAME%
echo OS: %OS%
ver
```

**Encode it:**
```batch
python hiro.py encode -i sysinfo.bat -o sysinfo.glyph -k system -t bat
```

**Run it:**
```batch
python glyph_runner.py sysinfo.glyph
```

### Example 2: File Management Script

**Create `cleanup.bat`:**
```batch
@echo off
echo Cleaning temporary files...
del /q /f %TEMP%\*.tmp 2>nul
echo Cleanup complete!
```

**Encode and run:**
```batch
python hiro.py encode -i cleanup.bat -o cleanup.glyph -k system -t bat
python glyph_runner.py cleanup.glyph
```

### Example 3: Automated Task Script

**Create `backup.bat`:**
```batch
@echo off
set BACKUP_DIR=C:\Backups
set SOURCE_DIR=C:\Projects

echo Creating backup...
mkdir %BACKUP_DIR% 2>nul
xcopy %SOURCE_DIR% %BACKUP_DIR% /E /I /Y
echo Backup complete: %BACKUP_DIR%
```

**Encode with password:**
```batch
python hiro.py encode -i backup.bat -o backup.glyph -k password -s "MySecurePassword" -t bat
python glyph_runner.py --key-type password --key "MySecurePassword" backup.glyph
```

## Available Batch Commands

The addon supports ALL native Windows batch commands, including:

### File Operations
- `dir`, `cd`, `mkdir`, `rmdir`, `del`, `copy`, `move`, `xcopy`, `robocopy`
- `type`, `more`, `find`, `findstr`, `attrib`

### System Commands
- `echo`, `cls`, `pause`, `exit`, `start`, `shutdown`
- `ver`, `date`, `time`, `systeminfo`, `tasklist`, `taskkill`

### Network Commands
- `ipconfig`, `ping`, `tracert`, `netstat`, `nslookup`

### Control Flow
- `if`, `else`, `goto`, `call`, `for`, `while`
- `set`, `setlocal`, `endlocal`

### And More!
- Environment variables (`%VAR%`)
- Command chaining (`&`, `&&`, `||`)
- Input/Output redirection (`>`, `>>`, `<`, `|`)
- Comments (`REM`, `::`)

## Advanced Usage

### Custom Options

The batch addon supports several optional parameters:

```python
# In your code or via glyph_runner modifications:
kwargs = {
    'keep_temp': False,      # Keep temporary .bat file (default: False)
    'temp_dir': None,        # Custom temp directory (default: system temp)
    'no_pause': False,       # Don't add pause at end (default: False)
    'echo_on': False,        # Show commands as executed (default: False)
    'timeout': None,         # Timeout in seconds (default: None)
}
```

### Security Considerations

⚠️ **Important Security Notes:**

1. **Execution Permissions** - Scripts run with your user permissions
2. **System Access** - Scripts have full access to your system within user rights
3. **Trust Source** - Only run batch scripts from trusted sources
4. **Review Code** - Review batch content before encoding/executing
5. **Password Protection** - Use strong passwords for sensitive scripts

### Encoding Types

The addon works with all three key types:

#### System Key (Hardware-Bound)
```batch
python hiro.py encode -i script.bat -o script.glyph -k system -t bat
python glyph_runner.py script.glyph
```

#### Password Key (Portable)
```batch
python hiro.py encode -i script.bat -o script.glyph -k password -s "MyPassword" -t bat
python glyph_runner.py --key-type password --key "MyPassword" script.glyph
```

#### AES Key (Maximum Security)
```batch
python hiro.py encode -i script.bat -o script.glyph -k aes -s "MyAESKey123" -t bat
python glyph_runner.py --key-type aes --key "MyAESKey123" script.glyph
```

## Troubleshooting

### Unicode Characters

If your batch script contains special Unicode characters (like box-drawing characters), they will be preserved in UTF-8 encoding. If there are encoding issues, the addon will automatically fall back to safer encodings.

### Exit Codes

The addon properly propagates exit codes from your batch scripts:
- `0` = Success
- Non-zero = Error/Failure

### Temporary Files

Temporary `.bat` files are automatically created and cleaned up. If you need to debug, use:
```python
# Keep temp files for inspection
kwargs = {'keep_temp': True}
```

## Integration with Hiroglyph Ecosystem

The batch addon seamlessly integrates with:

- ✅ **glyph_runner.py** - Main execution engine
- ✅ **hiro.py** - Encoding/decoding system
- ✅ **System Keys** - Hardware-bound encryption
- ✅ **Password Keys** - User-defined encryption
- ✅ **AES Keys** - Direct encryption
- ✅ **Other Addons** - HTML, Java, Python, etc.

## Why Use Batch with Hiroglyph?

### Use Cases

1. **Secure Automation** - Encrypt maintenance scripts
2. **System Administration** - Protect sensitive admin tasks
3. **Deployment Scripts** - Secure deployment automation
4. **Legacy Systems** - Work with existing batch infrastructure
5. **No Dependencies** - Works on any Windows system without installation

### Benefits

- ✅ Universal compatibility (Windows XP to Windows 11+)
- ✅ Zero installation overhead
- ✅ Native performance
- ✅ Full batch language support
- ✅ Encrypted storage of scripts
- ✅ Portable (with password/AES keys)

## Version History

### v1.0.0 (Current)
- Initial release
- Support for .bat, .cmd, .batch files
- Native cmd.exe execution
- UTF-8 encoding with fallbacks
- Automatic temp file cleanup
- Full integration with glyph_runner

## See Also

- [Main README](../README.md) - Hiroglyph project overview
- [Addon Development Guide](../ADDON_DEVELOPMENT_GUIDE.md) - Create your own addons
- [Usage Cheatsheet](../docs/USAGE_CHEATSHEET.md) - Quick reference guide

---

**Made with ❤️ for the Hiroglyph project**  
*Execute encrypted Windows batch scripts with zero dependencies!*
