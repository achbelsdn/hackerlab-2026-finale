# Broken relay (forensics, 3 × 100 points)

**Identifiants des challenges** : Dirty Story (200), Broken channel (201),
Conspiracy (203) — **Équipe** : DIVISION_0 — **Statut** : 2/3 résolues.

---

## Énoncé

> Les enquêtes ont permis de remonter à un individu suspecté d'appartenir au groupe
> de hackers ayant initié les attaques contre Tervalis.

La mission s'appuie sur la machine du suspect (archive Mega `thanos_desktop.zip`,
1,16 Go) puis sur la saisie de la plateforme de communication du groupe.

## Q1 — Dirty Story (100 points)

> Nous suspectons le groupe… d'utiliser une plateforme secrète leur permettant de
> planifier leurs opérations. Identifiez sur la machine du suspect l'URL utilisée
> pour accéder à la plateforme.

**Format officiel** (balise de l'API) : `flag:HLB2026{http://domain.tld:XXXX}` —
l'URL complète avec son port.

**Résolue** : l'analyse du disque du suspect (historiques, configurations et
artefacts de navigation de `thanos_desktop.zip`) permet de retrouver l'URL exacte
de la plateforme de planification. Réponse validée par DIVISION_0.

## Q2 — Broken channel (100 points)

> Le suspect semble utiliser un programme spécial lui permettant d'appliquer un
> algorithme de chiffrement particulier. Pouvez-vous fournir l'heure à laquelle il
> a été exécuté pour la première fois ?

**Format officiel** : `flag:HLB2026{HH:MM}`.

**Résolue** : l'heure de première exécution se dérive des artefacts d'exécution
Windows (Prefetch, journaux) associés au binaire de chiffrement identifié sur la
machine. Réponse validée par DIVISION_0.

## Q3 — Conspiracy (100 points) — non résolue

> Nous avons procédé à une saisie de la plateforme utilisée par le suspect pour
> communiquer avec ses complices. Pouvez-vous reconstituer les messages ?
>
> Artéfacts : seconde archive Mega (`BhMwBBiL#…`).

**Format officiel** : `flag:HLB2026{}` (contenu libre).

**État de l'investigation** : la question est restée ouverte — la reconstitution
des messages supposait de comprendre le chiffrement appliqué par le programme de
Q2 puis de le dérouler sur les données saisies. Le temps disponible après la
validation de Q2 n'a pas suffi pour boucler cette chaîne.

## Leçons

- **Les balises de format sont un contrat** : `http://domain.tld:XXXX` et `HH:MM`
  indiquent exactement la forme attendue — pas de chemin, pas de secondes, pas
  d'habillage.
- **Une mission en trois questions se planifie** : Q3 dépendait du programme
  identifié en Q2 ; engager la saisie des messages dès la compréhension du
  chiffrement (et non après la validation) aurait laissé le temps de conclure.
- **Les archives Mega de plusieurs Go se traitent hors machine locale** : disque
  mémoire, VM distante ou streaming sélectif — le disque système ne suit jamais.
