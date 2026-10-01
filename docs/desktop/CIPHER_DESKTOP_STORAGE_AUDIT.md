# CIPHER Desktop — Audit stockage

**Date :** 2026-10-01  

---

## Emplacements typiques

| OS | userData (typique) |
|----|-------------------|
| Windows | `%APPDATA%\Signal` (nom change avec productName) |
| macOS | `~/Library/Application Support/Signal` |
| Linux | `~/.config/Signal` |

Cipher devra un **chemin userData distinct** si coexistence avec Signal officiel (`productName` / `appId` différents) — **RISK** de collision données si mal géré.

---

## Composants

| Donnée | Mécanisme |
|--------|-----------|
| Messages / sessions / clés protocol | SQLCipher `sql/db.sqlite` |
| Clé DB | `config.json` → `encryptedKey` via `safeStorage` |
| Attachments | Fichiers chiffrés (main) + protocole `attachment://` |
| Config user / ephemeral | `user_config`, `ephemeral_config` |
| Caches médias / stickers | sous userData / caches app |
| Logs | fichiers locaux + éventuelle soumission manuelle |

---

## Recommandations Cipher

1. Ne pas stocker secrets en clair hors `safeStorage` / SQLCipher  
2. Enhanced Privacy : purge agressive caches + temp  
3. Documenter migration si renommage `appId`  
4. Pas de crypto maison sur la DB  

**SIGNAL COMPATIBILITY :** changements locaux de rétention = **SAFE**.
