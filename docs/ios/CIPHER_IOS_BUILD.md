# CIPHER iOS — Build

**Date :** 2026-10-01  

---

## Prérequis machine

- macOS + **Xcode 26.6** (aligné `.xcode-version`)  
- Ruby **3.4.9** (`.ruby-version`) + Bundler  
- Clone **avec sous-modules** (`git clone --recurse-submodules`)  
- Compte Apple Developer  

Cette copie workspace : `Pods/` **vide** → `make dependencies` obligatoire.

---

## Build local (upstream)

```bash
cd Signal-iOS-main
make dependencies
open Signal.xcworkspace
```

1. Team = ton Apple Developer (Signal + Share + NSE)  
2. Ajuster capabilities selon `BUILDING.md`  
3. Préfixe bundle : `SIGNAL_BUNDLEID_PREFIX`  
4. Scheme **Signal** → Build & Run  

Staging : scheme **Signal-Staging** (`USE_STAGING=1`).

---

## Script CI amont

`.github/workflows/main.yml` · `Scripts/build-and-test.sh`  
Runners macOS lourds · matrice Xcode · action `clone-everything` (parfois `ACCESS_TOKEN` pour pods privés).

---

## IPA

Pas de lane Fastfile IPA complète dans le dépôt public. Flux typique :

1. Config **App Store Release**  
2. Archive (entitlements App Store)  
3. Export IPA / TestFlight  

Signature / provisioning = **GitHub Secrets** uniquement (voir `docs/CI_SECRETS.md` à créer).

---

## Logo Cipher

Source : `LOGO.png` (racine workspace).  
À intégrer dans `Signal/AppIcons/` / Asset Catalog — **sans déformer** — après autorisation branding.
