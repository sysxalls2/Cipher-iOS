# CIPHER — Plan de Branding

**Nom affiché :** Cipher  
**Nom long :** Cipher Messenger  
**Positionnement :** messagerie dérivée de Signal open source, **indépendante**, **non officielle**  
**Compatibilité :** rester utilisable avec les utilisateurs Signal officiels  

---

## 1. Direction visuelle

| Oui | Non |
|-----|-----|
| Minimaliste, propre, moderne, premium | Vert Matrix, crânes, « hacker cliché » |
| Légèrement sombre | Violet/indigo générique AI-slop |
| Orienté confidentialité discret | Éléments agressifs |
| Immédiatement « type Signal » (nav, chats) | Copie exacte trompeuse de la marque Signal |

### Conserver
- Organisation des conversations
- Navigation principale
- Logique settings / appels / groupes
- Disposition générale des chats

### Modifier progressivement
- Nom, icône, splash
- Couleurs secondaires
- Typographie (si pertinent)
- Icônes settings Privacy
- Page Privacy & Security
- Micro-animations, espacements

---

## 2. État actuel (vérifié)

| Élément | Valeur upstream |
|---------|-----------------|
| `app_name` | `Signal` (`app/src/main/res/values/strings.xml`) |
| Icône | `@mipmap/ic_launcher` (+ alternatives disguise) |
| Package / applicationId | `org.thoughtcrime.securesms` |
| Schemes | `sgnl://`, `https://signal.me`, `tsdevice` |
| Licence | AGPLv3 + NOTICE |

---

## 3. Branding technique — étude avant changement

### 3.1 Ce qu’il ne faut PAS faire au premier passage
- Renommage massif `org.thoughtcrime.securesms` → autre package Java/Kotlin
- Changer `applicationId` sans plan de test FCM / deeplinks / providers

### 3.2 Package cible envisagé
```
org.cipher.messenger
```

### 3.3 Conséquences précises d’un changement d’applicationId

| Domaine | Impact |
|---------|--------|
| Installation côte-à-côte avec Signal | Possible (bénéfique pour tests) |
| Données locales | **Pas** de migration auto depuis `org.thoughtcrime.securesms` |
| ContentProviders | Authorities `${applicationId}.*` changent |
| Account sync / authenticator | `res/xml/authenticator.xml` accountType |
| Deep links | Conflit possible sur `sgnl` / `signal.me` (une seule app gagne) |
| FCM | Nouveau projet Firebase / clés requis pour push fiable |
| Intents internes | Beaucoup d’actions hardcodées `org.thoughtcrime.securesms.*` |
| Permission `ACCESS_SECRETS` | Noms liés au package |
| Shortcuts / contact mime types | `vnd.org.thoughtcrime.securesms.*` |
| Backup Android | Identité app différente |
| Play Store / signing | Nouvelle identité d’app |

**COMPATIBILITÉ RÉSEAU (protocole) :** COMPATIBLE si endpoints/crypto inchangés  
**COMPATIBILITÉ DEVICE / UX :** **RISQUE DE COMPATIBILITÉ**  
**Décision Phase 2–3 :** garder `org.thoughtcrime.securesms` en builds internes **ou** introduire flavor `cipher` avec nouvel ID **après** Phase 1 verte — **autorisation requise**.

### 3.4 Namespace Java
Reporter le move massif. Coût énorme, risque de régression, peu de gain privacy.

### 3.5 Firebase
Pas de `google-services.json` dans le tree audité. Push dépend de config Signal/Play. Fork public devra gérer FCM Cipher séparément.

### 3.6 Deep links
Conserver handlers Signal pour compat **tant que** l’objectif est de parler au réseau Signal.  
Branding Cipher : ne pas prétendre être l’app officielle sur `signal.me`.

---

## 4. Plan d’exécution branding

### Étape B0 — Prérequis docs
Ce document + audits.

### Étape B1 — Dev branding (sans applicationId)
- Flavor ou source set `cipher` / `debug` :
  - label `Cipher Dev`
  - icône teintée / badge « C »
- Strings about : indépendant + AGPL

**Compat :** COMPATIBLE

### Étape B2 — Nom visible release-candidate
- `app_name` = Cipher
- Revue strings marketing (pas 100 % i18n d’un coup : stratégie `values` + notes translators)

### Étape B3 — Splash & thème
- Splash Cipher
- Palette : fond sombre doux, accent discret (éviter vert Signal exact si différenciation, sans tomber dans cliché)

### Étape B4 — Privacy UI branding
- Section « Privacy & Security » Cipher
- Enhanced Privacy toggle

### Étape B5 — applicationId (optionnel, plus tard)
Uniquement après :
1. Build OK
2. Plan FCM
3. Plan deeplink coexistence
4. Autorisation écrite

---

## 5. Mentions légales (obligatoires)

Dans About / README Cipher :

> Cipher Messenger is an independent project based on the open-source Signal Messenger client.  
> It is **not** affiliated with, endorsed by, or related to Signal Messenger, LLC.  
> Signal is a trademark of Signal Messenger, LLC.

Conserver :
- `LICENSE` (AGPLv3)
- `NOTICE`
- Copyright headers upstream où requis

---

## 6. Assets à produire (plus tard)

| Asset | Priorité |
|-------|----------|
| Icône launcher adaptive | Haute |
| Splash | Haute |
| Feature graphic store (si distribution) | Basse |
| Monochrome / themed icon Android 13+ | Moyenne |

Pas de crânes, matrices, glitch excessif.

---

## 7. Commits proposés

```
chore(branding): introduce Cipher development branding
chore(branding): rename visible app strings to Cipher
ui: light Cipher visual modernization
docs: add Cipher branding plan
```
