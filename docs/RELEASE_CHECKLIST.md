# Release Checklist

## Goal

Use this checklist before publishing a new Project Hiro build or handing a binary to another team.

## 1. Working Tree

- confirm the intended files are present
- confirm the packaged executable is rebuilt from the current source
- confirm docs reflect the current entry points: `hiro.py` and `dist\hiroglyph.exe`

## 2. Rebuild the Executable

```powershell
powershell -ExecutionPolicy Bypass -File .\build_standalone.ps1 -Clean
```

Expected artifact:

- [dist/hiroglyph.exe](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/dist/hiroglyph.exe)

## 3. Run Regression Gates

Full release gate:

```powershell
powershell -ExecutionPolicy Bypass -File .\run_regression_suite.ps1
```

This covers:
- packaged exe regression tests
- crypto/security tests
- rainbow-table tests
- addon cleanup and no-temp tests

## 4. Run the Full Comprehensive Suite

```powershell
$env:PYTHONPATH=(Get-Location).Path
pytest tests\test_hiro_comprehensive.py -v -s
```

Expected:
- all tests pass
- benchmark output remains in the expected range for this machine class

## 5. Generate a Machine-Readable Benchmark Report

```powershell
python .\tools\generate_benchmark_report.py
```

Expected artifact:

- [artifacts/benchmark_report.json](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/artifacts/benchmark_report.json)

Review:
- encode/decode/run timings
- payload sizes
- pass/fail status of benchmark scenarios

## 6. Manual Smoke Pass

Run these manually from the repo root:

```powershell
.\dist\hiroglyph.exe -h
.\dist\hiroglyph.exe run --list-addons
.\dist\hiroglyph.exe encode "print('release smoke')" --file-type python --key-type password --key-source smoke --output smoke.glyph
.\dist\hiroglyph.exe run smoke.glyph --key-type password --key smoke --no-temp
Remove-Item smoke.glyph
```

Optional richer smoke:
- local import Python glyph
- async Python glyph
- HTML addon path
- JavaScript addon path

## 7. Review Security Positioning

Before release notes or external sharing, keep messaging aligned with:

- [docs/HIRO_VS_AES_AND_EXE_TESTING.md](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/docs/HIRO_VS_AES_AND_EXE_TESTING.md)

Key rule:
- describe Hiro as a protected execution and packaging framework
- do not describe Hiro as cryptographically stronger than AES-256-GCM

## 8. Archive Release Evidence

Keep:
- test logs
- `benchmark_report.json`
- exact executable size and timestamp
- any notes about skipped scenarios due to missing runtimes like Node.js or Java tools

## 9. Ship Decision

Good to ship when:
- exe rebuild succeeds
- regression suite passes
- comprehensive suite passes
- benchmark report is generated
- docs match the delivered behavior
