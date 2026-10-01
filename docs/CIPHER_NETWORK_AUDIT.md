# CIPHER — Audit Réseau (Signal-Android)

**Projet :** Cipher Messenger (fork indépendant)  
**Base :** `Signal-Android-main` 8.30.0  
**Date :** 2026-10-01  
**Objectif compatibilité :** rester sur l’infra **Signal officielle** pour parler aux utilisateurs Signal.

> Cipher **ne doit pas** pointer vers un serveur custom dans les phases actuelles.  
> Toute idée de réseau indépendant → `docs/FUTURE_CIPHER_NETWORK.md`.

**Sources vérifiées :** `app/build.gradle.kts`, `SignalServiceNetworkAccess.kt`, Manifest, modules `lib/network`, `lib/libsignal-service`, proxy settings.

---

## Principes

1. Les connexions ci-dessous sont celles du **client Signal upstream**.
2. Les désactiver peut casser messagerie, médias, appels, registration.
3. Aucune promesse d’anonymat absolu (IP/DNS restent visibles du réseau).
4. Proxy = **optionnel**, déjà supporté — ne pas forcer Tor/VPN.

---

## Tableau des connexions

### N-01 — Service messagerie principal
| Champ | Valeur |
|-------|--------|
| **Domaine** | `chat.signal.org` |
| **Type** | HTTPS + WebSocket (couche service / libsignal net) |
| **Pourquoi** | Envoi/réception messages, sync compte, keys |
| **Données exposées** | IP, métadonnées connexion, auth account ; contenu E2E chiffré |
| **Indispensable** | **Oui** |
| **Désactivable** | Non (casse tout) |
| **Compatibilité** | Modifier = **INCOMPATIBLE** |

### N-02 — Storage Service
| Champ | Valeur |
|-------|--------|
| **Domaine** | `storage.signal.org` |
| **Type** | HTTPS |
| **Pourquoi** | Sync contacts/settings storage manifest |
| **Données** | IP + blobs chiffrés storage |
| **Indispensable** | Oui pour multi-device / sync moderne |
| **Désactivable** | Non sans régression majeure |
| **Compat** | **INCOMPATIBLE** si retiré/changé |

### N-03 — CDN pièces jointes
| Champ | Valeur |
|-------|--------|
| **Domaines** | `cdn.signal.org`, `cdn2.signal.org`, `cdn3.signal.org` |
| **Type** | HTTPS |
| **Pourquoi** | Upload/download attachments, stickers, profiles media |
| **Données** | IP, tailles, timing ; contenu chiffré client-side |
| **Indispensable** | Oui pour médias |
| **Désactivable** | Non (texte seul dégradé) |
| **Compat** | **INCOMPATIBLE** si URLs changées |

### N-04 — CDSI (Contact Discovery)
| Champ | Valeur |
|-------|--------|
| **Domaine** | `cdsi.signal.org` |
| **Type** | HTTPS / protocole CDSI |
| **Pourquoi** | Découverte contacts privée |
| **Données** | IP + protocole CDSI (conçu pour limiter fuites) |
| **Indispensable** | Oui pour trouver qui est sur Signal |
| **Désactivable** | Non sans casser UX contacts |
| **Compat** | **INCOMPATIBLE** si remplacé |

### N-05 — SVR2 (Secure Value Recovery)
| Champ | Valeur |
|-------|--------|
| **Domaine** | `svr2.signal.org` |
| **Type** | HTTPS / enclave (mrenclave BuildConfig) |
| **Pourquoi** | PIN / restauration sécurisée |
| **Données** | IP + protocole SVR |
| **Indispensable** | Fortement (registration/restore) |
| **Désactivable** | Risqué |
| **Compat** | **INCOMPATIBLE** si endpoint/mrenclave faux |

