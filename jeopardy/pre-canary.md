# Pre_Canary (crypto, 50 points)

**Identifiant du challenge** : 215 — **Équipe** : DIVISION_0 — **Statut** : résolue.
**Flag** : `HLB2026{w3lc0m3_r0t4te_th3_d1g1ts}`

---

## Donnée fournie

```
FRkPZwNlAag8Z7kwZT5mK8VjqQE5MI45nQAsMQSaZKEmsD==
```

## Dérivation

1. **ROT18**, c'est-à-dire ROT13 pour les lettres et ROT5 pour les chiffres — le
   chiffrement « tourné » classique sur l'alphabet alphanumérique :

   ```
   FRkPZwNlAag8Z7kwZT5mK8VjqQE5MI45nQAsMQSaZKEmsD==
       --ROT18-->  SExCMjAyNnt3M2xjMG0zX3IwdDR0ZV90aDNfZDFnMXRzfQ==
   ```

2. **Décodage Base64** du résultat :

   ```
   HLB2026{w3lc0m3_r0t4te_th3_d1g1ts}
   ```

## Leçons

- Face à une chaîne Base64 dont le décodage donne du charabia, tester **ROT13/ROT18 sur
  la chaîne encodée** (et non sur le décodé) : une couche de César appliquée avant
  l'encodage est un schéma récurrent.
- Le texte du flag (`r0t4te_th3_d1g1ts`) confirme a posteriori la mécanique attendue :
  les flags auto-descriptifs servent d'oracle de validation de l'hypothèse.
