# Glyph Runner Addon Development Guide

## Overview

Glyph Runner supports extensibility through an addon system that allows developers to add support for new file types and programming languages. This document explains how to create addons for Glyph Runner.

## Architecture

### Addon System Components

1. **AddonManager**: Manages addon loading, registration, and execution
2. **Addon Classes**: Python classes that implement the addon interface
3. **Addon Files**: Python modules in the `addons/` directory

### File Structure

```
glyph_runner/
├── glyph_runner.py          # Main runner (loads addons)
├── addons/                  # Addon directory
│   ├── __init__.py         # Package initialization
│   ├── base.py             # Base addon framework (optional)
│   └── your_addon.py       # Your custom addon
└── hiro.py                 # Encoder (for reference)
```

## Creating a New Addon

### Step 1: Create the Addon File

Create a new Python file in the `addons/` directory (e.g., `my_language.py`).

### Step 2: Import Required Classes

```python
# No imports required - addons use duck typing
```

### Step 3: Create Your Addon Class

```python
class MyLanguageAddon:
    """Addon for handling MyLanguage files."""

    @property
    def name(self) -> str:
        return "MyLanguage Interpreter"

    @property
    def version(self) -> str:
        return "1.0.0"

    @property
    def supported_file_types(self) -> list:
        return ['mylang', 'ml']  # File types this addon handles

    def validate_content(self, content: str) -> bool:
        """Optional: Validate content before execution."""
        # Add validation logic here
        return True

    def execute(self, content: str, filepath: str, **kwargs) -> int:
        """Execute the content. Return exit code (0 = success)."""
        try:
            # Your execution logic here
            print(f"Executing MyLanguage code from {filepath}")
            # ... process content ...
            return 0
        except Exception as e:
            print(f"Error executing MyLanguage code: {e}")
            return 1

    def get_description(self) -> str:
        """Optional: Describe what this addon does."""
        return "MyLanguage addon for interpreting encrypted MyLanguage scripts"
```

### Step 4: Required Methods

#### `name` (property)
- Returns a human-readable name for your addon
- Used in status messages and listings

#### `version` (property)
- Returns version string (e.g., "1.0.0")
- Used for addon management and compatibility

#### `supported_file_types` (property)
- Returns list of file type strings (e.g., `['html', 'htm']`)
- These correspond to the `--file-type` parameter used when encoding

#### `execute(content, filepath, **kwargs)` (method)
- **Required**: Main execution method
- `content`: Decrypted file content as string
- `filepath`: Path to the original `.glyph` file
- `**kwargs`: Additional execution parameters
- Returns: Exit code (0 = success, non-zero = failure)

### Step 5: Optional Methods

#### `validate_content(content)` (method)
- Validate content before execution
- Return `True` if valid, `False` if invalid
- Called before `execute()` if implemented

#### `get_description()` (method)
- Return descriptive text about the addon
- Used in `--addon-info` output

## Complete Example: HTML Addon

```python
import os
import tempfile
import webbrowser

class HTMLAddon:
    """Addon for displaying HTML files in a browser."""

    @property
    def name(self) -> str:
        return "HTML Viewer"

    @property
    def version(self) -> str:
        return "1.0.0"

    @property
    def supported_file_types(self) -> list:
        return ['html', 'htm']

    def validate_content(self, content: str) -> bool:
        """Check if content looks like HTML."""
        content_lower = content.strip().lower()
        has_tags = '<' in content and '>' in content
        has_html = '<html' in content_lower or '<body' in content_lower
        return has_tags and (has_html or '<div' in content_lower)

    def execute(self, content: str, filepath: str, **kwargs) -> int:
        """Create temp HTML file and open in browser."""
        try:
            # Create temporary file
            with tempfile.NamedTemporaryFile(mode='w', suffix='.html',
                                          delete=False, encoding='utf-8') as f:
                f.write(content)
                temp_path = f.name

            print(f"Opening HTML file in browser: {temp_path}")
            webbrowser.open(f'file://{temp_path}')

            # Wait for user
            input("Press Enter when done viewing...")

            # Cleanup
            os.unlink(temp_path)
            return 0

        except Exception as e:
            print(f"Error displaying HTML: {e}")
            return 1

    def get_description(self) -> str:
        return "HTML Viewer addon for displaying encrypted HTML files in a browser"
```

