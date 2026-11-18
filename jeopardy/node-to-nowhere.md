# Node To Nowhere — traçage blockchain (2 questions)

**Catégorie** : forensics / blockchain — 200 pts chacun — **Équipe** : DIVISION_0
**Statut** : 2/2 résolues (Q207 après correction organisateur, Q211 après hint officiel)

Narratif : suivre le produit d'un swap THORChain depuis son point d'entrée jusqu'à sa
destination, à travers Bitcoin et Ethereum.

---

## Q2 — The Five of THOR (200 pts)

**Flag** (corrigé par l'organisateur en cours d'épreuve) :

```
HLB2026{bc1qhclwkt999458pek36ukcxvh23da3ntp5vyaq4s,bc1qegtq7nsmy7knv48lsy2telxshmtutatl25qtf4,bc1qa6te8zckz0tx688af3vzvatlnfy0lvcxx5l4pk,bc1qaaneuqyjqnwm9mpsz3easnpneqslqa3nt3a98l,bc1q9e6s4kxj3yt43ts60yl4wwdgsu3enn9tjvfvgr}
```

### Résolution

1. Un fichier `hachage.txt` fournit 5 SHA-256. **Preuve indépendante** : chaque préimage
   est une adresse bech32 — `sha256(bech32_addr)` = les 5 cibles, dans l'ordre du fichier.
2. Chaque adresse reçoit directement un outbound d'un swap THORChain via une adresse
   BTC legacy intermédiaire (sweeps datés 1740942810 / 1740944176) :

   | # | Adresse cible (SHA-256 préimage) | Sweep | Payout |
   |---|---|---|---|
   | 1 | `bc1qhclwkt…` | `2b6a2776…` | `41fb88ec…` |
   | 2 | `bc1qegtq7ns…` | `ddedc9f5…` | `e47d4949…` |
   | 3 | `bc1qa6te8zc…` | `679de681…` | `0d2a92c6…` |
   | 4 | `bc1qaaneuqy…` | `72eeb2d7…` | `5613931a…` |
   | 5 | `bc1q9e6s4kx…` | `37ebfc8e…` | `92716bbb…` |

3. Le flag = les 5 adresses bech32 **dans l'ordre du fichier de hashes**, séparées par
   des virgules sans espaces.

### Piège majeur

~365 soumissions rejetées avant la correction organisateur : toutes les permutations
d'ordre (temporel THOR, hauteurs de blocs), les txids de sweep/payout, les wrappers
(`HLB2O26{...}` typo de la description), les formats avec espaces… étaient refusés alors
que le contenu était **mathématiquement prouvé**. Le flag stocké ne correspondait à aucune
dérivation — l'organisateur a corrigé le flag plateforme, et la soumission « obvious »
(ordre fichier, virgules) est passée immédiatement.

**Leçon** : quand le contenu est prouvé et 0 solve global, c'est un bug de flag auteur —
signaler au lieu de brute-forcer.

---

## Q6 — Connecting the Dots (200 pts)

**Flag** (validé officiellement) :

```
HLB2026{74caf0e22e82c105358bdc912840ef0e08be05d84162fd6aabaf6c85277cfb5b,3f6403fffe71061be38f5d215aa9aa76afd395463463962e84fa17c9d3ddcf7a,d4f8337f921e5607035c7ff781c14dc853830f15dbc777c5e22dceca5fdcde33,0f1ade1492631f42953e748a066bf5281cbed6f3e67ca758536e86d5b50f09d0,f80b0fb8be1b6ee538b51d1947cc9ec8641b51439cda7ab1465c474e717b1f34,42b3c8bf0b963393c30dc2695138b9461a2991c139c364977b449d23aa9f0d5b,c1ac94f7ae75aeac2e22a8e1227dd441971bef1eeacc129def1aaa0a394964cc}
```

### Résolution

1. Le swap seed (midgard) : tx ETH
   `488A4636CC6F147A0A5D574067A73ECC0C17432EFA348E7F5D770BC338252703`,
   28.16 ETH depuis `0xe4ee9ed6…`, memo `=:BTC.BTC:bc1q2ckjg8a…:0/10/0`,
   sortie BTC `5983287f…` (0.80965529, streaming swap 9/9).
2. **Chemin de valeur unique** (chaque txid re-vérifié byte-exact via mempool.space) :

   ```
   5983287f → 74caf0e2 → 3f6403ff → d4f8337f → 0f1ade14 → f80b0fb8 → 42b3c8bf → c1ac94f7 → bc1qxn7a
   ```

3. **Hint officiel** publié par l'organisateur :

   > « N'incluez ni le hash de la transaction THORChain ni celui de la transaction
   > sortante du portefeuille (bc1qxn7a…). »

   → le flag = les **7 transactions intermédiaires** uniquement, virgules sans espaces.

### Analyse complémentaire (pourquoi c'était piégeux)

- `c1ac94f7` a **3 inputs** : le flux seed 8-tx, la chaîne dominante « DOUT » du vault
  THORChain (12 txs, memo OP_RETURN `DOUT:…`, swap 137.6 ETH), et de la poussière organique.
- La chaîne DOUT est **antérieure** au seed (1740226675 < 1740312577) : tout chemin
  seed→vault→DOUT est non-temporel (artefact d'outil d'analyse).
- Les deux chemins plausibles (8-tx temporel vs DOUT dominante) ont été rejetés ~160 fois
  avant le hint officiel.

**Leçon** : pour un flag « liste de txids », demander tôt le périmètre exact (inclure le
seed ? la sortie ?) — l'ambiguïté de format coûte plus que l'analyse on-chain.
