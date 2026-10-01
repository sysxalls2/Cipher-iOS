# CIPHER Desktop — Audit réseau

**Date :** 2026-10-01  

**Objectif phase actuelle :** rester sur l’infra **Signal officielle** pour parler aux utilisateurs Signal.

---

## Connexions (via config)

Définies dans `config/*.json` (noms exacts selon fichier) :

| Usage | Exemples de champs |
|-------|-------------------|
| Chat / API | `serverUrl` |
| Storage | `storageUrl` |
| CDN | `cdn` / CDN list |
| Content proxy | `contentProxyUrl` |
| SFU appels | `sfuUrl` |
| Updates | `updatesUrl` → `updates[2].signal.org/desktop` |
| Trust / ZK | `serverPublicParams`, `serverTrustRoots`, `backupServerPublicParams` |

Transport : HTTPS + WebSocket (libsignal Net / Chat) · `got` pour updates.

---

## Classification

| Action | SIGNAL COMPATIBILITY |
|--------|----------------------|
| Garder URLs / params officiels | **SAFE** |
| Désactiver Giphy / previews / telemetry locale | **SAFE** |
| Changer `serverUrl` vers serveur Cipher | **BREAKING** |
| Remplacer libsignal / protos | **BREAKING** |
| Auto-update Cipher custom | **SAFE** messagerie ; **RISK** si mal signé |

---

## Proxy

Étudier le support proxy existant avant toute addition.  
Optionnel, désactivable, **pas** de Tor forcé. Aucune promesse d’anonymat.

---

## À ne pas faire en phase 1

- Pointer vers `Signal-Server-main` custom  
- Forcer VPN  
- Modifier trust roots  

Idées serveur indépendant → `docs/FUTURE_CIPHER_NETWORK.md`.
