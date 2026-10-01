# CIPHER — Audit Sécurité Locale (Signal-Android)

**Projet :** Cipher Messenger (fork indépendant, non officiel)  
**Base auditée :** `Signal-Android-main` v8.30.0  
**Date :** 2026-10-01  
**Périmètre :** confidentialité locale, stockage, logs, UI protections — **pas** de crypto maison  

**Légende sévérité :** CRITICAL · HIGH · MEDIUM · LOW · INFORMATIONAL  

**COMPATIBILITÉ SIGNAL (globale de cet audit) :** les corrections proposées ici sont **COMPATIBLES** si elles restent locales et ne modifient pas protocole / endpoints / formats.

---

## Synthèse des protections déjà présentes

Signal Android embarque déjà :

| Fonctionnalité | Emplacement |
|----------------|-------------|
| Screen lock biométrique + timeout | `privacy/screenlock/*` |
| Screen security (`FLAG_SECURE`) | `PrivacySettingsFragment`, `WindowExtensions`, appels |
| Temporary screenshot security | `TemporaryScreenshotSecurity.kt` |
| Read receipts / typing indicators OFF | `PrivacySettingsFragment` |
| Incognito keyboard | `PrivacySettingsFragment` |
| Sealed sender / relay calls / censorship | `AdvancedPrivacySettings*` |
| Notification privacy `all` / `contact` / rien | `NotificationPrivacyPreference` |
| Clipboard sensible + clear timer (60s défaut) | `Util.copyToClipboardSensitive` |
| DB SQLCipher + AttachmentSecret | `SignalDatabase`, crypto utils |
| Scrubber de logs | `Scrubber.kt` |
| Analytics Firebase désactivés | Manifest + exclusions Gradle |
| Proxy utilisateur optionnel | `ProxyValues`, `EditProxy*` |

→ **Enhanced Privacy Cipher** doit **composer** avec ces réglages, pas les dupliquer aveuglément.

---

## Findings

### SEC-01 — `allowBackup="true"` + BackupAgent
| Champ | Contenu |
|-------|---------|
| **Sévérité** | HIGH |
| **Fichier** | `app/src/main/AndroidManifest.xml`, `absbackup/SignalBackupAgent.kt` |
| **Classe** | `SignalBackupAgent` |
| **Comportement actuel** | Backup Android activé ; agent sauvegarde `SvrAuthTokens` (key-value ABS), pas la DB complète |
| **Risque** | Fuite de jetons/auth SVR via Android Backup / transferts constructeur si mal configuré ; surface d’attaque élargie |
| **Correction proposée** | Option Cipher « Limit Android backups » ; documenter contenu exact ; envisager `allowBackup=false` en Enhanced Privacy **avec** avertissement UX (casse restore tokens) |
| **Impact Signal** | COMPATIBLE (local) — RISQUE UX restore |
| **Difficulté** | Moyenne |
| **Tests** | Restore compte, SVR PIN, backup/restore messages Signal |

### SEC-02 — Notifications : pas de niveau HIDDEN total dédié Cipher
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | `preferences/widgets/NotificationPrivacyPreference.java` + message notifier |
| **Comportement** | `all` = contact+message ; `contact` = expéditeur ; sinon rien affiché |
| **Risque** | Fuite métadonnées (nom) si `contact` ; contenu si `all` sur écran verrouillé |
| **Correction** | Niveaux Cipher : FULL / SENDER_ONLY / GENERIC (« Cipher — Nouveau message ») / HIDDEN ; mapper sur l’existant |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Faible–moyenne |
| **Tests** | Notifs 1:1, groupe, appel, reply wear |

### SEC-03 — Screen security optionnelle (OFF par défaut possible)
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | `PrivacySettingsFragment.kt`, `SettingsValues` / `SignalStore.settings` |
| **Comportement** | Toggle « Screen security » ajoute/retire `FLAG_SECURE` |
| **Risque** | Captures / screen recording / aperçu Recents si désactivé |
| **Correction** | Enhanced Privacy → ON par défaut ; rester configurable |
| **Impact Signal** | COMPATIBLE ; UX partage d’écran volontaire impacté |
| **Difficulté** | Faible |
| **Tests** | Screenshot bloqué, PiP appel, share screen |

