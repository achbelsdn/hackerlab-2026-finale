# Death Match — Battle 07 « Patchwork » (attack/defense)

**Épreuve** : Death Match vs **Alagbara** — **Équipe** : DIVISION_0
**Résultat** : **+200 pts (score 200–0)**, SLA défense 98,3 %, rang 1 au moment du round.

**Périmètre** :
- Attaque : `10.200.45.2` (HTTP 80)
- Défense : `10.200.46.2` (HTTP 80 + SSH 22) — Apache/2.4.68 Debian, PHP 8.4
- App « Patchwork » : portail RH PHP/SQLite, **mêmes sources sur les deux box**.

---

## Chaîne d'exploitation (attaque)

### 1. Login démo

`employee` / `patchwork` — affiché sur `login.php`.

### 2. RCE par injection de commande dans `upload.php`

```php
shell_exec('/usr/local/bin/scandoc ' . escapeshellarg($dest) . ' ' . $orig . ' 2>&1');
```

`$dest` est échappé mais **`$orig` (nom original du fichier côté client) ne l'est pas** →
injection de commande. `scandoc` est un script sh qui echo `$name`.

### 3. Trois pièges d'exfiltration

1. **Le rapport n'affiche que la première ligne / jusqu'à la première espace** de la
   sortie → exfiltrer en `base64 -w0`.
2. Un `/` littéral dans le nom fait échouer le rapport (sortie vide) → naviguer avec
   `cd ..&&cd ..&&cd ..&&cd flags&&base64 -w0 user.txt` au lieu de chemins absolus.
3. **Faux positif classique** : un `$(id)` dans le payload a été expansé par NOTRE shell
   local (`curl -F "..."` double-quoted) — toujours single-quoter les payloads.

### 4. Localisation du flag

`init_db.php` (lu via exfiltration) : flag = `/flags/user.txt`.

### 5. Exfiltration

```
aa2d59b8ff3f180c368f0938954abdb5163737a30850abfe   (cible adverse) → +200 pts
```

Exfiltration des sources par tranches de 200 chars base64 dans le champ `name`
(cap ~255) — script dédié.

**Format** : le flag adverse se soumet **brut** (48 hex) ; le wrapper `HLB2026{...}`
retourne `invalid_flag`.

---

## Deuxième vulnérabilité (audit des sources exfiltrées)

- `help.php?doc=` : `include` après allowlist realpath `docs/`, mais `/proc/` et
  `/etc/php/8.4/apache2/conf.d/` restent atteignables → log-poisoning via
  `/proc/self/fd/2`.
- `lib.php` : cookie `pw_prefs` = `base64(serialized)` → gadget
  `ViewPrefs.__wakeup` → `call_user_func(apply_hook, hook_arg)` (POI).

---

## Patchs appliqués (box défense 10.200.46.2)

1. `upload.php` : `escapeshellarg($orig)` — un nom `x;id;.pdf` reste littéral.
2. `lib.php` : `unserialize(..., ['allowed_classes' => false])` et `__wakeup` vide.
3. `help.php` : `doc` limité à `welcome|accounts|security|upload` — traversals → « not found ».
4. `/flags/user.txt` en 600 + cron root `/etc/cron.d/flagmode` qui re-chmod après chaque
   rotation de flag.
5. Uploads vidés (webshell `shell.php` inclus).
6. Compte démo laissé en place (login 302) et `health.php` = `OK users=2` → **SLA préservé**
   (98,3 %).
7. sshd : `PasswordAuthentication no`, `PermitRootLogin prohibit-password`.

---

## Leçons

- Un champ « traçabilité » passé à `shell_exec` sans `escapeshellarg` = RCE immédiate ;
  le champ **affiché dans un rapport** est aussi un canal d'exfiltration.
- Vérifier TOUJOURS si le payload a été expansé localement avant de conclure à une
  injection serveur.
- En A/D : **patcher la box miroir dès l'identification du vecteur** — les deux box
  exécutent le même code, donc la faille qu'on exploite chez l'adversaire existe chez nous.
- Défenser sans casser le healthcheck : garder les routes attendues fonctionnelles.
