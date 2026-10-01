# CIPHER iOS — Audit stockage

**Date :** 2026-10-01  

---

## Composants

| Donnée | Mécanisme |
|--------|-----------|
| Messages / sessions / PreKeys | GRDB + SQLCipher (`signal.sqlite`) |
| Clé DB | `GRDBKeyFetcher` / Keychain |
| Médias / attachments | fichiers sous App Group / directories SSK |
| Préférences | UserDefaults + stores SSK |
| Logs | chemins partagés NSE / Share |
| Legacy | traces YapDB (migration) |

---

## App Groups

`group.$(SIGNAL_BUNDLEID_PREFIX).signal.group` partagé entre app, NSE, Share.

Changer le prefix sans mettre à jour **tous** les entitlements = corruption / données invisibles (**RISK**).

---

## Recommandations Cipher

1. Purge caches / temp en Enhanced Privacy  
2. Pas de secrets hors Keychain / SQLCipher  
3. Documenter Data Protection capabilities (activées prudemment en fork)  
4. Clear Local Data = confirmation forte, local only  

**SIGNAL COMPATIBILITY :** rétention locale = **SAFE**.
