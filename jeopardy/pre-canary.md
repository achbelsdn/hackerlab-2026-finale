# Pre_Canary (crypto, 50 pts)

**Challenge ID** : 215 — **Équipe** : DIVISION_0 — **Statut** : résolue (solve équipe 04:44 UTC)
**Flag** : `HLB2026{w3lc0m3_r0t4te_th3_d1g1ts}`

---

## Donnée

Chaîne fournie :

```
FRkPZwNlAag8Z7kwZT5mK8VjqQE5MI45nQAsMQSaZKEmsD==
```

## Dérivation

1. **ROT18** = ROT13 (lettres) + ROT5 (chiffres) — le chiffre « tourné » classique :

   ```
   FRkPZwNlAag8Z7kwZT5mK8VjqQE5MI45nQAsMQSaZKEmsD==
       --ROT18-->  SExCMjAyNnt3M2xjMG0zX3IwdDR0ZV90aDNfZDFnMXRzfQ==
   ```

2. **Base64 decode** du résultat :

   ```
   HLB2026{w3lc0m3_r0t4te_th3_d1g1ts}
   ```

## Leçons

- Devant une chaîne Base64 qui décode en charabia, tester **ROT13/ROT18 sur l'encodé**
  (pas sur le décodé) : la couche de césure avant l'encodage est un classique.
- Le texte du flag (`r0t4te_th3_d1g1ts`) confirme a posteriori la mécanique attendue —
  les flags auto-descriptifs sont un oracle de validation.
