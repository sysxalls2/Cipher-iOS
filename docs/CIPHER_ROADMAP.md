# CIPHER — Roadmap

**Projet :** Cipher Messenger  
**Base :** Signal Android open source (fork indépendant, non officiel)  
**Priorité absolue :** compatibilité avec les utilisateurs Signal officiels  
**Plateforme initiale :** `Signal-Android-main` uniquement  

Avant toute modification réseau/protocole, classifier :

- **COMPATIBLE**
- **RISQUE DE COMPATIBILITÉ**
- **INCOMPATIBLE** → autorisation explicite obligatoire

---

## PHASE 0 — Audit ✅ (en cours de finalisation doc)

**Objectif :** comprendre sans modifier le comportement.

**Livrables :**
- [x] `docs/CIPHER_PROJECT_AUDIT.md`
- [x] `docs/CIPHER_SECURITY_AUDIT.md`
- [x] `docs/CIPHER_NETWORK_AUDIT.md`
- [x] `docs/CIPHER_ROADMAP.md`
- [x] `docs/CIPHER_BRANDING_PLAN.md`
- [x] `docs/CIPHER_COMPATIBILITY_MATRIX.md` (structure)
- [x] `docs/FUTURE_CIPHER_NETWORK.md`

**Exit criteria :** architecture, build, privacy, réseau, risques package documentés.

**Commit proposé :** `docs: add Cipher project audit`

---

## PHASE 1 — Compiler Signal Android sans modification

**Objectif :** prouver que le tree upstream build chez nous.

**Actions :**
1. JDK 21 + Android SDK + NDK 28.0.13004108 + CMake
2. Git init local (si archive sans `.git`) — avec accord
3. `gradlew.bat :Signal-Android:assemblePlayProdDebug`
4. Installer sur device/émulateur
5. Smoke : ouverture app (pas forcément compte)

**COMPATIBILITÉ :** COMPATIBLE (aucune modif)

**Commit :** `build: verify upstream Android build`

---

## PHASE 2 — Branding Cipher de développement

**Objectif :** distinguer les builds Cipher **sans** changer `applicationId`.

**Actions :**
- Suffixe label debug « Cipher Dev »
- Icône distincte (mipmap overlay / flavor `cipher` éventuel)
- Ne pas toucher package Java massivement

**COMPATIBILITÉ :** COMPATIBLE

**Commit :** `chore(branding): introduce Cipher development branding`

---

## PHASE 3 — Renommage visuel de l’application

**Objectif :** nom affiché Cipher, splash, strings user-facing prioritaires.

**Actions :**
- `app_name` → Cipher (stratégie translations)
- Splash / about : « indépendant, basé sur Signal open source »
- Conserver NOTICE/LICENSE

**Éviter :** usurpation marque Signal  
**applicationId :** documenter puis décider (défaut : reporter)

**COMPATIBILITÉ :** COMPATIBLE (visuel) ; applicationId = RISQUE

**Commit :** `chore(branding): rename visible app strings to Cipher`

---

## PHASE 4 — Modernisation légère de l’interface

**Objectif :** look premium sombre discret — **reconnaissable Signal-like**, pas clone pixel-perfect ni skin hacker.

**Actions :**
- Couleurs secondaires / typographie prudente
- Espacements, micro-animations
- Pas de refonte navigation conversations

**COMPATIBILITÉ :** COMPATIBLE

**Commit :** `ui: light Cipher visual modernization`

---

## PHASE 5 — Section Privacy & Security

**Objectif :** hub Cipher dans Settings.

**Chemin UI cible :**
```
Settings → Privacy & Security → Enhanced Privacy
```

S’appuyer sur `components/settings/app/privacy/*` existant.

**COMPATIBILITÉ :** COMPATIBLE

**Commit :** `ui(settings): add Cipher privacy and security section`

---

## PHASE 6 — Mode Enhanced Privacy

**Interrupteur global** qui active un preset, **chaque option reste individuelle**.

Preset proposé (ON) :
- Notifications GENERIC/HIDDEN
- Lock rapide
- Screenshots bloqués
- Recents masqués
- Clipboard auto-clear
- Link previews OFF
- Read receipts OFF
- Typing OFF
- Auto-download limité
- Caches courts
- Temp GC agressif

**COMPATIBILITÉ :**
- Local : COMPATIBLE
- Read receipts / typing sync : COMPATIBLE (settings Signal existants)
- Ne pas inventer flags serveur

**Commit :** `feat(privacy): add enhanced privacy settings`

---

## PHASE 7 — Hardening stockage local

**Objectif :** minimum nécessaire sur disque.

**Actions :**
- TTL caches / thumbs
- GC BlobProvider
- Options auto-delete médias locaux
- Revue `allowBackup` / ABS
- **Pas** de crypto maison ; garder SQLCipher

**COMPATIBILITÉ :** COMPATIBLE

**Commit :** `security(storage): reduce temporary data retention`

---

## PHASE 8 — Réduction logs et fichiers temporaires

**Actions :**
- Classification SAFE / POTENTIALLY SENSITIVE / SENSITIVE
- Abstraction logging Cipher en prod
- Désactiver upload debug par défaut en EP
- Purge LogDatabase plus agressive

**COMPATIBILITÉ :** COMPATIBLE

**Commit :** `security(logging): harden production logging`

---

## PHASE 9 — Notifications / captures / multitâche

**Actions :**
- Niveaux FULL / SENDER_ONLY / GENERIC / HIDDEN
- Block screenshots setting
- Hide in Recents
- Clipboard duration setting

**COMPATIBILITÉ :** COMPATIBLE

**Commit :** `feat(privacy): notification and screen protections`

---

## PHASE 10 — Audit réseau avancé

**Actions :**
- Inventory runtime (mitm doc interne, pas pour attaque)
- Confirmer aucune télémétrie surprise
- Documenter proxy UX
- **Ne pas** changer endpoints

**COMPATIBILITÉ :** COMPATIBLE si lecture seule / docs

---

## PHASE 11 — Tests avec Signal officiel

**Matrice :** `docs/CIPHER_COMPATIBILITY_MATRIX.md`

Couvrir : texte, image, vidéo, fichiers, réactions, replies, stickers, groupes, appels, éphémères, receipts, typing, usernames, contacts.

**Exit :** lignes critiques = Compatible + test OK

---

## PHASE 12 — Release hardening

**Actions :**
- ProGuard/R8 release
- Signing Cipher
- Update channel Cipher (pas se faire passer pour updates Signal)
- Mentions légales AGPL + « non affilié à Signal Messenger, LLC »
- Checklist sécurité
- Reproducible build optionnel

**Commit series :** `build:`, `docs:`, `security:`

---

## Hors roadmap immédiate (interdit sans autorisation)

| Idée | Niveau |
|------|--------|
| Serveur Cipher custom | INCOMPATIBLE (messagerie Signal) |
| Remplacer libsignal | INCOMPATIBLE |
| Forcer Tor | RISQUE |
| Renommage massif packages Java | RISQUE (coût / régression) |
| Changer applicationId trop tôt | RISQUE |

---

## Ordre d’exécution strict

```
0 Audit → 1 Build → 2 Branding dev → 3 Nom visible
  → 4 UI légère → 5 Section privacy → 6 Enhanced Privacy
  → 7 Storage → 8 Logs → 9 Screen/notifs → 10 Network audit
  → 11 Tests Signal officiel → 12 Release
```

**Règle :** une phase à la fois — analyser, lister fichiers, risques, compat, modifier le minimum, compiler, tester, résumer.
