# Challenges non résolus — rétrospective finale

État au coup de sifflet (17/11/2026). Pour chaque challenge : ce qui est prouvé, ce qui a
été rejeté, et le diagnostic.

---

## Safe Home (P5, 200 pts) — 0 solve global

Voir [jeopardy/man-who-sold-the-world.md](jeopardy/man-who-sold-the-world.md#p5--safe-home-200-pts--non-résolu).

**Diagnostic** : PJ intérieure = placeholder assumé ; canal Telegram référencé supprimé ;
~50 tentatives, toutes les adresses Shurgard plausibles rejetées. Probable challenge
incomplet ou donnée externe manquante.

---

## The Scorpion's Lair (212, forensics, 50 pts)

Description vide, aucun fichier, tag `q:9`. L'espace Kibana `cobra` expose la data view
« DFIR Logs » mais **aucun dashboard** « Journaux DFIR » ; les routes Discover/Search et
index management sont désactivées par la configuration courante. Aucune donnée de
challenge lue → rien à dériver.

**Diagnostic** : le contenu devait être délivré ailleurs (mission/DFIR/speedrun) ;
`q:9` = 9e question d'une chaîne dont les précédentes n'étaient pas accessibles.

---

## Sealed Order (250) / Deep Archive (255) — AD Tervalis

Voir [jeopardy/tervalis-ad.md](jeopardy/tervalis-ad.md#sealed-order-q250-bis--deep-archive-q255--non-résolus).

**Diagnostic** : accès SMB/LDAP larges obtenus mais les flags exigent une identité
à privilèges supérieurs (Administrator local FS01) non atteinte avant expiration des
instances (quota 3 instances concurrentes).

---

## The Gate (259, web, 70 pts)

- Dashboard affichant `HLB2026{N0RTHL1NE_P4RTN3R_G4TE}` = **leurre** (refusé).
- QR code → service de coordination `192.168.73.128:8182` (réseau interne du conteneur),
  non atteint (routage WireGuard /32 refusé par RTNETLINK).

**Piste restante** : atteindre le service de coordination = « informations permettant au
centre de coordination d'intercepter Thomas Crown ».

---

## Canary (216, 200 pts)

- `/api/telemetry` sans authentification : état `ARMED` puis `LEAKED`, 5 canaux,
  dossier `DOSSIER-002`, `safe_coord=46.045,5.87` — le flag n'est pas révélé.
- `/app.js` documente des formulaires opérationnels protégés par une clé de service.
- SQLi unique sur `/login` échouée ; le flag de Pre_Canary (215) ou des identifiants
  « Janus » étaient probablement la clé de la suite.

---

## Magik Box 1 (286, 100 pts)

Détails CTFd **vides** (pas de description, pas de fichier, pas de hint), mais 5 équipes
solveuses affichées → challenge probablement résolu via un canal hors description
(Discord/annonce) non identifié.

---

## Zebulon (274) — partiel

Q5/Q6/Q7 résolues (voir kiss-of-death.md). Le flag final « Coffre+ » (écran VAULT prévu
dans le CSS mais non câblé dans le JS) n'a jamais été dérivable : ~360 variantes rejetées
avant la résolution de Q5 qui clôturait la série accessible.

---

## Méta-leçons de la finale

1. **Un rejet ≠ une mauvaise réponse définitive** : deux flags ont été corrigés par
   l'organisateur en cours d'épreuve (Zebulon Q7, The Five of THOR). Relire les annonces
   et hints avant d'abandonner une piste prouvée.
2. **0 solve global + contenu prouvé = bug de flag auteur** : signaler tôt avec les
   preuves (hashes, chemins) plutôt que brute-forcer des formats.
3. **Le tag `flag:HLB2026{...}` = format exact attendu** : l'auteur utilise des
   placeholders littéraux (`ApplicationName:Code`, `prestataire_quartier_rue`).
4. **Quota d'instances (3) = ressource stratégique** : planifier les déploiements,
   détruire proprement, ne pas bloquer l'équipe sur une instance personnelle.
5. **Vérifier la validité du pipeline de soumission** (flag déjà accepté →
   `already_solved`, flag aléatoire → `incorrect` réel) avant de diagnostiquer un
   challenge cassé.
