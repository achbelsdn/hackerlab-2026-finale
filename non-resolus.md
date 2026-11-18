# Challenges non résolus — rétrospective de la finale

État des lieux au coup de sifflet (17 novembre 2026). Pour chaque challenge : ce qui est
établi, ce qui a été écarté, et le diagnostic final. Ces retrospectives sont aussi
formatrices que les réussites.

---

## Safe Home (P5, 200 points) — aucune résolution sur la plateforme

Le détail complet figure dans
[jeopardy/man-who-sold-the-world.md](jeopardy/man-who-sold-the-world.md#p5--safe-home-200-points--non-résolue).

**Diagnostic** : la pièce jointe intérieure est un placeholder assumé, le canal Telegram
référencé a été supprimé, et toutes les adresses Shurgard plausibles ont été explorées
sans succès. Challenge probablement incomplet, ou donnée attendue externe au corpus.

---

## The Scorpion's Lair (212, forensique, 50 points)

Description vide, aucun fichier joint, balise `q:9`. L'espace Kibana `cobra` expose la
vue de données « DFIR Logs », mais **aucun tableau de bord** « Journaux DFIR » ; les
routes Discover/Search et la gestion des index sont désactivées par la configuration
courante. Aucune donnée de challenge n'a donc pu être consultée.

**Diagnostic** : le contenu devait être délivré ailleurs (mission, DFIR ou speedrun) ;
`q:9` désigne la neuvième question d'une chaîne dont les précédentes n'étaient pas
accessibles à notre équipe.

---

## Sealed Order (250) et Deep Archive (255) — scénario Active Directory Tervalis

Le détail des accès obtenus figure dans
[jeopardy/tervalis-ad.md](jeopardy/tervalis-ad.md#sealed-order-et-deep-archive-q255--non-résolues).

**Diagnostic** : les accès SMB/LDAP étaient larges, mais les deux flags exigeaient une
identité à privilèges supérieurs (compte Administrator local de FS01) qui n'a pas été
obtenue avant l'expiration des instances, le quota de trois instances simultanées
limitant les redéploiements.

---

## The Gate / Last Departure (259, web, 70 points)

Le détail des accès obtenus (préfixe `PRT`, session partenaire Skarvik Marine
Services, expédition NL-24-7K42 = Thomas Crown) figure dans les notes de reprise.
Points établis :

- Le tableau de bord affichait `HLB2026{N0RTHL1NE_P4RTN3R_G4TE}` : un **leurre** clair.
- Le QR du reçu servi par `/receipt` pointe le service interne **`northline.lab:8182`**
  (le QR du PNG local, adresse de dev `192.168.73.128:8182`, est différent).
- Le vecteur est le formulaire `/waybills` (champ `source_url`, SSRF), mais la
  validation pré-fetch (« Adresse de préparation non reconnue. ») n'a jamais été
  satisfaite : aucun fetch observé vers nos listeners, quel que soit le candidat.
- **En fin d'épreuve**, la session authentifiée a enfin été atteinte (route via le
  tunnel `dvpn-gate`, puis liaison du socket directement à `wg259` pour contourner
  la route système). La réponse du serveur a livré un indice concret :
  `X-Upstream-Service: dispatch.northline.internal:8081`.
- Ce nom interne est **accepté par le formulaire** — ce qui confirme l'allowlist —
  mais le résolveur DNS du service échoue (`Name or service not known`), y compris
  avec l'alias Docker court `dispatch`. Le service de répartition était donc
  atteignable par son nom… si l'instance avait résolu son propre DNS interne.

**Diagnostic** : il ne restait qu'une variante de résolution à trouver (ou un
redéploiement sain de l'instance) quand le temps a expiré. La chaîne complète
préfixe → session → SSRF était maîtrisée ; c'est l'infrastructure DNS interne qui
a fait défaut.

---

## Canary (216, 200 points)

- L'endpoint `/api/telemetry`, accessible sans authentification, rapporte l'état
  `ARMED` puis `LEAKED`, cinq canaux, le dossier `DOSSIER-002` et les coordonnées
  `safe_coord=46.045,5.87` — sans révéler le flag.
- Le fichier `/app.js` documente des formulaires opérationnels protégés par une clé de
  service.
- Une tentative d'injection SQL sur `/login` a échoué ; le flag de Pre_Canary (215) ou
  des identifiants « Janus » étaient probablement la clé de la suite.

---

## Magik Box 1 (286, 100 points)

Les détails du challenge étaient **vides** (ni description, ni fichier, ni indice),
alors que cinq équipes figuraient parmi les résolvantes : le point d'entrée se trouvait
probablement dans un canal hors description (annonce, Discord) que nous n'avons pas
identifié.

---

## Zebulon (274) — partiellement résolu

Les questions 1 à 4 de la mission (Jericho, Confession, Enola, Mayim… sauf Mayim)
voire [jeopardy/kiss-of-death.md](jeopardy/kiss-of-death.md). La question 5, drapeau
final « Coffre+ » au format `HLB2026{ApplicationName:Code}`, n'a jamais été
dérivable :

- la dérivation serveur est **établie** : `POST /api/real {token,bearing}` bascule
  le profil réel, l'azimut **118°** est validé exactement par le serveur (zone
  holding 114–122°, dwell 2 000 ms) — l'application est « Boussole » ; un scan
  complet des 360 azimuts ne révèle qu'un seul profil réel ;