### SEC-04 — Aperçu multitâche (Recents)
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | lié à `FLAG_SECURE` / thèmes ; appels `excludeFromRecents` sur certaines activités |
| **Comportement** | `FLAG_SECURE` masque aussi l’aperçu Recents sur la plupart des OEM |
| **Risque** | Sans screen security, contenu conversation visible dans Recents |
| **Correction** | Option « Hide content in Recents » (= screen security ou overlay dédié API récente) |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Faible (réutilise FLAG_SECURE) à moyenne (overlay custom) |
| **Tests** | API 23–36, OEM Samsung/Pixel |

### SEC-05 — Presse-papiers : clear seulement pour « secrets »
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | `core/util/.../Util.java` |
| **Fonctions** | `copyToClipboard` vs `copyToClipboardSensitive` (timeout défaut **60s**) |
| **Risque** | Copies de messages/numéros via chemin non-sensitive restent indéfiniment |
| **Correction** | Réglage Cipher : 30s / 1m / 5m / Jamais pour copies conversation ; réutiliser alarm `ClearClipboardAlarmReceiver` |
| **Impact Signal** | COMPATIBLE — restrictions API 33+ (preview déjà gérée pour sensitive) |
| **Difficulté** | Moyenne |
| **Tests** | Copy message, copy number, paste autre app après timeout |

### SEC-06 — Logs persistants locaux
| Champ | Contenu |
|-------|---------|
| **Sévérité** | HIGH (si contenu non scrubbé) / MEDIUM (volume) |
| **Fichier** | `logging/PersistentLogger.kt`, `database/LogDatabase`, `Scrubber.kt` |
| **Comportement** | Logs écrits localement ; scrubber pour IPs, e.164, etc. ; upload optionnel debuglogs.org |
| **Risque** | Rétention longue ; soumission utilisateur peut exposer métadonnées ; debug builds plus verbeux |
| **Correction** | Mode production Cipher : rétention courte ; interdire upload auto ; audit lignes SENSITIVE ; abstraction log restrictive |
| **Impact Signal** | COMPATIBLE si protocole logger inchangé |
| **Difficulté** | Moyenne–élevée (volume) |
| **Tests** | Scrubber unit tests existants ; grep CI STOPSHIP / secrets |

### SEC-07 — Soumission debug logs / ShakeToReport
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | `ApplicationContext`, `ShakeToReport`, flux debuglogs |
| **Risque** | Exfiltration volontaire de logs vers infra Signal (`debuglogs.org`) |
| **Correction** | Enhanced Privacy : désactiver ShakeToReport / demander confirmation forte ; documenter destination |
| **Impact Signal** | COMPATIBLE (support plus difficile) |
| **Difficulté** | Faible |
| **Tests** | Shake désactivé, écran submit logs |

### SEC-08 — Caches médias / thumbnails / Glide / ExoPlayer
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichiers** | modules Glide, `SimpleExoPlayerPool`, jobs OptimizeMedia, fichiers cache app |
| **Risque** | Médias/thumbs restent après « suppression » apparente ; caches non chiffrés au même niveau que DB |
| **Correction** | TTL cache ; purge agressive Enhanced Privacy ; auto-delete médias locaux optionnel |
| **Impact Signal** | COMPATIBLE ; perf/réseau si re-téléchargement |
| **Difficulté** | Élevée |
| **Tests** | Envoi/réception image, delete message, vérifier filesystem |

### SEC-09 — Fichiers temporaires BlobProvider
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | BlobProvider / `initializeBlobProvider` dans `ApplicationContext` |
| **Risque** | Blobs orphelins après crash |
| **Correction** | Job de GC au démarrage / intervalle court en Enhanced Privacy |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Moyenne |
| **Tests** | Enregistrement audio interrompu, partage fichier |

### SEC-10 — WebView (captcha, debuglogs viewer, donations?)
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichiers** | captcha URLs BuildConfig ; `lib/debuglogs-viewer` ; registration WebView tests |
| **Risque** | Surface JS ; fuites URL ; captcha tiers (`signalcaptchas.org`) |
| **Correction** | Auditer JS bridges ; pas de WebView hors cas nécessaires ; MetricsOptOut déjà true |
| **Impact Signal** | RISQUE si captcha cassé (registration) |
| **Difficulté** | Moyenne |
| **Tests** | Registration captcha, open debug log viewer |

