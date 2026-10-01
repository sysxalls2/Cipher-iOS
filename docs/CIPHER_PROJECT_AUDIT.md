# CIPHER — Audit Projet (Signal-Android)

**Projet :** Cipher / Cipher Messenger  
**Base :** fork indépendant de Signal (open source) — **non officiel**  
**Périmètre :** `Signal-Android-main` uniquement pour les premières modifications  
**Date d’audit :** 2026-10-01  
**Version Android observée :** 8.30.0 (`canonicalVersionCode` 1760)  
**libsignal :** 0.103.0  

> Aucune modification fonctionnelle du code source n’a été effectuée pour cet audit.  
> Toutes les informations ci-dessous ont été vérifiées dans les fichiers du workspace.

---

## 1. Architecture générale

Monorepo Gradle multi-modules. Point d’entrée application : module `:app` (nom de projet Gradle `Signal-Android`).

```
Signal-Android-main/
├── app/                 # Application Android principale
├── core/                # util, util-jvm, models, models-jvm, network, ui, serialization
├── lib/                 # libsignal-service, network, media, billing, donations, …
├── feature/             # app-settings, registration, camera, media-send, …
├── demo/                # apps de démonstration / isolation UI
├── build-logic/         # plugins Gradle custom
├── lintchecks/, fast-lint/, benchmark/, baseline-profile/, microbenchmark/
├── gradle/              # catalogs (libs.versions.toml), verification-metadata
├── reproducible-builds/ # Docker builds reproductibles
└── wire-handler/        # plugin Wire
```

**Namespace / package applicatif :** `org.thoughtcrime.securesms`  
**Classe Application :** `org.thoughtcrime.securesms.ApplicationContext`

---

## 2. Langages utilisés

| Langage | Usage |
|---------|--------|
| Kotlin | Majorité du code moderne (settings, features, ViewModels, Compose) |
| Java | Legacy app (conversations, services, crypto glue, parties libsignal-service) |
| XML | Manifest, layouts, resources, navigation |
| Proto / Wire | Protocol buffers (`protowire/`) |
| Gradle Kotlin DSL | Build (`.kts`) |

---

## 3. Modules Gradle (inclus dans `settings.gradle.kts`)

### Application
- `:app` → projet nommé `Signal-Android`

### Core
- `:core:util`, `:core:util-jvm`, `:core:models`, `:core:models-jvm`
- `:core:network`, `:core:ui`, `:core:serialization`

### Lib
- `:lib:libsignal-service`, `:lib:network`, `:lib:glide`, `:lib:photoview`
- `:lib:sticky-header-grid`, `:lib:billing`, `:lib:paging`, `:lib:device-transfer`
- `:lib:donations`, `:lib:contacts`, `:lib:qr`, `:lib:spinner`, `:lib:video`
- `:lib:image-editor`, `:lib:debuglogs-viewer`, `:lib:blurhash`, `:lib:apng`
- `:lib:emoji`, `:lib:archive`, `:lib:ui-components`, `:lib:signal-login`
- `:lib:password-manager`

### Feature
- `:feature:app-settings`, `:feature:registration`, `:feature:camera`
- `:feature:media-send`, `:feature:chat-settings`, `:feature:media-keyboard`

### Qualité / perf
- `:lintchecks`, `:fast-lint`, `:benchmark`, `:baseline-profile`, `:microbenchmark`

---

## 4. Dépendances critiques

Source : `gradle/libs.versions.toml` + `app/build.gradle.kts`

| Dépendance | Version / note |
|------------|----------------|
| Android Gradle Plugin | 9.4.0 |
| Gradle Wrapper | 9.6.0 |
| Kotlin | 2.2.20 |
| Java / JVM target | 21 |
| compileSdk | 37 |
| targetSdk | 36 |
| minSdk | 23 |
| NDK | 28.0.13004108 |
| libsignal-client / libsignal-android | 0.103.0 |
| sqlcipher-android | 4.17.0 (`net.zetetic`) |
| Firebase Messaging | 25.0.1 (analytics/core exclus) |
| Compose BOM | 2026.09.00 |
| OkHttp / WebSocket | via couches network |
| Glide | 5.0.9 |
| Wire | 6.4.5 (classpath) |

Dépôts Maven Signal custom (sqlcipher, aesgcmprovider, build-artifacts.signal.org, MobileCoin).

---

## 5. Versions Android

| Paramètre | Valeur |
|-----------|--------|
| minSdk | **23** (Android 6.0) |
| targetSdk | **36** |
| compileSdk | **37** |
| buildTools | 36.0.0 |

---

## 6. libsignal

| Aspect | Détail |
|--------|--------|
| Artefacts | `org.signal:libsignal-android` + `org.signal:libsignal-client` 0.103.0 |
| Couche Java service | `lib/libsignal-service/` (`org.whispersystems.signalservice`) |
| Build from source | Optionnel via `libsignalClientPath` dans `gradle.properties` / settings — **absent du workspace** |
| Env réseau | `BuildConfig.LIBSIGNAL_NET_ENV` = PRODUCTION (staging en flavor staging) |

