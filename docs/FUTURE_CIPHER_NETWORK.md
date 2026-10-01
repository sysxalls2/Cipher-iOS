# FUTURE — Réseau Cipher indépendant

**Statut :** idées **futures uniquement**  
**Phase actuelle :** le client Android Cipher doit utiliser l’infrastructure **Signal officielle** pour rester compatible avec les utilisateurs Signal.

> Lancer un serveur Cipher et y pointer le client **casserait** l’objectif principal de compatibilité.

---

## Pourquoi ce document existe

Le workspace contient :
- `Signal-Server-main` (référence architecture)
- `foundationdb-7.4.8` (dépendance stockage serveur)

Ces dépôts servent à **comprendre** Signal-Server — **pas** à déployer un backend Cipher maintenant.

---

## Ce qu’impliquerait un réseau Cipher séparé

| Composant | Travail estimé | Impact compat Signal officiel |
|-----------|----------------|-------------------------------|
| Signal-Server custom | Très élevé | **INCOMPATIBLE** avec comptes Signal |
| FoundationDB ops | Élevé | N/A |
| CDN / stockage pièces jointes | Élevé | INCOMPATIBLE |
| SFU WebRTC | Élevé | INCOMPATIBLE appels cross-network |
| CDSI / SVR / keys | Très élevé | INCOMPATIBLE |
| Fédération avec Signal | Non supportée upstream | Non réaliste sans accord Signal |

**Conclusion :** un réseau Cipher = messagerie **fermée sur elle-même**, pas un client pour parler à Signal.

---

## Idées reportées (sans engagement)

1. Mode « Cipher Network » optionnel **en plus** du mode Signal (double stack) — complexité extrême  
2. Serveurs de secours / mirror — **INCOMPATIBLE** / dangereux sans être Signal  
3. Distribution APK Cipher via canal update propre (sans se faire passer pour `updates.signal.org`) — possible côté **client distribution**, pas serveur messages  

---

## Règle de gouvernance

Toute PR / changement qui :
- modifie `SIGNAL_URL` / CDN / SFU / CDSI / SVR vers un host non-Signal  
- ou introduit un build flavor « cipher-server »  

doit être marqué **INCOMPATIBLE** et **bloqué** jusqu’à autorisation explicite du propriétaire du projet, avec acceptation écrite de perdre l’interop Signal officiel.

---

## Lien avec la roadmap

Phases 0–12 actuelles = client Android + privacy locale + branding, **sur réseau Signal**.  
Ce fichier = parking lot post-interop éventuel.