### SEC-11 — Intents exportés / deep links Signal
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM (sécurité) / HIGH (branding si inchangé) |
| **Fichier** | `AndroidManifest.xml` — `sgnl`, `signal.me`, `tsdevice`, ShareActivity |
| **Risque** | Intent spoofing ; confusion avec app Signal officielle ; handlers partagés |
| **Correction** | Documenter ; à terme schemes Cipher **en plus** sans casser compat si possible ; ne pas usurper marque |
| **Impact Signal** | Changer schemes = **RISQUE DE COMPATIBILITÉ** (lien device, usernames) |
| **Difficulté** | Élevée |
| **Tests** | Link device, open signal.me profile |

### SEC-12 — ContentProviders (`part`, `blob`, `avatar`, `fileprovider`)
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | Manifest authorities `${applicationId}.*` |
| **Risque** | Accès cross-app si mal exportés ; URIs temporaires |
| **Correction** | Vérifier `exported=false` + permissions signature ; TTL URIs |
| **Impact Signal** | COMPATIBLE si applicationId stable ; RISQUE si renommage package |
| **Difficulté** | Moyenne |
| **Tests** | Share attachment vers autre app, widget avatar |

### SEC-13 — Stockage externe / permissions media
| Champ | Contenu |
|-------|---------|
| **Sévérité** | LOW–MEDIUM |
| **Fichier** | Manifest `WRITE_EXTERNAL_STORAGE` maxSdk 28 ; READ_MEDIA_* |
| **Risque** | Exports utilisateur hors coffre chiffré |
| **Correction** | Désactiver auto-download ; clarifier exports ; purge |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Moyenne |
| **Tests** | Save attachment, scoped storage API 29+ |

### SEC-14 — Secrets et Keystore
| Champ | Contenu |
|-------|---------|
| **Sévérité** | INFORMATIONAL (bon) / HIGH si réinvention |
| **Fichiers** | `DatabaseSecret`, `AttachmentSecret`, `MasterSecretUtil`, `KeyCachingService` |
| **Comportement** | SQLCipher + secrets attachments ; lock via passphrase/biometric |
| **Risque** | Faible si on **réutilise** ; critique si crypto maison |
| **Correction** | Durcir timeouts Extended Privacy ; **interdire** nouvel algo |
| **Impact Signal** | Remplacer crypto = **INCOMPATIBLE** |
| **Difficulté** | N/A (conserver) |
| **Tests** | Lock/unlock, wrong PIN, DB open |

### SEC-15 — Passphrase / KeyCachingService
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | `KeyCachingService`, privacy passphrase settings |
| **Risque** | Fenêtre mémoire où secrets sont unlock |
| **Correction** | Timeout plus agressif en Enhanced Privacy |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Faible |
| **Tests** | Background timeout, notification while locked |

### SEC-16 — Giphy / Maps / Stripe clés dans client
| Champ | Contenu |
|-------|---------|
| **Sévérité** | LOW (clés publiables) / MEDIUM (métadonnées) |
| **Fichier** | `app/build.gradle.kts` BuildConfig ; Manifest Maps |
| **Risque** | Requêtes tiers révèlent IP + usage ; Maps key embarquée |
| **Correction** | Link previews OFF ; Giphy opt-in ; documenter |
| **Impact Signal** | COMPATIBLE ; features réduites |
| **Difficulté** | Faible |
| **Tests** | Link preview, location attach, donation |

### SEC-17 — Fichiers après suppression de messages
| Champ | Contenu |
|-------|---------|
| **Sévérité** | HIGH |
| **Fichiers** | attachment delete paths, optimize media jobs |
| **Risque** | Données résiduelles (copies, thumbs, logs) |
| **Correction** | Secure delete pipeline ; Clear Local Data confirmé |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Élevée |
| **Tests** | Delete for me, view-once, expire timer |

### SEC-18 — Clear Local Data (à créer)
| Champ | Contenu |
|-------|---------|
| **Sévérité** | INFORMATIONAL (gap produit) |
| **Comportement** | Pas d’équivalent Cipher « effacement rapide local » distinct de delete account |
| **Risque** | Utilisateur croit être « clean » sans l’être |
| **Correction** | Settings → Privacy → Clear Local Data + double confirmation ; **pas** de trigger caché |
| **Impact Signal** | COMPATIBLE (local only) — ne pas appeler delete account serveur par erreur |
| **Difficulté** | Moyenne |
| **Tests** | Confirmation cancel ; post-clear app state |