**COMPATIBILITÉ SIGNAL :** ne pas remplacer ni modifier libsignal sans autorisation explicite → **INCOMPATIBLE** si remplacé.

---

## 7. Stockage local

| Type | Emplacement / mécanisme |
|------|-------------------------|
| Base principale | `signal.db` (SQLCipher) — `SignalDatabase` |
| Secrets DB | `DatabaseSecret` |
| Secrets pièces jointes | `AttachmentSecret` / `AttachmentSecretProvider` / Keystore |
| Préférences typées | `SignalStore` (KeyValue) |
| Blobs temporaires | `BlobProvider` |
| Accès pièces jointes | `PartAuthority` / ContentProviders `${applicationId}.part`, `.blob`, `.avatar`, `.fileprovider` |
| Logs persistants | `LogDatabase` + `PersistentLogger` |
| Passphrase / master secret | `MasterSecretUtil`, `KeyCachingService` |

Chemin DB Android typique : `/data/data/org.thoughtcrime.securesms/databases/signal.db`

---

## 8. Base de données

- Classe : `org.thoughtcrime.securesms.database.SignalDatabase`
- OpenHelper SQLCipher : `net.zetetic.database.sqlcipher.SQLiteOpenHelper`
- Migrations : `SignalDatabaseMigrations`
- Tables critiques : messages, attachments, threads, identity, sessions, prekeys (incl. Kyber), groups, recipients, reactions, stickers, calls, backups, etc.

---

## 9. Stockage des médias

- Chiffrement local via `AttachmentSecret`
- Tables `AttachmentTable`, `MediaTable`, stickers
- Upload CDN via APIs network (`CdnService`, `AttachmentApi`)
- Thumbnails / Glide cache (à durcir en Phase stockage)

---

## 10. Notifications

- Privacy preference : `NotificationPrivacyPreference` — valeurs `all` / `contact` / sinon rien
- Services FCM : Firebase Messaging (auto-init désactivé dans le manifest)
- Actions custom : `org.thoughtcrime.securesms.notifications.*`

---

## 11. Appels

- WebRTC : `WebRtcCallActivity`, `ActiveCallManager`
- SFU : `https://sfu.voip.signal.org`
- Relais TURN optionnel : Advanced Privacy → always relay calls
- `FLAG_SECURE` applicable pendant les appels selon réglages

---

## 12. Services réseau

Voir `docs/CIPHER_NETWORK_AUDIT.md`.

Points d’entrée principaux :
- `SignalServiceNetworkAccess.kt`
- `NetworkDependenciesModule.kt`
- `lib/libsignal-service/`, `lib/network/`, `core/network/`

---

## 13. Gestion des clés

- Signal Protocol via libsignal (sessions, PreKeys, Kyber PreKeys, sender keys)
- Identités : `IdentityTable`
- SVR2 : `https://svr2.signal.org` + mrenclave dans BuildConfig
- Sealed sender / unidentified access : settings Advanced Privacy
- UNIDENTIFIED_SENDER_TRUST_ROOTS, ZKGROUP / GENERIC / BACKUP server public params dans BuildConfig

---

## 14. Mécanismes de sauvegarde

| Type | Détail |
|------|--------|
| Android Backup Service | `allowBackup="true"` + `SignalBackupAgent` |
| Contenu ABS | Actuellement **SvrAuthTokens** seulement (pas toute la DB) |
| Backups messages Signal | Local + remote (feature backups v2) — modules `lib/archive`, UI settings |
| Device transfer | `lib:device-transfer` |

---

## 15. Crash reporting

- Pas de Crashlytics / Sentry / Bugsnag produit
- Stockage local crashes / ANR dans `LogDatabase`
- Soumission volontaire debug logs (`debuglogs.org`, ShakeToReport)
- `firebase_analytics_collection_deactivated=true`
- `TRACING_ENABLED=false` (BuildConfig)

---

## 16. Analytics / télémétrie

- Firebase Analytics **désactivé** explicitement (dépendances excluées + meta-data)
- WebView metrics opt-out
- Pas d’Amplitude / Segment détecté comme SDK produit

---

## 17. Logs

| Composant | Chemin |
|-----------|--------|
| API | `org.signal.core.util.logging.Log` |
| Scrubber | `Scrubber.kt` |
| Android | `AndroidLogger.kt` |
| Persistent | `PersistentLogger.kt` |
| Protocol logger | `CustomSignalProtocolLogger` |

Voir audit sécurité pour classification SAFE / SENSITIVE.

---

## 18. Composants critiques (ne pas casser)

