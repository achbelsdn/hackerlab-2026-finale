# The Man Who Sold The World — chaîne OSINT (5 questions)

**Catégorie** : OSINT / Misc / Web — **Équipe** : DIVISION_0 — **Statut** : 4/5 résolues

Narratif : Norveth a poussé Tervalis à des opérations clandestines ; un agent de liaison
(**Thomas Crown**) en a gardé la preuve. Une vidéo « parfaitement anodine » est le point
d'entrée : `gH3WC-6ZqUw` (YouTube).

---

## P1 — Silent Noise (100 pts)

**Flag** : `HLB2026{5ubtl3_5ubt1tl3_h1dd3n_1n_pl41ns1ght}`

### Résolution

1. Indice gratuit de la plateforme : **« Sous-Titres »** → fichier additionnel `meme.mkv`.
2. `ffprobe` révèle **4 pistes de sous-titres SRT** :
   - pistes 0–2 : dialogues leurres (EN / FR / ES) ;
   - **piste 3 (tag `eus` — Basque)** : un bloc **Base64**.
3. Extraction et décodage :

   ```bash
   ffmpeg -i meme.mkv -map 0:s:3 sub3.srt
   ```

   Le Base64 décode en message **PGP clearsigned** signé par « yoda » contenant en clair :

   ```
   Code de validation : HLB2026{5ubtl3_5ubt1tl3_h1dd3n_1n_pl41ns1ght}
   ```

**Leçon** : toujours extraire TOUS les flux annexes (`ffprobe -show_streams`) d'un
conteneur multimédia CTF avant la stégano pixels/audio.

---

## P2 — The Real Slim Shady (100 pts)

**Flag** : `HLB2026{3acd6d10080adbcfe91c62d4ac5649ac}`

### Résolution

1. La signature PGP de P1 a pour keyid **`0x6071195312D405CB`** → serveur HKP
   (Hockeypuck 2.2.2) sur l'instance : port `11371`.
2. La clé correspondante : UID primaire `yoda <anakin@proton.me>`, UID révoqué
   `Thomas Crown <thomas.crown@qmail.nrv>`.
3. **Recherche HKP par token** : le serveur matche sur le nom + local-part email.
   Un brute-force de tokens (script `hkp_bruteforce.sh`, token `hlb2026`) révèle une
   **seconde clé** dont l'UID est :

   ```
   Preuve d'acces (HLB2026{3acd6d10080adbcfe91c62d4ac5649ac}) <proof@qmail.nrv>
   ```

**Leçon** : la recherche HKP matche par token exact (nom et local-part), pas par domaine.

---

## P3 — Back Door Man (100 pts)

**Flag** : `HLB2026{5ffa72701330b0bc7001b23f9d64b4e9}`

### Résolution

1. Webmail qmail (Flask/gunicorn) sur l'instance, port 8080.
2. `POST /api/password/reset-request` **fuit le `reset_token` dans la réponse JSON**.
3. `POST /api/password/reset` `{token, new_password}` → login `thomas.crown@qmail.nrv`.
4. Le brouillon « notes » contient :

   ```
   jeton de session interne: HLB2026{5ffa72701330b0bc7001b23f9d64b4e9}
   ```

**Leçon** : une API de reset qui renvoie le token côté client = takeover complet.

---

## P4 — Pattern of Life (200 pts)

**Flag accepté par la plateforme** : `HLB2026{ODENPLAN}`

### Résolution

1. Les mails du webmail tracent la vie quotidienne de Thomas Crown : reçus de
   stationnement résidentiel (header `X-Originating-Zone: SE-AB-Vasastan`), commerces
   fréquentés autour d'**Odenplan** (Stockholm, Vasastan).
2. Les candidats larges (`VASASTAN`, `STOCKHOLM`, `SE-AB-Vasastan`) sont refusés ; la
   réponse attendue est le **quartier/point de vie précis** : `HLB2026{ODENPLAN}`.

### Note sur la pièce jointe

`entree_site.jpg` (440×509) est un crop de la photo officielle Shurgard
« Stockholm Södermalm » (template matching MSE ≈ 6 contre 5100 pour la 2e candidate →
Ringvägen 77, 118 61 Stockholm). Cette piste a alimenté P5 mais a été refusée pour P4.

---

## P5 — Safe Home (200 pts) — NON RÉSOLU

> « La preuve dort dans un coffre, quelque part dans la ville. Mais où ? »

**Format officiel** (tag API) : `flag:HLB2026{prestataire_quartier_rue}`.

Épuisement documenté (~50 tentatives, 0 solve platform-wide) :

- **Prouvé** : `entree_site.jpg` = crop de la photo officielle Shurgard Stockholm
  Södermalm → boutique Ringvägen 77 ; seconde adresse de stockage du site :
  Västgötagatan 20 (obemannat).
- **Rejetés** : Ringvägen 77 (tous formats/accents), Västgötagatan 20, Vanadisvägen 4
  (Shurgard Vanadis, seul site de Vasastan), Södermalm, Stockholm, Odengatan 40/65,
  Norrtullsgatan 4/6, Renstiernas gata 37 (Bahnhof Pionen), Avarn Security « SAFE Home »
  (produit homonyme)…
- Le webmail ne contient que 7 mails + 1 brouillon identiques entre déploiements ;
  `constat_interieur.jpg` est un **placeholder** assumé (« CONSTAT INTERIEUR — placeholder »).
- Canal Telegram `t.me/AnotherBizarreChannel` : n'existe pas (vérifié avec une session
  Telegram réelle) ; Discord HackerLab 2026 : aucune mention pertinente.

**Conclusion** : la donnée attendue n'était pas dérivable du corpus (PJ intérieure stubbée,
canal externe supprimé) — à signaler aux organisateurs. Voir [../non-resolus.md](../non-resolus.md).