### N-06 — SFU appels groupe / WebRTC
| Champ | Valeur |
|-------|--------|
| **Domaine** | `sfu.voip.signal.org` (+ staging/test variants) |
| **Type** | HTTPS + WebRTC media |
| **Pourquoi** | Appels groupe / selective forwarding |
| **Données** | IP, timing appel, media chiffré selon protocole |
| **Indispensable** | Pour appels groupe |
| **Désactivable** | Non sans casser appels |
| **Compat** | **INCOMPATIBLE** si changé |
| **Note** | Advanced Privacy « always relay calls » augmente relais (TURN) — **COMPATIBLE**, impact batterie/qualité |

### N-07 — Content Proxy (link previews)
| Champ | Valeur |
|-------|--------|
| **Domaine** | `contentproxy.signal.org:443` |
| **Type** | HTTPS proxy |
| **Pourquoi** | Préchargement aperçus de liens sans exposer l’IP directement au site cible (modèle Signal) |
| **Données** | URL demandée via proxy Signal ; IP vers Signal |
| **Indispensable** | Non (feature preview) |
| **Désactivable** | **Oui** — candidate Enhanced Privacy |
| **Compat** | **COMPATIBLE** (feature off) |

### N-08 — Service status
| Champ | Valeur |
|-------|--------|
| **Domaine** | `uptime.signal.org` |
| **Type** | HTTPS/DNS check |
| **Pourquoi** | Statut service |
| **Indispensable** | Non |
| **Désactivable** | Oui avec dégradation diagnostics |
| **Compat** | COMPATIBLE |

### N-09 — Mises à jour APK (website/github flavors)
| Champ | Valeur |
|-------|--------|
| **Domaine** | `updates.signal.org` (`latest.json`), `updates2.signal.org` (badges static) |
| **Type** | HTTPS |
| **Pourquoi** | Update manifest / assets badges |
| **Indispensable** | Pour builds non-Play |
| **Désactivable / remplacer** | Cipher devra **son propre** canal update plus tard — ne pas mentir avec updates Signal |
| **Compat messagerie** | COMPATIBLE si messagerie inchangée ; branding **RISQUE** |

### N-10 — Captcha
| Champ | Valeur |
|-------|--------|
| **Domaines** | `signalcaptchas.org` (registration + challenge) |
| **Type** | HTTPS / WebView |
| **Pourquoi** | Anti-abus registration |
| **Indispensable** | Souvent oui à l’inscription |
| **Désactivable** | Non sans casser register |
| **Compat** | **INCOMPATIBLE** si retiré arbitrairement |

### N-11 — Firebase Cloud Messaging
| Champ | Valeur |
|-------|--------|
| **Domaine** | Infrastructure Google FCM |
| **Type** | HTTPS long-lived / Play Services |
| **Pourquoi** | Wake-up push (pas le contenu message) |
| **Données** | Token FCM, IP Google, métadonnées push |
| **Indispensable** | Pratiquement oui sur Android stock |
| **Désactivable** | **RISQUE DE COMPATIBILITÉ** (retard messages) |
| **Note** | Analytics exclus ; auto-init FCM false |

### N-12 — Giphy
| Champ | Valeur |
|-------|--------|
| **API** | Clé dans BuildConfig `GIPHY_API_KEY` |
| **Pourquoi** | GIF search |
| **Données** | IP + requêtes recherche |
| **Indispensable** | Non |
| **Désactivable** | Oui — EP |
| **Compat** | COMPATIBLE |

### N-13 — Google Maps
| Champ | Valeur |
|-------|--------|
| **Clé** | `mapsKey` Manifest placeholder |
| **Pourquoi** | Partage localisation |
| **Données** | IP + tiles/queries Google |
| **Indispensable** | Non (feature location) |
| **Désactivable** | Oui |
| **Compat** | COMPATIBLE |

### N-14 — Stripe
| Champ | Valeur |
|-------|--------|
| **Domaine** | `api.stripe.com` |
| **Pourquoi** | Donations / badges |
| **Indispensable** | Non pour messagerie |
| **Désactivable** | Oui |
| **Compat messagerie** | COMPATIBLE |

