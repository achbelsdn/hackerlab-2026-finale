# Aizen — mission complète (web, pwn, reverse, forensics)

**Catégorie** : mission multi-questions — 4 × 200 points
**Auteur** : 5c0r7 — **Équipe** : DIVISION_0 — **Statut** : trois questions résolues par
nous, la quatrième par un coéquipier.

Cette mission met en scène le vol d'un coffre-fort numérique à travers quatre
disciplines : exploitation web, corruption de mémoire, rétro-ingénierie et forensique.

---

## Q1 — Cagliostro (web, 200 points)

**Flag** : `HLB2026{a24c076c09d485af5c79fff8fd7c3f8c}}` — le double `}}` est volontaire
de la part de l'auteur ; la valeur a été vérifiée par hachage avant toute conclusion.

**Cible** : Vantage Hosting (`172.28.27.2:5000`), un portail de coffres-forts numériques.

### Chaîne de résolution

1. **Connexion en invité** : la session repose sur un cookie **JWT signé en HS256**.
2. **Traversée de répertoire à double encodage URL** : l'application filtre `..` et `/`
   **avant** d'appliquer un second `unquote()`. Un chemin encodé deux fois traverse donc
   le filtre, puis se décode au moment de l'ouverture du fichier :

   ```
   GET /vault/3/download?f=%252fetc%252fpasswd
   ```

3. **Lecture de fichier arbitraire** : on récupère `/app/app.py` puis `/app/.env`, qui
   contient `JWT_SECRET=11045f3d...`, égal à `sha256(FLAG + ":jwt")[:48]`.
4. **Forge d'un JWT d'administrateur** avec ce secret, ce qui débloque les routes
   `/backups`, `/certificates` et `/network`.
5. **SSRF via WeasyPrint** : la requête `POST /admin/vault/3/settings` accepte une
   description HTML ; on y insère `<img src=http://127.0.0.1:8080/flag.svg>` afin que le
   générateur de PDF interne résolve la ressource locale.
6. `GET /admin/vault/3/export` produit le PDF d'export, dont on extrait le flag avec
   pdfminer.

### Leurre à connaître

Craquer le JWT (attaque HMAC sur les paires message/signature) est une impasse : **la
signature était valide**. Le secret fuit par la lecture de fichier, c'est là qu'il
fallait chercher.

---

## Q2 — Ishikawa Goemon (pwn, 200 points)

**Flag** : `HLB2026{9b3c0adc385cad6dbf940e5d4bbe3bbd}`

**Cible** : contrôleur de clés HSM `vault-keyctl` (`172.28.29.2:9000`).

### Deux défauts combinés

1. **Débordement d'un octet nul** : un PIN de 16 caractères écrit un octet nul à
   `rsp+0x30`, qui écrase l'indicateur « seal » de validation.
2. **Débordement d'entier 16 bits dans le marché de tokens** : commander 65 536 tokens
   fait boucler le compteur de coût, ce qui rend l'achat gratuit.

### Chaîne d'exploitation

```
register → market (65 536 tokens) → allocate 1 key pair → Master Key Export (PIN 16 caractères)
```

Le calcul de FNV-1a sur le PIN est du code mort : seule la corruption de l'indicateur
seal conditionne l'export.

---

## Q3 — Fujiko Mine (reverse, 200 points)

**Flag** : `HLB2026{60fa4abd3f8dd79a1455348e9db86aa9}`

**Cible** : Vault Handout (`172.28.44.2:8080`) — le binaire `vault-tool`, dépouillé de
ses symboles et compilé en PIE, n'expose que les commandes `info` et `verify`. Les
primitives réelles du coffre se trouvent dans le **code mort**.

### Chaîne de déchiffrement

1. **KEK = Feistel8(master, hdr[0x20:0x30])** : la clé de 16 octets est constituée du
   sel de 8 octets suivi des 8 octets suivants — un agencement peu intuitif.
2. **KWRAP personnalisé** : un XOR par bloc `t` avant le déchiffrement ECB, avec
   `t = i + (6 − r)(n − 1)` sur 6 rounds, appliqué à **hdr[0x40:0x68]**.
3. Clé du contenu obtenue, puis **AES-256-CBC** avec **IV = hdr[0x30:0x40]** — l'IV se
   trouve juste avant le blob chiffré, et non à l'offset « évident ».
4. Décompression gzip, désassemblage PKCS#7, extraction du TAR (plans de câblage de
   Tervalis) puis du flag.

### Méthode qui a fait la différence

- Environ 60 000 combinaisons « raisonnables » ont échoué avant que l'on scanne le
  **produit cartésien complet** des primitives natives via `ctypes` (le bit PIE de
  `DT_FLAGS_1` étant neutralisé pour charger le binaire comme bibliothèque).
- Un oracle gratuit valide chaque candidat : la constante AIV `a6a6` du KWRAP.

### Leçons

- Les noms de suite inscrits dans l'en-tête (`FEISTEL8-KWRAP-CBC256`) décrivent
  **réellement** la chaîne cryptographique : ne pas inventer une structure plus subtile
  que celle qui est annoncée.
- Lorsqu'un binaire fournit des primitives natives validées, les appeler toutes
  systématiquement plutôt que les réimplémenter.

---

## Q4 — Zenigata (forensics, 200 points)

Image disque du poste de la salle serveur — résolue par un coéquipier ; la chaîne
détaillée sera complétée à partir de ses notes.

---

## Synthèse pédagogique

- Un challenge du type « l'outil refuse de fonctionner sans HSM » se résout par la
  rétro-ingénierie du code mort, non par une attaque du binaire exposé.
- Face à des instances éphémères, traiter d'abord tout ce qui est hors ligne : les
  trois fichiers suffisent, et le flag est statique.