## Testing Your Addon

### 1. Create Test Content

Create a file with content for your language (e.g., `test.mylang`):

```
print "Hello from MyLanguage!"
x = 42
print x
```

### 2. Encode the File

Use hiro.py to encode with the appropriate file type:

```bash
python hiro.py encode --input test.mylang --key-type system --file-type mylang --output test.glyph
```

### 3. Test Execution

Run with Glyph Runner:

```bash
python glyph_runner.py test.glyph
```

### 4. Check Addon Loading

Verify your addon is loaded:

```bash
python glyph_runner.py --list-addons
python glyph_runner.py --addon-info
```

## Best Practices

### Security
- Validate input content to prevent malicious execution
- Use safe execution methods (avoid `eval()` when possible)
- Handle errors gracefully without exposing sensitive information
- Consider the security implications of your execution method

### Error Handling
- Return appropriate exit codes (0 = success)
- Provide clear error messages
- Log errors for debugging
- Handle edge cases (empty files, malformed content)

### Performance
- Keep validation lightweight
- Avoid unnecessary file I/O
- Clean up temporary files
- Consider memory usage for large files

### User Experience
- Provide progress feedback for long operations
- Allow configuration through kwargs
- Support both interactive and automated execution
- Document any special requirements

## Advanced Features

### Configuration Parameters

Addons can accept configuration through kwargs:

```python
def execute(self, content: str, filepath: str, **kwargs) -> int:
    debug = kwargs.get('debug', False)
    timeout = kwargs.get('timeout', 30)

    if debug:
        print(f"Debug: Processing {len(content)} characters")

    # Use timeout for execution...
```

### Multiple File Types

One addon can handle multiple related file types:

```python
@property
def supported_file_types(self) -> list:
    return ['javascript', 'js', 'typescript', 'ts']
```

### Complex Validation

Implement sophisticated content validation:

```python
def validate_content(self, content: str) -> bool:
    """Validate JavaScript syntax."""
    try:
        # Use a JavaScript parser or basic checks
        import re
        # Check for balanced braces, proper syntax, etc.
        return True
    except:
        return False
```

## Troubleshooting

### Addon Not Loading
- Check that the file is in the `addons/` directory
- Ensure the class has the required properties: `name`, `version`, `supported_file_types`, `execute`
- Verify the module can be imported (check for syntax errors)
- Look for import errors in the console output

### File Type Not Recognized
- Verify the file was encoded with `--file-type your_type`
- Check that `supported_file_types` includes your type
- Ensure no other addon handles the same type

### Execution Errors
- Add debug prints to your `execute()` method
- Check that required dependencies are installed
- Verify file permissions for temporary files
- Test with simple content first

## Distribution

### Sharing Addons
- Package addons as separate files in `addons/`
- Document dependencies and requirements
- Include version information
- Provide usage examples

### Version Management
- Use semantic versioning (MAJOR.MINOR.PATCH)
- Update version when making breaking changes
- Document changelog in addon comments

## Example Addons

The system includes these example addons:

- **HTML Addon** (`html.py`): Displays HTML files in browser
- **Base Framework** (`base.py`): Optional framework classes for building addons

Use these as templates for your own addons!

## Support

For questions about addon development:
1. Check the base.py source code for API details
2. Review existing addons for examples
3. Test incrementally - start simple and add features
4. Join the development community for help

---

*This guide is for Glyph Runner addon development. For general Glyph Runner usage and complete command reference, see the main README.md and USAGE_CHEATSHEET.md.*