# Issue 003: Asar Integrity Fuses Block Unsigned Apps

**Severity:** High — app launches but renderer is blank  
**Category:** Security / Fuses  
**Affects:** Electron Forge with FusesPlugin  

## Symptom

```
Not allowed to load local resource: file:///...app.asar/.vite/renderer/main_window/index.html
```

Even though the file exists in the asar.

## Root Cause

Two fuse options require code signing to work:

```typescript
[FuseV1Options.EnableEmbeddedAsarIntegrityValidation]: true,
[FuseV1Options.OnlyLoadAppFromAsar]: true,
```

- `EnableEmbeddedAsarIntegrityValidation` validates asar checksum against an embedded hash. Without code signing, the hash doesn't exist → validation fails → file loading blocked.
- `OnlyLoadAppFromAsar` restricts all file loads to asar only. Combined with failed integrity, nothing loads.

## Fix

Disable both until code signing is configured:

```typescript
new FusesPlugin({
  version: FuseVersion.V1,
  [FuseV1Options.RunAsNode]: false,
  [FuseV1Options.EnableCookieEncryption]: true,
  [FuseV1Options.EnableNodeOptionsEnvironmentVariable]: false,
  [FuseV1Options.EnableNodeCliInspectArguments]: false,
  // Enable these ONLY after setting up code signing
  [FuseV1Options.EnableEmbeddedAsarIntegrityValidation]: false,
  [FuseV1Options.OnlyLoadAppFromAsar]: false,
}),
```

## When to Re-enable

1. Windows: after obtaining EV code signing certificate
2. macOS: after setting up Apple Developer notarization
3. Linux: not applicable (no code signing standard)

## Key Lesson

> Electron fuses for asar integrity are **code-signing dependent**. Don't enable them in dev/beta builds. They silently block file loading without clear error messages — the app just shows a white screen.
