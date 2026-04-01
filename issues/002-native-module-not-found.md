# Issue 002: Native Module "Cannot find module" in Production

**Severity:** Critical — app crashes on startup  
**Category:** Native Modules  
**Affects:** Any native Node.js addon (serialport, sqlite3, sharp, etc.) with Electron Forge + Vite  

## Symptom

```
Error: Cannot find module 'serialport'
Require stack: .../app.asar/.vite/build/main.js
```

Works in `npm start` (dev mode) but crashes in packaged app.

## Root Cause

Three-layer failure:

1. **If NOT externalized:** Vite bundles serialport's JS into main.js, but `.node` native binary can't be bundled. Path resolution for the binary breaks → "No native build found"

2. **If externalized:** Vite leaves `require('serialport')` as-is. But electron-forge Vite plugin only packages `.vite/` output — `node_modules/` is NOT included in the asar/app. So `require('serialport')` fails → "Cannot find module"

3. **ABI mismatch:** Even if the module is found, prebuilt binaries from npm target Node.js ABI, not Electron ABI → "No native build found for runtime=electron"

## Fix (3 parts)

### Part 1: Lazy-load with eval to bypass Vite

```typescript
// src/main/serial.ts
import path from 'node:path';

let SerialPortClass: any = null;
let available = false;

function loadSerialPort(): void {
  const candidates = [
    // Packaged: native_modules/ copied by postPackage hook
    path.join(process.resourcesPath ?? '', '..', 'native_modules', 'node_modules', 'serialport'),
    // Dev mode
    'serialport',
  ];
  for (const candidate of candidates) {
    try {
      const m = eval('require')(candidate);  // eval prevents Vite from processing
      SerialPortClass = m.SerialPort ?? m;
      if (SerialPortClass?.list) { available = true; return; }
    } catch { continue; }
  }
}
```

### Part 2: Copy native modules in postPackage hook

```typescript
// forge.config.ts
hooks: {
  postPackage: async (_config, result) => {
    for (const outputPath of result.outputPaths) {
      const srcNM = path.join(process.cwd(), 'node_modules');
      const destNM = path.join(outputPath, 'native_modules', 'node_modules');
      for (const mod of ['serialport', '@serialport']) {
        const src = path.join(srcNM, mod);
        if (fs.existsSync(src)) {
          fs.cpSync(src, path.join(destNM, mod), { recursive: true });
        }
      }
    }
  },
},
```

### Part 3: electron-rebuild for correct ABI

```json
// package.json
"scripts": {
  "postinstall": "electron-rebuild -f -w serialport || true"
}
```

## Key Lesson

> Native Node.js modules in Electron + Vite require **three things**: (1) bypass Vite's bundler with `eval('require')`, (2) copy modules alongside the package, (3) rebuild for Electron's ABI. Missing any one causes a different but equally fatal error.

## Prevention

- Always test `npm run make` → install on clean machine before shipping
- Add graceful degradation: app must launch even if native module fails
