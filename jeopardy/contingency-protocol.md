# Contingency Protocol — Override (web3 / blockchain, 200 points)

**Identifiant du challenge** : 189 — **Équipe** : DIVISION_0 — **Statut** : résolue.
**Flag** : `HLB2026{efe2f7684bf731e5322e91068f4c28aa}`

---

## Énoncé

> Sous les sceaux d'un contrat, la vérité sommeille, mais l'horloge avance, et le
> silence veille. Un signe de vie aurait suffi à tout retenir. Prenez le contrôle
> avant le dernier battement, ou les secrets s'éveilleront au jugement du temps.

- Instance dédiée : `172.28.15.2:9030` — « Vault (tableau de bord) », accessible via
  un profil WireGuard dynamique.
- Le tableau de bord « Dossier Varga » expose le contrat, son code source et un RPC
  JSON-RPC limité.

## Reconnaissance

Le contrat déployé est un `Vault` (`0x5fbdb2315678afecb367f032d93f642f64180aa3`,
réseau local, chain ID `31337`) dont `/source` révèle deux fonctions clés :

```solidity
function claimOwnership() external {
    owner = msg.sender;          // aucun contrôle d'accès !
}

function disarm() external {
    require(tx.origin == owner); // le propriétaire peut désarmer
    encryptedCid = 0;
    disarmed = true;
}
```

Deux faiblesses se combinent :

1. **`claimOwnership()` n'a aucune restriction** : n'importe qui peut devenir
   `owner` en envoyant une transaction.
2. **`disarm()` ne vérifie que `tx.origin == owner`** : une fois propriétaire, la
   désactivation du mécanisme est immédiate.

Le dashboard fournit via `/state` le compte joueur (clé privée de challenge) et son
solde. Le RPC du challenge **désactive `eth_sendTransaction`** : il faut donc signer
les transactions localement et diffuser des transactions brutes (`eth_sendRawTransaction`).

## Exploitation

Le solveur (`scripts/disarm.py` dans nos notes locales) :

1. lit `/state` pour récupérer le compte et la clé du challenge (jamais affichée,
   stockée en mode `600`) ;
2. construit et **signe localement** deux transactions :
   - `claimOwnership()` — tx `0x2fca5366…7950e`, incluse au bloc 2 ;
   - `disarm()` — tx `0x1e132dbd…6bb93aa`, incluse au bloc 3 ;
3. vérifie on-chain `isSolved() == true` et `disarmed == true` ;
4. appelle `/flag`, qui révèle le flag puisque `solved=true` et `revealed=false`.

La narration du challenge (« un signe de vie aurait suffi à tout retenir ») décrit
exactement ce mécanisme : un contrat « coffre-fort » à déclenchement différé qui
n'attend qu'une revendication de propriété non protégée.

## Flag

```
HLB2026{efe2f7684bf731e5322e91068f4c28aa}
```

Soumission acceptée sur le challenge **Override** — le portail est passé de 6 à
7 équipes résolvantes.

## Leçons

- **Tout modificateur d'accès absent est une faille** : `claimOwnership()` sans
  `require` est l'équivalent Web3 d'une page d'admin sans authentification.
- **`tx.origin` n'est pas une authentification** : ici il a suffi, mais c'est un
  anti-pattern classique (vulnérable au phishing de contrat) ; un auteur qui
  l'utilise signale souvent une surface attaquable.
- **Un RPC bridé n'est pas un verrou** : désactiver `eth_sendTransaction` ne bloque
  rien si la clé est fournie — la signature locale et `eth_sendRawTransaction`
  contournent la restriction en deux lignes.
- **Hygiène opérationnelle** : le profil WireGuard du challenge a été monté,
  resserré en permissions (`600`), utilisé, puis démonté ; la clé de challenge
  n'apparaît nulle part dans les notes publiées.
