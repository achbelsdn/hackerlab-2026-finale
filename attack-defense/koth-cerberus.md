# King of the Throne — Pool A « Cerberus » (attack/defense)

**Épreuve** : KotH, round du 17/11 10:30 UTC — **Équipe** : DIVISION_0 — **Résultat** :
**1er du pool (300 pts)** devant Root_Guardians, We are legion, Saiyans (0).

**Cible** : `10.200.40.2` (tunnel WireGuard dédié au round). Box « Cerberus » :
FTP (vsftpd), SSH, nginx (monitoring statique), panel d'admin Gunicorn sur 8080.

---

## Chaîne d'exploitation

### 1. Recon

```
21/ftp   vsftpd — login anonyme autorisé, répertoire /backups
22/ssh   OpenSSH
80/http  nginx — page de monitoring statique
8080     gunicorn — panel d'administration
```

### 2. FTP anonyme → credentials du panel

`config-backup.tar.gz` contient `panel.conf` :

```
admin / md5(09b2588e...)
```

 Crack john + rockyou → mot de passe en clair : `cerberus1`.

### 3. RCE sur le panel (8080)

La page `/system-check` POST un champ `command=` **non restreint côté serveur** —
le `<select>` limitant les commandes n'existe que côté client → **RCE en `sysadm`
(uid 1001)**.

### 4. Architecture (via RCE)

`entrypoint.sh` révèle 5 flags `FLAG_*` écrits dans des fichiers et un *king file*
`/root/king.txt` (666, mais `/root` en 700 → root requis).

### 5. Escalade root

`/usr/local/bin/syscheck` est **SUID root:sysadm** et lit `/etc/syscheck.conf`
(`-rw-rw---- root:sysadm` → **group-writable** par sysadm). Chaque ligne `cmd=...` est
exécutée via `system()` → injection + trigger = exécution root.

- NB : un SUID bash copié dans `/tmp` ne marche pas (`nosuid`) → `syscheck` reste LE
  primitive root pour tout.

### 6. King (défense du trône)

Token `f71de0c5bbca5a6a` écrit dans `/root/king.txt`, puis :

```bash
chattr +i /root/king.txt
# + /etc/cron.d/king : réécrit le token + chattr +i chaque minute
```

→ défense anti-vol : même si un adversaire obtient root, il doit retomber sur notre
cron pour reprendre le trône.

---

## Flags (format = token hex BRUT, **sans** wrapper `HLB2026{}`)

| Flag | Points | Emplacement |
|---|---|---|
| `258339be3eafcbcc67afb35c01888c104b029df5aae6e252` | 50 | `/opt/cerberus-panel/.session_secret` |
| `312f15f1c8fe1ee04f0af80a44a28be12784639eebb54edf` | 50 | `/root/.cerberus_license` |
| `7be3c1dfdcc1491de9b80584766568bc06d2b19da11c4dd1` | 50 | `/srv/ftp/backups/.maintenance.log` |
| `d0f7638e110810bba8041da43dfe36a471a318f3a6a0fdce` | 50 | `/opt/cerberus-web/.db_backup.key` (1ère moitié) |
| `9c20d381435020173a91fce2d77edbfbcac1ecddddf2f091` | 50 | `/opt/cerberus-agent/.collector.pid` (2e moitié) |

Soumission : `POST https://ctf.hackerlab.bj/api/gamestack/submit {"flag": "..."}` →
réponse `accepted:<name>:<pts>`.

---

## Pièges

- **96 hex dans `.db_backup.key` = DEUX flags de 48 concaténés sans newline** — le `cat`
  a fusionné les sorties ; sans découpage, on ne voit qu'un blob.
- Le flag FTP a été refusé une fois (wrappé `HLB2026{...}`) puis accepté **en brut** :
  en KotH, le format de flag est celui du gamestack, pas celui du CTFd.
- `dash/sh` abandonne le SUID ; `/tmp` est `nosuid` → tout ce qui doit tourner en root
  passe par `syscheck`.

## Persistence / défense

- `/etc/syscheck.conf` réécrit avec nos `cmd` (le token est réécrit si syscheck relancé).
- `/etc/cron.d/king` : token + `chattr +i` chaque minute.
- Le RCE panel (`/system-check`) est le point d'entrée re-dérivable à volonté — c'est
  aussi LA faille à surveiller côté défense.