### N-15 — Debug logs upload
| Champ | Valeur |
|-------|--------|
| **Domaine** | `debuglogs.org` |
| **Type** | HTTPS |
| **Pourquoi** | Support / debug volontaire |
| **Données** | Logs scrubbés (idéalement) + métadonnées device |
| **Indispensable** | Non |
| **Désactivable** | Oui — recommandé Enhanced Privacy |
| **Compat** | COMPATIBLE |

### N-16 — DNS
| Champ | Valeur |
|-------|--------|
| **Mécanisme** | `SequentialDns` : System → Cloudflare `1.1.1.1` → `StaticDns` IPs BuildConfig |
| **Pourquoi** | Résilience + censorship circumvention |
| **Données** | Requêtes DNS (fuites métadonnées DNS) |
| **Indispensable** | Partie intégrante client |
| **Modifier** | **RISQUE** (connexion, appels) |
| **Anonymat** | Ne rend **pas** anonyme |

### N-17 — Censorship circumvention / Fastly / reflectors
| Champ | Valeur |
|-------|--------|
| **Hosts** | ex. `chat-signal.global.ssl.fastly.net`, `reflector-nrgwuv7kwq-uc.a.run.app`, variants CDN Fastly (voir `SignalServiceNetworkAccess.kt`) |
| **Pourquoi** | Contournement censure selon pays |
| **Réglage UI** | Advanced Privacy → Censorship circumvention |
| **Indispensable** | Selon réseau utilisateur |
| **Compat** | COMPATIBLE (feature Signal existante) |

### N-18 — Staging counterparts
Miroirs `*.staging.signal.org` pour flavor `staging` — **ne pas utiliser en prod Cipher grand public**.

---

## Proxy existant (étude obligatoire avant modification)

| Élément | Emplacement |
|---------|-------------|
| Stockage config | `keyvalue/ProxyValues.java` |
| UI | `preferences/EditProxyActivity.kt`, `EditProxyFragment.java` |
| Intégration OkHttp | `TlsProxySocketFactory`, `NetworkDependenciesModule` |
| libsignal net | `LibSignalNetworkExtensions.configureProxy` / `ProxyConfig` |
| Entrée settings | `AppSettingsActivity.proxy()` |

**Comportement :** proxy Signal optionnel (host/port), activable/désactivable, déjà branché sur la stack réseau.

### Recommandations Cipher
- **Ne pas** forcer Tor
- **Ne pas** forcer VPN
- Améliorer UX/docs proxy si besoin
- Toute option proxy supplémentaire : optionnelle, désactivable, sans promesse d’anonymat

### Impacts si proxy mal configuré
| Domaine | Impact |
|---------|--------|
| Compatibilité Signal | **RISQUE** (si proxy casse TLS/WebSocket) |
| Batterie | Possible hausse |
| Performance | Latence ↑ |
| Appels | Souvent dégradés / cassés |
| Pièces jointes | Uploads lents / timeouts |

---

## Métadonnées & anonymat (position Cipher)

Cipher **n’est pas** un réseau anonyme.

| Vecteur | Réalité |
|---------|---------|
| IP | Visible du serveur Signal / FCM / CDN |
| DNS | Visible du résolveur (system/1.1.1.1) |
| Sealed sender | Réduit métadonnées expéditeur **côté destinataire/serveur** dans le modèle Signal — déjà présent |
| Dispositif local | Beaucoup de traces (DB, médias, logs) — focus Cipher |

Pour chaque amélioration réseau future, documenter : avantage, inconvénient, compat, batterie, perf, appels, PJ.

---

## Ce qui est désactivable sans casser la messagerie de base

Priorité Enhanced Privacy / options :

1. Link previews / content proxy usage  
2. Giphy  
3. Maps / location attach  
4. Stripe / donations UI  
5. Debug log upload / ShakeToReport  
6. Auto-download médias (settings data)  
7. Read receipts / typing (déjà local+sync settings)  

**Ne pas désactiver :** chat, storage, CDN, CDSI, SVR, SFU (si appels requis), FCM (sans alternative).

