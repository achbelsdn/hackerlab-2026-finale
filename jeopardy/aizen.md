# Aizen — mission complète (web / pwn / rev / forensics)

**Catégorie** : mission multi-questions (web, pwn, rev, forensics) — 4 × 200 pts
**Auteur** : 5c0r7 — **Équipe** : DIVISION_0 — **Statut** : 3/4 résolues par nous, 4/4 équipe

---

## Q1 — Cagliostro (web, 200 pts)

**Flag** : `HLB2026{a24c076c09d485af5c79fff8fd7c3f8c}}` (double `}}` volontaire, vérifié par hash)

**Cible** : Vantage Hosting `172.28.27.2:5000` — portail de coffres-forts numériques.

### Chaîne

1. **Login guest** → session par cookie **JWT HS256**.
2. **Double-URL-encoding traversal** : le filtre supprime `..` et `/` **avant** un second
   `unquote()`. Un chemin encodé deux fois passe le filtre puis se décode au moment de l'ouverture :

   ```
   GET /vault/3/download?f=%252fetc%252fpasswd
   ```

3. **LFI** → lecture de `/app/app.py` et `/app/.env` :
   - `JWT_SECRET=11045f3d...` = `sha256(FLAG + ":jwt")[:48]`
4. **Forge de JWT admin** avec le secret récupéré → accès aux routes `/backups`,
   `/certificates`, `/network`.
5. **SSRF WeasyPrint** : `POST /admin/vault/3/settings` avec une description contenant
   `<img src=http://127.0.0.1:8080/flag.svg>` → le générateur PDF interne charge la ressource.
6. `GET /admin/vault/3/export` → le PDF exporté contient le flag (extraction via pdfminer).

### Leurre

Craquer le JWT (john, HMAC msg#sig) est inutile : **la signature était correcte**, le secret
fuit via la LFI.

---

## Q2 — Ishikawa Goemon (pwn, 200 pts)

**Flag** : `HLB2026{9b3c0adc385cad6dbf940e5d4bbe3bbd}`

**Cible** : HSM Key Controller `vault-keyctl`, `172.28.29.2:9000`.

### Deux bugs combinés

1. **Off-by-one NUL** : un PIN de 16 caractères écrit un NUL à `rsp+0x30`, écrasant le byte
   « seal » de validation.
2. **Overflow 16 bits du Token Market** : commander `65536` tokens fait boucler le compteur
   de coût → achat gratuit.

### Chaîne

```
register → market (65536 tokens) → allocate 1 key pair → Master Key Export (PIN 16 chars)
```

Le FNV-1a du PIN est du code mort : seule la corruption du byte seal compte.

---

## Q3 — Fujiko Mine (rev, 200 pts)

**Flag** : `HLB2026{60fa4abd3f8dd79a1455348e9db86aa9}`

**Cible** : Vault Handout `172.28.44.2:8080` — `vault-tool` (stripped, PIE) avec seulement
`info`/`verify` exposés. Le « code mort » contient les primitives réelles.

### Chaîne de déchiffrement du coffre

1. **KEK = Feistel8(master, hdr[0x20:0x30])** — la clé 16 o = salt 8 o + 8 o suivants.
2. **KWRAP custom** : `t XOR` avant déchiffrement ECB, `t = i + (6-r)(n-1)`, 6 rounds,
   sur **hdr[0x40:0x68]**.
3. Clé contenu → **AES-256-CBC** avec **IV = hdr[0x30:0x40]** (juste avant le blob —
   l'offset « évident » était faux).
4. gzip + PKCS#7 → TAR (plans de câblage Tervalis) → flag.

### Méthode gagnante

- ~60 000 combinaisons « raisonnables » KO avant de scanner le **produit cartésien complet**
  des primitives natives via `ctypes` (PIE neutralisé en effaçant `DT_FLAGS_1`/PIE bit).
- Oracle gratuit : la constante AIV `a6a6` du KWRAP valide les candidats.

### Leçons

- Les noms de suite du header (`FEISTEL8-KWRAP-CBC256`) décrivent **vraiment** la chaîne.
- Quand un binaire fournit des primitives natives validées, ne pas réimplémenter : les
  appeler toutes.

---

## Q4 — Zenigata (forensics, 200 pts)

Image disque du poste de la salle serveur — résolue par un autre membre de l'équipe
(chaîne non re-dérivable depuis nos notes : à compléter).

## Leçons transverses

- Challenges « l'outil refuse sans HSM » = reverse du code mort, pas d'attaque du binaire.
- Instances éphémères : faire tout le offline d'abord (les 3 fichiers suffisent), le flag
  est statique.
