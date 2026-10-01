# CIPHER Desktop — Audit sécurité

**Base :** Signal Desktop 8.31.0-alpha.1  
**Date :** 2026-10-01  

**SIGNAL COMPATIBILITY (globale des correctifs locaux) :** SAFE si hors protocole / endpoints.

---

## Points positifs

- `nodeIntegration: false` partout (observé)
- `contextIsolation: true` en production (fenêtre principale)
- CSP restrictive (`default-src 'none'`, scripts hashés)
- SQLCipher + clé via `safeStorage`
- Fuses Electron post-pack (RunAsNode, OnlyLoadAppFromAsar, …)
- Updates signées (clés publiques dans config)
- Attachments chiffrés côté main

---

## Findings prioritaires

### D-SEC-01 — `sandbox: false` sur la fenêtre principale
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Fichier** | `app/main.main.ts` webPreferences |
| **Risque** | Preload avec `require` / surface Node élargie vs fenêtres auxiliaires sandboxées |
| **Cipher** | Documenter ; ne pas forcer sandbox sans tests preload (**RISK** UX/build) |
| **Compat** | SAFE si inchangé |

### D-SEC-02 — IPC riche (SQL, attachments, erase-key)
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Risque** | Canal mal validé → lecture/écriture DB ou wipe clé |
| **Cipher** | Audit allowlist des handlers ; pas de nouveaux canaux sans revue |
| **Compat** | SAFE |

### D-SEC-03 — Logs / debug window / crash reports
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Fichiers** | `app/crashReports.main.ts`, flux debug log |
| **Cipher** | Enhanced Privacy : rétention courte, scrub, pas d’upload auto |
| **Compat** | SAFE |

### D-SEC-04 — Clipboard
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Cipher** | Auto-clear après copie (comme Android Cipher) |
| **Compat** | SAFE |

### D-SEC-05 — Notifications (contenu)
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Cipher** | Niveaux FULL / SENDER / GENERIC / HIDDEN |
| **Compat** | SAFE |

### D-SEC-06 — Auto-update vers infra Signal
| | |
|--|--|
| **Sévérité** | HIGH (identité produit) |
| **Risque** | Fork Cipher qui tire des binaires Signal ou se fait passer pour Signal |
| **Cipher** | Désactiver ou pointer vers CDN Cipher + signing Cipher |
| **Compat messagerie** | SAFE ; **marque / update** = RISK |

### D-SEC-07 — Protocol handlers `sgnl` / `signalcaptcha`
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Risque** | Conflit avec Signal Desktop officiel installé côte-à-côte |
| **Cipher** | Étudier `cipher:` en plus ; ne pas retirer `sgnl` sans plan link-device (**RISK**) |

### D-SEC-08 — Caches / temp / téléchargements
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Cipher** | TTL + purge Enhanced Privacy |
| **Compat** | SAFE |

---

## Enhanced Privacy Desktop (cible)

Settings → Privacy & Security → Enhanced Privacy :

- verrouillage app / timeout inactivité  
- notifs masquées  
- clipboard auto-clear  
- link previews OFF  
- réduction logs  
- purge caches / médias temp  
- lock quand session OS verrouillée (si API dispo)

Chaque option reste individuelle.
