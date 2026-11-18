# Death Match — Battle 07 « Patchwork » (attack/defense)

**Épreuve** : Death Match contre l'équipe **Alagbara** — **Équipe** : DIVISION_0
**Résultat** : **+200 points (score 200 à 0)**, disponibilité de la box défendue de
98,3 %, première place au tableau pendant le round.

**Périmètre** :

- Attaque : `10.200.45.2` (HTTP 80) ;
- Défense : `10.200.46.2` (HTTP 80 et SSH 22) — Apache 2.4.68 sous Debian, PHP 8.4 ;
- Application « Patchwork » : portail RH en PHP/SQLite, avec **les mêmes sources sur
  les deux box**.

---

## Chaîne d'exploitation (attaque)

### 1. Compte de démonstration

`employee` / `patchwork` — affiché directement sur `login.php`.

### 2. Exécution de code par injection de commande dans `upload.php`

```php
shell_exec('/usr/local/bin/scandoc ' . escapeshellarg($dest) . ' ' . $orig . ' 2>&1');
```

Le paramètre `$dest` est correctement échappé, mais **`$orig` (nom original du fichier
côté client) ne l'est pas** : c'est une injection de commande directe. Le programme
`scandoc` est un script shell qui affiche `$name` — la sortie du rapport devient donc un
canal de lecture.

### 3. Trois pièges d'exfiltration

1. **Le rapport n'affiche que la première ligne, jusqu'à la première espace** de la
   sortie : il faut exfiltrer en `base64 -w0` pour obtenir une chaîne continue.
2. Une barre oblique littérale dans le nom du fichier fait échouer le rapport (sortie
   vide) : on navigue avec des enchaînements `cd ..&&cd ..&&cd flags&&base64 -w0
   user.txt` plutôt qu'avec des chemins absolus.
3. **Faux positif classique** : un `$(id)` inséré dans la charge utile a été développé
   par **notre propre shell local** (guillemets doubles autour d'un `curl -F "..."`).
   Toujours placer les charges utiles entre guillemets simples.

### 4. Localisation du flag

La lecture de `init_db.php` (obtenue par exfiltration) indique que le flag se trouve
dans `/flags/user.txt` :

```
aa2d59b8ff3f180c368f0938954abdb5163737a30850abfe   (box adverse)
```

### 5. Exfiltration des sources

Les fichiers du serveur sont extraits par tranches de 200 caractères base64 insérées
dans le champ `name` (limite d'environ 255 caractères), à l'aide d'un script dédié.

---

## Seconde vulnérabilité (audit des sources exfiltrées)

- `help.php?doc=` : la inclusion passe par une liste blanche sur le chemin réel sous
  `docs/`, mais `/proc/` et `/etc/php/8.4/apache2/conf.d/` restent atteignables —
  un empoisonnement de journal via `/proc/self/fd/2` est envisageable.
- `lib.php` : le cookie `pw_prefs` contient `base64(serialized)` ; le gadget
  `ViewPrefs.__wakeup` aboutit à `call_user_func(apply_hook, hook_arg)` — un primitive
  d'appel de fonction arbitraire.

---

## Correctifs appliqués (box de défense 10.200.46.2)

1. `upload.php` : ajout de `escapeshellarg($orig)` — un nom `x;id;.pdf` reste désormais
   littéral dans le rapport.
2. `lib.php` : `unserialize(..., ['allowed_classes' => false])` et `__wakeup` vidé.
3. `help.php` : paramètre `doc` limité à `welcome|accounts|security|upload` ; les
   traversées de répertoire répondent « not found ».
4. `/flags/user.txt` passé en mode 600, avec une tâche cron root (`/etc/cron.d/flagmode`)
   qui réapplique les permissions après chaque rotation de flag.
5. Répertoire d'uploads vidé (le webshell `shell.php` inclus).
6. Le compte de démonstration est conservé (connexion fonctionnelle) et `health.php`
   répond `OK users=2` : **la disponibilité exigée par l'épreuve est préservée**
   (98,3 %).
7. sshd : `PasswordAuthentication no` et `PermitRootLogin prohibit-password`.

---

## Leçons

- Un champ de « traçabilité » transmis à `shell_exec` sans `escapeshellarg` équivaut à
  une exécution de code immédiate ; et lorsqu'un champ est **affiché dans un rapport**,
  il constitue aussi un canal d'exfiltration.
- Vérifier systématiquement si une charge utile a été développée localement avant de
  conclure à une injection côté serveur.
- En attack/defense, **corriger la box miroir dès l'identification du vecteur** : les
  deux machines exécutent le même code, la faille exploitée chez l'adversaire existe
  donc chez soi.
- Défendre sans casser le superviseur : conserver les routes attendues par le
  healthcheck, sinon la disponibilité — et les points — s'effondrent.
