# Issue 001: Renderer Not Included in Packaged App

**Severity:** Critical — app launches but shows blank white screen  
**Category:** Vite Config  
**Affects:** Electron Forge + Vite Plugin  

## Symptom

```
Not allowed to load local resource: file:///...app.asar/.vite/renderer/main_window/index.html
```

App starts (toolbar visible if rendered by main process), but renderer content is blank. DevTools shows the HTML file doesn't exist in the package.

## Root Cause

When `vite.renderer.config.ts` sets `root: path.resolve(__dirname, 'src/renderer')`, Vite outputs renderer build to `src/renderer/.vite/renderer/main_window/` (relative to the root).

But electron-forge Vite plugin expects renderer output at **project root** `.vite/renderer/main_window/`.

The packager only copies `.vite/` from the project root — so renderer files are left behind.

## Diagnosis

```bash
# Check what's actually in the package:
npx @electron/asar list out/AppName-platform/resources/app.asar
# Or without asar:
find out/AppName-platform/resources/app -type f

# Expected: .vite/renderer/main_window/index.html
# Actual: only .vite/build/main.js + .vite/build/preload.js
```

## Fix

Add explicit `outDir` to `vite.renderer.config.ts`:

```typescript
// vite.renderer.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'node:path';

export default defineConfig({
  plugins: [react()],
  root: path.resolve(__dirname, 'src/renderer'),
  build: {
    // CRITICAL: output to project-root .vite/ so electron-forge finds it
    outDir: path.resolve(__dirname, '.vite/renderer/main_window'),
    emptyOutDir: true,
  },
  resolve: {
    alias: {
      '@renderer': path.resolve(__dirname, 'src/renderer'),
      '@shared': path.resolve(__dirname, 'src/shared'),
    },
  },
});
```

## Key Lesson

> When using `root` in vite.renderer.config.ts, **always set `build.outDir` explicitly** to the project-root `.vite/renderer/<window_name>/`. Vite resolves `outDir` relative to `root`, not the project directory.

## Prevention

After `npm run package`, always verify:
```bash
find out/*/resources/app* -name "index.html"
# Must find: .vite/renderer/main_window/index.html
```
