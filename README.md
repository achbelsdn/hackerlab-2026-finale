# HackerLab Bénin 2026 — Finale (write-ups)

Write-ups de la **finale HackerLab Bénin 2026** (15–17 novembre 2026), réalisés par
l'équipe **DIVISION_0**. Ils sont rédigés dans un cadre pédagogique : chaque challenge
est présenté avec son contexte, sa chaîne de résolution complète et les leçons à retenir.

- Plateforme : `ctf.hackerlab.bj` (CTFd), avec en complément `speedrun.hackerlab.bj`
  (The Gauntlet) et `dfir.hackerlab.bj` (ELK).
- Format de flag : `HLB2026{...}`
- Fenêtre de l'épreuve : du vendredi 15 novembre 2026, 14 h 00 UTC, au dimanche
  17 novembre 2026, 14 h 00 UTC (H+48).
- Résultat de l'équipe : **12 146 points**, 6e du classement général.

> Publication post-événement, à but pédagogique. Les identifiants d'équipe, les
> configurations VPN et les secrets de session sont volontairement absents de ce dépôt.

## Sommaire

### Missions (jeopardy)

| Mission | Questions | Points | Statut | Write-up |
|---|---|---|---|---|
| Aizen | Cagliostro (web), Ishikawa Goemon (pwn), Fujiko Mine (rev), Zenigata (forensics) | 4 × 200 | 3/4 résolues | [jeopardy/aizen.md](jeopardy/aizen.md) |
| The Man Who Sold The World | Silent Noise, The Real Slim Shady, Back Door Man, Pattern of Life, Safe Home | 100 à 200 | 4/5 résolues | [jeopardy/man-who-sold-the-world.md](jeopardy/man-who-sold-the-world.md) |
| Node To Nowhere | The Five of THOR, Connecting the Dots | 200 | 2/2 résolues | [jeopardy/node-to-nowhere.md](jeopardy/node-to-nowhere.md) |
| Kiss of Death | Zebulon, Papyrus, Dreamer, Poseidon… | 50 à 500 | plusieurs résolues | [jeopardy/kiss-of-death.md](jeopardy/kiss-of-death.md) |
| The Fall Of JANUS | Ransomless, The Trojan Horse, New Blood, Royal Heist | 4 × 100 | 3/4 résolues | [jeopardy/fall-of-janus.md](jeopardy/fall-of-janus.md) |
| Broken relay | Dirty Story, Broken channel, Conspiracy | 3 × 100 | 2/3 résolues | [jeopardy/broken-relay.md](jeopardy/broken-relay.md) |
| The Fugitive | Ghost Passenger → Last Port of Call (6 questions) | 200 (mission) | 6/6 résolues | [jeopardy/the-fugitive.md](jeopardy/the-fugitive.md) |
| GHOST RATCHET | NordLink (ratchet Android/crypto) | 100 | résolue | [jeopardy/ghost-ratchet.md](jeopardy/ghost-ratchet.md) |
| Contingency Protocol | Override (web3) | 200 | résolue | [jeopardy/contingency-protocol.md](jeopardy/contingency-protocol.md) |
| Toguro Tempest | Gitea (web) | 200 | résolue | [jeopardy/toguro-tempest.md](jeopardy/toguro-tempest.md) |
| Tervalis (AD) | File Vault, Silent Broker | 200 | File Vault résolue | [jeopardy/tervalis-ad.md](jeopardy/tervalis-ad.md) |
| Human After All | Human After All 1 (misc) | 100 | résolue | [jeopardy/human-after-all.md](jeopardy/human-after-all.md) |
| Canary | Pre_Canary (crypto) | 50 | résolue | [jeopardy/pre-canary.md](jeopardy/pre-canary.md) |

### Attack / Defense (ARETZ DAM)

| Épreuve | Description | Résultat | Write-up |
|---|---|---|---|
| King of the Throne — Pool A « Cerberus » | Cinq flags à extraire d'une box durcie, en défendant sa position | 1er du pool | [attack-defense/koth-cerberus.md](attack-defense/koth-cerberus.md) |
| Death Match — Battle 07 « Patchwork » | Box d'attaque contre box de défense, RCE PHP | +200 points | [attack-defense/deathmatch-07-patchwork.md](attack-defense/deathmatch-07-patchwork.md) |

### Speedrun — The Gauntlet (DFIR)

| Épreuve | Description | Statut | Write-up |
|---|---|---|---|
| SpeedRun DFIR HLB2K26 | 30 questions, 60 secondes par question, corpus ELK de 7,1 millions d'événements | terminée | [speedrun/gauntlet-dfir.md](speedrun/gauntlet-dfir.md) |

### Challenges non résolus (rétrospective)

[non-resolus.md](non-resolus.md) — Safe Home (P5), Royal Heist, Conspiracy,
Welcome Back / Game On, The Scorpion's Lair, Sealed Order, Deep Archive, The Gate,
Canary, Zebulon… avec l'état des investigations et les leçons tirées.

## Leçons transverses

- **Extraire tous les flux annexes** d'un conteneur multimédia (sous-titres, chapitres,
  métadonnées) avant d'envisager une stéganographie complexe dans les pixels ou l'audio.
- **Ne pas sur-interpréter les indices** : lorsqu'un en-tête décrit la chaîne
  cryptographique (`FEISTEL8-KWRAP-CBC256`), il la décrit réellement. Scanner le produit
  cartésien des primitives natives disponibles vaut mieux que réimplémenter à l'aveugle.
- **Un indice de format est un contrat** : l'intitulé `flag:HLB2026{...}` affiché sur le
  panneau d'une mission indique le format exact attendu (placeholders littéraux,
  underscores pour les espaces).
- **Vérifier l'expansion locale des charges utiles** avant de conclure à une injection
  côté serveur : un faux positif de shell coûte cher en temps d'analyse.
- **En attack/defense, corriger la box miroir dès l'identification du vecteur** : les
  deux machines exécutent le même code, la faille exploitée chez l'adversaire existe
  donc chez soi.
- **Un rejet n'est pas toujours définitif** : deux flags ont été corrigés par les
  organisateurs en cours d'épreuve ; relire les annonces avant d'abandonner une piste
  pourtant démontrée.
