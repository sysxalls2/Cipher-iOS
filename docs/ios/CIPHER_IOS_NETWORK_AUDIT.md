# CIPHER iOS — Audit réseau

**Date :** 2026-10-01  

**Phase actuelle :** endpoints **Signal officiels** pour parler aux utilisateurs Signal.

---

## Couche

`SignalServiceKit/Network/` + `LibSignalClient.Net`  
Constantes : `TSConstants.swift` (prod / staging via schéma Signal-Staging / `USE_STAGING=1`).

Types de liens : HTTPS REST · WebSocket chat · CDN · storage · CDSI · SVR2 · SFU · content proxy · captcha.

---

## Classification

| Action | SIGNAL COMPATIBILITY |
|--------|----------------------|
| Garder URLs / params ZK / trust | **SAFE** |
| Désactiver previews / Giphy / logs upload | **SAFE** |
| Proxy utilisateur optionnel (existant) | **SAFE** si bien configuré |
| Serveur Cipher custom | **BREAKING** |
| Remplacer LibSignalClient | **BREAKING** |
| Retirer `sgnl` / domain association | **RISK** |

---

## Push

APNs → NSE → fetch Signal.  
Fork Cipher : propres certificats push ; sans cela, réception en background dégradée (**RISK** UX, pas crypto).

---

## Anonymat

Cipher iOS **n’est pas** anonyme (IP, DNS, APNs, métadonnées connexion).  
Sealed Sender et réglages privacy existants restent le modèle Signal.
