# Human After All 1 (misc, 100 pts)

**Équipe** : DIVISION_0 — **Statut** : Q4 résolue, Q5 non résolue
**Flag (Human After All 1)** : `HLB2026{78ff736fd389aa91a3df8e9e5a2db41c}`

**Objectif** : obtenir des informations sur `mstrand` (leaker) via une infrastructure
de communication simulée (instance éphémère, VPN WireGuard `172.28.x.0/24`).

---

## Résolution (question 4)

1. Recon de l'instance : services de messagerie/annuaire simulés, plusieurs identités.
2. Les défenses anti-bruteforce sont **délibérées** — la brute-force est un piège
   (rate-limiting agressif, réponses uniformes).
3. La voie gagnante est la **corrélation d'identité** : recouper les métadonnées
   disponibles sur les différents canaux pour identifier le numéro/compte de `mstrand`
   plutôt que deviner.
4. Flag dérivé depuis l'instance puis accepté par le portail :

   ```
   HLB2026{78ff736fd389aa91a3df8e9e5a2db41c}
   ```

   La soumission débloque la question 05 (téléphone).

## Question 5 — non résolue

- Piste principale : le profil « humain » de la cible (ingénierie sociale du service
  7.3) et une présence X (Twitter) de l'auteur.
- Instances successives (4f77bca9, etc.) sans nouveau point d'entrée dérivé avant la fin.

## Leçons

- Quand un challenge met en place des **défenses anti-bruteforce délibérées**, c'est un
  signal : la solution passe par la corrélation d'informations, pas par l'énumération.
- Gérer les instances éphémères par script (déploiement, attache VPN, expiration) pour
  ne pas perdre de temps de réflexion sur de la plomberie.
