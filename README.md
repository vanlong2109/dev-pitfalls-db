# Electron Forge Lessons Learned

Common pitfalls and solutions for **Electron Forge + Vite + native modules**. Documented from real production issues to avoid repeating mistakes across projects.

## Issues

| # | Issue | Severity | Category |
|---|-------|----------|----------|
| 1 | [Renderer not included in packaged app](issues/001-renderer-missing-from-package.md) | Critical | Vite Config |
| 2 | [Native module "Cannot find module" in production](issues/002-native-module-not-found.md) | Critical | Native Modules |
| 3 | [Asar integrity fuses block unsigned apps](issues/003-asar-fuses-block-unsigned.md) | High | Security/Fuses |
| 4 | [serialport ABI mismatch in Electron](issues/004-serialport-abi-mismatch.md) | High | Native Modules |

## Quick Reference

### Electron Forge + Vite Checklist

- [ ] Renderer `outDir` points to `.vite/renderer/<window_name>/` at **project root**
- [ ] Native modules loaded via `eval('require')` to bypass Vite bundler
- [ ] Native modules copied to package via `postPackage` hook
- [ ] `electron-rebuild` in postinstall for correct ABI
- [ ] Asar integrity fuses **disabled** until code signing is set up
- [ ] `asar: false` if renderer files excluded from asar

## Project

Maintained by [@vanlong2109](https://github.com/vanlong2109). Issues discovered during SkyPower IDE development.