---

## Classification compatibilité (rappel)

| Action | Niveau |
|--------|--------|
| Garder endpoints officiels | **COMPATIBLE** |
| Désactiver previews/Giphy/logs upload | **COMPATIBLE** |
| Activer proxy optionnel existant | **COMPATIBLE** (si serveur proxy OK) |
| Changer URLs serveur | **INCOMPATIBLE** |
| Serveur Cipher custom | **INCOMPATIBLE** (phase actuelle) |
| Remplacer FCM sans équivalent | **RISQUE DE COMPATIBILITÉ** |

---

## Complément d’audit approfondi (2026-10-01)

### Contournement de censure (détail)

Quand **activé** (pays / réglage Advanced Privacy) et **proxy Signal désactivé** :

| Mode | Domaines visibles (SNI) | Backend |
|------|-------------------------|---------|
| Google fronting | `www.google.com` (+ TLD locaux), `android.clients.google.com`, etc. | Host `reflector-nrgwuv7kwq-uc.a.run.app` → service/cdn/storage/cdsi/svr2 |
| Fastly fronting | `github.githubassets.com`, `pinterest.com`, `www.redditstatic.com` | `*-signal.global.ssl.fastly.net` / `*.prod.fastly.net` |

**COMPATIBILITÉ :** COMPATIBLE avec infra Signal.  
**Désactivable :** oui (setting) — **RISQUE** sur réseaux censurés.

### Proxy Signal (détail technique)

| Élément | Détail |
|---------|--------|
| Deep links | `https://signal.tube/#<hôte>` / `sgnl://` |
| Protocole | TLS proxy libsignal (`ProxyScheme.TLS`) — pas HTTP CONNECT classique |
| Effet | Force config **non censurée** + injecte proxy ; fronting Google/Fastly **non utilisé** |
| Proxy système Android | Utilisé seulement si config non censurée **et** sans proxy Signal ; `http` / `socks5` ; PAC loopback ignoré (`ProxyConfig.kt`) |

Fichiers : `core/network/.../ProxyConfig.kt`, `EditProxy*`, `ProxyValues`.

### Giphy via content proxy

Requêtes `api.giphy.com` / `*.giphy.com` passent par `contentproxy.signal.org` (`ContentProxySelector`) — l’IP utilisateur n’atteint pas Giphy directement, mais Signal voit la requête.

### MobileCoin (paiements P2P)

| Hôte | Rôle |
|------|------|
| `mc://node*.consensus.mob.production.namda.net` | Consensus |
| `fog://fog.prod.mobilecoinww.com` | Fog |
| `fog://fog-rpt-prd.namda.net` | Fog report |
| + API `chat.signal.org` | Autorisation paiements |

**Indispensable :** non · **Désactivable :** oui · **COMPATIBLE** messagerie si désactivé.

### Liens / hôtes sociaux (hors API core)

`signal.me`, `signal.group`, `signal.link`, `support.signal.org`, `signal.org` — deep links / aide.  
Changer les schemes sans plan = **RISQUE DE COMPATIBILITÉ**.

### Matrice désactivation vs fonctions

| Connexion | 1:1 | Groupes | Médias | Appels | Inscription |
|-----------|-----|---------|--------|--------|-------------|
| chat + storage + CDN | Oui | Oui | Oui | Partiel | Oui |
| CDSI | Optionnel | — | — | — | — |
| SVR2 | — | — | — | — | Restore PIN |
| SFU | — | — | — | Groupe | — |
| FCM | Dégradé | Dégradé | — | — | Push codes |
| Giphy / Maps / Stripe / MobileCoin / debuglogs | — | — | — | — | — |

### Fichiers de référence additionnels

- `app/.../push/SignalServiceNetworkAccess.kt`
- `core/network/.../ProxyConfig.kt`
- `app/.../net/ContentProxySelector.java`
- `app/.../logsubmit/SubmitDebugLogRepository.java` (`https://debuglogs.org`)
