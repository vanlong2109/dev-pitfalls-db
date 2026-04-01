# Dev Pitfalls Database

A knowledge base of **real bugs, fixes, and lessons learned** across all projects. Each issue is documented with root cause analysis, fix code, and prevention tips — so the same mistake is never repeated.

## Categories

| Category | Issues | Description |
|----------|--------|-------------|
| [Electron + Vite](issues/) | 4 | Forge packaging, native modules, fuses, ABI |
| _(more coming)_ | | React, CI/CD, databases, deployments, etc. |

## Issues Index

| # | Issue | Severity | Category | Source Project |
|---|-------|----------|----------|----------------|
| 001 | [Renderer missing from package](issues/001-renderer-missing-from-package.md) | Critical | Electron/Vite | SkyPower IDE |
| 002 | [Native module not found in production](issues/002-native-module-not-found.md) | Critical | Electron/Native | SkyPower IDE |
| 003 | [Asar fuses block unsigned apps](issues/003-asar-fuses-block-unsigned.md) | High | Electron/Security | SkyPower IDE |
| 004 | [serialport ABI mismatch](issues/004-serialport-abi-mismatch.md) | High | Electron/Native | SkyPower IDE |

## Issue Template

Each issue follows this structure:
- **Symptom** — what you see (error message, behavior)
- **Root Cause** — why it happens (deep analysis)
- **Fix** — working code solution
- **Key Lesson** — one-liner to remember
- **Prevention** — how to avoid it next time

## How to Use

1. Hit a bug? Search this repo by keyword or error message
2. Found a new pitfall? Add it using the template above
3. Starting a new project? Check the relevant category checklist

## Quick Checklists

### Electron Forge + Vite
- [ ] Renderer `outDir` → `.vite/renderer/<window_name>/` at project root
- [ ] Native modules: `eval('require')` + `postPackage` copy + `electron-rebuild`
- [ ] Asar fuses disabled until code signing configured
- [ ] Test: `npm run make` → install on clean machine before every release

---

Maintained by [@vanlong2109](https://github.com/vanlong2109)