- le téléphone perquisitionné (ADB) ne livre rien : schéma + PIN, locksettings.db
  de 118 octets (écho de l'azimut), `/data/data/com.coffreplus.vault` vide ;
- **~334 tentatives** soumises par l'équipe (≈190 formes uniques de
  `Boussole:118`, `Coffre+:…`, etc.), **0 solve à l'échelle du CTF** : la question
  est restée à 0 résolveur jusqu'à la fin, signe d'un flag incohérent ou d'un code
  attendu provenant d'une source jamais livrée (laptop du suspect, sim DOWN).

---

## Royal Heist (Q4 de The Fall Of JANUS, 100 points)

> L'attaquant a exporté la base de données des verdicts, puis l'a transférée vers
> une infrastructure distante. Déterminez l'heure du lancement de l'exportation,
> l'adresse IP et le port de destination du transfert.

La chronologie PowerShell établissait l'export vers `verdicts_dump.sql` puis un POST
vers `192.168.230.130:8700`. La soumission exigeait de recouper l'heure réelle du
lancement de `mysqldump` (Prefetch, horodatage du fichier exporté) avec l'heure des
commandes de transfert — distinction non tranchée avant la fin. Voir
[jeopardy/fall-of-janus.md](jeopardy/fall-of-janus.md).

---

## Conspiracy (Q3 de Broken relay, 100 points)

> Nous avons procédé à une saisie de la plateforme utilisée par le suspect pour
> communiquer avec ses complices. Pouvez-vous reconstituer les messages ?

Les questions 1 (Dirty Story, URL de la plateforme) et 2 (Broken channel, heure de
première exécution du programme de chiffrement) étaient validées ; la troisième
exigeait de reconstituer les messages d'une seconde archive Mega saisie. Elle est
restée ouverte. Voir [jeopardy/broken-relay.md](jeopardy/broken-relay.md).

---

## Welcome Back / Game On (278, misc, 100 points)

Description : `Find me.` — trois indices gratuits sur le nœud `welcome` de la carte :

1. `𝄞 est la clé. Listen me` (la clé de sol → « clé de SOL » ?) ;
2. `Courage` ;
3. `What does Discord say I'm doing ?` (le statut d'activité Discord de l'auteur
   W1z4rd).

La vidéo « Précédemment · Événements de Tervalis » (`precedent-hlb2026-v5.mp4`,
357 s, musique de 220 s à 357 s) a été téléchargée et analysée : spectrogrammes
(full/side/mid/L/R, diffs par segment) sans texte caché lisible. Le widget Discord
de la guilde HackerLab 2026 était désactivé (erreur 50004) et la preview renvoyait
401 : consulter le statut d'activité exigeait un compte connecté, ce qui n'a pas été
possible avant la fin.

**Diagnostic** : la musique de la vidéo restait à identifier (Shazam/manual), puis
le statut Discord de l'auteur donnait probablement le flag. Deux sources externes,
zéro dérivation locale possible.

---

## Leçons méthodologiques de la finale

1. **Un refus ne clôt pas l'investigation** : deux flags ont été corrigés par
   l'organisateur en cours d'épreuve (Zebulon Q7, The Five of THOR). Relire les
   annonces et les indices avant d'abandonner une piste démontrée.
2. **Réponse démontrée + aucune résolution au classement = soupçon de flag
   incohérent** : signaler tôt aux organisateurs, preuves à l'appui (empreintes,
   chemins), plutôt que multiplier les variantes de format.
3. **La balise `flag:HLB2026{...}` est le format exact attendu** : l'auteur emploie des
   placeholders littéraux (`ApplicationName:Code`, `prestataire_quartier_rue`).
4. **Le quota d'instances (trois) est une ressource stratégique** : planifier les
   déploiements, libérer proprement, ne jamais bloquer l'équipe sur une instance
   personnelle.
5. **Valider d'abord la chaîne d'outils** (identifiant de challenge correct, format de
   flag du bon mode de jeu) avant de diagnostiquer un challenge défaillant.
