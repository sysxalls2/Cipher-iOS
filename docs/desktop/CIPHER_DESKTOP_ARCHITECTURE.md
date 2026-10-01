# CIPHER Desktop — Architecture

**Source :** `Signal-Desktop-main/Signal-Desktop-main/` (dossier imbriqué)  
**Version observée :** 8.31.0-alpha.1  
**Licence :** AGPL-3.0-only  
**Date audit :** 2026-10-01  
**Lecture seule** — aucune modification fonctionnelle dans cet audit.

Référence détaillée : inspection [Audit Signal-Desktop architecture](5d1df6a5-cb3d-42bf-8191-d8e4fdf90a60).

---

## Identité package

| Champ | Valeur |
|-------|--------|
| `name` | `signal-desktop` |
| `productName` | `Signal` |
| `appId` (electron-builder) | `org.whispersystems.signal-desktop` |
| `main` | `bundles/main.js` |
| `desktopName` | `signal.desktop` |

---

## Stack

| Couche | Techno |
|--------|--------|
| Shell | Electron **44.2.0** |
| Node | **24.19.0** |
| Package manager | pnpm **11.24.0** |
| UI | React **19.2.4** + Redux + react-intl |
| Styles | Sass + Tailwind 4 |
| Bundler | Rolldown |
| Packaging | electron-builder **26.11.1** |
| Crypto client | `@signalapp/libsignal-client` **0.101.2** |
| DB | `@signalapp/sqlcipher` **4.1.0** |
| Appels | `@signalapp/ringrtc` **2.72.0** |

---

## Architecture processus

```
┌─────────────────────────────────────────────┐
│ Main (app/main.main.ts → bundles/main.js)   │
│  fenêtres, IPC, SQL workers, updates, tray  │
└───────────────┬─────────────────────────────┘
                │ IPC
┌───────────────▼─────────────────────────────┐
│ Preload (bundles/preload/wrapper.js)        │
│  Signal Protocol glue, Redux, jobs, WS      │
└───────────────┬─────────────────────────────┘
                │ contextBridge (limité)
┌───────────────▼─────────────────────────────┐
│ Renderer / DOM (React)                      │
│  background.html + fenêtres auxiliaires     │
└─────────────────────────────────────────────┘
```

### Suffixes de fichiers (convention projet)
- `.main.ts` — processus main Electron  
- `.preload.ts` — preload  
- `.node.ts` — Node natif  
- `.dom.tsx` — UI renderer sandboxée  
- `.std.ts` — partagé  

### Dossiers clés
| Chemin | Rôle |
|--------|------|
| `app/` | Main process |
| `ts/` | Preload + UI + métier |
| `ts/textsecure/` | API / WebSocket |
| `ts/sql/` | SQLCipher (main workers + client preload) |
| `ts/updater/` | Auto-update |
| `config/*.json` | URLs serveur, trust roots, flags |
| `protos/` | Protobuf wire format |
| `images/` | Assets / logo Signal actuel |

---

## Main / Preload / Renderer

**Main :** fenêtres, tray, menus, `safeStorage`, init SQL (4 workers), protocoles custom, attachments.

**Preload principal :** charge la logique applicative lourde (pas un renderer Node classique). Wrapper VM : `preload.wrapper.ts`.

**Renderer :** React monté depuis `ts/background.preload.ts` ; fenêtres secondaires (about, PDF, screen share, debug log, etc.).

---

## IPC (aperçu)

- SQL : `sql-channel:read` / `write` / `erase-sql-key` (`app/sql_channel.main.ts`)
- Attachments : `app/attachment_channel.main.ts`
- Config / locale : sync au boot
- Updater : `ts/updater/common.main.ts`

**Règle Cipher :** ne pas élargir l’IPC sans audit ; préférer réduire la surface.

---

## libsignal & compatibilité

- `@signalapp/libsignal-client` 0.101.2 + prebuilds `.node`
- Stores : `ts/LibSignalStores.node.ts`, `SignalProtocolStore.preload.ts`
- RingRTC 2.72.0 pour appels
- Protobufs : `SignalService.proto`, Groups, DeviceMessages, Backups, …

**SIGNAL COMPATIBILITY :** modifier libsignal / protos / trust roots = **BREAKING**.

---

## Base locale

- Fichier : `{userData}/sql/db.sqlite` (+ WAL)
- Accès SQL **uniquement dans le main** via workers
- Clé SQLCipher : 32 octets, stockée chiffrée via `electron.safeStorage` (`encryptedKey` dans `config.json`)

---

## Réseau

Config : `config/default.json` / `production.json` — `serverUrl`, CDN, SFU, storage, updates, public params ZK/backup.

Client : `ts/textsecure/*` + SocketManager libsignal.

**Cipher phase 1 :** garder endpoints Signal officiels = **SAFE** pour interop utilisateurs Signal.

---

## Packaging & plateformes

| OS | Artefacts | Notes |
|----|-----------|--------|
| Windows | NSIS x64/arm64 | `build-win32-all`, signature `scripts/sign-windows.mjs` |
| macOS | zip, dmg universal, MAS | hardenedRuntime, entitlements |
| Linux | deb (défaut), AppImage optionnel | `prepare-linux-build`, glibc ≥ 2.34 |

Protocols : `sgnl`, `signalcaptcha`.

Auto-update : `updates.signal.org` / `updates2.signal.org` — Cipher devra un canal **propre** plus tard (**RISK** si on conserve l’URL Signal).

---

## Sécurité Electron (constat)

| Pref | Fenêtre principale | Auxiliaires |
|------|--------------------|-------------|
| nodeIntegration | false | false |
| contextIsolation | true (prod) | true |
| sandbox | **false** | true |

CSP stricte dans HTML · Electron fuses dans `after-pack` (RunAsNode off, ASAR integrity, etc.).

Voir aussi `CIPHER_DESKTOP_SECURITY_AUDIT.md`.
