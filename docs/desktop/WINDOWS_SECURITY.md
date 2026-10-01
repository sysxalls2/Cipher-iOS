# Windows — sécurité Desktop Cipher

**Date :** 2026-10-01  

| Sujet | Notes upstream / Cipher |
|-------|-------------------------|
| Installer | NSIS via electron-builder |
| Code signing | `scripts/sign-windows.mjs` + secrets CI |
| AppData | `%APPDATA%\<productName>` |
| LocalAppData | caches / update staging |
| Registry | protocol handlers, uninstall |
| Auto-start | option tray / login item |
| Notifications | `@indutny/simple-windows-notifications` |
| Credential / clé DB | `safeStorage` (DPAPI sous-jacent Electron) |
| OS min | Windows 10 build 10240+ |

**Cipher :** userData séparé de Signal officiel ; notariser/signing avec certificat Cipher ; pas de secrets dans le repo.
