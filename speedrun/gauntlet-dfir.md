# The Gauntlet — SpeedRun DFIR HLB2K26

**Épreuve** : speedrun DFIR sur ELK (`dfir.hackerlab.bj`) — 30 questions, 60 s/question,
max 100 pts, decay 0,5, plancher 0,5, 3 changements de réponse max (la dernière
horodatée compte), avance automatique, injection CTFd.

**Statut** : terminé (18 équipes). Accès capitaine uniquement une fois le speedrun lancé ;
les questions ne sont pas lisibles sans jeton.

---

## Environnement

- **Kibana** : espace `cobra`, rôle `analyst_cobra` (compte d'équipe partagé).
- **Data view** : « DFIR Logs » — motif `.ds-logs-*,.ds-metrics-*`, champ temps `@timestamp`.
  ⚠️ L'alias `logs-*` ne renvoie **aucun shard** pour ce rôle : toujours passer par la
  data view ou les index concrets `.ds-logs-...`.

## Corpus

- **7 169 141 événements**, fenêtre unique : **2026-11-06 09:29:30Z → 13:14:53Z**
  (à poser en absolu dans le sélecteur, pas `now-15m`).
- Un seul hôte : `win-m16q1kog02o`.

| Index | Docs | Usage |
|---|---:|---|
| `.ds-logs-windows.sysmon_operational-*` | 2 825 554 | process, réseau, fichiers (Event 10 = bruit) |
| `.ds-logs-winlog.security-*` | 1 110 195 | 4656/4663/4658 ; 4688 = 109 seulement |
| `.ds-logs-windows.powershell*` | ~815 k | scripts |
| `.ds-logs-network_traffic.flow-*` | 335 113 | flux |
| `.ds-logs-mysql.slowlog-*` | 308 358 | SQL lent |
| `.ds-logs-iis.access-*` | 177 094 | HTTP |
| `.ds-logs-endpoint.alerts-*` | **39** | détecteurs — **à lire en premier** |

## Chaîne d'intrusion reconstituée

1. **IIS WordPress**. Source quasi unique : `192.168.24.128` (176 887 requêtes) ;
   secondaire `192.168.24.200` (207).
2. **11 432 hits** sur `/wp-login.php` ; aussi `/wp-config.php`, `/wp-config.bak`,
   `/wp-config.old`, `/xmlrpc.php`.
3. **Webshell** :
   `/wp-content/uploads/forminator/59_eb3138037cd77a85cda14d6e687009c4/uploads/pkKnndRhc7df-shell.php`
   (20 hits).
4. Parent processus : **`w3wp.exe` → `C:\PHP\php-cgi.exe`** (88 créations) — piste CGI PHP.
5. `2026-11-06T09:56:36Z`, user **`DefaultAppPool`**, parent `cmd.exe` : PowerShell
   `-enc` = **reverse shell TCP vers 192.168.24.128:5555** (Base64 UTF-16LE,
   `TCPClient("192.168.24.128",5555)` + `Invoke-Expression`).
6. **Alertes (39)**, par ordre d'intérêt : Potential Pentesting PowerShell (10),
   Base64 Decoding (6), Downloads (6), Credential Access via Known Utilities (3),
   Obfuscated PowerShell (3), Suspicious PowerShell Execution (3), Registry Hive Dump (2),
   NetCat (1), SeImpersonatePrivilege (1), Startup Persistence wscript (1),
   Scheduled Task (1), Run key (1), Windows.Exploit.FakePipe (1).
7. Comptes Sysmon : `Système`, `DefaultAppPool`, **`svc_update`** (31 756),
   `Administrateur` (1 342), plusieurs `sshd_<pid>`.
8. Destinations Sysmon Event 3 : `192.168.24.200` (3645), `.130` (2222), `.128`
   (14 — le C2), plus bruit multicast/loopback.

## Requêtes KQL types (Discover)

```
data_stream.dataset:endpoint.alerts
process.name:php-cgi.exe
process.parent.name:w3wp.exe
url.path:*shell.php
process.command_line:*192.168.24.128*
process.name:powershell.exe and process.command_line:*enc*
user.name:svc_update
process.name:(reg.exe or regsave or ntdsutil or procdump or net.exe or nc.exe or ncat.exe or wscript.exe or cscript.exe)
event.code:1 and process.name:whoami.exe
```

---

## Leçons speedrun

- **Les 39 alertes d'abord** : le dataset `endpoint.alerts` est minuscule et résume
  toute l'attaque — c'est la carte du corpus.
- Poser la **fenêtre temporelle absolue** dès la première minute ; `now-15m` sur un
  corpus de 2026 ne renvoie rien.
- Préparer les KQL d'avance (webshell, `-enc`, comptes de service) : à 60 s/question,
  la requête doit être prête avant que la question n'apparaisse.
- Le format de réponse (dates, chemins, IP) doit être calé sur le barème affiché —
  la dernière réponse horodatée compte : ne pas « gratter » à la fin.
