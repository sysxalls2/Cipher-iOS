# CIPHER — Matrice de compatibilité avec Signal officiel

**Objectif :** vérifier que Cipher peut discuter avec l’application Signal officielle.  
**Règle :** ne marquer « Compatible » que si test réel effectué.  
**Client de référence Cipher :** Android (fork)  
**Contrepartie :** Signal Android / iOS / Desktop officiels  

Légende **Compatible** : Oui / Non / Partiel / Non testé  

---

| Fonction | Signal officiel | Cipher | Compatible | Test effectué | Résultat | Remarques |
|----------|-----------------|--------|------------|---------------|----------|-----------|
| Texte 1:1 | Oui | Prévu identique protocole | Non testé | Non | — | Priorité P0 |
| Images | Oui | Idem CDN/crypto | Non testé | Non | — | |
| Vidéos | Oui | Idem | Non testé | Non | — | |
| Fichiers | Oui | Idem | Non testé | Non | — | |
| Réactions | Oui | Idem | Non testé | Non | — | |
| Réponses / quotes | Oui | Idem | Non testé | Non | — | |
| Stickers | Oui | Idem | Non testé | Non | — | |
| Groupes v2 | Oui | Idem | Non testé | Non | — | |
| Appels audio | Oui | SFU/WebRTC upstream | Non testé | Non | — | |
| Appels vidéo | Oui | Idem | Non testé | Non | — | |
| Messages éphémères | Oui | Idem | Non testé | Non | — | |
| Accusés de lecture | Oui | Setting existant | Non testé | Non | — | EP peut les OFF |
| Typing indicator | Oui | Setting existant | Non testé | Non | — | EP peut les OFF |
| Pièces jointes voix | Oui | Idem | Non testé | Non | — | |
| Usernames / liens | Oui | Deep links `signal.me` | Non testé | Non | — | RISQUE si schemes changés |
| Contacts / CDSI | Oui | Idem | Non testé | Non | — | |
| Sealed sender | Oui | Idem | Non testé | Non | — | |
| Stories | Oui | Idem | Non testé | Non | — | Hors cœur P0 |
| Link device Desktop | Oui | `sgnl` / `tsdevice` | Non testé | Non | — | |
| Registration | Oui | Serveurs officiels | Non testé | Non | — | Ne pas changer captcha/SVR |

---

## Protocole de test (Phase 11)

1. Device A : Signal officiel (Play Store)  
2. Device B : Cipher build  
3. Même réseau ; comptes distincts  
4. Pour chaque ligne : envoi A→B et B→A  
5. Capturer version Cipher, version Signal, date, opérateur réseau  
6. Toute régression protocole → **stop** et rollback  

---

## Modifications interdites pendant les tests de compat

- URLs `SIGNAL_*` BuildConfig  
- libsignal version forks custom non validés  
- Formats protobuf / wire  
- Serveur custom  

Voir `docs/FUTURE_CIPHER_NETWORK.md` pour un réseau Cipher **séparé** (hors objectif actuel).
