# Linux — sécurité Desktop Cipher

**Date :** 2026-10-01  

| Sujet | Notes |
|-------|-------|
| Packages | deb (défaut), AppImage optionnel |
| Config | `~/.config/<productName>` |
| Secret storage | `safeStorage` + backend Linux (suivi `safeStorageBackend`) |
| Keyring | dépend du DE (GNOME/KWallet) |
| Desktop file | `signal.desktop` → `cipher.desktop` plus tard |
| glibc | ≥ 2.34 |
| Permissions / policy files | `org.signalapp.*.policy` dans extraResources |

**Cipher :** documenter AppImage vs deb ; ne pas forcer keyring propriétaire.
