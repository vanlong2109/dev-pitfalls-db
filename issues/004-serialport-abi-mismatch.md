# Issue 004: serialport ABI Mismatch in Electron

**Severity:** High — serial features crash  
**Category:** Native Modules  
**Affects:** serialport, any native addon using node-pre-gyp or prebuildify  

## Symptom

```
Error: No native build was found for platform=win32 arch=x64 runtime=electron abi=145 uv=1 libc=glibc node=24.14.0 electron=41.1.0
```

## Root Cause

When you `npm install serialport`, it downloads prebuilt `.node` binaries for **Node.js ABI** (e.g., ABI 131 for Node 22). But Electron uses a different ABI (e.g., ABI 145 for Electron 41).

The prebuilt binary doesn't match → "No native build found."

## Fix

### Option A: electron-rebuild (recommended)

```json
// package.json
"scripts": {
  "postinstall": "electron-rebuild -f -w serialport || true"
}
"devDependencies": {
  "@electron/rebuild": "^4.0.3"
}
```

This recompiles the native addon for the correct Electron version on every `npm install`.

### Option B: Rebuild in CI only

```yaml
# .github/workflows/ci.yml
- name: Rebuild native modules for Electron
  run: npx @electron/rebuild -f -w serialport
```

### Option C: Use @electron-forge/plugin-auto-unpack-natives

Works when native modules are bundled normally (not with Vite). Less reliable with Vite plugin.

## CI Requirements

`electron-rebuild` compiles from source, so CI needs:
- **Windows:** Visual Studio Build Tools (included in `windows-latest` runner)
- **macOS:** Xcode Command Line Tools (included in `macos-latest` runner)
- **Linux:** `build-essential` + `python3` (included in `ubuntu-latest` runner)

## Key Lesson

> Every Electron project using native addons needs `electron-rebuild`. Without it, prebuilt binaries target Node.js, not Electron. This is the #1 cause of "No native build found" errors.

## ABI Reference

| Electron | Node | ABI |
|----------|------|-----|
| 28.x | 18.x | 116 |
| 30.x | 20.x | 121 |
| 33.x | 20.x | 125 |
| 41.x | 24.x | 145 |
