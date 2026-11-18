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

## The Gate (259, web, 70 points)

- Le tableau de bord affichait `HLB2026{N0RTHL1NE_P4RTN3R_G4TE}` : un **leurre** clair.
- Un code QR désignait le service de coordination à l'adresse `192.168.73.128:8182`
  (réseau interne du conteneur), resté inaccessible : l'ajout d'une route `/32` vers le
  tunnel concerné était refusé par le noyau (`RTNETLINK Operation not permitted`).

**Piste restante** : joindre le service de coordination, qui correspond à
« l'information permettant au centre de coordination d'intercepter Thomas Crown ».

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

Les questions 5, 6 et 7 sont résolues (voir
[jeopardy/kiss-of-death.md](jeopardy/kiss-of-death.md)). Le flag final « Coffre+ » —
dont l'écran était prévu dans la feuille de style mais jamais câblé dans le code
JavaScript — n'a jamais été dérivable : la question fermait une série dont les maillons
précédents manquaient.

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
