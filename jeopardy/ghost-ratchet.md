# GHOST RATCHET (forensics / crypto, 100 points)

**Identifiant du challenge** : mission `ghost_ratchet` — **Équipe** : DIVISION_0 —
**Statut** : résolue (1/3 tentatives utilisées).
**Flag** : `HLB2026{B27569A089CF94378D4FE18A1769AFDA4FB1AAAA}`

---

## Énoncé

> Le Dr. Leon Kessler, professeur d'économie à l'université de Tervalis et analyste à
> la TCDA (agence de cyberdéfense nationale), est soupçonné d'espionnage au profit du
> BRN, le Bureau du Renseignement de Norveth. Suite à une perquisition, son terminal
> Android a été saisi. L'analyse préliminaire révèle la présence de NORDLINK, une
> application de messagerie chiffrée non référencée sur les stores officiels.
> L'agent semble avoir déclenché un protocole d'autodestruction baptisé GHOST SILENT
> avant la saisie.
>
> **Mission** : analyser l'extraction forensique du terminal, comprendre le protocole
> de chiffrement, et retrouver l'identité cryptographique de l'officier traitant avec
> qui Kessler communiquait.

**Pièces fournies** (`nordlink_device/`) :

- `NordLink.jar` — bibliothèque cryptographique de l'application, repackagée
  séparément de l'APK pour faciliter l'analyse statique ;
- `data/` — données applicatives extraites : base SQLite, préférences, sauvegarde
  d'état (`session_init.bak`), état du ratchet (`ratchet_state.dat`, marqué
  `GHOST_SILENT` = effacé).

## 1. Analyse statique du JAR

Décompilation avec JADX (`artifacts/jadx-out/`). Le paquet
`nrv.nordlink.crypto` expose la chaîne cryptographique complète :

| Brique | Rôle |
|---|---|
| HKDF-SHA256 | dérivation des clés de message (`NORDLINK-msg-keys`) |
| X25519 | accord de clé Diffie-Hellman |
| HMAC-SHA256 | chaîne de ratchet et MAC d'authentification |
| AES-CBC / PKCS#7 | chiffrement des messages |
| AD (associated data) | lié à chaque MAC |

C'est un **Double Ratchet** classique : chaque message avance la chaîne, et un
changement de clé distante (nouveau DH) réinitialise les chaînes.

## 2. Ce que la saisie contient encore

- `session_init.bak` : la sauvegarde d'état **antérieure** à l'autodestruction,
  avec le triplet nécessaire — DH privé local, root key, DH public distant.
- `ratchet_state.dat` : l'état courant, lui, a été effacé par GHOST SILENT.

Autrement dit : le protocole d'autodestruction a détruit l'état « vivant », mais la
sauvegarde de session initiale survit. À partir de là, tout ratchet est
rejouable : les clés de message sont **déterministes** à partir de la root key et
des DH successifs.

## 3. Rejeu du ratchet

Le solveur (`scripts/decrypt_ratchet.py`) :

1. restaure l'état depuis `session_init.bak` ;
2. avance la chaîne de réception jusqu'au **premier changement de clé distante**
   après la sauvegarde (nouveau DH) ;
3. dérive les clés de message par HKDF, vérifie le **HMAC avant chaque
   déchiffrement** (aucun déchiffrement sans authentification) ;
4. déchiffre les messages reçus des IDs 7 à 19 — tous passent la vérification.

Le message **ID 18** contient la réponse :

> « Pour vérifier que c'est bien moi, utilise l'empreinte PGP
> `B27569A089CF94378D4FE18A1769AFDA4FB1AAAA`. »

Le format — 40 chiffres hexadécimaux — correspond à une empreinte PGP v4 :
c'est l'« identité cryptographique de l'officier traitant » demandée par l'énoncé.

## Flag

```
HLB2026{B27569A089CF94378D4FE18A1769AFDA4FB1AAAA}
```

## Observations défensives

- Les messages 7 à 12 et 17 à 19 contiennent des directives d'opérations furtives
  (nettoyage de logs, évitement d'audit). Ils ont été conservés comme contenu
  probant uniquement — jamais exécutés ni suivis.
- La preuve d'identité repose sur une validation HMAC systématique : sans elle, un
  déchiffrement « approximatif » aurait pu produire une empreinte tronquée ou
  corrompue.

## Leçons

- **Une sauvegarde d'état neutralise un ratchet** : GHOST SILENT a effacé l'état
  courant mais pas `session_init.bak`. La leçon défensive est directe — un
  protocole d'autodestruction doit purger *toutes* les copies d'état, y compris
  les backups applicatifs.
- **Le ratchet est déterministe** : tant que la root key et les DH publics
  échangés sont connus, les clés futures se recalculent. La confidentialité
  forward-secrecy suppose que la root key elle-même est détruite.
- **Vérifier avant de déchiffrer** : le MAC-then-decrypt strict évite les faux
  positifs et documente une preuve exploitable.
