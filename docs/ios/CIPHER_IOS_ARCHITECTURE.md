# CIPHER iOS — Architecture

**Source :** `Signal-iOS-main/`  
**Version marketing :** 8.32 (`Signal-Info.plist`)  
**Xcode :** 26.6 (`.xcode-version`) · Swift **5.0** · iOS **15.0+**  
**Date :** 2026-10-01  
**Lecture seule** — aucune modification fonctionnelle.

Référence : [Audit Signal-iOS architecture](b2036a71-0e96-48a8-93bc-0fbaa0fca034).

---

## Structure

| Couche | Chemin | Rôle |
|--------|--------|------|
| App | `Signal/` | UI, lancement, réglages, CallKit |
| Cœur | `SignalServiceKit/` | Crypto, messages, GRDB, réseau, groupes |
| UI partagée | `SignalUI/` | Composants conversation / médias |
| NSE | `SignalNSE/` | Notification Service Extension |
| Share | `SignalShareExtension/` | Share extension |
| Infra | `Config/`, `Scripts/`, `fastlane/`, `ci_scripts/`, `.github/` | Build & CI |

**Workspace :** ouvrir `Signal.xcworkspace` (pas le `.xcodeproj` seul).  
**Note locale :** sous-module `Pods` **non initialisé** dans cette copie — `make dependencies` requis.

---

## Targets & schemes

**Targets :** Signal · SignalTests · SignalServiceKit · SignalServiceKitTests · SignalUI · SignalUITests · SignalShareExtension · SignalNSE  

**Schemes :** Signal · Signal-Staging · SignalServiceKit · SignalShareExtension · SignalNSE  

**Configs :** Debug · App Store Release · Profiling · Testable Release  

---

## Langages

- **Swift** dominant (~2674 fichiers)  
- **Objective-C** résiduel (legacy SSK)  
- **Rust/C** via pods précompilés (LibSignalClient, RingRTC, SQLCipher) — pas de sources Rust dans l’app  

---

## Dépendances

**CocoaPods** (`Podfile` / `Podfile.lock`) :

| Pod | Version / source |
|-----|------------------|
| LibSignalClient | libsignal tag **v0.103.1** |
| SignalRingRTC | **v2.72.0** |
| GRDB.swift/SQLCipher | 5.26.0 |
| SQLCipher (fork Signal) | v4.6.1-f_barrierfsync |
| SwiftProtobuf | 1.38.1 |
| MobileCoin / LibMobileCoin | ~6.0.x |

Setup : `make dependencies` (pods + backup tests + RingRTC).  
SPM : uniquement outils sous `Scripts/` (pas l’app).

---

## Identifiants (critiques branding)

| Élément | Valeur défaut |
|---------|---------------|
| `SIGNAL_BUNDLEID_PREFIX` | `org.whispersystems` |
| App | `$(SIGNAL_BUNDLEID_PREFIX).signal` |
| Share | `…signal.shareextension` |
| NSE | `…signal.SignalNSE` |
| App Groups | `group.$(SIGNAL_BUNDLEID_PREFIX).signal.group` (+ `.staging`) |
| URL scheme | `sgnl` |
| Universal links | `signal.me`, `signal.group`, `signal.link`, … |

**SIGNAL COMPATIBILITY :** changer bundle ID / App Groups / Keychain trop tôt = **RISK**. Endpoints / libsignal = **BREAKING** si modifiés.

---

## Base & stockage

- GRDB + SQLCipher → `signal.sqlite` sous répertoire app / App Group  
- Keychain : `SSKKeychainStorage.swift`  
- Fichiers partagés via App Group (médias, logs NSE)  

---

## Réseau

`SignalServiceKit/Network/` — `NetworkManager`, WebSocket chat, `OWSSignalService` + **LibSignalClient.Net**.  
Endpoints prod : `TSConstants.swift` (`chat.signal.org`, CDN, storage, SVR2, SFU, …).

---

## Privacy UI

`Signal/src/ViewControllers/AppSettings/Privacy/`  
- `PrivacySettingsViewController.swift`  
- Screen Lock (`ScreenLock.swift` / `ScreenLockUI.swift`)  
- Advanced / Phone Number / Proxy / Block list  

Cible Cipher : Settings → Privacy & Security → Enhanced Privacy.
