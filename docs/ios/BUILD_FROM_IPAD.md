# BUILD FROM iPAD — Cipher iOS

**Objectif :** piloter les builds Cipher iOS depuis un iPad (pas de Xcode local sur iPad).

```
iPad → GitHub (workflow_dispatch) → macOS runner → Xcode → artefacts
```

---

## Ce qui marche depuis l’iPad

1. Éditer / commit via GitHub app / Codespaces / Cursor  
2. Déclencher Actions manuellement (`workflow_dispatch`)  
3. Télécharger logs / `.xcarchive` / IPA (si signing OK)  

## Ce qui ne marche pas sur iPad

- Compiler Signal iOS nativement avec Xcode  
- Signer localement sans Mac / secrets CI  

---

## Workflow cible

Fichier à créer (pas encore) :

`.github/workflows/cipher-ios-build.yml`

Étapes prévues :

1. checkout (+ submodules / token si pods privés)  
2. sélection Xcode **26.6**  
3. Ruby + `bundle install`  
4. `make dependencies`  
5. `xcodebuild` build / test  
6. archive logs  
7. `.xcarchive` si possible  
8. IPA **seulement** si secrets de signature présents  

---

## Secrets nécessaires (jamais dans Git)

Voir liste dans `docs/CI_SECRETS.md` (à créer) :

- Apple Team ID / Issuer / Key ID / `.p8`  
- Certificats / profils ou Fastlane Match  
- `ACCESS_TOKEN` si clone pods privés Signal  

---

## Checklist avant premier run

- [ ] Repo GitHub Cipher avec code iOS  
- [ ] Runner macOS disponible  
- [ ] Xcode 26.6 sur le runner  
- [ ] `Pods` résolus  
- [ ] Signing configuré **ou** build unsigned / archive seule  
- [ ] `workflow_dispatch` testé depuis iPad  

**SIGNAL COMPATIBILITY :** CI qui compile sans changer endpoints = **SAFE**.