1. libsignal + formats protobuf / wire
2. `libsignal-service` message send/receive
3. Sessions / PreKeys / groupes V2
4. WebSocket messaging
5. CDN attachments
6. WebRTC / SFU
7. Registration / account attributes
8. Sealed sender
9. Storage service sync
10. URLs et trust roots serveur Signal officiel

---

## 19. Configuration de build

- Flavors : `distribution` ∈ {play, website, github, nightly} × `environment` ∈ {prod, staging}
- Variante de développement recommandée : **`playProdDebug`**
- Signing debug : `keystore.debug.properties` (si présent)
- ABI splits : armeabi-v7a, arm64-v8a, x86, x86_64 + universal APK
- Tâches QA : `qa`, `ci`, `ktlintCheck`

---

## 20. Signatures

- Config debug optionnelle via `keystore.debug.properties`
- Release : signatures non présentes dans le workspace (normal)
- Permission signature : `${applicationId}.ACCESS_SECRETS`

---

## 21. Package ID

| Élément | Valeur actuelle |
|---------|-----------------|
| namespace | `org.thoughtcrime.securesms` |
| applicationId | *implicite = namespace* → `org.thoughtcrime.securesms` |
| Cible Cipher envisagée | `org.cipher.messenger` (**non appliquée**) |

**Conséquences d’un changement d’applicationId** (à documenter avant action) :
- Coexistence possible avec Signal officiel (souhaitable pour tests)
- Deep links `sgnl` / `signal.me` en conflit potentiel
- ContentProviders, account sync, shortcuts, FCM
- Pas de migration automatique des données de l’ancien package
- **COMPATIBILITÉ RÉSEAU : COMPATIBLE** si endpoints inchangés  
- **COMPATIBILITÉ EXPÉRIENCE / DEVICE : RISQUE**

---

## 22. Manifest

Fichier : `app/src/main/AndroidManifest.xml`

Points notables :
- Nombreuses permissions (contacts, micro, caméra, localisation, FGS, …)
- Deep links : `sgnl://`, `https://signal.me`, `tsdevice`
- Backup agent activé
- Firebase messaging auto-init **false**
- Analytics désactivés

---

## 23. Ressources UI

`app/src/main/res/` :
- `layout/`, `drawable*`, `mipmap*` (icône + alternatives “disguise”)
- `values/` + locales nombreuses, `values-night/`
- `navigation/` (graphs Privacy, settings, …)
- `xml/` (authenticator, syncadapter, shortcuts, file paths)

`app_name` actuel : **Signal** (`strings.xml`)

---

## 24. Architecture UI

- Navigation principale : `MainActivity` / `MainNavigator`
- Settings : DSL settings + Compose fragments
- Conversation list / conversation (Views historiques)
- Features Compose croissantes (`feature/*`, `core/ui`)
- Appels : activité dédiée WebRTC

Privacy UI existante (à étendre pour Cipher Enhanced Privacy) :
```
Settings → Privacy
  ├── Phone number privacy
  ├── Read receipts / typing
  ├── Screen lock
  ├── Screen security
  ├── Incognito keyboard
  └── Advanced
       ├── Always relay calls
       ├── Censorship circumvention
       └── Sealed sender options
```

---

## 25. Licence et marque

| Fichier | Contenu |
|---------|---------|
| `LICENSE` | GNU AGPLv3 |
| `NOTICE` | Dépendances tierces (Bouncy Castle, ZXing, …) |
| `README.md` | Copyright Signal Messenger, LLC ; trademarks Google Play |

**Obligations Cipher :**
- Conserver notices AGPL / copyright
- Ne jamais se présenter comme Signal officiel
- Marque « Signal » : ne pas utiliser de façon trompeuse

---

## 26. Autres dépôts du workspace (référence)

| Dépôt | Rôle | Action Phase 0–1 |
|-------|------|------------------|
| Signal-Desktop-main (imbriqué) | Client Desktop Electron/TS 8.31.0-alpha.1 | Inspecter seulement |
| Signal-iOS-main | Client iOS | Inspecter seulement |
| Signal-Server-main | Backend Maven/Java | Référence ; pas de serveur Cipher |
| foundationdb-7.4.8 (imbriqué) | Stockage serveur | Dépendance serveur |

Idées serveur indépendant → `docs/FUTURE_CIPHER_NETWORK.md` uniquement.

---

## 27. État Git / environnement (machine d’audit)

- Racine workspace : **pas** un repo Git
- `Signal-Android-main` : **pas** de `.git` (sources type archive)
- `git` / `java` / `adb` / Android SDK : **absents du PATH** au moment de l’audit
- Package manager disponible : **winget**

---

## 28. Méthode de compilation (upstream)

```bat
cd Signal-Android-main
gradlew.bat :Signal-Android:assemblePlayProdDebug
```

Prérequis : JDK 21, Android SDK, NDK 28.0.13004108, CMake, accès Maven.

---

*Document généré dans le cadre de la PHASE 0 — Cipher. Prochaine étape docs : sécurité, réseau, roadmap, branding.*
