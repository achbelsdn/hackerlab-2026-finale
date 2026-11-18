# King of the Throne — Pool A « Cerberus » (attack/defense)

**Épreuve** : King of the Throne, round ouvert le 17 novembre à 10 h 30 UTC.
**Équipe** : DIVISION_0 — **Résultat** : **premier du pool (300 points)**, devant
Root_Guardians, We are legion et Saiyans (0 point).

**Cible** : `10.200.40.2`, joignable par le tunnel WireGuard dédié au round. La box
« Cerberus » expose un service FTP (vsftpd), SSH, un site de monitoring statique sous
nginx et un panneau d'administration Gunicorn sur le port 8080.

---

## Chaîne d'exploitation

### 1. Reconnaissance

```
21/ftp   vsftpd — connexion anonyme acceptée, répertoire /backups lisible
22/ssh   OpenSSH
80/http  nginx — page de monitoring statique
8080     gunicorn — panneau d'administration
```

### 2. FTP anonyme : récupération des identifiants du panneau

L'archive `config-backup.tar.gz` contient `panel.conf` :

```
admin / md5(09b2588e...)
```

Le hachage cède à john the ripper avec le dictionnaire rockyou : mot de passe
`cerberus1`.

### 3. Exécution de code sur le panneau (port 8080)

La page `/system-check` transmet un champ `command=` **sans restriction côté serveur** :
la liste déroulante qui limite les commandes n'existe que dans le navigateur. Il en
résulte une **exécution de code sous l'identité `sysadm` (uid 1001)**.

### 4. Comprendre l'architecture (grâce à la RCE)

La lecture de `entrypoint.sh` révèle cinq flags `FLAG_*` déposés dans des fichiers, et
un fichier de royauté `/root/king.txt` (mode 666, mais `/root` en 700 : root est
indispensable pour y écrire).

### 5. Élévation de privilèges

Le binaire `/usr/local/bin/syscheck` est **SUID root:sysadm** et lit le fichier
`/etc/syscheck.conf` (`-rw-rw---- root:sysadm`, donc **modifiable par le groupe
sysadm**). Chaque ligne `cmd=...` y est exécutée via `system()` : il suffit d'injecter
une commande puis de déclencher `syscheck` pour obtenir une exécution en root.

- Remarque : un bash SUID copié dans `/tmp` ne fonctionne pas (option `nosuid`) —
  `syscheck` reste donc l'unique primitive root de la box.

### 6. Prendre et garder le trône

Le jeton `f71de0c5bbca5a6a` est écrit dans `/root/king.txt`, puis verrouillé :

```bash
chattr +i /root/king.txt
# + /etc/cron.d/king : réécrit le jeton et réapplique chattr +i chaque minute
```

Cette persistance empêche un adversaire devenu root de s'emparer durablement du trône :
il faudrait désactiver notre cron, ce qui laisse des traces et du temps de réaction.

---

## Flags (format : jeton hexadécimal brut, **sans** préfixe `HLB2026{}`)

| Jeton | Points | Emplacement |
|---|---|---|
| `258339be3eafcbcc67afb35c01888c104b029df5aae6e252` | 50 | `/opt/cerberus-panel/.session_secret` |
| `312f15f1c8fe1ee04f0af80a44a28be12784639eebb54edf` | 50 | `/root/.cerberus_license` |
| `7be3c1dfdcc1491de9b80584766568bc06d2b19da11c4dd1` | 50 | `/srv/ftp/backups/.maintenance.log` |
| `d0f7638e110810bba8041da43dfe36a471a318f3a6a0fdce` | 50 | `/opt/cerberus-web/.db_backup.key` (première moitié) |
| `9c20d381435020173a91fce2d77edbfbcac1ecddddf2f091` | 50 | `/opt/cerberus-agent/.collector.pid` (seconde moitié) |

---

## Pièges rencontrés

- **96 caractères hexadécimaux dans `.db_backup.key` = deux flags de 48 concaténés sans
  retour à la ligne** : un simple `cat` fusionne les sorties ; sans découpage, on ne
  voit qu'un bloc unique.
- En King of the Throne, les flags se présentent **sous forme de jeton brut**, sans le
  préfixe `HLB2026{}` habituel du CTFd : chaque mode de jeu a son propre format.
- `dash`/`sh` abandonne le bit SUID à l'exécution, et `/tmp` est monté `nosuid` : tout
  ce qui doit tourner en root passe par `syscheck`.

## Persistance et défense

- `/etc/syscheck.conf` réécrit avec nos commandes : le jeton est restauré si syscheck
  est relancé.
- `/etc/cron.d/king` : réécriture du jeton et `chattr +i` chaque minute.
- La RCE du panneau (`/system-check`) est le point d'entrée re-dérivable à volonté —
  c'est aussi la faille à surveiller en priorité lorsqu'on défend la box.
