# The Man Who Sold The World — chaîne OSINT (cinq questions)

**Catégorie** : OSINT, misc, web — **Équipe** : DIVISION_0 — **Statut** : quatre
questions résolues sur cinq.

**Contexte narratif** : Norveth a poussé Tervalis vers des opérations clandestines ; un
agent de liaison, **Thomas Crown**, en a conservé la preuve. Une vidéo « parfaitement
anodine » constitue le point d'entrée : l'identifiant YouTube `gH3WC-6ZqUw`.

---

## P1 — Silent Noise (100 points)

**Flag** : `HLB2026{5ubtl3_5ubt1tl3_h1dd3n_1n_pl41ns1ght}`

### Résolution

1. Un indice gratuit de la plateforme, **« Sous-Titres »**, débloque le fichier
   additionnel `meme.mkv`.
2. L'inspection avec `ffprobe` révèle **quatre pistes de sous-titres SRT** :
   - pistes 0 à 2 : dialogues leurres en anglais, français et espagnol ;
   - **piste 3 (étiquette `eus`, basque)** : un bloc encodé en **Base64**.
3. Extraction puis décodage :

   ```bash
   ffmpeg -i meme.mkv -map 0:s:3 sub3.srt
   ```

   Le Base64 décode en un message **PGP clearsigné**, signé par « yoda », qui contient
   en clair :

   ```
   Code de validation : HLB2026{5ubtl3_5ubt1tl3_h1dd3n_1n_pl41ns1ght}
   ```

**Leçon** : extraire systématiquement tous les flux annexes d'un conteneur multimédia
(`ffprobe -show_streams` : sous-titres, chapitres, métadonnées) avant d'explorer une
stéganographie des pixels ou de l'audio.

---

## P2 — The Real Slim Shady (100 points)

**Flag** : `HLB2026{3acd6d10080adbcfe91c62d4ac5649ac}`

### Résolution

1. La signature PGP de P1 porte l'identifiant de clé **`0x6071195312D405CB`** ; un
   serveur HKP (Hockeypuck 2.2.2) écoute sur le port 11371 de l'instance.
2. La clé correspondante présente un UID primaire `yoda <anakin@proton.me>` et un UID
   révoqué `Thomas Crown <thomas.crown@qmail.nrv>`.
3. **Recherche HKP par jeton** : le serveur apparie les requêtes sur le nom et la partie
   locale de l'adresse. Un balayage de jetons (script `hkp_bruteforce.sh`, jeton
   `hlb2026`) révèle une **seconde clé** dont l'UID est :

   ```
   Preuve d'acces (HLB2026{3acd6d10080adbcfe91c62d4ac5649ac}) <proof@qmail.nrv>
   ```

**Leçon** : la recherche HKP apparie sur un jeton exact (nom et partie locale de
l'adresse), jamais sur le domaine : il faut deviner le bon identifiant, pas le bon
fournisseur de messagerie.

---

## P3 — Back Door Man (100 points)

**Flag** : `HLB2026{5ffa72701330b0bc7001b23f9d64b4e9}`

### Résolution

1. Le webmail qmail (Flask sous gunicorn) écoute sur le port 8080 de l'instance.
2. La requête `POST /api/password/reset-request` **renvoie le jeton de réinitialisation
   dans la réponse JSON** : une fuite de conception complète.
3. La requête `POST /api/password/reset` avec ce jeton permet de fixer un nouveau mot de
   passe, puis de se connecter sous `thomas.crown@qmail.nrv`.
4. Le brouillon intitulé « notes » contient :

   ```
   jeton de session interne: HLB2026{5ffa72701330b0bc7001b23f9d64b4e9}
   ```

**Leçon** : une API de réinitialisation qui expose son jeton côté client équivaut à une
prise de contrôle totale des comptes.

---

## P4 — Pattern of Life (200 points)

**Flag retenu par la plateforme** : `HLB2026{ODENPLAN}`

### Résolution

1. Les courriels du webmail décrivent le quotidien de Thomas Crown : reçus de
   stationnement résidentiel (en-tête `X-Originating-Zone: SE-AB-Vasastan`), commerces
   fréquentés autour d'**Odenplan**, à Stockholm (quartier de Vasastan).
2. Les réponses trop larges (`VASASTAN`, `STOCKHOLM`, `SE-AB-Vasastan`) sont refusées :
   l'énoncé demande le **point de vie précis**, à savoir `HLB2026{ODENPLAN}`.

### Remarque sur la pièce jointe

`entree_site.jpg` (440 × 509) est un recadrage de la photographie officielle Shurgard
« Stockholm Södermalm » : le template matching donne une erreur quadratique moyenne
d'environ 6, contre 5 100 pour la meilleure autre candidate, soit l'adresse Ringvägen 77,
118 61 Stockholm. Cette piste a nourri la question suivante, mais a été refusée pour P4.

---

## P5 — Safe Home (200 points) — non résolue

> « La preuve dort dans un coffre, quelque part dans la ville. Mais où ? »

**Format officiel** (balise de l'API) : `flag:HLB2026{prestataire_quartier_rue}`.

L'investigation a été menée à exhaustion :

- **Établi** : `entree_site.jpg` est un recadrage de la photo officielle Shurgard
  Stockholm Södermalm (boutique au Ringvägen 77) ; la seconde adresse de stockage du
  site est Västgötagatan 20 (site sans personnel).
- **Réponses écartées** : Ringvägen 77 (tous formats et accents), Västgötagatan 20,
  Vanadisvägen 4 (Shurgard Vanadis, seul site de Vasastan), Södermalm, Stockholm,
  Odengatan 40 et 65, Norrtullsgatan 4 et 6, Renstiernas gata 37 (data center Bahnhof
  Pionen), le produit « SAFE Home » d'Avarn Security (homonyme troublant)…
- Le webmail ne contient que sept messages et un brouillon, identiques d'un déploiement
  à l'autre ; `constat_interieur.jpg` est un **placeholder** assumé (« CONSTAT INTERIEUR
  — placeholder »).
- Le canal Telegram `t.me/AnotherBizarreChannel` n'existe pas (vérifié avec une session
  réelle) ; aucune mention pertinente sur le Discord de l'événement.

**Diagnostic** : la donnée attendue n'était pas dérivable du corpus fourni (pièce jointe
intérieure remplacée par un placeholder, canal externe supprimé). Le dossier complet
figure dans [../non-resolus.md](../non-resolus.md).
