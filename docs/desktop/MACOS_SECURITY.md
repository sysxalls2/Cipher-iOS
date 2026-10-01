# macOS — sécurité Desktop Cipher

**Date :** 2026-10-01  

| Sujet | Notes |
|-------|-------|
| Keychain / safeStorage | clé SQLCipher via Electron safeStorage |
| Hardened runtime | `build.mac.hardenedRuntime: true` |
| Entitlements | `build/entitlements.mac.plist` (+ MAS variants) |
| Notarization | pipeline amont ; secrets Apple dans CI |
| Code signing | `scripts/sign-macos.mjs` |
| Bundle | `.app` / DMG universal / MAS |
| Auto-update | `electron.autoUpdater` + signatures |
| OS min | macOS 13+ (vendor minOSVersion 22.1.0) |

**Cipher :** Team ID / provisioning propres ; ne pas réutiliser certificats Signal.
