# CIPHER Desktop — Build

**Racine :** `Signal-Desktop-main/Signal-Desktop-main/`  

---

## Prérequis

- Node **24.19.0** (Volta / engines)  
- pnpm **11.24.0**  
- Python / build tools natifs (modules `.node`)  
- OS cible pour packaging (Windows / macOS / Linux)

---

## Commandes utiles

```bash
cd Signal-Desktop-main/Signal-Desktop-main
pnpm install
pnpm run generate          # protobuf, locales, styles, …
pnpm start                 # electron .
pnpm test                  # node + electron + lint
pnpm run build-win32-all   # Windows NSIS arm64+x64
pnpm run build-linux       # Linux packages
pnpm run build:release     # release générique
pnpm run build:release:mas # macOS App Store
```

Sortie typique : dossier `release/`.

---

## CI prévue (à créer, pas encore dans le repo Cipher)

- `.github/workflows/cipher-desktop-windows.yml`  
- `.github/workflows/cipher-desktop-macos.yml`  
- `.github/workflows/cipher-desktop-linux.yml`  

Tous en `workflow_dispatch` (déclenchement iPad).

Secrets : voir `docs/CI_SECRETS.md` (à créer) — **jamais** de certificats privés dans Git.

---

## Branding build (plus tard)

| Champ | Upstream | Cible Cipher |
|-------|----------|--------------|
| productName | Signal | Cipher |
| appId | org.whispersystems.signal-desktop | à décider (ex. org.cipher.messenger.desktop) |
| executableName | signal-desktop | cipher-desktop |
| protocols | sgnl | garder + éventuellement cipher |

Changer `appId` trop tôt = **RISK** (données userData, coexistence).

---

## Logo

Source Cipher : `LOGO.png` (racine workspace).  
À dériver en `.ico` (Windows), `.icns` (macOS), PNG (Linux) — **sans déformation**.
