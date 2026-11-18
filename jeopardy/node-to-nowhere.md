# Node To Nowhere — traçage blockchain (deux questions)

**Catégorie** : forensique blockchain — 200 points chacune — **Équipe** : DIVISION_0
**Statut** : les deux questions résolues, dont une après correction du flag par les
organisateurs.

**Contexte narratif** : suivre le produit d'un échange THORChain depuis son point
d'entrée jusqu'à sa destination finale, à travers les chaînes Bitcoin et Ethereum.

---

## Q2 — The Five of THOR (200 points)

**Flag** (dans sa version corrigée par les organisateurs) :

```
HLB2026{bc1qhclwkt999458pek36ukcxvh23da3ntp5vyaq4s,bc1qegtq7nsmy7knv48lsy2telxshmtutatl25qtf4,bc1qa6te8zckz0tx688af3vzvatlnfy0lvcxx5l4pk,bc1qaaneuqyjqnwm9mpsz3easnpneqslqa3nt3a98l,bc1q9e6s4kxj3yt43ts60yl4wwdgsu3enn9tjvfvgr}
```

### Résolution

1. Le fichier `hachage.txt` fournit cinq empreintes SHA-256. **Preuve indépendante** :
   chaque préimage est une adresse bech32 — `sha256(adresse_bech32)` redonne les cinq
   valeurs cibles, dans l'ordre du fichier.
2. Chaque adresse reçoit directement une sortie d'un échange THORChain, via une adresse
   Bitcoin legacy intermédiaire (balayages horodatés 1740942810 et 1740944176) :

   | # | Adresse cible (préimage du SHA-256) | Balayage | Paiement |
   |---|---|---|---|
   | 1 | `bc1qhclwkt…` | `2b6a2776…` | `41fb88ec…` |
   | 2 | `bc1qegtq7ns…` | `ddedc9f5…` | `e47d4949…` |
   | 3 | `bc1qa6te8zc…` | `679de681…` | `0d2a92c6…` |
   | 4 | `bc1qaaneuqy…` | `72eeb2d7…` | `5613931a…` |
   | 5 | `bc1q9e6s4kx…` | `37ebfc8e…` | `92716bbb…` |

3. Le flag est la liste des cinq adresses bech32 **dans l'ordre du fichier d'empreintes**,
   séparées par des virgules sans espaces.

### Le point délicat

Le contenu était **mathématiquement démontré** (cinq préimages sur cinq), mais aucune
variation de format n'était acceptée : permutations d'ordre (chronologique THOR, hauteurs
de blocs), identifiants de balayage ou de paiement, coquilles présentes dans l'énoncé
(`HLB2O26`), séparateurs variés. Le flag stocké ne correspondait à aucune dérivation
possible ; les organisateurs l'ont corrigé en cours d'épreuve, et la réponse « évidente »
(ordre du fichier, virgules) a été acceptée immédiatement après.

**Leçon** : lorsque la réponse est démontrée et que le challenge n'a aucune résolution
dans le classement, l'hypothèse d'un flag incohérent côté auteur doit être envisagée et
signalée aux organisateurs, preuves à l'appui, plutôt que combattue par force brute.

---

## Q6 — Connecting the Dots (200 points)

**Flag** (validé officiellement) :

```
HLB2026{74caf0e22e82c105358bdc912840ef0e08be05d84162fd6aabaf6c85277cfb5b,3f6403fffe71061be38f5d215aa9aa76afd395463463962e84fa17c9d3ddcf7a,d4f8337f921e5607035c7ff781c14dc853830f15dbc777c5e22dceca5fdcde33,0f1ade1492631f42953e748a066bf5281cbed6f3e67ca758536e86d5b50f09d0,f80b0fb8be1b6ee538b51d1947cc9ec8641b51439cda7ab1465c474e717b1f34,42b3c8bf0b963393c30dc2695138b9461a2991c139c364977b449d23aa9f0d5b,c1ac94f7ae75aeac2e22a8e1227dd441971bef1eeacc129def1aaa0a394964cc}
```

### Résolution

1. L'échange initial (visible via l'API midgard de THORChain) est la transaction
   Ethereum `488A4636CC6F147A0A5D574067A73ECC0C17432EFA348E7F5D770BC338252703`
   : 28,16 ETH depuis `0xe4ee9ed6…`, mémo `=:BTC.BTC:bc1q2ckjg8a…:0/10/0`, sortie
   Bitcoin `5983287f…` (0,80965529 BTC, échange progressif 9/9).
2. **Chemin de valeur unique**, chaque identifiant étant revérifié à l'octet près via
   mempool.space :

   ```
   5983287f → 74caf0e2 → 3f6403ff → d4f8337f → 0f1ade14 → f80b0fb8 → 42b3c8bf → c1ac94f7 → bc1qxn7a
   ```

3. **Indice officiel** publié par les organisateurs :

   > « N'incluez ni le haché de la transaction THORChain ni celui de la transaction
   > sortante du portefeuille (bc1qxn7a…). »

   Le flag est donc la liste des **sept transactions intermédiaires** uniquement,
   séparées par des virgules sans espaces.

### Pourquoi cette question était piégeuse

- La transaction `c1ac94f7` possède **trois entrées** : le flux principal de huit
  transactions, la chaîne dominante « DOUT » issue du coffre THORChain (douze
  transactions, mémo OP_RETURN `DOUT:…`, échange de 137,6 ETH), et de la poussière
  organique d'avril.
- La chaîne DOUT est **antérieure** au flux principal (horodatage 1740226675 contre
  1740312577) : tout chemin passant du flux vers le coffre viole la chronologie — c'est
  un artefact d'outil d'analyse, pas un flux réel.
- Les deux chemins plausibles (chronologique et DOUT) ont été longuement explorés avant
  que l'indice officiel ne tranche le périmètre attendu.

**Leçon** : face à un flag « liste d'identifiants de transactions », clarifier tôt le
périmètre exact (faut-il inclure la transaction initiale ? la sortie finale ?) —
l'ambiguïté de format coûte davantage de temps que l'analyse on-chain elle-même.
