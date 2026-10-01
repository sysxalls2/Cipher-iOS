# Cipher — Documentation

Documentation du projet **Cipher Messenger** (fork indépendant basé sur Signal open source).

**Cipher n’est pas une application Signal officielle.**

## Phase 0 — Audits

| Document | Description |
|----------|-------------|
| [CIPHER_PROJECT_AUDIT.md](./CIPHER_PROJECT_AUDIT.md) | Architecture, modules, build, stockage, UI |
| [CIPHER_SECURITY_AUDIT.md](./CIPHER_SECURITY_AUDIT.md) | Sécurité / confidentialité locale |
| [CIPHER_NETWORK_AUDIT.md](./CIPHER_NETWORK_AUDIT.md) | Connexions réseau & proxy |
| [CIPHER_ROADMAP.md](./CIPHER_ROADMAP.md) | Phases 0–12 |
| [CIPHER_BRANDING_PLAN.md](./CIPHER_BRANDING_PLAN.md) | Branding & applicationId |
| [CIPHER_COMPATIBILITY_MATRIX.md](./CIPHER_COMPATIBILITY_MATRIX.md) | Tests interop Signal officiel |
| [FUTURE_CIPHER_NETWORK.md](./FUTURE_CIPHER_NETWORK.md) | Serveur Cipher (hors scope actuel) |
| [CIPHER_BUILD_ENVIRONMENT.md](./CIPHER_BUILD_ENVIRONMENT.md) | Toolchain & compilation |

## Desktop

| Document | Description |
|----------|-------------|
| [desktop/CIPHER_DESKTOP_ARCHITECTURE.md](./desktop/CIPHER_DESKTOP_ARCHITECTURE.md) | Electron / React / IPC / libsignal |
| [desktop/CIPHER_DESKTOP_SECURITY_AUDIT.md](./desktop/CIPHER_DESKTOP_SECURITY_AUDIT.md) | Sécurité Electron & privacy |
| [desktop/CIPHER_DESKTOP_STORAGE_AUDIT.md](./desktop/CIPHER_DESKTOP_STORAGE_AUDIT.md) | SQLCipher, userData, caches |
| [desktop/CIPHER_DESKTOP_NETWORK_AUDIT.md](./desktop/CIPHER_DESKTOP_NETWORK_AUDIT.md) | Endpoints & compatibilité |
| [desktop/CIPHER_DESKTOP_BUILD.md](./desktop/CIPHER_DESKTOP_BUILD.md) | Build Windows / macOS / Linux |
| [desktop/WINDOWS_SECURITY.md](./desktop/WINDOWS_SECURITY.md) | Notes Windows |
| [desktop/MACOS_SECURITY.md](./desktop/MACOS_SECURITY.md) | Notes macOS |
| [desktop/LINUX_SECURITY.md](./desktop/LINUX_SECURITY.md) | Notes Linux |

## Règle de compatibilité

Toute modification susceptible d’affecter le réseau doit être classée :

- **COMPATIBLE**
- **RISQUE DE COMPATIBILITÉ**
- **INCOMPATIBLE** (autorisation explicite requise)
