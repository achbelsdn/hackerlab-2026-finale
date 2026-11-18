# HackerLab Bénin 2026 — Finale (write-ups)

Write-ups de la **finale HackerLab Bénin 2026** (15–17 novembre 2026) réalisés par l'équipe **DIVISION_0**.

- Plateforme : `ctf.hackerlab.bj` (CTFd) + `speedrun.hackerlab.bj` (The Gauntlet) + `dfir.hackerlab.bj` (ELK)
- Format de flag : `HLB2026{...}`
- Fenêtre : H = 15/11/2026 14:00 UTC → H+48 = 17/11/2026 14:00 UTC
- Score final DIVISION_0 : **12 146 pts** (6e au classement général)

> ⚠️ Publié après la fin de l'événement, à but pédagogique. Les identifiants d'équipe,
> configurations VPN et secrets de session sont volontairement absents de ce dépôt.

## Sommaire

### Missions (jeopardy)

| Mission | Questions | Points | Statut | Write-up |
|---|---|---|---|---|
| Aizen | Cagliostro (web), Ishikawa Goemon (pwn), Fujiko Mine (rev), Zenigata (forensics) | 4×200 | 3/4 résolues | [jeopardy/aizen.md](jeopardy/aizen.md) |
| The Man Who Sold The World | Silent Noise, The Real Slim Shady, Back Door Man, Pattern of Life, Safe Home | 100–200 | 4/5 résolues | [jeopardy/man-who-sold-the-world.md](jeopardy/man-who-sold-the-world.md) |
| Node To Nowhere | The Five of THOR, Connecting the Dots | 200 | 2/2 résolues (après correction organisateur) | [jeopardy/node-to-nowhere.md](jeopardy/node-to-nowhere.md) |
| Kiss of Death | Zebulon (téléphone), Papyrus, Dreamer, Poseidon… | 50–500 | plusieurs résolues | [jeopardy/kiss-of-death.md](jeopardy/kiss-of-death.md) |
| Toguro Tempest | Gitea (web) | 200 | résolue | [jeopardy/toguro-tempest.md](jeopardy/toguro-tempest.md) |
| Tervalis (AD) | File Vault, Silent Broker | 200 | File Vault résolue | [jeopardy/tervalis-ad.md](jeopardy/tervalis-ad.md) |
| Human After All | Human After All 1 (misc) | 100 | résolue | [jeopardy/human-after-all.md](jeopardy/human-after-all.md) |
| Canary | Pre_Canary (crypto) | 50 | résolue | [jeopardy/pre-canary.md](jeopardy/pre-canary.md) |

### Attack / Defense (ARETZ DAM)

| Épreuve | Description | Statut | Write-up |
|---|---|---|---|
| King of the Throne — Pool A « Cerberus » | 5 flags sur une box durcie, king of the hill | 1er du pool | [attack-defense/koth-cerberus.md](attack-defense/koth-cerberus.md) |
| Death Match — Battle 07 « Patchwork » | Box attaque vs box défense, RCE PHP | +200 pts | [attack-defense/deathmatch-07-patchwork.md](attack-defense/deathmatch-07-patchwork.md) |

### Speedrun — The Gauntlet (DFIR)

| Épreuve | Description | Statut | Write-up |
|---|---|---|---|
| SpeedRun DFIR HLB2K26 | 30 questions, 60 s/question, corpus ELK 7,1 M d'événements | terminé | [speedrun/gauntlet-dfir.md](speedrun/gauntlet-dfir.md) |

### Non résolus (rétrospective)

[non-resolus.md](non-resolus.md) — Safe Home (P5), The Scorpion's Lair, Sealed Order,
Deep Archive, The Five of THOR avant correction, The Gate, Canary… avec les pistes
explorées et les leçons.

## Leçons transverses

- **Extraire tous les flux annexes** d'un conteneur multimédia (sous-titres, chapitres,
  metadata) avant de chercher de la stégano complexe.
- **Ne pas sur-interpréter** : quand un header décrit la chaîne crypto (`FEISTEL8-KWRAP-CBC256`),
  il la décrit vraiment. Scanner le produit cartésien des primitives natives via `ctypes`
  plutôt que réimplémenter à l'aveugle.
- **Un rejet n'est pas définitif** : l'auteur peut corriger un flag en cours d'épreuve
  (Zebulon Q7, The Five of THOR) — relire les hints/annonces avant d'abandonner.
- **Le tag `flag:HLB2026{...}` du panneau mission est le format EXACT attendu**
  (placeholders littéraux, underscores pour espaces).
- **Vérifier si un payload a été expansé localement** avant de conclure à une injection
  serveur (faux positifs shell).
- **En A/D** : patcher la box miroir dès l'identification du vecteur — les deux box
  exécutent le même code.
