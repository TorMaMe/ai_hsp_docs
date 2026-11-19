# HSP documentation

Concise design documentation for the planned Rust version of **High Speed Projection**, a Windows companion to VideoPsalm. Based on the Rust low-level design **v0.1, 2026-11-18**. This repository documents the product; it does not contain the Rust app or a deployed Modal service.

## Preview locally

Python 3.12 is the tested documentation toolchain. In PowerShell, from this repository:

```powershell
py -3.12 -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
.venv\Scripts\python.exe -m mkdocs serve
```

Open **http://127.0.0.1:8000** yourself. Changes reload automatically. Press Ctrl+C to stop.

## Build and check

```powershell
.venv\Scripts\python.exe -m mkdocs build --strict
```

The generated site is in `site/` and is ignored by Git. CI checks the build without deploying or publishing it.

## Edit

Pages live in `docs/`; navigation and theme settings are in `mkdocs.yml`. Start with [the overview](docs/index.md). Keep proposed behavior, measured results, and open decisions distinct. Record verified VideoPsalm behavior with its exact version and test conditions. Never include credentials or private sermon recordings in documentation fixtures.