### SEC-19 — Télémétrie
| Champ | Contenu |
|-------|---------|
| **Sévérité** | INFORMATIONAL (positif) |
| **Fichier** | Manifest meta-data analytics ; Gradle excludes |
| **Comportement** | Analytics désactivés |
| **Risque** | Faible |
| **Correction** | Maintenir exclusions dans fork Cipher |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Faible |
| **Tests** | Network log : pas de hits analytics Google |

### SEC-20 — Classification logging (stratégie)

| Classe | Règle Cipher |
|--------|--------------|
| SAFE | IDs techniques scrubbés, états UI non personnels |
| POTENTIALLY SENSITIVE | recipientId, threadId, tailles médias, hostnames |
| SENSITIVE | contenu message, e.164, tokens, clés, attachment bytes, auth |

Production Cipher : **zéro SENSITIVE** dans logs persistés / uploadables.

---

## Matrice Enhanced Privacy (cible)

| # | Fonction | Existant Signal ? | Action Cipher |
|---|----------|-------------------|---------------|
| 1 | Verrouillage auto app | Oui (screen lock) | Étendre délais |
| 2 | Biométrie | Oui | Réutiliser |
| 3 | Code local secondaire | Partiel (passphrase) | Étudier sans double crypto |
| 4 | Notifs sans aperçu | Partiel | Ajouter GENERIC/HIDDEN |
| 5 | Block screenshots | Oui | Default ON en EP |
| 6 | Masquage Recents | Via FLAG_SECURE | Brancher EP |
| 7 | Clipboard clean | Partiel (secrets) | Généraliser |
| 8 | Link previews OFF | À vérifier settings | Brancher EP |
| 9 | Read receipts OFF | Oui | Brancher EP |
| 10 | Typing OFF | Oui | Brancher EP |
| 11 | Auto-download OFF | Data & storage | Brancher EP |
| 12 | Auto-delete médias locaux | Non / partiel | Nouveau |
| 13 | Expiration caches | Partiel | Renforcer |
| 14 | Temp files GC | Partiel | Renforcer |
| 15 | Réduction logs | Partiel | Renforcer |
| 16 | Limitation backups | ABS partiel | Option |
| 17 | Protection DB | SQLCipher déjà | Pas de crypto maison |
| 18 | Lock après délai | Oui | UI Cipher |
| 19 | Écran masqué background | Via secure | Brancher EP |
| 20 | Clear Local Data | À créer | Nouveau + confirmations |

---

## Priorisation recommandée

1. SEC-01 / SEC-21 backups ABS + jetons SVR  
2. SEC-22 legacy DB secret plaintext (si présent)  
3. SEC-05 / SEC-23 clipboard messages  
4. SEC-02 / SEC-24 notifications (défaut `all`)  
5. SEC-03 / SEC-04 screen + recents (défaut OFF upstream)  
6. SEC-06 / SEC-07 / SEC-25 logging & crashes  
7. SEC-08 / SEC-09 / SEC-17 / SEC-27 stockage temp  
8. SEC-18 Clear Local Data  

**Ne pas toucher** sans autorisation : libsignal, formats, endpoints, applicationId (phase dédiée).

---

## Complément d’audit approfondi (2026-10-01)

Findings additionnels issus d’une revue statique ciblée (`app` / `core` / `lib`). Aucune modification de code.

### SEC-21 — Liste ABS limitée à `SvrAuthTokens`
| Champ | Contenu |
|-------|---------|
| **Sévérité** | HIGH |
| **Fichier** | `absbackup/backupables/SvrAuthTokens.kt` |
| **Comportement** | Encode `SignalStore.svr.svr2AuthTokens` pour Android Backup ; restore si store vide |
| **Risque** | Jetons de récupération hors coffre messages |
| **Correction** | Retirer de `SignalBackupAgent.items` ou `allowBackup=false` en Enhanced Privacy |
| **Impact Signal** | COMPATIBLE protocole ; RISQUE UX réinstall via ABS |
| **Difficulté** | Faible |
| **Tests** | Backup/restore Google ; réinscription SVR |

