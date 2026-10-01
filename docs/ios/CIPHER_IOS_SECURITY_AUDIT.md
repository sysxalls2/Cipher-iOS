# CIPHER iOS — Audit sécurité

**Date :** 2026-10-01  

**SIGNAL COMPATIBILITY (correctifs locaux) :** SAFE sauf mention contraire.

---

## Points positifs

- SQLCipher + clé dérivée / Keychain  
- Screen Lock Face ID / Touch ID déjà présents  
- Extensions isolées (NSE / Share) avec App Groups  
- PrivacyInfo.xcprivacy  
- Contournement censure / proxy optionnels (ne pas forcer Tor)

---

## Findings prioritaires

### I-SEC-01 — Bundle ID / App Groups / Keychain liés à `org.whispersystems`
| | |
|--|--|
| **Sévérité** | HIGH (identité) |
| **Risque** | Collision avec Signal officiel ; Keychain / groups cassés si half-rename |
| **Cipher** | Plan progressif + nouveau prefix `org.cipher…` seulement après docs + CI |
| **Compat** | **RISK** |

### I-SEC-02 — Scheme `sgnl` + universal links Signal
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Risque** | Conflit deep links ; link device |
| **Cipher** | Conserver pour interop ; ajouter schemes Cipher plus tard |
| **Compat** | **RISK** si retiré |

### I-SEC-03 — Push APNs (certificat)
| | |
|--|--|
| **Sévérité** | HIGH (fonctionnel) |
| **Risque** | Push / registration incomplets sans certifs Cipher |
| **Cipher** | Secrets CI Apple ; documenter limite Builds iPad |
| **Compat** | SAFE protocole ; **RISK** livraison messages en background |

### I-SEC-04 — Notifications (contenu)
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Fichiers** | NSE + prefs privacy |
| **Cipher** | Niveaux GENERIC / HIDDEN en Enhanced Privacy |
| **Compat** | SAFE |

### I-SEC-05 — App Switcher / captures
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Cipher** | Blur / obscure en background (API iOS) |
| **Compat** | SAFE |

### I-SEC-06 — Clipboard
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Cipher** | Auto-clear (aligné Android Cipher) |
| **Compat** | SAFE |

### I-SEC-07 — Logs / debug
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Cipher** | Rétention courte ; pas d’upload auto |
| **Compat** | SAFE |

### I-SEC-08 — Caches / temp / médias
| | |
|--|--|
| **Sévérité** | MEDIUM |
| **Cipher** | TTL + purge Enhanced Privacy |
| **Compat** | SAFE |

---

## Enhanced Privacy iOS (cible)

- Face ID / Touch ID lock  
- Timeout auto-lock  
- Protection App Switcher  
- Notifications masquées  
- Clipboard clear  
- Link previews OFF  
- Réduction logs / temp / cache  
- Pas de crypto maison  

Chaque réglage reste individuel.
