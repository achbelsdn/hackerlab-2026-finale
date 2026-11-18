# Human After All 1 (misc, 100 points)

**Équipe** : DIVISION_0 — **Statut** : question 4 résolue, question 5 non résolue.
**Flag (Human After All 1)** : `HLB2026{78ff736fd389aa91a3df8e9e5a2db41c}`

**Objectif** : identifier des informations sur `mstrand`, la source de la fuite, à
travers une infrastructure de communication simulée (instance éphémère, tunnel
WireGuard sur un sous-réseau `172.28.x.0/24`).

---

## Résolution (question 4)

1. Reconnaissance de l'instance : services de messagerie et d'annuaire simulés,
   plusieurs identités coexistantes.
2. Les défenses anti-force-brute sont **délibérées** (limitation de débit agressive,
   réponses uniformes) : l'énumération aveugle est un piège assumé par l'auteur.
3. La voie gagnante est la **corrélation d'identités** : recouper les métadonnées
   disponibles sur les différents canaux pour relier les comptes entre eux et isoler le
   numéro de `mstrand`, plutôt que deviner.
4. Le flag dérivé de l'instance est le suivant :

   ```
   HLB2026{78ff736fd389aa91a3df8e9e5a2db41c}
   ```

   Sa validation ouvre la question 5, consacrée au téléphone de la cible.

## Question 5 — non résolue

- Piste principale : le profil « humain » de la cible (ingénierie sociale du service
  mentionné en 7.3) et une présence sur X (Twitter) côté auteur.
- Plusieurs instances successives ont été déployées sans qu'un nouveau point d'entrée
  dérivable n'apparaisse avant la fin de l'épreuve.

## Leçons

- Lorsqu'un challenge met en place des **défenses anti-force-brute délibérées**, c'est
  un signal de conception : la solution passe par la corrélation d'informations, pas
  par l'énumération.
- Scripter la gestion des instances éphémères (déploiement, rattache VPN, expiration)
  afin de consacrer le temps de réflexion à l'analyse plutôt qu'à la plomberie.