### SEC-22 — Secret DB legacy non chiffré (migration)
| Champ | Contenu |
|-------|---------|
| **Sévérité** | CRITICAL si présent / INFORMATIONAL après migration |
| **Fichier** | `keyvalue/PlainTextKeyValueStore.kt`, `DatabaseSecretProvider` |
| **Comportement** | Anciennes installs : `pref_database_unencrypted_secret` en SharedPreferences jusqu’à scellage Keystore |
| **Risque** | Clé SQLCipher en clair (root / forensic / backup prefs) |
| **Correction** | Migration forcée au démarrage Cipher ; refuser legacy non migré |
| **Impact Signal** | COMPATIBLE si migration idempotente |
| **Difficulté** | Moyenne |
| **Tests** | Fixture prefs legacy → upgrade → absence plaintext |

### SEC-23 — Copie de messages sans `copyToClipboardSensitive`
| Champ | Contenu |
|-------|---------|
| **Sévérité** | HIGH |
| **Fichiers** | `conversation/v2/ConversationRepository.kt` ; aussi `MessageHeaderViewHolder`, `ScheduledMessagesBottomSheet`, `EmojiEditText` |
| **Comportement** | Corps de message → `Util.copyToClipboard` (pas de flag sensitive API 33+, pas de clear timer) |
| **Risque** | Contenu message persiste dans le presse-papiers / suggestions clavier |
| **Correction** | Router vers `copyToClipboardSensitive` ; timeout configurable Enhanced Privacy |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Faible–moyenne |
| **Tests** | Étendre `UtilTest_copyToClipboard` ; copie multi-part |

### SEC-24 — Défaut notifications = `"all"`
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | `keyvalue/SettingsValues.kt` → `messageNotificationsPrivacy` |
| **Comportement** | Défaut upstream affiche contact **et** extrait |
| **Correction Cipher** | Défaut `"contact"` ou mode GENERIC ; rester configurable |
| **Impact Signal** | COMPATIBLE (défaut fork différent) |
| **Difficulté** | Faible |

### SEC-25 — Stack traces crash non scrubbées
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | `util/SignalUncaughtExceptionHandler.java` → `LogDatabase.crashes().saveCrash` |
| **Comportement** | Stack complète en clair dans `signal-logs.db` (SQLCipher) ; pas de `Scrubber` (contrairement à `PersistentLogger`) |
| **Correction** | Scrubber avant save ; opt-out stockage crash |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Faible |

### SEC-26 — WebViews captcha / Stripe 3DS
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichiers** | `ratelimit/RecaptchaProofActivity.java` ; `subscription/.../Stripe3DSDialogFragment.kt` |
| **Comportement** | JS activé ; URLs distantes ; pas de `addJavascriptInterface` repéré sur captcha |
| **Risque** | Surface WebView ; 3DS avec DOM storage |
| **Correction** | Durcir settings WebView ; allowlist Stripe ; `FLAG_SECURE` pendant flux |
| **Impact Signal** | RISQUE si captcha/3DS cassés (registration / dons) |
| **Difficulté** | Moyenne |

### SEC-27 — Temp files attachments `.mms`
| Champ | Contenu |
|-------|---------|
| **Sévérité** | MEDIUM |
| **Fichier** | `database/AttachmentTable.kt` (`createTempPartFile`, `PartFileProtector`) |
| **Comportement** | Temps fichiers sous répertoire app ; protection mémoire ~10 min — pas chiffrement au repos pendant traitement |
| **Correction** | Fenêtre de vie courte ; wipe ; pipeline chiffré immédiat si faisable sans casser perf |
| **Impact Signal** | COMPATIBLE / RISQUE perf pipeline |
| **Difficulté** | Élevée |

### SEC-28 — Enquête qualité d’appel : share debug log coché par défaut
| Champ | Contenu |
|-------|---------|
| **Sévérité** | LOW |
| **Fichier** | `calls/quality/CallQualityScreens.kt` (`isShareDebugLogSelected = true`) |
| **Correction** | Défaut décoché en Cipher |
| **Impact Signal** | COMPATIBLE |
| **Difficulté** | Triviale |

### Points positifs confirmés (à conserver)
- Lint `LogNotSignal` / pas de `android.util.Log` direct dans `app/src/main`
- `LogDatabase` SQLCipher
- `copyToClipboardSensitive` + `ClearClipboardAlarmReceiver` pour secrets
- Analytics Firebase désactivés + WebView MetricsOptOut
- Confidentialité notifications déjà implémentée (seul le **défaut** est permissif)

### Limites
- Pas de test runtime (OEM clipboard, backup Google, WebView live)
- Modules `demo/*` non audités en détail
- Couverture `Scrubber` = heuristique regex, pas preuve formelle
